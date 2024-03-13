# Configuração Inicial de um Projeto com NestJS, TypeORM, GraphQL e Microservices

Este guia fornece uma visão geral detalhada para configurar um projeto base utilizando NestJS, TypeORM, GraphQL, e microservices com a biblioteca `@nestjs/microservices`, focando em uma arquitetura de microservices.

## Sumário

- [Configuração Inicial de um Projeto com NestJS, TypeORM, GraphQL e Microservices](#configuração-inicial-de-um-projeto-com-nestjs-typeorm-graphql-e-microservices)
  - [Sumário](#sumário)
  - [Pré-requisitos](#pré-requisitos)
  - [Criação do Workspace do NestJS](#criação-do-workspace-do-nestjs)
    - [1. Instalação da CLI do NestJS](#1-instalação-da-cli-do-nestjs)
    - [2. Criação de um Novo Projeto (Workspace)](#2-criação-de-um-novo-projeto-workspace)
    - [3. Estrutura do Projeto](#3-estrutura-do-projeto)
    - [4. Organização do Workspace](#4-organização-do-workspace)
  - [Geração de um Novo Microservice](#geração-de-um-novo-microservice)
    - [1. Estrutura do Projeto](#1-estrutura-do-projeto)
    - [2. Desenvolvimento Independente](#2-desenvolvimento-independente)
    - [3. Executando o Microservice](#3-executando-o-microservice)
  - [Instalação das Dependências](#instalação-das-dependências)
    - [Verificação das Dependências Instaladas](#verificação-das-dependências-instaladas)

---

## Pré-requisitos

- Node.js (versão 14.x ou superior)
- NPM ou Yarn
- CLI do NestJS instalada globalmente (`npm i -g @nestjs/cli`)

---

## Criação do Workspace do NestJS

O primeiro passo para iniciar um projeto baseado em microservices com NestJS é criar um workspace. Um workspace do NestJS pode hospedar múltiplos projetos, como bibliotecas e aplicações, incluindo microservices.

### 1. Instalação da CLI do NestJS

Se você ainda não tem a CLI do NestJS instalada globalmente em sua máquina, você deve fazer isso primeiro. A CLI facilita a criação, o desenvolvimento e a manutenção de aplicações NestJS.

```bash
npm install -g @nestjs/cli
```

### 2. Criação de um Novo Projeto (Workspace)

Com a CLI do NestJS instalada, você está pronto para criar um novo projeto que atuará como seu workspace. Este workspace será a base para os microservices que você criará posteriormente.

```bash
nest new workspace-name
```

Substitua `workspace-name` pelo nome desejado para o seu workspace. Este comando cria um novo diretório com o nome `workspace-name`, instala as dependências necessárias e configura um projeto NestJS básico.

Durante a criação, a CLI perguntará qual pacote gerenciador você prefere (`npm`, `yarn`, ou `pnpm`). Escolha o que você preferir.

### 3. Estrutura do Projeto

Após a criação, você terá uma estrutura de diretório básica como esta:

```

workspace-name/
├── src/
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test/
├── nest-cli.json
├── package.json
├── tsconfig.build.json
├── tsconfig.json
└── ...
```

Este é o ponto de partida para o seu workspace. A partir daqui, você pode começar a adicionar microservices como aplicativos dentro deste workspace.

### 4. Organização do Workspace

Conforme você adiciona mais microservices, seu workspace se expandirá. Cada microservice terá sua própria pasta dentro do diretório `apps/`, permitindo-lhe manter uma separação clara entre diferentes partes do seu sistema.

---

## Geração de um Novo Microservice

Para adicionar um novo microservice ao seu workspace, utilize o comando `nest g app` da CLI do NestJS. Esse comando gera um novo aplicativo dentro do seu workspace, ideal para a arquitetura de microservices.

```bash
nest g app microservice-name
```

!!!- Substitua `microservice-name` pelo nome que você deseja dar ao seu microservice. Esse comando cria um diretório separado no nível raiz do seu projeto para o microservice, incluindo um esqueleto básico de aplicação NestJS.

### 1. Estrutura do Projeto

Após adicionar um ou mais microservices, a estrutura do seu projeto terá a seguinte aparência:

```
workspace-name/
├── apps/
│   ├── microservice-name/
│   │   ├── src/
│   │   ├── test/
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── tsconfig.build.json
│   │   ├── tsconfig.json
│   │   └── ...
│   └── ...
├── libs/
├── node_modules/
├── nest-cli.json
├── package.json
├── tsconfig.build.json
├── tsconfig.json
└── ...
```

Cada microservice adicionado estará sob o diretório `apps/`, permitindo uma separação clara entre diferentes partes do seu sistema.

### 2. Desenvolvimento Independente

Cada microservice pode ser desenvolvido, construído e executado de forma independente, utilizando os comandos da CLI do NestJS especificamente dentro do diretório do microservice. Isso facilita a gestão de dependências e a execução de microservices separadamente, um aspecto crucial para o desenvolvimento baseado em microservices.

### 3. Executando o Microservice

Para executar um microservice específico, navegue até o diretório do microservice e utilize os comandos da CLI do NestJS:

```bash
cd apps/microservice-name
nest start
```

Este comando inicia o microservice, tornando-o acessível conforme definido em seu módulo principal e controladores.

---

## Instalação das Dependências

Após a criação dos microservices, o próximo passo é garantir que as dependências comuns necessárias para o funcionamento do projeto estejam instaladas. Isso inclui pacotes essenciais como GraphQL, TypeORM e a biblioteca de microservices do NestJS. Vamos detalhar o processo de instalação dessas dependências comuns:

Para instalar as dependências comuns necessárias, execute o seguinte comando:

```bash
npm install @nestjs/graphql @nestjs/typeorm @nestjs/microservices graphql apollo-server-express typeorm
```

Este comando instala os seguintes pacotes:

- `@nestjs/graphql`: Módulo NestJS para suporte a GraphQL.
- `@nestjs/typeorm`: Integração do TypeORM com o NestJS.
- `@nestjs/microservices`: Biblioteca para criar microservices no NestJS.
- `graphql`: Biblioteca GraphQL para Node.js.
- `apollo-server-express`: Um servidor GraphQL compatível com Express.
- `typeorm`: ORM (Object-Relational Mapping) para Node.js e TypeScript.

### Verificação das Dependências Instaladas

Após a conclusão da instalação, verifique se todas as dependências foram instaladas com sucesso. Você pode verificar as dependências instaladas no arquivo `package.json` do seu workspace.
