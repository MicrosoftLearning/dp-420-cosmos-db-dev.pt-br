---
lab: title: 'Configurar o ambiente de laboratório'' módulo: 'Configuração'
---

# Configuração do ambiente de laboratório local

Idealmente, você deve concluir esses laboratórios em um ambiente de laboratório hospedado. Se você quiser concluí-los em seu próprio computador, isso poderá ser feito instalando o seguinte software. Você pode enfrentar diálogos e comportamento inesperados ao usar seu próprio ambiente. Devido à ampla gama de configurações locais possíveis, a equipe do curso não pode oferecer suporte a problemas que você possa encontrar em seu próprio ambiente.

## Ferramentas de linha de comando do Azure

1. [CLI do Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) ou [Azure Cloud Shell](https://shell.azure.com) – instale se quiser executar comandos por meio da CLI em vez do portal do Azure.

## Python

1. Baixe e instale o Python 3.11+ em [python.org/downloads] com \<install location\>\Python36 e \<install location>\Python36\Scripts adicionados ao seu PATH.

    - Use as opções padrão no instalador.

## Git

1. Baixe e instale do [git-scm.com/downloads].

    - Use as opções padrão no instalador.

## Visual Studio Code (e extensões)

1. Baixe e instale do [code.visualstudio.com/download].

    - Use as opções padrão no instalador.

1. Após a instalação, inicie o Visual Studio Code.

1. No menu **Extensões**, pesquise e instale as seguintes extensões da Microsoft:

    - [Extensão do Python para Visual Studio Code][marketplace.visualstudio.com/mms-python.python]

### Emulador do Azure Cosmos DB

1. Baixe e instale do [docs.microsoft.com/azure/cosmos-db/local-emulator].
    - Use as opções padrão no instalador.

### Clonar o repositório do laboratório

Se você ainda não clonou o repositório de código do laboratório **Criar copilotos com o Azure Cosmos DB** para o ambiente no qual está trabalhando nesse laboratório, siga as seguintes etapas para fazê-lo. Caso contrário, abra a pasta clonada anteriormente no **Visual Studio Code**.

1. Inicie o **Visual Studio Code**.

    > &#128221; Se você ainda não se familiarizou com a interface do Visual Studio Code, confira o [Guia de introdução ao Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra a paleta de comandos e execute **Git: Clone** para clonar o repositório ``https://github.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs`` do GitHub em uma pasta local de sua escolha.

    > &#128161; Você pode usar o atalho de teclado **CTRL+SHIFT+P** para abrir a paleta de comandos.

1. Após o repositório ter sido clonado, abra a pasta local que você selecionou no **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[code.visualstudio.com/download]: https://code.visualstudio.com/download
[git-scm.com/downloads]: https://git-scm.com/downloads
[python.org/downloads]: https://www.python.org/downloads/
[marketplace.visualstudio.com/mms-python.python]: https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview
