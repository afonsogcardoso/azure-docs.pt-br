---
title: Torne-se um parceiro de codificador no local - Azure Media Services
description: Este artigo discute como verificar seus codificadores de transmissão ao vivo no local.
services: media-services
author: johndeu
manager: johndeu
ms.author: johndeu
ms.date: 03/02/2020
ms.topic: article
ms.service: media-services
ms.openlocfilehash: f98d9942f8c30f06b0144503b056c1e8a393ae52
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79298624"
---
# <a name="how-to-verify-your-on-premises-live-streaming-encoder"></a>Como verificar seu codificador de transmissão ao vivo no local

Como um parceiro de codificador local dos Serviços de Mídia do Azure, os Serviços de Mídia promovem seu produto, recomendando seu codificador aos clientes corporativos. Para tornar-se um parceiro de codificador local, é necessário verificar a compatibilidade do seu codificador local com os Serviços de Mídia. Para fazer isso, conclua as seguintes verificações.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="pass-through-live-event-verification"></a>Verificação de evento ao vivo de passagem

1. Na conta dos Serviços de Mídia, verifique se o **Ponto de Extremidade de Streaming** está em execução. 
2. Crie e inicie um evento ao vivo de **passagem**. <br/> Para saber mais, confira [Estados e cobrança do Evento ao vivo](live-event-states-billing.md).
3. Obtenha as URLs de ingestão e configure seu codificador local para usar a URL para enviar o fluxo ao vivo com múltiplas taxas de bits para os Serviços de Mídia.
4. Obtenha a URL de visualização e use-a para verificar se a entrada do codificador está sendo realmente recebida.
5. Crie um novo objeto **Asset.**
6. Crie uma **saída ao vivo** e use o nome do ativo que você criou.
7. Crie um **localizador de streaming** com os tipos de diretiva de **streaming** incorporados.
8. Liste os caminhos no **Localizador de Streaming** para retornar as URLs a serem usadas.
9. Obtenha o nome do host para o **Ponto de Extremidade de Streaming** do qual você deseja transmitir.
10. Combine a URL da etapa 8 com o nome do host na etapa 9 para obter a URL completa.
11. Execute o codificador dinâmico por aproximadamente 10 minutos.
12. Interrompa o Evento ao Vivo. 
13. Use um player como o [Player de Mídia do Azure](https://aka.ms/azuremediaplayer) para assistir ao ativo arquivado para verificar se a reprodução não tem falhas visíveis em nenhum nível de qualidade. Ou assista e valide por meio da URL de visualização durante a sessão ao vivo.
14. Registre a ID do ativo, a URL de streaming publicada para os arquivos ao vivo e as configurações e a versão usadas no codificador dinâmico.
15. Reinicie o estado do evento ao vivo após criar cada exemplo.
16. Repita as etapas de 5 a 15 para todas as configurações compatíveis com o codificador (com e sem sinalização de anúncios, legendas ou diferentes velocidades de codificação).

## <a name="live-encoding-live-event-verification"></a>Verificação do evento ao vivo de codificação ativa

1. Na conta dos Serviços de Mídia, verifique se o **Ponto de Extremidade de Streaming** está em execução. 
2. Crie e inicie o evento ao vivo de **codificação ativa**. <br/> Para saber mais, confira [Estados e cobrança do Evento ao vivo](live-event-states-billing.md).
3. Obtenha as URLs de ingestão e configure seu codificador para enviar por push uma transmissão ao vivo de taxa de bits única para os Serviços de Mídia.
4. Obtenha a URL de visualização e use-a para verificar se a entrada do codificador está sendo realmente recebida.
5. Crie um novo objeto **Asset.**
6. Crie uma **saída ao vivo** e use o nome do ativo que você criou.
7. Crie um **localizador de streaming** com os tipos de diretiva de **streaming** incorporados.
8. Liste os caminhos no **Localizador de Streaming** para retornar as URLs a serem usadas.
9. Obtenha o nome do host para o **Ponto de Extremidade de Streaming** do qual você deseja transmitir.
10. Combine a URL da etapa 8 com o nome do host na etapa 9 para obter a URL completa.
11. Execute o codificador dinâmico por aproximadamente 10 minutos.
12. Interrompa o Evento ao Vivo.
13. Use um player como o [Player de Mídia do Azure](https://aka.ms/azuremediaplayer) para assistir ao ativo arquivado para verificar se a reprodução não tem galhas visíveis em nenhum nível. Ou assista e valide por meio da URL de visualização durante a sessão ao vivo.
14. Registre a ID do ativo, a URL de streaming publicada para os arquivos ao vivo e as configurações e a versão usadas no codificador dinâmico.
15. Reinicie o estado do evento ao vivo após criar cada exemplo.
16. Repita as etapas de 5 a 15 para todas as configurações compatíveis com o codificador (com e sem sinalização de anúncios, legendas ou diferentes velocidades de codificação).

## <a name="longevity-verification"></a>Verificação de longevidade

Siga as mesmas etapas que as da [verificação de evento ao vivo de passagem](#pass-through-live-event-verification), exceto pela etapa 11. <br/>Em vez de dez minutos, execute o codificador dinâmico por uma semana ou mais. Use um player como o [Player de Mídia do Azure](https://aka.ms/azuremediaplayer) para assistir à transmissão ao vivo periodicamente (ou um ativo arquivado) para verificar se a reprodução não tem falhas visíveis.

## <a name="email-your-recorded-settings"></a>Envie suas configurações registradas por email

Por fim, envie por email as configurações registradas e os parâmetros dos arquivos ao vivo aos Serviços de Mídia do Azure em amshelp@microsoft.com como uma notificação de que todas as verificações de autoverificação foram aprovadas. Além disso, inclua suas informações de contato para quaisquer acompanhamentos. Contate a equipe dos Serviços de Mídia do Azure para tratar quaisquer questões sobre esse processo.

## <a name="see-also"></a>Confira também

[Codificadores testados no local](recommended-on-premises-live-encoders.md)

## <a name="next-steps"></a>Próximas etapas

[Transmissão ao vivo com os Serviços de Mídia v3](live-streaming-overview.md)
