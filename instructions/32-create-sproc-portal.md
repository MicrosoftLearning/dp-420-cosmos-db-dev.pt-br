---
lab:
  title: Criar um procedimento armazenado com o portal do Azure
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Criar um procedimento armazenado com o portal do Azure

Os procedimentos armazenados são uma das maneiras pelas quais você pode executar o lado do servidor lógico de negócios no Azure Cosmos DB. Com um procedimento armazenado, você pode executar operações CRUD (Criar, Ler, Atualizar, Excluir) básicas com um contêiner em vários documentos dentro de um único escopo transacional.

Neste laboratório, você criará um procedimento armazenado que cria um documento em seu contêiner. Em seguida, você usará uma consulta SQL para validar os resultados do procedimento armazenado.

## Criar um procedimento armazenado

Os procedimentos armazenados são criados em JavaScript integrado à linguagem e dão suporte à execução de operações CRUD básicas dentro do mecanismo de banco de dados. O JavaScript em execução no mecanismo de banco de dados é possibilitado usando o SDK JavaScript do lado do servidor para o Azure Cosmos DB e uma série de métodos auxiliares.

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Azure (``portal.azure.com``).

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

1. Aguarde a conclusão da tarefa de implantação antes de continuar esta tarefa.

1. Acesse o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, selecione **Novo contêiner** e crie um novo contêiner com as seguintes configurações, deixando todas as configurações restantes em seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **ID do banco de dados** | *Criar novo* &vert; *``cosmicworks``* |
    | **Compartilhar taxa de transferência entre contêineres** | *Selecionar esta opção* |
    | **Taxa de transferência do banco de dados** | *Manual* &vert; *400* |
    | **ID do contêiner** | *``products``* |
    | **Indexação** | *Automático* |
    | **Chave de partição** | *``/categoryId``* |

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e selecione o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione **Novo procedimento armazenado**.

1. No campo **ID do procedimento armazenado**, insira o valor **createDoc**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova função JavaScript chamada **createDoc** sem parâmetros de entrada:

    ```
    function createDoc() {
        
    }
    ```

1. Dentro da função **createDoc**, invoque o método [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] interno e armazene o resultado em uma variável chamada **context**:

    ```
    var context = getContext();
    ```

1. Invoque o método [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] do objeto de contexto e armazene o resultado em uma variável nomeada **container**:

    ```
    var container = context.getCollection();
    ```

1. Crie um novo objeto chamado **doc** com duas propriedades:

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *primeiro documento* |
    | **ID da categoria** | *demo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. Invoque o método **createDocument** do objeto de contêiner que passa o resultado da invocação do método **getSelfLink** do objeto de contêiner e do novo documento como parâmetros:

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. Depois de terminar, o código de procedimento armazenado agora deve incluir:

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. Selecione **Salvar** para persistir as alterações no procedimento armazenado.

1. Selecione **Executar** e execute o procedimento armazenado usando os seguintes parâmetros de entrada:

    | **Configuração** | **Chave** | **Valor** |
    | ---: | :--- | :--- |
    | **Valor da chave de partição** | *Cadeia de caracteres* | *demo* |

1. Observe o resultado vazio. Embora o procedimento armazenado tenha sido executado com sucesso, o código JavaScript nunca retornou uma resposta legível por humanos.

## Implementar práticas recomendadas para um procedimento armazenado

Embora o procedimento armazenado criado anteriormente neste laboratório tenha funcionalidade básica, também faltam algumas técnicas comuns de tratamento de erros que devem ser implementadas em todos os procedimentos armazenados. Primeiro, o procedimento armazenado pressupõe que sempre terá tempo para concluir a operação, e não verifica o valor retornado do método **createDocument** para garantir que haja tempo suficiente. Em segundo lugar, o procedimento armazenado pressupõe que todos os documentos sejam inseridos com êxito sem verificar ou gerar mensagens de erro em potencial. Por fim, o procedimento armazenado não retorna o documento recém-criado como a resposta HTTP para a solicitação que invocou originalmente o procedimento armazenado. Você fará essas três alterações no procedimento armazenado para implementar melhores práticas comuns.

1. Retorne ao editor para o procedimento armazenado **createDoc**.

1. Localize a linha 1 no código que define a função **createDoc**:

    ```
    function createDoc() {
    ```

    e atualize a linha de código para incluir um parâmetro chamado **título**:

    ```
    function createDoc(title) {
    ```

1. Localize a linha 5 no código que define a propriedade de **nome** do objeto **doc**:

    ```
    name: 'first document',
    ```

    e atualize a linha de código para usar o valor do parâmetro de **título**:

    ```
    name: title,
    ```

1. Localize a linha 8 no código que invoca o método **createDocument**:

    ```
    container.createDocument(
    ```

    e atualize a linha de código para armazenar o resultado da invocação do método em uma variável nomeada **accepted**

    ```
    var accepted = container.createDocument(
    ```

1. Adicione uma nova linha de código após a invocação do método **createDocument** para verificar o valor da variável **accepted** e retornar o método se ele não for verdadeiro:

    ```
    if (!accepted) return;
    ```

1. Por fim, adicione um terceiro parâmetro à invocação do método **createDocument** que é uma função que usa dois parâmetros chamados **error** e **newDoc**, verifica se o erro é nulo e, em seguida, define o newDoc para o corpo da resposta do procedimento armazenado:

    ```
    ,
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. Depois de terminar, o código de procedimento armazenado agora deve incluir:

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. Selecione **Atualizar** para persistir as alterações no procedimento armazenado.

1. Selecione **Executar** e execute o procedimento armazenado usando os seguintes parâmetros de entrada:

    | **Configuração** | **Chave** | **Valor** |
    | ---: | :--- | :--- |
    | **Valor da chave de partição** | *Cadeia de caracteres* | *demo* |
    | **Parâmetros de entrada** | *Cadeia de caracteres* | *segundo documento* |

1. Observe o resultado JSON. Depois que o procedimento armazenado foi executado com êxito, o documento recém-criado foi retornado como uma resposta para a solicitação HTTP original.

## Consultar documentos

Para concluir, você usará o Data Explorer para emitir uma consulta SQL que retornará os dois documentos criados neste laboratório.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e selecione o nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione **Nova Consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará todos os documentos em que **categoryId** é equivalente à **demonstração**:

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. Selecione **Executar Consulta**.

1. Observe os dois documentos que você criou neste laboratório como os resultados da execução dessa consulta.

1. Feche a janela ou a guia do navegador da Web.

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
