---
lab:
  title: Medir o desempenho das entidades do cliente
  module: Module 8 - Implement a data modeling and partitioning strategy for Azure Cosmos DB for NoSQL
---

# Medir o desempenho das entidades do cliente

Neste exercício, você medirá a diferença das entidades de cliente quando você modelar as entidades como contêineres separados em vez de modelá-las para um banco de dados NoSQL inserindo as entidades em um documento individual.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não se familiarizou com a interface do Visual Studio Code, confira o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Depois que o repositório tiver sido clonado, abra a pasta local selecionada no **Visual Studio Code**.

1. No **Visual Studio Code**, no painel **Explorer**, navegue até a pasta **16-measure-performance**.

1. Abra o menu de contexto da pasta **16-measure-performance** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

1. Se o terminal for aberto como um terminal do **Windows PowerShell**, abra um novo terminal do **Git Bash**.

    > &#128161; Para abrir um terminal do **Git Bash**, no lado direito do menu do terminal, clique no menu suspenso ao lado do sinal **+** e escolha *Git Bash*.

1. No **terminal do Git Bash**, execute os comandos a seguir. Os comandos abrem uma janela do navegador para se conectar ao portal do azure, onde você usará as credenciais de laboratório fornecidas.

    ```
    "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\python.exe" -m pip install pip-system-certs
    az login
    cd 16-measure-performance
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1

    ```
    > &#128161; Se você executou o laboratório de **Custo de desnormalização de dados** primeiro e não removeu os recursos do Azure criados por esse laboratório, feche o terminal integrado, ignore a etapa a seguir e vá para a próxima seção. Observe que, se você já tiver os recursos criados pelo laboratório de **Custo de desnormalizar dados** e tentar executar o script abaixo, o script falhará.

1. No **terminal do Git Bash**, execute os comandos a seguir. Os comandos executam um script que cria uma nova conta do Azure Cosmos DB e, em seguida, criam e iniciam o aplicativo que você usa para preencher o banco de dados e concluir os exercícios. *Depois de inserir a credencial fornecida para a conta do Azure, o build poderá levar de 15 a 20 minutos para ser concluído, portanto, talvez seja uma boa hora para tomar um café ou chá*.

    ```
    bash init.sh
    dotnet build
    dotnet run --load-data
    echo "Data load process completed."

    ```
1. Feche o terminal integrado.

## Medir o desempenho de entidades em contêineres separados

No Database-v1, os dados são armazenados em contêineres individuais. Nesse banco de dados, execute as consultas para obter as entidades Customer, CustomerAddress e CustomerPassword. Examine o preço da solicitação de cada uma dessas consultas.

### Consultar a entidade de cliente

No Database-v1, execute uma consulta para obter a entidade Customer e examinar o preço da solicitação.

1. Em uma nova janela ou guia do navegador da Web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. No menu do portal do Azure ou na **Home page**, selecione **Azure Cosmos DB**.
1. Selecione a conta do Azure Cosmos DB com o nome que começa com **cosmicworks**.
1. Selecione **Data Explorer** no lado esquerdo.
1. Expanda **Database-v1**.
1. Selecione o contêiner **Customer**.
1. Na parte superior da tela, selecione **Nova consulta SQL**.
1. Copie e cole o texto SQL a seguir e selecione **Executar consulta**.

    ```
    SELECT * FROM c WHERE c.id = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. Escolha a guia **Estatísticas de Consulta** e anote o preço da solicitação de 2,83.

    ![Captura de tela que mostra as estatísticas de consulta para consulta de cliente no banco de dados.](media/17-customer-query-v1.png)

### Consultar o endereço do cliente

Execute uma consulta para obter a entidade de endereço do cliente e examinar o preço da solicitação.

1. Selecione o contêiner **CustomerAddress**.
1. Na parte superior da tela, selecione **Nova consulta SQL**.
1. Copie e cole o texto SQL a seguir e selecione **Executar consulta**.

    ```
    SELECT * FROM c WHERE c.customerId = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. Escolha a guia **Estatísticas de Consulta** e anote o preço da solicitação de 2,83.

    ![Captura de tela que mostra as estatísticas de consulta para a consulta de endereço do cliente no banco de dados.](media/17-customer-address-query-v1.png)

### Consultar a senha do cliente

Execute uma consulta para obter a entidade de senha do cliente e examinar o preço da solicitação.

1. Selecione o contêiner **CustomerPassword**.
1. Na parte superior da tela, selecione **Nova consulta SQL**.
1. Copie e cole o texto SQL a seguir e selecione **Executar consulta**.

    ```
    SELECT * FROM c WHERE c.id = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. Escolha a guia **Estatísticas de Consulta** e anote o preço da solicitação de 2,83.

    ![Captura de tela que mostra as estatísticas de consulta para consulta de senha do cliente no banco de dados.](media/17-customer-password-query-v1.png)

### Somar os custos da solicitação

Agora que executamos todas as consultas, vamos somar todos os custos das Unidades de Solicitação delas.

|**Consulta**|**Custo de RU/s**|
|---------|---------|
|Customer|2,83|
|Endereço do Cliente|2,83|
|Senha do cliente|2,83|
|**Total de RU/s**|**8,49**|

## Medir o desempenho de entidades inseridas

Agora, vamos consultar as mesmas informações, mas com as entidades inseridas em um documento individual.

1. Escolha o banco de dados **Database-v2**.
1. Selecione o contêiner **Customer**.
1. Execute a consulta a seguir. 

    ```
    SELECT * FROM c WHERE c.id = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. Observe que os dados retornados agora são uma hierarquia de dados de cliente, endereço e senha.

    ![Captura de tela que mostra os resultados da consulta para cliente no banco de dados.](media/17-customer-query-v2.png)

1. Selecione **Estatísticas de Consulta**. Anote o preço da solicitação de 2,83 em comparação com as RU/s de 8,49 das três consultas executadas anteriormente.

## Comparar o desempenho dos dois modelos

Ao comparar as RU/s de cada consulta executada, você verá que a última consulta em que as entidades de cliente estão em um documento individual é muito menos cara do que o custo combinado da execução das três consultas de modo independente. A latência de retorno desses dados é menor porque os dados são retornados em uma só operação.

Quando você pesquisar um item individual e souber a chave de partição e a ID dos dados, pode recuperar esses dados por meio de uma *leitura de ponto* chamando `ReadItemAsync()` no SDK do Azure Cosmos DB. Uma leitura de ponto é ainda mais rápida do que a consulta. Para os mesmos dados do cliente, o custo é de apenas 1 RU/s, o que é uma melhoria quase três vezes maior.

## Limpar

Exclua o Grupo de Recursos criado neste laboratório.  Se você não tiver acesso para remover o Grupo de Recursos, remova todos os objetos do Azure criados por este laboratório.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
