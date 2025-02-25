---
title: 06 – Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL
lab:
  title: 06 – Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL
  module: Author complex queries with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 9
parent: JavaScript SDK labs
---

# Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL

As consultas do Azure Cosmos DB normalmente terão várias páginas de resultados. A paginação é feita automaticamente no lado do servidor quando o Azure Cosmos DB não pode retornar todos os resultados da consulta em uma única execução. Em muitos aplicativos, você desejará escrever código usando o SDK para processar os resultados da consulta em lotes de maneira eficiente.

Neste laboratório, você criará um iterador de feed que pode ser usado em um loop para iterar sobre todo o conjunto de resultados.

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

1. No **Visual Studio Code**, no painel **Explorer**, navegue até a pasta **javascript/06-sdk-pagination**.

1. Abra o menu de contexto da pasta**javascript/06-sdk-pagination** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **javascript/06-sdk-pagination**.

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

Ao processar resultados de consultas, é preciso garantir que o código avance por todas as páginas de resultados e verifique se há mais páginas restantes antes de fazer solicitações subsequentes.

1. No **Visual Studio Code**, no painel **Explorer**, navegue até a pasta **javascript/06-sdk-pagination**.

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

1. Crie um novo método chamado **paginateResults** e código para executar esse método ao executar o script. Você adicionará o código para consultar o contêiner dentro deste método:

    ```javascript
    async function paginateResults() {
        // Query the container
    }

    paginateResults().catch((error) => {
        console.error(error);
    });
    ```

1. Dentro do método **paginateResults**, adicione o seguinte código para se conectar ao banco de dados e ao contêiner que você criou anteriormente:

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. Crie uma variável de cadeia de caracteres de consulta chamada `sql` com um valor `SELECT * FROM products p`.

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. Crie uma nova variável chamada `options` e defina-a como um objeto com a propriedade `enableCrossPartitionQuery` definida como `true`. Essa propriedade permite enviar mais de uma solicitação para executar a consulta no serviço do Azure Cosmos DB. Mais de uma solicitação será necessária se a consulta não tiver como escopo um único valor de chave de partição. Defina a propriedade `maxItemCount` como `50` para limitar o número de itens por página:

    ```javascript
    const options = {
        enableCrossPartitionQuery: true,
        maxItemCount: 50 // Set the maximum number of items per page
    };
    ```

1. Invoque o método [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) com as variáveis `sql` e `options` como parâmetros para o construtor. Este método retorna um iterador que pode ser usado para buscar a próxima página de resultados:

    ```javascript
    const iterator = container.items.query(query, options);
    ```

1. Itere sobre os resultados paginados e imprima o `id`, `name` e `price` de cada item. O método `iterator.getAsyncIterator` retorna um iterador assíncrono que pode ser usado para buscar o loop `for await...of` para recuperar cada página de resultados. Esse loop manipula automaticamente a iteração assíncrona nas páginas.

    ```javascript
    for await (const page of iterator.getAsyncIterator()) {
        page.resources.forEach(product => {
            console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
        });
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

    async function paginateResults() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products WHERE products.price > 500";

        const options = {
            enableCrossPartitionQuery: true,
            maxItemCount: 50 // Set the maximum number of items per page
        };
        
        const iterator = container.items.query(query, options);

        for await (const page of iterator.getAsyncIterator()) {
            page.resources.forEach(product => {
                console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
            });
        }
    }
    
    paginateResults().catch((error) => {
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

1. O script agora gerará páginas de 50 itens por vez.

    > &#128161; A consulta corresponderá a centenas de itens no contêiner de produtos.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
