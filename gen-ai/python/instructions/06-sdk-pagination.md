---
lab:
  title: 06 – Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL
  module: Author complex queries with the Azure Cosmos DB for NoSQL
---

# Paginar resultados de consultas entre produtos com o SDK do Azure Cosmos DB for NoSQL

As consultas do Azure Cosmos DB normalmente terão várias páginas de resultados. A paginação é feita automaticamente no lado do servidor quando o Azure Cosmos DB não pode retornar todos os resultados da consulta em uma única execução. Em muitos aplicativos, você desejará escrever código usando o SDK para processar os resultados da consulta em lotes de maneira eficiente.

Neste laboratório, você criará um iterador de feed que pode ser usado em um loop para iterar sobre todo o conjunto de resultados.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md).

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#create-azure-cosmos-db-database-and-container-with-sample-data). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Criar um banco de dados e um contêiner do Azure Cosmos DB com dados de amostra

Se você já criou um banco de dados do Azure Cosmos DB chamado **cosmicworks-full** e um contêiner dentro dele chamado **products**, que é pré-carregado com dados de amostra, você poderá usá-lo para este laboratório e pular para a [próxima seção](#install-the-azure-cosmos-library). Caso contrário, siga as etapas abaixo para criar um novo banco de dados e contêiner de amostra.

<details markdown=1>
<summary markdown="span"><strong>Clique para expandir/recolher as etapas para criar banco de dados e contêiner com dados de amostra</strong></summary>

1. Dentro do recurso de conta do **Azure Cosmos DB** recém-criado, navegue até o painel do **Data Explorer**.

1. Na home page do **Data Explorer**, selecione **Lançar início rápido**.

1. No formulário **Novo contêiner**, insira os seguintes valores:

    - **Database id**: `cosmicworks-full`
    - **Container id**: `products`
    - **Chave de partição**: `/categoryId`
    - **Repositório analítico**: `Off`

1. Selecione **OK** para criar o novo contêiner. Esse processo levará um ou dois minutos enquanto cria os recursos e pré-carrega o contêiner com dados de produto de amostra.

1. Mantenha a guia do navegador aberta, pois voltaremos a ela mais tarde.

1. Volte ao **Visual Studio Code**.

</details>

## Instalar a biblioteca azure-cosmos

A biblioteca **azure-cosmos** está disponível no **PyPI** para facilitar a instalação em seus projetos do Python.

1. No **Visual Studio Code**, no painel **Explorer**, navegue até a pasta **python/06-sdk-pagination**.

1. Abra o menu de contexto da pasta **python/06-sdk-pagination** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **python/06-sdk-pagination**.

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

## Paginar através de pequenos conjuntos de resultados de uma consulta SQLusando o SDK

Ao processar resultados de consultas, é preciso garantir que o código avance por todas as páginas de resultados e verifique se há mais páginas restantes antes de fazer solicitações subsequentes.

1. No **Visual Studio Code**, no painel **Explorer**, navegue até a pasta **python/06-sdk-pagination**.

1. Abra o arquivo Python em branco chamado **script.py**.

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

1. Adicione o seguinte código para se conectar ao banco de dados e ao contêiner que você criou anteriormente:

   ```python
   database = client.get_database_client("cosmicworks-full")
   container = database.get_container_client("products")
   ```

1. Crie uma nova variável chamada **sql** do tipo *cadeia de caracteres* com o valor **SELECT * FROM products WHERE products.price > 500**:

   ```python
   sql = "SELECT * FROM products WHERE products.price > 500"
   ```

1. Invoque o método [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) com a variável `sql` como um parâmetro para o construtor. Defina `max_item_count` como `50` para limitar o número de itens retornados em cada página.

   ```python
   iterator = container.query_items(
       query=sql,
       max_item_count=50  # Set maximum items per page
   )
   ```

1. Crie um loop **for** assíncrono que invoque de forma assíncrona o método [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page) no objeto iterador. Esse método retorna uma página de resultados cada vez que é chamado.

   ```python
   async for page in iterator.by_page():
   ```

1. No loop **for** assíncrono, itere de forma assíncrona sobre os resultados paginados e imprima o `id`, `name` e `price` de cada item.

   ```python
   async for product in page:
       print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")
   ```

1. Abaixo do método `main`, adicione o seguinte código para executar o método `main` usando a biblioteca `asyncio`:

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. Seu arquivo **script.py** será assim:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Get database and container clients
           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products WHERE products.price > 500"
        
           iterator = container.query_items(
               query=sql,
               max_item_count=50  # Set maximum items per page
           )
        
           async for page in iterator.by_page():
               async for product in page:
                   print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")

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

1. O script agora gerará páginas de 50 itens por vez.

    > &#128161; A consulta corresponderá a centenas de itens no contêiner de produtos.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
