---
title: Serialização personalizada de cache de token (MSAL Python) | Azure
titleSuffix: Microsoft identity platform
description: Saiba como serializar o cache de token satisde para Python
services: active-directory
author: rayluo
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/13/2019
ms.author: rayluo
ms.reviewer: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 2593cc856afb98cf5186c4e33032c5e9151614f0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76704384"
---
# <a name="custom-token-cache-serialization-in-msal-for-python"></a>Serialização personalizada de cache de token em MSAL para Python

No MSAL Python, um cache de token na memória que persiste durante a sessão do aplicativo, é fornecido por padrão quando você cria uma instância de [ClientApplication](https://msal-python.readthedocs.io/en/latest/#confidentialclientapplication).

A serialização do cache de token, para que diferentes sessões do seu aplicativo possam acessá-lo, não é fornecida "fora da caixa". Isso porque o MSAL Python pode ser usado em tipos de aplicativos que não têm acesso ao sistema de arquivos - como aplicativos da Web. Para ter um cache de token persistente em um aplicativo MSAL Python, você deve fornecer serialização personalizada de cache de token.

As estratégias para serializar o cache de token diferem dependendo se você está escrevendo um aplicativo cliente público (Desktop) ou um aplicativo cliente confidencial (Web App, Web API ou aplicativo Daemon).

## <a name="token-cache-for-a-public-client-application"></a>Cache de token para um aplicativo cliente público

Aplicativos de clientes públicos são executados no dispositivo de um usuário e gerenciam tokens para um único usuário. Neste caso, você pode serializar todo o cache em um arquivo. Lembre-se de fornecer bloqueio de arquivos se seu aplicativo, ou outro aplicativo, pode acessar o cache simultaneamente. Para obter um exemplo simples de como serializar um cache de token em um arquivo sem bloqueio, consulte o exemplo na documentação de referência da classe [SerializableTokenCache.](https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache)

## <a name="token-cache-for-a-web-app-confidential-client-application"></a>Cache de token para um aplicativo web (aplicativo cliente confidencial)

Para aplicativos da Web ou APIs da Web, você pode usar a sessão, ou um cache Redis ou um banco de dados para armazenar o cache do token. Deve haver um cache de token por usuário (por conta) para garantir que você serialize o cache do token por conta.

## <a name="next-steps"></a>Próximas etapas

Consulte [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/master/app.py#L64-L72) para um exemplo de como usar o cache de token para um aplicativo Windows ou Linux Web ou API da Web. O exemplo é para um aplicativo web que chama a API do Microsoft Graph.
