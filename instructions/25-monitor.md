---
lab:
  title: Usar o Azure Monitor para analisar uma conta do Azure Cosmos DB for NoSQL
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Usar o Azure Monitor para analisar uma conta do Azure Cosmos DB for NoSQL

O Azure Monitor é um serviço de monitoramento de pilha completo no Azure que fornece um conjunto completo de recursos para monitorar os recursos do Azure.  O Azure Cosmos DB cria dados de monitoramento usando o Azure Monitor.  O Azure Monitor captura os dados de métricas e telemetria do Cosmos DB.

Neste laboratório, você executará uma carga de trabalho simulada em contêineres do Azure Cosmos DB e analisará como essa carga de trabalho afeta a conta do Azure Cosmos DB.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não se familiarizou com a interface do Visual Studio Code, confira o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Depois que o repositório tiver sido clonado, abra a pasta local selecionada no **Visual Studio Code**.

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
    | **Aplicar Desconto na Camada Gratuita** | *Não aplicar* |
    | **Limitar a quantidade total de taxa de transferência que pode ser provisionada nesta conta** | *Desmarcar* |

    > &#128221; Seus ambientes de laboratório podem ter restrições que impedem a criação de um grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Esse painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta a partir do SDK. Especificamente:

    1. Observe o campo **URI**. Você usará esse valor de **ponto de extremidade** mais adiante neste exercício.

    1. Observe o campo **CHAVE PRIMÁRIA**. Você usará esse valor de **chave** mais adiante neste exercício.

1. Minimize, mas não feche a janela do navegador. Voltaremos ao portal do Azure alguns minutos depois de iniciarmos uma carga de trabalho em segundo plano nas próximas etapas.


## Importar as bibliotecas Microsoft.Azure.Cosmos e Newtonsoft.Json para um script .NET

A CLI do .NET inclui um comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar pacotes de um feed de pacote configurado previamente. Uma instalação do .NET usa o NuGet como o feed de pacote padrão.

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **25-monitor**.

1. Abra o menu de contexto da pasta **25-monitor** e selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Este comando abrirá o terminal com o diretório inicial já definido como a pasta **25-monitor**.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Adicione o pacote [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

## Executar um script para criar os contêineres e a carga de trabalho

Agora estamos prontos para executar uma carga de trabalho para monitorar o uso da conta do Azure Cosmos DB.  O script que vamos executar, nos bastidores. Esse script criará três contêineres e carregará alguns dados neles. Em seguida, o script executará algumas consultas SQL aleatoriamente para emular vários aplicativos de usuário que atingem a conta do Azure Cosmos DB. 

1. No **Visual Studio Code**, no painel do **Explorer**, navegue até a pasta **25-monitor**.

1. Abra o arquivo de código **Program.cs**.

1. Atualize a variável existente chamada **ponto de extremidade** com o seu valor definido como o **ponto de extremidade** da conta do Azure Cosmos DB criada anteriormente.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por exemplo, se o ponto de extremidade for **https&shy;://dp420.documents.azure.com:443/**, a instrução C# será **ponto de extremidade de cadeia de caracteres somente leitura estático privado = "https&shy;://dp420.documents.azure.com:443/";**.

1. Atualize a variável existente nomeada **chave** com o seu valor definido como a **chave** da conta do Azure Cosmos DB criada anteriormente.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Por exemplo, se a chave for **fDR2ci9QgkdkvERTQ==**, a instrução C# será **private static readonly string key = "fDR2ci9QgkdkvERTQ==";**.

1. Salve o arquivo **Program.cs**.

1. Volte ao *Terminal Integrado*.

1. Crie e execute o projeto usando o comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```
    > &#128221; A primeira parte desse script criará os três contêineres e carregará os dados neles. Isso vai demorar cerca de 2 minutos. Para emular alguns eventos de limitação de taxa, o script define a taxa de transferência provisionada como 400 RU/s. Em seguida, você deverá receber a mensagem ***Criando carga de trabalho em segundo plano simulada. Aguarde de 5 a 10 minutos e vá para a próxima etapa do exercício.*** Como os recursos do Azure carregam os dados de monitoramento para o Azure Monitor de maneira assíncrona, precisamos aguardar um pouco para começar a obter alguns dados de diagnóstico nas Métricas e nos Insights do Azure Monitor. Após 5 a 10 minutos, vá para a próxima etapa. Se desejar, para coletar dados de diagnóstico adicionais, não será necessário interromper o script após 5 a 10 minutos e apenas aguardar até o final do laboratório para fazer isso.

    > &#128221; Você observará alguns avisos em amarelo, pois o compilador detecta que o script executa muitas operações de maneira síncrona e não aguarda uma resposta das operações. Você pode ignorar esses avisos, pois esse é o comportamento esperado para a execução de vários scripts SQL simultaneamente.

## Usar o Azure Monitor para analisar o uso da conta do Azure Cosmos DB

Nesta parte do exercício, vamos voltar ao navegador e revisar alguns dos relatórios dos Insights e das Métricas do Azure Monitor.

### Relatórios das Métricas do Azure Monitor

1. Volte à janela aberta do navegador que minimizamos anteriormente. Se você a fechou, abra uma nova e vá para a página de conta do Azure Cosmos DB em portal.azure.com.

1. No menu à esquerda do Azure Comsos DB, em *Monitoramento*, selecione **Métricas**. Você vai observar que os campos **Escopo** e **Namespace da Métrica** são preenchidos previamente com as informações corretas. Nas etapas a seguir, vamos dar uma olhada em algumas opções de **Métrica** e nos recursos *Adicionar filtro* e *Aplicar divisão*.

1. Por padrão, a seção *Métricas* nos mostrará as informações de diagnóstico das últimas 24 horas. Precisamos ter uma granularidade maior para analisar as métricas durante a carga de trabalho que criamos na etapa anterior. No canto superior direito, selecione o botão rotulado ***Hora local: Últimas 24 horas (Automático)*** e obteremos uma janela com várias opções de intervalo de tempo no botão de opção.  Escolha o botão de opção rotulado ***Últimos 30 minutos*** e selecione o botão **Aplicar**. Se necessário, você pode ter muito mais granularidade escolhendo o botão de opção *Personalizado* e escolhendo uma data e uma hora de início e término. 

1. Agora que temos um bom intervalo de tempo para nossos gráficos de diagnóstico, vamos dar uma olhada em algumas métricas. Começaremos com uma métrica comum. Na lista suspensa *Métrica*, escolha **Total de Unidades de Solicitação**. Por padrão, essa métrica será exibida como a soma total de RUs. Se preferir, altere a lista suspensa Agregação para avg ou max. Depois de marcar essas duas agregações, defina-a de volta como *Soma* para as etapas a seguir.

1. Essa métrica nos dá uma boa ideia de quantas unidades de solicitações foram usadas em nossa conta do Azure Cosmos DB. No entanto, nosso gráfico atual pode não nos ajudar a criar um problema quando temos vários bancos de dados ou contêineres na conta. Vamos alterar isso e revisar como nosso consumo de RU foi feito pelo banco de dados. No menu sob o título char, selecione **Aplicar divisão**, na lista suspensa **Valores**, escolha **DatabaseName** e selecione qualquer lugar no gráfico para aceitar as alterações. Um botão **Dividir por = DatabaseName** agora será colocado logo acima do gráfico. 

1. Ficou muito melhor, pois agora sabemos qual banco de dados está fazendo a maior parte do trabalho. Embora essas informações sejam boas, não temos ideia de qual contêiner está fazendo todo o trabalho.  Selecione o botão **Dividir por = DatabaseName** para alterar a condição de Divisão e escolha **CollectionName** na lista suspensa *Valores*. Ótimo! Agora teremos os dados para as coleções **customer** e **salesOrder**. Só há um problema nesse gráfico: a coleção **salesOrder** está em dois bancos de dados, **database-v2** e **database-v3**. Portanto, esse valor é uma agregação do nome dessa coleção nos dois bancos de dados.

1. Essa será uma correção fácil: selecione o botão **Adicionar filtro**, na lista suspensa *Propriedades*, escolha **DatabaseName** e, em *Valores*, escolha **database-V3**.

1. Vamos dar uma olhada em mais duas métricas. Editaremos o gráfico existente e você também poderá criar um gráfico, se desejar. Acima do gráfico, selecione o botão com o *nome da conta do Azure Cosmos DB* e o rótulo **Total de Unidades de Solicitação**. Escolha **Total de Solicitações** na lista suspensa *Métrica*, observe que a única agregação disponível é *Contagem*.

1. Dois filtros de chave aqui podem nos ajudar a solucionar diferentes tipos de problemas. Vamos adicionar um filtro com o **StatusCode** da propriedade (observe que um filtro semelhante com um tipo diferente de detalhe será **Status**). Em *Valores*, escolha **200** e **429**. Altere a Divisão para que ela use StatusCode. Observe que há uma grande quantidade de mensagem com o status 429, solicitações de limitação de taxa, em comparação com o status 200, solicitações bem-sucedidas. As exceções com o status 429 ocorreram porque o script está enviando milhares de solicitações por segundo enquanto definimos a taxa de transferência provisionada como 400 RU/s. *Esse grande número de exceções com o status 429 em comparação com a solicitação bem-sucedida não deve ser normal em um ambiente de produção. Em um ambiente de produção, as exceções com o status 429 devem ocorrer com pouca frequência em uma conta íntegra do Azure Cosmos DB*.  Você também pode usar **StatusCode** ou **Status** *Properties* de maneira semelhante à solução de problemas em **Total de Unidades de Solicitação**

1. Vamos continuar analisando **Total de Solicitações**, mas vamos alterar a divisão para **OperationType**.  Essa propriedade nos ajudará a determinar as operações de leitura ou gravação que estão fazendo a maior parte do trabalho. Novamente, essa propriedade também pode ser usada da mesma forma em **Total de Unidades de Solicitação**

1. Como fizemos com o **Total de Unidades de Solicitação**, experimente escolher filtros e opções de divisão diferentes. 

1. A métrica final que vamos analisar neste exercício é a métrica **Consumo de RU Normalizada**. Altere a divisão para **PartitionKeyRangeId**. Essa métrica nos ajuda a identificar o uso do intervalo de chaves de partição que está mais quente. A métrica nos dá a distorção da taxa de transferência para um intervalo de chaves de partição. Vá em frente e escolha essa métrica na lista suspensa *Métrica*. Este gráfico agora nos mostrará um sistema muito não íntegro, que atinge um **Consumo de RU Normalizada** constante de 100%.

> &#128221; Se você quiser analisar mais de um gráfico por vez, clique na opção **+ Novo Gráfico** acima do nome do gráfico. 

> &#128221; Embora não possamos salvar diretamente as métricas, você pode criar ou usar um painel existente e adicionar esse gráfico a ele clicando no botão **Fixar no painel** no canto superior direito do gráfico.  Clique no botão e escolha a guia **Criar**, dê a ela o nome *Laboratórios DP-420* e clique em **Criar e fixar**. Para exibir seus painéis privados, acesse o menu do portal no canto superior esquerdo e escolha Painel nas opções de recursos do Azure. O painel pode levar alguns minutos para aparecer na primeira vez.

> &#128221; Mais uma forma de compartilhar seu gráfico é clicar na lista suspensa Compartilhar e baixá-la como um arquivo do Excel ou a opção Copiar link.

### Relatórios de Insights do Azure Monitor

Talvez seja necessário gastar algum tempo ajustando nossos relatórios de diagnóstico das Métricas do Azure Monitor.  Os Insights do Cosmos DB apresentam uma visão do desempenho geral, das falhas e da integridade operacional dos recursos do Azure Cosmos DB. Esses gráficos dos Insights serão gráficos criados previamente semelhantes aos da Métrica. Vamos dar uma olhada em alguns deles.

1. No menu à esquerda do Azure Comsos DB, em *Monitoramento*, selecione **Insights**. Você observará que há várias guias, de Visão Geral a Opções de Gerenciamento. Vamos dar uma olhada em alguns desses gráficos do **Insight**. A primeira guia, a guia Visão geral, fornece um resumo dos gráficos mais comuns que você pode usar. Por exemplo, gráficos como total de solicitações, uso de dados e índice, 429 exceções e consumo de RU normalizada.  Vimos a maioria desses gráficos na seção anterior.

1. Observe que, na parte superior dos gráficos, podemos entrar no **Intervalo de Tempo**. Portanto, selecione *15* ou *30* minutos para avaliar a carga de trabalho neste exercício.

1. No canto superior direito de *cada* gráfico, você vai notar uma opção para ***Abrir o Metric Explorer***. Vamos em frente e selecionar a opção  **Abrir o Metric Explorer** do gráfico **Total de Solicitações** . Você observará que, ao selecionar essa opção, será levado aos relatórios de Métrica que vimos anteriormente. A vantagem de abrir o Metric Explorer é que uma boa parte do gráfico já foi criada para nós.

1. Vamos voltar à página Insights selecionando o **X** no canto superior direito do gráfico de Métricas.

1. Selecione a guia Taxa de Transferência. Esses gráficos são bons para identificar problemas de taxa de transferência.  Preste muita atenção ao gráfico **Consumo de RU Normalizada (%) por PartitionKeyRangeID**, que pode ser usado para detectar partições ativas.

1. Selecione a guia Solicitações. Esses gráficos são ótimos para analisar o número de eventos de limitação dos quais a conta tem experiência (429 vs. 200) ou para analisar o número de solicitações por tipo de operação.  

1. Selecione a guia Armazenamento. Esses gráficos nos mostram o crescimento das nossas coleções e o uso de dados e do índice.  

1. Selecione a guia Sistema. Se o aplicativo estava criando, excluindo ou consultando os metadados das contas com frequência, é possível que haja exceções do status 429.  Esses gráficos nos ajudam a determinar se esse acesso frequente aos metadados é a causa das exceções com o status 429. Além disso, podemos determinar o status das solicitações de metadados.  

### Relatórios de Insights do Azure Monitor

1. Se o programa ainda estiver em execução, volte ao terminal de comando do Visual Studio Code.

1. Feche o terminal integrado.

1. Feche o **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[nuget.org/packages/Newtonsoft.Json/13.0.1]: https://www.nuget.org/packages/Newtonsoft.Json/13.0.1
