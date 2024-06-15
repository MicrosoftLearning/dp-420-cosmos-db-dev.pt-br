---
lab:
  title: Configurar a taxa de transferência para o Azure Cosmos DB for NoSQL com o portal do Azure
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Configurar a taxa de transferência para o Azure Cosmos DB for NoSQL com o portal do Azure

Uma das coisas mais importantes a serem feitas é configurar a taxa de transferência no Azure Cosmos DB for NoSQL. Para criar um contêiner do Azure Cosmos DB for NoSQL, primeiro você deve criar uma conta e, em seguida, um banco de dados, nesta ordem.

Neste laboratório, você provisionará a taxa de transferência usando vários métodos no Data Explorer. Você provisionará a taxa de transferência manualmente ou usando a escala automática, no banco de dados e no nível do contêiner.

## Criar uma conta sem servidor

Vamos começar de forma simples, criando uma conta sem servidor. Não há muito o que configurar aqui, pois é tudo sem servidor. Quando criamos nosso banco de dados e contêiner, não precisamos provisionar a taxa de transferência. Você verá tudo isso à medida que passarmos para a criação da conta.

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
    | **Grupo de Recursos** | **Use um grupo de recursos existente ou crie um grupo de recursos.** *Todos os recursos precisam pertencer a um grupo de recursos.*|
    | **Account Name** |  **Insira qualquer nome globalmente exclusivo.** *O nome da conta globalmente exclusivo. Esse nome será usado como parte do endereço DNS para solicitações.  O portal verificará o nome em tempo real.* |
    | **Localidade** | **Escolha qualquer região disponível.** *Selecione a região geográfica na qual seu banco de dados será hospedado inicialmente.* |
    | **Modo de capacidade** | **Selecionar sem servidor** |

1. Selecione **Examinar + Criar** para navegar até a guia **Examinar + Criar** e selecione **Criar**.

    > &#128221; A conta do Azure Cosmos DB for NoSQL pode demorar 10 a 15 minutos para ficar pronta para uso.

1. Observe o painel **Implantação**. Quando a implantação for concluída, o painel será atualizado com uma mensagem **Implantação bem-sucedida**.

1. Ainda no painel **Implantação**, selecione **Ir para o recurso**.

1. No painel de conta do **Azure Cosmos DB**, selecione **Data Explorer** no menu de recursos.

1. No painel do **Data Explorer**, expanda **Novo contêiner** e selecione **Novo banco de dados**.

1. No pop-up de **Novo Banco de Dados**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *`cosmicworks`* |

1. De volta ao painel do **Data Explorer**, observe o nó de banco de dados do **cosmicworks** dentro da hierarquia.

1. No painel do **Data Explorer**, selecione **Novo Contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *cosmicworks* |
    | **ID do contêiner** | *`products`* |
    | **Chave de partição** | *`/categoryId`* |

1. De volta ao painel do **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de **produtos** dentro da hierarquia.

1. Retorne à **Página Inicial** do portal do Azure.

## Criar uma conta provisionada

Agora, vamos criar uma conta de taxa de transferência provisionada com opções de configuração mais tradicionais. Este tipo de conta abrirá um mundo de opções de configuração para nós, o que pode ser um pouco avassalador. Vamos percorrer alguns exemplos de emparelhamentos de banco de dados e contêineres que são possíveis aqui.

1. Na categoria de **serviços do Azure**, selecione **Criar um recurso** e selecione **Azure Cosmos DB**.

    > &#128161; Como alternativa, expanda o menu **&#8801;**, selecione **Todos os Serviços**. Na categoria **Bancos de dados**, selecione **Azure Cosmos DB** e, em seguida, **Criar**.

1. No painel **Selecionar opção de API**, escolha a opção **Criar** na seção **Azure Cosmos DB for NoSQL**.

1. No painel **Criar Conta do Azure Cosmos DB**, observe a guia **Informações Básicas**.

1. Na guia **Básico**, insira os seguintes valores para cada configuração:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **Assinatura** | **Use a assinatura do Azure que você já tem.** *Todos os recursos precisam pertencer a um grupo de recursos. Cada grupo de recursos precisa pertencer a uma assinatura.* |
    | **Grupo de Recursos** | **Use um grupo de recursos existente ou crie um grupo de recursos.** *Todos os recursos precisam pertencer a um grupo de recursos.*|
    | **Account Name** |  **Insira qualquer nome globalmente exclusivo.** *O nome da conta globalmente exclusivo. Esse nome será usado como parte do endereço DNS para solicitações.  O portal verificará o nome em tempo real.* |
    | **Localidade** | **Escolha qualquer região disponível.** *Selecione a região geográfica na qual seu banco de dados será hospedado inicialmente.* |
    | **Modo de capacidade** | **Taxa de transferência provisionada** |
    | **Aplicar Desconto na Camada Gratuita** | **Não aplicar** |
    | **Limitar a quantidade total de taxa de transferência que pode ser provisionada nesta conta** | **Desmarcado** |

1. Selecione **Examinar + Criar** para navegar até a guia **Examinar + Criar** e selecione **Criar**.

    > &#128221; A conta do Azure Cosmos DB for NoSQL pode demorar 10 a 15 minutos para ficar pronta para uso.

1. Observe o painel **Implantação**. Quando a implantação for concluída, o painel será atualizado com uma mensagem **Implantação bem-sucedida**.

1. Ainda no painel **Implantação**, selecione **Ir para o recurso**.

1. No painel de conta do **Azure Cosmos DB**, selecione **Data Explorer** no menu de recursos.

1. No painel do **Data Explorer**, expanda **Novo contêiner** e selecione **Novo banco de dados**.

1. No pop-up de **Novo Banco de Dados**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *`nothroughputdb`* |
    | **Taxa de transferência de provisionamento** | *Desmarcado* |

1. De volta ao painel do **Data Explorer**, observe o nó de banco de dados **nothroughputdb** dentro da hierarquia.

1. No painel Do **Data Explorer**, selecione **Novo contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *nothroughputdb* |
    | **ID do contêiner** | *`requiredthroughputcontainer`* |
    | **Chave de partição** | *`/primarykey`* |
    | **Taxa de transferência do contêiner** | *Manual* |
    | **RU/s** | *`400`* |

1. De volta ao painel do **Data Explorer**, expanda o nó de banco de dados **nothroughputdb** e observe o nó de contêiner **requiredthroughputcontainer** dentro da hierarquia.

1. No painel do **Data Explorer**, expanda **Novo contêiner** e selecione **Novo banco de dados**.

1. No pop-up de **Novo Banco de Dados**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *`manualthroughputdb`* |
    | **Taxa de transferência de provisionamento** | *Verificado* |
    | **Taxa de transferência do banco de dados** | *Manual* |
    | **RU/s** | *`400`* |

1. De volta ao painel do **Data Explorer**, observe o nó de banco de dados **manualthroughputdb** dentro da hierarquia.

1. No painel Do **Data Explorer**, selecione **Novo contêiner**.

1. No pop-up **Novo Contêiner**, insira os seguintes valores para cada configuração e, a seguir, selecione **OK**:

    | **Configuração** | **Valor** |
    | --: | :-- |
    | **ID do banco de dados** | *Usar existente* &vert; *manualthroughputdb* |
    | **ID do contêiner** | *`childcontainer`* |
    | **Chave de partição** | *`/primarykey`* |
    | **Provisionar taxa de transferência dedicada para este contêiner** | *Verificado* |
    | **Taxa de transferência do contêiner** | *Manual* |
    | **RU/s** | *`1000`* |

1. De volta ao painel do **Data Explorer**, expanda o nó de banco de dados **manualthroughputdb** e observe o nó de contêiner **childcontainer** dentro da hierarquia.
