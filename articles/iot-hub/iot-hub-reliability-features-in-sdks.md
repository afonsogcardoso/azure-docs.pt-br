---
title: Gerenciar conectividade IoT Hub & SDKs de mensagens confiáveis
description: Saiba como melhorar a conectividade do dispositivo e as mensagens ao usar SDKs do dispositivo do Hub IoT
services: iot-hub
author: robinsh
ms.author: robinsh
ms.date: 07/07/2018
ms.topic: article
ms.service: iot-hub
ms.openlocfilehash: 1ca7219824a00a5af0bed7d42da75fc06ce2010d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79284441"
---
# <a name="manage-connectivity-and-reliable-messaging-by-using-azure-iot-hub-device-sdks"></a>Gerenciar a conectividade e mensagens confiáveis usando SDKs do dispositivo do Hub IoT do Azure

Este artigo fornece diretrizes de alto nível para ajudar a projetar aplicativos de dispositivo que são mais resilientes. Mostra como tirar proveito dos recursos de sistema de mensagens confiáveis e conectividade em SDKs do dispositivo IoT do Azure. O objetivo deste guia é ajudar você a gerenciar os seguintes cenários:

* Correção de uma conexão de rede removida

* Alternar entre as conexões de rede diferentes

* Reconexão devido a erros de conexão transitórios de serviço

Detalhes de implementação podem variar por idioma. Para obter mais informações, consulte a documentação API ou SDK específico:

* [C/iOS SDK](https://github.com/azure/azure-iot-sdk-c)

* [.NET SDK](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [Java SDK](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [SDK do Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [Python SDK](https://github.com/Azure/azure-iot-sdk-python) (Confiabilidade ainda não implementada)

## <a name="designing-for-resiliency"></a>Design para resiliência

Os dispositivos IoT geralmente dependem de conexões de rede não contínuas instáveis (por exemplo, GSM ou satélite). Erros podem ocorrer quando os dispositivos interagem com serviços baseados em nuvem devido por causa da disponibilidade do serviço intermitente e nível de infraestrutura ou falhas transitórias. Um aplicativo que é executado em um dispositivo tem que gerenciar os mecanismos de conexão, reconexão e a lógica de repetição para envio e recebimento de mensagens. Além disso, os requisitos da estratégia de repetição dependem consideravelmente o do cenário de IoT do dispositivo, contexto, recursos.

Os SDKs do dispositivo Hub IoT do Azure têm como objetivo simplificar a conexão e comunicação da nuvem para o dispositivo e o dispositivo para a nuvem. Esses SDKs fornecem uma maneira robusta para se conectar ao Hub IoT do Azure e um conjunto abrangente de opções para enviar e receber mensagens. Os desenvolvedores também podem modificar a implementação existente para personalizar uma estratégia de tentativa melhor para um determinado cenário.

Os recursos relevantes do SDK que dão suporte para conectividade e mensagens confiáveis são abordados nas seções a seguir.

## <a name="connection-and-retry"></a>Conexão e repetição

Esta seção fornece uma visão geral dos padrões de reconexão e repetição disponíveis ao gerenciar conexões. Ela fornece detalhes sobre diretrizes de implementação para usar uma política de repetição diferente em seu aplicativo de dispositivo e lista as APIs relevantes de SDKs do dispositivo.

### <a name="error-patterns"></a>Padrões de erros

Falhas de conexão podem acontecer em vários níveis:

* Erros de rede: soquete desconectado e erros de resolução de nome

* Erros no nível de protocolo para HTTP, AMQP e MQTT transport: links desanexados ou sessões expiradas

* Erros no nível de aplicativo que resultam de erros locais, como credenciais inválidas ou comportamento de serviço (por exemplo, excedendo a cota ou limitação)

Os SDKs do dispositivo detectam erros nos três níveis. Erros relacionados a SO e erros de hardware não são detectados e manipulados pelos SDKs do dispositivo. O design SDK é baseado nas [Diretrizes de Tratamento de Falhas Transitórias](/azure/architecture/best-practices/transient-faults#general-guidelines) do Centro de Arquitetura do Azure.

### <a name="retry-patterns"></a>Padrões de repetição

As etapas a seguir descrevem o processo de repetição quando forem detectados erros de conexão:

1. O SDK detecta o erro e o erro associado na rede, protocolo ou aplicativo.

2. O SDK usa o filtro de erro para determinar o tipo de erro e decidir se uma nova tentativa é necessária.

3. Se o SDK identifica um **erro irrecuperável**, operações, como a conexão, enviar e receber são interrompidos. O SDK notificará o usuário. Exemplos de erros irrecuperáveis incluem um erro de autenticação e um erro de ponto de extremidade ruim.

4. Se o SDK identifica um **erro recuperável**, ele tentará novamente de acordo com a política de repetição especificada até que tenha decorrido o tempo limite definido.  Observe que o SDK usa a política de repetição por padrão **de retorno exponencial com tremulação**.
5. Quando o tempo limite definido expira, o SDK para de tentar conectar ou enviar. Notifica o usuário.

6. O SDK permite ao usuário anexar um retorno de chamada para receber alterações no status da conexão.

Os SDKs fornecem três políticas de repetição:

* **Retirada exponencial com tremulação**: essa política de repetição padrão tende a ser agressiva no início e lenta para baixo ao longo do tempo até atingir um atraso máximo. O design é baseado nas [Diretrizes de repetição do Azure Architecture Center](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific). 

* **Repetição personalizada**: para alguns idiomas do SDK, você pode criar uma política de repetição personalizada que é mais adequada para seu cenário e, em seguida, injetar RetryPolicy. A repetição personalizada não está disponível no C SDK, e atualmente não é suportada no Python SDK. O Python SDK se reconecta conforme necessário.

* **Sem repetição**: você pode definir a política de repetição para "Nenhuma repetição," que desabilita a lógica de repetição. O SDK tenta conectar uma vez e enviar uma mensagem uma vez, assumindo que a conexão foi estabelecida. Essa política é normalmente usada em cenários com questões de largura de banda ou custo. Se você escolher essa opção, as mensagens que não forem enviadas serão perdidas e não poderão ser recuperadas.

### <a name="retry-policy-apis"></a>APIs da política de repetição

   | . | Método SetRetryPolicy | Implementações de política | Diretrizes de implementação |
   |-----|----------------------|--|--|
   |  C/iOS  | [IOTHUB_CLIENT_RESULT IoTHubClient_SetRetryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/2018-05-04/iothub_client/inc/iothub_client.h#L188)        | **Padrão**: [IOTHUB_CLIENT_RETRY_EXPONENTIAL_BACKOFF](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)<BR>**Personalizado:** usa [retryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies) disponível<BR>**Sem repetição:** [IOTHUB_CLIENT_RETRY_NONE](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)  | [Implementação do C/iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#)  |
   | Java| [Política de setRetry](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.deviceclientconfig.setretrypolicy?view=azure-java-stable)        | **Padrão**: [classe ExponentialBackoffWithJitter](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)<BR>**Personalizado:** implementa [interface RetryPolicy](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/RetryPolicy.java)<BR>**Sem repetição:** [classe NoRetry](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)  | [Implementação Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md) |
   | .NET| [DeviceClient.SetRetryPolicy](/dotnet/api/microsoft.azure.devices.client.deviceclient.setretrypolicy?view=azure-dotnet) | **Padrão**: [classe ExponentialBackoff](/dotnet/api/microsoft.azure.devices.client.exponentialbackoff?view=azure-dotnet)<BR>**Personalizado:** implementa [interface IRetryPolicy](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.iretrypolicy?view=azure-dotnet)<BR>**Sem repetição:** [classe NoRetry](/dotnet/api/microsoft.azure.devices.client.noretry?view=azure-dotnet) | [Implementação C#](https://github.com/Azure/azure-iot-sdk-csharp) | |
   | Nó| [setRetryPolicy](/javascript/api/azure-iot-device/client?view=azure-iot-typescript-latest) | **Padrão**: [classe ExponentialBackoffWithJitter](/javascript/api/azure-iot-common/exponentialbackoffwithjitter?view=azure-iot-typescript-latest)<BR>**Personalizado:** implementa [interface RetryPolicy](/javascript/api/azure-iot-common/retrypolicy?view=azure-iot-typescript-latest)<BR>**Sem repetição:** [classe NoRetry](/javascript/api/azure-iot-common/noretry?view=azure-iot-typescript-latest) | [Implementação Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them) |
   | Python| Sem suporte no momento | Sem suporte no momento | Sem suporte no momento |

Os exemplos de código a seguir ilustram esse fluxo:

#### <a name="net-implementation-guidance"></a>Diretrizes de implementação .NET

O exemplo de código abaixo mostra como definir e definir a política de repetição padrão:

   ```csharp
   // define/set default retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(int.MaxValue, TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(10), TimeSpan.FromMilliseconds(100));
   SetRetryPolicy(retryPolicy);
   ```

Para evitar o alto uso da CPU, as novas tentativas são limitadas se o código falhar imediatamente. Por exemplo, quando não há nenhuma rede ou rota para o destino. O tempo mínimo para executar a próxima repetição é de 1 segundo.

Se o serviço estiver respondendo com um erro de limitação, a política de repetição será diferente e não poderá ser alterada por meio da API pública:

   ```csharp
   // throttled retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(RetryCount, TimeSpan.FromSeconds(10), 
     TimeSpan.FromSeconds(60), TimeSpan.FromSeconds(5)); SetRetryPolicy(retryPolicy);
   ```

O mecanismo de repetição será interrompido após `DefaultOperationTimeoutInMilliseconds`, que atualmente é definido em 4 minutos.

#### <a name="other-languages-implementation-guidance"></a>Outras diretrizes de implementação de linguagens

Para obter exemplos de código em outros idiomas, revise os seguintes documentos de implementação. O repositório contém exemplos que demonstram o uso de APIs de política de repetição.

* [C/iOS SDK](https://github.com/azure/azure-iot-sdk-c)

* [.NET SDK](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [Java SDK](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [SDK do Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [SDK do Python](https://github.com/Azure/azure-iot-sdk-python)

## <a name="next-steps"></a>Próximas etapas

* [Usar SDKs de serviço e dispositivo](./iot-hub-devguide-sdks.md)

* [Usar o SDK do dispositivo IoT para C](./iot-hub-device-sdk-c-intro.md)

* [Desenvolver para dispositivos restritos](./iot-hub-devguide-develop-for-constrained-devices.md)

* [Desenvolver para dispositivos móveis](./iot-hub-how-to-develop-for-mobile-devices.md)

* [Solucionar problemas de desconexões de dispositivos](iot-hub-troubleshoot-connectivity.md)
