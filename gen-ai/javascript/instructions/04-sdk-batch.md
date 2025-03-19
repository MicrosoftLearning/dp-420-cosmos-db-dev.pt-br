---
lab:
  title: 04 – Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL

A classe `TransactionalBatch` no SDK do JavaScript para Azure Cosmos DB fornece funcionalidade para compor e executar operações em lote dentro da mesma chave de partição lógica. Usando esse recurso, você pode executar várias operações em uma única transação e garantir que todas ou nenhuma das operações seja concluída.

Neste laboratório, você usará o SDK do JavaScript para executar duas operações em lote de item duplo em que você tenta criar dois itens como uma única unidade lógica.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md).

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#import-the-azurecosmos-library). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo a função de **Colaborador de Dados Internos do Cosmos DB**.

## Importar a biblioteca @azure/cosmos

A biblioteca **@azure/cosmos** está disponível no **npm** para facilitar a instalação em seus projetos JavaScript.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/04-sdk-batch**.

1. Abra o menu de contexto da pasta **javascript/04-sdk-batch** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **javascript/04-sdk-batch**.

1. Inicializar um novo projeto de Node.js:

    ```bash
    npm init -y
    ```

1. Instale o pacote [@azure/cosmos][npmjs.com/package/@azure/cosmos] usando o seguinte comando:

    ```bash
    npm install @azure/cosmos
    ```

1. Instale a biblioteca [@azure/identity][npmjs.com/package/@azure/identity], que nos permite usar a autenticação do Azure para nos conectarmos ao workspace do Azure Cosmos DB, usando o seguinte comando:

    ```bash
    npm install @azure/identity
    ```

## Usar a biblioteca @azure/cosmos

Depois que a biblioteca do Azure Cosmos DB do SDK do Azure para JavaScript tiver sido importada, você poderá usar imediatamente suas classes para se conectar a uma conta do Azure Cosmos DB for NoSQL. A classe **CosmosClient** é a classe principal usada para fazer a conexão inicial com uma conta do Azure Cosmos DB for NoSQL.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/04-sdk-batch**.

1. Abra o arquivo JavaScript vazio chamado **script.js**.

1. Adicione as seguintes instruções `require` para importar as bibliotecas **@azure/cosmos** e **@azure/identity**:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. Adicione as variáveis chamadas **ponto de extremidade** e **credencial** e defina o valor do **ponto de extremidade** como o **ponto de extremidade** da conta do Azure Cosmos DB criada anteriormente. A variável **credencial** deve ser definida como uma nova instância da classe **DefaultAzureCredential**:

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; Por exemplo, se o seu ponto de extremidade for: **https://dp420.documents.azure.com:443/**, a instrução seria: **const endpoint = "https://dp420.documents.azure.com:443/";.**

1. Adicione uma nova variável chamada **cliente** e inicialize-a como uma nova instância da classe **CosmosClient** usando as variáveis **ponto de extremidade** e **credencial** :

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. Adicione o seguinte código para criar um banco de dados e um contêiner, caso ainda não existam:

    ```javascript
    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. O arquivo **script.js** ficará assim:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Salve** o arquivo **script.js**.

1. Antes de executar o script, você deve fazer logon no Azure usando o comando `az login`. Na janela do terminal, execute:

    ```bash
    az login
    ```

1. Execute um script para criar o banco de dados e o contêiner:

    ```bash
    node script.js
    ```

1. Alterne para a janela do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o novo nó do contêiner **products** dentro da árvore de navegação **API NoSQL**.

## Como criar um lote transacional

Primeiro, vamos criar um lote transacional simples que adiciona dois produtos fictícios ao contêiner. Esse lote vai inserir um assento usado e um guidão enferrujado no contêiner com o mesmo identificador de categoria “acessórios usados”. Ambos os itens têm a mesma chave de partição lógica, garantindo uma operação em lote bem-sucedida.

1. Volte para o **Visual Studio Code**. Se ainda não estiver aberto, abra o arquivo de código **script.js** na pasta **javascript/04-sdk-batch**.

1. Defina os dois itens de produto a serem inseridos no lote transacional:

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    ```

1. Crie um lote transacional para a mesma chave de partição lógica e adicione os itens:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = container.items.batch(partitionKey)
        .create(saddle)
        .create(handlebar);
    ```

1. Execute o lote e imprima o status da operação:

    ```javascript
    const response = await batch.execute();
    console.log(`Status: ${response.statusCode}`);
    ```

1. Quando terminar, seu arquivo de código deverá incluir:
  
    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: saddle },
            { operationType: BulkOperationType.Create, resourceBody: handlebar },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Salve** e execute o script de novo:

    ```bash
    node script.js
    ```

1. A saída indicará um código de status de êxito para cada operação.

## Como criar um lote transacional errante

Agora, vamos criar um lote transacional que erra propositalmente. Esse lote tentará inserir dois itens que têm chaves de partição lógicas diferentes. Criaremos uma luz estroboscópica cintilante na categoria “acessórios usados” e um novo capacete na categoria “acessórios novos”. Por definição, essa deve ser uma solicitação incorreta e retornar um erro ao executar essa transação.

1. Retorne à guia do editor do arquivo de código **script.js**.

1. Exclua as seguintes linhas de código:

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };

    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: saddle },
        { operationType: BulkOperationType.Create, resourceBody: handlebar },
    ];
    ```

1. Modifique o script para criar uma nova **luz estroboscópica cintilante** e um **novo capacete** com diferentes valores de chave de partição.

    ```javascript
    const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    ```

1. Crie uma variável de cadeia de caracteres chamada **partition_key** com o valor **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Crie um novo lote com os itens **luz** e **capacete**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: light },
        { operationType: BulkOperationType.Create, resourceBody: helmet },
    ];
    ```

1. Quando terminar, seu arquivo de código deverá incluir:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: light },
            { operationType: BulkOperationType.Create, resourceBody: helmet },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Salve** e execute o script de novo:

    ```bash
    node script.js
    ```

1. Observe a saída do terminal. O código de status nos itens será **424** para **Dependência com falha** ou **400** para **Solicitação incorreta**. Isso ocorreu porque todos os itens dentro da transação não compartilhavam o mesmo valor de chave de partição que o lote transacional.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
