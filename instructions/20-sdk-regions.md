---
lab:
  title: Conectar-se a diferentes regiões com o SDK do Azure Cosmos DB for NoSQL
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Conectar-se a diferentes regiões com o SDK do Azure Cosmos DB for NoSQL

Ao habilitar a redundância geográfica para uma conta do Azure Cosmos DB for NoSQL, você pode usar o SDK para ler dados de regiões em qualquer ordem que configurar. Essa técnica é benéfica quando você distribui suas solicitações de leitura por todas as suas regiões de leitura disponíveis.

Nesse laboratório, você irá configurar a classe CosmosClient para se conectar a regiões de leitura em uma ordem de fallback que você irá configurar manualmente.

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
    | **Aplicar Desconto na Camada Gratuita** | *Não Aplicar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Acesse o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Replicar dados globalmente**.

1. No painel **Replicar dados globalmente**, adicione duas regiões de leitura extras à conta e, em seguida, clique em **Salvar** suas alterações.

1. Aguarde a conclusão da tarefa de replicação antes de continuar com essa tarefa.

    > &#128221; Essa operação pode levar cerca de 5 a 10 minutos.

1. Registre os nomes da região de **Gravação** (primária) e das duas regiões de **Leitura**. Você usará esses nomes de regiões posteriormente nesse exercício.

    > &#128221; Por exemplo, se sua região primária for **Norte da Europa** e suas duas regiões secundárias de leitura forem **Leste dos EUA 2** e **Norte da África do Sul**, você irá gravar esses três nomes no estado em que se encontram.

1. Na folha de recursos, navegue até o painel do **Data Explorer**.

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

Usando as credenciais da conta recém-criada, você irá se conectar com as classes do SDK e acessar a instância de contêiner e de banco de dados de uma região diferente.

1. No painel do **Explorer**, navegue até a pasta **20-sdk-regions**.

1. Abra o menu de contexto da pasta **20-sdk-regions** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido como a pasta **20-sdk-regions**.

1. Crie o projeto usando o comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; É possível que você veja um aviso do compilador informando que as variáveis **ponto de extremidade** e **chave** estão atualmente sem uso. Você pode ignorar esse aviso com segurança, pois usará essas variáveis nessa tarefa.

1. Feche o terminal integrado.

1. Abra o arquivo de código **script.cs** dentro da pasta **20-sdk-regions**.

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

1. **Salve** o arquivo do código **script.cs**.

## Configurar o SDK do .NET com uma lista de regiões preferenciais

A classe **CosmosClientOptions** inclui uma propriedade para configurar a lista de regiões às quais você gostaria de se conectar com o SDK. A lista é ordenada por prioridade de failover e tenta se conectar a cada região na ordem que você configurar.

1. Crie uma nova variável do tipo genérico **List\<string\>** que contenha uma lista das regiões que você configurou com sua conta, começando com a terceira região e terminando com a primeira região (primária). Por exemplo, se você criou sua conta do Azure Cosmos DB for NoSQL na região **Oeste dos EUA**, em seguida, adicionou **Norte da África do Sul** e, para terminar, adicionou **Leste da Ásia**, então sua variável list conterá:

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; Alternativamente, você pode usar a classe estática [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions], que inclui propriedades integradas de cadeia de caracteres para várias regiões do Azure.

1. Crie uma nova instância da classe **CosmosClientOptions** chamada **options** com a propriedade [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] definida como a variável **regions**:

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    ```

1. Crie uma nova instância da classe **CosmosClient** chamada **client** repassando as variáveis **endpoint**, **key** e **options** como parâmetros do construtor:

    ```
    using CosmosClient client = new (endpoint, key, options); 
    ```

1. Use o método **GetContainer** da variável **client** para recuperar o contêiner existente usando o nome do banco de dados (*cosmicworks*) e o nome do contêiner (*produtos*):

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Use o método [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] da variável **container** para recuperar um item específico do servidor e armazenar o resultado em uma variável chamada **response** do tipo [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] anulável:

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. Invoque o método estático **Console.WriteLine** para imprimir o identificador do item atual e os dados de diagnóstico JSON:

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. Após você terminar, seu arquivo de código agora deve incluir:
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **Salve** o arquivo do código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto para a pasta **20-sdk-regions** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

1. Crie e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O nome do contêiner e os dados de diagnóstico JSON devem ser impressos na saída do console.

1. Reveja os dados de diagnóstico JSON. Procure uma propriedade chamada **HttpResponseStats** e uma propriedade filho chamada **RequestUri**. O valor dessa propriedade deve ser um URI que inclui o nome e a região que você configurou anteriormente nesse laboratório.

    > &#128221; Por exemplo, se o nome da sua conta for: **dp420** e a primeira região que você configurou for **Leste da Ásia**, o valor da propriedade JSON será: **dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products**.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
