---
title: Considerações sobre o programa Provedor de Soluções na Nuvem
description: Para parceiros CSP, o gerenciamento de recursos delegados do Azure ajuda a melhorar a segurança e o controle oferecendo permissões granulares.
ms.date: 12/18/2019
ms.topic: conceptual
ms.openlocfilehash: 66ea74751f12a499a1e2d9e083497da31746e3c7
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75456920"
---
# <a name="azure-lighthouse-and-the-cloud-solution-provider-program"></a>Azure Lighthouse e o programa Provedor de Soluções na Nuvem

Se você é [parceiro CSP (Provedor de Soluções na Nuvem)](https://docs.microsoft.com/partner-center/csp-overview), já pode acessar as assinaturas do Azure criadas para seus clientes por meio do programa CSP usando a funcionalidade [AOBO (Administrar em nome de)](https://channel9.msdn.com/Series/cspdev/Module-11-Admin-On-Behalf-Of-AOBO). Esse acesso permite que você dê suporte, configure e gerencie diretamente as assinaturas dos seus clientes.

Com [o Azure Lighthouse,](../overview.md)você pode usar o gerenciamento de recursos delegado do Azure junto com o AOBO. Isso ajuda a melhorar a segurança e reduz o acesso desnecessário habilitando permissões mais granulares para seus usuários. Isso também possibilita maior eficiência e escalabilidade, pois os usuários podem trabalhar em várias assinaturas de clientes usando um único logon em seu locatário.

> [!TIP]
> Para ajudar a proteger os recursos do cliente, lembre-se de examinar e seguir nossas [práticas de segurança recomendadas](recommended-security-practices.md), juntamente com os [requisitos de segurança do parceiro](https://docs.microsoft.com/partner-center/partner-security-requirements).

## <a name="administer-on-behalf-of-aobo"></a>AOBO (Administrar em Nome de)

Com o AOBO, qualquer usuário com a função [Agente Administrador](https://docs.microsoft.com/partner-center/permissions-overview#manage-commercial-transactions-in-partner-center-azure-ad-and-csp-roles) em seu locatário terá acesso AOBO às assinaturas do Azure que você criar por meio do programa CSP. Os usuários que precisarem de acesso às assinaturas de clientes deverão ser um membro deste grupo. O AOBO não permite a flexibilidade de criar grupos distintos que funcionam com clientes diferentes ou de habilitar funções diferentes para grupos ou usuários.

![Gerenciamento de locatário usando AOBO](../media/csp-1.jpg)

## <a name="azure-delegated-resource-management"></a>Gerenciamento de recursos delegados do Azure

Usando o gerenciamento de recursos delegados do Azure, é possível atribuir diferentes grupos a diferentes clientes ou funções, conforme mostrado no diagrama a seguir. Como os usuários terão o nível apropriado de acesso por meio do gerenciamento de recursos delegados do Azure, é possível reduzir o número de usuários que têm a função Agente Administrador (e, portanto, ter acesso AOBO total). Isso ajuda a melhorar a segurança limitando o acesso desnecessário aos recursos dos seus clientes. Também oferece mais flexibilidade de gerenciar vários clientes em escala.

A integração de uma assinatura que você criou por meio do programa CSP segue as etapas descritas em [Integrar uma assinatura ao gerenciamento de recursos delegados do Azure](../how-to/onboard-customer.md). Qualquer usuário que tenha a função Agente Administrador em seu locatário poderá realizar essa integração.

![Gerenciamento de locatários usando AOBO e gerenciamento de recursos delegados do Azure](../media/csp-2.jpg)

> [!NOTE]
> A página [ **Meus clientes** no portal Azure](../how-to/view-manage-customers.md) agora inclui uma seção **Cloud Solution Provider (Preview),** que exibe informações e recursos de cobrança para clientes CSP que [assinaram o Acordo de Clientes microsoft (MCA)](https://docs.microsoft.com/partner-center/confirm-customer-agreement) e estão [sob o plano Azure.](https://docs.microsoft.com/partner-center/azure-plan-get-started) Para saber mais, confira [Introdução à conta de cobrança do Contrato de Parceiro da Microsoft](../../billing/mpa-overview.md).
>
> Os clientes csp podem aparecer nesta seção, quer eles também tenham sido ou não a bordo para o gerenciamento de recursos delegados do Azure. Se tiverem, também aparecerão na seção **Clientes,** conforme descrito em [Exibir e gerenciar clientes e recursos delegados](../how-to/view-manage-customers.md). Da mesma forma, um cliente CSP não precisa aparecer na seção **Cloud Solution Provider (Preview)** dos **Meus clientes** para que você os faça a bordo para o gerenciamento de recursos delegado do Azure.

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre as [experiências de gerenciamento entre locatários](cross-tenant-management-experience.md).
- Saiba como [integrar uma assinatura ao gerenciamento de recursos delegados do Azure](../how-to/onboard-customer.md).
- Saiba mais sobre o [programa Provedor de Soluções na Nuvem](https://docs.microsoft.com/partner-center/csp-overview).
