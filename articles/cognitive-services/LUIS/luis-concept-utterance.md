---
title: Boas declarações de exemplo - LUIS
titleSuffix: Azure Cognitive Services
description: Enunciados são entradas do usuário que seu aplicativo precisa interpretar. Colete frases que você acredita que os usuários vão inserir. Inclua enunciados que tenham o mesmo significado, mas sejam construídos de modo diferente em termos de comprimento e posicionamento de palavras.
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/15/2019
ms.author: diberry
ms.openlocfilehash: 7412677773b60a1894a6ece7251e797bfddee091
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79219913"
---
# <a name="understand-what-good-utterances-are-for-your-luis-app"></a>Reconheça quais são os enunciados ideais para o aplicativo LUIS

**Enunciados** são entrada do usuário que seu aplicativo precisa interpretar. Para treinar o LUIS a extrair intenções e entidades dos enunciados, é importante capturar uma variedade de diferentes enunciados para cada intenção. Aprendizado ativo, ou o processo de continuar a treinar em novos enunciados, é essencial para a inteligência de aprendizado de máquina que o LUIS fornece.

Colete enunciados que você acredita que os usuários irão inserir. Inclua enunciados que tenham o mesmo significado mas são construídos de várias maneiras diferentes:

* Duração do enunciado - curta, média e longa para o aplicativo cliente
* Palavra e comprimento de frase 
* Posicionamento de palavras - entidade no início, meio e fim do enunciado
* Gramática 
* Pluralização
* Lematização
* Escolher substantivo e verbo
* Pontuação - uma boa variedade usando pontuação correta, inadequação e agramatical

## <a name="how-to-choose-varied-utterances"></a>Como escolher enunciados variados

Quando você começar a [adicionar adicionando enunciados de exemplo](luis-how-to-add-example-utterances.md) ao seu modelo LUIS, aqui estão alguns princípios a ter em mente.

### <a name="utterances-arent-always-well-formed"></a>Os enunciados nem sempre são bem formados

Pode ser uma sentença como "Reserve uma passagem para Paris para mim", ou um fragmento de uma sentença, como "Reserva" ou "Voo Paris".  Os usuários costumam cometer erros de ortografia. Ao planejar o aplicativo, considere se você usará ou não a [Verificação Ortográfica do Bing](luis-tutorial-bing-spellcheck.md) para corrigir a entrada do usuário, antes de passar para o LUIS. 

Se você não realizar a verificação ortográfica de enunciados do usuário, deverá treinar o LUIS em enunciados que incluam erros de digitação e erros de ortografia.

### <a name="use-the-representative-language-of-the-user"></a>Usar o idioma representante do usuário

Ao escolher enunciados, esteja ciente de que o que você acredita ser uma frase ou termo comum pode não ser o adequado para o usuário típico do aplicativo cliente. Eles não podem ter a experiência de domínio. Tenha cuidado ao usar termos ou frases que um usuário só diria se fosse um especialista.

### <a name="choose-varied-terminology-as-well-as-phrasing"></a>Escolha a terminologia e frases variadas

Você descobrirá que, mesmo que você se esforce para criar padrões de frases variados, ainda repetirá algum vocabulário.

Pegue estes enunciados de exemplo:

|Exemplo de enunciados|
|--|
|Como obtenho um computador?|
|Onde obtenho um computador?|
|Quero obter um computador, como faço isso?|
|Quando posso ter um computador?| 

O termo principal aqui, "computador", não é variado. Use alternativas como computadores, laptops, estações de trabalho ou até mesmo máquinas. LUIS pode inferir inteligentemente sinônimos do contexto, mas quando você cria expressões para treinamento, é sempre melhor variá-los.

## <a name="example-utterances-in-each-intent"></a>Enunciados de exemplo em cada intenção

Cada intenção precisa ter exemplos de enunciados, pelo menos 15. Se você tiver uma intenção que não tenha nenhum enunciado de exemplo, não poderá treinar o LUIS. Se você tem uma intenção com um ou muito poucos exemplos de declarações, LUIS pode não prever com precisão a intenção. 

## <a name="add-small-groups-of-15-utterances-for-each-authoring-iteration"></a>Adicione pequenos grupos de 15 enunciados para cada iteração de criação

Em cada iteração do modelo, não adicione uma grande quantidade de enunciados. Adicione enunciados em quantidades de 15. [Treine](luis-how-to-train.md), [publique](luis-how-to-publish-app.md) e [teste](luis-interactive-test.md) novamente.  

O LUIS constrói modelos eficazes com enunciados que são cuidadosamente selecionados pelo criador do modelo LUIS. Adicionar muitos enunciados não é positivo porque gera confusão.

É melhor começar com alguns enunciados e, em seguida, [examinar enunciados de ponto de extremidade](luis-how-to-review-endpoint-utterances.md) para a extração de entidade e previsão de intenção corretas.

## <a name="utterance-normalization"></a>Normalização da emissão

A normalização da enunciação é o processo de ignorar os efeitos da pontuação e diacríticos durante o treinamento e a previsão.

## <a name="utterance-normalization-for-diacritics-and-punctuation"></a>Normalização de enunciados para diacríticos e pontuação

A normalização do enunciado é definida quando você cria ou importa o aplicativo porque é uma configuração no arquivo JSON do aplicativo. As configurações de normalização do enunciado são desatadas por padrão. 

Diacríticos são marcas ou sinais dentro do texto, tais como: 

```
İ ı Ş Ğ ş ğ ö ü
```

Se o aplicativo ligar a normalização, as pontuações no painel **de teste,** testes em lote e consultas de ponto final serão alteradas para todas as declarações usando diacritics ou pontuação.

Ative a normalização do enunciado para diacríticos ou pontuação `settings` para o seu arquivo do aplicativo LUIS JSON no parâmetro.

```JSON
"settings": [
    {"name": "NormalizePunctuation", "value": "true"},
    {"name": "NormalizeDiacritics", "value": "true"}
] 
```

A **pontuação normalizadora** significa que antes de seus modelos serem treinados e antes que suas consultas de ponto final sejam previstas, a pontuação será removida das declarações. 

Normalizar **diacritics** substitui os personagens por diacritics em expressões por caracteres regulares. Por `Je parle français` exemplo: `Je parle francais`torna-se . 

A normalização não significa que você não verá pontuação e diacríticos em suas declarações de exemplo ou respostas de previsão, apenas que elas serão ignoradas durante o treinamento e a previsão.


### <a name="punctuation-marks"></a>Marcas de pontuação

Pontuação é um token separado no LUIS. Uma expressão que contém um período no final versus uma expressão que não contém um período no final são duas declarações separadas e podem obter duas previsões diferentes. 

Se a pontuação não for normalizada, o LUIS não ignora as marcas de pontuação, por padrão, porque alguns aplicativos clientes podem colocar significado nessas marcas. Verifique se suas declarações de exemplo usam pontuação e nenhuma pontuação para que os dois estilos retornem as mesmas pontuações relativas. 

Certifique-se de que o modelo manipule pontuação nos exemplos de expressões (com e sem pontuação) ou nos [padrões](luis-concept-patterns.md), onde é mais fácil ignorar a pontuação com a sintaxe especial: `I am applying for the {Job} position[.]`

Se a pontuação não tiver um significado específico no aplicativo do cliente, considere [ignorar a pontuação](#utterance-normalization) normalizando a pontuação. 

### <a name="ignoring-words-and-punctuation"></a>Ignorando palavras e pontuação

Se você quiser ignorar palavras específicas ou pontuação em padrões, use um [padrão](luis-concept-patterns.md#pattern-syntax) com `[]`a sintaxe _ignorar_ de colchetes quadrados, . 

## <a name="training-utterances"></a>Treinando enunciados

O treinamento geralmente não é determinístico: a previsão de enunciado pode variar ligeiramente entre versões ou aplicativos. É possível remover o treinamento não determinístico, atualizando a API de [configurações de versão](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) com o par de nome/valor `UseAllTrainingData` para usar todos os dados de treinamento.

## <a name="testing-utterances"></a>Testando enunciados 

Os desenvolvedores devem começar a testar o aplicativo LUIS com tráfego real, enviando enunciados para a URL do [ponto de extremidade de previsão](luis-how-to-azure-subscription.md). Esses enunciados são usados para melhorar o desempenho de intenções e entidades com [Examinar enunciados](luis-how-to-review-endpoint-utterances.md). Testes enviadas com o painel de teste de site do LUIS não são enviados por meio do ponto de extremidade e, portanto, não contribuem para aprendizado ativo. 

## <a name="review-utterances"></a>Examinar enunciados

Depois que seu modelo estiver treinado, publicado e recebendo consultas de [ponto de extremidade](luis-glossary.md#endpoint), [examine os enunciados](luis-how-to-review-endpoint-utterances.md) sugeridos pelo LUIS. O LUIS seleciona enunciados de ponto de extremidade que têm pontuações baixas para intenção ou entidade. 

## <a name="best-practices"></a>Práticas recomendadas

Revise as [melhores práticas](luis-concept-best-practices.md) e aplique-as como parte do ciclo de criação regular.

## <a name="label-for-word-meaning"></a>Rótulo para o significado da palavra

Se a escolha ou a disposição das palavras for a mesma, mas não significar a mesma coisa, não a rotule com a entidade. 

Nas declarações a seguir, a palavra `fair` é um homógrafo. Sua ortografia é a mesma, mas tem um significado diferente:

|Enunciado|
|--|
|Que tipos de festivais regionais estão acontecendo na área de Seattle neste verão?|
|A classificação atual para a análise de Seattle é justa?|

Se você quiser que uma entidade de evento localize todos os dados do evento, rotule a palavra `fair` na primeira declaração, mas não na segunda.


## <a name="next-steps"></a>Próximas etapas
Consulte [Adicionar enunciados de exemplo](luis-how-to-add-example-utterances.md) para obter informações sobre o treinamento de um aplicativo de LUIS para entender enunciados do usuário.

