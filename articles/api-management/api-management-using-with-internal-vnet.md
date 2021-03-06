---
title: Use o Gerenciamento de API do Azure com redes virtuais internas
titleSuffix: Azure API Management
description: Saiba como instalar e configurar o Gerenciamento de API do Azure na rede virtual interna
services: api-management
documentationcenter: ''
author: vladvino
manager: kjoshi
editor: ''
ms.assetid: dac28ccf-2550-45a5-89cf-192d87369bc3
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/31/2019
ms.author: apimpm
ms.openlocfilehash: 6054c595bca26dc2a0432c53369a60a61e3efde0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76841856"
---
# <a name="using-azure-api-management-service-with-an-internal-virtual-network"></a>Usar o serviço de Gerenciamento de API do Azure com rede virtual interna
Com as VNETs (Redes Virtuais) do Azure, o Gerenciamento de API pode gerenciar as APIs que não estão acessíveis pela Internet. Várias tecnologias de VPN estão disponíveis para fazer a conexão. O Gerenciamento de API pode ser implantado em dois modos principais dentro de uma rede virtual:
* Externo
* Interna

Quando o Gerenciamento de API é implantado no modo de rede virtual interna, todos os pontos finais de serviço (o gateway proxy, o portal do Desenvolvedor, o gerenciamento direto e o Git) só são visíveis dentro de uma rede virtual à a que você controla o acesso. Nenhum dos pontos de extremidade de serviço é registrado no servidor DNS público.

> [!NOTE]
> Como não há entradas de DNS para os pontos finais do serviço, esses pontos finais não estarão acessíveis até que o [DNS esteja configurado](#apim-dns-configuration) para a rede virtual.

Usando o Gerenciamento de API no modo interno você pode chegar aos seguintes cenários:

* Torne as APIs hospedadas em seu datacenter privado seguras e acessíveis por terceiros externamente usando as conexões VPN Site a Site ou ExpressRoute do Azure.
* Habilite cenários de nuvem híbrida expondo as APIs baseadas em nuvem e as APIs locais por meio de um gateway comum.
* Gerencie suas APIs hospedadas em várias localizações geográficas usando um único ponto de extremidade de gateway.

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Pré-requisitos

Para executar as etapas descritas neste artigo, você precisa ter:

+ **Uma assinatura ativa do Azure**.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ **Uma instância de Gerenciamento de API do Azure**. Para obter mais informações, consulte [Criar uma instância do Gerenciamento de API do Azure](get-started-create-service-instance.md).
+ Quando um serviço de gerenciamento de API é implantado em uma rede virtual, uma [lista de portas](./api-management-using-with-vnet.md#required-ports) é usada e precisa ser aberta. 

## <a name="creating-an-api-management-in-an-internal-virtual-network"></a><a name="enable-vpn"> </a>Criar um Gerenciamento de API em uma rede virtual interna
O serviço de Gerenciamento de API em uma rede virtual interna está hospedado atrás de um [balanceador de carga interno (clássico)](https://docs.microsoft.com/azure/load-balancer/load-balancer-get-started-ilb-classic-cloud). Esta é a única opção disponível e não pode ser alterada.

### <a name="enable-a-virtual-network-connection-using-the-azure-portal"></a>Habilite uma conexão de rede virtual usando o portal do Azure

1. Navegue até sua instância de Gerenciamento de API do Azure no [portal do Azure](https://portal.azure.com/).
2. Selecione **rede virtual**.
3. Configure a instância de Gerenciamento de API a ser implantada dentro da rede virtual.

    ![Menu para configurar um Gerenciamento de API do Azure em uma rede virtual interna][api-management-using-internal-vnet-menu]

4. Selecione **Salvar**.

Após o sucesso da implantação, você deve ver endereço IP virtual **privado** e endereço IP virtual **público** do serviço de gerenciamento de API na lâmina de visão geral. O endereço IP virtual **privado** é um endereço IP balanceado de `gateway`carga `portal` `management` de `scm` dentro da sub-rede delegada de gerenciamento de API sobre a qual , e pontos finais podem ser acessados. O endereço IP virtual **público** é usado `management` **apenas** para controlar o tráfego de aviões para endpoint sobre a porta 3443 e pode ser bloqueado na tag de serviço [apiManagement.][ServiceTags]

![Painel de Gerenciamento de API com uma rede virtual interna configurada][api-management-internal-vnet-dashboard]

> [!NOTE]
> O console de Teste disponível no portal do Azure não funcionará para o serviço implantado do VNET **Interno**, pois a URL do Gateway não está registrada no DNS público. É necessário usar o Console de Teste fornecido no **portal do Desenvolvedor**.

### <a name="enable-a-virtual-network-connection-by-using-powershell-cmdlets"></a>Habilitar uma conexão de rede virtual usando cmdlets do PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Você também pode habilitar a conectividade de rede virtual usando cmdlets do PowerShell.

* Crie um serviço de gerenciamento de API dentro de uma rede virtual: Use o cmdlet [New-AzApiManagement](/powershell/module/az.apimanagement/new-azapimanagement) para criar um serviço de gerenciamento de API azure dentro de uma rede virtual e configurá-lo para usar o tipo de rede virtual interna.

* Atualize uma implantação existente de um serviço de gerenciamento de API dentro de uma rede virtual: Use o cmdlet [Update-AzApiManagementRegion](/powershell/module/az.apimanagement/update-azapimanagementregion) para mover um serviço de gerenciamento de API existente dentro de uma rede virtual e configurá-lo para usar o tipo de rede virtual interna.

## <a name="dns-configuration"></a><a name="apim-dns-configuration"></a>Configuração de DNS
Quando o Gerenciamento de API está no modo de rede virtual externa, o DNS é gerenciado pelo Azure. Para o modo de rede virtual interna, você precisa gerenciar o seu próprio roteamento.

> [!NOTE]
> O serviço de Gerenciamento de API não escuta as solicitações que vêm de endereços IP. Ele só responde às solicitações para o nome de host configurado em seus pontos de extremidade de serviço. Esses pontos de extremidade incluem o gateway, o portal do Azure, o portal do Desenvolvedor, o ponto de extremidade de gerenciamento direto e o Git.

### <a name="access-on-default-host-names"></a>Acesso em nomes de host padrão
Quando você cria um serviço de gerenciamento de API, chamado "contosointernalvnet", por exemplo, os seguintes pontos finais de serviço são configurados por padrão:

   * Gateway ou proxy: contosointernalvnet.azure-api.net

   * O portal desenvolvedor: contosointernalvnet.portal.azure-api.net

   * O novo portal de desenvolvedores: contosointernalvnet.developer.azure-api.net

   * Ponto final da gestão direta: contosointernalvnet.management.azure-api.net

   * Git: contosointernalvnet.scm.azure-api.net

Para acessar esses pontos de extremidade do serviço de Gerenciamento de API, você pode criar uma máquina virtual em uma sub-rede conectada à rede virtual na qual o Gerenciamento de API está implantado. Supondo que o endereço IP virtual interno do seu serviço seja 10.1.0.5, você pode mapear o arquivo hosts, %SystemDrive%\drivers\etc,como segue:

   * 10.1.0.5 contosointernalvnet.azure-api.net

   * 10.1.0.5 contosointernalvnet.portal.azure-api.net

   * 10.1.0.5 contosointernalvnet.developer.azure-api.net

   * 10.1.0.5 contosointernalvnet.management.azure-api.net

   * 10.1.0.5 contosointernalvnet.scm.azure-api.net

Então você pode acessar todos os pontos de extremidade do serviço da máquina virtual criada.
Se você usar um servidor DNS personalizado em uma rede virtual, também poderá criar registros DNS A e acessar esses pontos de extremidade de qualquer lugar em sua rede virtual.

### <a name="access-on-custom-domain-names"></a>Acessar em nomes de domínio personalizados

1. Se você não quiser acessar o serviço de Gerenciamento de API com os nomes de host padrão, poderá configurar nomes de domínio personalizados para todos os seus pontos de extremidade de serviço, conforme mostra a imagem abaixo:

   ![Configurar o domínio personalizado para Gerenciamento de API][api-management-custom-domain-name]

2. Em seguida, você pode criar registros no seu servidor DNS para acessar os pontos de extremidade que só estão acessíveis pela rede virtual.

## <a name="routing"></a><a name="routing"> </a> Roteamento

* Um endereço IP virtual *privado* balanceado de carga a partir da faixa de sub-rede será reservado e usado para acessar os pontos finais do serviço de gerenciamento de API de dentro da rede virtual. Este endereço IP *privado* pode ser encontrado na lâmina Visão Geral para o serviço no portal Azure. Este endereço deve ser registrado nos servidores DNS usados pela rede virtual.
* Um endereço IP *público* (VIP) de carga também será reservado para fornecer acesso ao ponto final do serviço de gerenciamento sobre a porta 3443. Este endereço IP *público* pode ser encontrado na lâmina Visão Geral para o serviço no portal Azure. O endereço IP *público* é usado apenas `management` para controlar o tráfego de aviões até o ponto final sobre a porta 3443 e pode ser bloqueado na tag de serviço [ApiManagement.][ServiceTags]
* Os endereços IP da faixa IP (DIP) da sub-rede serão atribuídos a cada VM no serviço e serão usados para acessar recursos dentro da rede virtual. Um endereço IP público (VIP) será usado para acessar recursos fora da rede virtual. Se as listas de restrição de IP forem usadas para proteger recursos dentro da rede virtual, toda a gama para a sub-rede onde o serviço de gerenciamento de API é implantado deve ser especificada para conceder ou restringir o acesso do serviço.
* Os endereços IP públicos e privados de carga podem ser encontrados na lâmina Visão Geral no portal Azure.
* Os endereços IP atribuídos para acesso público e privado podem ser alterar se o serviço for removido e, em seguida, adicionado de volta à rede virtual. Se isso acontecer, pode ser necessário atualizar registros de DNS, regras de roteamento e listas de restrição de IP dentro da rede virtual.

## <a name="related-content"></a><a name="related-content"> </a>Conteúdo relacionado
Para saber mais, leia os seguintes artigos:
* [Problemas comuns de configuração de rede ao configurar o Gerenciamento de API do Azure em uma rede virtual][Common network configuration problems]
* [Perguntas frequentes sobre rede virtual](../virtual-network/virtual-networks-faq.md)
* [Criando um registro no DNS](/previous-versions/windows/it-pro/windows-2000-server/bb727018(v=technet.10))

[api-management-using-internal-vnet-menu]: ./media/api-management-using-with-internal-vnet/api-management-using-with-internal-vnet.png
[api-management-internal-vnet-dashboard]: ./media/api-management-using-with-internal-vnet/api-management-internal-vnet-dashboard.png
[api-management-custom-domain-name]: ./media/api-management-using-with-internal-vnet/api-management-custom-domain-name.png

[Create API Management service]: get-started-create-service-instance.md
[Common network configuration problems]: api-management-using-with-vnet.md#network-configuration-issues

[ServiceTags]: ../virtual-network/security-overview.md#service-tags

