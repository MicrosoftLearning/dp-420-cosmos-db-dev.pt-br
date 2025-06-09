---
lab:
  title: Conectar-se a uma conta de gravação de várias regiões com o SDK do Azure Cosmos DB for NoSQL
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Conectar-se a uma conta de gravação de várias regiões com o SDK do Azure Cosmos DB for NoSQL

A classe **CosmosClientBuilder** é uma classe fluente projetada para criar o cliente SDK para se conectar ao contêiner e executar operações. Usando o construtor, você pode configurar uma região de aplicativo preferencial para operações de gravação se sua conta do Azure Cosmos DB for NoSQL já estiver configurada para gravações de várias regiões.

Neste laboratório, você configurará uma conta do Azure Cosmos DB for NoSQL com várias regiões e habilitará gravações de várias regiões. Em seguida, você usará o SDK para executar operações em uma região específica.

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

1. No painel **Replicar dados globalmente**, adicione pelo menos uma região extra à conta.

1. Ainda no painel **Replicar dados globalmente**, habilite **Gravações de várias regiões** e, em seguida, **Salve** suas alterações.

1. Aguarde a conclusão da tarefa de replicação antes de continuar com essa tarefa.

    > &#128221; Essa operação pode levar cerca de 5 a 10 minutos.

1. Observe pelo menos uma das regiões extras que você criou. Você usará esse valor de região posteriormente neste exercício.

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

1. Na folha de recursos, navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave**posteriormente neste exercício.

1. Volte para o **Visual Studio Code**.

## Conectar-se a uma conta do Azure Cosmos DB for NoSQL usando o SDK

Usando as credenciais da conta recém-criada, você se conectará às classes do SDK e criará um novo banco de dados e uma nova instância de contêiner. Em seguida, você usará o Data Explorer para validar a existência das instâncias no portal do Azure.

1. No painel do **Explorer**, navegue até a pasta **22-sdk-multi-region**.

1. Abra o menu de contexto da pasta **22-sdk-multi-region** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido como a pasta **22-sdk-multi-region**.

1. Crie o projeto usando o comando [compilação do dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

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

1. **Salve** o arquivo do código **script.cs**.

## Configurar a região de gravação para o SDK

O método **WithApplicationRegion** fluente é usado para configurar a região preferencial para as operações subsequentes usando a classe de construtor.

1. Crie uma nova instância da classe [ CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder] chamada **construtor** passando no **ponto de extremidade** e variáveis de **chave** como parâmetros de construtor:

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. Crie uma nova variável chamada **região** do tipo **cadeia de caracteres** com o nome da região extra criada anteriormente no laboratório. Por exemplo, se você criou sua conta do Azure Cosmos DB for NoSQL na região **Leste dos EUA** e então adicionou **Sul do Brasil**; em seguida, sua variável de cadeia de caracteres conterá:

    ```
    string region = "Brazil South"; 
    ```

    > &#128161; Alternativamente, você pode usar a classe estática [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions], que inclui propriedades integradas de cadeia de caracteres para várias regiões do Azure.

1. Invoque o método [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion] com um parâmetro de **região** e o método [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] fluentemente na variável **construtor** armazenando o resultado em uma variável chamada **cliente** do tipo **CosmosClient** encapsulado em uma instrução using:

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. Use o método **GetContainer** da variável **cliente** para recuperar o contêiner existente usando o nome do banco de dados (*cosmicworks*) e o nome do contêiner (*produtos*):

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Crie duas variáveis de **cadeia de caracteres** denominadas **ID** e **categoryId**gerando um novo valor de **Guid** e armazenando o resultado como uma cadeia de caracteres:

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. Crie uma nova variável chamada **item** do tipo **Product** passando na variável **ID**, um valor de cadeia de caracteres de **Quadro de bicicleta polido** e a variável **categoryId** como parâmetros de construtor:

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. Invoque de forma assíncrona o método **CreateItemAsync\<\>** da variável **contêiner** passando a variável **item** como parâmetro e armazenando o resultado em uma variável chamada **resposta**:

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. Invoque o método estático **Console.WriteLine** para imprimir o código de status HTTP da resposta e o preço da solicitação (em unidades de solicitação):

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. Quando terminar, seu arquivo de código deverá incluir:

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";    

    CosmosClientBuilder builder = new (endpoint, key);            
    
    string region = "West Europe";
    
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);
    
    var response = await container.CreateItemAsync<Product>(item);
    
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. **Salve** o arquivo do código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto para a pasta **22-sdk-multi-region** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

1. Crie e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O código de status HTTP e o preço da solicitação (em RUs) devem ser impressos no console.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
