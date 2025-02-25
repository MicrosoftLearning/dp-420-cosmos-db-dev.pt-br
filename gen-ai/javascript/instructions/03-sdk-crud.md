---
title: 03 – Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL
lab:
  title: 03 – Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: JavaScript SDK labs
---

# Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL

A biblioteca `@azure/cosmos` inclui métodos para criar, recuperar, atualizar e excluir (CRUD) itens dentro de um contêiner do Azure Cosmos DB for NoSQL. Juntos, esses métodos executam algumas das operações “CRUD” mais comuns em vários itens dentro de contêineres da API NoSQL.

Neste laboratório, você usará o SDK do JavaScript para executar operações CRUD diárias em um item dentro de um contêiner do Azure Cosmos DB for NoSQL.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md) para fazer isso.

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#import-the-azurecosmos-library). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Importar a biblioteca @azure/cosmos

A biblioteca **@azure/cosmos** está disponível no **npm** para facilitar a instalação em seus projetos JavaScript.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/03-sdk-crud**.

1. Abra o menu de contexto da pasta **javascript/03-sdk-crud** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **javascript/03-sdk-crud**.

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

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/03-sdk-crud**.

1. Abra o arquivo JavaScript vazio chamado **script.js**.

1. Adicione as seguintes instruções `require` para importar as bibliotecas **@azure/cosmos** e **@azure/identity**:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    const { CosmosClient } = require("@azure/cosmos");
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

## Executar operações de ponto de leitura e criação em itens com o SDK

Agora você usará o conjunto de métodos da classe **Contêiner** para executar operações comuns em itens em um contêiner da API NoSQL.

1. Volte para o **Visual Studio Code**. Se ainda não estiver aberto, abra o arquivo de código **script.js** na pasta **javascript/03-sdk-crud**.

1. Crie um novo item de produto e atribua-o a uma variável chamada **saddle** com as seguintes propriedades. Adicione o seguinte código à função `main`:

    | Propriedade | Valor |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **marcas** | *{ tan, new, crisp }* |

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };
    ```

1. Invoque o método [`create`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-create) da classe **itens** do contêiner, passando a variável **saddle** como o parâmetro do método:

    ```javascript
    const { resource: item } = await container
        .items.create(saddle);
    ```

1. Quando terminar, seu arquivo de código deverá incluir:
  
    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const saddle = {
            id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
            categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
            name: "Road Saddle",
            price: 45.99,
            tags: ["tan", "new", "crisp"]
        };
    
        const { resource: item } = await container
                .items.create(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Salve** e execute o script de novo:

    ```bash
    node script.js
    ```

1. Observe o novo item no **Data Explorer**.

1. Volte para o **Visual Studio Code**.

1. Retorne à guia do editor do arquivo de código **script.js**.

1. Exclua as seguintes linhas de código:

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };

    const { resource: item } = await container
            .items.create(saddle);
    ```

1. Crie uma variável de cadeia de caracteres chamada **item_id** com o valor **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

    ```javascript
    const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Crie uma variável de cadeia de caracteres chamada **partition_key** com o valor **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Invoque o método [](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-read)`read` da classe **item** do contêiner, passando as variáveis **itemId** e **partitionKey** como os parâmetros do método:

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();
    ```

    > &#128161; O método `read` permite que você execute uma operação de leitura de ponto em um item no contêiner. O método requer os parâmetros `itemId` e `partitionKey` para identificar o item a ser lido. Em vez de executar uma consulta usando a linguagem de consulta SQL do Cosmos DB para localizar o único item, o método `read` é uma maneira mais eficiente e econômica de recuperar um único item. As leituras de ponto podem ler os dados diretamente e não exigem que o mecanismo de consulta processe a solicitação.

1. Imprima o objeto saddle usando uma cadeia de caracteres de saída formatada:

    ```javascript
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```

1. Quando terminar, seu arquivo de código deverá incluir:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();
    
        console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Salve** e execute o script de novo:

    ```bash
    node script.js
    ```

1. Observe a saída do terminal. Especificamente, observe o texto de saída formatado com a ID, o nome e o preço do item.

## Executar operações de ponto de atualização e exclusão com o SDK

Ao aprender o SDK, não é incomum usar uma conta online ou o emulador do Azure Cosmos DB para atualizar um item e oscilar entre o Data Explorer e o IDE de sua escolha ao executar uma operação e verificar se a alteração foi aplicada. Aqui, você fará exatamente isso ao atualizar e excluir um item usando o SDK.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Escolha o único item dentro do contêiner e observe os valores das propriedades **name** e **price** do item.

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *Road Saddle* |
    | **Price** | *$45.99* |

    > &#128221; Neste ponto, esses valores não devem ter sido alterados desde que você criou o item. Você vai alterar esses valores neste exercício.

1. Volte para o **Visual Studio Code**. Retorne à guia do editor do arquivo de código **script.js**.

1. Exclua a seguinte linha de código:

    ```javascript
    console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    ```

1. Altere a variável **saddle** definindo o valor da propriedade price como **32.55**:

    ```javascript
    // Update the item
    saddle.price = 32.55;
    ```

1. Modifique a variável **saddle** novamente, definindo o valor da propriedade **name** como **Road LL Saddle**:

    ```javascript
    saddle.name = "Road LL Saddle";
    ```

1. Invoque o método [`replace`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-replace) da classe **item** do contêiner, passando a variável **saddle** como um parâmetro de método:

    ```javascript
    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Quando terminar, seu arquivo de código deverá incluir:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();

        // Update the item
        saddle.price = 32.55;
        saddle.name = "Road LL Saddle";
    
        await container.item(saddle.id, partitionKey).replace(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Salve** e execute o script de novo:

    ```bash
    node script.js
    ```

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Escolha o único item dentro do contêiner e observe os valores das propriedades **name** e **price** do item.

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *Road LL Saddle* |
    | **Price** | *$32.55* |

    > &#128221; Neste ponto, esses valores devem ter sido alterados desde que você observou o item.

1. Volte para o **Visual Studio Code**. Retorne à guia do editor do arquivo de código **script.js**.

1. Exclua as seguintes linhas de código:

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();

    // Update the item
    saddle.price = 32.55;
    saddle.name = "Road LL Saddle";

    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Invoque o método [`delete`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-delete) da classe **item** do contêiner, passando as variáveis **itemId** e **partitionKey** como parâmetros de método:

    ```javascript
    // Delete the item
    await container.item(itemId, partitionKey).delete();
    ```

1. Salve e execute o script de novo:

    ```bash
    node script.js
    ```

1. Feche o terminal integrado.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Observe que a lista de itens agora está vazia.

1. Feche a janela ou a guia do navegador da Web.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
