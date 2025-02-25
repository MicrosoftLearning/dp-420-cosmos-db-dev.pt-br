---
title: 07.2 – Gerar incorporações de vetores com o OpenAI do Azure e armazená-las no Azure Cosmos DB for NoSQL
lab:
  title: 07.2 – Gerar incorporações de vetores com o OpenAI do Azure e armazená-las no Azure Cosmos DB for NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 11
parent: Python SDK labs
---

# Gerar incorporações de vetores com o OpenAI do Azure e armazená-las no Azure Cosmos DB for NoSQL

O OpenAI do Azure fornece acesso aos modelos de linguagem avançados do OpenAI, incluindo os modelos `text-embedding-ada-002`, `text-embedding-3-small` e `text-embedding-3-large`. Ao usar um desses modelos, você pode gerar representações vetoriais de dados textuais, que podem ser armazenadas em um repositório de vetores como o Azure Cosmos DB for NoSQL. Isso facilita pesquisas de similaridade eficientes e precisas, aumentando significativamente a capacidade de um copiloto de recuperar informações relevantes e fornecer interações contextualmente ricas.

Neste laboratório, você criará um serviço OpenAI do Azure e implantará um modelo de incorporação. Em seguida, você usará o código Python para criar clientes do OpenAI Azure e do Cosmos DB usando os respectivos SDKs do Python para gerar representações vetoriais de descrições de produtos e gravá-las em seu banco de dados.

> &#128721; O exercício anterior deste módulo é um pré-requisito para este laboratório. Se você ainda precisar concluir esse exercício, termine-o antes de continuar, pois ele fornece a infraestrutura necessária para este laboratório.

## Criar um serviço OpenAI do Azure

O OpenAI do Azure fornece acesso à API REST para modelos de linguagem avançados do OpenAI. Esses modelos podem ser facilmente adaptados à sua tarefa específica, incluindo, entre outros, geração de conteúdo, sumarização, reconhecimento de imagem, pesquisa semântica e tradução de linguagem natural para código.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

2. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

3. Selecione **Criar um recurso**, procure *OpenAI do Azure* e crie um novo recurso do **OpenAI do Azure** com as configurações a seguir, deixando todas as configurações restantes com seus valores padrão:

    | Configuração | Valor |
    | ------- | ----- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Região** | *Escolha uma região disponível que aceite o modelo `text-embedding-3-small` * na [lista de regiões de suporte](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings). |
    | **Nome** | *Insira um nome globalmente exclusivo* |
    | **Tipo de preço** | *Escolha Padrão 0* |

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

4. Aguarde a conclusão da tarefa de implantação antes de passar para a próxima tarefa.

## Implementar um modelo de incorporação

Para usar o OpenAI do Azure para gerar incorporações, primeiro você deve implantar uma instância do modelo de incorporação desejado em seu serviço.

1. Navegue até o serviço OpenAI do Azure recém-criado no portal do Azure (``portal.azure.com``).

2. Na página **Visão geral** do serviço OpenAI do Azure, inicie a **Fábrica de IA do Azure** clicando no link **Ir para o portal da Fábrica de IA do Azure** na barra de ferramentas.

3. Na Fábrica de IA do Azure, selecione **Implantações** no menu à esquerda.

4. Na página **Implantações de modelo**, selecione **Implantar modelo** e **Implantar modelo base** na lista suspensa.

5. Selecione `text-embedding-3-small` na lista de modelos.

    > &#128161; Você pode filtrar a lista para exibir apenas modelos de *Incorporações* usando o filtro de tarefas de inferência.

    > &#128221; Se você não vir o modelo `text-embedding-3-small`, talvez tenha selecionado uma região do Azure que não ainda não aceita o modelo. Nesse caso, você pode usar o modelo `text-embedding-ada-002` para este laboratório. Ambos os modelos geram vetores com dimensões 1536, portanto, nenhuma alteração é necessária na política de vetor de contêiner definida no contêiner `Products` no Azure Cosmos DB.

6. Selecione **Confirmar** para implantar o modelo.

7. Na página **Implantações de modelo** na Fábrica de IA do Azure, anote o **Nome** da implantação do modelo `text-embedding-3-small`, pois você precisará dele posteriormente neste exercício.

## Implantar um modelo de conclusão de chat

Além do modelo de incorporação, você precisará de um modelo de conclusão de chat para o copiloto. Você usará o grande modelo de linguagem `gpt-4o` do OpenAI para gerar respostas no copiloto.

1. Ainda na página **Implantações de modelo** na Fábrica de IA do Azure, clique no botão **Implantar modelo** novamente e escolha **Implantar modelo base** na lista suspensa.

2. Selecione o modelo de conclusão de chat **gpt-4o** na lista.

3. Selecione **Confirmar** para implantar o modelo.

4. Na página **Implantações de modelo** na Fábrica de IA do Azure, anote o **Nome** da implantação de modelo `gpt-4o`, pois você precisará dele posteriormente neste exercício.

## Atribuir a função RBAC do Usuário do OpenAI de Serviços Cognitivos

Para permitir que sua identidade de usuário interaja com o serviço OpenAI do Azure, você pode atribuir à sua conta a função **Usuário do OpenAI de Serviços Cognitivos**. O Serviço OpenAI do Azure dá suporte ao controle de acesso baseado em função do Azure (RBAC do Azure), um sistema de autorização para gerenciar o acesso individual aos recursos do Azure. Usando o RBAC do Azure, você atribui diferentes níveis de permissões a diferentes membros da equipe com base em suas necessidades para um determinado projeto.

> &#128221; O RBAC (Controle de Acesso Baseado em Função) do Microsoft Entra ID para autenticação em serviços do Azure, como o OpenAI do Azure, aumenta a segurança por meio de controles de acesso precisos personalizados para as funções do usuário, reduzindo efetivamente os riscos de acesso não autorizado. Simplificar o gerenciamento de acesso seguro usando o RBAC do Entra ID cria uma solução mais eficiente e escalonável para aproveitar os serviços do Azure.

1. No portal do Azure (``portal.azure.com``), navegue até o recurso OpenAI do Azure.

2. Selecione **Controle de acesso (IAM)** no painel de navegação à esquerda.

3. Selecione **Adicionar** e **Adicionar atribuição de função**.

4. Na guia **Função**, selecione a função **Usuário do OpenAI de Serviços Cognitivos** e selecione **Avançar**.

5. Na guia **Membros**, selecione Atribuir acesso a um usuário, grupo ou entidade de serviço e escolha **Selecionar membros**.

6. Na caixa de diálogo **Selecionar membros**, busque seu nome ou endereço de email e selecione a sua conta.

7. Na guia **Examinar + atribuir**, selecione **Examinar + atribuir** para atribuir a função.

## Criar um ambiente virtual do Python

Os ambientes virtuais em Python são essenciais para manter um espaço de desenvolvimento limpo e organizado, permitindo que projetos individuais tenham seu próprio conjunto de dependências, isolados dos demais. Isso evita conflitos entre diferentes projetos e garante consistência em seu fluxo de trabalho de desenvolvimento. Ao usar ambientes virtuais, você pode gerenciar versões de pacotes facilmente, evitar conflitos de dependência e manter seus projetos funcionando sem problemas. Essa é uma prática recomendada que mantém o ambiente de codificação estável e confiável, tornando o processo de desenvolvimento mais eficiente e menos propenso a problemas.

1. Usando o Visual Studio Code, abra a pasta na qual você clonou o repositório de código de laboratório para o módulo de aprendizado **Criar copilotos com o Azure Cosmos DB**.

2. No Visual Studio Code, abra uma nova janela de terminal e altere os diretórios para a pasta `python/07-build-copilot`.

3. Crie um ambiente virtual chamado `.venv` executando o seguinte comando no prompt do terminal:

    ```bash
    python -m venv .venv 
    ```

    O comando acima criará uma pasta `.venv` na pasta `07-build-copilot`, que fornecerá um ambiente em Python dedicado para os exercícios deste laboratório.

4. Ative o ambiente virtual selecionando o comando apropriado para seu sistema operacional e shell na tabela abaixo e executando-o no prompt do terminal.

    | Plataforma | Shell | Comando para ativar o ambiente virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

5. Instale as bibliotecas definidas em `requirements.txt`:

    ```bash
    pip install -r requirements.txt
    ```

    O arquivo `requirements.txt` contém um conjunto de bibliotecas Python que você usará neste laboratório.

    | Biblioteca | Versão | Descrição |
    | ------- | ------- | ----------- |
    | `azure-cosmos` | 4.9.0 | SDK do Azure Cosmos DB para Python – Biblioteca de cliente |
    | `azure-identity` | 1.19.0 | SDK do Azure Identity para Python |
    | `fastapi` | 0.115.5 | Estrutura da Web para criar APIs com Python |
    | `openai` | 1.55.2 | Fornece acesso à API REST do OpenAI do Azure em aplicativos Python. |
    | `pydantic` | 2.10.2 | Validação de dados usando dicas de tipo Python. |
    | `requests` | 2.32.3 | Enviar solicitações HTTP. |
    | `streamlit` | 1.40.2 | Transforma scripts em Python em aplicativos Web interativos. |
    | `uvicorn` | 0.32.1 | Uma implementação de servidor Web ASGI para Python. |
    | `httpx` | 0.27.2 | Um cliente HTTP de última geração para Python. |

## Adicionar uma função Python para vetorizar texto

O SDK do Python para o OpenAI do Azure fornece acesso a classes síncronas e assíncronas que podem ser usadas para criar incorporações para dados textuais. Essa funcionalidade pode ser encapsulada em uma função em seu código Python.

1. No painel **Explorer** no Visual Studio Code, navegue até a pasta `python/07-build-copilot/api/app` e abra o arquivo `main.py`.

    > &#128221; Esse arquivo servirá como o ponto de entrada para uma API Python de back-end que você criará no próximo exercício. Neste exercício, você fornecerá algumas funções assíncronas que podem ser usadas para importar dados com incorporações no Azure Cosmos DB que serão usadas pela API.

2. Para usar o SDK assíncrono do OpenAI do Azure para Python, importe a biblioteca adicionando o seguinte código à parte superior do arquivo `main.py`:

   ```python
   from openai import AsyncAzureOpenAI
   ```

3. Você acessará o OpenAI do Azure e o Cosmos DB de forma assíncrona usando a autenticação do Azure e as funções de RBAC do Entra ID que você atribuiu anteriormente à sua identidade de usuário. Adicione a seguinte linha abaixo da instrução import `openai` na parte superior do arquivo para importar as classes necessárias da biblioteca `azure-identity`:

   ```python
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   ```

    > &#128221; Para garantir que você possa interagir com segurança com os serviços do Azure a partir de sua API, você usará o SDK do Azure Identity para Python. Essa abordagem permite que você evite ter que armazenar ou interagir com chaves a partir do código, usando as funções de RBAC atribuídas à sua conta para acessar o Azure Cosmos DB e o OpenAI do Azure nos exercícios anteriores.

4. Crie variáveis para armazenar a versão e o ponto de extremidade da API do OpenAI do Azure, substituindo o token `<AZURE_OPENAI_ENDPOINT>` pelo valor do ponto de extremidade do serviço OpenAI do Azure. Além disso, crie uma variável para o nome da implantação do modelo de incorporação. Insira o código a seguir abaixo das instruções `import` no arquivo:

   ```python
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   ```

    Se o nome da implantação de incorporação for diferente, atualize o valor atribuído à variável de acordo.

    > &#128161; A versão `2024-10-21` da API era a versão de disponibilidade geral mais recente no momento da produção deste material. Você pode usar essa ou uma nova versão, se houver uma disponível. A documentação de especificações da API contém uma [tabela com as versões mais recentes da API](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs).

    > &#128221; O `EMBEDDING_DEPLOYMENT_NAME` é o valor **Nome** que você anotou depois de implantar o modelo `text-embedding-3-small` na Fábrica de IA do Azure. Se você precisar consultá-lo, inicie a Fábrica de IA do Azure, navegue até a página **Implantações** e localize a implantação com `text-embedding-3-small` como **Nome do modelo**. Em seguida, copie o valor do campo **Nome** desse item. Se você implantou o modelo `text-embedding-ada-002`, use o nome dessa implantação.

5. Use a classe `DefaultAzureCredential` do SDK do Azure Identity para Python para criar uma credencial assíncrona para acessar o OpenAI do Azure e o Azure Cosmos DB usando a autenticação RBAC do Microsoft Entra ID inserindo o seguinte código abaixo das declarações de variável:

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

6. Para manipular a criação de incorporações, insira o seguinte, que adiciona uma função para gerar incorporações usando um cliente OpenAI do Azure:

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
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
   ```

    A criação do cliente OpenAI do Azure não requer o valor `api_key` porque está recuperando um token de portador usando a classe `get_bearer_token_provider` do SDK do Azure Identity.

7. O arquivo `main.py` será assim:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
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
   ```

8. Salve o arquivo `main.py`.

## Testar a função de incorporação

Para garantir que a função `generate_embeddings` no arquivo `main.py` esteja funcionando corretamente, você adicionará algumas linhas de código na parte inferior do arquivo para permitir que ele seja executado diretamente. Essas linhas permitem que você execute a função `generate_embeddings` a partir da linha de comando, passando o texto a ser incorporado.

1. Adicione um bloco de **proteção principal** contendo uma chamada para `generate_embeddings` na parte inferior do arquivo `main.py`:

   ```python
   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

    > &#128221; O bloco `if __name__ == "__main__":` é comumente referenciado como **proteção principal** ou **ponto de entrada** em Python. Ele garante que determinado código seja executado apenas quando o script é executado diretamente, e não quando é importado como um módulo em outro script. Essa prática ajuda na organização do código e o torna mais reutilizável e modular.

2. Salve o arquivo `main.py`, que será assim:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
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

   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

3. No Visual Studio Code, abra uma janela de terminal integrado.

4. Antes de executar a API, que enviará solicitações ao OpenAI do Azure, faça logon no Azure usando o comando `az login`. Na janela do terminal, execute:

   ```bash
   az login
   ```

5. Conclua o processo de logon em seu navegador.

6. No prompt do terminal, altere os diretórios para `python/07-build-copilot`.

7. Certifique-se de que a janela do terminal integrado esteja em execução no ambiente virtual do Python ativando o ambiente virtual usando um comando da tabela abaixo, selecionando o comando apropriado para o sistema operacional e o shell.

    | Plataforma | Shell | Comando para ativar o ambiente virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

8. No prompt do terminal, altere os diretórios para `api/app` e execute o seguinte comando:

   ```python
   python main.py "Hello, world!"
   ```

9. Observe a saída na janela do terminal. Você verá uma matriz de número de ponto flutuante, que é a representação vetorial da cadeia de caracteres "Olá, mundo!". Ela será semelhante à seguinte saída abreviada:

   ```bash
   [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
   ```

## Criar uma função para gravar dados no Azure Cosmos DB

Usando o SDK do Azure Cosmos DB para Python, você pode criar uma função que permite a inserção de documentos em seu banco de dados. Uma operação upsert atualizará um registro se uma correspondência for encontrada e inserirá um novo registro se não for.

1. Retorne ao arquivo `main.py` aberto no Visual Studio Code e importe a classe `CosmosClient` assíncrona a partir do SDK do Azure Cosmos DB para Python inserindo a seguinte linha logo abaixo das instruções `import` que já estão no arquivo:

   ```python
   from azure.cosmos.aio import CosmosClient
   ```

2. Adicione outra instrução import para fazer referência à classe `Product` a partir do módulo *modelos* na pasta `api/app`. A classe `Product` define a forma dos produtos no conjunto de dados do Cosmic Works.

   ```python
   from models import Product
   ```

3. Crie um novo grupo de variáveis contendo valores de configuração associados ao Azure Cosmos DB e adicione-os ao arquivo `main.py` abaixo das variáveis do OpenAI do Azure que você inseriu anteriormente. Certifique-se de substituir o token `<AZURE_COSMOSDB_ENDPOINT>` pelo ponto de extremidade da sua conta do Azure Cosmos DB.

   ```python
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
   ```

4. Adicione uma função chamada `upsert_product` para executar upserting (atualizar ou inserir) de documentos no Cosmos DB, inserindo o seguinte código abaixo da função `generate_embeddings` no arquivo `main.py`:

   ```python
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
   ```

5. Salve o arquivo `main.py`, que será assim:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"

   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
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
       import sys
       print(generate_embeddings(sys.argv[1]))
   ```

## Vetorizar dados de amostra

Agora está tudo pronto para testar as funções `generate_embeddings` e `upsert_document` juntas. Para fazer isso, você substituirá o bloco de proteção principal `if __name__ == "__main__"` por um código que baixa um arquivo de dados de amostra contendo informações do produto Cosmic Works no GitHub e, em seguida, vetoriza o campo `description` de cada produto e faz upsert dos documentos no contêiner `Products` em seu banco de dados do Azure Cosmos DB.

> &#128221; Essa abordagem está sendo usada para demonstrar as técnicas de geração com o OpenAI do Azure e armazenamento de incorporações no Azure Cosmos DB. Em um cenário do mundo real, no entanto, uma abordagem mais robusta, como usar uma função do Azure disparada pelo feed de alterações do Azure Cosmos DB, seria mais adequada para manipular a adição de incorporações a documentos novos e existentes.

1. No arquivo `main.py`, substitua o bloco de código `if __name__ == "__main__":` por:

   ```python
   if __name__ == "__main__":
       import asyncio
       from models import Product
       import requests
    
       async def main():
           product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
           products = [Product(**data) for data in requests.get(product_raw_data).json()]
    
           # Call the generate_embeddings function, passing in an argument from the command line.    
           for product in products:
               print(f"Generating embeddings for product: {product.name}", end="\r")
               product.embedding = await generate_embeddings(product.description)
               await upsert_product(product.model_dump())
    
           print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
           # Close open credential sessions
           await credential.close()
    
       asyncio.run(main())
   ```

2. No prompt do terminal integrado aberto no Visual Studio Code, execute o arquivo `main.py` novamente usando o comando:

   ```python
   python main.py
   ```

3. Aguarde a conclusão da execução do código, indicada por uma mensagem indicando que foi feito upsert de todos os produtos com descrições vetorizadas no contêiner do Cosmos DB. Pode levar de 10 a 15 minutos para que o processo de vetorização e upsert de dados seja concluído para os 295 registros no conjunto de dados de produtos. Se algum produto não for inserido, você poderá executar `main.py` novamente usando o comando acima para adicionar os produtos restantes.

## Examinar dados de amostra de upsert no Cosmos DB

1. Retorne ao portal do Azure (``portal.azure.com``) e navegue até a sua conta Microsoft Azure Cosmos DB.

2. No menu de navegação esquerdo, clique em **Data Explorer**.

3. Expanda o banco de dados **CosmicWorks** e o contêiner **Produtos** e selecione **Itens** no contêiner.

4. Selecione vários documentos aleatórios dentro do contêiner e certifique-se de que o campo `embedding` esteja preenchido com uma matriz vetorial.
