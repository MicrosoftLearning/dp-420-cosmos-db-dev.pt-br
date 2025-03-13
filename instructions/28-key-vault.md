---
lab:
  title: Armazenar chaves de conta do Azure Cosmos DB for NoSQL no Azure Key Vault
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Armazenar chaves de conta do Azure Cosmos DB for NoSQL no Azure Key Vault

Adicionar um código de conexão de conta do Azure Cosmos DB ao seu aplicativo é tão simples quanto fornecer o URI e as chaves da conta. Essas informações de segurança poderão, às vezes, estar incluídas no código do aplicativo. No entanto, se o seu aplicativo estiver sendo implantado no Serviço de Aplicativo do Azure, você poderá salvar as informações criptografadas da conexão no Azure Key Vault.

Nesse laboratório, vamos criptografar e armazenar a cadeia de conexão da conta do Azure Cosmos DB no Azure Key Vault. Em seguida, vamos criar um aplicativo web do Serviço de Aplicativo do Azure que irá recuperar essas credenciais do Azure Key Vault. O aplicativo irá usar essas credenciais e se conectar à conta do Azure Cosmos DB. Em seguida, o aplicativo irá criar alguns documentos nos contêineres da conta do Azure Cosmos DB e retornar o respectivo status para uma página da web.

## Preparar seu ambiente de desenvolvimento

Se você ainda não clonou o repositório de código do laboratório do **DP-420** para o ambiente no qual está trabalhando nesse laboratório, siga essas etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221: Se ainda não estiver familiarizado com a interface do Visual Studio Code, leia o [Guia de Introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Após o repositório ter sido clonado, ***FECHE*** o *Visual Studio Code*. Mais tarde, iremos abri-lo apontando diretamente para a pasta **28-key-vault**.

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

    > &#128221; Seus ambientes de laboratório podem ter restrições impedindo que você crie um novo grupo de recursos. Se for esse o caso, use o grupo de recursos pré-criado existente.

1. Aguarde a conclusão da tarefa de implantação antes de continuar com essa tarefa.

1. Vá para o recurso de conta do **Azure Cosmos DB** recém-criado e navegue até o painel **Chaves**.

1. Este painel contém os detalhes da conexão e as credenciais necessárias para se conectar à conta do SDK. Especificamente, o campo **PRIMARY CONNECTION STRING**. Você usará esse valor de **cadeia de conexão** posteriormente nesse exercício.

## Criar um Azure Key Vault e armazenar as credenciais da conta do Azure Cosmos DB como um segredo

Antes de criarmos nosso aplicativo web, vamos proteger a cadeia de conexão da conta do Azure Cosmos DB ao copiá-las para um *segredo* criptografado do *Azure Key Vault*. Faremos isso agora.

1. Em uma nova guia do navegador, navegue até o portal do Azure e abra a página **Key Vaults**.

1. Adicione um cofre selecionando o botão ***+ Criar*** e preencha o cofre com as configurações a seguir, *deixando todas as configurações restantes com seus valores padrão*. Em seguida, selecione criar o cofre:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Assinatura** | *Sua assinatura existente do Azure* |
    | **Grupo de recursos** | *Selecionar um grupo de recursos existente ou criar um novo* |
    | **Nome do cofre de chaves** | *Insira um nome globalmente exclusivo* |
    | **Região** | *Escolha qualquer região disponível* |
    | **Acesse o modelo de configuração/permissão** | *Política de acesso ao cofre* |
    | **Acesse as políticas de configuração/acesso** | *selecione a caixa de seleção do nome de usuário atual* |

    > &#128221; Observe que, em um ambiente de produção, você provavelmente selecionaria o controle RBAC em vez da política de Acesso ao Cofre e seu administrador provavelmente atribuirá a você a função RBAC adequada para limitar seu acesso ao Key Vault.

1. Após o cofre ter sido criado, navegue até o cofre.

1. Na seção *Objetos*, selecione **Segredos**.

1. Selecione **+ Gerar/Importar** para criptografar nossa cadeia de conexão de credenciais e preencha os valores do *segredo* com as configurações a seguir, *deixando todas as configurações restantes com seus valores padrão*. Em seguida, selecione criar o segredo:

    | **Configuração** | **Valor** |
    | ---: | :--- |
    | **Opções de upload** | *Manual* |
    | **Nome** | *O nome com o qual você irá rotular seu segredo* |
    | **Valor** | *Esse é o campo mais importante que você irá preencher. Copie aqui o valor de PRIMARY CONNECTION STRING da seção de chave da sua conta do Azure Cosmos DB. Esse valor será convertido em um segredo.* |
    | **Enabled** | *Sim* |
 
1. Em Segredos, agora você deverá ver seu novo segredo listado. Precisamos obter o *identificador do segredo* que iremos adicionar ao código do nosso aplicativo web. Selecione o **segredo** que você criou.

1. O Azure Key Vault permite que você crie várias versões do seu segredo, mas para os fins desse laboratório, só precisamos de uma versão. Selecione a **Versão atual**.

1. Registre o valor do campo **Identificador do segredo**. Esse é o valor que usaremos no código do nosso aplicativo para obter o segredo do Key Vault.  Observe que esse valor é uma URL. Há mais uma etapa que precisamos executar para que esse segredo funcione corretamente, mas vamos executá-la um pouco mais tarde.

## Criar um aplicativo web do Serviço de Aplicativo do Azure

Vamos criar um aplicativo web que irá se conectar à conta do Azure Cosmos DB e criar alguns contêineres e documentos. Não vamos incluir as *credenciais* do Azure Cosmos DB no código desse aplicativo; em vez disso, vamos incluir no código o **Identificador do Segredo** do cofre de chaves. Veremos como esse identificador é inútil sem os direitos adequados atribuídos ao aplicativo web na camada do Azure. Vamos começar a escrever o código.



1. Abra o **Visual Studio Code**.  Abra a pasta **28-key-vault** selecionando Arquivo->Abrir a pasta e navegando até a pasta **28-key-vault**.

    > &#128221; Observe que você deve ver apenas a pasta **28-key-vault** e os respectivos arquivos e subpastas na árvore do **Explorer**. Se conseguir ver o repositório inteiro do GitHub que clonamos anteriormente, ***Feche o Visual Studio Code*** e o reabra diretamente para a pasta **28-key-vault**.  O aplicativo web não funcionará corretamente se esse diretório não for seu diretório raiz de projetos, então certifique-se de que você possa ver somente a pasta **28-key-vault** e os respectivos arquivos e subpastas na árvore do **Explorer**.

1. Abra o menu de contexto da pasta **28-key-vault** e, em seguida, selecione **Abrir no Terminal Integrado** para abrir uma nova instância do terminal.

    > &#128221; Esse comando abrirá o terminal com o diretório inicial já definido como a pasta **28-key-vault**.

1. Vamos criar um shell do aplicativo web MVC. Vamos substituir alguns dos arquivos gerados daqui a pouco. Execute o comando a seguir para criar o aplicativo web:

    ```
    dotnet new mvc
    ```


    > &#128221; Esse comando criou o shell de um aplicativo web e, portanto, adicionou vários arquivos e diretórios. Já temos alguns arquivos com todo o código de que precisamos. 

1. Substitua os arquivos **.\Controllers\HomeController.cs** e **.\Views\Home\Index.cshtml** pelos respectivos arquivos do diretório **.\KeyvaultFiles**.

1. Após substituir os arquivos, ***EXCLUA*** o diretório **.\KeyvaultFiles**.

## Importar as várias bibliotecas ausentes para o script do .NET

A CLI do .NET inclui um comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar pacotes de um feed de pacotes pré-configurado. Uma instalação do .NET usa o NuGet como seu feed de pacotes padrão.

1. Adicione o pacote [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Adicione o pacote [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] do NuGet usando o seguinte comando:

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. Adicione o pacote [Microsoft.Azure.KeyVault][nuget.org/packages/Microsoft.Azure.KeyVault] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. Adicione o pacote [Microsoft.Azure.Services.AppAuthentication][nuget.org/packages/Microsoft.Azure.Services.AppAuthentication] do NuGet usando o seguinte comando:

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.6.2
    ```

## Como adicionar o Identificador do Segredo ao seu aplicativo web

1. No Visual Studio, abra o arquivo `.\Controllers\HomeControler.cs`

1. A função **GetKeyVaultSecret** definida pelo usuário obterá o segredo da conta do Azure Cosmos DB. A função começa na *linha 98* e deve se parecer com o script abaixo.

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. Vamos rever as chamadas importantes que essa função efetua.

    - Na *linha 100*, definimos o token do aplicativo web atual. Esse token será fornecido ao Azure Key Vault para identificar qual aplicativo está tentando acessar o cofre. 
    - Nas *linhas 104-105*, preparamos o *Cliente do Key Vault* que se conectará ao Azure Key Vault. Observe que enviamos o token do aplicativo web como um parâmetro. 
    - Nas *linhas 107-108*, fornecemos ao Cliente do Key Vault o endereço da URL do nosso **Identificador do Segredo** que retornaria o segredo armazenado nesse cofre de chaves. 

1.  Antes de podermos implantar nosso aplicativo web, ainda precisaremos enviar a URL do **Identificador do Segredo**.  Na *linha 107*, substitua a cadeia de caracteres ***<Key Vault Secret Identifier>*** pela URL do **Identificador do Segredo** que registramos na seção *segredo* e salve o arquivo.

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## (Opcional) Instalar a Extensão dos Serviços de Aplicativos do Azure

No Visual Studio, se você abrir a paleta de comandos (**CTRL+SHIFT+P**) e ela não retornar nada quando você pesquisar comandos do Recurso do Aplicativo Azure, vamos precisar instalar a extensão.

1. No menu do lado esquerdo do Visual Studio Code, selecione a opção **Extensões**.

1. Na barra de pesquisa, procure o Serviço de Aplicativo do Azure e o selecione.

1. Selecione o botão Instalar para instalá-lo.

1. Feche a guia **Extensões** e volte para o seu código.

## Implantar seu aplicativo no Serviço de Aplicativo do Azure

O resto do código é muito direto: obter a cadeia de conexão, conectar-se ao Azure Cosmos DB e adicionar alguns documentos. O aplicativo também deve nos fornecer um feedback sobre quaisquer problemas. Não deveremos precisar fazer mais nenhuma alteração após a implantação o aplicativo. Vamos começar. 

> &#128221; A maioria das etapas abaixo será executada na paleta de comandos (**CTRL+SHIFT+P**) na metade superior da tela do Visual Studio. 

1. No Visual Studio Code, abra a paleta de comandos e procure o ***Serviço de Aplicativo do Azure: Criar um novo aplicativo web ... (Avançado) ***

1. Selecione ***Entrar no Azure...***. Essa opção irá abrir uma janela do navegador da web, seguir o processo de login, fechar o navegador quando terminar e retornar ao Visual Studio Code.

1. (Opcional) Se sua assinatura for solicitada, selecione sua assinatura.

1. Insira um nome exclusivo globalmente para o aplicativo Web.

1. Selecione um Grupo de Recursos existente ou crie um novo, se necessário.

1. Selecione **.NET 8 (LTS)**.

1. Selecione **Windows**.

1. Selecione um local disponível.

1. Selecione **+ Criar um novo Plano do Serviço de Aplicativo**.

1. Aceite o nome padrão para o Plano do Serviço de Aplicativo (deve ter o mesmo que o seu aplicativo web) ou escolha um nome novo.

1. Selecione **Experimente o Azure Gratuitamente (F1) sem nenhum custo**.

1. Selecione **Ignorar por enquanto** para o Application Insights.

1. A implantação agora deve estar em execução com uma barra de status no canto inferior direito. 

1. Selecione **Implantar** quando solicitado.

1. Selecione **Procurar**, e você deve estar dentro da pasta **28-key-vault**. Selecione essa pasta.

1. Um pop-up com a mensagem **Uma configuração obrigatória para a implantação está ausente do "28-key-vault"** deve aparecer. Selecione o botão Adicionar Configuração.  Essa opção criará a pasta `.vscode` ausente.

    > &#128221; Muito importante: se esse pop-up não aparecer na sua primeira implantação do aplicativo, alguns arquivos estarão ausentes do upload para os Serviços de Aplicativos do Azure. A implantação será bem-sucedida, mas o site sempre retornará a mensagem *Você não tem permissão para ver esse diretório ou página.* A causa mais provável para isso é que o Visual Studio Code foi aberto no repositório Clonado do GitHub, em vez de apenas na pasta **28-key-vault**.

1. Selecione **Sim** quando solicitado a sempre implantar nesse workspace.

1. Selecione **Procurar site** quando solicitado.  Alternativamente, abra um navegador e vá para **`https://<yourwebappname>.azurewebsites.net`**. Em ambos os casos, teremos um problema. Você verá uma mensagem definida pelo usuário na nossa página da web. A mensagem deverá ser, **O Key Vault não estava acessível**, com uma mensagem de erro mais ampla. Vamos corrigir isso.

## Permitir que o nosso aplicativo use uma identidade gerenciada

O primeiro problema que precisaremos corrigir é permitir que o nosso aplicativo use uma identidade gerenciada. O uso de uma identidade gerenciada permitirá que nosso aplicativo use Serviços do Azure como o Azure Key Vault.

1. Abra seu navegador e faça login no portal do Azure.

1. Abra a página **Serviços de Aplicativos**. O nome do seu aplicativo Web deve estar listado. Selecione-o.

1. Na seção *Configurações*, selecione **Identidade**.

1. Em Status, selecione **Ativado** e **Salvar**.  Selecione **Sim**, se for solicitado a habilitar a *Identidade Gerenciada Atribuída*.

1. Vamos experimentar nosso aplicativo web novamente.  No seu navegador, vá para **`https://<yourwebappname>.azurewebsites.net`**.

1. Ainda temos um problema. Embora a primeira mensagem seja uma mensagem definida pelo usuário que o nosso programa está enviando, a segunda é uma mensagem gerada pelo Sistema. O que a segunda mensagem significa é que nos foi concedido acesso à conexão com o Key Vault, mas não o acesso para ver o segredo dentro do cofre.  Vamos definir uma configuração final para corrigir esse problema.

## Como conceder ao nosso aplicativo web uma política de acesso aos segredos do Key Vault

O objetivo original desse laboratório era impedir que nossas Contas do Azure Cosmos DB fossem incluídas no código dos nossos aplicativos. Mas nós incluímos no código a URL do nosso **Identificador do Segredo**, que qualquer pessoa pode ver. Então, o podemos fazer para proteger nossas credenciais? A boa notícia é que o Identificador do Segredo isoladamente é inútil. O **Identificador do Segredo** leva você somente até a porta do Azure Key Vault, mas o cofre decide quem entra e quem fica na porta. O que isso significa é que precisaremos criar uma política de acesso ao Key Vault para que o nosso aplicativo possa ver os segredos nesse cofre. Vamos examinar essa solução.

1. (Opcional) Antes de criarmos a política, vamos rever o conteúdo atual do nosso banco de dados do Azure Cosmos DB.  No portal do Azure, vá para a sua conta do Azure Cosmos DB. Você está vendo um Banco de Dados **GlobalCustomers**? Se não estiver presente, o banco de dados será criado pela execução bem-sucedida do aplicativo web. Se estiver presente, reveja o número de itens no banco de dados; a execução bem-sucedida do aplicativo web irá adicionar mais itens.

1. No portal do Azure, vá para o Key Vault que criamos anteriormente.

1. Na seção *Configurações*, selecione **Configuração de acesso**.

1. Certifique-se de que a **Política de acesso ao cofre** esteja selecionada e selecione **Ir para as políticas de acesso**.

1. Selecione **+ Criar**.

1. Na guia **Permissões**, selecione a caixa de seleção **Obter** de **Permissões de chave** e **Permissões do segredo** e selecione **Avançar**.

1. Na guia **Entidade de Segurança**, na caixa de pesquisa, insira o nome que você deu ao seu Serviço de Aplicativo, selecione-o na lista e, a seguir, selecione **Avançar**.

1. Na guia **Aplicativo (opcional),** selecione **Avançar**.
    
1. Na guia **Revisar + criar**, selecione **Criar**.

1. Vamos experimentar nosso aplicativo web novamente.  No seu navegador, vá para **`https://<yourwebappname>.azurewebsites.net`**.

1. Êxito! Nossa página da web deve indicar que inserimos novos itens no contêiner do cliente. Também podemos ver o Segredo real sendo exibido.

    > &#128221; Em um ambiente de produção, **nunca** mostre o segredo. Fizemos isso apenas para fins de ilustração.


1. Vá para a sua conta do Azure Cosmos DB e verifique se você tem um novo banco de dados **GlobalCustomers** contendo dados, ou se o banco de dados já existia, verifique se agora contém mais itens.

Agora usamos o Azure Key Vault com sucesso para proteger as chaves da sua conta do Azure Cosmos DB.
