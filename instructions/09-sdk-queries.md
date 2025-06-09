---
lab:
  title: Executar uma consulta com o SDK do Azure Cosmos DB for NoSQL
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Executar uma consulta com o SDK do Azure Cosmos DB for NoSQL

A versão mais recente do SDK do .NET para Azure Cosmos DB for NoSQL torna mais fácil do que nunca consultar um contêiner e iterar de forma assíncrona sobre conjuntos de resultados usando as melhores práticas e recursos de linguagem do C# mais recentes.

Esta biblioteca tem uma funcionalidade especial para facilitar a consulta ao Azure Cosmos DB usando [https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet].

Neste laboratório, você usará um fluxo assíncrono para iterar em um conjunto de resultados grande retornado do Azure Cosmos DB for NoSQL. Você usará o SDK do .NET para consultar e iterar sobre os resultados.

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

1. Entre no portal usando as credenciais Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, pesquise por *Cosmos DB* e, em seguida, crie um recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes em seus valores padrão:

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

## Iterar sobre os resultados de uma consulta SQL usando o SDK

Agora você usará um fluxo assíncrono para criar um loop foreach simples de entender sobre os resultados paginados do Azure Cosmos DB. Nos bastidores, o SDK gerenciará o iterador de feed e garantirá que as solicitações subsequentes sejam invocadas corretamente.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **09-execute-query-sdk**.

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

    > &#128221; Por exemplo, se a sua chave for: **fDR2ci9QgkdkvERTQ==**, a instrução C# será: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Vamos acrescentar um código adicional ao final do arquivo **script.cs**, criar uma nova variável chamada **sql** de tipo *cadeia de caracteres* com um valor de **SELECT * FROM products p**:

    ```
    string sql = "SELECT * FROM products p";
    ```

1. Crie uma nova variável do tipo [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] passando a variável **sql** como um parâmetro para o construtor:

    ```
    QueryDefinition query = new (sql);
    ```

1. Crie um loop **while** invocando o método genérico [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] da classe [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] passando a variável de **consulta** como um parâmetro e iterando sobre os resultados:

    ```
    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
    }
    ```

1. No loop **while**, leia o próximo resultado de forma assíncrona, use o método estático interno **Console.WriteLine** para formatar e imprimir as propriedades de **id**, **nome** e **preço** da variável **product**:

    ```
    FeedResponse<Product> response = await feed.ReadNextAsync();
    foreach (Product product in response)
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. Depois de terminar, o arquivo de código agora deve incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
        FeedResponse<Product> response = await feed.ReadNextAsync();
        foreach (Product product in response)
        {
            Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
        }
    }
    ```

1. **Salve** o arquivo **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **09-execute-query-sdk** e selecione **Abrir no terminal integrado** para abrir uma nova instância de terminal.

1. Adicione o pacote Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. O script agora produzirá todos os produtos no contêiner

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet]: https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
