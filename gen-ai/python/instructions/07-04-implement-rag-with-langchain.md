---
lab:
  title: 07.4 – Implementar a RAG com o LangChain e a busca em vetores do Azure Cosmos DB for NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Implementar a RAG com o LangChain e a busca em vetores do Azure Cosmos DB for NoSQL

Os recursos de orquestração do LangChain trazem uma infinidade de benefícios em relação à implementação da integração de LLM do seu copiloto usando o cliente OpenAI do Azure diretamente. O LangChain permite uma integração mais perfeita com várias fontes de dados, incluindo o Azure Cosmos DB, permitindo uma busca em vetores eficiente que aprimora o processo de recuperação. O LangChain oferece ferramentas robustas para gerenciar e otimizar fluxos de trabalho, facilitando a criação de aplicativos complexos com componentes modulares e reutilizáveis. Essa flexibilidade não apenas simplifica o desenvolvimento, mas também garante escalabilidade e capacidade de manutenção.

Neste laboratório, você aprimorará seu copiloto fazendo a transição do ponto de extremidade `/chat` da API do uso do cliente OpenAI do Azure para aproveitar os poderosos recursos de orquestração do LangChain. Essa mudança permitirá uma recuperação de dados mais eficiente e um desempenho aprimorado integrando a funcionalidade de busca em vetores ao Azure Cosmos DB for NoSQL. Esteja você querendo otimizar o processo de recuperação de informações do seu aplicativo ou simplesmente explorar o potencial da RAG, este módulo guiará você na conversão, demonstrando como o LangChain pode simplificar e elevar os recursos do seu aplicativo. Vamos embarcar nessa jornada para desbloquear novas eficiências e insights com o LangChain e o Azure Cosmos DB!

> &#128721; Os exercícios anteriores deste módulo são pré-requisitos para este laboratório. Se você ainda precisar concluir algum desses exercícios, termine-os antes de continuar, pois eles fornecem a infraestrutura necessária e o código inicial para este laboratório.

## Instalar as bibliotecas do LangChain

1. Usando o Visual Studio Code, abra a pasta na qual você clonou o repositório de código de laboratório para o módulo de aprendizado **Criar copilotos com o Azure Cosmos DB**.

2. No painel do **Explorer** no Visual Studio Code, navegue até a pasta **python/07-build-copilot** e abra o arquivo `requirements.txt` encontrado nela.

3. Atualize o arquivo `requirements.txt` para incluir as bibliotecas LangChain necessárias:

   ```ini
   langchain==0.3.9
   langchain-openai==0.2.11
   ```

4. Inicie uma nova janela de terminal integrado no Visual Studio Code e altere os diretórios para `python/07-build-copilot`.

5. Certifique-se de que a janela de terminal integrado seja executada em seu ambiente virtual Python ativando-a usando o comando apropriado para seu sistema operacional e shell na tabela a seguir:

    | Plataforma | Shell | Comando para ativar o ambiente virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

6. Atualize seu ambiente virtual com as bibliotecas LangChain executando o seguinte comando no prompt do terminal integrado:

   ```bash
   pip install -r requirements.txt
   ```

7. Feche o terminal integrado.

## Atualizar a API de back-end

No laboratório anterior, você executou um padrão RAG usando o cliente e os dados do OpenAI do Azure no Azure Cosmos DB. Agora, você atualizará a API de back-end para usar um agente LangChain com ferramentas para executar as mesmas ações.

Usar o LangChain para interagir com modelos de linguagem implantados em seu Serviço OpenAI do Azure é um pouco mais simples do ponto de vista do código...

1. Remova a instrução import `from openai import AzureOpenAI` na parte superior do arquivo `main.py`. Essa biblioteca de cliente não é mais necessária, pois todas as interações com o OpenAI do Azure passarão por classes fornecidas pelo LangChain.

2. Exclua as seguintes instruções de importação na parte superior do arquivo `main.py`, pois elas não serão mais necessárias:

   ```python
   from openai import AsyncAzureOpenAI
   import json
   ```

### Atualizar ponto de extremidade de incorporação

1. Importe a classe `AzureOpenAIEmbeddings` da biblioteca `langchain_openai` adicionando a seguinte instrução import na parte superior do arquivo `main.py`:

   ```python
   from langchain_openai import AzureOpenAIEmbeddings
   ```

2. Localize o método `generate_embeddings` no arquivo e substitua-o pelo seguinte, que usa a classe `AzureOpenAIEmbeddings` para processar interações com o OpenAI do Azure:

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Use LangChain's Azure OpenAI Embeddings class
       azure_openai_embeddings = AzureOpenAIEmbeddings(
           azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
       return await azure_openai_embeddings.aembed_query(text)
   ```

    A classe `AzureOpenAIEmbeddings` fornece uma interface para interagir com a API de Incorporações do OpenAI do Azure, retornando um objeto de resposta simplificado que contém apenas o vetor gerado.

### Atualizar o ponto de extremidade de chat

1. Atualize a instrução import `lanchain_openai` para anexar a classe `AzureChatOpenAI`:

   ```python
   from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
   ```

1. Importe os seguintes objetos adicionais do LangChain que serão usados ao criar o ponto de extremidade `/chat` revisado :

   ```python
   from langchain.agents import AgentExecutor, create_openai_functions_agent
   from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
   from langchain_core.tools import StructuredTool
   ```

1. O histórico de chats será injetado na conversa do copiloto de forma diferente usando um agente LangChain, portanto, exclua as linhas de código imediatamente após a definição de `system_prompt`. Você deve excluir as linhas:

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{ "role": "system", "content": system_prompt }]

   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)

   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

1. No lugar do código que você acabou de excluir, defina um objeto `prompt` usando a classe `ChatPromptTemplate` do LangChain:

   ```python
   prompt = ChatPromptTemplate.from_messages(
       [
           ("system", system_prompt),
           MessagesPlaceholder("chat_history", optional=True),
           ("user", "{input}"),
           MessagesPlaceholder("agent_scratchpad")
       ]
   )
   ```

    O `ChatPromptTemplate` está sendo criado com vários componentes em uma ordem específica. Veja como essas peças se encaixam:

    - **Mensagem do sistema**: usa o `system_prompt` para dar uma persona ao copiloto, fornecendo instruções sobre como o assistente deve se comportar e interagir com os usuários.
    - **Histórico de chats**: permite que o `chat_history`, contendo uma lista de mensagens anteriores na conversa, seja incorporado ao contexto no qual o LLM está trabalhando.
    - **Entrada de usuário**: a mensagem do usuário atual.
    - **Bloco de rascunho do agente**: permite anotações intermediárias ou etapas executadas pelo agente.

    O prompt resultante fornece uma entrada estruturada para o agente de IA de conversão, ajudando-o a gerar uma resposta com base no contexto fornecido.

1. Em seguida, substitua a definição de matriz `tools` pelo seguinte, que usa a classe `StructuredTool` do LangChain para extrair definições de função no formato adequado:

   ```python
   tools = [
       StructuredTool.from_function(coroutine=apply_discount),
       StructuredTool.from_function(coroutine=get_category_names),
       StructuredTool.from_function(coroutine=get_similar_products)
   ]
   ```

    O método `StructuredTool.from_function` em LangChain cria uma ferramenta a partir de uma determinada função, usando os parâmetros de entrada e a descrição docstring da função. Para usá-lo com métodos assíncronos, você especifica passar o nome da função para o parâmetro de entrada `coroutine`.

    Em Python, uma docstring (abreviação de cadeia de documentação) é um tipo especial de cadeia usada para documentar uma função, método, classe ou módulo. Ela fornece uma maneira conveniente de associar a documentação ao código Python e normalmente é colocada entre aspas triplas (""" ou '''). As docstrings são colocadas imediatamente após a definição da função (ou método, classe ou módulo) que documentam.

    O uso dessa função automatiza a criação das definições de função JSON que você precisava criar manualmente usando o cliente OpenAI do Azure, simplificando o processo de chamada de função.

1. Exclua todo o código entre a definição de matriz `tools` concluída acima e a instrução `return` no final da função. Usando o cliente OpenAI do Azure, você precisava fazer duas chamadas no modelo de linguagem. A primeira para permitir que ele determine quais chamadas de função, se houver, ele precisa fazer para aumentar o prompt e a segunda para solicitar a conclusão de uma RAG. Nesse meio-tempo, você tem que usar o código para inspecionar a resposta da primeira chamada para determinar se as chamadas de função eram necessárias e, em seguida, escrever o código para "manipular" a chamada dessas funções. Em seguida, você tinha que inserir a saída dessas chamadas de função nas mensagens sendo enviadas ao LLM, para que ele pudesse ter o prompt enriquecido para raciocinar ao formular uma resposta de conclusão. O LangChain simplifica muito o processo de chamar um LLM usando um padrão de RAG, como você verá abaixo. O código que você deve remover é:

   ```python
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

   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )

   return final_response.choices[0].message.content
   ```

1. Trabalhando logo abaixo da definição de matriz `tools`, crie uma referência à API OpenAI do Azure usando a classe `AzureChatOpenAI` em LangChain:

   ```python
   # Connect to Azure OpenAI API
   azure_openai = AzureChatOpenAI(
       azure_deployment=COMPLETION_DEPLOYMENT_NAME,
       azure_endpoint=AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
       api_version=AZURE_OPENAI_API_VERSION
   )
   ```

1. Para permitir que o agente do LangChain interaja com as funções que você definiu, você criará um agente usando o método `create_openai_functions_agent`, para o qual fornecerá o objeto `AzureChatOpenAI`, a matriz `tools` e o objeto `ChatPromptTemplate`:

   ```python
   agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
   ```

    A função `create_openai_functions_agent` no LangChain cria um agente que pode chamar funções externas para executar tarefas usando ferramentas e um modelo de linguagem especificados. Isso permite a integração de vários serviços e funcionalidades no fluxo de trabalho do agente, proporcionando flexibilidade e recursos aprimorados.

1. No LangChain, a classe `AgentExecutor` é usada para gerenciar o fluxo de execução dos agentes, como o que você criou com o método `create_openai_functions_agent`. Ela manipula o processamento de entradas, a invocação de ferramentas ou modelos e a manipulação de saídas. Use o código abaixo para criar um executor de agente para seu agente:

   ```python
   agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
   ```

    O `AgentExecutor` garante que todas as etapas necessárias para gerar uma resposta sejam executadas na ordem correta. Ele abstrai as complexidades de execução dos agentes, fornecendo uma camada adicional de funcionalidade e estrutura e facilitando a criação, o gerenciamento e a escala de agentes sofisticados.

1. Você usará o método `invoke` do executor do agente para enviar a mensagem recebida do usuário para o LLM. Você também incluirá o histórico de chats. Insira o código a seguir logo abaixo da definição `agent_executor`:

   ```python
   completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
   ```

   Os tokens `input` e `chat_history` foram definidos no objeto do prompt criado usando o `ChatPromptTemplate`. Com o método `invoke`, eles serão injetados no prompt, permitindo que o LLM use essas informações ao criar uma resposta.

1. Por fim, atualize a instrução return para usar a `output` do objeto de conclusão do agente:

   ```python
   return completion["output"]
   ```

1. Salve o arquivo `main.py`. A função do ponto de extremidade `/chat` ficará assim:

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
       When asked to provide a list of products, you should:
           - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
       """
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", system_prompt),
               MessagesPlaceholder("chat_history", optional=True),
               ("user", "{input}"),
               MessagesPlaceholder("agent_scratchpad")
           ]
       )
    
       # Define function calling tools
       tools = [
           StructuredTool.from_function(apply_discount),
           StructuredTool.from_function(get_category_names),
           StructuredTool.from_function(get_similar_products)
       ]
    
       # Connect to Azure OpenAI API
       azure_openai = AzureChatOpenAI(
           azure_deployment=COMPLETION_DEPLOYMENT_NAME,
           azure_endpoint=AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
           api_version=AZURE_OPENAI_API_VERSION
       )
    
       agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
       agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
        
       completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
            
       return completion["output"]
   ```

## Iniciar os aplicativos de API e interface do usuário

1. Para iniciar a API, abra uma nova janela do terminal integrado no Visual Studio Code.

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

1. Abra uma nova janela de terminal integrado, altere os diretórios para `python/07-build-copilot` para ativar o ambiente Python, altere os diretórios para a pasta `ui` e execute o seguinte para iniciar seu aplicativo de interface do usuário:

   ```bash
   python -m streamlit run index.py
   ```

1. Se a interface do usuário não abrir automaticamente em uma janela do navegador, inicie uma nova guia ou janela do navegador e navegue até <http://localhost:8501> para abrir a interface do usuário.

## Testar o copiloto

1. Antes de enviar mensagens para a interface do usuário, retorne ao Visual Studio Code e selecione a janela de terminal integrado associada ao aplicativo de API. Dentro desta janela, você verá a saída "detalhada" gerada pelo executor do agente do LangChain, que fornece informações sobre como o LangChain está processando as solicitações que você envia. Preste atenção à saída nesta janela ao enviar as solicitações abaixo, verificando novamente após cada chamada.

1. No prompt de chat na interface do usuário, digite "Aplicar um desconto" e envie a mensagem.

    Você receberá uma resposta solicitando a porcentagem de desconto que deseja aplicar e para qual categoria de produto.

1. Responda: "Luvas".

    Você receberá uma resposta perguntando qual porcentagem de desconto você gostaria de aplicar à categoria "Luvas".

1. Envie uma mensagem dizendo "25%".

    Você receberá a resposta: "Um desconto de 25% foi aplicado a todos os produtos na categoria "Luvas".

1. Peça ao copiloto para "mostrar todas as luvas."

    Na resposta, você deve ver uma lista de todas as luvas no banco de dados, que incluirá o preço com o desconto de 25%.

1. Por fim, pergunte: "Quais são as melhores luvas para andar em clima frio?" para fazer uma busca em vetores. Isso envolve uma chamada de função para o método `get_similar_items`, que chama o método `generate_embeddings` que você atualizou para usar uma implementação LangChain e a função `vector_search`.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.
