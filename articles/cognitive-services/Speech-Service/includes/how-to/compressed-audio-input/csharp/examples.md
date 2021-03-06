---
author: IEvangelist
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: dapine
ms.openlocfilehash: 0e8a6a0ffa451337b9e7ea7590cfe614fc1129a1
ms.sourcegitcommit: 27bbda320225c2c2a43ac370b604432679a6a7c0
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/31/2020
ms.locfileid: "80409695"
---
Para transmitir em um formato de áudio `PullAudioInputStream` compactado para o serviço Speech, crie ou `PushAudioInputStream`. Em seguida, `AudioConfig` crie uma instância de uma classe de fluxo, especificando o formato de compactação do fluxo.

Vamos supor que você tem `pushStream` uma classe de fluxo de entrada chamada e está usando OPUS/OGG. Seu código pode parecer assim:

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

// ... omitted for brevity

var speechConfig =
    SpeechConfig.FromSubscription(
        "YourSubscriptionKey",
        "YourServiceRegion");

// Create an audio config specifying the compressed
// audio format and the instance of your input stream class.
var audioFormat =
    AudioStreamFormat.GetCompressedFormat(
        AudioStreamContainerFormat.OGG_OPUS);
var audioConfig =
    AudioConfig.FromStreamInput(
        pushStream,
        audioFormat);

using var recognizer = new SpeechRecognizer(speechConfig, audioConfig);
var result = await recognizer.RecognizeOnceAsync();

var text = result.Text;
```