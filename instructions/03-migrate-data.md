---
lab:
  title: Migrar dados existentes usando o Azure Data Factory
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Migrar dados existentes usando o Azure Data Factory

No Azure Data Factory, o Azure Cosmos DB tem suporte como uma fonte de ingestão de dados e como um destino (coletor) de saída de dados.

Neste laboratório, preencheremos o Azure Cosmos DB usando um utilitário de linha de comando útil e, em seguida, usaremos o Azure Data Factory para mover um subconjunto de dados de um contêiner para outro.

## Criar e propagar sua conta do Azure Cosmos DB for NoSQL

Você usará um utilitário de linha de comando que cria um banco de dados **cosmicworks** e um contêiner de **produtos** de **4.000** unidades de solicitação por segundo (RU/s). Depois de criado, você ajustará a taxa de transferência para 400 RU/s.

Para acompanhar o contêiner de produtos, você criará manualmente um contêiner **flatproducts** que será o destino da transformação ETL e da operação de carga no final deste laboratório.

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
    | **Limitar a quantidade total de taxa de transferência que pode ser provisionada nesta conta** | *Desmarcado* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impedem a criação de um novo grupo de recursos. Se esse for o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar esta tarefa.

1. Acesse o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Este painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará este valor de **ponto de extremidade** posteriormente neste exercício.

    1. Observe o campo **PRIMARY KEY**. Você usará este valor de **chave** posteriormente neste exercício.

1. Mantenha a guia do navegador aberta, pois retornaremos a ela mais tarde.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não estiver familiarizado com a interface do Visual Studio Code, examine o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. No **Visual Studio Code**, abra o menu **Terminal** e selecione **Novo Terminal** para abrir uma nova instância de terminal.

1. Instale a ferramenta de linha de comando [cosmicworks][nuget.org/packages/cosmicworks] para uso global em seu computador.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Este comando pode levar alguns minutos para ser concluído. Este comando gerará a mensagem de aviso (*A ferramenta "cosmicworks" já está instalada") se você já tiver instalado a versão mais recente dessa ferramenta no passado.

1. Execute o cosmicworks para propagar sua conta do Azure Cosmos DB com as seguintes opções de linha de comando:

    | **Opção** | **Valor** |
    | ---: | :--- |
    | **--endpoint** | *O valor do ponto de extremidade que você verificou anteriormente neste laboratório* |
    | **--key** | *O valor da chave que você verificou anteriormente neste laboratório* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for: **https&shy;://dp420.documents.azure.com:443/** e a sua chave for: **fDR2ci9QgkdkvERTQ==**, o comando será: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Aguarde até que o comando **cosmicworks** termine de preencher a conta com um banco de dados, um contêiner e itens.

1. Feche o terminal integrado.

1. Volte para o navegador da Web, abra uma nova guia e navegue até o portal do Azure (``portal.azure.com``).

1. Selecione **Grupos de recursos**, selecione o grupo de recursos que você criou ou exibiu anteriormente neste laboratório e selecione o recurso de **conta do Azure Cosmos DB** que você criou neste laboratório.

1. No recurso de conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, expanda o nó de contêiner de **produtos** e selecione **Itens**.

1. Observe e selecione os vários itens JSON no contêiner de **produtos**. Esses são os itens criados pela ferramenta de linha de comando usada nas etapas anteriores.

1. Selecione o nó **Escala e configurações**. Na guia **Escala e configurações**, selecione **Manual**, atualize a configuração de **taxa de transferência necessária**56 de **4000 RU/s** para **400 RU/s** e **salve** suas alterações**.

1. No painel Do **Data Explorer**, selecione **Novo contêiner**.

1. No item pop-up **Novo contêiner**, insira os seguintes valores para cada configuração e selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *`flatproducts`* |
    | **Chave de partição** | *`/category`* |
    | **Taxa de transferência do contêiner (escala automática)** | *Manual* |
    | **RU/s** | *`400`* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner **flatproducts** dentro da hierarquia.

1. Retorne à **Página Inicial** do portal do Azure.

## Criar recurso do Azure Data Factory

Agora que os recursos do Azure Cosmos DB for NoSQL estão em vigor, você criará um recurso do Azure Data Factory e configurará todos os componentes e conexões necessários para executar uma movimentação de dados única de um contêiner de API NoSQL para outro para extrair dados, transformá-los e carregá-los em outro contêiner de API NoSQL.

1. Selecione **+ Criar um recurso**, pesquise por *Data Factory* e crie um novo recurso do **Data Factory** com as seguintes configurações, deixando todas as configurações restantes em seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Nome** | *Insira um nome globalmente exclusivo* |
    | **Região** | *Escolha qualquer região disponível* |
    | **Versão** | *V2* |
    | **Configuração do Git** | *Configurar o Git mais tarde* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impedem a criação de um novo grupo de recursos. Se esse for o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar esta tarefa.

1. Vá para o recurso do **Data Factory** recém-criado e selecione **Iniciar estúdio**.

    > &#128161; Como alternativa, você pode navegar até (``adf.azure.com/home``), selecionar o recurso do Data Factory recém-criado e, em seguida, selecionar o ícone de casa.

1. Da tela inicial. Selecione a opção **Ingestão** para iniciar o assistente rápido para executar uma operação de cópia única na operação de escala e ir para a etapa **Propriedades** do assistente.

1. Começando com a etapa **Propriedades** do assistente, na seção **Tipo de tarefa**, selecione a **Tarefa de cópia interna**.

1. Na seção **Cadência da tarefa ou agendamento de tarefas**, selecione **Executar uma vez agora** e, em seguida, selecione **Avançar** para ir para a etapa **Origem** do assistente.

1. Na etapa **Origem** do assistente, na lista de **Tipos de origem**, selecione **Azure Cosmos DB for NoSQL**.

1. Na seção **Conexão**, selecione **+ Nova conexão**.

1. No item pop-up **Nova conexão (Azure Cosmos DB for NoSQL)**, configure a nova conexão com os seguintes valores e selecione **Criar**:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Nome** | *`CosmosSqlConn`* |
    | **Conectar-se por meio do runtime de integração** | *AutoResolveIntegrationRuntime* |
    | **Método de autenticação** | *Chave de conta* &vert; *Cadeia de conexão* |
    | **Método de seleção de conta** | *Na assinatura do Azure* |
    | **Assinatura do Azure** | *Sua assinatura existente do Azure* |
    | **Nome da conta do Azure Cosmos DB** | *Seu nome de conta existente do Azure Cosmos DB que você escolheu anteriormente neste laboratório* |
    | **Nome do banco de dados** | *cosmicworks* |

1. Na seção **Armazenamento de dados de origem**, na seção **Tabelas de origem**, selecione **Usar consulta**.

1. Na lista de **Nomes de tabela**, selecione **produtos**.

1. No editor de **Consultas**, exclua o conteúdo existente e insira a seguinte consulta:

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. Selecione **Pré-visualizar dados** para testar a validade da consulta. Selecione **Avançar** para ir para a etapa **Destino** do assistente.

1. Na etapa **Destino** do assistente, na lista de **Tipos de destino**, selecione **Azure Cosmos DB for NoSQL**.

1. Na lista **Conexão**, selecione **CosmosSqlConn**.

1. Na lista de **Destino**, selecione **flatproducts** e, em seguida, selecione **Avançar** para ir para a etapa **Configurações** do assistente.

1. Na etapa **Configurações** do assistente, no campo **Nome da tarefa**, insira **`FlattenAndMoveData`**.

1. Deixe todos os campos restantes para seus valores padrão em branco e selecione **Avançar** para ir para a etapa final do assistente.

1. Examine o **Resumo** das etapas selecionadas no assistente e selecione **Avançar**.

1. Observe as várias etapas na implantação. Quando a implantação for concluída, selecione **Concluir**.

1. Retorne à guia do navegador que tem sua **conta do Azure Cosmos DB** e navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, selecione o nó de contêiner **flatproducts** e selecione **Nova consulta SQL**.

1. Exclua o conteúdo da área do editor.

1. Crie uma nova consulta SQL que retornará todos os documentos em que o **nome** seja equivalente ao **Headset HL**:

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        flatproducts p
    WHERE
        p.name = 'HL Headset'
    ```

1. Selecione **Executar Consulta**.

1. Observe os resultados da consulta.

1. Feche a janela ou a guia do navegador da Web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
