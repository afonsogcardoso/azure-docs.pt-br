---
title: Categorização de imagens - Visão computacional
titleSuffix: Azure Cognitive Services
description: Aprenda mais sobre conceitos relacionados ao recurso de categorização de imagem da API da Pesquisa Visual Computacional.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 477349f1addf71a30e8ecb179266d8eac5510887
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80244743"
---
# <a name="categorize-images-by-subject-matter"></a>Categorizar imagens por assunto

Além das marcas e da descrição, a Pesquisa Visual Computacional retorna as categorias baseada em taxonomia detectadas em uma imagem. Ao contrário das marcas, as categorias são organizadas em uma hierarquia hereditária pai/filho e há menos delas (86, em vez de milhares de marcas). Todos os nomes de categoria são em inglês. A categorização pode ser feita por si só ou junto com o modelo mais recente de marcas.

## <a name="the-86-category-concept"></a>O conceito de 86 categorias

A visão computacional pode categorizar uma imagem de forma ampla ou específica, usando a lista de 86 categorias no diagrama a seguir. Para obter a taxonomia completa em formato de texto, confira [Taxonomia de categoria](category-taxonomy.md).

![Listas agrupadas de todas as categorias na categoria taxonomia](./Images/analyze_categories-v2.png)

## <a name="image-categorization-examples"></a>Exemplos de categorização de imagens

A resposta JSON a seguir ilustra o que a Computer Vision retorna ao categorizar a imagem de exemplo com base em seus recursos visuais.

![Uma mulher no telhado de um edifício residencial](./Images/woman_roof.png)

```json
{
    "categories": [
        {
            "name": "people_",
            "score": 0.81640625
        }
    ],
    "requestId": "bae7f76a-1cc7-4479-8d29-48a694974705",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

A tabela a seguir ilustra um conjunto de imagens típico e a categoria retornada pelo Computer Vision para cada imagem.

| Imagem | Categoria |
|-------|----------|
| ![Quatro pessoas posando juntas como uma família](./Images/family_photo.png) | people_group |
| ![Um filhote de cão sentado em um gramado](./Images/cute_dog.png) | animal_dog |
| ![Uma pessoa que está em uma montanha ao pôr do sol](./Images/mountain_vista.png) | outdoor_mountain |
| ![Uma montanha de pães em uma mesa](./Images/bread.png) | food_bread |

## <a name="use-the-api"></a>Usar a API

O recurso de categorização faz parte da API [Analisar imagem.](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa) Você pode chamar essa API por meio de um SDK nativo ou por meio de chamadas REST. Inclua `Categories` no parâmetro de consulta **visualFeatures.** Em seguida, quando você receber a resposta JSON completa, `"categories"` basta analisar a seqüência para o conteúdo da seção.

* [Partida rápida: Visão computacional .NET SDK](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
* [Quickstart: Analise uma imagem (Rest API)](./quickstarts/csharp-analyze.md)

## <a name="next-steps"></a>Próximas etapas

Aprenda os conceitos relacionados de [marcar imagens](concept-tagging-images.md) e [descrever imagens.](concept-describing-images.md)
