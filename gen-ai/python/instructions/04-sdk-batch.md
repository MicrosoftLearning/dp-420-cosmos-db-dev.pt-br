---
title: 04 – Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL
lab:
  title: 04 – Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 7
parent: Python SDK labs
---

# Agrupar várias operações de pontos com o SDK do Azure Cosmos DB for NoSQL

O SDK do Python `azure-cosmos` fornece o método `execute_item_batch` para executar várias operações de ponto em uma única etapa lógica. Isso permite que os desenvolvedores agrupem com eficiência várias operações e determinem se elas foram concluídas com êxito no lado do servidor.

Neste laboratório, você usará o SDK do Python para executar operações em lote de item duplo que demonstram lotes transacionais bem-sucedidos e errôneos.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md) para fazer isso.

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#install-the-azure-cosmos-library). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Instalar a biblioteca azure-cosmos

A biblioteca **azure-cosmos** está disponível no **PyPI** para facilitar a instalação em seus projetos do Python.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/04-sdk-batch**.

1. Abra o menu de contexto da pasta **python/04-sdk-batch** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **python/04-sdk-batch**.

1. Criar e ativar um ambiente virtual para gerenciar dependências:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Instale o pacote [azure-cosmos][pypi.org/project/azure-cosmos] usando o seguinte comando:

   ```bash
   pip install azure-cosmos
   ```

1. Como estamos usando a versão assíncrona do SDK, precisamos instalar a biblioteca `asyncio` também:

   ```bash
   pip install asyncio
   ```

1. A versão assíncrona do SDK também requer a biblioteca `aiohttp`. Instale-o usando o comando a seguir:

   ```bash
   pip install aiohttp
   ```

1. Instale a biblioteca [azure-identity][pypi.org/project/azure-identity], que nos permite usar a autenticação do Azure para nos conectarmos ao workspace do Azure Cosmos DB, usando o seguinte comando:

   ```bash
   pip install azure-identity
   ```

## Usar a biblioteca azure-cosmos

Usando as credenciais da conta recém-criada, você se conectará às classes do SDK e criará um novo banco de dados e uma nova instância de contêiner. Em seguida, você usará o Data Explorer para validar a existência das instâncias no portal do Azure.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/03-sdk-crud**.

1. Abra o arquivo Python em branco chamado **script.py**.

1. Adicione a seguinte instrução `import` para importar a classe **PartitionKey**:

   ```python
   from azure.cosmos import PartitionKey
   ```

1. Adicione as seguintes instruções `import` para importar a classe assíncrona **CosmosClient**, a classe **DefaultAzureCredential** e a biblioteca **asyncio**:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio
   ```

1. Adicione as variáveis chamadas **ponto de extremidade** e **credencial** e defina o valor do **ponto de extremidade** como o **ponto de extremidade** da conta do Azure Cosmos DB criada anteriormente. A variável **credencial** deve ser definida como uma nova instância da classe **DefaultAzureCredential**:

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Por exemplo, se o seu ponto de extremidade for: **https://dp420.documents.azure.com:443/**, a instrução seria: **endpoint = "https://dp420.documents.azure.com:443/"**.

1. Qualquer interação com o Cosmos BD começa com uma instância da `CosmosClient`. Para usar o cliente assíncrono, precisamos usar as palavras-chave async/await, que só podem ser usadas em métodos assíncronos. Crie um novo método assíncrono chamado **main** e adicione o seguinte código para criar uma nova instância da classe **CosmosClient** assíncrona usando as variáveis **ponto de extremidade** e **credencial**:

   ```python
   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
   ```

    > &#128161; Como estamos usando o cliente **CosmosClient** assíncrono, para usá-lo corretamente, você também precisa aquecê-lo e fechá-lo. Recomendamos usar as palavras-chave `async with` conforme demonstrado no código acima para iniciar seus clientes – essas palavras-chave criam um gerenciador de contexto que aquece, inicializa e limpa automaticamente o cliente, para que você não precise fazer isso.

1. Adicione o seguinte código para criar um banco de dados e um contêiner, caso ainda não existam:

   ```python
   # Create database
   database = await client.create_database_if_not_exists(id="cosmicworks")
    
   # Create container
   container = await database.create_container_if_not_exists(
       id="products",
       partition_key=PartitionKey(path="/categoryId"),
       offer_throughput=400
   )
   ```

1. Abaixo do método `main`, adicione o seguinte código para executar o método `main` usando a biblioteca `asyncio`:

   ```python
   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. Seu arquivo **script.py** será assim:

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Salve** o arquivo **script.py**.

1. Antes de executar o script, você deve fazer logon no Azure usando o comando `az login`. Na janela do terminal, execute:

   ```bash
   az login
   ```

1. Execute um script para criar o banco de dados e o contêiner:

   ```bash
   python script.py
   ```

1. Alterne para a janela do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e observe o novo nó do contêiner **products** dentro da árvore de navegação **API NoSQL**.

## Como criar um lote transacional

Primeiro, vamos criar um lote transacional simples que cria dois produtos fictícios. Esse lote vai inserir um assento usado e um guidão enferrujado no contêiner com o mesmo identificador de categoria “acessórios usados”. Ambos os itens têm a mesma chave de partição lógica, garantindo que teremos uma operação em lote bem-sucedida.

1. Volte para o **Visual Studio Code**. Se ainda não estiver aberto, abra o arquivo de código **script.py** na pasta **python/04-sdk-batch**.

1. Crie dois dicionários representando produtos: um **assento usado** e um **guidão** enferrujado. Ambos os itens compartilham o mesmo valor de chave de partição **"9603ca6c-9e28-4a02-9194-51cdb7fea816"**.

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   ```

1. Defina o valor da chave de partição.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Crie um lote contendo os dois itens.

   ```python
   batch = [saddle, handlebar]
   ```

1. Execute o lote usando o método `execute_item_batch` do objeto `container` e imprima a resposta para cada item no lote.

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. Quando terminar, seu arquivo de código deverá incluir:
  
   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           saddle = ("create", (
               {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           handlebar = ("create", (
               {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [saddle, handlebar]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Salve** e execute o script de novo:

   ```bash
   python script.py
   ```

1. A saída indicará um código de status de êxito para cada operação.

## Como criar um lote transacional errante

Agora, vamos criar um lote transacional que erra propositalmente. Esse lote tentará inserir dois itens que têm chaves de partição lógicas diferentes. Criaremos uma luz estroboscópica cintilante na categoria “acessórios usados” e um novo capacete na categoria “acessórios novos”. Por definição, essa deve ser uma solicitação incorreta e retornar um erro ao executar essa transação.

1. Retorne à guia do editor do arquivo de código **script.py**.

1. Exclua as seguintes linhas de código:

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

   batch = [saddle, handlebar]
   ```

1. Modifique o script para criar uma nova **luz estroboscópica cintilante** e um **novo capacete** com diferentes valores de chave de partição.

   ```python
   light = ("create", (
       {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   helmet = ("create", (
       {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
   ))
   ```

1. Defina o valor da chave de partição para o lote.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Crie um novo lote contendo os dois itens.

   ```python
   batch = [light, helmet]
   ```

1. Quando terminar, seu arquivo de código deverá incluir:

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           light = ("create", (
               {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           helmet = ("create", (
               {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [light, helmet]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **Salve** e execute o script de novo:

   ```bash
   python script.py
   ```

1. Observe a saída do terminal. O código de status no segundo item (o "Novo Capacete") deve ser **400** para **Solicitação Incorreta**. Isso ocorreu porque todos os itens dentro da transação não compartilhavam o mesmo valor de chave de partição que o lote transacional.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
