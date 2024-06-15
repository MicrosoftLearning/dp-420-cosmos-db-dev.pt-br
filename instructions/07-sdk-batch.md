---
lab:
  title: Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL

As classes [TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] e [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] juntas são a chave para compor e decompor operações em uma só etapa lógica. Usando essas classes, você pode escrever seu código para executar várias operações e determinar se elas foram concluídas com êxito no lado do servidor.

Neste laboratório, você usará o SDK para executar duas operações de item duplo em que você tenta criar dois itens como uma só unidade lógica.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se ainda não estiver familiarizado com a interface do Visual Studio Code, revise a [documentação da Introdução][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Após o repositório ter sido clonado, abra a pasta local que você selecionou no **Visual Studio Code**.

## Criar uma conta do Azure Cosmos DB for NoSQL e configurar o projeto do SDK

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Cosmos DB* e, em seguida, crie um novo recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Taxa de transferência provisionada* |
    | **Aplicar Desconto na Camada Gratuita** | *Não Aplicar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave**posteriormente neste exercício.

1. Volte para o **Visual Studio Code**.

1. No painel **Explorer**, navegue até a pasta **07-sdk-batch**.

1. Abra o arquivo de código **script.cs** dentro da pasta **07-sdk-batch**.

    > &#128221; A biblioteca **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** já foi importada previamente do NuGet.

1. Localize a variável **string** chamada **endpoint**. Defina seu valor como o **ponto de extremidade** da conta do Azure Cosmos DB criado anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/**, a instrução C# será: **ponto de extremidade da cadeia de caracteres = "https&shy;://dp420.documents.azure.com:443/";**.

1. Localize a variável de **cadeia de caracteres** chamada **chave**. Defina seu valor como a **chave** da conta do Azure Cosmos DB criada anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se sua chave for: **fDR2ci9QgkdkvERTQ==**, então a instrução C# será: **chave de cadeia de caracteres = "fDR2ci9QgkdkvERTQ==";**.

1. **Salve** o arquivo de código **script.cs**.

1. Abra o menu de contexto da pasta **07-sdk-batch** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **07-sdk-batch**.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Crie o projeto usando o comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Feche o terminal integrado.

## Como criar um lote transacional

Primeiro, vamos criar um lote transacional simples que cria dois produtos fictícios. Esse lote vai inserir um assento usado e um guidão enferrujado no contêiner com o mesmo identificador de categoria “acessórios usados”. Ambos os itens têm a mesma chave de partição lógica, garantindo que teremos uma operação em lote bem-sucedida.

1. Volte à guia do editor do arquivo de código **script.cs**.

1. Crie uma variável **Product** chamada **saddle** com o identificador exclusivo **0120**, o nome **Worn Saddle** e o identificador de categoria **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Crie uma variável **Product** chamada **handlebar** com o identificador exclusivo **012A**, o nome **Rusty Handlebar** e o identificador de categoria **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Crie uma variável do tipo **PartitionKey** chamada **partitionKey**, transmitindo **9603ca6c-9e28-4a02-9194-51cdb7fea816** como um parâmetro de construtor:

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Invoque o método [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] da variável **container**, transmitindo a variável **partitionkey** como um parâmetro de método e usando a sintaxe fluente para invocar os métodos genéricos [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] transmitindo as variáveis **saddle** e **handlebar** como itens para criar operações individuais e armazenar o resultado em uma variável chamada **batch** do tipo **TransactionalBatch**:

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. Em uma instrução using, invoque de maneira assíncrona o método **ExecuteAsync** da variável **batch** e armazene o resultado em uma variável do tipo **TransactionalBatchResponse** chamada **response**:

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Invoque o método estático **Console.WriteLine** para gerar o valor da propriedade **StatusCode** da variável **response**:

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Depois de terminar, o arquivo de código deverá incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **07-sdk-batch** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O código de status deve ser **OK**.

1. Feche o terminal integrado.

## Como criar um lote transacional errante

Agora, vamos criar um lote transacional que erra propositalmente. Esse lote tentará inserir dois itens que têm chaves de partição lógicas diferentes. Criaremos uma luz estroboscópica cintilante na categoria “acessórios usados” e um novo capacete na categoria “acessórios novos”. Por definição, essa deve ser uma solicitação incorreta e retornar um erro ao executar essa transação.

1. Volte à guia do editor do arquivo de código **script.cs**.

1. Exclua as seguintes linhas de código:

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Crie uma variável **Product** chamada **light** com o identificador exclusivo **012B**, o nome **Flickering Strobe Light** e o identificador de categoria **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Crie uma variável **Product** chamada **helmet** com o identificador exclusivo **012C**, o nome **New Helmet** e o identificador de categoria **0feee2e4-687a-4d69-b64e-be36afc33e74**:

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. Crie uma variável do tipo **PartitionKey** chamada **partitionKey**, transmitindo **9603ca6c-9e28-4a02-9194-51cdb7fea816** como um parâmetro de construtor:

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Invoque o método **CreateTransactionalBatch** da variável **container**, transmitindo a variável **partitionkey** como um parâmetro de método e usando a sintaxe fluente para invocar os métodos genéricos **CreateItem<>**, transmitindo as variáveis **light** e **helmet** como itens para criar operações individuais e armazenar o resultado em uma variável chamada **batch** do tipo **TransactionalBatch**:

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. Em uma instrução using, invoque de maneira assíncrona o método **ExecuteAsync** da variável **batch** e armazene o resultado em uma variável do tipo **TransactionalBatchResponse** chamada **response**:

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Invoque o método estático **Console.WriteLine** para gerar o valor da propriedade **StatusCode** da variável **response**:

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Depois de terminar, o arquivo de código deverá incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **07-sdk-batch** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O código de status deve ser uma **Solicitação Incorreta** ou um **Conflito**. Isso ocorreu porque todos os itens dentro da transação não compartilhavam o mesmo valor de chave de partição que o lote transacional.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
