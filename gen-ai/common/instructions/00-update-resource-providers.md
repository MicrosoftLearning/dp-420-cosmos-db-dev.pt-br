---
title: Habilitar provedores de recursos
lab:
  title: Habilitar provedores de recursos
  module: Setup
layout: default
nav_order: 2
parent: Common setup instructions
---

# Habilitar provedores de recursos do Azure

Há alguns provedores de recursos que devem ser registrados em sua assinatura do Azure. Siga estas etapas para garantir que eles sejam registrados.

1. Em uma nova guia ou janela do navegador da web, navegue até o portal do Azure (``portal.azure.com``).

1. Entre no portal usando as credenciais da Microsoft associadas à sua assinatura.

1. Na página **Inicial**, selecione **Assinaturas**.

    > &#128161; Alternativamente; expanda o **&#8801;** menu, selecione **Todos os Serviços** e, na categoria **Todos**, selecione **Assinaturas**.

1. Selecione sua assinatura do Azure.

    > &#128221; Se você tiver várias assinaturas, selecione a que você criou resgatando o Azure Pass.

1. Na folha de sua assinatura, na seção**Configurações**, selecione **Provedores de recursos**.

1. Na lista de provedores de recursos, verifique se os seguintes provedores estão registrados:
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; Se um provedor não estiver registrado, selecione esse provedor e selecione **Registrar**.

1. Feche a janela ou a guia do navegador da Web.

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
