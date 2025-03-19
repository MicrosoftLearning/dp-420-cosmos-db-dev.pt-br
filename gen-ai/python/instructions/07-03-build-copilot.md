---
lab:
  title: 07.3 – Criar um copiloto com o Python e o Azure Cosmos DB for NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Criar um copiloto com o Python e o Azure Cosmos DB for NoSQL

Ao utilizar os recursos de programação versáteis do Python e os recursos escalonáveis de banco de dados NoSQL e busca em vetores do Azure Cosmos DB, você pode criar copilotos de IA poderosos e eficientes, simplificando fluxos de trabalho complexos.

Neste laboratório, você criará um copiloto usando o Python e o Azure Cosmos DB for NoSQL, criando uma API de back-end que fornecerá pontos de extremidade necessários para interagir com os serviços do Azure (OpenAI do Azure e Azure Cosmos DB) e uma interface do usuário de front-end para facilitar a interação do usuário com o copiloto. O copiloto servirá como assistente para ajudar os usuários do Cosmic Works a gerenciar e encontrar produtos relacionados à bicicleta. Especificamente, o copiloto permitirá que os usuários apliquem e removam descontos de categorias de produtos, pesquisem categorias de produtos para ajudar a informar os usuários sobre quais tipos de produtos estão disponíveis e usem a busca em vetores para realizar pesquisas de similaridade para produtos.

![Um diagrama de arquitetura de copiloto de alto nível, mostrando uma interface do usuário desenvolvida em Python usando o Streamlit, uma API de back-end escrita em Python e interações com o Azure Cosmos DB e o OpenAI do Azure.](media/07-copilot-high-level-architecture-diagram.png)

Separar a funcionalidade do aplicativo em uma interface do usuário dedicada e uma API de back-end ao criar um copiloto em Python oferece vários benefícios. Em primeiro lugar, melhora a modularidade e a capacidade de manutenção, permitindo que você atualize a interface do usuário ou o back-end de forma independente, sem interromper o outro. O Streamlit fornece uma interface intuitiva e interativa que simplifica as interações do usuário, enquanto o FastAPI garante tratamento de solicitações assíncronas e de alto desempenho e processamento de dados. Essa separação também promove a escalabilidade, pois diferentes componentes podem ser implantados em vários servidores, otimizando o uso de recursos. Além disso, permite melhores práticas de segurança, uma vez que a API de back-end pode processar dados confidenciais e autenticação separadamente, reduzindo o risco de expor vulnerabilidades na camada da interface do usuário. Essa abordagem leva a uma aplicação mais robusta, eficiente e fácil.

> &#128721; Os exercícios anteriores deste módulo são pré-requisitos para este laboratório. Se você ainda precisar concluir algum desses exercícios, termine-os antes de continuar, pois eles fornecem a infraestrutura necessária e o código inicial para este laboratório.

## Construir uma API de back-end

A API de back-end para o copiloto enriquece suas habilidades de processar dados complexos, fornecer insights em tempo real e conectar-se sem problemas a diversos serviços, tornando as interações mais dinâmicas e informativas. Para criar a API para seu copiloto, você usará a biblioteca Python FastAPI. O FastAPI é uma estrutura da Web moderna e de alto desempenho projetada para permitir que você crie APIs com Python com base em dicas de tipo padrão do Python. Ao desacoplar o copiloto do back-end usando essa abordagem, você garante maior flexibilidade, capacidade de manutenção e escalabilidade, permitindo que o copiloto evolua independentemente das alterações no back-end.

> &#128721; A API de back-end se baseia no código que você adicionou ao arquivo `main.py` na pasta `python/07-build-copilot/api/app` no exercício anterior. Se você ainda não terminou o exercício anterior, complete-o antes de continuar.

1. Usando o Visual Studio Code, abra a pasta na qual você clonou o repositório de código de laboratório para o módulo de aprendizado **Criar copilotos com o Azure Cosmos DB**.

1. No painel **Explorer** no Visual Studio Code, navegue até a pasta **python/07-build-copilot/api/app** e abra o arquivo `main.py` dentro dela.

1. Adicione as seguintes linhas de código abaixo às instruções existentes `import` na parte superior do arquivo `main.py` para trazer as bibliotecas que serão usadas para executar ações assíncronas usando o FastAPI:

   ```python
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
   ```

1. Para habilitar o ponto de extremidade `/chat` que você criará para receber dados no corpo da solicitação, você passará o conteúdo por meio de um objeto `CompletionRequest` definido no módulo de *modelos* de projetos. Atualize a instrução import de `from models import Product` na parte superior do arquivo para incluir a classe `CompletionRequest` do módulo `models`. A instrução import ficará assim:

   ```python
   from models import Product, CompletionRequest
   ```

1. Você precisará do nome de implantação do modelo de conclusão de chat criado no Serviço OpenAI do Azure. Crie uma variável na parte inferior do bloco de variável de configuração do OpenAI do Azure para fornecer isso:

   ```python
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
   ```

    Se o nome da implantação de conclusão for diferente, atualize o valor atribuído à variável de acordo.

1. Os SDKs do Azure Cosmos DB e do Identity fornecem métodos assíncronos para trabalhar com esses serviços. Cada uma dessas classes será usada em várias funções em sua API, portanto, você criará instâncias globais de cada uma, permitindo que o mesmo cliente seja compartilhado entre métodos. Insira as seguintes declarações de variável global abaixo do bloco de variáveis de configuração do Cosmos DB:

   ```python
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   ```

1. Exclua as seguintes linhas de código do arquivo, pois a funcionalidade passará para a função `lifespan` que você definirá na próxima etapa:

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

1. Para criar instâncias singleton das classes `CosmosClient` e `DefaultAzureCredentail`, você usará o objeto `lifespan` no FastAPI: esse método gerencia essas classes durante o ciclo de vida do aplicativo de API. Adicione o seguinte código para definir a `lifespan`:

   ```python
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
   ```

   No FastAPI, os eventos de vida útil são operações especiais executadas no início e no final do ciclo de vida do aplicativo. Essas operações são executadas antes que o aplicativo comece a processar solicitações e depois que ele para, tornando-as ideais para inicializar e limpar recursos usados em todo o aplicativo e compartilhados entre solicitações. Essa abordagem garante que a configuração necessária seja concluída antes que qualquer solicitação seja processada e que os recursos sejam gerenciados adequadamente ao desligar.

1. Crie uma instância da classe FastAPI usando o código a seguir. Isso deve ser inserido abaixo da função `lifespan`:

   ```python
   app = FastAPI(lifespan=lifespan)
   ```

   Ao chamar `FastAPI()`, você está inicializando uma nova instância do aplicativo FastAPI. Essa instância, conhecida como `app`, servirá como o principal ponto de entrada para seu aplicativo Web. Passar o `lifespan` anexa o manipulador de eventos de vida útil ao seu aplicativo.

1. Em seguida, faça o stub dos pontos de extremidade da sua API. O método `api_status` será anexado à URL raiz da API e atuará como uma mensagem de status para mostrar que a API está em funcionamento corretamente. Você criará o ponto de extremidade `/chat` posteriormente neste exercício. Insira o seguinte código abaixo do código para criar o cliente, o banco de dados e o contêiner do Cosmos DB:

   ```python
   @app.get("/")
   async def api_status():
       """Display a status message for the API"""
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

1. Substitua o bloco de proteção principal na parte inferior do arquivo para iniciar o servidor web da ASGI (Interface de Gateway de Servidor Assíncrono) do `uvicorn` quando o arquivo for executado na linha de comando:

   ```python
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Salve o arquivo `main.py`. Ele estará semelhante ao seguinte, incluindo os métodos `generate_embeddings` e `upsert_product` que você adicionou no exercício anterior:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product, CompletionRequest
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
    
   app = FastAPI(lifespan=lifespan)
    
   @app.get("/")
   async def api_status():
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """ Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
    
   async def generate_embeddings(text: str):
       # Create Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
    
   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
    
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Para testar rapidamente sua API, abra uma nova janela de terminal integrada no Visual Studio Code.

1. Use o comando `az login` para verificar a conexão com o Azure. Executando o seguinte no prompt do terminal:

   ```bash
   az login
   ```

1. Conclua o processo de logon em seu navegador.

1. Altere os diretórios para `python/07-build-copilot` no prompt do terminal.

1. Certifique-se de que a janela do terminal integrado seja executada em seu ambiente virtual Python ativando-a usando um comando da tabela abaixo e selecionando o comando apropriado para seu sistema operacional e shell.

    | Plataforma | Shell | Comando para ativar o ambiente virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

1. No prompt do terminal, altere os diretórios para `api/app` e execute o seguinte comando para executar o aplicativo Web do FastAPI:

   ```bash
   uvicorn main:app
   ```

1. Se não abrir automaticamente, abra uma nova janela ou guia do navegador da Web e vá para <http://127.0.0.1:8000>.

    Uma mensagem de `{"status":"ready"}` na janela do navegador indica que sua API está em execução.

1. Navegue até a interface do usuário do Swagger para a API anexando `/docs` ao final da URL: <http://127.0.0.1:8000/docs>.

    > &#128221; A interface do usuário do Swagger é uma interface interativa baseada na Web para explorar e testar pontos de extremidade de API gerados a partir de especificações do OpenAPI. Ela permite que desenvolvedores e usuários visualizem, interajam e depurem chamadas de API em tempo real, aprimorando a usabilidade e a documentação.

1. Retorne ao Visual Studio Code e interrompa o aplicativo de API pressionando **CTRL+C** na janela do terminal integrado associada.

## Incorporar dados do produto no Azure Cosmos DB

Ao usar os dados do Azure Cosmos DB, o copiloto pode simplificar fluxos de trabalho complexos e ajudar os usuários a concluir tarefas com eficiência. O copiloto pode atualizar registros e recuperar valores de pesquisa em tempo real, garantindo informações precisas e rápidas. Esse recurso permite que o copiloto forneça interações avançadas, aprimorando a capacidade dos usuários de navegar e concluir tarefas com rapidez e precisão.

As funções permitirão que o copiloto de gerenciamento de produtos aplique descontos a produtos dentro de uma categoria. Essas funções serão o mecanismo por meio do qual o copiloto recupera e interage com os dados do produto Cosmic Works no Azure Cosmos DB.

1. O copiloto usará uma função assíncrona nomeada `apply_discount` para adicionar e remover descontos e preços de venda em produtos dentro de uma categoria especificada. Insira o seguinte código de função abaixo da função `upsert_product` perto da parte inferior do arquivo `main.py`:

   ```python
   async def apply_discount(discount: float, product_category: str) -> str:
       """Apply a discount to products in the specified category."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
           """,
           parameters = [
               {"name": "@product_category", "value": product_category}
           ]
       )
    
       # Apply the discount to the products
       async for item in query_results:
           item['discount'] = discount
           item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
           await container.upsert_item(item)
    
       return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
   ```

    Essa função executa uma pesquisa no Azure Cosmos DB para efetuar pull de todos os produtos dentro de uma categoria e aplicar o desconto solicitado a esses produtos. Ela também calcula o preço de venda do item usando o desconto especificado e o insere no banco de dados.

2. Em seguida, você adicionará uma segunda função chamada `get_category_names`, que o copiloto chamará para ajudar a saber quais categorias de produtos estão disponíveis ao aplicar ou remover descontos de produtos. Adicione o método abaixo da função `apply_discount` no arquivo:

   ```python
   async def get_category_names() -> list:
       """Retrieve the names of all product categories."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
       # Get distinct product categories
       query_results = container.query_items(
           query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
       )
       categories = []
       async for category in query_results:
           categories.append(category)
       return list(categories)
   ```

    A função `get_category_names` consulta o contêiner `Products` para recuperar uma lista de nomes de categoria distintos a partir do banco de dados.

3. Salve o arquivo `main.py`.

## Implementar o ponto de extremidade de chat

O ponto de extremidade de `/chat` na API de back-end serve como a interface por meio da qual a interface do usuário de front-end interage com modelos do OpenAI do Azure e dados internos do produto Cosmic Works. Esse ponto de extremidade atua como a ponte de comunicação, permitindo que a entrada da interface do usuário seja enviada ao serviço OpenAI do Azure, que processa essas entradas usando modelos de linguagem sofisticados. Os resultados são então retornados ao front-end, permitindo conversas inteligentes em tempo real. Ao aproveitar essa configuração, os desenvolvedores podem garantir uma experiência do usuário perfeita e responsiva enquanto o back-end lida com a complexa tarefa de processar linguagem natural e gerar respostas apropriadas. Essa abordagem também oferece suporte à escalabilidade e à capacidade de manutenção, desacoplando o front-end da infraestrutura de IA subjacente.

1. Localize o stub do ponto de extremidade `/chat` que você adicionou anteriormente no arquivo `main.py`.

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

    A função aceita uma `CompletionRequest` como parâmetro. A utilização de uma classe para o parâmetro de entrada permite que várias propriedades sejam passadas para o ponto de extremidade da API no corpo da solicitação. A classe `CompletionRequest` é definida no módulo *modelos* e inclui as propriedades de mensagem do usuário, histórico de chat e histórico máximo. O histórico de chat permite que o copiloto faça referência a aspectos anteriores da conversa com o usuário, mantendo o conhecimento do contexto de toda a discussão. A propriedade `max_history` permite definir o número de mensagens de histórico que devem ser passadas para o contexto do LLM. Isso permite que você controle o uso de token para seu prompt e evite limites de TPM em solicitações.

2. Para começar, exclua a linha `raise NotImplementedError("The chat endpoint is not implemented yet.")` da função ao iniciar o processo de implementar o ponto de extremidade.

3. A primeira coisa que você fará no método de ponto de extremidade de chat é fornecer um prompt do sistema. Esse prompt define a "persona" dos copilotos, ditando como o copiloto deve interagir com os usuários, responder a perguntas e aproveitar as funções disponíveis para executar ações.

   ```python
   # Define the system prompt that contains the assistant's persona.
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   """
   ```

4. Em seguida, crie uma matriz de mensagens para enviar ao LLM, adicionando o prompt do sistema, todas as mensagens no histórico de chat e a mensagem do usuário recebida. Esse código deve ir diretamente abaixo da declaração de prompt do sistema na função:

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{"role": "system", "content": system_prompt }]
    
   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)
    
   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

    A propriedade `messages` encapsulará o histórico da conversa em andamento. Ela inclui toda a sequência de entradas do usuário e as respostas da IA, o que ajuda o modelo a manter o contexto. Ao fazer referência a esse histórico, a IA pode gerar respostas coerentes e contextualmente relevantes, garantindo que as interações permaneçam fluidas e dinâmicas. Essa propriedade é crucial para permitir que a IA entenda o fluxo e as nuances da conversa à medida que ela desenrola.

5. Para permitir que o copiloto use as funções definidas acima para interagir com dados do Azure Cosmos DB, você deve definir uma coleção de "ferramentas". O LLM chamará essas ferramentas como parte de sua execução. O OpenAI do Azure usa definições de função para habilitar interações estruturadas entre a IA e várias ferramentas ou APIs. Quando uma função é definida, ela descreve as operações que pode executar, os parâmetros necessários e todas as entradas necessárias. Para criar uma matriz de `tools`, forneça o seguinte código contendo definições de função para os métodos `apply_discount` e `get_category_names` que você definiu anteriormente:

   ```python
   # Define function calling tools
   tools = [
       {
           "type": "function",
           "function": {
               "name": "apply_discount",
               "description": "Apply a discount to products in the specified category",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "discount": {"type": "number", "description": "The percent discount to apply."},
                       "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                   },
                   "required": ["discount", "product_category"]
               }
           }
       },
       {
           "type": "function",
           "function": {
               "name": "get_category_names",
               "description": "Retrieves the names of all product categories"
           }
       }
   ]
   ```

    Ao usar definições de função, o OpenAI do Azure garante que as interações entre a IA e os sistemas externos sejam bem organizadas, seguras e eficientes. Essa abordagem estruturada permite que a IA execute tarefas complexas de maneira contínua e confiável, aprimorando seus recursos gerais e a experiência do usuário.

6. Crie um cliente OpenAI do Azure assíncrono para fazer solicitações ao seu modelo de conclusão de chat:

   ```python
   # Create Azure OpenAI client
   aoai_client = AsyncAzureOpenAI(
       api_version = AZURE_OPENAI_API_VERSION,
       azure_endpoint = AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
   )
   ```

7. O ponto de extremidade de chat fará duas chamadas para o OpenAI do Azure para aproveitar a chamada de função. A primeiro fornecerá ao cliente OpenAI do Azure acesso às ferramentas:

   ```python
   # First API call, providing the model to the defined functions
   response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages,
       tools = tools,
       tool_choice = "auto"
   )
    
   # Process the model's response and add it to the conversation history
   response_message = response.choices[0].message
   messages.append(response_message)
   ```

8. A resposta desta primeira chamada contém informações do LLM sobre quais ferramentas ou funções ele determinou serem necessárias para responder à solicitação. Você deve incluir código para processar as saídas de chamada de função, inserindo-as no histórico da conversa para que o LLM possa usá-las para formular uma resposta sobre os dados contidos nessas saídas:

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

    A chamada de função no OpenAI do Azure permite a integração perfeita de APIs ou ferramentas externas diretamente na saída do modelo. Quando o modelo detecta uma solicitação relevante, ele constrói um objeto JSON com os parâmetros necessários, que você executa na sequência. O resultado é devolvido ao modelo, permitindo que ele forneça uma resposta final abrangente enriquecida com dados externos.

9. Para concluir a solicitação com os dados enriquecidos do Azure Cosmos DB, você precisa enviar uma segunda solicitação ao OpenAI do Azure para gerar uma conclusão:

   ```python
   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )
   ```

10. Por fim, retorne a resposta de conclusão à interface do usuário:

   ```python
   return final_response.choices[0].message.content
   ```

11. Salve o arquivo `main.py`. O método `generate_chat_completion` do ponto de extremidade `/chat` será assim:

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
   ```

## Criar uma interface de usuário de chat simples

A interface de usuário do Streamlit fornece uma interface para os usuários interagirem com o seu copiloto.

1. A interface de usuário será definida usando o arquivo `index.py` localizado na pasta `python/07-build-copilot/ui`.

2. Para começar, abra o arquivo `index.py` e adicione as seguintes instruções import à parte superior do arquivo:

   ```python
   import streamlit as st
   import requests
   ```

3. Configure a página do Streamlit definida no arquivo `index.py` adicionando a seguinte linha abaixo das instruções `import`:

   ```python
   st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
   ```

4. A interface do usuário interagirá com a API de back-end usando a biblioteca `requests` para fazer chamadas ao ponto de extremidade `/chat` definido na API. Você pode encapsular a chamada à API em um método que espera a mensagem do usuário atual e uma lista de mensagens do histórico de chat.

   ```python
   async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
       """Send a message to the Copilot chat endpoint."""
       try:
           api_endpoint = "http://localhost:8000"
           request = {"message": message, "chat_history": chat_history}
           response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
           return response.json()
       except Exception as e:
           st.error(f"An error occurred: {e}")
           return""
   ```

5. Defina a função `main`, que é o ponto de entrada para chamadas no aplicativo.

   ```python
   async def main():
       """Main function for the Cosmic Works Product Management Copilot UI."""
    
       st.write(
           """
           # Cosmic Works Product Management Copilot
        
           Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
        
           **Ask the copilot to apply or remove a discount on a category of products or to find products.**
           """
       )
    
       # Add a messages collection to the session state to maintain the chat history.
       if "messages" not in st.session_state:
           st.session_state.messages = []
    
       # Display message from the history on app rerun.
       for message in st.session_state.messages:
           with st.chat_message(message["role"]):
               st.markdown(message["content"])
    
       # React to user input
       if prompt := st.chat_input("What can I help you with today?"):
           with st. spinner("Awaiting the copilot's response to your message..."):
               # Display user message in chat message container
               with st.chat_message("user"):
                   st.markdown(prompt)
                
               # Send the user message to the copilot API
               response = await send_message_to_copilot(prompt, st.session_state.messages)
    
               # Display assistant response in chat message container
               with st.chat_message("assistant"):
                   st.markdown(response)
                
               # Add the current user message and assistant response messages to the chat history
               st.session_state.messages.append({"role": "user", "content": prompt})
               st.session_state.messages.append({"role": "assistant", "content": response})
   ```

6. Por fim, adicione um bloco de **proteção principal** no final do arquivo:

   ```python
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

7. Salve o arquivo `index.py`.

## Testar o copiloto por meio da interface do usuário

1. Retorne à janela de terminal integrado que você abriu no Visual Studio Code para o projeto de API e insira o seguinte para iniciar o aplicativo de API:

   ```bash
   uvicorn main:app
   ```

2. Abra uma nova janela de terminal integrado, altere os diretórios para `python/07-build-copilot` para ativar o ambiente Python, altere os diretórios para a pasta `ui` e execute o seguinte para iniciar seu aplicativo de interface do usuário:

   ```bash
   python -m streamlit run index.py
   ```

3. Se a interface do usuário não abrir automaticamente em uma janela do navegador, inicie uma nova guia ou janela do navegador e navegue até <http://localhost:8501> para abrir a interface do usuário.

4. No prompt de chat da interface do usuário, digite "Aplicar desconto" e envie a mensagem.

    Como você precisava fornecer ao copiloto mais detalhes para agir, a resposta deve ser uma solicitação de mais informações, como informar a porcentagem de desconto que você gostaria de aplicar e a categoria de produtos aos quais o desconto deve ser aplicado.

5. Para entender quais categorias estão disponíveis, peça ao copiloto para fornecer uma lista de categorias de produtos.

    O copiloto fará uma chamada de função usando a função `get_category_names` e enriquecerá as mensagens de conversa com essas categorias para responder adequadamente.

6. Você também pode solicitar um conjunto mais específico de categorias, como "Me forneça uma lista de categorias relacionadas a roupas".

7. Em seguida, peça ao copiloto para aplicar um desconto de 15% em todos os produtos de vestuário.

8. Você pode verificar se o desconto de preço foi aplicado abrindo sua conta do Azure Cosmos DB no portal do Azure, selecionando o **Data Explorer** e executando uma consulta no contêiner `Products` para exibir todos os produtos na categoria "roupas", como:

   ```sql
   SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
   WHERE CONTAINS(LOWER(c.category_name), "clothing")
   ```

    Observe que cada item nos resultados da consulta tem um valor `discount` de `0.15`, e deve o `sale_price` ser 15% menor que o `price` original.

9. Retorne ao Visual Studio Code e pressione **Ctrl+C** para interromper o aplicativo de API na janela do terminal que está executando o aplicativo. Você pode deixar a interface do usuário em execução.

## Integrar a busca em vetores

Até agora, você deu ao copiloto a capacidade de executar ações para aplicar descontos a produtos, mas ele ainda não tem conhecimento dos produtos armazenados no banco de dados. Nesta tarefa, você adicionará recursos de busca em vetores que permitirão solicitar produtos com certas qualidades e encontrar produtos semelhantes no banco de dados.

1. Retorne ao arquivo `main.py` na pasta `api/app` e forneça um método para executar buscas em vetores no contêiner `Products` em sua conta do Azure Cosmos DB. Você pode inserir esse método abaixo das funções existentes perto da parte inferior do arquivo.

   ```python
   async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
       """Search for similar product vectors in Azure Cosmos DB"""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
           FROM Products p
           WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
           ORDER BY VectorDistance(p.embedding, @query_embedding)
           """,
           parameters = [
               {"name": "@query_embedding", "value": query_embedding},
               {"name": "@num_results", "value": num_results},
               {"name": "@similarity_score", "value": similarity_score}
           ]
       )
       similar_products = []
       async for result in query_results:
           similar_products.append(result)
       formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
       return formatted_results
   ```

2. Em seguida, crie um método chamado `get_similar_products` que servirá como a função usada pelo LLM para realizar buscas em vetores no banco de dados:

   ```python
   async def get_similar_products(message: str, num_results: int):
       """Retrieve similar products based on a user message."""
       # Vectorize the message
       embedding = await generate_embeddings(message)
       # Perform vector search against products in Cosmos DB
       similar_products = await vector_search(embedding, num_results=num_results)
       return similar_products
   ```

    A função `get_similar_products` faz chamadas assíncronas para a função `vector_search` definida acima, bem como para a função `generate_embeddings` criada no exercício anterior. As incorporações são geradas na mensagem recebida do usuário para permitir que ela seja comparada aos vetores armazenados no banco de dados usando a função `VectorDistance` interna no Cosmos DB.

3. Para permitir que o LLM use as novas funções, atualize a matriz `tools` criada anteriormente, adicionando uma definição de função para o método `get_similar_products`:

   ```json
   {
       "type": "function",
       "function": {
           "name": "get_similar_products",
           "description": "Retrieve similar products based on a user message.",
           "parameters": {
               "type": "object",
               "properties": {
                   "message": {"type": "string", "description": "The user's message looking for similar products"},
                   "num_results": {"type": "integer", "description": "The number of similar products to return"}
               },
               "required": ["message"]
           }
       }
   }
   ```

4. Você também deve adicionar código para processar a saída da nova função. Adicione a seguinte condição `elif` ao bloco de código que processa saídas de chamada de função:

   ```python
   elif call.function.name == "get_similar_products":
       func_response = await get_similar_products(**json.loads(call.function.arguments))
       messages.append(
           {
               "role": "tool",
               "tool_call_id": call.id,
               "name": call.function.name,
               "content": json.dumps(func_response)
           }
       )
   ```

    O bloco concluído ficará assim:

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
           elif call.function.name == "get_similar_products":
               func_response = await get_similar_products(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

5. Por fim, você precisa atualizar a definição do prompt do sistema para fornecer instruções sobre como executar buscas em vetores. Insira o seguinte na parte inferior do `system_prompt`:

   ```plaintext
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   ```

    O prompt do sistema atualizado será semelhante a:

   ```python
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   """
   ```

6. Salve o arquivo `main.py`.

## Testar o recurso de busca em vetores

1. Reinicie o aplicativo de API executando o seguinte na janela de terminal integrado aberta para esse aplicativo no Visual Studio Code:

   ```bash
   uvicorn main:app
   ```

2. A interface do usuário ainda estará em execução, mas se você a interrompeu, retorne à janela do terminal integrado e execute:

   ```bash
   python -m streamlit run index.py
   ```

3. Retorne à janela do navegador que está executando a interface do usuário e, no prompt de chat, insira:

   ```bash
   Tell me about the mountain bikes in stock
   ```

    Esta pergunta retornará alguns produtos que correspondem à sua busca.

4. Tente outras buscas, como "Me mostre pedais duráveis", "Me forneça uma lista de cinco camisetas elegantes" e "Me dê os detalhes sobre todas as luvas adequadas para andar em clima quente".

    Nas duas últimas consultas, observe que os produtos contêm o desconto de 15% e o preço promocional que você aplicou anteriormente.
