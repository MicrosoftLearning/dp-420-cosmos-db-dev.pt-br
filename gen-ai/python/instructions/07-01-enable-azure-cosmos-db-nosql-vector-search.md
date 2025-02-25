---
title: 07.1 – Habilitar a busca em vetores do Azure Cosmos DB for NoSQL
lab:
  title: 07.1 – Habilitar a busca em vetores do Azure Cosmos DB for NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 10
parent: Python SDK labs
---

# Habilitar a busca em vetores do Azure Cosmos DB for NoSQL

O Azure Cosmos DB for NoSQL fornece uma funcionalidade eficiente de indexação e busca em vetores projetada para armazenar e consultar vetores de alta dimensão com eficiência e precisão em qualquer escala. Para usar esse recurso, você deve habilitar sua conta para usar o recurso *Busca em vetores da API NoSQL*.

Neste laboratório, você criará uma conta do Azure Cosmos DB for NoSQL e habilitará o recurso Busca em vetores nela para preparar um banco de dados para uso como repositório de vetores.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md) para fazer isso.

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#enable-vector-search-for-nosql-api). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Habilitar a busca em vetores para a API NoSQL

Nesta tarefa, você habilitará o recurso *Busca em vetores para a API NoSQL* em sua conta do Azure Cosmos DB usando a CLI do Azure.

1. Abra o Cloud Shell na barra de ferramentas do [portal do Azure](https://portal.azure.com).

    ![Ícone do Cloud Shell destacado na barra de ferramentas do portal do Azure.](media/07-azure-portal-toolbar-cloud-shell.png)

2. No prompt do Cloud Shell, verifique se a assinatura do exercício é usada para comandos subsequentes executando `az account set -s <SUBSCRIPTION_ID>`, substituindo o token de espaço reservado `<SUBSCRIPTION_ID>` pela ID da assinatura que você está usando para este exercício.

3. Habilite o recurso *Busca em vetores para a API NoSQL* executando o comando a seguir no Azure Cloud Shell, substituindo os tokens `<RESOURCE_GROUP_NAME>` e `<COSMOS_DB_ACCOUNT_NAME>` pelo nome do grupo de recursos e pelo nome da conta do Azure Cosmos DB, respectivamente.

     ```bash
     az cosmosdb update \
       --resource-group <RESOURCE_GROUP_NAME> \
       --name <COSMOS_DB_ACCOUNT_NAME> \
       --capabilities EnableNoSQLVectorSearch
     ```

4. Aguarde até que o comando seja executado com êxito antes de sair do Cloud Shell.

5. Feche o Cloud Shell.

## Criar um banco de dados e um contêiner para hospedar vetores

1. Selecione **Data Explorer** no menu à esquerda da sua conta do Azure Cosmos DB no [portal do Azure](https://portal.azure.com) e, em seguida, selecione **Novo Contêiner**.

2. Na caixa de diálogo **Novo contêiner**:
   1. Em **ID do banco de dados**, selecione **Criar novo** e insira "CosmicWorks" no campo ID do banco de dados.
   2. Na caixa **ID do contêiner**, insira o nome "Produtos".
   3. Atribua "/category_id" como a **chave de partição**.

      ![Captura de tela das configurações do Novo contêiner especificadas acima inseridas na caixa de diálogo.](media/07-azure-cosmos-db-new-container.png)

   4. Role até a parte inferior da caixa de diálogo **Novo contêiner**, expanda **Política de vetor de contêiner** e selecione **Adicionar incorporação de vetores**.

   5. Na seção de configurações da **Política de Vetor de Contêiner**, defina o seguinte:

      | Configuração | Valor |
      | ------- | ----- |
      | **Caminho** | Insira */embedding*. |
      | **Tipo de dado** | Selecione *float32*. |
      | **Função de distância** | Selecione *cosseno*. |
      | **Dimensões** | Insira *1536* para corresponder ao número de dimensões produzidas pelo modelo `text-embedding-3-small` do OpenAI. |
      | **Tipo de índice** | Selecione *diskANN*. |
      | **Tamanho do byte de quantização** | Deixe em branco. |
      | **Tamanho da lista de pesquisa de indexação** | Aceite o valor padrão *100*. |

      ![Captura de tela da Política de Vetor de Contêiner especificada acima inserida na caixa de diálogo Novo Contêiner.](media/07-azure-cosmos-db-container-vector-policy.png)

   6. Selecione **OK** para criar o banco de dados e o contêiner.

   7. Aguarde a criação do contêiner antes de continuar. Pode demorar alguns minutos para o contêiner ficar pronto.
