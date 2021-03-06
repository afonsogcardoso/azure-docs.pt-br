---
title: Publicar oferta de aplicativos do Azure | Mercado Azure
description: Descreva o processo e etapas para publicar uma oferta do aplicativo do Azure no Azure Marketplace.
author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 01/25/2019
ms.author: dsindona
ms.openlocfilehash: 9f89e31c2d17ef74971d2057ba58e9572e92184c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80280092"
---
# <a name="publish-azure-application-offer"></a>Publicar oferta de aplicativo do Azure

Você pode publicar uma oferta logo após criá-la fornecendo as informações na página **Nova Oferta**. Selecione **publicar** para iniciar o processo de publicação.

O diagrama a seguir mostra as principais etapas no processo de publicação para uma oferta para "entrar no ar".

![Etapas de publicação da oferta](./media/offer-publishing-steps.png)


## <a name="detailed-description-of-publishing-steps"></a>Descrição detalhada de etapas de publicação

A tabela a seguir lista e descreve cada etapa de publicação, além de fornecer um tempo estimado para a conclusão de cada etapa.  Estimativas de tempo em "dias" são definidas como dias úteis, que não incluem finais de semana e feriados.

|  **Etapa de publicação**           | **Tempo**    | **Descrição**                                                            |
|  -------------------           | --------    | ---------------                                                            |
| Validar os pré-requisitos         | < 15 min    | Informações de oferta e configurações de oferta são validadas.                        |
| Validar configurações de receita influenciadas | < 15 min  | A atribuição de uso de recursos do Azure para a oferta é verificada.             |
| Certificação                  | < 1 dia     | A oferta é analisada pela equipe de certificação do Azure. A oferta é exeminada em busca de vírus, malware, conformidade de segurança e problemas de segurança. A oferta é verificada para ver se atende aos critérios de qualificação. Para mais informações, consulte os [pré-requisitos](./cpp-prerequisites.md). Comentários serão fornecidos se um problema for encontrado. |
| Validação de test drive          | < 2 horas   | (Opcional) Se há um Test Drive, a Microsoft valida que ele pode ser implantado e replicado.  |
| Registro de embalagem e geração de leads | < 1 hora  | Os ativos técnicos da oferta são embalados para uso do cliente e os sistemas de chumbo são configurados e implantados. |
|  Aprovação do publicador             |  manual    | Revisão final do editor e confirmação antes que a oferta seja publicada. A oferta agora está disponível para versão prévia.  Você pode implantar sua oferta nas assinaturas selecionadas (nas etapas a informações de oferta) para verificar se ele atende a todos os seus requisitos.  Após você verificar a oferta, selecione **Go Live** para que sua oferta possa passar para a próxima etapa. |
| Revisão da Microsoft                | 7 a 14 dias | A Microsoft revisa holisticamente seu aplicativo do Azure e contata você via email se algum problema é descoberto.  O comprimento desta etapa depende da complexidade do aplicativo, dos problemas descobertos e de quão prontamente você responde a eles.  |
| Live                           | < 1 dia | A oferta é lançada, replicada para as regiões especificadas e disponibilizada ao público. |
|   |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|   |

Você pode monitorar o processo de publicação na guia **Status** para sua oferta no Portal do Cloud Partner.

![Guia Status para uma oferta de aplicativo do Azure](./media/offer-status-tab.png)

Depois de concluir o processo de publicação, a oferta será listada na [categoria de aplicativo do Microsoft Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/).

>[!Note]
>O opt-in do canal parceiro Cloud Solution Providers (CSP) já está disponível.  Consulte [os Provedores de Soluções em Nuvem](../../cloud-solution-providers.md) para obter mais informações sobre o marketing de sua oferta através dos canais parceiros microsoft CSP.

## <a name="errors-and-review-feedback"></a>Erros e comentários de revisão

Além de exibir o status da publicação da sua oferta, a guia **Status** também exibe mensagens de erro e comentários de quaisquer etapas de publicação em que um problema é encontrado.  Se o problema é crítico, a publicação é cancelada.  Em seguida, você precisa corrigir os problemas relatados e republicar a oferta.  Já que a etapa **revisão da Microsoft** representa uma revisão abrangente de sua oferta e seus ativos técnicos associados (especialmente o modelo do Azure Resource Manager), os problemas normalmente são apresentados como links de solicitações de pull.  Obter uma explicação de como exibir e responder a essas solicitações de pull, confira [Como lidar com comentários de revisão](./cpp-handling-review-feedback.md).


## <a name="next-steps"></a>Próximas etapas

Se encontrar erros em uma ou mais das etapas de publicação, você precisará corrigi-los e republicar sua oferta.  Se problemas críticos forem encontrados na etapa de **revisão da Microsoft**, você precisará [lidar com os comentários da revisão](./cpp-handling-review-feedback.md) acessando o repositório do Azure DevOps da equipe de revisão da Microsoft.

Depois que um aplicativo do Azure for publicado com êxito, você poderá [atualizar a oferta existente](./cpp-update-existing-offer.md) para refletir a mudança de requisitos empresariais ou técnicos. 
