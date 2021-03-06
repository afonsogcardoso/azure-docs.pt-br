---
title: IBM DB2 pureScale no Azure
description: Neste artigo, mostramos uma arquitetura para executar um ambiente IBM DB2 pureScale no Azure.
author: njray
manager: edprice
editor: edprice
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.workload: infrastructure-services
ms.topic: article
ms.date: 11/09/2018
ms.author: edprice
ms.openlocfilehash: d8309a69c9c38610fa7bea3fee202a60d836980c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78945049"
---
# <a name="ibm-db2-purescale-on-azure"></a>IBM DB2 pureScale no Azure

O ambiente IBM DB2 pureScale fornece um cluster de banco de dados para o Azure com alta disponibilidade e escalabilidade nos sistemas operacionais Linux. Este artigo mostra uma arquitetura para executar o DB2 pureScale no Azure.

## <a name="overview"></a>Visão geral

As empresas há muito tempo têm usado plataformas tradicionais de sistema de gerenciamento de banco de dados relacional (RDBMS) para atender às suas necessidades de processamento de transações on-line (OLTP). Atualmente, muitos estão migrando seus ambientes de banco de dados baseados em mainframe para o Azure como uma forma de expandir a capacidade, reduzir custos e manter uma estrutura de custo operacional estável. A migração é frequentemente o primeiro passo na modernização de uma plataforma legada. 

Recentemente, um cliente corporativo reorganizou seu ambiente IBM DB2 em execução no z/OS para o IBM DB2 pureScale no Azure. A solução de cluster de banco de dados Db2 pureScale fornece alta disponibilidade e escalabilidade em sistemas operacionais Linux. O cliente executou o Db2 com sucesso como uma instância de escala autônoma em uma única máquina virtual (VM) em um sistema de grande escala no Azure antes de instalar o Db2 pureScale. 

Apesar de não ser idêntico ao ambiente original, o IBM DB2 pureScale no Linux oferece recursos semelhantes de alta disponibilidade e escalabilidade como o IBM DB2 para z/OS em execução em uma configuração de Parallel Sysplex no mainframe. Neste cenário, o cluster é conectado via iSCSI a um cluster de armazenamento compartilhado. Usamos o sistema de arquivos GlusterFS, um sistema de arquivos distribuídos gratuito, escalável e de código aberto especificamente otimizado para armazenamento em nuvem. No entanto, a IBM não suporta mais essa solução. Para manter seu suporte da IBM, você precisa usar um sistema de arquivos compatível com iSCSI suportado. A Microsoft oferece o Storage Spaces Direct (S2D) como opção

Este artigo descreve a arquitetura usada para essa migração do Azure. O cliente usou o Red Hat Linux 7.4 para testar a configuração. Esta versão está disponível no Azure Marketplace. Antes de escolher uma distribuição do Linux, não se esqueça de verificar as versões com suporte no momento. Para obter detalhes, consulte a documentação para [IBM DB2 pureScale](https://www.ibm.com/support/knowledgecenter/SSEPGG) e [GlusterFS](https://docs.gluster.org/en/latest/).

Este artigo é um ponto de partida para seu plano de implementação do DB2. Seus requisitos de negócios serão diferentes, mas o mesmo padrão básico se aplica. Você também pode usar esse padrão arquitetural para aplicativos OLAP (processamento analítico online) no Azure.

Este artigo não cobre diferenças e possíveis tarefas de migração para mover um banco de dados IBM DB2 para z/OS para o IBM DB2 pureScale em execução no Linux. Tampouco fornece estimativas de dimensionamento e análises de carga de trabalho para passar do DB2 z/OS para o DB2 pureScale. 

Para ajudá-lo a decidir sobre a melhor arquitetura DB2 pureScale para seu ambiente, é recomendável que você faça uma estimativa completa do dimensionamento e crie uma hipótese. No sistema de origem, considere DB2 z/OS Parallel Sysplex com arquitetura de compartilhamento de dados, configuração do Coupling Facility e estatísticas de uso do DDF (facilitador de dados distribuídos).

> [!NOTE]
> Este artigo descreve uma abordagem à migração DB2, mas há outras. Por exemplo, o DB2 pureScale também pode ser executado em ambientes locais virtualizados. A IBM dá suporte para o DB2 no Microsoft Hyper-V em várias configurações. Para obter mais informações, consulte [a arquitetura de virtualização DB2 pureScale](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/r0061462.html) no IBM Knowledge Center.

## <a name="architecture"></a>Arquitetura

Para oferecer suporte à alta disponibilidade e escalabilidade no Azure, é possível usar uma arquitetura de dados compartilhada em expansão pode ser usada para o DB2 pureScale. A migração do cliente usou o exemplo de arquitetura a seguir.

![DB2 pureScale em máquinas virtuais Do Zure mostrando armazenamento e rede](media/db2-purescale-on-azure/pureScaleArchitecture.png "DB2 pureScale em máquinas virtuais Do Zure mostrando armazenamento e rede")


O diagrama mostra as camadas lógicas necessárias para um cluster DB2 pureScale. Isso inclui máquinas virtuais para um cliente, para gerenciamento, para armazenamento em cache, para o mecanismo de banco de dados e para armazenamento compartilhado. 

Além dos nós do Mecanismo de Banco de Dados, o diagrama inclui dois nós usados para CFs (recursos de armazenamento em cache) do cluster. Um mínimo de dois nós são usados para o próprio mecanismo de banco de dados. Um servidor DB2 que pertence a um cluster pureScale é chamado de membro. 

O cluster é conectado via iSCSI a um cluster de armazenamento compartilhado de três nós para fornecer armazenamento em escala e alta disponibilidade. O DB2 pureScale é instalado em máquinas virtuais do Azure que executam o Linux.

Essa abordagem é um modelo que você pode modificar para o tamanho e escala da sua organização. É baseada no seguinte:

-   dois ou mais membros do banco de dados são combinados com pelo menos dois nós do CF. Os nós gerenciam um GBP (pool de buffers global) para serviços de memória compartilhada e GLM (gerenciador de bloqueio global) para controlar o acesso compartilhado e bloquear a contenção de membros ativos. Um nó CF atua como primário e o outro como o nó CF secundário de failover. Para evitar um ponto único de falha no ambiente, um cluster de pureScale DB2 requer pelo menos quatro nós.

-   Armazenamento compartilhado de alto desempenho (mostrado em tamanho P30 no diagrama). Cada nó usa este armazenamento.

-   Rede de alto desempenho para os membros de dados e armazenamento compartilhado.

### <a name="compute-considerations"></a>Considerações de computação

Essa arquitetura executa os níveis de aplicativo, armazenamento e dados nas máquinas virtuais do Azure. Os [scripts de configuração de implantação](https://aka.ms/db2onazure) criam o seguinte:

-   Um cluster DB2 pureScale. O tipo de recursos de computação necessários no Azure depende da sua configuração. Em geral, é possível usar duas abordagens:

    -   Use uma rede no estilo de computação de alto desempenho e vários nós (HPC), em que instâncias de pequeno a médio porte acessam o armazenamento compartilhado. Para esse tipo de HPC de configuração, [máquinas virtuais](https://docs.microsoft.com/azure/virtual-machines/windows/sizes) do Azure E-series otimizadas para memória ou L-series otimizadas para armazenamento fornecem a potência de computação necessária.

    -   Use menos instâncias grandes de máquinas virtuais para os mecanismos de dados. Para instâncias grandes, as maiores máquinas virtuais [M-series](https://azure.microsoft.com/pricing/details/virtual-machines/series/) com otimização de memória são ideais para cargas de trabalho com muita memória. Você pode precisar de uma instância dedicada, dependendo do tamanho da LPAR (partição lógica) que é usada para executar o DB2.

-   O DB2 CF usa máquinas virtuais otimizadas para memória, como E-series ou L-series.

-   Um cluster de armazenamento\_compartilhado que\_usa máquinas virtuais Padrão DS4 v2 executando o Linux.

-   O jumpbox de\_gerenciamento é\_uma máquina virtual Padrão DS2 v2 executando o Linux.  Uma alternativa é o Azure Bastion, um serviço que oferece uma experiência RDP/SSH segura para todas as VMs em sua rede virtual.

-   O cliente é uma máquina virtual padrão \_DS3\_ v2 executando o Windows (usada para teste).

-   *Opcional*. Um servidor de testemunhas. Isso é necessário apenas com algumas versões anteriores do Db2 pureScale. Este exemplo usa\_uma\_máquina virtual Padrão DS3 v2 executando o Linux (usado para DB2 pureScale).

> [!NOTE]
> Um cluster DB2 pureScale requer pelo menos duas instâncias de DB2. Também requer uma instância de cache e uma instância do gerenciador de bloqueio.

### <a name="storage-considerations"></a>Considerações de armazenamento

Como o Oracle RAC, o DB2 pureScale é um banco de dados de expansão de E/S de bloco de alto desempenho. Recomendamos o uso da maior opção de [SSD Premium do Azure](disks-types.md) que atenda às suas necessidades. Opções de armazenamento menores podem ser adequadas para ambientes de desenvolvimento e teste, enquanto ambientes de produção geralmente precisam de maior capacidade de armazenamento. A arquitetura de exemplo usa [P30](https://azure.microsoft.com/pricing/details/managed-disks/) devido à sua proporção de IOPS para tamanho e preço. Independentemente do tamanho, use o Armazenamento Premium para obter o melhor desempenho.

O DB2 pureScale usa uma arquitetura de tudo compartilhado, na qual todos os dados são acessíveis de todos os nós do cluster. O armazenamento premium deve ser compartilhado em várias instâncias, seja demanda ou em instâncias dedicadas.

Um grande cluster DB2 pureScale pode exigir 200 TB (terabytes) ou mais de armazenamento compartilhado Premium, com IOPS de 100.000. O DB2 pureScale dá suporte a uma interface de bloco iSCSI que pode ser usada no Azure. A interface iSCSI requer um cluster de armazenamento compartilhado que você pode implementar com o S2D ou outra ferramenta. Esse tipo de solução cria um dispositivo de rede de área de armazenamento virtual (vSAN) no Azure. O DB2 pureScale usa o vSAN para instalar o sistema de arquivos em cluster que é usado para compartilhar dados entre máquinas virtuais.

### <a name="networking-considerations"></a>Considerações de rede

A IBM recomenda a rede InfiniBand para todos os membros em um cluster DB2 pureScale. O DB2 pureScale também usa RDMA (acesso remoto direto à memória), quando disponível, para os CFs.

Durante a instalação, você cria um [grupo de recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) do Azure para conter todas as máquinas virtuais. Em geral, os recursos são agrupados com base no tempo de vida e em quem vai gerenciá-los. As máquinas virtuais nessa arquitetura exigem [rede acelerada](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/). É um recurso do Azure que fornece latência de rede consistente e ultrabaixa por meio de SR-IOV (virtualização de E/S de raiz única) para uma máquina virtual.

Cada máquina virtual do Azure é implantada em uma rede virtual que tem sub-redes: main, front-end Gluster FS (gfsfe), back-end Gluster FS (bfsbe), DB2 pureScale (db2be) e front-end dB2 purescale (db2fe). O script de instalação também cria as [NICs](https://docs.microsoft.com/azure/virtual-machines/linux/multiple-nics) primárias nas máquinas virtuais na sub-rede principal.

Use [grupos de segurança de rede](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) para restringir o tráfego de rede dentro da rede virtual e para isolar as sub-redes.

No Azure, o DB2 pureScale precisa usar TCP/IP como a conexão de rede para armazenamento.

## <a name="next-steps"></a>Próximas etapas

-   [Implemente esta arquitetura no Azure](deploy-ibm-db2-purescale-azure.md)
