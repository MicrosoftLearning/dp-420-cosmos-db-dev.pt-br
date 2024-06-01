---
lab:
  title: "Otimize a política de indexação de\_um contêiner do Azure Cosmos DB for NoSQL para operações comuns"
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Otimize a política de indexação de um contêiner do Azure Cosmos DB for NoSQL para operações comuns

Para cargas de trabalho com gravação intensa ou cargas de trabalho com objetos JSON grandes, pode ser vantajoso otimizar a política de indexação para indexar apenas as propriedades que você sabe que serão utilizadas nas consultas.

Neste laboratório, usaremos um aplicativo .NET de teste para inserir um item JSON grande em um contêiner do Azure Cosmos DB for NoSQL usando a política de indexação padrão e, em seguida, usando uma política de indexação que foi ligeiramente ajustada.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **DP-420** no ambiente em que está trabalhando neste laboratório, siga estas etapas para fazer isso. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não estiver familiarizado com a interface do Visual Studio Code, consulte a documentação de [Introdução][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório do GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Depois que o repositório tiver sido clonado, abra a pasta local que você selecionou no **Visual Studio Code**.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você selecionará a qual das APIs deseja que a conta dê suporte (por exemplo, **API do Mongo** ou **API NoSQL**). Quando o provisionamento da conta do Azure Cosmos DB for NoSQL estiver concluído, você poderá recuperar o ponto de extremidade e a chave e usá-los para se conectar à conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para .NET ou qualquer outro SDK de sua escolha.

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Microsoft Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, pesquise *Cosmos DB* e, em seguida, crie um recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes em seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecione um grupo de recursos existente ou crie um* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Sem servidor* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que o impeçam de criar um grupo de recursos. Se esse for o caso, use o grupo de recursos predefinido existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com esta tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Data Explorer**.

1. No painel **Data Explorer**, selecione **Novo Contêiner**.

1. Na janela pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Criar* &vert; *``cosmicworks``* |
    | **ID do contêiner** | *``products``* |
    | **Chave de partição** | *``/categoryId``* |

1. De volta ao painel **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó do contêiner **produtos** na hierarquia.

1. Na folha de recursos, navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** mais adiante neste exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave** mais adiante neste exercício.

1. Volte para o **Visual Studio Code**.

## Execute o aplicativo .NET de teste usando a política de indexação padrão

Este laboratório tem um aplicativo .NET de teste predefinido que pegará um objeto JSON grande e criará um novo item no contêiner do Azure Cosmos DB for NoSQL. Quando a operação de gravação única for concluída, o aplicativo exibirá o identificador exclusivo do item e a taxa de RU na janela do console.

1. No painel **Explorer**, navegue até a pasta **23-index-optimization**.

1. Abra o menu de contexto da pasta **23-index-optimization** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido para a pasta **23-index-optimization**.

1. Compile o projeto usando o comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; É possível que você veja um aviso do compilador informando que as variáveis **ponto de extremidade** e **chave** estão atualmente sem uso. Você pode ignorar esse aviso com segurança, pois usará essas variáveis nesta tarefa.

1. Feche o terminal integrado.

1. Abra o arquivo de código **script.cs**.

1. Localize a variável **cadeia de caracteres** denominada **ponto de extremidade**. Defina seu valor como **ponto de extremidade** da conta do Azure Cosmos DB que você criou anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se seu ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/**, então a instrução C# seria: **ponto de extremidade da cadeia de caracteres = "https&shy;://dp420.documents.azure.com:443/";**.

1. Localize a variável **cadeia de caracteres** denominada **chave**. Defina seu valor como **chave** da conta do Azure Cosmos DB que você criou anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se sua chave for: **fDR2ci9QgkdkvERTQ==**, então a instrução C# seria: **chave da cadeia de caracteres = "fDR2ci9QgkdkvERTQ==";**.

1. **Salve** o arquivo de código **script.cs**.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **23-index-optimization** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Compile e execute o projeto usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe a saída do terminal. O identificador exclusivo do item e a taxa de solicitação da operação (em RUs) devem ser impressos no console.

1. Compile e execute o projeto pelo menos mais duas vezes usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observe a carga de RUs na saída do console:

    ```
    dotnet run
    ```

1. Deixe o terminal integrado aberto.

    > &#128221; Você reutilizará esse terminal mais tarde neste exercício. É importante deixar o terminal aberto para que você possa comparar as taxas de RU originais e atualizadas.

## Atualize a política de indexação e execute novamente o aplicativo .NET

Este cenário de laboratório presumirá que nossas consultas futuras se concentrarão principalmente nas propriedades nome e categoryName. Para otimizar nosso grande item JSON, você excluirá todos os outros campos do índice criando uma política de indexação que começa excluindo todos os caminhos. Em seguida, a política incluirá seletivamente caminhos específicos.

1. Retorne ao seu navegador da Web.

1. No recurso da conta **Azure Cosmos DB**, navegue até o painel **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, expanda o nó do contêiner **produtos** e selecione **Configuração**.

1. Na guia **Configurações**, navegue até a seção **Política de indexação**.

1. Observe a política de indexação padrão:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. Substitua a política de indexação por esse objeto JSON modificado e, em seguida, **Salve** as alterações:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
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

1. Volte para o **Visual Studio Code**. Retorne ao terminal aberto.

1. Compile e execute o projeto pelo menos mais duas vezes usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observe a nova carga de RU na saída do console, que deve ser significativamente menor do que a carga original. Como você não está indexando todas as propriedades do item, o custo de gravação é significativamente menor ao atualizar o índice. No entanto, isso pode custar muito caro se as leituras precisarem consultar as propriedades que não estão indexadas.  

    ```
    dotnet run
    ```

    > &#128221; Se não estiver vendo uma cobrança de RU atualizada, talvez seja necessário aguardar alguns minutos.

1. Retorne ao seu navegador da Web.

    > &#128221; Se a página **Política de Indexação** não estiver aberta, vá para **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, expanda o nó do contêiner **produtos**, selecione **Configurações** e navegue até a seção **Política de Indexação**.

1. Substitua a política de indexação por esse objeto JSON modificado e, em seguida, **Salve** as alterações:

    ```
    {
      "indexingMode": "none"
    }
    ```

1. Feche a janela ou a guia do navegador da Web.

1. Volte para o **Visual Studio Code**. Retorne ao terminal aberto.

1. Compile e execute o projeto pelo menos mais duas vezes usando o comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observe a nova carga de RU na saída do console, que deve ser muito menor do que a carga original.  Como isso pode acontecer? Como esse script está medindo as RUs quando você escreve o item, ao optar por não ter índice, não há sobrecarga para manter esse índice. O outro lado disso é que, embora suas gravações gerem menos RUs, suas leituras serão muito caras.

    ```
    dotnet run
    ```

    > &#128221; Se não estiver vendo uma cobrança de RU atualizada, talvez seja necessário aguardar alguns minutos.

1. Fechar o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
