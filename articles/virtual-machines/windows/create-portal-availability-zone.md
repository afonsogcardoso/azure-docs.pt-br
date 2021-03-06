---
title: Crie uma VM windows zoneada com o portal Azure
description: Crie uma VM do Windows em uma Região de Disponibilidade com o portal do Azure
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: ''
ms.openlocfilehash: c6f843693ccfa16f9b31027ba370242e0462b138
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74033887"
---
# <a name="create-a-windows-virtual-machine-in-an-availability-zone-with-the-azure-portal"></a>Crie uma máquina virtual do Windows em uma Região de Disponibilidade com o portal do Azure

Este artigo aborda usando o portal do Azure para criar uma máquina virtual em uma região de disponibilidade do Azure. Uma [zona de disponibilidade](../../availability-zones/az-overview.md) é uma zona fisicamente separada em uma região do Azure. Use zonas de disponibilidade para proteger seus aplicativos e dados de uma improvável falha ou perda de um datacenter inteiro.

Para usar uma zona de disponibilidade, crie a máquina virtual em uma [região do Azure com suporte](../../availability-zones/az-overview.md#services-support-by-region).

## <a name="sign-in-to-azure"></a>Entrar no Azure 

Entre no Portal do Azure em https://portal.azure.com.

## <a name="create-virtual-machine"></a>Criar máquina virtual

1. Clique **em Criar um recurso** no canto superior esquerdo do portal Azure.

2. Selecione **Computação** e, em seguida, selecione **Windows Server 2016 Datacenter**. 

3. Insira as informações da máquina virtual. O nome do usuário e a senha inseridos aqui são usados para fazer logon na máquina virtual. A senha deve ter no mínimo 12 caracteres e atender a [requisitos de complexidade definidos](faq.md#what-are-the-password-requirements-when-creating-a-vm). Escolha um local como Leste dos EUA 2 que oferece suporte a zonas de disponibilidade. Quando concluído, clique em **OK**.

    ![Insira as informações básicas sobre sua VM na folha do portal](./media/create-portal-availability-zone/create-windows-vm-portal-basic-blade.png)

4. Escolha um tamanho para a VM. Selecione um tamanho recomendado ou filtro com base nos recursos. Confirme se que o tamanho está disponível na zona que você deseja usar.

    ![Selecione um tamanho de VM](./media/create-portal-availability-zone/create-windows-vm-portal-sizes.png)  

5. Em **Configurações** > **De alta disponibilidade,** selecione uma das zonas numeradas da menu de **disponibilidade** suspensa, mantenha os demais padrões e clique em **OK**.

    ![Selecione uma zona de disponibilidade](./media/create-portal-availability-zone/create-windows-vm-portal-availability-zone.png)

6. Na página de resumo, clique em **Criar** para iniciar a implantação da máquina virtual.

7. A VM será fixada ao painel do portal do Azure. Depois que a implantação for concluída, a folha de resumo da VM abrirá automaticamente.

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>Confirme a zona do disco gerenciado e endereço IP

Quando a VM é implantada em uma zona de disponibilidade, um disco gerenciado para a VM é criado na mesma zona de disponibilidade. Por padrão, um endereço IP público também é criado nessa região.

Você pode confirmar as configurações de zona para esses recursos no portal.  

1. Clique em **Grupos de recursos** e, em seguida, no nome do grupo de recursos da VM, como *myResourceGroup*.

2. Clique no nome do recurso de disco. A página **Visão geral** inclui detalhes sobre a zona local e de disponibilidade do recurso.

    ![Zona de disponibilidade do disco gerenciado](./media/create-portal-availability-zone/create-windows-vm-portal-disk.png)

3. Clique no nome do recurso de endereço IP público. A página **Visão geral** inclui detalhes sobre a zona local e de disponibilidade do recurso.

    ![Zona de disponibilidade do endereço IP](./media/create-portal-availability-zone/create-windows-vm-portal-ip.png)



## <a name="next-steps"></a>Próximas etapas

Neste artigo, você aprendeu a criar uma VM em uma zona de disponibilidade. Saiba mais sobre [a disponibilidade](availability.md) para VMs Azure.
