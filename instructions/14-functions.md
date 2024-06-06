---
lab:
  title: Processar dados do Azure Cosmos DB for NoSQL usando o Azure Functions
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Processar dados do Azure Cosmos DB for NoSQL usando o Azure Functions

O gatilho do Azure Cosmos DB para o Azure Functions é implementado usando um processador de feed de alterações. Com esse conhecimento, você pode criar funções que respondam às operações de criar e atualizar no contêiner do Azure Cosmos DB for NoSQL. Se você implementou um processador de feed de alterações manualmente, a configuração do Azure Functions será semelhante.

Neste laboratório, você vai:

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você irá selecionar a qual API você quer que a conta dê suporte (por exemplo, a **API do Mongo** ou a **API do NoSQL**). Após a conta do Azure Cosmos DB for NoSQL ter terminado de provisionar, você poderá recuperar o ponto de extremidade e a chave e usá-los para se conectar à conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para .NET ou qualquer outro SDK de sua escolha.

1. Em uma nova janela ou guia do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Cosmos DB* e, em seguida, crie um novo recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Sem servidor* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impeçam você de criar um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para você se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente neste exercício.

    1. Observe o campo **PRIMARY KEY**. Você usará esse valor de **chave** posteriormente neste exercício.

1. Selecione **Data Explorer** no menu de recursos.

1. No painel do **Data Explorer**, expanda **Novo contêiner** e, a seguir, selecione **Novo Banco de Dados**.

1. No pop-up de **Novo Banco de Dados**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *``cosmicworks``* |

1. De volta ao painel do **Data Explorer**, observe o nó de banco de dados do **cosmicworks** dentro da hierarquia.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *``products``* |
    | **Chave de partição** | *``/categoryId``* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados do **cosmicworks** e observe o nó do contêiner de **produtos** dentro da hierarquia.

1. No painel do **Data Explorer**, selecione **Novo Contêiner** novamente.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *``productslease``* |
    | **Chave de partição** | *``/id``* |

1. De volta ao painel do **Data Explorer**, expanda o nó de banco de dados do ** cosmicworks** e observe o nó de contêiner de **productslease** dentro da hierarquia.

1. Retorne à **Página Inicial** do portal do Azure.

## Criar o Application Insight

Antes de criar o *Aplicativo de Função do Azure*, você precisará habilitar um *Insight de Aplicativo do Azure* para poder monitorar a função do aplicativo. Primeiro, o Insight de Aplicativo precisará de um *workspace do Log Analytics*.

1. Na caixa de pesquisa, procure **workspaces do Log Analytics**.

1. Selecione a opção **+Criar** para criar um novo *workspace do Log Analytics*.

1. Na caixa de diálogo do **workspace do Log Analytics**, insira os seguintes valores para cada configuração e, a seguir, selecione **Examinar+Criar** e selecione **Criar**:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Nome** | *``lab14laworkspace``* |
    | **Localidade** | *Escolha qualquer região disponível* |

1. Após seu *workspace do Log Analytics* ter sido criado, na caixa de pesquisa, procure **Application Insights**.

1. Selecione para **+Criar** um novo *Insight de Aplicativo*.

1. Na caixa de diálogo do **Application Insights**, insira os seguintes valores para cada configuração, selecione **Examinar+Criar** e, em seguida, selecione **Criar**:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura (ambas as entradas)** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Nome** | *``lab14appinsight``* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Workspace do Log Analytics** | *lab14laworkspace* |

Agora você deve conseguir monitorar sua função de aplicativo.

## Criar um aplicativo de funções do Azure e uma função disparada pelo Azure Cosmos DB

Antes de poder começar a escrever código, você precisará criar o recurso do Azure Functions e respectivos recursos dependentes (Application Insights, Armazenamento) usando o assistente de criação.

1. Selecione **+ Criar um recurso**, procure *Funções* e, a seguir, crie um novo recurso de conta do **Aplicativo de Funções** com as configurações a seguir, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Nome** | *Insira um nome globalmente exclusivo* |
    | **Publicar** | *Código* |
    | **Pilha de runtime** | *.NET* |
    | **Versão** | *6 (LTS) em processo* |
    | **Região** | *Escolha qualquer região disponível* |
    | **Conta de armazenamento** | *Criar uma nova conta de armazenamento* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impeçam você de criar um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar essa tarefa.

1. Vá para o recurso de conta do **Azure Functions** recém-criado e navegue até o painel **Funções**.

1. No painel **Funções**, selecione **+ Criar**.

1. No pop-up **Criar função**, crie uma nova função com as configurações a seguir, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Ambiente de desenvolvimento** | *Desenvolvimento no portal* |
    | **Selecionar um modelo** | *Gatilho do Azure Cosmos DB* |
    | **Nova Função** | *``ItemsListener``* |
    | **Conexão da conta do Cosmos DB** | *Selecione Nova* &vert; *Selecione Conta do Azure Cosmos DB* &vert; *Selecione a conta do Azure Cosmos DB que você criou anteriormente* |
    | **Nome do banco de dados** | *``cosmicworks``* |
    | **Nome da coleção** | *``products``* |
    | **Nome da coleção para concessões** | *``productslease``* |
    | **Criar coleção de concessões se ela não existir** | *Não* |

## Implementar o código da função no .NET

A função que você criou anteriormente é um script em C# editado dentro do portal. Agora você usará o portal para escrever uma função curta que irá gerar o identificador exclusivo de qualquer item inserido ou atualizado no contêiner.

1. No painel **ItemsListener** &vert; **Função**, navegue até o painel **Código + Teste**.

1. No editor do script **run.csx**, exclua o conteúdo da área do editor.

1. Na área do editor, faça referência à biblioteca ** Microsoft.Azure.DocumentDB.Core**:

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. Adicione blocos using para os namespaces ** System**, **System.Collections.Generic** e [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents]:

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. Crie um novo método estático chamado **Run** com dois parâmetros:

    1. Um parâmetro chamado **input** do tipo **IReadOnlyList\<\>** com um tipo genérico [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document].

    1. Um parâmetro chamado **log** do tipo **ILogger**.

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. Dentro do método **Run**, invoque o método **LogInformation** da variável **log** repassando uma cadeia de caracteres que calcula o número de itens no lote atual:

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. Ainda dentro do método **Run**, crie um loop foreach que itera sobre a variável **input** usando a variável **item** para representar uma instância do tipo **Document**:

    ```
    foreach(Document item in input)
    {
    }
    ```

1. No loop foreach do método **Run**, invoque o método **LogInformation** da variável **log** repassando uma cadeia de caracteres que imprime a propriedade [ID][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] da variável **item** :

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. Após terminar, o arquivo do seu código agora deve incluir:
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. Expanda a seção **Logs** para se conectar aos logs de fluxo da função atual.

    > &#128161; Pode levar alguns segundos para você se conectar ao serviço de log de fluxo. Você verá uma mensagem na saída do log quando estiver conectado.

1. **Salve** o código da função atual.

1. Observe o resultado da compilação de código em C#. Você deve esperar ver uma mensagem de ** Compilação bem-sucedida** no final do resultado do log.

    > &#128221; Você talvez veja mensagens de aviso na saída do log. Esses avisos não afetarão esse laboratório.

1. **Maximize** a seção do log para expandir a janela da saída de modo a preencher o máximo de espaço disponível.

    > &#128221; Você usará outra ferramenta para gerar itens no seu contêiner do Azure Cosmos DB for NoSQL. Após gerar os itens, você retornará a essa janela do navegador para observar a saída. Não feche a janela do navegador antes da hora.

## Propagar sua conta do Azure Cosmos DB for NoSQL com amostras de dados

Você usará um utilitário de linha de comando que cria um banco de dados do **cosmicworks** e um contêiner de **produtos**. Em seguida, a ferramenta criará um conjunto de itens que você irá observar usando o processador de feed de alterações em execução na janela do seu terminal.

1. Inicie o **Visual Studio Code**.

    > &#128221: Se ainda não estiver familiarizado com a interface do Visual Studio Code, leia o [Guia de Introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. No **Visual Studio Code**, abra o menu ** Terminal** e, a seguir, selecione **Novo Terminal** para abrir uma nova instância de terminal.

1. Instale a ferramenta de linha de comando [nuget.org/packages/cosmicworks] do [cosmicworks] para uso global no seu computador.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Esse comando poderá demorar alguns minutos para ser concluído. Esse comando irá gerar a mensagem de aviso (* A ferramenta "cosmicworks" já está instalada) se você já tiver instalado a versão mais recente dessa ferramenta anteriormente.

1. Execute o cosmicworks para propagar sua conta do Azure Cosmos DB com as seguintes opções de linha de comando:

    | **Opção** | **Valor** |
    | ---: | :--- |
    | **--endpoint** | *O valor do ponto de extremidade copiado anteriormente nesse laboratório* |
    | **--key** | *O valor da chave que você copiou anteriormente nesse laboratório* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por exemplo, se o seu ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/** e sua chave for: **fDR2ci9QgkdkvERTQ==**, o comando será: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Aguarde até que o comando do **cosmicworks** termine de preencher a conta com um banco de dados, um contêiner e itens.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

1. Retorne à guia ou janela do navegador abertas atualmente, com a seção de log do Azure Functions expandida.

1. Observe a saída de log da sua função. O terminal gera uma mensagem de **Operação Detectada** para cada alteração que foi enviada para ele usando o feed de alterações. As operações são separadas em lotes com grupos de aproximadamente 100 operações.

1. Feche a guia ou a janela do seu navegador da web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
