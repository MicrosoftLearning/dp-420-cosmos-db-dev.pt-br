---
lab:
  title: 01 – Conectar-se ao Azure Cosmos DB for NoSQL com o SDK
  module: Use the Azure Cosmos DB for NoSQL SDK
---

# Conectar-se ao Azure Cosmos DB for NoSQL com o SDK

O SDK do Azure para Python é um conjunto de bibliotecas de cliente que fornece uma interface de desenvolvedor consistente para interagir com muitos serviços do Azure. As bibliotecas de cliente são pacotes usados para consumir esses recursos e interagir com eles.

Neste laboratório, você se conectará a uma conta do Azure Cosmos DB for NoSQL usando o SDK do Azure para Python.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório para **Criar copilotos com o Azure Cosmos DB** e configurou seu ambiente local, veja as instruções de como [Configurar o ambiente de laboratório local](00-setup-lab-environment.md).

## Criar uma conta do Azure Cosmos DB for NoSQL

Se você já criou uma conta do Azure Cosmos DB for NoSQL para os laboratórios **Criar copilotos com o Azure Cosmos DB** neste site, poderá usá-la para este laboratório e pular para a [próxima seção](#install-the-azure-cosmos-library). Caso contrário, veja as instruções de como [Configurar o Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para criar uma conta do Azure Cosmos DB for NoSQL que você usará em todos os módulos de laboratório e conceda acesso à identidade do usuário para gerenciar dados na conta atribuindo-os à função de **Colaborador de dados internos do Cosmos DB**.

## Instalar a biblioteca azure-cosmos

A biblioteca **azure-cosmos** está disponível no **PyPI** para facilitar a instalação em seus projetos do Python.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/01-sdk-connect**.

1. Abra o menu de contexto da pasta **python/01-sdk-connect** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância de terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido para a pasta **python/01-sdk-connect**.

1. Criar e ativar um ambiente virtual para gerenciar dependências:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Instale o pacote [azure-cosmos][pypi.org/project/azure-cosmos] usando o seguinte comando:

   ```bash
   pip install azure-cosmos
   ```

1. Instale a biblioteca [azure-identity][pypi.org/project/azure-identity], que nos permite usar a autenticação do Azure para nos conectarmos ao workspace do Azure Cosmos DB, usando o seguinte comando:

   ```bash
   pip install azure-identity
   ```

1. Feche o terminal integrado.

## Usar a biblioteca azure-cosmos

Depois que a biblioteca do Azure Cosmos DB do SDK do Azure para Python tiver sido importada, você poderá usar imediatamente suas classes para se conectar a uma conta do Azure Cosmos DB for NoSQL. A classe **CosmosClient** é a classe principal usada para fazer a conexão inicial com uma conta do Azure Cosmos DB for NoSQL.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **python/01-sdk-connect**.

1. Abra o arquivo Python em branco chamado **script.py**.

1. Adicione a seguinte instrução `import` para importar as classes **CosmosClient** e **DefaultAzureCredential**:

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential
   ```

1. Adicione as variáveis chamadas **ponto de extremidade** e **credencial** e defina o valor do **ponto de extremidade** como o **ponto de extremidade** da conta do Azure Cosmos DB criada anteriormente. A variável **credencial** deve ser definida como uma nova instância da classe **DefaultAzureCredential**:

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Por exemplo, se o seu ponto de extremidade for: **https://dp420.documents.azure.com:443/**, a instrução seria: **endpoint = "https://dp420.documents.azure.com:443/"**.

1. Adicione uma nova variável chamada **cliente** e inicialize-a como uma nova instância da classe **CosmosClient** usando as variáveis **ponto de extremidade** e **credencial** :

   ```python
   client = CosmosClient(endpoint, credential=credential)
   ```

1. Adicione uma função chamada **main** para ler e imprimir as propriedades da conta:

   ```python
   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
   ```

1. Seu arquivo **script.py** será assim:

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   client = CosmosClient(endpoint, credential=credential)

   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
    ```

1. **Salve** o arquivo **script.py**.

## Testar o script

Agora que o código Python para conexão com a conta do Azure Cosmos DB for NoSQL está concluído, você pode testar o script. Esse script imprimirá o nível de consistência padrão e o nome da primeira região gravável. Ao criar a conta, você especificou um local e deve esperar ver esse mesmo valor de local impresso como resultado desse script.

1. No **Visual Studio Code**, abra o menu de contexto da pasta **python/01-sdk-connect** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Antes de executar o script, você deve fazer logon no Azure usando o comando `az login`. Na janela do terminal, execute:

   ```bash
   az login
   ```

1. Execute o script usando o comando `python`:

   ```bash
   python script.py
   ```

1. O script agora produzirá o nível de consistência padrão e a primeira região gravável. Por exemplo, se o nível de consistência padrão da conta for **Sessão** e a primeira região gravável for **Leste dos EUA**, o script gerará:

   ```text
   Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
   Primary Region: East US
   ```

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
