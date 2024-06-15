---
lab:
  title: Conectar-se ao Azure Cosmos DB for NoSQL com o SDK
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Conectar-se ao Azure Cosmos DB for NoSQL com o SDK

O SDK do Azure para .NET é um conjunto de bibliotecas que fornece uma interface de desenvolvedor consistente para interagir com muitos serviços do Azure. O SDK do Azure para .NET foi desenvolvido de acordo com a especificação .NET Standard 2.0, o que garante que ele possa ser usado em aplicativos .NET Framework (4.6.1 ou superior), .NET Core (2.1 ou superior) e .NET (5 ou superior).

Neste laboratório, você se conectará a uma conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para .NET.

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
    | **Aplicar Desconto na Camada Gratuita** | *Não aplicar* |
    | **Limitar a quantidade total de taxa de transferência que pode ser provisionada nesta conta** | *Desmarcado* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impedem a criação de um grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará este valor de **chave** posteriormente neste exercício.

1. Mantenha a guia do navegador aberta, pois voltaremos a ela mais tarde.

## Exibir a biblioteca Microsoft.Azure.Cosmos no NuGet

O site do NuGet contém um índice pesquisável de pacotes que estão disponíveis para importação em seus aplicativos .NET. Para importar pacotes de pré-lançamento, como **Microsoft.Azure.Cosmos**, você pode usar o site do NuGet para obter as versões e os comandos apropriados para importar o pacote para seus aplicativos.

1. Em uma nova guia do navegador, navegue até o site do NuGet (``nuget.org``).

1. Examine a descrição do NuGet, o gerenciador de pacotes para .NET, e seus recursos.

1. Pesquise a biblioteca **Microsoft.Azure.Cosmos** em NuGet.org.

1. Selecione a guia **CLI do .NET** para observar o comando necessário para importar a versão mais recente dessa biblioteca para um projeto .NET.

    > &#128161; Não é necessário registrar esse comando. Você usará uma versão específica da biblioteca mais adiante neste exercício.

1. Feche a janela ou a guia do navegador da Web.

## Importar a biblioteca Microsoft.Azure.Cosmos para um projeto .NET

A CLI do .NET inclui um comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar pacotes de um feed de pacotes pré-configurado. Uma instalação do .NET usa o NuGet como seu feed de pacotes padrão.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **04-sdk-connect**.

1. Abra o menu de contexto da pasta **04-sdk-connect** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido para a pasta **04-sdk-connect**.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.*
    ```

1. Feche o terminal integrado.

## Usar a biblioteca Microsoft.Azure.Cosmos

Depois que a biblioteca do Azure Cosmos DB do SDK do Azure para .NET tiver sido importada, você poderá usar imediatamente suas classes no namespace [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] para se conectar a uma conta do Azure Cosmos DB for NoSQL. A classe [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] é a classe principal usada para fazer a conexão inicial com uma conta do Azure Cosmos DB for NoSQL.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **04-sdk-connect**.

1. Abra o arquivo de código **script.cs** vazio.

1. Adicione blocos de uso para os namespaces **System** e **System.Linq** incorporados:

    ```
    using System;
    using System.Linq;
    ```

1. Adicione um bloco de uso para o namespace [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]:

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. Adicione uma variável de **cadeia de caracteres** denominada **ponto de extremidade** com seu valor definido como o **ponto de extremidade** da conta do Azure Cosmos DB que você criou anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/**, a instrução C# será: **ponto de extremidade da cadeia de caracteres = "https&shy;://dp420.documents.azure.com:443/";**.

1. Adicione uma variável de **cadeia de caracteres** chamada **chave** com seu valor definido como a **chave** da conta do Azure Cosmos DB que você criou anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se sua chave for: **fDR2ci9QgkdkvERTQ==**, então a instrução C# será: **chave de cadeia de caracteres = "fDR2ci9QgkdkvERTQ==";**.

1. Adicione uma nova variável chamada **cliente** do tipo [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] usando as variáveis **ponto de extremidade** e **chave** no constructo:
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. Adicione uma nova variável chamada **conta** do tipo [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] usando o resultado assíncrono da invocação do método [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] da variável **cliente**:

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. Use o método estático interno **Console.WriteLine** para imprimir a propriedade [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] da classe AccountProperties com um cabeçalho intitulado **Nome da Conta**:

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. Use o método estático interno **Console.WriteLine** para consultar a propriedade [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] da classe AccountProperties e, em seguida, imprima a propriedade [Nome][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] do primeiro resultado com um cabeçalho intitulado **Região Primária**:

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. Quando terminar, seu arquivo de código deverá incluir:
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **Save** o arquivo de código **script.cs**.

## Testar o script

Agora que o código .NET para conexão com a conta do Azure Cosmos DB for NoSQL está concluído, você pode testar o script. Esse script imprimirá o nome da conta e o nome da primeira região gravável. Ao criar a conta, você especificou um local e deve esperar ver esse mesmo valor de local impresso como resultado desse script.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **04-sdk-connect** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. O script agora emitirá o nome da conta e a primeira região gravável. Por exemplo, se você der à conta o nome **dp420** e a primeira região gravável for **Oeste dos EUA 2**, o script apresentará a saída:

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
