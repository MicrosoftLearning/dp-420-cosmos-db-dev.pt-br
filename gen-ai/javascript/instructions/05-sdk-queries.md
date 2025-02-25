---
title: 05 – Executar uma consulta com o SDK do Azure Cosmos DB for NoSQL
lab:
  title: 05 – Executar uma consulta com o SDK do Azure Cosmos DB for NoSQL
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: JavaScript SDK labs
---

# Executar uma consulta com o SDK do Azure Cosmos DB for NoSQL

A última versão do SDK do JavaScript para o Azure Cosmos DB for NoSQL simplifica a consulta de um contêiner e a iteração em conjuntos de resultados usando os recursos modernos do JavaScript.

A biblioteca `@azure/cosmos` tem funcionalidade interna para tornar a consulta do Azure Cosmos DB eficiente e simples.

Neste laboratório, você usará um iterador para processar um conjunto de resultados grande retornado do Azure Cosmos DB for NoSQL. Você usará o SDK do JavaScript para consultar e iterar sobre os resultados.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md) para fazer isso.

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para o laboratório **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#create-azure-cosmos-db-database-and-container-with-sample-data). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Criar um banco de dados e um contêiner do Azure Cosmos DB com dados de amostra

Se você já criou um banco de dados do Azure Cosmos DB chamado **cosmicworks-full** e um contêiner dentro dele chamado **products**, que é pré-carregado com dados de amostra, você poderá usá-lo para este laboratório e pular para a [próxima seção](#import-the-azurecosmos-library). Caso contrário, siga as etapas abaixo para criar um novo banco de dados e contêiner de amostra.

<details markdown=1>
<summary markdown="span"><strong>Clique para expandir/recolher as etapas para criar banco de dados e contêiner com dados de amostra</strong></summary>

1. Dentro do recurso de conta do **Azure Cosmos DB** recém-criado, navegue até o painel do **Data Explorer**.

1. Na home page do **Data Explorer**, selecione **Lançar início rápido**.

1. No formulário **Novo contêiner**, insira os seguintes valores:

    - **Database id**: `cosmicworks-full`
    - **Container id**: `products`
    - **Chave de partição**: `/categoryId`
    - **Repositório analítico**: `Off`

1. Selecione **OK** para criar o novo contêiner. Esse processo levará um ou dois minutos enquanto cria os recursos e pré-carrega o contêiner com dados de produto de amostra.

1. Mantenha a guia do navegador aberta, pois voltaremos a ela mais tarde.

1. Volte ao **Visual Studio Code**.

</details>

## Importar a biblioteca @azure/cosmos

A biblioteca **@azure/cosmos** está disponível no **npm** para facilitar a instalação em seus projetos JavaScript.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/05-sdk-queries**.

1. Abra o menu de contexto da pasta **javascript/05-sdk-queries** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido como a pasta **javascript/05-sdk-queries**.

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

## Iterar sobre os resultados de uma consulta SQL usando o SDK

Usando as credenciais da conta recém-criada, você se conectará às classes do SDK e ao banco de dados e ao contêiner provisionados em uma etapa anterior e iterará sobre os resultados de uma consulta SQL usando o SDK.

Agora você usará um iterador para criar um loop foreach simples de entender sobre os resultados paginados no Azure Cosmos DB. Nos bastidores, o SDK gerenciará o iterador de feed e garantirá que as solicitações subsequentes sejam invocadas corretamente.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/05-sdk-queries**.

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

1. Crie um novo método chamado **queryContainer** e código para executar esse método ao executar o script. Você adicionará o código para consultar o contêiner dentro deste método:

    ```javascript
    async function queryContainer() {
        // Query the container
    }

    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. Dentro do método **queryContainer**, adicione o seguinte código para se conectar ao banco de dados e ao contêiner que você criou anteriormente:

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. Crie uma variável de cadeia de caracteres de consulta chamada `sql` com um valor `SELECT * FROM products p`.

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. Invoque o método [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) com a variável `sql` como um parâmetro para o construtor. O parâmetro `enableCrossPartitionQuery`, quando definido como `true`, permite enviar mais de uma solicitação para executar a consulta no serviço do Azure Cosmos DB. Mais de uma solicitação será necessária se a consulta não tiver como escopo um único valor de chave de partição.

    ```javascript
    const iterator = container.items.query(
        query,
        { enableCrossPartitionQuery: true }
    );
    ```

1. Itere sobre os resultados paginados e imprima o `id`, `name` e `price` de cada item:

    ```javascript
    while (iterator.hasMoreResults()) {
        const { resources } = await iterator.fetchNext();
        for (const item of resources) {
            console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
        }
    }
    ```

1. O arquivo **script.js** ficará assim:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function queryContainer() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products p";
    
        const iterator = container.items.query(
            query,
            { enableCrossPartitionQuery: true }
        );
        
        while (iterator.hasMoreResults()) {
            const { resources } = await iterator.fetchNext();
            for (const item of resources) {
                console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
            }
        }
    }
    
    queryContainer().catch((error) => {
        console.error(error);
    });
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

1. O script agora produzirá todos os produtos no contêiner.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
