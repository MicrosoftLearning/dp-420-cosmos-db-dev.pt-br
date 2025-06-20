---
lab:
  title: Otimizar a política de indexação de um contêiner do Azure Cosmos DB for NoSQL para uma consulta específica
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Otimizar a política de indexação de um contêiner do Azure Cosmos DB for NoSQL para uma consulta

Ao planejar uma conta do Azure Cosmos DB for NoSQL, conhecer nossas consultas mais populares pode nos ajudar a ajustar a política de indexação para que as consultas tenham o melhor desempenho possível.

Neste laboratório, usaremos o Data Explorer para testar consultas SQL com a política de indexação padrão e uma política de indexação que inclui um índice composto.

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

1. Aguarde a conclusão da tarefa de implantação antes de continuar esta tarefa.

1. Acesse o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel do **Data Explorer**.

1. No painel do **Data Explorer**, expanda **Novo contêiner** e, a seguir, selecione **Novo Banco de Dados**.

1. No pop-up de **Novo Banco de Dados**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *``cosmicworks``* |
    | **Taxa de transferência de provisionamento** | Habilitado |
    | **Taxa de transferência do banco de dados** | **Manual** |
    | **RU/s necessárias para o banco de dados** | ``1000`` |

1. De volta ao painel do **Data Explorer**, observe o nó de banco de dados do **cosmicworks** dentro da hierarquia.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *``products``* |
    | **Chave de partição** | *``/category/name``* |

1. De volta ao painel **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó do contêiner **produtos** na hierarquia.

1. Na folha de recursos, navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo**PRIMARY CONNECTION STRING**. Você usará esse valor de **cadeia de conexão** posteriormente nesse exercício.

1. Abra o **Visual Studio Code**.

## Propagar sua conta do Azure Cosmos DB for NoSQL com amostras de dados

Você usará um utilitário de linha de comando que cria um banco de dados do **cosmicworks** e um contêiner de **produtos**. Em seguida, a ferramenta criará um conjunto de itens que você irá observar usando o processador de feed de alterações em execução na janela do seu terminal.

1. No **Visual Studio Code**, abra o menu ** Terminal** e, a seguir, selecione **Novo Terminal** para abrir um novo terminal.

1. Instale a ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] para uso global em seu computador.

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; Esse comando poderá demorar alguns minutos para ser concluído. Esse comando irá gerar a mensagem de aviso (*A ferramenta "cosmicworks" já está instalada) se você já tiver instalado a versão mais recente dessa ferramenta anteriormente.

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

1. Feche o **Visual Studio Code** e retorne ao navegador.

## Executar consultas SQL e medir o preço da unidade de solicitação

Antes de modificar a política de indexação, primeiro você executará algumas consultas SQL de amostra para obter um preço de unidade de solicitação de linha de base expresso em RUs

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, selecione o nó de contêiner **produtos** e selecione **Nova consulta SQL**.

1. Selecione **Executar Consulta** para executar a consulta padrão:

    ```
    SELECT * FROM c
    ```

1. Observe os resultados da consulta. Selecione **Estatísticas de Consulta** para exibir o preço da unidade de solicitação em RUs.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará os três valores de todos os documentos:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p    
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados e estatísticas da consulta. O preço da unidade de solicitação é quase o mesmo que a primeira consulta.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará três valores de todos os documentos ordenados por **categoryName**:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados e estatísticas da consulta. O preço da unidade de solicitação aumentou devido à cláusula **ORDER BY**.

## Criar um índice composto na política de indexação

Agora, você precisará criar um índice composto se classificar os itens usando várias propriedades. Nesta tarefa, você criará um índice composto para classificar itens por seu categoryName e, em seguida, seu nome real.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, selecione o nó de contêiner **produtos** e selecione **Nova consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que ordenará os resultados pela **categoria** em ordem decrescente inicialmente e, em seguida, pelo **preço** em ordem crescente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.price ASC
    ```

1. Selecione **Executar Consulta**.

1. A consulta deve falhar com o erro **A ordem por consulta não tem um índice composto correspondente que possa ser atendido de**.

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
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, selecione o nó de contêiner **produtos** e selecione **Nova consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que ordenará os resultados pelo **categoryName** em ordem decrescente primeiro e, em seguida, pelo **preço** em ordem crescente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.price ASC
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados e estatísticas da consulta. Desta vez, como a consulta foi concluída, você pode revisar novamente o preço de RUs.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que ordenará os resultados pela **categoria** em ordem decrescente primeiro, depois por **nome** em ordem crescente e, por fim, pelo **preço** em ordem crescente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. Selecione **Executar Consulta**.

1. A consulta deve falhar com o erro **A ordem por consulta não tem um índice composto correspondente que possa ser atendido de**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, expanda o nó do contêiner **produtos** e selecione **Configurações** novamente.

1. Na guia **Configurações**, navegue até a seção **Política de indexação**.

1. Substitua a política de indexação por esse objeto JSON modificado e, em seguida, **Salve** as alterações:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, selecione o nó de contêiner **produtos** e selecione **Nova consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que ordenará os resultados pela **categoria** em ordem decrescente primeiro, depois por **nome** em ordem crescente e, por fim, pelo **preço** em ordem crescente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados e estatísticas da consulta. Desta vez, como a consulta foi concluída, você pode revisar novamente o preço de RUs.

1. Feche a guia ou a janela do navegador da web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
