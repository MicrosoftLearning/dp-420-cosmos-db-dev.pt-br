---
lab:
  title: 01 – Conectar-se ao Azure Cosmos DB for NoSQL com o SDK
  module: Use the Azure Cosmos DB for NoSQL SDK
---

# Conectar-se ao Azure Cosmos DB for NoSQL com o SDK

O SDK do Azure para JavaScript (Node.js e Browser) é um conjunto de bibliotecas de cliente que fornece uma interface de desenvolvedor consistente para interagir com muitos serviços do Azure. As bibliotecas de cliente são pacotes usados para consumir esses recursos e interagir com eles.

Neste laboratório, você se conectará a uma conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para JavaScript.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md).

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#import-the-azurecosmos-library). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo a função de **Colaborador de Dados Internos do Cosmos DB**.

## Importar a biblioteca @azure/cosmos

A biblioteca **@azure/cosmos** está disponível no **npm** para facilitar a instalação em seus projetos JavaScript.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/01-sdk-connect**.

1. Abra o menu de contexto da pasta **javascript/01-sdk-connect** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido para a pasta **javascript/01-sdk-connect**

1. Inicializar um novo projeto Node.js:

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

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **javascript/01-sdk-connect**.

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

1. Adicione uma função `async` chamada **main** para ler e imprimir as propriedades da conta:

    ```javascript
    async function main() {
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }
    ```

1. Invoque a função **main**:

    ```javascript
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
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }

    main().catch((error) => console.error(error));
    ```

1. **Salve** o arquivo **script.js**.

## Testar o script

Agora que o código JavaScript para conexão com a conta do Azure Cosmos DB for NoSQL está concluído, você pode testar o script. Esse script imprimirá o nível de consistência padrão e o nome da primeira região gravável. Ao criar a conta, você especificou um local e deve esperar ver esse mesmo valor de local impresso como resultado desse script.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **javascript/01-sdk-connect** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Antes de executar o script, você deve fazer logon no Azure usando o comando `az login`. Na janela do terminal, execute:

    ```bash
    az login
    ```

1. Execute o script usando o comando `node`:

    ```bash
    node script.js
    ```

1. O script agora exibirá o nível de consistência padrão da conta e a primeira região gravável. Por exemplo, se o nível de consistência padrão da conta for **Sessão** e a primeira região gravável for **Leste dos EUA**, o script gerará:

    ```text
    Consistency Policy: Session
    Primary Region: East US
    ```

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
