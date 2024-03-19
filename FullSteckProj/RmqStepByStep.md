# RabbitMQ Step By Step

## Funções

### Cadastro de Usuário

1. O **Microservice-1 (Autenticação)** recebe uma solicitação de cadastro de um novo usuário.
2. Ele processa essa solicitação, criando o usuário no seu banco de dados e atribuindo uma classe ao usuário.
3. Após o cadastro bem-sucedido, o Microservice-1 publica um evento de "cadastro" no RabbitMQ, contendo os dados do usuário recém-criado e sua classe.

### Autenticação e Permissões

   1. O **Microservice-1 (Autenticação)** também é responsável pela autenticação de usuários.
   2. Quando um usuário tenta fazer login, ele emite um evento de "login" no RabbitMQ, contendo as credenciais do usuário.
   3. O **Microservice-1** assina esse evento de "login" e verifica a classe do usuário no seu banco de dados para conceder as devidas permissões.
   4. Se o usuário for autenticado com sucesso, o Microservice-1 emite um evento de "autenticação" no RabbitMQ, indicando que o usuário foi autenticado e suas permissões foram verificadas.

### 3. Gerenciamento de Pedidos

   1. O **Microservice-2 (Gerenciamento de Pedidos)** é responsável por lidar com os pedidos dos usuários autenticados.
   2. Quando um usuário autenticado cria ou atualiza um pedido, o Microservice-2 processa essa ação e atualiza seu banco de dados com as informações do pedido.
   3. Não há comunicação direta entre Microservice-2 e Microservice-1 para autenticação. Em vez disso, o Microservice-2 confia nos eventos de "login" e "autenticação" publicados pelo Microservice-1 para garantir a autenticidade do usuário.

### 4. Envio de Pedidos

   1. O **Microservice-3 (Envio)** é responsável pelo processamento e envio dos pedidos.
   2. Quando um pedido está pronto para ser enviado, o Microservice-2 emite um evento de "envio" no RabbitMQ, contendo os detalhes do pedido.
   3. O Microservice-3 assina esse evento de "envio" e inicia o processo de envio do pedido, atualizando seu banco de dados conforme necessário.

### Resumo

- O cadastro de usuários e a autenticação são tratados pelo Microservice-1 (Autenticação), que publica eventos relacionados no RabbitMQ.
- Os outros microservices (como o Microservice-2 para Gerenciamento de Pedidos e Microservice-3 para Envio) assinam esses eventos para realizar ações correspondentes, mantendo assim a independência e escalabilidade de cada microservice.
- Essa abordagem de mensageria assíncrona facilita a comunicação entre os microservices e permite que eles funcionem de forma distribuída e eficiente.
