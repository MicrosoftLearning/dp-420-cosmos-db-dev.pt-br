---
lab:
  title: Ajustar a taxa de transferência provisionada usando um script da CLI do Azure
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Ajustar a taxa de transferência provisionada usando um script da CLI do Azure

A CLI do Azure é um conjunto de comandos que você pode usar para gerenciar vários recursos no Azure. O Azure Cosmos DB tem um grupo de comandos avançado que pode ser usado para gerenciar várias facetas de uma conta do Azure Cosmos DB, independentemente da API selecionada.

Neste laboratório, você criará uma conta, um banco de dados e um contêiner do Azure Cosmos DB usando a CLI do Azure. Em seguida, você fará ajustes na taxa de transferência provisionada usando a CLI do Azure.

## Faça logon na CLI do Azure

Antes de usar a CLI do Azure, primeiro você deve verificar a versão da CLI e fazer logon usando suas credenciais do Azure.

1. Inicie o **Visual Studio Code**.

1. Abra o menu **Terminal** e selecione **Novo terminal** para abrir uma nova instância de terminal.

1. Exiba a versão da CLI do Azure usando o seguinte comando:

    ```
    az --version
    ```

1. Exiba os grupos de comandos mais comuns da CLI do Azure usando o seguinte comando:

    ```
    az --help
    ```

1. Instale os certificados tls/ssl antes de fazer logon no Azure:

    ```
    cd "C:\Program Files\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
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

    i. Obter o nome da localização mais próxima de você nesta lista

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. Crie o grupo de recursos.  *Lembre-se de que algumas configurações de laboratório podem ser bloqueadas e você precisará de um administrador para criar o grupo de recursos para você.*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

## Criar uma conta do Azure Cosmos DB usando a CLI do Azure

O grupo de comandos **cosmosdb** contém comandos básicos para criar e gerenciar contas do Azure Cosmos DB usando a CLI. Como uma conta do Azure Cosmos DB tem um URI endereçável, é importante criar um nome globalmente exclusivo para a sua nova conta, mesmo que você a crie por meio de script.

1. Retorne à instância do terminal já aberta no **Visual Studio Code**.

1. Exiba os comandos mais comuns da CLI do Azure relacionados ao **Azure Cosmos DB** usando o seguinte comando:

    ```
    az cosmosdb --help
    ```

1. Crie uma nova variável chamada **suffix** com o cmdlet [Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] do PowerShell usando o seguinte comando:

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; O cmdlet Get-Random gera um inteiro aleatório entre 0 e 1.000.000. Isto é útil porque nossos serviços exigem um nome globalmente exclusivo.

1. Crie outro **accountName** de nome de variável usando o **csms** de cadeia de caracteres codificada e substituição variável para injetar o valor da variável **$suffix** usando o seguinte comando:

    ```
    $accountName="csms$suffix"
    ```

1. Crie outro **resourceGroup** de nome de variável usando o nome do grupo de recursos criado ou exibido anteriormente neste laboratório usando o seguinte comando:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Por exemplo, se o grupo de recursos for denominado **dp420**, o comando será **$resourceGroup="dp420"**.

1. Use o cmdlet **echo** para gravar o valor das variáveis **$accountName** e **$resourceGroup** na saída do terminal usando o seguinte comando:

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. Exiba as opções para **az cosmosdb create** usando o seguinte comando:

    ```
    az cosmosdb create --help
    ```

1. Crie uma nova conta do Azure Cosmos DB usando as variáveis predefinidas e o seguinte comando:

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. Aguarde até que a execução do comando **create** seja concluída e retorne antes de prosseguir com este laboratório.

    > &#128161; O comando **create** pode levar de dois a doze minutos para ser concluído, em média.

## Criar recursos do Azure Cosmos DB for NoSQL usando a CLI do Azure

O grupo de comandos **cosmosdb sql** contém comandos para gerenciar recursos específicos da API NoSQL para o Azure Cosmos DB. Você sempre pode usar o sinalizador **--help** para examinar as opções desses grupos de comandos.

1. Retorne à instância do terminal já aberta no **Visual Studio Code**.

1. Exiba os grupos de comandos mais comuns da CLI do Azure relacionados ao **Azure Cosmos DB for NoSQL** usando o seguinte comando:

    ```
    az cosmosdb sql --help
    ```

1. Exiba os comandos da CLI do Azure para gerenciar bancos de dados do **Azure Cosmos DB for NoSQL** usando o seguinte comando:

    ```
    az cosmosdb sql database --help
    ```

1. Crie um novo banco de dados do Azure Cosmos DB usando as variáveis predefinidas, o nome do banco de dados **cosmicworks** e o seguinte comando:

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Aguarde até que a execução do comando **create** seja concluída e retorne antes de prosseguir com este laboratório.

1. Exiba os comandos da CLI do Azure para gerenciar contêineres do **Azure Cosmos DB for NoSQL** usando o seguinte comando:

    ```
    az cosmosdb sql container --help
    ```

1. Crie um novo contêiner do Azure Cosmos DB usando as variáveis predefinidas, o nome do banco de dados **cosmicworks**, os **produtos** de nome de contêiner e o seguinte comando:

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Aguarde até que a execução do comando **create** seja concluída e retorne antes de prosseguir com este laboratório.

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais Microsoft associadas à sua assinatura.

1. Selecione **Grupos de recursos**, selecione o grupo de recursos que você criou ou exibiu anteriormente neste laboratório e selecione o recurso de **conta do Azure Cosmos DB** que você criou neste laboratório com o prefixo **csms**.

1. No recurso de conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de novos **produtos** dentro da árvore de navegação da **API NoSQL**.

1. Selecione o nó de contêiner de **produtos** na árvore de navegação da **API NoSQL** e selecione **Escala e configurações**.

1. Observe os valores na guia **Dimensionar**. Especificamente, observe que a opção **Manual** está selecionada na seção **Taxa de transferência** e que a taxa de transferência provisionada está definida como **400** RU/s.

1. Feche a janela ou a guia do navegador da Web.

## Ajustar a taxa de transferência de um contêiner existente usando a CLI do Azure

A CLI do Azure pode ser usada para migrar um contêiner entre o provisionamento manual e de escala automática de taxa de transferência. Se o contêiner estiver usando a taxa de transferência de escala automática, a CLI poderá ser usada para ajustar dinamicamente o valor máximo de taxa de transferência permitido.

1. Retorne à instância do terminal já aberta no **Visual Studio Code**.

1. Exiba os comandos da CLI do Azure para gerenciar a taxa de transferência de contêiner do **Azure Cosmos DB for NoSQL** usando o seguinte comando:

    ```
    az cosmosdb sql container throughput --help
    ```

1. Migre a taxa de transferência do contêiner de **produtos** de provisionamento manual para escala automática usando o seguinte comando:

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Aguarde até que a execução do comando de **migração** seja concluída e retorne antes de prosseguir com este laboratório.

1. Consulte o contêiner de **produtos** para determinar o valor mínimo de taxa de transferência possível usando o seguinte comando:

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Atualize a taxa de transferência máxima de escala automática do contêiner de **produtos** do valor padrão atual de **1.000** para um novo valor de **5.000** usando o seguinte comando:

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Aguarde até que o comando de **atualização** conclua a execução e retorne antes de prosseguir com este laboratório.

1. Feche o **Visual Studio Code**.

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais Microsoft associadas à sua assinatura.

1. Selecione **Grupos de recursos**, selecione o grupo de recursos que você criou ou exibiu anteriormente neste laboratório e selecione o recurso de **conta do Azure Cosmos DB** que você criou neste laboratório com o prefixo **csms**.

1. No recurso de conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o nó de contêiner de novos **produtos** dentro da árvore de navegação da **API NoSQL**.

1. Selecione o nó de contêiner de **produtos** na árvore de navegação da **API NoSQL** e selecione **Escala e configurações**.

1. Observe os valores na guia **Dimensionar**. Especificamente, observe se a opção **Escala automática** está selecionada na seção **Taxa de transferência** e se a taxa de transferência provisionada está definida como **5.000** RU/s.

1. Feche a janela ou a guia do navegador da Web.

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
