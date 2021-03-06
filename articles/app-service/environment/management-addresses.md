---
title: Endereços de gerenciamento
description: Encontre os endereços de gerenciamento usados para controlar um Ambiente de Serviço de Aplicativo. Configurou-os em uma tabela de rota para evitar problemas de roteamento assimétricos.
author: ccompy
ms.assetid: a7738a24-89ef-43d3-bff1-77f43d5a3952
ms.topic: article
ms.date: 11/13/2019
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 7d7f97552e8faadee1af928a9ce4e1eea2df476e
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74687105"
---
# <a name="app-service-environment-management-addresses"></a>Endereços de gerenciamento de Ambiente de Serviço de Aplicativo

O App Service Environment (ASE) é uma única implantação de inquilino do Azure App Service que é executado em sua Rede Virtual Azure (VNet).  Embora o ASE seja executado em seu VNet, ele ainda deve estar acessível a partir de vários endereços IP dedicados que são usados pelo Azure App Service para gerenciar o serviço.  No caso de uma ASE, o tráfego de gerenciamento atravessa a rede controlada pelo usuário. Se o tráfego estiver bloqueado ou estiver encaminhado erroneamente, o ASE será suspenso. Para obter detalhes sobre as dependências de rede ASE, leia [considerações de networking e o ambiente de serviço do aplicativo][networking]. Para obter informações gerais sobre o ASE, você pode iniciar com [Introdução ao Ambiente do Serviço de Aplicativo][intro].

Todos os ASEs têm um VIP público no qual o tráfego de gerenciamento entra. O tráfego de gerenciamento de entrada desses endereços é proveniente das portas 454 e 455 no VIP público de seu ASE. Este documento lista os endereços de origem do Serviço de Aplicativo para o tráfego de gerenciamento para o ASE. Esses endereços também estão na tag de serviço IP chamada AppServiceManagement.

Os endereços anotados abaixo podem ser configurados em uma tabela de rota para evitar problemas de roteamento assimétricos com o tráfego de gerenciamento. As rotas atuam no tráfego no nível IP e não têm uma consciência da direção do tráfego ou que o tráfego faz parte de uma mensagem de resposta TCP. Se o endereço de resposta de uma solicitação TCP for diferente do endereço para o qual foi enviado, você terá um problema de roteamento assimétrico. Para evitar problemas de roteamento assimétricos com o tráfego de gerenciamento de ASE, você precisa garantir que as respostas sejam enviadas de volta do mesmo endereço para o qual foram enviadas. Para obter detalhes sobre como configurar o ASE para operar em um ambiente em que o tráfego de saída é enviado localmente, leia [Configurar ASE com túnel forçado][forcedtunnel]

## <a name="list-of-management-addresses"></a>Lista de endereços de gerenciamento ##

| Região | Endereços |
|--------|-----------|
| Todas as regiões públicas | 13.64.115.203, 13.66.140.0, 13.67.8.128, 13.69.64.128, 13.69.227.128, 13.70.73.128, 13.71.170.64, 13.71.194.129, 13.75.34.192, 13.75.127.117, 13.77.50.128, 13.78.109.0, 13.89.171.0, 13.94.141.115, 13.94.143.126, 13.94.149.179, 20.36.106.128, 20.36.114.64, 23.100.226.236, 23.102.135.246, 23.102.188.65, 40.69.106.128, 40.70.146.128, 40.71.13.64, 40.74.100.64, 40.78.194.128, 40.79.130.64, 40.79.178.128, 40.83.120.64, 40.83.121.56, 40.83.125.161, 40.90.240.166, 40.91.126.196, 40.112.242.192, 40.119.4.111, 40.124.47.188, 51.140.146.64, 51.140.210.128, 52.151.25.45, 52.162.80.89, 52.162.106.192, 52.165.152.214, 52.165.153.122, 52.165.154.193, 52.165.158.140, 52.174.22.21, 52.178.177.147, 52.178.184.149, 52.178.190.65, 52.178.195.197, 52.187.56.50, 52.187.59.251, 52.187.63.19, 52.187.63.37, 52.224.105.172, 52.225.177.153, 52.231.18.64, 52.231.146.128, 65.52.14.230, 65.52.172.237, 65.52.193.203, 70.37.57.58, 70.37.89.222, 104.43.242.137, 104.44.129.141, 104.44.129.243, 104.44.129.255, 104.44.134.255, 104.208.54.11, 104.211.81.64, 104.211.146.128, 104.214.49.0, 157.55.176.93, 157.55.208.185, 191.233.203.64, 191.236.154.88 |
| Microsoft Azure Governamental | 23.97.29.209, 13.72.53.37, 13.72.180.105, 23.97.0.17, 23.97.16.184 |

## <a name="configuring-a-network-security-group"></a>Configurando um Grupo de Segurança de Rede

Com os Grupos de Segurança de Rede, você não precisa se preocupar com os endereços individuais ou manter sua própria configuração. Há uma tag de serviço IP chamada AppServiceManagement que é mantida atualizada com todos os endereços. Para usar esta tag de serviço IP em seu NSG, vá para o portal, abra sua ui de grupos de segurança de rede e selecione regras de segurança de entrada. Se você tiver uma regra pré-existente para o tráfego de gerenciamento de entrada, edite-a. Se esse NSG não foi criado com o ASE ou se ele é novo, selecione **Adicionar**. No menu suspenso Origem, selecione **Marca de Serviço**.  Na tag De serviço Origem, selecione **AppServiceManagement**. Defina os intervalos da porta de origem como \*, Destino como **Qualquer**, os intervalos da porta de destino como **454-455**, Protocolo como **TCP** e Ação como **Permitir**. Se você está fazendo a regra, então você precisa definir a Prioridade. 

![criando um NSG com a marca de serviço][1]

## <a name="configuring-a-route-table"></a>Configurando uma tabela de rotas

Os endereços de gerenciamento podem ser colocados em uma tabela de rotas com um próximo salto da Internet para garantir que todo o tráfego de gerenciamento de entrada possa percorrer novamente o mesmo caminho. Essas rotas são necessárias ao configurar o túnel forçado. Para criar a tabela de rotas, use o portal, o PowerShell ou a CLI do Azure.  Os comandos para criar uma tabela de rotas usando a CLI do Azure em um prompt do PowerShell são mostrados abaixo. 

    $rg = "resource group name"
    $rt = "route table name"
    $location = "azure location"
    $managementAddresses = "13.64.115.203", "13.66.140.0", "13.67.8.128", "13.69.64.128", "13.69.227.128", "13.70.73.128", "13.71.170.64", "13.71.194.129", "13.75.34.192", "13.75.127.117", "13.77.50.128", "13.78.109.0", "13.89.171.0", "13.94.141.115", "13.94.143.126", "13.94.149.179", "20.36.106.128", "20.36.114.64", "23.100.226.236", "23.102.135.246", "23.102.188.65", "40.69.106.128", "40.70.146.128", "40.71.13.64", "40.74.100.64", "40.78.194.128", "40.79.130.64", "40.79.178.128", "40.83.120.64", "40.83.121.56", "40.83.125.161", "40.90.240.166", "40.91.126.196", "40.112.242.192", "40.119.4.111", "40.124.47.188", "51.140.146.64", "51.140.210.128", "52.151.25.45", "52.162.80.89", "52.162.106.192", "52.165.152.214", "52.165.153.122", "52.165.154.193", "52.165.158.140", "52.174.22.21", "52.178.177.147", "52.178.184.149", "52.178.190.65", "52.178.195.197", "52.187.56.50", "52.187.59.251", "52.187.63.19", "52.187.63.37", "52.224.105.172", "52.225.177.153", "52.231.18.64", "52.231.146.128", "65.52.14.230", "65.52.172.237", "65.52.193.203", "70.37.57.58", "70.37.89.222", "104.43.242.137", "104.44.129.141", "104.44.129.243", "104.44.129.255", "104.44.134.255", "104.208.54.11", "104.211.81.64", "104.211.146.128", "104.214.49.0", "157.55.176.93", "157.55.208.185", "191.233.203.64", "191.236.154.88"

    az network route-table create --name $rt --resource-group $rg --location $location
    foreach ($ip in $managementAddresses) {
        az network route-table route create -g $rg --route-table-name $rt -n $ip --next-hop-type Internet --address-prefix ($ip + "/32")
    }

Depois de criar a tabela de rotas, você precisará configurá-la na sub-rede do ASE.  

## <a name="get-your-management-addresses-from-api"></a>Obter seus endereços de gerenciamento de API ##

Você pode listar os endereços de gerenciamento que correspondem ao seu ASE com a seguinte chamada à API.

    get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01

A API retorna um documento JSON que inclui todos os endereços de entrada do ASE. A lista de endereços inclui os endereços de gerenciamento, o VIP usado pelo seu ASE e o intervalo de endereços de sub-rede do ASE em si.  

Para chamar a API com o [armclient](https://github.com/projectkudu/ARMClient) use os seguintes comandos, mas substitua a ID da assinatura, o grupo de recursos e o nome do ASE.  

    armclient login
    armclient get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01


<!--IMAGES-->
[1]: ./media/management-addresses/managementaddr-nsg.png

<!-- LINKS -->
[networking]: ./network-info.md
[intro]: ./intro.md
[forcedtunnel]: ./forced-tunnel-support.md
