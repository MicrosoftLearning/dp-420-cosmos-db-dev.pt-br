---
lab:
  title: Criar um contêiner do Azure Cosmos DB for NoSQL usando modelos do Azure Resource Manager
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Criar um contêiner do Azure Cosmos DB for NoSQL usando modelos do Azure Resource Manager

Os modelos do Azure Resource Manager são arquivos JSON que definem declarativamente a infraestrutura que você deseja implantar no Azure. Os modelos do Azure Resource Manager são uma solução comum de infraestrutura como código para implantar serviços no Azure. O Bicep leva o conceito um pouco mais longe definindo uma linguagem mais fácil de ler específica do domínio que pode ser usada para criar modelos JSON.

Neste laboratório, você criará uma nova conta, um banco de dados e um contêiner do Azure Cosmos DB usando um modelo do Azure Resource Manager. Primeiro, você criará o modelo com base no JSON bruto e, em seguida, criará o modelo usando a linguagem específica do domínio Bicep.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não se familiarizou com a interface do Visual Studio Code, confira o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Após o repositório ter sido clonado, abra a pasta local que você selecionou no **Visual Studio Code**.

## Criar recursos do Azure Cosmos DB for NoSQL usando modelos do Azure Resource Manager

O provedor de recursos **Microsoft.DocumentDB** no Azure Resource Manager possibilita a implantação de contas, bancos de dados e contêineres usando arquivos JSON. Embora os arquivos possam ser complexos, eles seguem um formato previsível e podem ser gravados com a ajuda de uma extensão do Visual Studio Code.

> &#128161; Se você estiver empacado e não conseguir descobrir um erro de sintaxe com seu modelo, use este [modelo de solução do Azure Resource Manager][github.com/arm-template-guide] como guia.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **31-create-container-arm-template**.

1. Abra o arquivo de **deploy.json**.

1. Observe o modelo vazio do Azure Resource Manager:

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. Na matriz de **recursos**, adicione um novo objeto JSON para criar uma nova conta do Azure Cosmos DB:

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    O objeto é configurado com as seguintes configurações:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts* |
    | **Versão da API** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *cadeia de caracteres exclusiva gerada a partir do nome da conta*  |
    | **Localidade** | *Localização atual do grupo de recursos* |
    | **Tipo de oferta de conta** | *Standard* |
    | **Locais** | *Apenas Oeste dos EUA* |

1. Salve o arquivo **deploy.json**.

1. Abra o menu de contexto da pasta **31-create-container-arm-template** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **31-create-container-arm-template**.

1. Instale os certificados tls/ssl antes de fazer logon no Azure:

    ```
    $CurrentDirectory=$pwd
    CD "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    CD $CurrentDirectory
    ```

1. Inicie o procedimento de logon interativo para a CLI do Azure usando o seguinte comando:

    ```
    az login
    ```

1. A CLI do Azure abrirá automaticamente uma janela ou guia do navegador da Web. Na instância do navegador, entre na CLI do Azure usando as credenciais Microsoft associadas à sua assinatura.

1. Feche a janela ou a guia do navegador da Web.

1. Verifique se o provedor de laboratório criou um grupo de recursos para você; caso contrário, registre seu nome, pois você precisará dele na próxima seção.

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    Este comando pode retornar vários nomes de grupo de recursos.

1. (Opcional) ***Se nenhum grupo de recursos tiver sido criado para você***, escolha um nome de grupo de recursos e crie-o. *Lembre-se de que alguns ambientes de laboratório podem estar bloqueados e você precisará de um administrador para criar o grupo de recursos para você.*

    i. Obter o nome de localização mais perto de você nesta lista

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. Crie o grupo de recursos.  *Lembre-se de que alguns ambientes de laboratório podem estar bloqueados e você precisará de um administrador para criar o grupo de recursos para você.*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

1. Crie um novo nome de variável **resourceGroup** usando o nome do grupo de recursos criado ou exibido anteriormente neste laboratório usando o seguinte comando:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Por exemplo, se o grupo de recursos for denominado **dp420**, o comando será **$resourceGroup="dp420"**.

1. Use o cmdlet **echo** para gravar o valor da variável **$resourceGroup** na saída do terminal usando o seguinte comando:

    ```
    echo $resourceGroup
    ```

1. Implante o modelo do Azure Resource Manager usando o comando [az deployment group create][docs.microsoft.com/cli/azure/deployment/group]:

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Deixe o terminal integrado aberto e retorne ao editor para o arquivo **deploy.json**.

1. Na matriz de **recursos**, adicione um novo objeto JSON para criar um novo banco de dados do Azure Cosmos DB for NoSQL:

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    O objeto é configurado com as seguintes configurações:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Versão da API** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *cadeia de caracteres exclusiva gerada a partir do nome da conta*&amp; */cosmicworks*  |
    | **ID do recurso** | *cosmicworks* |
    | **Dependências** | *databaseAccount criado anteriormente no modelo* |

1. Salve o arquivo **deploy.json**.

1. Retorne ao terminal integrado.

1. Implante o modelo do Azure Resource Manager usando o comando **az deployment group create**:

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Deixe o terminal integrado aberto e retorne ao editor para o arquivo **deploy.json**.

1. Na matriz de **recursos**, adicione um novo objeto JSON para criar um novo contêiner do Azure Cosmos DB for NoSQL:

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    O objeto é configurado com as seguintes configurações:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **Versão da API** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *cadeia de caracteres exclusiva gerada a partir do nome da conta*&amp; */cosmicworks/products*  |
    | **ID do recurso** | *produtos* |
    | **Taxa de transferência** | *400* |
    | **Chave de partição** | */categoryId* |
    | **Dependências** | *Conta e banco de dados criados anteriormente no modelo* |

1. Salve o arquivo **deploy.json**.

1. Retorne ao terminal integrado.

1. Implante o modelo final do Azure Resource Manager usando o comando **az deployment group create**:

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Feche o terminal integrado.

## Observar os recursos implantados do Azure Cosmos DB

Depois que os recursos do Azure Cosmos DB for NoSQL forem implantados, você poderá navegar até os recursos no portal do Azure. Usando o Data Explorer, você validará que a conta, o banco de dados e o contêiner foram todos implantados e configurados corretamente.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **Grupos de recursos**, selecione o grupo de recursos que você criou ou exibiu anteriormente neste laboratório e selecione o recurso de **conta do Azure Cosmos DB** que você criou neste laboratório com o prefixo **csmsarm**.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de novos **produtos** dentro da árvore de navegação da **API NoSQL**.

1. Selecione o nó de contêiner de **produtos** na árvore de navegação da **API NoSQL** e selecione **Escala e configurações**.

1. Observe os valores na guia **Escalar**. Especificamente, observe que a opção **Manual** está selecionada na seção **Taxa de transferência** e que a taxa de transferência provisionada está definida como **400** RU/s.

1. Observe os valores na seção **Configurações**. Especificamente, observe que o valor da **Chave de partição** está definido como **/categoryId**.

1. Feche a janela ou a guia do navegador da Web.

## Criar recursos do Azure Cosmos DB for NoSQL usando modelos Bicep

O Bicep é uma linguagem específica de domínio eficiente que simplifica e facilita a implantação de recursos do Azure do que modelos do Azure Resource Manager. Você implantará o mesmo recurso exato usando o Bicep e um nome diferente para ilustrar a(s) diferença\[s\].

> &#128161; Se você estiver empacado e não conseguir descobrir um erro de sintaxe com seu modelo, use este [modelo de solução do Bicep][github.com/bicep-template-guide] como guia.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **31-create-container-arm-template**.

1. Abra o arquivo **deploy.bicep** vazio.

1. No arquivo, adicione um novo objeto para criar uma nova conta do Azure Cosmos DB:

    ```
    param location string = resourceGroup().location
    
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    O objeto é configurado com as seguintes configurações:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Alias** | *Conta* |
    | **Nome** | *csmsarm* &amp; *cadeia de caracteres exclusiva gerada a partir do nome da conta* |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Versão da API** | *2021-05-15* |
    | **Localidade** | *Localização atual do grupo de recursos* |
    | **Tipo de oferta de conta** | *Standard* |
    | **Locais** | *Apenas Oeste dos EUA* |

1. Salve o arquivo **deploy.bicep**.

1. Abra o menu de contexto da pasta **31-create-container-arm-template** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Crie um novo nome de variável **resourceGroup** usando o nome do grupo de recursos criado ou exibido anteriormente neste laboratório usando o seguinte comando:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Por exemplo, se o grupo de recursos for denominado **dp420**, o comando será **$resourceGroup="dp420"**.

1. Implante o modelo do Bicep usando o comando **az deployment group create**:

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Deixe o terminal integrado aberto e retorne ao editor para o arquivo **deploy.bicep**.

1. No arquivo, adicione um novo objeto para criar um novo banco de dados do Azure Cosmos DB:

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    O objeto é configurado com as seguintes configurações:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Pai** | *Conta criada anteriormente no modelo* |
    | **Alias** | *Backup de banco de dados* |
    | **Nome** | *cosmicworks*  |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Versão da API** | *2021-05-15* |
    | **ID do recurso** | *cosmicworks* |

1. Salve o arquivo **deploy.bicep**.

1. Retorne ao terminal integrado.

1. Implante o modelo do Bicep usando o comando **az deployment group create**:

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Deixe o terminal integrado aberto e retorne ao editor para o arquivo **deploy.bicep**.

1. No arquivo, adicione um novo objeto para criar um novo contêiner do Azure Cosmos DB:

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    O objeto é configurado com as seguintes configurações:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Pai** | *Banco de dados criado anteriormente no modelo* |
    | **Alias** | *Contêiner* |
    | **Nome** | *produtos*  |
    | **ID do recurso** | *produtos* |
    | **Taxa de transferência** | *400* |
    | **Caminho da chave de partição** | */categoryId* |

1. Salve o arquivo **deploy.bicep**.

1. Retorne ao terminal integrado.

1. Implante o modelo final do Bicep usando o comando **az deployment group create**:

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

## Observar os resultados da implantação do modelo do Bicep

As implantações do Bicep podem ser validadas usando muitas das mesmas técnicas que as implantações do Azure Resource Manager. Você não só validará que sua conta, banco de dados e contêiner foram implantados com sucesso; você também exibirá o histórico de implantação em todas as seis implantações.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **Grupos de recursos** e, em seguida, selecione o grupo de recursos criado ou exibido anteriormente neste laboratório.

1. No grupo de recursos, navegue até o painel **Implantações**.

1. Observe as seis implantações dos modelos do Azure Resource Manager e dos arquivos do Bicep.

1. Ainda dentro do grupo de recursos, navegue até o painel **Visão geral**.

1. Ainda no grupo de recursos, selecione o recurso **da conta do Azure Cosmos DB** criado neste laboratório com o prefixo **csmsbicep**.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de novos **produtos** dentro da árvore de navegação da **API NoSQL**.

1. Selecione o nó de contêiner de **produtos** na árvore de navegação da **API NoSQL** e selecione **Escala e configurações**.

1. Observe os valores na guia **Escalar**. Especificamente, observe que a opção **Manual** está selecionada na seção **Taxa de transferência** e que a taxa de transferência provisionada está definida como **400** RU/s.

1. Observe os valores na seção **Configurações**. Especificamente, observe que o valor da **Chave de partição** está definido como **/categoryId**.

1. Feche a janela ou a guia do navegador da Web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
