---
lab:
  title: Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL

As consultas do Azure Cosmos DB normalmente terão várias páginas de resultados. A paginação é feita automaticamente no lado do servidor quando o Azure Cosmos DB não pode retornar todos os resultados da consulta em uma única execução. Em muitos aplicativos, você desejará escrever código usando o SDK para processar os resultados da consulta em lotes de maneira eficiente.

Neste laboratório, você criará um iterador de feed que pode ser usado em um loop para iterar sobre todo o conjunto de resultados.

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

    1. Observe o campo**PRIMARY CONNECTION STRING**. Você usará esse valor de **cadeia de conexão** posteriormente nesse exercício.

1. Volte para o **Visual Studio Code**.

## Propagar a conta do Azure Cosmos DB for NoSQL com dados

A ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] implanta dados de exemplo em qualquer conta do Azure Cosmos DB for NoSQL. A ferramenta é de software livre e está disponível por meio do NuGet. Você instalará esta ferramenta no Azure Cloud Shell e a usará para propagar o seu banco de dados.

1. No **Visual Studio Code**, abra o menu **Terminal** e selecione **Novo Terminal** para abrir uma nova instância de terminal.

1. Instale a ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] para uso global em seu computador.

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; Esse comando poderá levar alguns minutos para ser concluído. Esse comando irá gerar a mensagem de aviso (*A ferramenta "cosmicworks" já está instalada) se você já tiver instalado a versão mais recente dessa ferramenta anteriormente.

1. Execute o cosmicworks para propagar sua conta do Azure Cosmos DB com as seguintes opções de linha de comando:

    | **Opção** | **Valor** |
    | ---: | :--- |
    | **-c** | *O valor da cadeia de conexão que você verificou anteriormente neste laboratório* |
    | **--number-of-employees** | *O comando cosmicworks preenche o banco de dados com contêineres de funcionários e produtos com 1000 e 200 itens, respectivamente, a menos que especificado de outra forma* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; Por exemplo, se o seu ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/** e sua chave for: **fDR2ci9QgkdkvERTQ==**, o comando será: ``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. Aguarde até que o comando do **cosmicworks** termine de preencher a conta com um banco de dados, um contêiner e itens.

1. Feche o terminal integrado.

## Paginar através de pequenos conjuntos de resultados de uma consulta SQLusando o SDK

Ao processar resultados de consultas, é preciso garantir que o código avance por todas as páginas de resultados e verifique se há mais páginas restantes antes de fazer solicitações subsequentes.

1. No **Visual Studio Code**, no painel **Explorer**, navegue até a pasta **10-paginate-results-sdk**.

1. Abra o arquivo de código **product.cs**.

1. Observe a classe **Product** e suas propriedades correspondentes. Especificamente, este laboratório usará as propriedades **id**, **nome** e **preço**.

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

1. Crie uma nova variável chamada **sql** do tipo *cadeia de caracteres* com um valor de **SELECT p.id, p.name, p.price FROM products p**:

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. Crie uma nova variável do tipo [QueryDefinition][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition] passando a variável **sql** como parâmetro para o constructo:

    ```
    QueryDefinition query = new (sql);
    ```

1. Crie uma nova variável do tipo [QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] chamada **options** usando o constructo vazio padrão:

    ```
    QueryRequestOptions options = new ();
    ```

1. Defina a propriedade [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] da variável **options** como o valor **50**:

    ```
    options.MaxItemCount = 50;
    ```

1. Crie uma nova variável denominada **iterator** do tipo [FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1] invocando o método genérico [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] da classe [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container], passando as variáveis **query** e **options** como parâmetros:

    ```
    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);
    ```

1. Crie um loop **while** que verifique a propriedade [HasMoreResults][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults] da variável **iterator**:

    ```
    while (iterator.HasMoreResults)
    {
        
    }
    ```

1. Dentro do loop **while**, invoque de forma assíncrona o método [ReadNextAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync] da variável **iterator**, armazenando o resultado em uma variável chamada **products** do tipo genérico [FeedResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1] usando a classe **Product**:

    ```
    FeedResponse<Product> products = await iterator.ReadNextAsync();
    ```

1. Ainda dentro do loop **while**, crie um novo loop **foreach** iterando sobre a variável **products** usando a variável **product** para representar uma instância do tipo **Product**:

    ```
    foreach (Product product in products)
    {

    }
    ```

1. No loop **foreach**, use o método estático interno **Console.WriteLine** para formatar e imprimir as propriedades **id**, **name** e **price** da variável **product**:

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. De volta ao loop **while**, use o método estático interno **Console.WriteLine** para imprimir a mensagem *Pressione qualquer tecla para obter mais resultados*:

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. Ainda dentro do loop **while**, use o método estático interno **Console.ReadKey** para ouvir a próxima entrada de pressionamento de tecla:

    ```
    Console.ReadKey();
    ```

1. Depois de terminar, seu arquivo de código deverá incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    QueryDefinition query = new (sql);

    QueryRequestOptions options = new ();
    options.MaxItemCount = 50;

    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);

    while (iterator.HasMoreResults)
    {
        FeedResponse<Product> products = await iterator.ReadNextAsync();
        foreach (Product product in products)
        {
            Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
        }

        Console.WriteLine("Press any key for next page of results");
        Console.ReadKey();        
    }
    ```

1. **Salve** o arquivo **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **10-paginate-results-sdk** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. O script agora exibirá o primeiro conjunto de 50 itens que correspondem à consulta. Pressione qualquer tecla para obter o próximo conjunto de 50 itens até que a consulta tenha iterado sobre todos os itens correspondentes.

    > &#128161; A consulta corresponderá a centenas de itens no contêiner de produtos.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
