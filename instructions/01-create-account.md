---
lab:
  title: Criar uma conta do Azure Cosmos DB for NoSQL
  module: Module 1 - Get started with Azure Cosmos DB for NoSQL
---

# Criar uma conta do Azure Cosmos DB for NoSQL

Antes de se aprofundar no Azure Cosmos DB, é importante saber os conceitos básicos da criação dos recursos que você mais usará. Na maioria dos cenários, você precisará ter facilidade para criar contas, bancos de dados, contêineres e itens. Em um cenário real, você também deve ter algumas consultas básicas “em mãos” para testar se criou todos os seus recursos corretamente.

Neste laboratório, você criará uma nova conta do Azure Cosmos DB usando o NoSQL. Em seguida, você usará o Data Explorer para criar um banco de dados, um contêiner e dois itens. Por fim, você consultará o banco de dados em busca dos itens criados.

## Criar uma conta do Azure Cosmos DB

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você selecionará a qual das APIs deseja que a conta dê suporte (por exemplo, **API Mongo** ou **API NoSQL**).

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Na categoria **Serviços do Azure**, selecione **Criar um recurso** e **Azure Cosmos DB**.

    > &#128161; Como alternativa, expanda o menu **&#8801;**, selecione **Todos os Serviços**. Na categoria **Bancos de dados**, selecione **Azure Cosmos DB** e, em seguida, **Criar**.

1. No painel **Selecionar opção de API**, escolha a opção **Criar** na seção **Azure Cosmos DB for NoSQL**.

1. No painel **Criar Conta do Azure Cosmos DB**, observe a guia **Informações Básicas**.

1. Na guia **Básico**, insira os seguintes valores para cada configuração:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **Assinatura** | **Use a assinatura do Azure que você já tem.** *Todos os recursos precisam pertencer a um grupo de recursos. Cada grupo de recursos precisa pertencer a uma assinatura.* |
    | **Grupo de Recursos** | **Use um grupo de recursos existente ou crie um grupo de recursos.** *Todos os recursos precisam pertencer a um grupo de recursos.* |
    | **Account Name** | **Insira qualquer nome globalmente exclusivo.** *O nome da conta globalmente exclusivo. Esse nome será usado como parte do endereço DNS para solicitações.  O portal verificará o nome em tempo real.* |
    | **Localidade** | **Escolha qualquer região disponível.** *Selecione a região geográfica na qual seu banco de dados será hospedado inicialmente.* |
    | **Modo de capacidade** | **Taxa de transferência provisionada** |
    | **Aplicar Desconto na Camada Gratuita** | **Não Aplicar** |

1. Selecione **Examinar + Criar** para navegar até a guia **Examinar + Criar** e selecione **Criar**.

    > &#128221; A conta do Azure Cosmos DB for NoSQL pode demorar 10 a 15 minutos para ficar pronta para uso.

1. Observe o painel **Implantação**. Quando a implantação for concluída, o painel será atualizado com uma mensagem **Implantação bem-sucedida**.

1. Ainda no painel **Implantação**, selecione **Ir para o recurso**.

## Usar o Data Explorer para criar um banco de dados e um contêiner

O Data Explorer será sua principal ferramenta para gerenciar o banco de dados e os contêineres do Azure Cosmos DB para NoSQL no portal do Azure. Você criará um banco de dados básico e um contêiner para usar neste laboratório.

1. No painel **Conta do Azure Cosmos DB**, selecione **Data Explorer** no menu de recursos.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *cosmicworks* |
    | **Compartilhar taxa de transferência entre contêineres** | *Desmarcado* |
    | **ID do contêiner** | *products* |
    | **Chave de partição** | */categoryId* |
    | **Taxa de transferência do contêiner (dimensionamento automático)** | *Manual* |
    | **RU/s** | *400* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner **products** na hierarquia.

## Usar o Data Explorer para criar novos itens

O Data Explorer também inclui um conjunto de recursos para consultar, criar e gerenciar itens em um contêiner do Azure Cosmos DB for NoSQL. Você criará dois itens básicos usando um JSON bruto no Data Explorer.

1. No painel do **Data Explorer**, expanda o nó do banco de dados **cosmicworks**, expanda o nó do contêiner **products** e selecione **Itens**.

1. Selecione **Novo Item** na barra de comandos e, no editor, substitua o item de JSON do espaço reservado pelo seguinte conteúdo:

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Escolha **Salvar** na barra de comandos para adicionar o primeiro item de JSON:

1. De volta à guia **Itens**, selecione **Novo Item** na barra de comandos. No editor, substitua o item de JSON do espaço reservado pelo seguinte conteúdo:

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. Selecione **Salvar** na barra de comandos para adicionar o segundo item de JSON:

1. Na guia **Itens**, observe os dois novos itens no painel **Itens**.

## Usar o Data Explorer para emitir uma consulta básica

Por fim, o Data Explorer tem um editor de consultas interno que é usado para emitir consultas, observar os resultados e medir o impacto em termos de RU/s (unidades de solicitação por segundo).

1. No painel do **Data Explorer**, selecione **Nova Consulta SQL**.

1. Na guia Consulta, selecione **Executar Consulta** para exibir uma consulta padrão que seleciona todos os itens sem nenhum filtro.

1. Exclua o conteúdo da área do editor.

1. Substitua a consulta de espaço reservado pelo seguinte conteúdo:

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; Essa consulta selecionará todos os itens em que **price** é maior que US$ 500.

1. Selecione **Executar Consulta**.

1. Observe os resultados da consulta, que deve incluir um só item de JSON e todas as propriedades.

1. Na guia **Consulta**, selecione **Estatísticas de Consulta**.

1. Ainda na guia **Consulta**, observe o valor do campo **Solicitação de Preço** na seção **Estatísticas de Consulta**.

    > &#128221; Normalmente, a solicitação de preço para essa consulta simples é entre 2 e 3 RU/s quando o tamanho do contêiner é pequeno.

1. Feche a janela ou a guia do navegador da Web.
