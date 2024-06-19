---
lab:
  title: Recuperar um banco de dados ou um contêiner de um ponto de recuperação
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Recuperar um banco de dados ou um contêiner de um ponto de recuperação 

O Azure faz backups criptografados de seus dados automaticamente. Esses backups são feitos segundo dois modos: os modos de backup **Periódico** e **Contínuo**.

Nesse laboratório, você fará um **backup** e **restaurações** usando o modo de backup contínuo. Primeiro, você irá Criar uma conta do Azure Cosmos DB. A seguir, você criará dois contêineres e adicionará a eles alguns documentos. Em seguida, você irá atualizar alguns documentos nesses contêineres. Para terminar, você criará restaurações da conta até um ponto anterior a cada exclusão.

## Criar uma conta do Azure Cosmos DB for NoSQL

O Azure Cosmos DB é um serviço de banco de dados NoSQL baseado em nuvem que dá suporte a várias APIs. Ao provisionar uma conta do Azure Cosmos DB pela primeira vez, você irá selecionar a quais APIs você quer que a conta dê suporte (por exemplo, a **API do Mongo** ou a **API do NoSQL**). Após a conta do Azure Cosmos DB for NoSQL ter concluído o provisionamento, você poderá recuperar o ponto de extremidade e a chave. Use o ponto de extremidade e a chave para se conectar à conta do Azure Cosmos DB for NoSQL programaticamente. Use o ponto de extremidade e a chave nas cadeias de conexão do SDK do Azure para .NET ou qualquer outro SDK.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Selecione **+ Criar um recurso**, procure *Cosmos DB* e, em seguida, crie um novo recurso de conta do **Azure Cosmos DB for NoSQL** com as seguintes configurações, deixando todas as configurações restantes com seus valores padrão:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Account Name** | *Insira um nome globalmente exclusivo* |
    | **Localidade** | *Escolha qualquer região disponível* |
    | **Modo de capacidade** | *Taxa de transferência provisionada* |
    | **Aplicar Desconto na Camada Gratuita** | *Não Aplicar* |
    | Guia **Distribuição Global** | Desabilitar gravações em várias regiões |

    > &#128221; Observe que você pode habilitar o modo ** Contínuo** (7 dias) durante a criação da conta do Azure Cosmos DB ao selecioná-lo na guia **Política de Backup**. Nesse Laboratório, você tem a opção de habilitar esse recurso durante a criação da conta ou após a conta ser criada na seção opcional abaixo. **No entanto, habilitar o recurso <ins>*após*</ins> a conta ter sido criada *pode levar mais de 5 minutos*.**

    > &#128221; Observe que *[Não há suporte para contas de gravação em várias regiões para backups contínuos atualmente][/azure/cosmos-db/continuous-backup-restore-introduction]*.

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

## Adicionar um banco de dados e dois contêineres à conta

Vamos criar um banco de dados e um par de contêineres.

1. No portal do Azure, navegue até a sua conta do Azure Cosmos DB.

1. No **Data Explorer**, adicione um novo contêiner com as configurações a seguir

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **ID do banco de dados** | *Crie um novo nome*: *`Sales`* |
    | **Compartilhar a taxa de transferência entre contêineres** | *Não selecione* |
    | **ID do contêiner** | *`customer`* |
    | **Chave de partição** | *`/id`* |
    | **Taxa de transferência de contêiner (400 a ilimitadas RU/s)** | Taxa de transferência *manual* : *400*|

1. No **Data Explorer**, adicione um novo contêiner com as configurações a seguir

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **ID do banco de dados** | *Use o nome existente*: *Vendas* |
    | **ID do contêiner** | *`salesOrder`* |
    | **Chave de partição** | *`/id`* |
    | **Taxa de transferência de contêiner (400 a ilimitadas RU/s)** | Taxa de transferência *manual* : *400*|

## Adicionar itens aos contêineres

Vamos adicionar alguns documentos a esses contêineres.

1. No portal do Azure, navegue até a sua conta do Azure Cosmos DB.

1. No **Data Explorer**, adicione os dois documentos a seguir ao contêiner do **cliente**.

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. No **Data Explorer**, adicione os três documentos a seguir ao contêiner **salesOrder**.

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## Altere o modo de backup padrão para contínuo (opcional, se o recurso não tiver sido habilitado durante a criação da conta)

*Se não tiver habilitado o recurso ao criar sua conta do Azure Cosmos DB, você precisará fazer isso agora.*  Alterar o modo de backup é simples; é necessário apenas alterar uma configuração para **Ativado**. Vamos alterar isso agora.

1. No portal do Azure, navegue até a sua conta do Azure Cosmos DB.

1. Na seção **Configurações**, selecione **Backup e Restauração**.

1. Selecione **Alterar** ao lado do modo **Política de Backup**; na tela, selecione a opção **Contínuo (7 Dias)** e, em seguida, selecione **Salvar**. ***Habilitar esse recurso pode levar mais de cinco minutos***.

    > &#128221; Observe que *[Não há suporte para contas de gravação em várias regiões para backups contínuos atualmente][/azure/cosmos-db/continuous-backup-restore-introduction]*. Se não tiver desabilitado as gravações em várias regiões ao criar sua conta do Azure Cosmos DB, você precisará fazer isso agora ou a habilitação do recurso de backup contínuo irá falhar.  Você pode desabilitar as gravações em várias regiões na seção **Replicar dados globalmente** em *Configurações*.

## Excluir um dos documentos de salesOrder

1. No **Data Explorer**, execute a consulta a seguir para obter a data e hora atuais. Copie esse carimbo de data/hora para o bloco de notas. Esse carimbo de data/hora deve estar em UTC.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. No **Data Explorer**, localize o **documento salesOrder** com a **ID** `0019092E-BD25-48F5-8050-7051B2655BC5`. Exclua o Documento e verifique se o documento não está mais lá.

## Restaure o banco de dados até um ponto antes de você ter excluído o documento salesOrder

1. No portal do Azure, navegue até a sua conta do Azure Cosmos DB.

1. Na seção *Configurações*, selecione **Restauração para um Ponto no Tempo**. Use as configurações a seguir:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Ponto de Restauração (UTC)** | Converta a data e a hora adequadamente. O horário precisará estar no formato AM/PM|
    | **Localidade** | *Selecionado um local disponível* |
    | **Selecione os recursos que você gostaria de restaurar** | *Banco de dados/contêineres selecionados* |
    | **Restaurar o Recurso** | *salesOrder* |
    | **Restaurar a Conta de Destino** | *escolha um* ***novo*** *nome de conta do Azure Cosmos DB* |

    > &#128221; Para restaurações do Azure Cosmos DB, você ***nunca*** restaura para uma conta *existente* e sempre precisará criar uma conta nova do Azure Cosmos DB.

    > &#128221; Embora você possa ter optado por restaurar todo o banco de dados ou até mesmo a conta inteira, em um ambiente de produção real os bancos de dados podem ser enormes. Em muitos cenários, pode ser mais rápido restaurar apenas os contêineres ou bancos de dados necessários.

1. Essa restauração pode levar 15 minutos ou mais; vá para a próxima seção e deixe essa restauração sendo executada em segundo plano.

## Exclua o contêiner do cliente

1. No **Data Explorer**, execute a consulta a seguir para obter a data e hora atuais. Copie esse carimbo de data/hora para o bloco de notas.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. Selecione o contêiner do **cliente**.

## Restaure o banco de dados até um ponto antes de você ter excluído o documento salesOrder

1. No portal do Azure, navegue até a sua conta do Azure Cosmos DB.

1. Na seção *Configurações*, selecione **Restauração para um Ponto no Tempo**. Use as configurações a seguir:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Localidade** | *Selecionado um local disponível* |
    | **Ponto de Restauração (UTC)** | Converta a data e a hora adequadamente. O horário precisará estar no formato AM/PM|
    | **Selecione os recursos que você gostaria de restaurar** | *Banco de dados/contêineres selecionados* |
    | **Restaurar o Recurso** | *`customer`* |
    | **Restaurar a Conta de Destino** | *escolha um* ***novo*** *nome de conta do Azure Cosmos DB* |

    > &#128221; Para restaurações do Azure Cosmos DB, você ***nunca*** restaura para uma conta *existente* e sempre precisará criar uma conta nova do Azure Cosmos DB.

    > &#128221; Embora você possa ter optado por restaurar todo o banco de dados ou até mesmo a conta inteira, em um ambiente de produção real os bancos de dados podem ser enormes. Em muitos cenários, pode ser mais rápido restaurar apenas os contêineres ou bancos de dados necessários.

1. Essa restauração pode levar 15 minutos ou mais; vá para a próxima seção e deixe essa restauração sendo executada em segundo plano.

## Reveja os dados restaurados

As restaurações podem levar muito tempo, dependendo do tamanho do banco de dados e de outros fatores. Após as restaurações da conta do Azure Cosmos DB serem concluídas:

1. Para a nossa primeira restauração, certifique-se de que o terceiro documento foi recuperado.

1. Para a segunda restauração, deveríamos ter restaurado a tabela do cliente.

## Limpeza

1. Exclua as duas contas novas do Azure Cosmos DB que foram criadas pelas restaurações da conta.

1. Exclua o banco de dados Vendas e, se necessário, exclua a conta original do Azure Cosmos DB.

[/azure/cosmos-db/continuous-backup-restore-introduction]:https://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction

