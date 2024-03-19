# Microservices com Nestjs e RabbitMQ

Visão Geral

Além das arquiteturas de aplicação tradicionais (às vezes chamadas de monolíticas), o Nest suporta nativamente o estilo arquitetônico de desenvolvimento de microserviços. A maioria dos conceitos discutidos em outras partes desta documentação, como injeção de dependência, decoradores, filtros de exceção, pipes, guards e interceptadores, se aplicam igualmente aos microserviços. Sempre que possível, o Nest abstrai os detalhes de implementação para que os mesmos componentes possam ser executados em plataformas baseadas em HTTP, WebSockets e Microserviços. Esta seção aborda os aspectos do Nest específicos para microserviços.

No Nest, um microserviço é fundamentalmente uma aplicação que utiliza uma camada de transporte diferente do HTTP.

O Nest suporta várias implementações de camada de transporte integradas, chamadas de transportadoras, que são responsáveis por transmitir mensagens entre diferentes instâncias de microserviços. A maioria das transportadoras suporta nativamente estilos de mensagem baseados em solicitação-resposta e baseados em eventos. O Nest abstrai os detalhes de implementação de cada transportadora por trás de uma interface canônica para mensagens baseadas em solicitação-resposta e baseadas em eventos. Isso facilita a troca de uma camada de transporte por outra -- por exemplo, para aproveitar os recursos específicos de confiabilidade ou desempenho de uma determinada camada de transporte -- sem impactar o código da sua aplicação.

## Instalação

Para começar a construir microserviços, primeiro instale o pacote necessário:

```bash
$ npm i --save @nestjs/microservices
```

## Introdução

Para instanciar um microserviço, use o método `createMicroservice()` da classe `NestFactory`:

```javascript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
    },
  );
  await app.listen();
}
bootstrap();
```

> **_Dica_**
> Por padrão, os microserviços usam a camada de transporte TCP.

O segundo argumento do método `createMicroservice()` é um objeto de opções. Este objeto pode consistir em dois membros:

- `transport`: Especifica a transportadora (por exemplo, `Transport.NATS`)
- `options`: Um objeto de opções específico da transportadora que determina o comportamento da transportadora

O objeto de opções é específico para a transportadora escolhida. A transportadora TCP expõe as propriedades descritas abaixo. Para outras transportadoras (por exemplo, Redis, MQTT, etc.), consulte o capítulo relevante para uma descrição das opções disponíveis.

- `host`: Nome do host de conexão
- `port`: Porta de conexão
- `retryAttempts`: Número de vezes para tentar enviar a mensagem novamente (padrão: 0)
- `retryDelay`: Atraso entre as tentativas de reenvio da mensagem (ms) (padrão: 0)
- `serializer`: Serializador personalizado para mensagens de saída
- `deserializer`: Deserializador personalizado para mensagens de entrada
- `socketClass`: Um Socket personalizado que estende TcpSocket (padrão: JsonSocket)
- `tlsOptions`: Opções para configurar o protocolo TLS

## Padrões

Os microserviços reconhecem tanto mensagens quanto eventos por meio de padrões. Um padrão é um valor simples, por exemplo, um objeto literal ou uma string. Os padrões são automaticamente serializados e enviados pela rede junto com a parte de dados de uma mensagem. Dessa forma, os remetentes de mensagens e os consumidores podem coordenar quais solicitações são consumidas por quais manipuladores.

### Solicitação-resposta

O estilo de mensagem de solicitação-resposta é útil quando você precisa trocar mensagens entre vários serviços externos. Com esse paradigma, você pode ter certeza de que o serviço realmente recebeu a mensagem (sem a necessidade de implementar manualmente um protocolo de ACK de mensagem). No entanto, o paradigma de solicitação-resposta nem sempre é a melhor escolha. Por exemplo, transportadores de streaming que usam persistência baseada em log, como Kafka ou NATS streaming, são otimizados para resolver uma gama diferente de problemas, mais alinhados com um paradigma de mensagens de eventos (veja abaixo mais detalhes sobre mensagens baseadas em eventos).

Para habilitar o tipo de mensagem de solicitação-resposta, o Nest cria dois canais lógicos - um é responsável por transferir os dados enquanto o outro aguarda respostas de entrada. Para alguns transportes subjacentes, como NATS, esse suporte de canal duplo é fornecido pronto para uso. Para outros, o Nest compensa criando manualmente canais separados. Pode haver sobrecarga para isso, então se você não requer um estilo de mensagem de solicitação-resposta, você deve considerar o uso do método baseado em eventos.

Para criar um manipulador de mensagens baseado no paradigma de solicitação-resposta, use o decorador `@MessagePattern()`, que é importado do pacote `@nestjs/microservices`. Este decorador deve ser usado apenas dentro das classes controladoras, já que elas são os pontos de entrada para sua aplicação. Usá-los dentro de provedores não terá efeito, pois são simplesmente ignorados pelo runtime do Nest.

```javascript
// math.controller.ts

import { Controller } from "@nestjs/common";
import { MessagePattern } from "@nestjs/microservices";

@Controller()
export class MathController {
  @MessagePattern({ cmd: "sum" })
  accumulate(data: number[]): number {
    return (data || []).reduce((a, b) => a + b, 0);
  }
}
```

No código acima, o manipulador de mensagens `accumulate()` ouve por mensagens que preenchem o padrão de mensagem `{ cmd: 'sum' }`. O manipulador de mensagens recebe um único argumento, os dados passados pelo cliente. Neste caso, os dados são um array de números que devem ser acumulados.

#### Respostas Assíncronas

Os manipuladores de mensagens podem responder tanto de forma síncrona quanto assíncrona. Assim, métodos assíncronos são suportados.

```javascript
@MessagePattern({ cmd: 'sum' })
async accumulate(data: number[]): Promise<number> {
  return (data || []).reduce((a, b) => a + b, 0);
}
```

Um manipulador de mensagens também pode retornar um Observable, caso em que os valores do resultado serão emitidos até que o fluxo seja concluído.

```javascript
@MessagePattern({ cmd: 'sum' })
accumulate(data: number[]): Observable<number> {
  return from([1, 2, 3]);
}
```

No exemplo acima, o manipulador de mensagens responderá 3 vezes (com cada item do array).

### Baseado em Eventos

Enquanto o método de solicitação-resposta é ideal para a troca de mensagens entre serviços, ele é menos adequado quando seu estilo de mensagem é baseado em eventos - quando você apenas quer publicar eventos sem esperar por uma resposta. Nesse caso, você não quer a sobrecarga requerida pela solicitação-resposta para manter dois canais.

Suponha que você gostaria de simplesmente notificar outro serviço de que uma certa condição ocorreu nesta parte do sistema. Este é o caso de uso ideal para o estilo de mensagem baseado em eventos.

Para criar um manipulador de eventos, usamos o decorador `@EventPattern()`, que é importado do pacote `@nestjs/microservices`.

```javascript
@EventPattern('user_created')
async handleUserCreated(data: Record<string, unknown>) {
  // Sua lógica para lidar com o evento aqui
}
```

> **Dica**
> Você pode registrar múltiplos manipuladores de eventos para umúnico padrão de evento e todos eles serão automaticamente disparados em paralelo.

O manipulador de eventos `handleUserCreated()` escuta pelo evento 'user_created'. O manipulador de eventos recebe um único argumento, os dados passados pelo cliente (neste caso, um payload de evento que foi enviado pela rede).

Este mecanismo fornece uma maneira poderosa e flexível de implementar a comunicação assíncrona entre diferentes partes de seu sistema, permitindo que serviços reajam a eventos sem a necessidade de uma resposta direta. Esse paradigma é especialmente útil em sistemas distribuídos onde a desacoplamento e a escalabilidade são chaves para o sucesso do projeto.

## Cliente

Uma aplicação cliente Nest pode trocar mensagens ou publicar eventos para um microserviço Nest usando a classe `ClientProxy`. Esta classe define vários métodos, como `send()` (para mensagens de solicitação-resposta) e `emit()` (para mensagens baseadas em eventos) que permitem a comunicação com um microserviço remoto. Obtenha uma instância desta classe de uma das seguintes maneiras.

Uma técnica é importar o `ClientsModule`, que expõe o método estático `register()`. Este método aceita um argumento que é um array de objetos representando transportadores de microserviços. Cada um desses objetos tem uma propriedade `name`, uma propriedade opcional `transport` (o padrão é `Transport.TCP`), e uma propriedade opcional de opções específicas do transportador.

A propriedade `name` serve como um token de injeção que pode ser usado para injetar uma instância de `ClientProxy` onde necessário. O valor da propriedade `name`, como um token de injeção, pode ser uma string arbitrária ou símbolo JavaScript, conforme descrito aqui.

A propriedade `options` é um objeto com as mesmas propriedades que vimos no método `createMicroservice()` anteriormente.

```javascript
@Module({
  imports: [
    ClientsModule.register([
      { name: 'MATH_SERVICE', transport: Transport.TCP },
    ]),
  ]
  ...
})
```

Uma vez que o módulo tenha sido importado, podemos injetar uma instância do `ClientProxy` configurado conforme especificado através das opções do transportador 'MATH_SERVICE' mostradas acima, usando o decorador `@Inject()`.

```javascript
constructor(
  @Inject('MATH_SERVICE') private client: ClientProxy,
) {}
```

> **Dica**
> As classes `ClientsModule` e `ClientProxy` são importadas do pacote `@nestjs/microservices`.

Às vezes, podemos precisar buscar a configuração do transportador de outro serviço (digamos, um `ConfigService`), em vez de codificá-la diretamente em nossa aplicação cliente. Para fazer isso, podemos registrar um provedor personalizado usando a classe `ClientProxyFactory`. Esta classe tem um método estático `create()`, que aceita um objeto de opções do transportador, e retorna uma instância personalizada de `ClientProxy`.

```javascript
@Module({
  providers: [
    {
      provide: 'MATH_SERVICE',
      useFactory: (configService: ConfigService) => {
        const mathSvcOptions = configService.getMathSvcOptions();
        return ClientProxyFactory.create(mathSvcOptions);
      },
      inject: [ConfigService],
    }
  ]
  ...
})
```

> **Dica**
> A `ClientProxyFactory` é importada do pacote `@nestjs/microservices`.

Outra opção é usar o decorador de propriedade `@Client()`.

```javascript
@Client({ transport: Transport.TCP })
client: ClientProxy;
```

> **Dica**
> O decorador `@Client()` é importado do pacote `@nestjs/microservices`.

Usar o decorador `@Client()` não é a técnica preferida, pois é mais difícil de testar e mais difícil de compartilhar uma instância do cliente.

O `ClientProxy` é preguiçoso. Ele não inicia uma conexão imediatamente. Em vez disso, será estabelecido antes da primeira chamada ao microserviço e, em seguida, reutilizado em cada chamada subsequente. No entanto, se você quiser atrasar o processo de inicialização da aplicação até que uma conexão seja estabelecida, você pode iniciar manualmente uma conexão usando o método `connect()` do objeto `ClientProxy` dentro do gancho de ciclo de vida `OnApplicationBootstrap`.

```javascript
async onApplicationBootstrap() {
  await this.client.connect();
}
```

Se a conexão não puder ser criada, o método `connect()` será rejeitado com o objeto de erro correspondente.

### Enviando mensagens

O `ClientProxy` expõe um método `send()`. Este método é destinado a chamar o microserviço e retorna um Observable com sua resposta. Assim, podemos nos inscrever facilmente nos valores emitidos.

```javascript
accumulate(): Observable<number> {
  const pattern = { cmd: 'sum' };
  const payload = [1, 2, 3];
  return this.client.send<number>(pattern, payload);
}
```

O método `send()` aceita dois argumentos, padrão e carga útil. O padrão deve corresponder a um definido em um decorador `@MessagePattern()`. A carga útil é uma mensagem que queremos transmitir ao microserviço remoto. Este método retorna um Observable frio, o que significa que você tem que se inscrever explicitamente nele antes que a mensagem seja enviada.

### Publicando eventos

Para enviar um evento, use o método `emit()` do objeto `ClientProxy`. Este método publica um evento no message broker.

```javascript
async publish() {
  this.client.emit<number>('user_created', new UserCreatedEvent());
}
```

O método `emit()` aceita dois argumentos, padrão e carga útil. O padrão deve corresponder a um definido em um decorador `@EventPattern()`. A carga útil é um payload de evento que queremos transmitir ao microserviço remoto. Este método retorna um Observable quente (diferentemente do Observable frio retornado por `send()`), o que significa que, quer você se inscreva explicitamente no Observable ou não, o proxy tentará entregar o evento imediatamente.

## Exemplo

Vamos criar um exemplo detalhado de configuração e comunicação entre dois microserviços chamados "user" e "auth", utilizando RabbitMQ, GraphQL e TypeORM.

Primeiro, vamos configurar o microserviço "user":

1. Crie um novo projeto NestJS para o microserviço "user":

```bash
nest new user-service
```

2. Instale os pacotes necessários:

```bash
cd user-service
npm install @nestjs/graphql @nestjs/typeorm graphql-tools graphql typeorm
npm install @nestjs/microservices amqplib
```

3. Configure o RabbitMQ no arquivo `user-service/src/main.ts`:

```typescript
import { NestFactory } from "@nestjs/core";
import { Transport } from "@nestjs/microservices";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.RMQ,
    options: {
      urls: ["amqp://localhost:5672"],
      queue: "user_queue",
      queueOptions: {
        durable: false,
      },
    },
  });
  await app.listen(() => console.log("User Microservice is listening"));
}
bootstrap();
```

4. Crie uma entidade de exemplo para usuário em `user-service/src/user/user.entity.ts`:

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // Outros campos e métodos podem ser adicionados conforme necessário
}
```

5. Configure o TypeORM no arquivo `user-service/src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { User } from "./user/user.entity";

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: "sqlite",
      database: ":memory:", // Use um banco de dados adequado para produção
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

6. Implemente o GraphQL no arquivo `user-service/src/user/user.resolver.ts`:

```typescript
import { Resolver, Query } from "@nestjs/graphql";
import { User } from "./user.entity";
import { Repository } from "typeorm";
import { InjectRepository } from "@nestjs/typeorm";

@Resolver(() => User)
export class UserResolver {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>
  ) {}

  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userRepository.find();
  }
}
```

7. Crie um Dockerfile para o microserviço "user" para facilitar a execução em um contêiner Docker, se necessário.

Agora, vamos configurar o microserviço "auth" para se comunicar com o microserviço "user":

1. Siga os passos 1-4 para configurar um novo projeto NestJS para o microserviço "auth".

2. Adicione o cliente RabbitMQ ao arquivo `auth-service/src/main.ts`:

```typescript
import { NestFactory } from "@nestjs/core";
import { Transport } from "@nestjs/microservices";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.RMQ,
    options: {
      urls: ["amqp://localhost:5672"],
      queue: "user_queue",
      queueOptions: {
        durable: false,
      },
    },
  });
  await app.listen(() => console.log("Auth Microservice is listening"));
}
bootstrap();
```

3. Crie uma conexão com o microserviço "user" no arquivo `auth-service/src/auth/auth.service.ts`:

```typescript
import { Injectable } from "@nestjs/common";
import {
  ClientProxy,
  ClientProxyFactory,
  Transport,
} from "@nestjs/microservices";

@Injectable()
export class AuthService {
  private client: ClientProxy;

  constructor() {
    this.client = ClientProxyFactory.create({
      transport: Transport.RMQ,
      options: {
        urls: ["amqp://localhost:5672"],
        queue: "user_queue",
      },
    });
  }

  async getUsers(): Promise<any[]> {
    return this.client.send("users", {}).toPromise();
  }
}
```

4. Implemente uma rota GraphQL no arquivo `auth-service/src/auth/auth.resolver.ts` para chamar o serviço "user":

```typescript
import { Resolver, Query } from "@nestjs/graphql";
import { AuthService } from "./auth.service";

@Resolver()
export class AuthResolver {
  constructor(private readonly authService: AuthService) {}

  @Query(() => [Object])
  async users(): Promise<any[]> {
    return this.authService.getUsers();
  }
}
```

5. Adicione a configuração GraphQL ao arquivo `auth-service/src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { GraphQLModule } from "@nestjs/graphql";
import { AuthResolver } from "./auth/auth.resolver";
import { AuthService } from "./auth/auth.service";

@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: true,
    }),
  ],
  providers: [AuthResolver, AuthService],
})
export class AppModule {}
```

6. Crie um Dockerfile para o microserviço "auth" se necessário.

O RabbitMQ é um middleware de mensagens de código aberto que funciona como um intermediário entre remetentes (produtores) e receptores (consumidores) de mensagens. Ele implementa o protocolo AMQP (Advanced Message Queuing Protocol), que permite a comunicação assíncrona entre aplicativos distribuídos.

No contexto dos microserviços "user" e "auth", o RabbitMQ será utilizado para facilitar a comunicação assíncrona entre eles. Vamos detalhar o processo:

1. **Remetente (Produtor) - Microserviço "User"**:
   - Quando o microserviço "User" deseja enviar uma mensagem (por exemplo, obter usuários), ele cria um cliente RabbitMQ usando o pacote `@nestjs/microservices`.
   - O cliente RabbitMQ é configurado para se conectar ao servidor RabbitMQ e publicar mensagens na fila "user_queue".
   - Ao chamar o método `this.client.send('users', {})` (no exemplo, para obter usuários), o microserviço "User" envia uma mensagem para a fila "user_queue" com o padrão `'users'`.

2. **Fila RabbitMQ ("user_queue")**:
   - A fila "user_queue" é criada no servidor RabbitMQ e aguarda a chegada de mensagens enviadas pelo microserviço "User".
   - Quando uma mensagem é publicada na fila "user_queue", o RabbitMQ a armazena temporariamente até que um consumidor a processe.

3. **Receptor (Consumidor) - Microserviço "Auth"**:
   - O microserviço "Auth" atua como um consumidor de mensagens da fila "user_queue".
   - Ele cria um cliente RabbitMQ semelhante ao do microserviço "User", configurado para se conectar ao servidor RabbitMQ e consumir mensagens da fila "user_queue".
   - Ao inicializar, o microserviço "Auth" começa a ouvir a fila "user_queue" e espera a chegada de novas mensagens.
   - Quando uma mensagem é recebida da fila "user_queue", o microserviço "Auth" a processa conforme necessário.

4. **Mensagem Transmitida**:
   - No exemplo fornecido, a mensagem transmitida é uma solicitação para obter usuários.
   - Ela contém um padrão `'users'`, que é usado para identificar o tipo de mensagem que está sendo enviada.
   - No caso de uma solicitação para obter usuários, nenhum payload adicional foi fornecido (`{}`), mas poderia incluir dados relevantes para a consulta, como filtros ou parâmetros de paginação.
   - O microserviço "Auth" recebe essa mensagem e a processa, retornando a resposta apropriada para o microserviço "User", se necessário.

