---
title: Visão geral do dimensionamento automático em Aplicativos Web, Serviços de Nuvem e Máquinas Virtuais
description: Dimensionamento automático no Microsoft Azure. É aplicável a Máquinas Virtuais, conjuntos de dimensionamento de máquinas virtuais, Serviços de Nuvem e Aplicativos Web.
ms.subservice: autoscale
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: a60c03f1928b38c78a59edca4b5493307d7d19d3
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75364366"
---
# <a name="overview-of-autoscale-in-microsoft-azure-virtual-machines-cloud-services-and-web-apps"></a>Visão geral do dimensionamento automático em Aplicativos Web, Serviços de Nuvem e Máquinas Virtuais do Microsoft Azure
Este artigo descreve o que é dimensionamento automático do Microsoft Azure, seus benefícios e como começar a usá-lo.  

O dimensionamento automático do Azure Monitor aplica-se somente aos [Conjuntos de Dimensionamento de Máquinas Virtuais](https://azure.microsoft.com/services/virtual-machine-scale-sets/), aos [Serviços de Nuvem](https://azure.microsoft.com/services/cloud-services/), ao [Serviço de Aplicativo – Aplicativos Web](https://azure.microsoft.com/services/app-service/web/) e aos [Serviços de Gerenciamento de API](https://docs.microsoft.com/azure/api-management/api-management-key-concepts).

> [!NOTE]
> O Azure tem dois métodos de dimensionamento automático. Uma versão mais antiga do dimensionamento automático aplica-se às Máquinas Virtuais (conjuntos de disponibilidade). Esse recurso tem suporte limitado e recomendamos migrar para os conjuntos de dimensionamento da máquina virtual para obter um suporte mais rápido e confiável do dimensionamento automático. Um link sobre como usar a tecnologia mais antiga será incluído neste artigo.  
>
>

## <a name="what-is-autoscale"></a>O que é dimensionamento automático?
O dimensionamento automático permite ter a quantidade certa de recursos em execução para lidar com a carga em seu aplicativo. Ele permite adicionar recursos para lidar com os aumentos de carga e também economizar dinheiro removendo os recursos que estão ociosos. Você especifica um número mínimo e máximo de instâncias a serem executadas e adiciona ou remove as VMs automaticamente com base em um conjunto de regras. Ter um mínimo assegura que seu aplicativo estará sempre em execução, mesmo sem carga. Ter um máximo limita seu custo por hora total possível. Você dimensiona automaticamente entre esses dois extremos usando as regras criadas.

 ![Dimensionamento automático explicado. Adicionar e remover as VMs](./media/autoscale-overview/AutoscaleConcept.png)

Quando as condições da regra forem atendidas, uma ou mais ações de dimensionamento automático são disparadas. Você pode adicionar e remover as VMs ou executar outras ações. O seguinte diagrama conceitual mostra esse processo.  

 ![Diagrama do Fluxo do Dimensionamento Automático](./media/autoscale-overview/Autoscale_Overview_v4.png)

A explicação a seguir se aplica às partes do diagrama anterior.   

## <a name="resource-metrics"></a>Métricas do recurso
Os recursos emitem métricas, que são processadas posteriormente por regras. As métricas são fornecidas por métodos diferentes.
Os conjuntos de dimensionamento de máquina virtual usam dados de telemetria de agentes de diagnóstico do Azure, enquanto a telemetria para aplicativos Web e serviços de nuvem vêm diretamente da Infraestrutura do Azure. Algumas estatísticas usadas normalmente incluem o Uso da CPU, utilização de memória, contagens de thread, comprimento da fila e uso do disco. Para obter uma lista de quais dados de telemetria você pode usar, confira [Métricas comuns de dimensionamento automático](../../azure-monitor/platform/autoscale-common-metrics.md).

## <a name="custom-metrics"></a>Métricas personalizadas
Você também pode usar suas próprias métricas personalizadas que seus aplicativos podem emitir. Se tiver configurado seu aplicativo para enviar métricas para o Application Insights, você poderá aproveitar essas métricas para tomar decisões quanto ao dimensionamento.

## <a name="time"></a>Hora
Regras com base em agendamento têm base em UTC. Ao configurar as regras, você deve definir o fuso horário corretamente.  

## <a name="rules"></a>Regras
O diagrama mostra apenas uma regra de dimensionamento automático, mas você pode ter muitas deles. Você pode criar regras complexas de sobreposição, conforme o necessário para sua situação.  Os tipos de regra incluem  

* **Com base em métrica** - Por exemplo, executar esta ação quando o uso da CPU estiver acima de 50%.
* **Com base no tempo** - Por exemplo, disparar um webhook todas os sábados às 8h em um determinado fuso horário.

As regras baseadas na métrica medem a carga do aplicativo e adicionam ou removem as VMs com base nessa carga. As regras baseadas no agendamento permitem que você dimensione quando vê os padrões de tempo em sua carga e deseja dimensionar antes que ocorra um possível aumento ou diminuição de carga.  

## <a name="actions-and-automation"></a>Ações e automação
As regras podem disparar um ou mais tipos de ações.

* **Escala** - Aumenta ou diminui as VMs
* **Email** - Envia um email para os administradores e coadministradores da assinatura, e/ou para o endereço de email adicional especificado
* **Automatizar via webhooks** - Chama webhooks, que podem disparar várias ações complexas dentro ou fora do Azure. Dentro do Azure, você pode iniciar um runbook de Automação do Azure, Função do Azure ou Aplicativo Lógico do Azure. Entre os exemplos de URL de terceiros fora do Azure estão serviços como o Slack e o Twilio.

## <a name="autoscale-settings"></a>Configurações de dimensionamento automático
O dimensionamento automático usa a seguinte terminologia e estrutura.

- Uma **configuração do dimensionamento automático** é lido pelo mecanismo de dimensionamento automático para determinar se é para aumentar ou reduzir. Ele contém um ou mais perfis, informações sobre o recurso de destino e as configurações de notificação.

  - Um **perfil de dimensionamento automático** é um combinação de:

    - **configuração de capacidade**, que indica os valores mínimo, máximo e padrão do número de instâncias.
    - **conjunto de regras**, cada um deles inclui um gatilho (tempo ou métrica) e uma ação de dimensionamento (para cima ou para baixo).
    - **recorrência**, que indica quando o dimensionamento automático deve colocar esse perfil em vigor.

      Você pode ter vários perfis, que permitem cuidar dos diferentes requisitos de sobreposição. Você pode ter diferentes perfis de dimensionamento automático para diferentes horas do dia ou dias da semana, por exemplo.

  - Uma **configuração de notificação** define quais notificações devem ocorrer quando acontece um evento de dimensionamento automático com base no atendimento dos critérios de um dos perfis da configuração do dimensionamento automático. O dimensionamento automático pode notificar um ou mais endereços de email ou fazer chamadas para um ou mais webhooks.


![Configuração do dimensionamento automático do Azure, perfil e estrutura de regras](./media/autoscale-overview/AzureResourceManagerRuleStructure3.png)

A lista completa de campos configuráveis e descrições está disponível na [API REST do Dimensionamento Automático](https://msdn.microsoft.com/library/dn931928.aspx).

Para obter exemplos de código, consulte

* [Configuração avançada de escala automática usando modelos do Gerenciador de recursos para conjuntos de escala vm](../../azure-monitor/platform/autoscale-virtual-machine-scale-sets.md)  
* [API REST do Dimensionamento Automático](https://msdn.microsoft.com/library/dn931953.aspx)

## <a name="horizontal-vs-vertical-scaling"></a>Dimensionamento horizontal vs. vertical
O dimensionamento automático ocorre apenas horizontalmente, que é um aumento ("out") ou uma diminuição ("in") do número de instâncias de VM.  O dimensionamento horizontal é mais flexível em uma nuvem, pois permite que você potencialmente execute milhares de VMs para manipular carga.

Já o dimensionamento vertical é diferente. Ele mantém o mesmo número de VMs, mas torna as VMs mais ("para cima") ou menos ("para baixo") potentes. A potência é medida na memória, velocidade da CPU, espaço em disco, etc.  O dimensionamento vertical tem mais limitações. Ele depende da disponibilidade de um hardware maior, que atinge rapidamente um limite superior e pode variar por região. O dimensionamento vertical normalmente também exige que uma VM pare e reinicie.


## <a name="methods-of-access"></a>Métodos de acesso
Você pode configurar o dimensionamento automático via

* [Portal Azure](../../azure-monitor/platform/autoscale-get-started.md)
* [Powershell](../../azure-monitor/platform/powershell-quickstart-samples.md#create-and-manage-autoscale-settings)
* [CLI (Interface de linha de comando) de plataforma cruzada](../../azure-monitor/platform/cli-samples.md#autoscale)
* [Azure Monitor REST API](https://msdn.microsoft.com/library/azure/dn931953.aspx)

## <a name="supported-services-for-autoscale"></a>Serviços com suporte para o dimensionamento automático
| Serviço | Esquema e Documentos |
| --- | --- |
| Aplicativos Web |[Dimensionamento de Aplicativos Web](../../azure-monitor/platform/autoscale-get-started.md) |
| Serviços de Nuvem |[Dimensionamento Automático de um Serviço de Nuvem](../../cloud-services/cloud-services-how-to-scale-portal.md) |
| Máquinas Virtuais: Clássico |[Dimensionamento de conjuntos de disponibilidade da máquina virtual clássica](https://blogs.msdn.microsoft.com/kaevans/2015/02/20/autoscaling-azurevirtual-machines/) |
| Máquinas Virtuais: Conjuntos de Dimensionamento do Windows |[Dimensionamento de conjuntos de dimensionamento de máquina virtual no Windows](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md) |
| Máquinas Virtuais: Conjunto de Dimensionamento do Linux |[Dimensionamento de conjuntos de dimensionamento de máquina virtual no Linux](../../virtual-machine-scale-sets/tutorial-autoscale-cli.md) |
| Máquinas Virtuais: Exemplo para Windows |[Configuração avançada de escala automática usando modelos do Gerenciador de recursos para conjuntos de escala vm](../../azure-monitor/platform/autoscale-virtual-machine-scale-sets.md) |
| Serviço de Gerenciamento de API|[Dimensionar automaticamente uma instância do Gerenciamento de API do Azure](https://docs.microsoft.com/azure/api-management/api-management-howto-autoscale)

## <a name="next-steps"></a>Próximas etapas
Para saber mais sobre o dimensionamento automático, use as Explicações Passo a Passo sobre o Dimensionamento Automático listadas anteriormente ou consulte os recursos a seguir:

* [Métricas comuns de dimensionamento automático do Azure Monitor](../../azure-monitor/platform/autoscale-common-metrics.md)
* [Práticas recomendadas para dimensionamento automático do Azure Monitor](../../azure-monitor/platform/autoscale-best-practices.md)
* [Use ações de autoescala para enviar notificações de alerta de e-mail e webhook](../../azure-monitor/platform/autoscale-webhook-email.md)
* [API REST do Dimensionamento Automático](https://msdn.microsoft.com/library/dn931953.aspx)
* [Solução de Problemas do Dimensionamento Automático dos Conjuntos de Dimensionamento da Máquina Virtual](../../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)

