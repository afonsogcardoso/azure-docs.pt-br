---
title: Visão geral do SAP HANA no Azure (Instâncias Grandes) | Microsoft Docs
description: Visão geral de como implantar o SAP HANA no Azure (Instâncias Grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 39fcf5d0fe2273c4debd3ae5ebe5fd1190ddc959
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77616962"
---
#  <a name="what-is-sap-hana-on-azure-large-instances"></a>O que é SAP HANA no Azure (Instâncias Grandes)?

SAP HANA do Azure (Instâncias Grandes) é uma solução exclusiva para o Azure. Além de fornecer máquinas virtuais para implantar e executar o SAP HANA, o Azure oferece a possibilidade de executar e implantar o SAP HANA em servidores bare-metal dedicados a você. A solução do SAP HANA do Azure (Instâncias Grandes) baseia-se no hardware bare-metal de host/servidor não compartilhado que é atribuído a você. O hardware de servidor é inserido em selos maiores que contêm infraestrutura de computação/servidor, de rede e de armazenamento. Como uma combinação, é a TDI (integração de data center personalizada) certificada do HANA. O SAP HANA do Azure (Instâncias Grandes) oferece diferentes tamanhos ou SKUs de servidor. As unidades podem ter 36 núcleos de CPU Intel e 768 GB de memória, além de unidades que têm até 480 núcleos de CPU Intel e até 24 TB de memória.

O isolamento do cliente dentro do carimbo de infraestrutura é executado em locatários, o que é semelhante a:

- **Rede**: isolamento dos clientes dentro da pilha de infraestrutura por meio de redes virtuais por locatário atribuído pelo cliente. Um locatário é atribuído a um único cliente. Um cliente pode ter vários locatários. O isolamento de rede dos locatários proíbe a comunicação de rede entre locatários no nível do carimbo de infraestrutura, mesmo se os locatários pertencerem ao mesmo cliente.
- **Componentes de armazenamento**: isolamento por meio de máquinas virtuais de armazenamento que têm volumes de armazenamento atribuídos a elas. Volumes de armazenamento podem ser atribuídos a somente uma máquina virtual de armazenamento. Uma máquina virtual de armazenamento é atribuída exclusivamente a um único locatário na pilha de infraestrutura certificada pela TDI do SAP HANA. Como resultado, os volumes de armazenamento atribuídos a uma máquina virtual de armazenamento podem ser acessados apenas em um locatário específico e relacionado. Eles não são visíveis entre os diferentes locatários implantados.
- **Servidor ou host**: uma unidade de host ou servidor não é compartilhada entre os clientes ou locatários. Um servidor ou host, implantado para um cliente, é uma unidade de computação bare-metal atômica que é atribuída a um único locatário. *Nenhum* particionamento de hardware ou particionamento flexível é usado, o que pode resultar no compartilhamento de um host ou de um servidor com outro cliente. Volumes de armazenamento que são atribuídos à máquina virtual de armazenamento do locatário específico são montados para um servidor desse tipo. Um locatário pode ter unidades de servidor de um para muitos de diferentes SKUs atribuídos de modo exclusivo.
- Em um carimbo de infraestrutura do SAP HANA do Azure (Instâncias Grandes), muitos locatários diferentes são implantados e isolados uns em relação a outros por meio dos conceitos de locatário no nível de computação, armazenamento e rede. 


Essas unidades de servidor bare-metal têm suporte para executar apenas o SAP HANA. A camada de aplicativo do SAP ou a camada de middleware de carga de trabalho executa em máquinas virtuais. Os carimbos de infraestrutura que executam o SAP HANA em unidades do Azure (Instâncias Grandes) são conectados aos backbones de serviços de rede do Azure. Dessa maneira, é fornecida uma conectividade de baixa latência entre as unidades do SAP HANA do Azure (Instâncias Grandes) e as máquinas virtuais.

A partir de julho de 2019, diferenciamos entre duas revisões diferentes de selos HANA Large Instance e localização de implantações:

- "Revisão 3" (Rev 3): São os selos que foram disponibilizados para o cliente ser implantado antes de julho de 2019
- "Revisão 4" (Rev 4): Novo design de selo que é implantado próximo aos hosts Azure VM e que até agora são lançados nas regiões azure de:
    -  Oeste dos EUA 2 
    -  Leste dos EUA 
    -  Europa Ocidental
    -  Norte da Europa


Este documento é um dos vários documentos que cobrem o SAP HANA do Azure (Instâncias Grandes). Este documento apresenta a arquitetura básica, as responsabilidades e os serviços fornecidos pela solução. Recursos de alto nível da solução também são abordados. Para a maioria das outras áreas, como rede e conectividade, quatro outros documentos cobrem detalhes e informações detalhadas. A documentação do SAP HANA do Azure (Instâncias Grandes) não abrange aspectos da instalação ou das implantações do SAP NetWeaver do SAP NetWeaver nas VMs. O SAP NetWeaver no Azure é abordado em documentos separados, localizados no mesmo contêiner de documentação do Azure. 


Os diferentes documentos de diretrizes de Instância grande do HANA abordam as seguintes áreas:

- [Visão geral e arquitetura sap hana (instâncias grandes) no Azure](hana-overview-architecture.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Infra-estrutura e conectividade SAP HANA (Grandes Instâncias) no Azure](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Instalar e configurar o SAP HANA (Instâncias Grandes) do Azure](hana-installation.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [SAP HANA (Grandes Instâncias) alta disponibilidade e recuperação de desastres no Azure](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Solução de problemas e monitoramento do SAP HANA (Instâncias Grandes) do Azure](troubleshooting-monitoring.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Configuração de alta disponibilidade no SUSE usando STONITH](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/ha-setup-with-stonith)
- [Backup e restauração do Sistema Operacional para SKUs tipo II de selos de Revisão 3](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/os-backup-type-ii-skus)

**Próximas etapas**
- Veja [Conhecer os termos](hana-know-terms.md)