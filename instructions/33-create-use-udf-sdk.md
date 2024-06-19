---
lab:
  title: Implementar e usar funções definidas pelo usuário com o SDK
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Implementar e usar funções definidas pelo usuário com o SDK

O SDK do .NET para o Azure Cosmos DB for NoSQL pode ser usado para gerenciar e invocar constructos de programação do lado do servidor diretamente de um contêiner. Ao preparar um novo contêiner, pode fazer sentido usar o SDK do .NET para publicar UDFs diretamente em um contêiner, em vez de executar as tarefas manualmente usando o Data Explorer.

Neste laboratório, você criará um nova UDF usando o SDK do .NET e, em seguida, usará o Data Explorer para validar se a UDF está funcionando corretamente.

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
    | **Modo de capacidade** | *Taxa de transferência provisionada* |
    | **Aplicar Desconto na Camada Gratuita** | *Não Aplicar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave** posteriormente neste exercício.

1. Sem fechar a janela do navegador, abra o **Visual Studio Code**.

## Propagar a conta do Azure Cosmos DB for NoSQL com dados

A ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] implanta dados de exemplo em qualquer conta do Azure Cosmos DB for NoSQL. A ferramenta é de software livre e está disponível por meio do NuGet. Você instalará esta ferramenta no Azure Cloud Shell e a usará para propagar o seu banco de dados.

1. No **Visual Studio Code**, abra o menu **Terminal** e selecione **Novo Terminal** para abrir uma nova instância de terminal.

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

1. Aguarde até que o comando do **cosmicworks** termine de preencher a conta com um banco de dados, um contêiner e itens.

1. Feche o terminal integrado.

## Crie uma função definida pelo usuário (UDF) usando o SDK do .NET

A classe [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] no SDK do .NET inclui uma propriedade [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] que é usada para executar operações CRUD em procedimentos armazenados, UDFs e gatilhos diretamente do SDK. Você usará essa propriedade para criar um nova UDF e, em seguida, enviá-lo para um contêiner do Azure Cosmos DB for NoSQL. A UDF que criaremos usando o SDK calculará o preço do produto com o imposto, o que nos permitirá executar consultas SQL nos produtos usando o preço com o imposto.

1. Em **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **33-create-use-udf-sdk**.

1. Abra o arquivo de código **script.cs**.

1. Adicione um bloco de uso para o namespace [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]:

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

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

1. Crie uma nova variável do tipo [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] chamada props usando o constructo vazio padrão:

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. Defina a propriedade [ID][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] da variável **props** com o valor do **imposto**:

    ```
    props.Id = "tax";
    ```

1. Defina a propriedade [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] da variável **props** como um valor de **props.Body = "function tax(i) { return i * 1.25; }";**:

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. Chame de forma assíncrona o método [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] da variável **container**, passando a variável **props** como parâmetro e salvando o resultado em uma variável denominada **udf** do tipo [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]:

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. Use o método estático interno **Console.WriteLine** para imprimir a propriedade [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] da classe UserDefinedFunctionResponse com um cabeçalho intitulado **Criar UDF**:

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. Depois de terminar, seu arquivo de código deverá incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **Salve** o arquivo **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **33-create-use-udf-sdk** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. O script agora exibirá o nome da UDF recém-criada:

    ```
    Created UDF [tax]
    ```

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

## Teste a UDF usando o Data Explorer

Agora que um nova UDF foi criada no contêiner do Azure Cosmos DB, você usará o Data Explorer para validar se a UDF está funcionando conforme o esperado.

1. Retorne ao seu navegador da Web.

1. No recurso de conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de novos **produtos** dentro da árvore de navegação da **API NOSQL**.

1. Selecione o nó de contêiner de **produtos** na árvore de navegação da **API NOSQL** e selecione **Nova Consulta SQL**.

1. Na guia Consulta, selecione **Executar Consulta** para exibir uma consulta padrão que seleciona todos os itens sem nenhum filtro.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retorne todos os documentos com dois valores de preço projetados. O primeiro valor é o valor bruto do preço do contêiner e o segundo valor é o valor do preço calculado pela UDF:

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. Selecione **Executar Consulta**.

1. Observe os documentos e compare seus campos **price** e **priceWithTax**.

    > &#128221; O campo **priceWithTax** deve ter um valor 25% maior do que o campo **price**.

1. Feche a janela ou a guia do navegador da Web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
