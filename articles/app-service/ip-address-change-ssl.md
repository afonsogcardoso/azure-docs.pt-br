---
title: Prepare-se para a alteração de endereço IP SSL
description: Se seu endereço IP SSL vai ser alterado, saiba o que fazer para que seu aplicativo continue a funcionar após a alteração.
ms.topic: article
ms.date: 06/28/2018
ms.custom: seodec18
ms.openlocfilehash: 34f35eb67cada6066e35227fcd6a0eaf425ac007
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74672398"
---
# <a name="how-to-prepare-for-an-ssl-ip-address-change"></a>Como se preparar para uma alteração de endereço IP SSL

Se você recebeu uma notificação de que o endereço IP SSL de seu aplicativo do Serviço de Aplicativo do Azure está mudando, siga as instruções neste artigo para liberar o endereço IP SSL existente e atribuir um novo.

## <a name="release-ssl-ip-addresses-and-assign-new-ones"></a>Liberar endereços IP SSL e atribuir novos

1.  Abra o [portal Azure.](https://portal.azure.com)

2.  No menu de navegação à esquerda, selecione **Serviços de Aplicativos**.

3.  Selecione o aplicativo do Serviço de Aplicativo na lista.

4.  No cabeçalho **Configurações**, clique em **Configurações de SSL** no painel de navegação à esquerda.

1. Na seção Associações SSL, selecione o registro de nome do host. No editor que é aberto, escolha **SSL SNI** no menu suspenso **Tipo de SSL** e clique em **Adicionar Associação**. Quando você vir a mensagem de êxito da operação, o endereço IP existente foi liberado.

6.  Na seção **Associações SSL**, selecione novamente o mesmo registro de nome do host com o certificado. No editor que é aberto, escolha dessa vez **SSL baseado em IP** no menu suspenso **Tipo de SSL** e clique em **Adicionar Associação**. Quando vir a mensagem de êxito da operação, você já adquiriu um novo endereço IP.

7.  Se um registro (registro DNS que aponta diretamente para o endereço IP) for configurado no Portal de Registro de Domínio (Provedor DNS de terceiros ou DNS do Azure), substitua o endereço IP existente pelo recém-gerado. Você pode encontrar o novo endereço IP seguindo as instruções na próxima seção.

## <a name="find-the-new-ssl-ip-address-in-the-azure-portal"></a>Você pode encontrar o novo endereço SSL no Portal do Azure.

1.  Aguarde alguns minutos e, em seguida, abra o [Portal do Azure](https://portal.azure.com).

2.  No menu de navegação à esquerda, selecione **Serviços de Aplicativos**.

3.  Selecione o aplicativo do Serviço de Aplicativo na lista.

4.  No cabeçalho **Configurações**, clique em **Propriedades** no painel de navegação esquerdo e localize a seção rotulada como **Endereço IP Virtual**.

5. Copie o endereço IP e reconfigure o registro de domínio ou o mecanismo de IP.

## <a name="next-steps"></a>Próximas etapas

Este artigo explicou como se preparar para uma alteração de endereço IP que foi iniciada pelo Azure. Para obter mais informações sobre endereços IP no Serviço de Aplicativo do Azure, consulte [SSL e endereços IP SSL no Serviço de Aplicativo do Azure](overview-inbound-outbound-ips.md).
