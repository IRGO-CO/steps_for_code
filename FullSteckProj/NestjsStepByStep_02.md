## Sumário

- [Sumário](#sumário)
  - [Configuração do TypeORM](#configuração-do-typeorm)
  - [Configuração do GraphQL](#configuração-do-graphql)

---

### Configuração do TypeORM

O TypeORM é uma ferramenta extremamente poderosa para trabalhar com bancos de dados relacionais em aplicações Node.js. Vamos detalhar como configurá-lo em um microsserviço NestJS.

1. **Instale o TypeORM**:
   Certifique-se de ter o TypeORM instalado no seu projeto. Se ainda não tiver instalado, use o seguinte comando:

   ```bash
   npm install --save @nestjs/typeorm typeorm
   ```

2. **Configuração da Conexão com o Banco de Dados**:
   No arquivo `app.module.ts` do seu serviço, importe o módulo `TypeOrmModule` e configure a conexão com o banco de dados. Aqui está um exemplo de como isso pode ser feito:

   ```typescript
   import { Module } from "@nestjs/common";
   import { TypeOrmModule } from "@nestjs/typeorm";

   @Module({
     imports: [
       TypeOrmModule.forRoot({
         type: "mysql", // tipo do banco de dados (MySQL, PostgreSQL, etc.)
         host: "localhost",
         port: 3306,
         username: "root",
         password: "sua_senha",
         database: "nome_do_banco_de_dados",
         entities: [__dirname + "/**/*.entity{.ts,.js}"], // caminho para suas entidades
         synchronize: true, // sincronizar as entidades com o banco de dados (apenas em ambiente de desenvolvimento)
       }),
     ],
   })
   export class AppModule {}
   ```

3. **Defina Suas Entidades**:
   Crie suas entidades utilizando as classes do TypeORM. Aqui está um exemplo básico de uma entidade de usuário (`user.entity.ts`):

   ```typescript
   import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

   @Entity()
   export class User {
     @PrimaryGeneratedColumn()
     id: number;

     @Column()
     name: string;

     @Column()
     email: string;
   }
   ```

### Configuração do GraphQL

O GraphQL é uma linguagem de consulta para APIs e um tempo de execução para executar essas consultas com seus dados existentes. Vamos detalhar como configurá-lo em um microsserviço NestJS.

1. **Instale o GraphQL**:
   Certifique-se de ter o pacote `@nestjs/graphql` instalado no seu projeto. Se ainda não tiver instalado, use o seguinte comando:

   ```bash
   npm install --save @nestjs/graphql graphql-tools graphql
   ```

2. **Configuração do Módulo GraphQL**:
   No arquivo `app.module.ts` do seu serviço, importe o módulo `GraphQLModule` e configure-o. Aqui está um exemplo básico de como isso pode ser feito:

   ```typescript
   import { Module } from "@nestjs/common";
   import { GraphQLModule } from "@nestjs/graphql";

   @Module({
     imports: [
       GraphQLModule.forRoot({
         autoSchemaFile: true,
       }),
     ],
   })
   export class AppModule {}
   ```

   - `autoSchemaFile`: Este parâmetro permite que o NestJS gere automaticamente um esquema GraphQL com base nos resolvers e tipos definidos na aplicação.

3. **Crie Resolvers GraphQL**:
   Para cada serviço que utiliza GraphQL, crie resolvers para suas consultas, mutações, etc. Um resolver é uma função que recebe uma requisição GraphQL e retorna os dados correspondentes. Aqui está um exemplo de um resolver de usuário:

   ```typescript
   import { Resolver, Query } from "@nestjs/graphql";
   import { UserService } from "./user.service";

   @Resolver("User")
   export class UserResolver {
     constructor(private readonly userService: UserService) {}

     @Query("users")
     async getUsers() {
       return this.userService.findAll();
     }
   }
   ```

4. **Defina Tipos de Entidade GraphQL**:
   Para cada entidade do TypeORM que deseja expor através do GraphQL, defina os tipos GraphQL correspondentes. Isso pode ser feito usando a anotação `@ObjectType()` do NestJS. Aqui está um exemplo de como definir um tipo de usuário:

   ```typescript
   import { ObjectType, Field, Int } from "@nestjs/graphql";

   @ObjectType()
   export class User {
     @Field((type) => Int)
     id: number;

     @Field()
     name: string;

     @Field()
     email: string;
   }
   ```

   - `@ObjectType()`: Esta anotação define uma classe como um tipo GraphQL.
   - `@Field()`: Esta anotação define os campos que serão expostos no esquema GraphQL.
