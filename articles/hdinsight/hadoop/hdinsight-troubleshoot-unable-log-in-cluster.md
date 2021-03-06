---
title: Não é possível fazer login no cluster Azure HDInsight
description: Solução de problemas por que não é possível entrar no cluster Apache Hadoop no Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/31/2019
ms.openlocfilehash: 2099d1b7583017733498946a5866ab37de43ba9c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75894063"
---
# <a name="scenario-unable-to-log-into-azure-hdinsight-cluster"></a>Cenário: Não é possível fazer login no cluster Azure HDInsight

Este artigo descreve etapas de solução de problemas e possíveis resoluções para problemas ao interagir com clusters Azure HDInsight.

## <a name="issue"></a>Problema

Não é possível fazer login no cluster Azure HDInsight.

## <a name="cause"></a>Causa

As razões podem variar. Tenha em mente que ao fazer login no cluster ou nos painéis do aplicativo, use suas credenciais "cluster login" ou HTTP. Ao conectar-se remotamente, use suas credenciais do Secure Shell (SSH) ou a área de trabalho remota.

## <a name="resolution"></a>Resolução

Para resolver problemas comuns, tente uma ou mais das etapas a seguir.

* Tente abrir o painel do cluster em uma nova guia do navegador no modo de privacidade.

* Se você não conseguir lembrar suas credenciais SSH, você pode [redefinir as credenciais dentro da UI Ambari](../hdinsight-administer-use-portal-linux.md#change-passwords).

## <a name="next-steps"></a>Próximas etapas

Se você não encontrou seu problema ou não conseguiu resolver seu problema, visite um dos seguintes canais para obter mais suporte:

* Obtenha respostas de especialistas do Azure através [do Azure Community Support](https://azure.microsoft.com/support/community/).

* Conecte-se com [@AzureSupport](https://twitter.com/azuresupport) - a conta oficial do Microsoft Azure para melhorar a experiência do cliente conectando a comunidade Azure aos recursos certos: respostas, suporte e especialistas.

* Se você precisar de mais ajuda, você pode enviar uma solicitação de suporte do [portal Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Selecione **Suporte** na barra de menus ou abra o centro **de suporte Ajuda +.** Para obter informações mais detalhadas, consulte [Como criar uma solicitação de suporte ao Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request). O acesso ao gerenciamento de assinaturas e suporte ao faturamento está incluído na assinatura do Microsoft Azure, e o suporte técnico é fornecido através de um dos Planos de Suporte do [Azure](https://azure.microsoft.com/support/plans/).
