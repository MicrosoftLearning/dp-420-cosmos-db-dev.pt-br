---
title: 02 – Configurar o SDK do Python do Azure Cosmos DB for NoSQL para desenvolvimento offline
lab:
  title: 02 – Configurar o SDK do Python do Azure Cosmos DB for NoSQL para desenvolvimento offline
  module: Configure the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 5
parent: Python SDK labs
---

# Configurar o SDK do Python do Azure Cosmos DB for NoSQL para desenvolvimento offline

O emulador do Azure Cosmos DB é uma ferramenta local que emula o serviço do Azure Cosmos DB para desenvolvimento e teste. O emulador dá suporte à API NoSQL e pode ser usado no lugar do serviço de nuvem ao desenvolver código usando o SDK do Azure para Python.

Neste laboratório, você se conectará ao emulador do Azure Cosmos DB a partir do SDK do Azure para Python.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md) para fazer isso.

## Inicie o emulador do Azure Cosmos DB

Se você estiver usando um ambiente de laboratório hospedado, o emulador já estará instalado. Caso contrário, confira as [instruções de instalação](https://docs.microsoft.com/azure/cosmos-db/local-emulator) para instalar o emulador do Azure Cosmos DB. Depois que o emulador for iniciado, você poderá recuperar a cadeia de conexão e usá-la para se conectar ao emulador usando o SDK do Azure para Python.

> &#128161; Opcionalmente, você pode instalar o novo [Emulador do Azure Cosmos DB baseado em Linux (em versão prévia),](https://learn.microsoft.com/azure/cosmos-db/emulator-linux) que está disponível como um contêiner do Docker. Ele dá suporte à execução em uma ampla variedade de processadores e sistemas operacionais.

1. Inicie o **Emulador do Azure Cosmos DB**.

    > 💡 Se estiver usando o Windows, o emulador do Azure Cosmos DB está fixado na barra de tarefas e no menu Iniciar do Windows. Se ele não iniciar a partir dos ícones fixados, tente abri-lo clicando duas vezes no arquivo **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe**.

1. Aguarde o emulador abrir o seu navegador padrão e navegue até a página de aterrissagem **https://localhost:8081/_explorer/index.html**.

1. No painel **Início Rápido**, anote a **Cadeia de Conexão Primária**. Você usará essa cadeia de conexão mais tarde.

> &#128221; Às vezes, a página de aterrissagem não carrega, mesmo que o emulador esteja em execução. Se isso acontecer, você poderá usar a cadeia de conexão conhecida para se conectar ao emulador. A cadeia de conexão conhecida é: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## Instalar a biblioteca azure-cosmos

A biblioteca **azure-cosmos** está disponível no **PyPI** para facilitar a instalação em seus projetos do Python.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/02-sdk-offline**.

1. Abra o menu de contexto da pasta **python/02-sdk-offline** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **python/02-sdk-offline**.

1. Criar e ativar um ambiente virtual para gerenciar dependências:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Instale o pacote [azure-cosmos][pypi.org/project/azure-cosmos] usando o seguinte comando:

   ```bash
   pip install azure-cosmos
   ```

## Conectar-se ao emulador a partir do SDK do Python

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/02-sdk-offline**.

1. Abra o arquivo Python em branco chamado **script.py**.

1. Adicione o seguinte código para se conectar ao emulador, criar um banco de dados e imprimir sua ID:

   ```python
   from azure.cosmos import CosmosClient, PartitionKey
   
   # Connection string for the Azure Cosmos DB Emulator
   connection_string = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
    
   # Initialize the Cosmos client
   client = CosmosClient.from_connection_string(connection_string)
    
   # Create a database
   database_name = "cosmicworks"
   database = client.create_database_if_not_exists(id=database_name)
    
   # Print the database ID
   print(f"New Database: Id: {database.id}")
   ```

1. **Salve** o arquivo **script.py**.

## Executar o script

1. Use a mesma janela de terminal no **Visual Studio Code** que você usou para configurar o ambiente Python para este laboratório. Se tiver fechado, abra o menu de contexto da pasta **python/02-sdk-offline** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

1. Execute o script usando o comando `python`:

   ```bash
   python script.py
   ```

1. O script criará um banco de dados chamado `cosmicworks` no emulador. Você deverá ver uma saída semelhante à seguinte:

   ```text
   New Database: Id: cosmicworks
   ```

## Criar e exibir um novo contêiner

Você pode estender o script para criar um contêiner no banco de dados.

### Código atualizado

1. Modifique o arquivo `script.py` para adicionar o seguinte código na parte inferior do arquivo para criar um contêiner:

   ```python
   # Create a container
   container_name = "products"
   partition_key_path = "/categoryId"
   throughput = 400
    
   container = database.create_container_if_not_exists(
       id=container_name,
       partition_key=PartitionKey(path=partition_key_path),
       offer_throughput=throughput
   )
    
   # Print the container ID
   print(f"New Container: Id: {container.id}")
   ```

### Executar o script atualizado

1. Execute o script atualizado com o seguinte comando:

   ```bash
   python script.py
   ```

1. O script criará um contêiner chamado `products` no emulador. Você deverá ver uma saída semelhante à seguinte:

   ```text
   New Container: Id: products
   ```

### Verificar resultados

1. Alterne para o navegador em que o Data Explorer do emulador está aberto.

1. Atualize a **API NoSQL** para observar o novo banco de dados **cosmicworks** e o contêiner **products**.

## Pare o emulador do Azure Cosmos DB

É importante interromper o emulador quando terminar de usá-lo para liberar recursos do sistema. Siga as etapas abaixo de acordo com o seu sistema operacional:

### No macOS ou no Linux:

Se você iniciou o emulador em uma janela de terminal, siga estas etapas:

1. Localize a janela do terminal em que o emulador está sendo executado.

1. Pressione `Ctrl + C` para encerrar o processo do emulador.

Como alternativa, se você precisar interromper o processo do emulador manualmente:

1. Kudu uma nova janela de terminal.

1. Para localizar o processo do emulador, use o seguinte comando:

   ```bash
   ps aux | grep CosmosDB.Emulator
   ```

Identifique o **PID** (ID do processo) do processo do emulador na saída. Use o comando kill para encerrar o processo do emulador:

```bash
kill <PID>
```

### No Windows:

1. Localize o ícone do Emulador do Azure Cosmos DB na Bandeja do Sistema do Windows (próximo ao relógio na barra de tarefas).

1. Clique com o botão direito do mouse no emulador para abrir o menu de contexto.

1. Selecione **Sair** para encerrar o emulador.

> 💡 Pode levar um minuto para que todas as instâncias do emulador sejam encerradas.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
