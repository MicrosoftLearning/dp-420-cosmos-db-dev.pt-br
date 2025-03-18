---
lab:
  title: Configurar o Azure Cosmos DB
  module: Setup
---

# Configurar o Azure Cosmos DB

Neste exercício, você criará uma conta do Azure Cosmos DB for NoSQL que usará em todos os módulos de laboratório e concederá acesso à identidade do usuário para gerenciar dados na conta, atribuindo-a à função de **Colaborador de Dados Internos do Cosmos DB**. Isso permitirá que você use a autenticação do Azure para acessar o banco de dados a partir do código de laboratório e evitar a necessidade de armazenar e gerenciar chaves.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você irá selecionar a qual API você quer que a conta dê suporte. Quando o provisionamento da conta do Azure Cosmos DB for NoSQL estiver concluído, você poderá recuperar o ponto de extremidade e a chave e usá-los para se conectar à conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para Python ou qualquer outro SDK de sua escolha.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Cosmos DB* e, em seguida, crie um novo recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Sem servidor* |
    | **Aplicar Desconto na Camada Gratuita** | *Não Aplicar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Copie o campo **URI** e salve-o em um editor de texto para mais tarde. Você usará esse valor de **ponto de extremidade** posteriormente nesse exercício.

1. Mantenha a guia do navegador aberta para a próxima etapa.

## Forneça à sua identidade de usuário a função RBAC de Colaborador de Dados Internos do Cosmos DB

Como tarefa final neste exercício, você concederá à identidade de usuário do Microsoft Entra ID acesso para gerenciar dados em sua conta do Azure Cosmos DB for NoSQL atribuindo-a à função RBAC do **Colaborador de Dados Internos do Cosmos DB**. Isso permitirá que você use a autenticação do Azure para acessar o banco de dados a partir do seu código e evitar a necessidade de armazenar e gerenciar chaves.

> &#128221; A utilização do RBAC (controle de acesso baseado em função) do Microsoft Entra ID para autenticação em serviços do Azure, como o Azure Cosmos DB, apresenta vários benefícios principais em relação aos métodos baseados em chave. O RBAC do Entra ID aumenta a segurança por meio de controles de acesso precisos adaptados às funções do usuário, reduzindo com eficácia os riscos de acesso não autorizado. Ele também simplifica o gerenciamento de usuários, permitindo que os administradores atribuam e modifiquem permissões dinamicamente sem o incômodo de distribuir e manter chaves criptográficas. Além disso, essa abordagem aprimora a conformidade e a auditabilidade, alinhando-se às políticas organizacionais e facilitando o monitoramento e a revisão abrangentes do acesso. Ao simplificar o gerenciamento de acesso seguro, o RBAC do Entra ID cria uma solução mais eficiente e escalonável para aproveitar os serviços do Azure.

1. Abra o Cloud Shell na barra de ferramentas do [portal do Azure](https://portal.azure.com).

    ![Ícone do Cloud Shell destacado na barra de ferramentas do portal do Azure.](media/azure-portal-toolbar-cloud-shell.png)

1. No prompt do Cloud Shell, verifique se a assinatura do exercício é usada para comandos subsequentes executando `az account set -s <SUBSCRIPTION_ID>`, substituindo o token de espaço reservado `<SUBSCRIPTION_ID>` pela ID da assinatura que você está usando para este exercício.

1. Copie a saída do comando acima para usar como token `<PRINCIPAL_OBJECT_ID>` no comando `az cosmosdb sql role assignment create` abaixo.

1. Em seguida, você recuperará a ID de definição da função de **Colaborador de Dados Internos do Cosmos DB**. Execute o comando a seguir, garantindo que você substitua os tokens `<RESOURCE_GROUP_NAME>` e `<COSMOS_DB_ACCOUNT_NAME>`.

    ```bash
    az cosmosdb sql role definition list --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>"
    ```

    Revise a saída e localize a definição de função chamada **Colaborador interno de dados do Cosmos DB**. A saída contém o identificador exclusivo da definição de função na propriedade `name`. Registre esse valor, pois ele será necessário para uso na etapa de atribuição mais adiante.

1. Agora está tudo pronto para se atribuir à definição de função de **Colaborador de Dados Internos do Cosmos DB**. Insira o comando a seguir no prompt, certificando-se de substituir os tokens `<RESOURCE_GROUP_NAME>` e `<COSMOS_DB_ACCOUNT_NAME>`.

    > &#128221; No comando abaixo, o `role-definition-id` é definido como `00000000-0000-0000-0000-000000000002`, que é o valor padrão para a definição de função de **Colaborador de Dados Internos do Cosmos DB**. Se o valor recuperado do comando `az cosmosdb sql role definition list` for diferente, substitua o valor no comando abaixo antes da execução. O comando `az ad signed-in-user show` recuperará a ID de objeto do usuário conectado do Entra ID.

    ```bash
    az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
    ```

1. Quando o comando terminar de ser executado, você poderá executar o código localmente para inserir a interação com os dados armazenados no banco de dados NoSQL do Cosmos DB.

1. Feche o Cloud Shell.
