---
title: 03 – Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL
lab:
  title: 03 – Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: Python SDK labs
---

# Criar e atualizar documentos com o SDK do Azure Cosmos DB for NoSQL

A biblioteca `azure-cosmos` inclui métodos para criar, recuperar, atualizar e excluir (CRUD) itens dentro de um contêiner do Azure Cosmos DB for NoSQL. Juntos, esses métodos executam algumas das operações “CRUD” mais comuns em vários itens dentro de contêineres da API NoSQL.

Neste laboratório, você usará o SDK do Python para executar operações CRUD diárias em um item dentro de um contêiner do Azure Cosmos DB for NoSQL.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md) para fazer isso.

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#install-the-azure-cosmos-library). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Instalar a biblioteca azure-cosmos

A biblioteca **azure-cosmos** está disponível no **PyPI** para facilitar a instalação em seus projetos do Python.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/03-sdk-crud**.

1. Abra o menu de contexto da pasta **python/03-sdk-crud** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **python/03-sdk-crud**.

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
       asyncio.run(query_items_async())
   ```

1. Seu arquivo **script.py** será assim:

   ```python
   from azure.cosmos import PartitionKey
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

## Executar operações de ponto de leitura e criação em itens com o SDK

Agora você usará o conjunto de métodos na classe **ContainerProxy** para executar operações comuns em itens em um contêiner da API NoSQL.

1. Volte para o **Visual Studio Code**. Se ainda não estiver aberto, abra o arquivo de código **script.py** na pasta **python/03-sdk-crud**.

1. Crie um novo item de produto e atribua-o a uma variável chamada **saddle** com as seguintes propriedades:

    | Propriedade | Valor |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **marcas** | *{ tan, new, crisp }* |

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
   ```

1. Invoque o método [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item) da variável **contêiner**, passando a variável **saddle** como o parâmetro do método:

   ```python
   await container.create_item(body=saddle)
   ```

1. Quando terminar, seu arquivo de código deverá incluir:
  
   ```python
   from azure.cosmos import PartitionKey
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
        
           saddle = {
               "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
               "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
               "name": "Road Saddle",
               "price": 45.99,
               "tags": ["tan", "new", "crisp"]
           }
            
           await container.create_item(body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Salve** e execute o script de novo:

   ```bash
   python script.py
   ```

1. Observe o novo item no **Data Explorer**.

1. Volte para o **Visual Studio Code**.

1. Retorne à guia do editor do arquivo de código **script.py**.

1. Exclua as seguintes linhas de código:

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
    
   await container.create_item(body=saddle)
   ```

1. Crie uma variável de cadeia de caracteres chamada **item_id** com o valor **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

   ```python
   item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
   ```

1. Crie uma variável de cadeia de caracteres chamada **partition_key** com o valor **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Invoque o método [](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item)`read_item` da variável **contêiner** passando as variáveis **item_id** e **partition_key** como os parâmetros do método:

   ```python
   # Read item    
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
   ```

    > &#128161; O método `read_item` permite que você execute uma operação de leitura de ponto em um item no contêiner. O método requer os parâmetros `item_id` e `partition_key` para identificar o item a ser lido. Em vez de executar uma consulta usando a linguagem de consulta SQL do Cosmos DB para localizar o único item, o método `read_item` é uma maneira mais eficiente e econômica de recuperar um único item. As leituras de ponto podem ler os dados diretamente e não exigem que o mecanismo de consulta processe a solicitação.

1. Imprima o objeto saddle usando uma cadeia de caracteres de saída formatada:

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Quando terminar, seu arquivo de código deverá incluir:

   ```python
   from azure.cosmos import PartitionKey
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
       
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Salve** e execute o script de novo:

   ```bash
   python script.py
   ```

1. Observe a saída do terminal. Especificamente, observe o texto de saída formatado com a ID, o nome e o preço do item.

## Executar operações de ponto de atualização e exclusão com o SDK

Ao aprender o SDK, não é incomum usar uma conta online ou o emulador do Azure Cosmos DB para atualizar um item e oscilar entre o Data Explorer e o IDE de sua escolha ao executar uma operação e verificar se a alteração foi aplicada. Aqui, você fará exatamente isso ao atualizar e excluir um item usando o SDK.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Escolha o único item dentro do contêiner e observe os valores das propriedades **name** e **price** do item.

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *Road Saddle* |
    | **Price** | *$45.99* |

    > &#128221; Neste ponto, esses valores não devem ter sido alterados desde que você criou o item. Você vai alterar esses valores neste exercício.

1. Volte para o **Visual Studio Code**. Retorne à guia do editor do arquivo de código **script.py**.

1. Exclua a seguinte linha de código:

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Altere a variável **saddle** definindo o valor da propriedade price como **32.55**:

   ```python
   saddle["price"] = 32.55
   ```

1. Modifique a variável **saddle** novamente, definindo o valor da propriedade **name** como **Road LL Saddle**:

   ```python
   saddle["name"] = "Road LL Saddle"
   ```

1. Invoque o método [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item) da variável **contêiner** passando as variáveis **item_id** e **partition_key** como os parâmetros do método:

   ```python
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Quando terminar, seu arquivo de código deverá incluir:

   ```python
   from azure.cosmos import PartitionKey
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
        
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           saddle["price"] = 32.55
           saddle["name"] = "Road LL Saddle"
    
           await container.replace_item(item=item_id, body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **Salve** e execute o script de novo:

   ```bash
   python script.py
   ```

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Escolha o único item dentro do contêiner e observe os valores das propriedades **name** e **price** do item.

    | **Propriedade** | **Valor** |
    | ---: | :--- |
    | **Nome** | *Road LL Saddle* |
    | **Price** | *$32.55* |

    > &#128221; Neste ponto, esses valores devem ter sido alterados desde que você observou o item.

1. Volte para o **Visual Studio Code**. Retorne à guia do editor do arquivo de código **script.py**.

1. Exclua as seguintes linhas de código:

   ```python
   # Read item
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
    
   saddle["price"] = 32.55
   saddle["name"] = "Road LL Saddle"
    
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Invoque o método [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item) da variável **contêiner** passando as variáveis **item_id** e **partition_key** como os parâmetros do método:

   ```python
   # Delete the item
   await container.delete_item(item=item_id, partition_key=partition_key)
   ```

1. Salve e execute o script de novo:

   ```bash
   python script.py
   ```

1. Feche o terminal integrado.

1. Volte à janela ou à guia do navegador da Web.

1. No recurso da conta do **Azure Cosmos DB**, navegue até o painel do **Data Explorer**.

1. No **Data Explorer**, expanda o nó do banco de dados **cosmicworks** e expanda o novo nó de contêiner de **produtos** na árvore de navegação da **API NoSQL**.

1. Selecione o nó **Items**. Observe que a lista de itens agora está vazia.

1. Feche a janela ou a guia do navegador da Web.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
