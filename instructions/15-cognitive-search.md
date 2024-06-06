---
lab:
  title: Pesquisar dados usando a Pesquisa de IA do Azure e o Azure Cosmos DB for NoSQL
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Pesquisar dados usando a Pesquisa de IA do Azure e o Azure Cosmos DB for NoSQL

A Pesquisa de IA do Azure combina um mecanismo de pesquisa como serviço a uma integração profunda com os recursos de IA para enriquecer as informações no índice de pesquisa.

Nesse laboratório, você vai criar um índice da Pesquisa de IA do Azure que indexa dados automaticamente em um contêiner do Azure Cosmos DB for NoSQL e enriquece os dados usando a funcionalidade de Tradução dos Serviços Cognitivos do Azure.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você irá selecionar a qual API você quer que a conta dê suporte (por exemplo, a **API do Mongo** ou a **API do NoSQL**). Após a conta do Azure Cosmos DB for NoSQL ter terminado de provisionar, você poderá recuperar o ponto de extremidade e a chave e usá-los para se conectar à conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para .NET ou qualquer outro SDK de sua escolha.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associada à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Cosmos DB* e, em seguida, crie um novo recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Sem servidor* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

    1. Observe o campo **PRIMARY KEY**. Você usará esse valor de **chave** posteriormente nesse exercício.

    1. Observe o campo**PRIMARY CONNECTION STRING**. Você usará esse valor de **cadeia de conexão** posteriormente nesse exercício.

1. Selecione **Data Explorer** no menu de recursos.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Criar novo* &vert; *``cosmicworks``* |
    | **ID do contêiner** | *``products``* |
    | **Chave de partição** | *``/categoryId``* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados do **cosmicworks** e observe o nó do contêiner de **produtos** dentro da hierarquia.

## Propagar sua conta do Azure Cosmos DB for NoSQL com amostras de dados

Você usará um utilitário de linha de comando que cria um banco de dados **cosmicworks** e um contêiner de **produtos**.

1. No **Visual Studio Code**, abra o menu **Terminal** e selecione **Novo Terminal**.

1. Instale a ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] para uso global no seu computador.

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

## Criar um recurso do Azure AI Search

Antes de prosseguir nesse exercício, você precisa primeiro criar uma nova instância da Pesquisa de IA do Azure.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associada à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Pesquisa de IA* e, a seguir, crie um novo recurso de conta da **Pesquisa de IA do Azure** com as configurações a seguir, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Nome** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Acesse o recurso de conta da **Pesquisa de IA do Azure** recém-criado .

## Compilar o indexador e o índice dos dados do Azure Cosmos DB for NoSQL

Você criará um indexador que indexa um subconjunto de dados em um contêiner específico do Azure Cosmos DB for NoSQL a cada hora.

1. Na folha de recursos da **Pesquisa de IA**, selecione **Importar dados**.

1. Na etapa **Conectar-se aos seus dados** do assistente de **Importação de dados**, na lista **Fonte de Dados**, selecione **Azure Cosmos DB**.

1. Defina a fonte de dados com as configurações a seguir, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Nome da fonte de dados** | *``products-cosmossql-source``* |
    | **Cadeia de conexão** | ***cadeia de conexão** da conta do Azure Cosmos DB for NoSQL criada anteriormente* |
    | **Backup de banco de dados** | *cosmicworks* |
    | **Coleção** | *produtos* |

1. No campo **consulta**, insira a seguinte consulta SQL para criar uma exibição materializada de um subconjunto de seus dados no contêiner:

    ```sql
    SELECT 
        p.id, 
        p.categoryId, 
        p.name, 
        p.price,
        p._ts
    FROM 
        products p 
    WHERE 
        p._ts > @HighWaterMark 
    ORDER BY 
        p._ts
    ```

1. Selecione a caixa de seleção **Resultados da consulta ordenados por _ts**.

    > &#128221; Essa caixa de seleção permite que a Pesquisa de IA do Azure saiba que a consulta ordena os resultados pelo campo **_ts**. Esse tipo de ordenação permite o acompanhamento incremental do progresso. Se falhar, o indexador poderá recomeçar novamente do mesmo valor de **_ts**, já que os resultados são ordenados pelo carimbo de data/hora.

1. Selecione **Avançar: Adicionar habilidades cognitivas**.

1. Selecione **Passar para: Personalizar índice de destino**.

1. Na etapa **Personalizar índice de destino** do assistente, defina o índice com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Nome do índice** | *``products-index``* |
    | **Chave** | *id* |

1. Na tabela de campos, configure as opções **Recuperáveis**, **Filtráveis**, **Classificáveis**, **Facetáveis** e **Pesquisáveis** para cada campo usando a tabela a seguir:

    | **Campo** | **Recuperável** | **Filtrável** | **Classificável** | **Com faceta** | **Pesquisável** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | **categoryId** | &#10004; | &#10004; | &#10004; | &#10004; | |
    | **name** | &#10004; | &#10004; | &#10004; | | &#10004; (Inglês — Microsoft) |
    | **price** | &#10004; | &#10004; | &#10004; | &#10004; | |

1. Selecione **Próximo: Criar um indexador**.

1. Na etapa **Criar um indexador** do assistente, defina o indexador com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Nome** | *``products-cosmosdb-indexer``* |
    | **Agendar** | *A cada hora* |

1. Escolha **Enviar** para criar a fonte de dados, o índice e o indexador.

    > &#128221; Você talvez seja solicitado a ignorar um pop-up de pesquisa após criar seu primeiro indexador.

1. Na folha de recursos da **Pesquisa de IA**, navegue até a guia ** Indexadores** para observar o resultado da sua primeira operação de indexação.

1. Aguarde até que o indexador **products-cosmosdb-indexer** apresente o status de **Sucesso** antes de prosseguir com essa tarefa.

    > &#128221; Você talvez precise usar a opção **Atualizar** para atualizar a folha se ela não atualizar automaticamente.

1. Navegue até a guia **Índices** e, a seguir, selecione o índice **products-index**.

## Validar o índice com exemplos de consultas de pesquisa

Agora que sua exibição materializada dos dados do Azure Cosmos DB for NoSQL está no índice de pesquisa, você pode executar algumas consultas básicas que aproveitam os recursos da Pesquisa de IA do Azure.

> &#128221; Esse laboratório não se destina a ensinar a sintaxe da Pesquisa de IA do Azure. Essas consultas foram selecionadas para mostrar alguns dos recursos disponíveis no mecanismo e no índice de pesquisa.

1. Na guia **Gerenciador de pesquisa**, selecione o menu flutuante **Exibição** e, em seguida, selecione o **modo de exibição JSON**.

1. Observe, no **editor de consultas JSON**, a sintaxe da consulta de pesquisa JSON padrão que retorna todos os resultados possíveis usando um operador **\*** (curinga).

   ```json
   {
       "search": "*"
   }
   ```

1. Selecione o botão **Pesquisar** para efetuar a pesquisa.

1. Observe que essa consulta de pesquisa retorna todos os resultados possíveis.

1. No **editor de consultas JSON**, insira a consulta a seguir e selecione **Pesquisar**:

    ```json
    {
        "search": "touring 3000"
    }
    ```

1. Observe que essa consulta de pesquisa retorna resultados que contêm os termos **touring** ou **3000**, atribuindo uma pontuação mais alta aos resultados que contêm ambos os termos. Os resultados são, a seguir, classificados em ordem decrescente pelo campo **@search.score**.

1. No **editor de consultas JSON**, insira a consulta a seguir e selecione **Pesquisar**:

    ```json
    {
        "search": "red"
        , "count": true
    }
    ```

1. Observe que essa consulta de pesquisa retorna resultados com o termo **vermelho**, mas agora também inclui um campo de metadados que indica o número total de resultados, mesmo que não estejam todos incluídos na mesma página.

1. No **editor de consultas JSON**, insira a consulta a seguir e selecione **Pesquisar**:

    ```json
    {
        "search": "blue"
        , "count": true
        , "top": 6
    }
    ```

1. Observe que essa consulta de pesquisa retorna apenas um conjunto de seis resultados de cada vez, embora existam mais correspondências no lado do servidor.

1. No **editor de consultas JSON**, insira a consulta a seguir e selecione **Pesquisar**:

    ```json
    {
        "search": "mountain"
        , "count": true
        , "top": 25
        , "skip": 50
    }
    ```

1. Observe que essa consulta de pesquisa ignora os primeiros 50 resultados e retorna um conjunto de 25 resultados. Se essa fosse uma exibição paginada em um aplicativo no lado do cliente, você poderia inferir que essa seria a terceira "página" de resultados.

1. No **editor de consultas JSON**, insira a consulta a seguir e selecione **Pesquisar**:

    ```json
    {
        "search": "touring"
        , "count": true
        , "filter": "price lt 500"
    }
    ```

1. Observe que essa consulta de pesquisa retorna apenas os resultados em que o valor do campo numérico de preço é inferior a 500.

1. No **editor de consultas JSON**, insira a consulta a seguir e selecione **Pesquisar**:

    ```json
    {
        "search": "road"
        , "count": true
        , "top": 15
        , "facets": ["price,interval:500"]
    }
    ```

1. Observe que essa consulta de pesquisa retorna uma coleção de dados de facetas que indica quantos itens pertencem a cada categoria, mesmo que não estejam todos presentes na página de resultados atual. Nesse exemplo, os itens correspondentes são divididos em categorias numéricas de preço em intervalos de 500. Isso costuma ser usado para preencher filtros e auxiliares de navegação em aplicativos no lado do cliente.

1. Feche a guia ou a janela do navegador da web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
