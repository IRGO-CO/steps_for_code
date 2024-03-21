# Data Flow

## Exemplo simplificado do fluxo de dados em uma api nestjs com graphql e rabbitMQ

### Cenário

Temos dois microservices: `DistanceConverterMicroservice` e `DistanceCalculatorMicroservice`. Cada um desses microservices representa uma operação matemática diferente. O `DistanceConverterMicroservice` recebe uma solicitação GraphQL com uma distância em metros, converte essa distância em quilômetros e persiste o resultado. Em seguida, envia uma mensagem para o `DistanceCalculatorMicroservice`, que recebe o valor em quilômetros, converte-o para milhas e persiste o resultado final.

### Relação entre RabbitMQ e solicitações GraphQL:

1. Um cliente envia uma solicitação GraphQL para o `DistanceConverterMicroservice`, passando uma distância em metros.

2. O `DistanceConverterMicroservice` recebe a solicitação GraphQL com a distância em metros e a converte em quilômetros. Por exemplo, se a distância for 500 metros, o resultado será 0.5 quilômetros. Ele persiste o resultado no banco de dados e envia uma mensagem para uma fila RabbitMQ indicando que a conversão foi concluída.

3. O `DistanceCalculatorMicroservice` está ouvindo essa fila RabbitMQ e consome a mensagem enviada pelo `DistanceConverterMicroservice`.

4. Ao receber a mensagem, o `DistanceCalculatorMicroservice` extrai o valor em quilômetros da mensagem, realiza a conversão para milhas (1 quilômetro ≈ 0.621371 milhas) e persiste o resultado final no banco de dados.

5. O processo é concluído com sucesso, e o `DistanceCalculatorMicroservice` está pronto para lidar com outras conversões semelhantes no futuro.

Este cenário simplificado demonstra como os microservices podem trabalhar em conjunto para processar dados de acordo com diferentes operações matemáticas, persistindo os resultados intermediários e finais em seus respectivos bancos de dados.

Vou exemplificar o cenário descrito usando TypeScript e NestJS para criar os microservices e o RabbitMQ como broker de mensagens assíncronas.

Primeiro, vou mostrar como seria a configuração do RabbitMQ com NestJS, e então mostrarei os códigos para os dois microservices: `DistanceConverterMicroservice` e `DistanceCalculatorMicroservice`.

### Configuração do RabbitMQ com NestJS:

> #### rabbitmq.config.ts
>
> > Este arquivo contém a configuração do RabbitMQ para o microservice. Ele define os parâmetros de conexão, como a URL do RabbitMQ, o nome da fila e outras opções relevantes para a comunicação assíncrona entre os microservices.
> > .

```typescript
// rabbitmq.config.ts

import { Transport } from "@nestjs/microservices";

export const rabbitMQConfig = {
  transport: Transport.RMQ,
  options: {
    urls: ["amqp://localhost:5672"], // URL do RabbitMQ
    queue: "distance_queue", // Nome da fila
    queueOptions: {
      durable: false,
    },
  },
};
```

---

### Microservice 1: `DistanceConverterMicroservice`

> #### 1 - main.ts
>
> > Este arquivo é o ponto de entrada do microservice. Ele é responsável por inicializar o servidor NestJS como um microservice e conectar-se ao RabbitMQ para troca de mensagens.
> > .

```typescript
// main.ts

import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { rabbitMQConfig } from "./rabbitmq.config";

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, rabbitMQConfig);
  await app.listenAsync();
}
bootstrap();
```

> #### distance-converter.service.ts
>
> > Este arquivo contém a implementação do serviço `DistanceConverterService`. Este serviço é responsável por receber uma distância em metros, converter essa distância em quilômetros, persistir o resultado e enviar uma mensagem para o microservice `DistanceCalculatorMicroservice` através do RabbitMQ.
> > .

```typescript
// distance-converter.service.ts

import { Injectable } from "@nestjs/common";
import { rabbitMQConfig } from "./rabbitmq.config";
import { ClientProxy, ClientProxyFactory } from "@nestjs/microservices";

@Injectable()
export class DistanceConverterService {
  private client: ClientProxy;

  constructor() {
    this.client = ClientProxyFactory.create(rabbitMQConfig);
  }

  async processDistance(distanceInMeters: number): Promise<void> {
    // Converte metros para quilômetros
    const distanceInKilometers = distanceInMeters / 1000;

    // Persiste o resultado (simulado)
    // código para persistir em um banco de dados

    // Envia mensagem para o microservice 2 (DistanceCalculatorMicroservice)
    this.client.emit("distance_converted", distanceInKilometers);
  }
}
```

---

### Microservice 2: `DistanceCalculatorMicroservice`

> #### 2 - main.ts
>
> > Este arquivo é o ponto de entrada do microservice. Ele é responsável por inicializar o servidor NestJS como um microservice e conectar-se ao RabbitMQ para troca de mensagens.
> > .

```typescript
// main.ts

import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { rabbitMQConfig } from "./rabbitmq.config";

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, rabbitMQConfig);
  await app.listenAsync();
}
bootstrap();
```

> #### distance-calculator.service.ts
>
> > Este arquivo contém a implementação do serviço `DistanceCalculatorService`. Este serviço é responsável por receber a mensagem do microservice `DistanceConverterMicroservice` contendo a distância em quilômetros, converter essa distância em milhas, persistir o resultado final e finalizar o processamento.
> > .

```typescript
// distance-calculator.service.ts

import { Injectable } from "@nestjs/common";
import { rabbitMQConfig } from "./rabbitmq.config";
import { ClientProxy, ClientProxyFactory } from "@nestjs/microservices";

@Injectable()
export class DistanceCalculatorService {
  private client: ClientProxy;

  constructor() {
    this.client = ClientProxyFactory.create(rabbitMQConfig);
    this.setupListeners();
  }

  setupListeners() {
    // Escuta a fila RabbitMQ para receber mensagens
    this.client.subscribe(
      "distance_converted",
      (distanceInKilometers: number) => {
        // Converte quilômetros para milhas
        const distanceInMiles = distanceInKilometers * 0.621371;

        // Persiste o resultado (simulado)
        // código para persistir em um banco de dados
      }
    );
  }
}
```

Estes são apenas esboços para ilustrar o funcionamento dos microservices. Lembre-se de configurar corretamente o RabbitMQ, bem como os serviços de banco de dados e outros detalhes de implementação conforme necessário. Esses códigos são simplificados e precisarão de ajustes para atender aos requisitos específicos do seu projeto.
