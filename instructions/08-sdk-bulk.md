---
lab:
  title: Migrar vários documentos em massa com o SDK do Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Migrar vários documentos em massa com o SDK do Azure Cosmos DB for NoSQL

A maneira mais fácil de aprender a executar uma operação em massa é tentar enviar muitos documentos para uma conta do Azure Cosmos DB for NoSQL na nuvem. Usando os recursos em massa do SDK, isso pode ser feito com uma pequena ajuda do namespace [System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks].

Nesse laboratório, você usará a biblioteca [Bogus][nuget.org/packages/bogus/33.1.1] do NuGet para gerar dados fictícios e colocá-los em uma conta do Azure Cosmos DB.

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
    | **Tipo de carga de trabalho** | **Aprendizado** |
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

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave** posteriormente nesse exercício.

1. Ainda dentro do recurso de conta do **Azure Cosmos DB** recém-criado, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, selecione **Novo Contêiner** e, a seguir, crie um novo contêiner com as configurações a seguir, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **ID do banco de dados** | *Criar novo* &vert; *`cosmicworks`* |
    | **Compartilhar a taxa de transferência entre contêineres** | *Não selecione* |
    | **ID do contêiner** | *`products`* |
    | **Chave de partição** | *`/categoryId`* |
    | **Taxa de transferência do contêiner** | *Dimensionamento automático* &vert; *`4000`* |

1. Volte para o **Visual Studio Code**.

1. No painel do **Explorer**, navegue até a pasta **08-sdk-bulk**.

1. Abra o arquivo de código **script.cs** dentro da pasta **08-sdk-bulk**.

    > &#128221; A biblioteca **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** já foi importada do NuGet previamente.

1. Localize a variável **cadeia de caracteres** denominada **ponto de extremidade**. Defina seu valor como o **ponto de extremidade** da conta do Azure Cosmos DB criado anteriormente.
  
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

1. Abra o menu de contexto da pasta **08-sdk-bulk** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido como a pasta **08-sdk-bulk**.

1. Adicione o pacote Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Crie o projeto usando o comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Feche o terminal integrado.

## Como inserir 25 mil documentos em massa

Vamos "agir com entusiasmo" e tentar inserir um monte de documentos para ver como isso funciona. Em nossos testes internos, isso pode levar aproximadamente de 1 a 2 minutos se a máquina virtual do laboratório e a conta do Azure Cosmos DB for NoSQL estiverem relativamente próximas umas das outras geograficamente falando.

1. Retorne à guia do editor do arquivo de código **script.cs**.

1. Crie uma nova instância da classe de [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] chamada **options** com a propriedade **AllowBulkExecution** definida com um valor **true**:

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. Crie uma nova instância da classe **CosmosClient** chamada **client** repassando as variáveis **endpoint**, **key** e **options** como parâmetros do construtor:

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. Use o método [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] da variável **client** para recuperar o contêiner existente usando o nome do banco de dados (*cosmicworks*) e o nome do contêiner (*produtos*):

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Use essa amostra especial de código para gerar **25.000** produtos fictícios usando a classe **Faker** da biblioteca Bogus importada do NuGet.

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; A biblioteca [Bogus][nuget.org/packages/bogus/33.1.1] é uma biblioteca de código aberto usada para criar dados fictícios e testar aplicativos de interface do usuário e é excelente para aprender a desenvolver aplicativos de importação/exportação em massa.

1. Crie uma nova lista genérica **List<>** do tipo **Tarefa** denominada **concurrentTasks**:

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. Crie um loop foreach que irá iterar sobre a lista de produtos que foram gerados anteriormente nesse aplicativo:

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. No loop foreach, crie uma **Tarefa** para inserir um produto no Azure Cosmos DB for NoSQL de forma assíncrona certificando-se de especificar explicitamente a chave de partição e de adicionar a tarefa à lista de tarefas chamada **concurrentTasks**:

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. Após o loop foreach, aguarde assincronamente o resultado de **Task.WhenAll** na variável **concurrentTasks**:

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. Use o método estático integrado **Console.WriteLine** para imprimir uma mensagem estática de **Tarefas em massa concluídas** para o console:

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. Após você terminar, seu arquivo de código agora deve incluir:
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto para a pasta **08-sdk-bulk** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

1. Crie e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. O aplicativo deve ser executado silenciosamente e deve levar aproximadamente um a dois minutos para ser executado antes de ser concluído silenciosamente.

1. Feche o terminal integrado.

## Observe os resultados

Agora que você enviou 25.000 itens para o Azure Cosmos DB, vamos examinar o Data Explorer.

1. Retorne ao navegador da web e navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e, a seguir, observe o nó do contêiner **produtos** dentro da árvore de navegação da **API do NOSQL**.

1. Expanda o nó **produtos** e, a seguir, selecione o nó **Itens**. Observe a lista de itens dentro do seu contêiner.

1. Selecione o nó de contêiner **produtos** dentro da árvore de navegação da **API do NOSQL** e selecione **Nova Consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará o número de todos os documentos criados usando a operação em massa:

    ```
    SELECT COUNT(1) FROM items
    ```

1. Selecione **Executar Consulta**.

1. Observe o número dos itens no seu contêiner.

1. Feche a guia ou a janela do navegador da web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
