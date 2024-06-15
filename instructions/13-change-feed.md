---
lab:
  title: Processar eventos do feed de alterações usando o SDK do Azure Cosmos DB for NoSQL
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Processar eventos do feed de alterações usando o SDK do Azure Cosmos DB for NoSQL

O feed de alterações do Azure Cosmos DB for NoSQL é a chave para a criação de aplicativos complementares impulsionados por eventos da plataforma. O SDK do .NET para o Azure Cosmos DB for NoSQL é fornecido com um conjunto de classes para criar seus aplicativos que se integram ao feed de alterações e escutam notificações sobre operações em seus contêineres.

Neste laboratório, você usará a funcionalidade do processador de feed de alterações no SDK do .NET para criar um aplicativo que seja notificado quando uma operação de criação ou atualização for executada em um item no contêiner especificado.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não se familiarizou com a interface do Visual Studio Code, confira o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Depois que o repositório tiver sido clonado, abra a pasta local selecionada no **Visual Studio Code**.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você irá selecionar a qual API você quer que a conta dê suporte (por exemplo, a **API do Mongo** ou a **API do NoSQL**). Quando o provisionamento da conta do Azure Cosmos DB for NoSQL estiver concluído, você poderá recuperar o ponto de extremidade e a chave e usá-los para se conectar à conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para .NET ou qualquer outro SDK de sua escolha.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Cosmos DB* e, em seguida, crie um novo recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Sem servidor* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave** posteriormente neste exercício.

1. Selecione **Data Explorer** no menu de recursos.

1. No painel do **Data Explorer**, expanda **Novo contêiner** e, a seguir, selecione **Novo Banco de Dados**.

1. No pop-up de **Novo Banco de Dados**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *``cosmicworks``* |

1. De volta ao painel do **Data Explorer**, observe o nó de banco de dados do **cosmicworks** dentro da hierarquia.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *``products``* |
    | **Chave de partição** | *``/categoryId``* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados do **cosmicworks** e observe o nó do contêiner de **produtos** dentro da hierarquia.

1. No painel do **Data Explorer**, selecione **Novo Contêiner** novamente.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *``productslease``* |
    | **Chave de partição** | *``/partitionKey``* |

1. De volta ao painel do **Data Explorer**, expanda o nó de banco de dados do ** cosmicworks** e observe o nó de contêiner de **productslease** dentro da hierarquia.

1. Volte para o **Visual Studio Code**.

## Implementar o processador de feed de alterações no SDK do .NET

A classe **Microsoft.Azure.Cosmos.Container** é fornecida com uma série de métodos para criar o processador de feed de alterações com fluidez. Para começar, você precisa de uma referência ao contêiner monitorado, ao contêiner de concessão e a um delegado em C\# (para lidar com cada lote de alterações).

1. No painel do **Explorer**, navegue até a pasta **13-change-feed**.

1. Abra o arquivo de código **product.cs**.

1. Observe a classe **Product** e suas propriedades correspondentes. Especificamente, este laboratório usará as propriedades de **ID** e **nome**.

1. De volta ao painel do **Explorer** do **Visual Studio Code**, abra o arquivo de código **script.cs**.

1. Atualize a variável existente chamada **ponto de extremidade** com o seu valor definido como o **ponto de extremidade** da conta do Azure Cosmos DB que você criou anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/**, a instrução C# será: **ponto de extremidade da cadeia de caracteres = "https&shy;://dp420.documents.azure.com:443/";**.

1. Atualize a variável existente nomeada **chave** com o seu valor definido como a **chave** da conta do Azure Cosmos DB criada anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se sua chave for: **fDR2ci9QgkdkvERTQ==**, então a instrução C# será: **chave de cadeia de caracteres = "fDR2ci9QgkdkvERTQ==";**.

1. Use o método **GetContainer** da variável do **cliente** para recuperar o contêiner existente usando o nome do banco de dados (*cosmicworks*) e o nome do contêiner (*produtos*) e armazene o resultado em uma variável chamada **sourceContainer** do tipo **Contêiner**:

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. Use o método **GetContainer** da variável do **cliente** para recuperar o contêiner existente usando o nome do banco de dados (*cosmicworks*) e o nome do contêiner (*productslease*) e armazene o resultado em uma variável chamada **leaseContainer** do tipo **Contêiner**:

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. Crie uma nova variável delegada chamada **handleChanges** do tipo [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] usando uma função anônima assíncrona vazia que tem dois parâmetros de entrada:

    1. Um parâmetro chamado **alterações** do tipo **IReadOnlyCollection\<Product\>**.
    
    1. Um parâmetro chamado **cancellationToken** do tipo **CancellationToken**.

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. Dentro da função anônima, use o método estático **Console.WriteLine** interno para imprimir a cadeia de caracteres bruta **lote de alterações START\tHandling...**:

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. Ainda dentro da função anônima, crie um loop foreach que itera sobre a variável **alterações** usando a variável **produto** para representar uma instância do tipo **Product**:

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. No loop foreach da função anônima, use o método estático assíncrono **Console.WriteLineAsync** interno para imprimir as propriedades de **ID** e **nome** da variável **produto**:

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. Fora do loop foreach e da função anônima, crie uma nova variável chamada **construtor** que armazena o resultado da invocação de [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] na variável **sourceContainer** usando os seguintes parâmetros:

    | **Parâmetro** | **Valor** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. Invoque o método [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] com um parâmetro de **consoleApp**, o método [WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] com um parâmetro de **leaseContainer** e o método [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] fluentemente na variável **construtor** armazenando o resultado em uma variável chamada **processador** do tipo [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]:

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. Invoque de forma assíncrona a [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync] da variável do **processador**:

    ```
    await processor.StartAsync();
    ```

1. Use métodos estáticos **Console.WriteLine** e **Console.ReadKey** internos para imprimir a saída no console e fazer com que o aplicativo aguarde o pressionamento de uma tecla:

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. Invoque de forma assíncrona a [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] da variável do **processador**:

    ```
    await processor.StopAsync();
    ```

1. Quando terminar, seu arquivo de código deverá incluir:
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **Salve** o arquivo **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **13-change-feed** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Deixe o **Visual Studio Code** e o terminal abertos.

    > &#128221; Você usará outra ferramenta para gerar itens no seu contêiner do Azure Cosmos DB for NoSQL. Depois de gerar os itens, você retornará a esse terminal para observar a saída. Não feche o terminal prematuramente.

## Propagar sua conta do Azure Cosmos DB for NoSQL com amostras de dados

Você usará um utilitário de linha de comando que cria um banco de dados do **cosmicworks** e um contêiner de **produtos**. Em seguida, a ferramenta criará um conjunto de itens que você irá observar usando o processador de feed de alterações em execução na janela do seu terminal.

1. No **Visual Studio Code**, abra o menu ** Terminal** e, a seguir, selecione **Dividir Terminal** para abrir um novo terminal lado a lado com a instância existente.

1. Instale a ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] para uso global em seu computador.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Esse comando poderá levar alguns minutos para ser concluído. Esse comando irá gerar a mensagem de aviso (*A ferramenta "cosmicworks" já está instalada) se você já tiver instalado a versão mais recente dessa ferramenta anteriormente.

1. Execute o cosmicworks para propagar sua conta do Azure Cosmos DB com as seguintes opções de linha de comando:

    | **Opção** | **Valor** |
    | ---: | :--- |
    | **--ponto de extremidade** | *O valor do ponto de extremidade copiado anteriormente nesse laboratório* |
    | **--chave** | *O valor da chave copiado anteriormente nesse laboratório* |
    | **--conjuntos de dados** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por exemplo, se o seu ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/** e sua chave for: **fDR2ci9QgkdkvERTQ==**, o comando será: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Aguarde até que o comando **cosmicworks** termine de preencher a conta com um banco de dados, um contêiner e itens.

1. Observe a saída do terminal do aplicativo .NET. O terminal gera uma mensagem de **Operação Detectada** para cada alteração que foi enviada para ele usando o feed de alterações.

1. Feche ambos os terminais integrados.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
