---
lab:
  title: Solucionar problemas de um aplicativo usando o SDK do Azure Cosmos DB for NoSQL
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Solucionar problemas de um aplicativo usando o SDK do Azure Cosmos DB for NoSQL

O Azure Cosmos DB oferece um amplo conjunto de códigos de resposta, que nos ajudam a solucionar facilmente problemas que podem surgir com nossos diferentes tipos de operação. O problema é garantir que programemos o tratamento adequado de erros ao criar aplicativos para o Azure Cosmos DB.

Neste laboratório, criaremos um programa orientado por menu que nos permitirá inserir ou excluir um de dois documentos. O principal objetivo deste laboratório é nos apresentar como usar alguns dos códigos de resposta mais comuns e como usá-los no código de tratamento de erros do nosso aplicativo.  Embora codifiquemos o tratamento de erros para vários códigos de resposta, só acionaremos dois tipos diferentes de condições.  Além disso, o tratamento de erros não fará nada complexo, dependendo do código de resposta, exibirá uma mensagem na tela ou aguardará 10 segundos e repetirá a operação mais uma vez. 

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não se familiarizou com a interface do Visual Studio Code, confira o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Depois que o repositório tiver sido clonado, abra a pasta local selecionada no **Visual Studio Code**.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você irá selecionar a quais APIs você quer que a conta dê suporte (por exemplo, a **API do Mongo** ou a **API do NoSQL**). Após a conta do Azure Cosmos DB for NoSQL ter concluído o provisionamento, você poderá recuperar o ponto de extremidade e a chave. Use o ponto de extremidade e a chave para se conectar à conta do Azure Cosmos DB for NoSQL programaticamente. Use o ponto de extremidade e a chave nas cadeias de conexão do SDK do Azure para .NET ou qualquer outro SDK.

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

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** mais adiante neste exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave**posteriormente neste exercício.

1. Minimize, mas não feche a janela do navegador. Voltaremos ao portal do Azure alguns minutos depois de iniciarmos uma carga de trabalho em segundo plano nas próximas etapas.


## Importar a biblioteca Microsoft.Azure.Cosmos para um script .NET

A CLI do .NET inclui um comando [adicionar pacote][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar pacotes de um feed de pacote pré-configurado. Uma instalação do .NET usa o NuGet como seu feed de pacotes padrão.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **26-sdk-troubleshoot**.

1. Abra o menu de contexto da pasta **26-sdk-troubleshoot** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido para a pasta **26-sdk-troubleshoot**.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

## Execute um script para criar opções orientadas por menu para inserir e excluir documentos.

Antes de executarmos nosso aplicativo, precisamos conectá-lo à nossa conta do Azure Cosmos DB. 

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **26-sdk-troubleshoot**.

1. Abra o arquivo de código **Program.cs**.

1. Atualize a variável existente chamada **ponto de extremidade** com o seu valor definido como o **ponto de extremidade** da conta do Azure Cosmos DB criada anteriormente.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for **https&shy;://dp420.documents.azure.com:443/**, a instrução C# será **ponto de extremidade de cadeia de caracteres somente leitura estático privado = "https&shy;://dp420.documents.azure.com:443/";**.

1. Atualize a variável existente nomeada **chave** com o seu valor definido como a **chave** da conta do Azure Cosmos DB criada anteriormente.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se a chave for **fDR2ci9QgkdkvERTQ==**, a instrução C# será **chave de cadeia de caracteres somente leitura estática privada = "fDR2ci9QgkdkvERTQ==";**.

1. Salve o arquivo.

1. Crie e execute o projeto usando o comando [executar dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```
    > &#128221; Este é um programa muito simples.  Ele exibirá um menu com cinco opções, conforme mostrado abaixo. Duas opções para inserir um documento predefinido, duas para excluir um documento predefinido e uma opção para sair do programa.

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## Hora de inserir e excluir documentos.

1. Selecione **1** e **ENTER** para inserir o primeiro documento. O programa inserirá o primeiro documento e retornará a mensagem a seguir.

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. Novamente, selecione **1** e **ENTER** para inserir o primeiro documento. Desta vez, o programa falhará com uma exceção. Ao examinar a pilha de erros, poderemos encontrar o motivo da falha do programa. Como podemos ver na mensagem extraída da pilha de erros, nos deparamos com uma exceção não tratada "Conflito (409)"

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. Como estamos inserindo um documento, precisaremos revisar a lista de [criação de códigos de status de documento][/rest/api/cosmos-db/create-a-document#status-codes] comuns retornados quando um documento for criado. A descrição desse código é que *a ID fornecida para o novo documento foi usada por um documento existente*. Isso é óbvio, já que acabamos de executar a opção de menu para criar o mesmo documento há alguns instantes.

1. Analisando melhor a pilha, podemos ver que essa exceção foi chamada a partir da linha 100 e que, por sua vez, foi chamada a partir da linha 64.

    ```
    at Program.CreateDocument1(Container Customer) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 100   
   at Program.CompleteTaskOnCosmosDB(String consoleinputcharacter, Container container) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 64
    ```

1. Ao revisar a linha 100, conforme esperado, o erro foi causado pela operação * CreateItemAsync*. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. Além disso, ao revisar as linhas 100 a 103, é óbvio que esse código não tem tratamento de erros. Precisaremos corrigir isso. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. Precisaremos decidir o que nosso código de tratamento de erros deve fazer. Ao revisar a [criação de códigos de status de documento][/rest/api/cosmos-db/create-a-document#status-codes], poderíamos optar por criar código de tratamento de erros para todos os códigos de status possíveis para esta operação.  Neste laboratório, consideraremos apenas os códigos de status 403 a 409 dessa lista.  Todos os outros códigos de status retornados apenas exibirão a mensagem de erro do sistema.

    > &#128221; Observe que, embora codifiquemos uma tarefa de tratamento de erros para exceções 403, neste laboratório não geraremos uma exceção 403.

1. Vamos adicionar tratamento de erros para a função chamada **CompleteTaskOnCosmosDB**. Localize o loop **while** na função **Main** na linha **45** e encapsule as chamadas de **CompleteTaskOnCosmosDB** com código e tratamento de erros. Substituiremos a instrução **CompleteTaskOnCosmosDB** na linha **47** para o código abaixo.  A primeira coisa a observar neste novo código é que, na **captura** estamos capturando uma exceção do tipo da classe **CosmosException**.  Essa classe inclui a propriedade **StatusCode**, que retorna o código de status de conclusão da solicitação do serviço do Azure Cosmos DB. A propriedade **StatusCode** é do tipo **System.Net.HttpStatusCode**. Podemos usar esse valor e compará-lo com os nomes de campo do [Código de status HTTP][dotnet/api/system.net.httpstatuscode] do .NET.  

    ```C#
        try
        {
            await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. Salve o arquivo e, como falhamos, precisaremos executar nosso programa Menu novamente, portanto, execute o comando:

    ```
    dotnet run
    ```
 
1. Novamente, selecione **1** e **ENTER** para inserir o primeiro documento. Desta vez, não falhamos, mas recebemos uma mensagem mais amigável sobre o que aconteceu.

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    ```

1. Esse código adicionou o tratamento de erros para exceções *403* e *409*. Agora, vamos adicionar código para alguns tipos comuns de exceções de comunicação. Há três tipos comuns de exceções de comunicação: *429*, *503*e *408* ou muitas solicitações, serviço indisponível e tempo limite de solicitação, respectivamente. Ao redor da linha *66* agora deve haver uma instrução **padrão**, portanto, adicione o código abaixo logo após a instrução **interromper:** anterior e logo antes da instrução **padrão**.  O código verificará se encontramos alguma dessas exceções de comunicação e, em caso afirmativo, aguardará 10 segundos e, em seguida, tentará inserir o documento mais uma vez.  Vamos adicionar além do código:

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; Observe que, embora codifiquemos uma tarefa sobre o que fazer se encontrarmos uma exceção 429, 503 ou 408, neste laboratório não geraremos um erro com esse tipo de exceção.

1. Nossa função **Main** agora deve ser semelhante a esta:

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                 try
                 {
                     await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                 }
                 catch (CosmosException e)
                 {
                     switch (e.StatusCode.ToString())
                     {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                     }
                }
                

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. Observe que a função **CreateDocument2** também será corrigida pelas alterações acima.

1. Por fim, as funções **DeleteDocument1** e **DeleteDocument2** também precisam que o código a seguir seja substituído pelo código adequado de tratamento de erros semelhante à função **CreateDocument1**. A única diferença com essas funções além de usar **DeleteItemAsync** em vez de **CreateItemAsync** é que [códigos de status de exclusão][/rest/api/cosmos-db/delete-a-document] são diferentes dos códigos de status de inserção. Para as exclusões, nos preocupamos apenas com um código de status **404**, que representa o documento não encontrado. Vamos atualizar o tratamento de erros da chamada de função **CompleteTaskOnCosmosDB** com caso adicional.  Na função **Main**, o código a seguir precisa ser adicionado acima do caso **padrão**:

    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1. Salve o arquivo.

1. Quando terminar de corrigir todas as funções, teste todas as opções de menu várias vezes para ter certeza de que o aplicativo está retornando uma mensagem quando encontrar uma exceção e não está com falha.  Se o aplicativo falhar, corrija os erros e execute novamente o comando:

    ```
    dotnet run
    ```


1. Não espie, mas quando terminar, seus códigos `Main` deverão ser mais ou menos assim.

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## Conclusão

Mesmo os desenvolvedores mais juniores sabem que o tratamento de erros adequado deve ser adicionado a todo o código. Embora o tratamento de erros nesse código seja simples, ele deve ter dado as noções básicas sobre os componentes de exceção do Azure Cosmos DB que permitirão que você crie soluções robustas de tratamento de erros em seu código.


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

