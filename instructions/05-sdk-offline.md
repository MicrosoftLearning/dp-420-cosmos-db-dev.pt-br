---
lab:
  title: Configurar o SDK do Azure Cosmos DB for NoSQL para desenvolvimento offline
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Configurar o SDK do Azure Cosmos DB for NoSQL para desenvolvimento offline

O emulador do Azure Cosmos DB é uma ferramenta local que emula o serviço do Azure Cosmos DB para desenvolvimento e teste. O emulador dá suporte ao NoSQL e pode ser usado no lugar do serviço de nuvem ao desenvolver código usando o SDK do Azure para .NET.

Neste laboratório, você se conectará ao emulador do Azure Cosmos DB do SDK do Azure para .NET.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código de laboratório para **DP-420** no ambiente em que você está trabalhando neste laboratório, siga estas etapas para fazer isso. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não estiver familiarizado com a interface do Visual Studio Code, examine a [Documentação de Introdução][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clonar** para clonar o repositório GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Depois que o repositório tiver sido clonado, abra a pasta local selecionada no **Visual Studio Code**.

## Inicie o emulador do Azure Cosmos DB

Seu ambiente já deve ter o emulador pré-instalado. Caso contrário, confira as [instruções de instalação][docs.microsoft.com/azure/cosmos-db/local-emulator] para instalar o emulador do Azure Cosmos DB. Depois que o emulador for iniciado, você poderá recuperar a cadeia de conexão e usá-la para se conectar ao emulador usando o SDK do Azure para .NET ou qualquer outro SDK de sua escolha.

1. Inicie o **Emulador do Azure Cosmos DB**.

    > &#128221; Você pode ser solicitado a conceder acesso ao administrador para iniciar o emulador. No ambiente de laboratório, a conta de **administrador** tem a mesma senha que a conta de **estudante**.

    > &#128161; O emulador do Azure Cosmos DB está fixado na barra de tarefas do Windows e no Menu Iniciar. ***Se o Emulador não iniciar a partir dos ícones fixados, tente abri-lo clicando duas vezes no*** ***arquivo*** **C:\Arquivos de Programas\Emulador do Azure Cosmos DB\CosmosDB.Emulator.exe**. Observe que o emulador leva de 20 a 30 segundos para ser iniciado.

1. Aguarde até que o emulador abra automaticamente o seu navegador padrão e navegue até a página de aterrissagem **localhost:8081/_explorer/index.html**.

1. Na página de aterrissagem do **Emulador do Azure Cosmos DB**, navegue até o painel **Início rápido**.

1. Este painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta do SDK. Especificamente:

    > &#128221; Observe o campo **Cadeia de conexão primária**. Você usará este valor de **cadeia de conexão** mais adiante neste exercício.

1. Navegue até o painel do **Explorer**.

1. No **Data Explorer**, observe que não há nós na árvore de navegação da **API NoSQL**.

1. Deixe esta guia aberta e alterne para o **Visual Studio Code**.

## Conectar-se ao emulador do SDK

A biblioteca **Microsoft.Azure.Cosmos** já foi pré-instalada no script do .NET que você usará neste exercício. Além disso, parte do código clichê já foi escrito para economizar tempo. Você precisará atualizar o valor da cadeia de conexão clichê e gravar algumas linhas de código para concluir o script.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **05-sdk-offline**.

1. Abra o arquivo de código **script.cs** dentro da pasta **05-sdk-offline**.

1. Atualize a variável existente chamada **connectionString** com o seu valor definido para a **cadeia de conexão** do Emulador do Azure Cosmos DB.
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; O URI do emulador normalmente é ***localhost:[port]*** usando SSL com a porta padrão definida como **8081**.

    > &#128221; *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* é a chave padrão para todas as instalações do emulador. Esta chave pode ser alterada usando opções de linha de comando.

1. Invoque de forma assíncrona o método [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] da variável **cliente** que passa o nome do novo banco de dados (**cosmicworks**) que você deseja criar dentro do emulador e armazenar o resultado em uma variável do tipo [Banco de dados][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]:

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Use o método estático interno **Console.WriteLine** para imprimir a propriedade [ID][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] da classe Database com um cabeçalho intitulado **Novo banco de dados**:

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. Depois de terminar, o arquivo de código agora deve incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **05-sdk-offline** e selecione **Abrir no terminal integrado** para abrir uma nova instância de terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **05-sdk-offline**.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Feche o terminal integrado.

## Exibir os dados no emulador

Agora que você criou um novo banco de dados no emulador do Azure Cosmos DB, usará o **Data Explorer** online para observar o novo banco de dados da API NoSQL no emulador.

1. Voltar para o navegador

1. Na página de aterrissagem do **Emulador do Azure Cosmos DB**, navegue até o painel do **Explorer**.

1. No **Data Explorer**, atualize a **API NoSQL** para observar o novo nó de banco de dados **cosmicworks** dentro da árvore de navegação.

1. Volte para o **Visual Studio Code**.

## Criar e exibir um novo contêiner

A criação de um novo contêiner é semelhante ao padrão usado para criar um novo banco de dados. O código que você aprender aqui será relevante se você criar ou não recursos na nuvem ou no emulador, basta alterar a cadeia de conexão. Você expandirá ainda mais o arquivo de script para criar um novo contêiner junto com o banco de dados.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **05-sdk-offline**.

1. Abra o arquivo de código **script.cs** dentro da pasta **05-sdk-offline** novamente.

1. Invoque de forma assíncrona o método [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] da variável de **banco de dados** que passa o nome do novo contêiner (**produtos**), o caminho da chave de partição (**/categoryId**) e a taxa de transferência (**400**) que você deseja criar no banco de dados **cosmicworks** e armazenar o resultado em uma variável do tipo [Contêiner][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]:

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. Use o método estático interno **Console.WriteLine** para imprimir a propriedade [ID][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] da classe Container com um cabeçalho intitulado **Novo contêiner**:

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. Depois de terminar, o arquivo de código agora deve incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **05-sdk-offline** e selecione **Abrir no terminal integrado** para abrir uma nova instância de terminal.

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

1. Alterne para o navegador.

1. Na página de aterrissagem do **Emulador do Azure Cosmos DB**, navegue até o painel do **Explorer**.

1. No **Data Explorer**, atualize a **API do SQL** para observar o novo nó de contêiner **produtos** dentro do nó de banco de dados **cosmicworks**.

1. Feche a janela ou a guia do navegador da Web.

## Pare o emulador do Azure Cosmos DB

É importante interromper o emulador quando terminar de usá-lo, pois ele pode usar recursos do sistema em seu ambiente. Você usará o ícone de bandeja do sistema para interromper o emulador e todas as instâncias em execução.

 Navegue até o ícone do emulador na bandeja do sistema Windows, abra o menu de contexto e selecione **Sair** para desligar o emulador.

> &#128221; Pode levar um minuto para que todas as instâncias do emulador sejam encerradas.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
