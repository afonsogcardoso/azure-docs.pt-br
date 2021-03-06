---
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 03/31/2020
ms.author: juliako
ms.openlocfilehash: 99aeb5384b317d1b4d291c769b5402e829247b30
ms.sourcegitcommit: 62c5557ff3b2247dafc8bb482256fef58ab41c17
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2020
ms.locfileid: "80656190"
---
> [!NOTE]
> Para recursos que não são fixos, abra um boleto de apoio para pedir um aumento nas cotas. Não crie contas adicionais do Azure Media Services na tentativa de obter limites mais altos.

### <a name="account-limits"></a>Limites da conta

| Recurso | Limite padrão | 
| --- | --- | 
| Contas de Serviços de Mídia em uma única assinatura | 25 (fixo) |

### <a name="asset-limits"></a>Limites de ativos

| Recurso | Limite padrão | 
| --- | --- | 
| Ativos por conta de Serviços de Mídia | 1.000.000|

### <a name="storage-media-limits"></a>Limites de armazenamento (mídia)

| Recurso | Limite padrão | 
| --- | --- | 
| Tamanho de arquivo| Em alguns cenários, há um limite no tamanho máximo de arquivo com suporte para o processamento nos Serviços de Mídia. <sup>(1)</sup> |
| Contas de armazenamento | 100<sup>(2)</sup> (fixo) |

<sup>1</sup> O tamanho máximo com suporte para um único blob atualmente é de até 5 TB no Azure Blob Storage. Limites adicionais se aplicam nos Serviços de Mídia com base nos tamanhos de VM que são usados pelo serviço. O limite de tamanho se aplica aos arquivos que você carrega e também aos arquivos que são gerados como resultado do processamento do Media Services (codificação ou análise). Se o arquivo de origem for maior que 260 GB, o trabalho provavelmente falhará. 

A tabela a seguir mostra os limites das unidades reservadas de mídia S1, S2 e S3. Se o arquivo de origem for maior do que os limites definidos na tabela, seu trabalho de codificação falhará. Se você codificar fontes de resolução 4K de longa duração, você será obrigado a usar unidades reservadas de mídia S3 para alcançar o desempenho necessário. Se você tiver conteúdo 4K maior que o limite de 260 GB nas unidades reservadas de mídia S3, abra um bilhete de suporte.

|Tipo de unidade reservada de mídia|Tamanho máximo de entrada (GB)|
|---|---|
|S1 |    26|
|S2    | 60|
|S3    |260|

<sup>2</sup> As contas de armazenamento devem ser da mesma assinatura do Azure.

### <a name="jobs-encoding--analyzing-limits"></a>Limites de empregos (codificação & análise)

| Recurso | Limite padrão | 
| --- | --- | 
| Trabalhos por conta dos Serviços de Mídia | 500.000 <sup>(3)</sup> (fixo)|
| Entradas de emprego por Trabalho | 50  (fixo)|
| Saídas de emprego por Trabalho | 20 (fixo) |
| Transformações por conta dos Serviços de Mídia | 100  (fixo)|
| Transformar saídas em uma Transformação | 20 (fixo) |
| Arquivos por entrada de trabalho|10 (fixo)|

<sup>3</sup> Este número inclui empregos enfileirados, acabados, ativos e cancelados. Ele não inclui trabalhos excluídos. 

Qualquer registro de Trabalho em sua conta com mais de 90 dias será excluído automaticamente, mesmo que o número total de registros esteja abaixo da cota máxima. 

### <a name="live-streaming-limits"></a>Limites de transmissão ao vivo

| Recurso | Limite padrão | 
| --- | --- | 
| Eventos ao vivo <sup>(4)</sup> por conta de Serviços de Mídia |5|
| Saídas ao vivo por evento ao vivo |3 <sup>(5)</sup> |
| Duração máxima da saída ao vivo | 25 horas |

<sup>4</sup> Para obter informações detalhadas sobre as limitações do Evento Ao Vivo, consulte [comparação e limitações dos tipos de evento ao vivo](../articles/media-services/latest/live-event-types-comparison.md).

<sup>5</sup> Saídas ao vivo começam na criação e param quando excluídas.

### <a name="packaging--delivery-limits"></a>Limites de entrega de & de embalagens

| Recurso | Limite padrão | 
| --- | --- | 
| Pontos de extremidade de streaming (interrompidos ou em execução) por conta dos Serviços de Mídia|2 (fixo)|
| Filtros de Manifesto Dinâmico|100|
| Políticas de Streaming | 100 <sup>(6)</sup> |
| Localizadores de streaming exclusivos associados a um Ativo ao mesmo tempo | 100<sup>(7)</sup> (fixo) |

<sup>6</sup> Ao usar uma política de [streaming](https://docs.microsoft.com/rest/api/media/streamingpolicies)personalizada, você deve projetar um conjunto limitado de tais políticas para sua conta do Serviço de Mídia e reutilizá-las para seus StreamingLocators sempre que as mesmas opções e protocolos de criptografia forem necessários. Você não deve criar uma nova Política de Streaming para cada Localizador de Streaming.

<sup>7</sup> Os localizadores de streaming não foram projetados para gerenciar o controle de acesso por usuário. Para conceder direitos de acesso diferentes para usuários individuais, use as soluções de gerenciamento de direitos digitais (DRM).

### <a name="protection-limits"></a>Limites de proteção

| Recurso | Limite padrão | 
| --- | --- | 
| Opções por política de chave de conteúdo |30 | 
| Licenças por mês para cada um dos tipos de DRM em serviços de mídia serviço de entrega chave por conta|1.000.000|

### <a name="support-ticket"></a>Tíquete de suporte

Para recursos que não são fixos, você pode solicitar que as cotas sejam levantadas, mediante abertura de um [boleto de apoio.](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) Inclua informações detalhadas na solicitação sobre as alterações de cota desejadas, cenários de casos de uso e regiões necessárias. <br/>**Não** crie contas adicionais dos Serviços de Mídia do Azure em uma tentativa de obter limites mais altos.
