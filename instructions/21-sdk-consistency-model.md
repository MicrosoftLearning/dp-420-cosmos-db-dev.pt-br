---
lab:
  title: Configurar modelos de consistência no portal e no SDK do Azure Cosmos DB for NoSQL
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Configurar modelos de consistência no portal e no SDK do Azure Cosmos DB for NoSQL

O nível de consistência padrão para as novas contas do Azure Cosmos DB for NoSQL é a consistência da sessão. Essa configuração padrão pode ser modificada para todas as solicitações futuras. Em um nível individual de solicitação, você pode ir além e relaxar o nível de consistência para essa solicitação específica.

Nesse laboratório, configuraremos o nível de consistência padrão para uma conta do Azure Cosmos DB for NoSQL e, em seguida, configuraremos um nível de consistência para uma operação individual usando o SDK.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se ainda não estiver familiarizado com a interface do Visual Studio Code, revise a [documentação da Introdução][code.visualstudio.com/docs/getstarted]

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
    | **Tipo de carga de trabalho** | **Aprendizado** |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Taxa de transferência provisionada* |
    | **Distribuição Global** &vert; **Redundância Geográfica** | *Habilitar* |
    | **Aplicar Desconto na Camada Gratuita** | *Não Aplicar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Acesse o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Replicar dados globalmente**.

1. No painel **Replicar dados globalmente**, adicione duas regiões de leitura extras à conta e, em seguida, **Salvar** suas alterações.

    > &#128221; Em algumas etapas, você será solicitado a alterar o nível de consistência para Forte, mas observe que a consistência forte para contas com regiões que abrangem mais de 8.000 quilômetros (5000 milhas) é bloqueada por padrão devido à alta latência de gravação. Escolha regiões mais próximas.  Em um ambiente de produção, para habilitar esse recurso, entre em contato com o suporte.

1. Aguarde a conclusão da tarefa de replicação antes de continuar com essa tarefa.

    > &#128221; Essa operação pode levar cerca de 5 a 10 minutos. Em seguida, navegue até o painel **Consistência padrão**.

1. Na folha de recursos, navegue até o painel **Consistência padrão**.

1. No painel **Consistência padrão**, selecione a opção **Forte** e, em seguida, **Salvar** suas alterações.

1. Aguarde até que a alteração no nível de consistência padrão persista antes de continuar com essa tarefa.

1. Na folha de recursos, navegue até o painel **Data Explorer**.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Criar novo* &vert; *``cosmicworks``* |
    | **Compartilhar a taxa de transferência entre contêineres** | *Não selecione* |
    | **ID do contêiner** | *``products``* |
    | **Chave de partição** | *``/categoryId``* |
    | **Taxa de transferência do contêiner** | *Manual* &vert; *400* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados do **cosmicworks** e observe o nó do contêiner de **produtos** dentro da hierarquia.

1. No painel **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, expanda o nó do contêiner de **produtos** e, a seguir, selecione **Itens**.

1. Ainda no painel do **Data Explorer**, selecione **Novo Item** na barra de comandos. No editor, substitua o item JSON do espaço reservado pelo seguinte conteúdo:

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Selecione **Salvar** na barra de comandos para adicionar o item JSON:

1. Na guia **Itens**, observe o novo item no painel **Itens**.

1. Na folha de recursos, navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave**posteriormente neste exercício.

1. Volte para o **Visual Studio Code**.

## Conectar-se a uma conta do Azure Cosmos DB for NoSQL usando o SDK

Usando as credenciais da conta recém-criada, você se conectará às classes do SDK e criará um novo banco de dados e uma nova instância de contêiner. Em seguida, você usará o Data Explorer para validar a existência das instâncias no portal do Azure.

1. No painel **Explorer**, navegue até a pasta **21-sdk-consistency-model**.

1. Abra o menu de contexto da pasta **21-sdk-consistency-model** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido para a pasta **21-sdk-consistency-model**.

1. Crie o projeto usando o comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; É possível que você veja um aviso do compilador informando que as variáveis **ponto de extremidade** e **chave** estão atualmente sem uso. Você pode ignorar esse aviso com segurança, pois usará essas variáveis nessa tarefa.

1. Feche o terminal integrado.

1. Abra o arquivo de código **product.cs**.

1. Observe o registro **Product** e suas propriedades correspondentes. Especificamente, esse laboratório usará as propriedades **id**, **name** e **categoryId**.

1. De volta ao painel do **Explorer** do **Visual Studio Code**, abra o arquivo de código **script.cs**.

    > &#128221; A biblioteca **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0]** já foi importada do NuGet previamente.

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

## Configurar o nível de consistência para uma operação pontual

A classe **ItemRequestOptions** contém propriedades de configuração por solicitação. Usando essa classe, você relaxará o nível de consistência do padrão atual de forte para consistência eventual.

1. Crie uma variável de cadeia de caracteres chamada **id** com um valor de **7d9273d9-5d91-404c-bb2d-126abb6e4833**:

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. Crie uma variável de cadeia de caracteres chamada **categoryId** com um valor de **78d204a2-7d64-4f4a-ac29-9bfc437ae959**:

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. Crie uma variável do tipo **PartitionKey** chamada **partitionKey**, passando a variável **categoryId** como parâmetro do construtor:

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Invoque de forma assíncrona o método genérico **ReadItemAsync\<\>** da variável **container**, passando as variáveis **id** e **partitionkey** como parâmetros do método, usando **Product** como o tipo genérico e armazenando o resultado em uma variável chamada **response** do tipo **ItemResponse\<Product\>**:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Invoque o método estático **Console.WriteLine** para imprimir a cobrança da solicitação usando uma cadeia de caracteres de saída formatada:

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Depois de terminar, seu arquivo de código deverá incluir:

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **21-sdk-consistency-model** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O encargo da solicitação (em RUs) deve estar impresso no console.

    > &#128221; O encargo da solicitação atual deve ser de **2 RUs**. Isso ocorre devido à forte consistência que exige uma leitura de pelo menos duas réplicas para garantir que ela tenha a gravação mais recente.

1. Feche o terminal integrado.

1. Volte à guia do editor do arquivo de código **script.cs**.

1. Exclua as seguintes linhas de código:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Crie uma nova variável chamada **options** do tipo [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions], definindo a propriedade [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] como o valor de enumeração [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]:

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. Invoque de forma assíncrona o método genérico **ReadItemAsync\<\>** da variável **container**, passando as variáveis **id**, **partitionKey** e **options** como parâmetros do método, usando **Product** como o tipo genérico e armazenando o resultado em uma variável chamada **response** do tipo **ItemResponse\<Product\>**:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. Invoque o método estático **Console.WriteLine** para imprimir a cobrança da solicitação usando uma cadeia de caracteres de saída formatada:

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Depois de terminar, seu arquivo de código deverá incluir:

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **21-sdk-consistency-model** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O encargo da solicitação (em RUs) deve estar impresso no console.

    > &#128221; O encargo da solicitação atual deve ser de **1 RU**. Isso ocorre devido à consistência eventual que exige apenas uma leitura de uma única réplica.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
