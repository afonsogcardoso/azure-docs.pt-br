---
title: Sincronizar conteúdo de uma pasta na nuvem
description: Saiba como implantar seu aplicativo no Azure App Service via sincronização de conteúdo a partir de uma pasta na nuvem, incluindo OneDrive ou Dropbox.
ms.assetid: 88d3a670-303a-4fa2-9de9-715cc904acec
ms.topic: article
ms.date: 12/03/2018
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: 9f18eef607a5f655aecc0b6ebe1c85a239b34192
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77482965"
---
# <a name="sync-content-from-a-cloud-folder-to-azure-app-service"></a>Sincronizar o conteúdo de uma pasta de nuvem para o Serviço de Aplicativo do Azure
Este artigo mostra como sincronizar seu conteúdo do Dropbox e OneDrive com o [Serviço de Aplicativo do Azure](https://go.microsoft.com/fwlink/?LinkId=529714). 

A implantação de sincronização de conteúdo sob demanda é ativada pelo [mecanismo de implantação do Kudu](https://github.com/projectkudu/kudu/wiki) do Serviço de Aplicativo. Você pode trabalhar com o código e conteúdo do aplicativo em uma pasta de nuvem designada e, em seguida, sincronizar com o App Service com o clique de um botão. Sincronização de conteúdo usa o servidor de build do Kudu. 

## <a name="enable-content-sync-deployment"></a>Habilitar a implantação da sincronização de conteúdo

Para habilitar a sincronização de conteúdo, navegue até a página do aplicativo do Serviço de Aplicativo no [portal do Azure](https://portal.azure.com).

No menu à esquerda, clique em **Deployment Center** > **OneDrive** ou **Dropbox** > **Authorize**. Siga as solicitações de autorização. 

![](media/app-service-deploy-content-sync/choose-source.png)

Você só precisa autorizar uma vez com o OneDrive ou Dropbox. Se você já tiver autorização, basta clicar em **Continuar**. Você pode alterar a conta autorizada do OneDrive ou Dropbox clicando em **Alterar conta**.

![](media/app-service-deploy-content-sync/continue.png)

Na página **Configurar**, selecione a pasta que deseja sincronizar. Esta pasta é criada no seguinte caminho de conteúdo designado no OneDrive ou Dropbox. 
   
* **OneDrive**:`Apps\Azure Web Apps`
* **Dropbox**:`Apps\Azure`

Ao terminar, clique em **Continuar**.

Na página **Resumo**, verifique as opções e clique em **Concluir**.

## <a name="synchronize-content"></a>Sincronizar o conteúdo

Se você quiser sincronizar o conteúdo em sua pasta de nuvem com o Serviço de Aplicativo, volte para a página **Centro de Implantação** e clique em **Sincronizar**.

![](media/app-service-deploy-content-sync/synchronize.png)
   
   > [!NOTE]
   > Devido a diferenças subjacentes nas APIs, o **OneDrive for Business** não tem suporte no momento. 
   > 
   > 

## <a name="disable-content-sync-deployment"></a>Desabilitar a implantação da sincronização de conteúdo

Para desabilitar a sincronização de conteúdo, navegue até sua página do aplicativo de Serviço de Aplicativo no [portal do Azure](https://portal.azure.com).

No menu à esquerda, clique em **'Desconectar centro de implantação '''Desconexão'.** **Deployment Center** > 

![](media/app-service-deploy-content-sync/disable.png)

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Implantação por meio do repositório Git local](deploy-local-git.md)
