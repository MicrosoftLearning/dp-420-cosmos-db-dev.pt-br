---
lab:
  title: Configurar uma política de índice de contêiner do Azure Cosmos DB for NoSQL usando o SDK
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Configurar uma política de índice de contêiner do Azure Cosmos DB for NoSQL usando o SDK

As políticas de indexação podem ser gerenciadas de qualquer um dos SDKs do Azure Cosmos DB. O SDK do .NET inclui especificamente um conjunto de classes que podem ser usadas para arquitetar e enviar por push uma nova política de indexação para um contêiner no Azure Cosmos DB for NoSQL.

Neste laboratório, você criará uma política de indexação personalizada para um contêiner usando o SDK do .NET

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

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave**posteriormente neste exercício.

1. Volte para o **Visual Studio Code**.

## Criar uma nova política de indexação usando o SDK do .NET

O SDK do .NET contém um conjunto de classes relacionadas à classe [Microsoft.Azure.Cosmos.IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] pai para criar novas políticas de indexação no código.

1. No painel **Explorer**, navegue até a pasta **12-custom-index-policy**.

1. Abra o arquivo de código **script.cs**.

1. Atualize a variável existente chamada **ponto de extremidade** com o seu valor definido como o **ponto de extremidade** da conta do Azure Cosmos DB criada anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/**, a instrução C# será: **ponto de extremidade da cadeia de caracteres = "https&shy;://dp420.documents.azure.com:443/";**.

1. Atualize a variável existente nomeada **chave** com o seu valor definido como a **chave** da conta do Azure Cosmos DB criada anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se sua chave for: **fDR2ci9QgkdkvERTQ==**, então a instrução C# será: **chave de cadeia de caracteres = "fDR2ci9QgkdkvERTQ==";**.

1. Crie uma nova variável do tipo [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] chamada **política** usando o construtor vazio padrão:

    ```
    IndexingPolicy policy = new ();
    ```

1. Defina a propriedade [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode] da variável **política** como um valor de [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]:

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. Adicione um novo objeto do tipo [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] com sua propriedade [Caminho][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] definida como um valor de **/*** à propriedade de coleção [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] na variável de **política**:

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. Adicionar um novo objeto do tipo [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] com sua propriedade [Caminho][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] definida como um valor de **/nome/?** para a propriedade de coleção [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] na variável de **política**:

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. Crie uma nova variável do tipo [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] chamada **opções** passando os valores ``products`` e ``/categoryId`` como parâmetros de construtor:

    ```
    ContainerProperties options = new ("products", "/category/name");
    ```

1. Atribua a variável de **política** à propriedade [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] da variável **opções**:

    ```
    options.IndexingPolicy = policy;
    ```

1. Invoque de forma assíncrona o método [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] da variável de **banco de dados** passando na variável **opções** como um parâmetro de construtor e armazenando o resultado em uma variável do tipo [Contêiner][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] chamada **contêiner**:

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. Use o método estático interno **Console.WriteLine** para imprimir a propriedade [ID][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] da classe Contêiner com um cabeçalho intitulado **Contêiner Criado**:

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. Depois de terminar, seu arquivo de código deverá incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    IndexingPolicy policy = new ();
    policy.IndexingMode = IndexingMode.Consistent;
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );

    ContainerProperties options = new ("products", "/category/name");
    options.IndexingPolicy = policy;

    Container container = await database.CreateContainerIfNotExistsAsync(options);
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. **Salve** o arquivo **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **12-custom-index-policy** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. O script agora produzirá o nome do contêiner recém-criado:

    ```
    Container Created [products]
    ```

1. Feche o terminal integrado.

1. Retorne ao navegador da Web.

## Observe uma política de indexação criada pelo SDK do .NET usando o Data Explorer

Assim como acontece com qualquer outra política de indexação, você pode usar o Data Explorer para exibir políticas enviadas por push usando os SDKs do .NET. Agora você usará o portal para revisar a política criada neste laboratório a partir do código.

1. No recurso de conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o novo nó do contêiner **products** dentro da árvore de navegação **API NoSQL**.

1. No nó do contêiner de **produtos** da árvore de navegação da **API NOSQL**, selecione **Configurações**.

1. Observe a política de indexação na seção **Política de indexação**:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; Esta é a representação JSON da política de indexação que você criou usando o SDK do .NET neste laboratório.

1. Feche a janela ou a guia do navegador da Web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
