---
lab:
  title: Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL

A classe [Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] inclui um conjunto de métodos de membro para criar, recuperar, atualizar e excluir itens em um contêiner do Azure Cosmos DB for NoSQL. Juntos, esses métodos executam algumas das operações “CRUD” mais comuns em vários itens dentro de contêineres da API NoSQL.

Neste laboratório, você usará o SDK para executar operações CRUD diárias em um item dentro de um contêiner do Azure Cosmos DB for NoSQL.

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

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará este valor de **chave** mais tarde neste exercício.

1. Volte ao **Visual Studio Code**.

## Conectar-se a uma conta do Azure Cosmos DB for NoSQL usando o SDK

Usando as credenciais da conta recém-criada, você se conectará às classes do SDK e criará um novo banco de dados e uma nova instância de contêiner. Em seguida, você usará o Data Explorer para validar a existência das instâncias no portal do Azure.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **06-sdk-crud**.

1. Abra o menu de contexto da pasta **06-sdk-crud** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **06-sdk-crud**.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Crie o projeto usando o comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Feche o terminal integrado.

1. Abra o arquivo de código **script.cs** dentro da pasta **06-sdk-crud**.

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

    > &#128221; Por exemplo, se a chave for: **fDR2ci9QgkdkvERTQ==**, a instrução C# será **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Invoque de maneira assíncrona o método CreateDatabaseIfNotExistsAsync da variável **client** transmitindo o nome do novo banco de dados (**cosmicworks**) que deseja criar e armazenando o resultado em uma variável do tipo **Database**:

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Invoque de maneira assíncrona o método **CreateContainerIfNotExistsAsync** da variável **database**, transmitindo o nome do novo contêiner (**products**), o caminho da chave de partição (**/categoryId**) e a taxa de transferência (**400**) que você deseja criar no banco de dados **cosmicworks**, além de armazenar o resultado em uma variável do tipo **Container**:
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
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
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **06-sdk-crud** e selecione **Abrir no terminal integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Feche o terminal integrado.

1. Alterne para a janela do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o novo nó do contêiner **products** dentro da árvore de navegação **API NoSQL**.

## Executar operações de ponto de leitura e criação em itens com o SDK

Agora você usará o conjunto de métodos assíncronos da classe Microsoft.Azure.Cosmos.Container para executar operações comuns em itens em um contêiner da API NoSQL. Todas essas operações são feitas por meio do modelo de programação assíncrona da tarefa em C#.

1. Volte para o **Visual Studio Code**. Abra o arquivo de código **product.cs** dentro da pasta **06-sdk-crud**.

    > &#128221; Não feche o editor do arquivo **script.cs**.

1. Observe a classe **Product** nesse arquivo de código. Essa classe representa um item de produto que será armazenado e processado dentro do contêiner.

1. Volte à guia do editor do arquivo de código **script.cs**.

1. Crie um objeto do tipo **Product** chamado **saddle** com as seguintes propriedades:

    | Propriedade | Valor |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **marcas** | *{ tan, new, crisp }* |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. Invoque de maneira assíncrona o método [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] genérico da variável **container**, transmitindo a variável **saddle** como o parâmetro de método e usando **Product** como o tipo genérico:

    ```
    await container.CreateItemAsync<Product>(saddle);
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

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **06-sdk-crud** e selecione **Abrir no terminal integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Feche o terminal integrado.

1. Volte à guia do editor do arquivo de código **script.cs**.

1. Exclua as seguintes linhas de código:

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. Crie uma variável de cadeia de caracteres chamada **id** com o valor **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Crie uma variável de cadeia de caracteres chamada **categoryId** com o valor **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Crie uma variável do tipo [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] chamada **partitionKey**, transmitindo a variável **categoryId** como um parâmetro de construtor:

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Invoque de maneira assíncrona o método genérico [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] da variável **container**, transmitindo as variáveis **id** e **partitionkey** como parâmetros de método, usando **Product** como o tipo genérico e armazenando o resultado em uma variável chamada **saddle** do tipo **Product**:

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Invoque o método **Console.WriteLine** estático para imprimir o objeto saddle usando uma cadeia de caracteres de saída formatada:

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
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

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **06-sdk-crud** e selecione **Abrir no terminal integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. Especificamente, observe o texto de saída formatado com a ID, o nome e o preço do item.

1. Feche o terminal integrado.

## Executar operações de ponto de atualização e exclusão com o SDK

Enquanto você aprende a usar o SDK, não é incomum usar uma conta online ou o emulador do SDK do Azure Cosmos DB para atualizar um item e oscilar entre o Data Explorer e seu IDE preferido ao executar uma operação e verificar se a alteração foi aplicada. Aqui, você fará exatamente isso ao atualizar e excluir um item usando o SDK.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Escolha o único item dentro do contêiner e observe os valores das propriedades **name** e **price** do item.

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *Road Saddle* |
    | **Price** | *$45.99* |

    > &#128221; Neste ponto, esses valores não devem ter sido alterados desde que você criou o item. Você vai alterar esses valores neste exercício.

1. Volte para o **Visual Studio Code**. Volte à guia do editor do arquivo de código **script.cs**.

1. Exclua as seguintes linhas de código:

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. Altere a variável **saddle** definindo o valor da propriedade price como **32.55**:

    ```
    saddle.price = 32.55d;
    ```

1. Modifique a variável **saddle** novamente, definindo o valor da propriedade **name** como **Road LL Saddle**:

    ```
    saddle.name = "Road LL Saddle";
    ```

1. Invoque de maneira assíncrona o método genérico [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] da variável **container**, transmitindo a variável **saddle** como o parâmetro de método e usando **Product** como o tipo genérico:

    ```
    await container.UpsertItemAsync<Product>(saddle);
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

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **06-sdk-crud** e selecione **Abrir no terminal integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Feche o terminal integrado.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Escolha o único item dentro do contêiner e observe os valores das propriedades **name** e **price** do item.

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *Road LL Saddle* |
    | **Price** | *$32.55* |

    > &#128221; Neste ponto, esses valores devem ter sido alterados desde que você observou o item.

1. Volte para o **Visual Studio Code**. Volte à guia do editor do arquivo de código **script.cs**.

1. Exclua as seguintes linhas de código:

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. Invoque de maneira assíncrona o método [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] genérico da variável **container**, transmitindo as variáveis **id** e **partitionkey** como o parâmetro de método e usando **Product** como o tipo genérico:

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **06-sdk-crud** e selecione **Abrir no terminal integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Feche o terminal integrado.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Observe que a lista de itens agora está vazia.

1. Feche a janela ou a guia do navegador da Web.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
