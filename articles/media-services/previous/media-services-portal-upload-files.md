---
title: Carregar arquivos para uma conta dos Serviços de Mídia no portal do Azure | Microsoft Docs
description: Este tutorial orienta você pelas etapas de carregamento de arquivos para uma conta do Serviços de Mídia do Azure no portal do Azure
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 3ad3dcea-95be-4711-9aae-a455a32434f6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 3ebeb3c601dd3f734265d49d60728056561928be
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "61127797"
---
# <a name="upload-files-to-a-media-services-account-in-the-azure-portal"></a>Carregar arquivos para uma conta dos Serviços de Mídia no portal do Azure 

> [!div class="op_single_selector"]
> * [Portal](media-services-portal-upload-files.md)
> * [.NET](media-services-dotnet-upload-files.md)
> * [Resto](media-services-rest-upload-files.md)
> 

> [!NOTE]
> Não estão sendo adicionados novos recursos ou funcionalidades aos Serviços de Mídia v2. <br/>Confira a versão mais recente, [Media Services v3](https://docs.microsoft.com/azure/media-services/latest/). Além disso, veja [as orientações de migração de v2 para v3](../latest/migrate-from-v2-to-v3.md)

Nos Serviços de Mídia do Azure, você pode carregar seus arquivos digitais para um ativo. O ativo pode conter vídeo, áudio, imagens, coleções de miniaturas, sequências de texto e arquivos de legendas (e os metadados desses arquivos). Depois que os arquivos são carregados, o conteúdo é armazenado com segurança na nuvem para processamento adicional e transmissão.

Os Serviços de Mídia têm um tamanho máximo de arquivo para processamento. Para obter detalhes sobre os limites de tamanho de arquivo, confira [Cotas e limitações dos Serviços de Mídia](media-services-quotas-and-limitations.md).

Para concluir este tutorial, você precisa de uma conta do Azure. Para obter detalhes, consulte [a avaliação gratuita do Azure](https://azure.microsoft.com/pricing/free-trial/). 

## <a name="upload-files"></a>Carregar arquivos
1. No [Portal do Azure](https://portal.azure.com/), selecione sua conta dos Serviços de Mídia do Azure.
2. Selecione**Ativos** **de configurações** > . Em seguida, selecione o botão **Carregar**.
   
    ![Carregar arquivos](./media/media-services-portal-vod-get-started/media-services-upload.png)
   
    A janela **Carregar um ativo de vídeo** é exibida.
   
   > [!NOTE]
   > Os Serviços de Mídia não limitam o tamanho do arquivo no carregamento de vídeos.
 
3. Em seu computador, vá para o vídeo que você deseja carregar. Selecione o vídeo e selecione **OK**.  
   
    O carregamento é iniciado. Você pode ver o progresso abaixo do nome do arquivo.  

Quando o carregamento é concluído, o novo ativo fica listado no painel **Ativos**. 

## <a name="media-services-learning-paths"></a>Roteiros de aprendizagem dos Serviços de Mídia
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fornecer comentários
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Próximas etapas
* Saiba como [codificar seus ativos carregados](media-services-portal-encode.md).

* Você também pode usar as Azure Functions para acionar um trabalho de codificação quando arquivo chega no contêiner configurado. Para saber mais, confira o exemplo em [Serviços de Mídia: integração dos Serviços de Mídia do Azure com o Azure Functions e os Aplicativos Lógicos](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/).


