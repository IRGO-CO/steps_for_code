# Criação e Configuração de um Repositório Git com Husky

!!! - Este passo a passo guia você através do processo de criação de um repositório Git de forma profissional, incluindo a configuração de hooks com Husky para manter a qualidade do código.

## Sumário

- [Criação e Configuração de um Repositório Git com Husky](#criação-e-configuração-de-um-repositório-git-com-husky)
  - [Sumário](#sumário)
    - [1. Inicialização do Repositório Git](#1-inicialização-do-repositório-git)
    - [2. Criação do `.gitignore`](#2-criação-do-gitignore)
    - [3. Criação do `README.md`](#3-criação-do-readmemd)
    - [5. Adição de Arquivos ao Staging Area](#5-adição-de-arquivos-ao-staging-area)
    - [6. Confirmação das Mudanças](#6-confirmação-das-mudanças)
    - [7. Criação do Repositório Remoto](#7-criação-do-repositório-remoto)
    - [8. Envio das Mudanças para o Repositório Remoto](#8-envio-das-mudanças-para-o-repositório-remoto)
    - [9. Configuração do Husky](#9-configuração-do-husky)
    - [10. Configuração do Lint-Staged](#10-configuração-do-lint-staged)

### 1. Inicialização do Repositório Git

Inicie seu projeto criando um diretório e inicializando o Git:

```bash
mkdir meu_projeto
cd meu_projeto
git init
```

### 2. Criação do `.gitignore`

Crie um `.gitignore` para especificar arquivos e diretórios que o Git deve ignorar:

```bash
touch .gitignore
```

Adicione ao `.gitignore` arquivos como `node_modules`, `.env`, etc.

### 3. Criação do `README.md`

O `README.md` é crucial para explicar o propósito do seu projeto, como configurá-lo e usá-lo:

```bash
touch README.md
```

!!!- Escreva uma descrição simples, requisitos, instruções de instalação e uso.

### 5. Adição de Arquivos ao Staging Area

**Adicione** suas mudanças ao staging area do Git:

```bash
git add .
```

### 6. Confirmação das Mudanças

Confirme as mudanças adicionadas com um commit:

```bash
git commit -m "Mensagem descritiva do commit"
```

### 7. Criação do Repositório Remoto

Crie um repositório no GitHub, GitLab, Bitbucket ou outro serviço de hospedagem Git e vincule-o ao seu repositório local:

```bash
git remote add origin <URL_DO_REPOSITÓRIO_REMOTO>
```

### 8. Envio das Mudanças para o Repositório Remoto

Envie seus commits para o repositório remoto:

```bash
git push -u origin master
```

### 9. Configuração do Husky

Instale o Husky para usar hooks do Git:

```bash
npm install husky --save-dev
```

Configure os hooks desejados no `package.json`:

```json
"husky": {
  "hooks": {
    "pre-commit": "lint-staged",
    "pre-push": "npm test"
  }
}
```

### 10. Configuração do Lint-Staged

Para garantir que apenas os arquivos prontos sejam commitados, configure o Lint-Staged:

```bash
npm install lint-staged --save-dev
```

Adicione a configuração ao `package.json`:

```json
"lint-staged": {
  "*.js": ["eslint --fix", "git add"]
}
```
