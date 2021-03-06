---
title: Conversão de dados - LUIS
titleSuffix: Azure Cognitive Services
description: Saiba como as expressões podem ser alteradas antes das previsões no Reconhecimento vocal (LUIS)
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 07/29/2019
ms.author: diberry
ms.openlocfilehash: aeb9094db83b14af988f70485788934a7854200c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "68619797"
---
# <a name="convert-data-format-of-utterances"></a>Converter o formato de dados de declarações
LUIS fornece as seguintes conversões de um enunciado do usuário antes da previsão"

* Fala a texto usando serviço [de fala de serviços cognitivos.](../Speech-Service/overview.md) 

## <a name="speech-to-text"></a>Conversão de fala em texto

A fala ao texto é fornecida como uma integração com o LUIS. 

### <a name="intent-conversion-concepts"></a>Conceitos de conversão de intenções
Conversão de fala em texto no LUIS permite que você envie expressões faladas a um ponto de extremidade e receba uma resposta de previsão LUIS. O processo é uma integração entre o serviço de [Fala](https://docs.microsoft.com/azure/cognitive-services/Speech) com LUIS. Saiba mais sobre a conversão de fala em intenção com um [tutorial](../speech-service/how-to-recognize-intents-from-speech-csharp.md).

### <a name="key-requirements"></a>Requisitos de chave
Você não precisa criar uma chave de **API de Fala do Bing** para esta integração. Uma chave de **Reconhecimento vocal** criada no portal do Azure funciona para este exercício. Não use a chave de inicialização do LUIS.

### <a name="pricing-tier"></a>Camada de preços
Essa integração usa um modelo de [preço](luis-boundaries.md#key-limits) diferente dos tipos de preço comuns do Reconhecimento vocal. 

### <a name="quota-usage"></a>Uso da cota
Consulte [limites de chave](luis-boundaries.md#key-limits) para obter informações. 

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Extraindo dados](luis-concept-data-extraction.md)

