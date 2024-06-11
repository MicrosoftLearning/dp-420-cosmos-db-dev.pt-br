---
lab:
  title: Examine a política de índice padrão para um contêiner do Azure Cosmos DB for NoSQL com o portal
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Examine a política de índice padrão para um contêiner do Azure Cosmos DB for NoSQL com o portal

Cada contêiner no Azure Cosmos DB tem uma política de indexação que direciona o serviço sobre como indexar itens dentro do contêiner. Por padrão, esta política de indexação indexa cada propriedade de cada item. A política de indexação padrão facilita a introdução ao Azure Cosmos DB rapidamente, pois você não precisa pensar em indexação, desempenho e gerenciamento no início de um projeto.

Neste laboratório, você observará e manipulará a política de índice padrão para alguns contêineres usando o Data Explorer.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você selecionará a qual das APIs deseja que a conta dê suporte (por exemplo, **API Mongo** ou **API NoSQL**). Depois que o provisionamento da conta do Azure Cosmos DB for NoSQL for concluído, você poderá recuperar o ponto de extremidade e a chave e usá-los para se conectar à conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para .NET ou qualquer outro SDK de sua escolha.

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso **, pesquise por *Cosmos DB* e, em seguida, crie um recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes em seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Taxa de transferência provisionada* |
    | **Aplicar Desconto na Camada Gratuita** | *Não aplicar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impedem a criação de um novo grupo de recursos. Se esse for o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com esta tarefa.

1. Acesse o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Este painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará este valor de **ponto de extremidade** posteriormente neste exercício.

    1. Observe o campo **PRIMARY KEY**. Você usará este valor de **chave** posteriormente neste exercício.


## Propagar a conta do Azure Cosmos DB for NoSQL com dados

A ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] implanta dados de exemplo em qualquer conta do Azure Cosmos DB for NoSQL. A ferramenta é de software livre e está disponível por meio do NuGet. Você instalará esta ferramenta no Azure Cloud Shell e a usará para propagar o seu banco de dados.

1. Inicie o **Visual Studio Code**.

1. No **Visual Studio Code**, abra o menu **Terminal** e selecione **Novo Terminal** para abrir uma nova instância de terminal.

    > &#128221; Se você ainda não estiver familiarizado com a interface do Visual Studio Code, examine o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Instale a ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] para uso global em seu computador.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```
  
    > &#128161; Este comando pode levar alguns minutos para ser concluído. Este comando gerará a mensagem de aviso (*A ferramenta "cosmicworks" já está instalada") se você já tiver instalado a versão mais recente dessa ferramenta no passado.

1. Execute o cosmicworks para propagar a sua conta do Azure Cosmos DB com as seguintes opções de linha de comando:

    | **Opção** | **Valor** |
    | ---: | :--- |
    | **--endpoint** | *O valor do ponto de extremidade copiado anteriormente neste laboratório* |
    | **--key** | *O valor da chave que você copiou anteriormente neste laboratório* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/** e a sua chave for: **fDR2ci9QgkdkvERTQ==**, o comando será: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Aguarde até que o comando **cosmicworks** termine de preencher a conta com um banco de dados, um contêiner e itens.

1. Feche o terminal integrado.

## Exibir e manipular a política de indexação padrão

Quando um contêiner é criado por código, portal ou uma ferramenta; a política de indexação será definida como um padrão inteligente se você não especificá-la de outra forma. Você observará essa política de indexação padrão e fará uma alteração na política.

1. Retorne ao navegador da Web.

1. No recurso de conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de novos **produtos** dentro da árvore de navegação da **API NOSQL**.

1. Selecione o nó de contêiner de **produtos** na árvore de navegação da **API NoSQL** e selecione **Nova consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará todos os documentos em que o **nome** seja equivalente ao **Headset HL**:

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados da consulta.

1. Na guia **Consulta**, selecione **Estatísticas de consulta**.

1. Observe o valor do campo **Cobrança de solicitação** na seção **Estatísticas de consulta**.

    > &#128221; Todos os caminhos estão indexados no momento, portanto, esta consulta deve ser relativamente eficiente.

1. No nó de contêiner de **produtos** da árvore de navegação da **API NoSQL**, selecione **Escala e configurações**.

1. Observe a política de indexação padrão na seção **Política de indexação**:

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

    > &#128221; Esta política padrão indexará todos os caminhos possíveis com exceção de **_etag**.

1. No editor, substitua o conteúdo da política de indexação para indexar apenas o caminho **/price**:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. Selecione **Salvar** para manter as alterações.

1. Selecione **Nova Consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará todos os documentos em que o **nome** seja equivalente ao **Headset HL**:

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados da consulta.

1. Na guia **Consulta**, selecione **Estatísticas de consulta**.

1. Observe o valor do campo **Cobrança de solicitação** na seção **Estatísticas de consulta**.

    > &#128221; Agora que a propriedade **name** não está indexada, a cobrança da solicitação aumentou.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará todos os documentos cujo **preço** seja maior que **US$ 3.000**:

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados da consulta.

1. Na guia **Consulta**, selecione **Estatísticas de consulta**.

1. Observe o valor do campo **Cobrança de solicitação** na seção **Estatísticas de consulta**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
