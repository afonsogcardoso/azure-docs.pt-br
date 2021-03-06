---
title: Carregar e indexar vídeos com o Video Indexer
titleSuffix: Azure Media Services
description: Este tópico demonstra como usar APIs para carregar e indexar seus vídeos com o Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 02/18/2020
ms.author: juliako
ms.openlocfilehash: 245eabdf4d77682c87062c2581239a554112d748
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77468755"
---
# <a name="upload-and-index-your-videos"></a>Carregar e indexar seus vídeos  

Ao carregar vídeos com a API do Video Indexer, você tem as seguintes opções de upload: 

* Carregue o vídeo de uma URL (preferido),
* envie o arquivo de vídeo como uma matriz de bytes no corpo da solicitação,
* Use o ativo existente do Azure Media Services fornecendo o [ID de ativos](https://docs.microsoft.com/azure/media-services/latest/assets-concept) (suportado apenas em contas pagas).

Uma vez que seu vídeo tenha sido carregado, o Video Indexer (opcionalmente) codifica o vídeo (discutido no artigo). Ao criar uma conta do Video Indexer, você pode escolher uma conta de avaliação gratuita (em que você obtém um determinado número de minutos de indexação gratuitos) ou uma opção paga (onde você não está limitado pela cota). Com o teste gratuito, o Video Indexer fornece até 600 minutos de indexação gratuita para usuários do site e até 2400 minutos de indexação gratuita para usuários da API. Com a opção paga, você cria uma conta do Video Indexer que está [conectada à sua assinatura do Azure e a uma conta dos Serviços de Mídia do Azure](connect-to-azure.md). Você paga por minutos indexados, bem como os encargos relacionados à conta de mídia. 

O artigo mostra como carregar e indexar seus vídeos com essas opções:

* [O site do Video Indexer](#website) 
* [As APIs do indexador de vídeo](#apis)

## <a name="uploading-considerations-and-limitations"></a>Carregando considerações e limitações
 
- Um nome do vídeo não deve ter mais de 80 caracteres.
- Ao carregar seu vídeo com base na URL (preferida) o ponto final deve ser protegido com TLS 1.2 (ou superior).
- O tamanho do upload com a opção URL é limitado a 30GB.
- O comprimento da URL de solicitação é limitado a 6144 caracteres onde o comprimento da URL da seqüência de caracteres da consulta é limitado a 4096 caracteres .
- O tamanho de upload com a opção byte array é limitado a 2GB.
- A opção de matriz de byte saem após 30 min.
- A URL fornecida `videoURL` no param precisa ser codificada.
- A indexação de ativos de Serviços de Mídia tem a mesma limitação que a indexação da URL.
- O Indexador de Vídeo tem um limite máximo de duração de 4 horas para um único arquivo.
- A URL precisa ser acessível (por exemplo, uma URL pública). 

    Se for uma URL privada, o token de acesso precisa ser fornecido na solicitação.
- A URL tem que apontar para um arquivo de mídia válido e `www.youtube.com` não para uma página da Web, como um link para a página.
- Em uma conta paga você pode carregar até 50 filmes por minuto, e em uma conta de teste até 5 filmes por minuto.

> [!Tip]
> É recomendável usar o .NET Framework versão 4.6.2. ou superior porque versões mais antigas do .NET Framework não usam TLS 1.2 por padrão.
>
> Se você precisar usar versões mais antigas do .NET Framework, adicione uma linha em seu código antes de fazer a chamada à API REST:  <br/> System.Net.ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls | SecurityProtocolType.Tls11 | SecurityProtocolType.Tls12;

## <a name="supported-file-formats-for-video-indexer"></a>Formatos de arquivo compatíveis com o Video Indexer

Confira o artigo [contêiner de entrada/formatos de arquivo](../latest/media-encoder-standard-formats.md#input-containerfile-formats) para obter uma lista de formatos de arquivo que você pode usar com o Video Indexer.

## <a name="upload-and-index-a-video-using-the-video-indexer-website"></a><a id="website"/>Faça upload e indexe um vídeo usando o site do Video Indexer

> [!NOTE]
> Um nome do vídeo não deve ter mais de 80 caracteres.

1. Conecte-se ao site do [Video Indexer](https://www.videoindexer.ai/).
2. Para carregar um vídeo, pressione o botão ou o link **Carregar**.

    ![Carregar](./media/video-indexer-get-started/video-indexer-upload.png)

    Depois que o seu vídeo tiver sido carregado, o Video Indexer iniciará a indexação e a análise do vídeo.

    ![Carregado](./media/video-indexer-get-started/video-indexer-uploaded.png) 

    Depois que o Video Indexer terminar de analisar, você receberá uma notificação com um link para seu vídeo e uma breve descrição do que foi encontrado nele. Por exemplo: people (pessoas), topics (tópicos), OCRs.

## <a name="upload-and-index-with-api"></a><a id="apis"/>Upload e índice com API

Use a API [de vídeo Upload](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) para carregar e indexar seus vídeos com base em uma URL. A amostra de código a seguir inclui o código de saída comentado que mostra como carregar a matriz de bytes. 

### <a name="configurations-and-params"></a>Configurações e parâmetros

Esta seção descreve alguns parâmetros opcionais e quando você deseja defini-los.

#### <a name="externalid"></a>externalID 

Este parâmetro permite que você especifique uma ID que será associada ao vídeo. A ID pode ser aplicada à integração de sistema externa VCM (Gerenciamento de Conteúdo de Vídeo). Os vídeos localizados no portal do Video Indexer podem ser pesquisados usando a ID externa especificada.

#### <a name="callbackurl"></a>callbackUrl

Uma URL usada para notificar o cliente (usando uma solicitação POST) sobre os eventos a seguir:

- Alteração de estado de indexação: 
    - Propriedades:    
    
        |Nome|Descrição|
        |---|---|
        |id|O vídeo ID|
        |state|O estado do vídeo|  
    - Exemplo: https:\//test.com/notifyme?projectName=MyProject&id=1234abcd&state=Processed
- Pessoa identificada no vídeo:
  - Propriedades
    
      |Nome|Descrição|
      |---|---|
      |id| O vídeo ID|
      |faceId|A identificação de face que aparece no índice de vídeo|
      |knownPersonId|A ID da pessoa que é exclusiva dentro de um modelo de detecção facial|
      |personName|O nome da pessoa|
        
    - Exemplo: https:\//test.com/notifyme?projectName=MyProject&id=1234abcd&faceid=12&conhecidoPersonId=CCA84350-89B7-4262-861C-3CAC796542A5&personName=Inigo_Montoya 

##### <a name="notes"></a>Observações

- O Video Indexer retorna quaisquer parâmetros existentes fornecidos na URL original.
- A URL fornecida deve ser codificada.

#### <a name="indexingpreset"></a>indexingPreset

Use esse parâmetro se gravações brutas ou externas contiverem ruídos de fundo. Esse parâmetro é usado para configurar o processo de indexação. É possível especificar os seguintes valores:

- `AudioOnly` – Indexar e extrair insights usando apenas áudio (ignorando vídeo)
- `VideoOnly`- Indexar e extrair insights usando apenas vídeo (ignorando áudio)
- `Default` – Indexar e extrair insights usando áudio e vídeo
- `DefaultWithNoiseReduction` – Indexar e extrair insights de áudio e vídeo durante a aplicação de algoritmos de redução de ruído no fluxo de áudio

> [!NOTE]
> O Video Indexer cobre até duas faixas de áudio. Se houver mais faixas de áudio no arquivo, elas serão tratadas como uma faixa.<br/>
Se você quiser indexar as faixas separadamente, você precisará extrair `AudioOnly`o arquivo de áudio relevante e indexá-lo como .

O preço depende da opção de indexação selecionada.  

#### <a name="priority"></a>priority

Os vídeos são indexados pelo Video Indexer de acordo com a prioridade. Use o parâmetro **priority** para especificar a prioridade de índice. Os seguintes valores são válidos: **Baixo**, **Normal** (padrão) e **Alto**.

O parâmetro **priority** tem suporte apenas para contas pagas.

#### <a name="streamingpreset"></a>streamingPreset

Depois que o vídeo tiver sido carregado, o Video Indexer codificará opcionalmente o vídeo. Em seguida, passe para a indexação e análise do vídeo. Quando o Video Indexer terminar a análise, você receberá uma notificação com a ID do vídeo.  

Ao usar a API [Carregar vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) ou [Reindexar vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?), um dos parâmetros opcionais é `streamingPreset`. Se você definir `streamingPreset` como `Default`, `SingleBitrate` ou `AdaptiveBitrate`, o processo de codificação será disparado. Depois que os trabalhos de indexação e codificação forem concluídos, o vídeo será publicado para que você também possa transmiti-lo. O Ponto de extremidade de streaming do qual você deseja transmitir o vídeo deve estar no estado **Executando**.

Para executar trabalhos de indexação e de codificação, a [conta dos Serviços de Mídia do Azure conectada à sua conta do Video Indexer](connect-to-azure.md), requer unidades reservadas. Para obter mais informações, consulte [Scaling Media Processing](https://docs.microsoft.com/azure/media-services/previous/media-services-scale-media-processing-overview) (Colocação em escala do processamento de mídia). Como esses são trabalhos com uso intensivo de computação, o tipo de unidade S3 é altamente recomendável. O número de RUs define o número máximo de trabalhos que podem ser executados em paralelo. A recomendação de linha de base é de 10 RUs de S3. 

Se você apenas deseja indexar seu vídeo, mas não o codificar, defina `streamingPreset` como `NoStreaming`.

#### <a name="videourl"></a>VideoUrl

Uma URL do arquivo de áudio/vídeo a serem indexados. O URL deve apontar para um arquivo de mídia (as páginas HTML não são suportadas). O arquivo pode ser protegido por um token de acesso fornecido como parte do URI e o terminal que atende ao arquivo deve ser protegido com o TLS 1.2 ou superior. A URL deve ser codificado. 

Se o `videoUrl` não for especificado, o Indexador de Vídeo espera que você passe o arquivo como um conteúdo de corpo de várias partes / formulário.

### <a name="code-sample"></a>Exemplo de código

O seguinte snippet de código em C# demonstra o uso de todas as APIs do indexador de vídeo juntos.

#### <a name="instructions-for-running-this-code-sample"></a>Instruções para executar esta amostra de código

Depois de copiar esse código em sua plataforma de desenvolvimento, você precisará fornecer dois parâmetros: chave de autenticação de gerenciamento de API e URL de vídeo.

* Chave de API – A chave da API é a chave de assinatura de gerenciamento de API pessoal, que permitirá que você obtenha um token de acesso para executar operações em sua conta do Indexador de Vídeo. 

    Para obter sua chave de API, passe por esse fluxo:

    * Navegue até https://api-portal.videoindexer.ai/
    * Logon
    * Ir para a**assinatura de autorização de** **produtos** -> **Authorization** -> 
    * Copie a **chave principal**
* URL de vídeo – Uma URL do arquivo de vídeo/áudio a ser indexado. O URL deve apontar para um arquivo de mídia (as páginas HTML não são suportadas). O arquivo pode ser protegido por um token de acesso fornecido como parte do URI e o terminal que atende ao arquivo deve ser protegido com o TLS 1.2 ou superior. A URL deve ser codificado.

O resultado da execução com sucesso da amostra de código incluirá uma URL de widget de insight e uma URL de widget do jogador que permitirá examinar os insights e o vídeo carregados, respectivamente. 


```csharp
public async Task Sample()
{
    var apiUrl = "https://api.videoindexer.ai";
    var apiKey = "..."; // replace with API key taken from https://aka.ms/viapi

    System.Net.ServicePointManager.SecurityProtocol =
        System.Net.ServicePointManager.SecurityProtocol | System.Net.SecurityProtocolType.Tls12;

    // create the http client
    var handler = new HttpClientHandler();
    handler.AllowAutoRedirect = false;
    var client = new HttpClient(handler);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);

    // obtain account information and access token
    string queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"generateAccessTokens", "true"},
            {"allowEdit", "true"},
        });
    HttpResponseMessage result = await client.GetAsync($"{apiUrl}/auth/trial/Accounts?{queryParams}");
    var json = await result.Content.ReadAsStringAsync();
    var accounts = JsonConvert.DeserializeObject<AccountContractSlim[]>(json);
    
    // take the relevant account, here we simply take the first, 
    // you can also get the account via accounts.First(account => account.Id == <GUID>);
    var accountInfo = accounts.First();

    // we will use the access token from here on, no need for the apim key
    client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

    // upload a video
    var content = new MultipartFormDataContent();
    Console.WriteLine("Uploading...");
    // get the video from URL
    var videoUrl = "VIDEO_URL"; // replace with the video URL

    // as an alternative to specifying video URL, you can upload a file.
    // remove the videoUrl parameter from the query params below and add the following lines:
    //FileStream video =File.OpenRead(Globals.VIDEOFILE_PATH);
    //byte[] buffer =new byte[video.Length];
    //video.Read(buffer, 0, buffer.Length);
    //content.Add(new ByteArrayContent(buffer));

    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", accountInfo.AccessToken},
            {"name", "video_name"},
            {"description", "video_description"},
            {"privacy", "private"},
            {"partition", "partition"},
            {"videoUrl", videoUrl},
        });
    var uploadRequestResult = await client.PostAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos?{queryParams}", content);
    var uploadResult = await uploadRequestResult.Content.ReadAsStringAsync();

    // get the video ID from the upload result
    string videoId = JsonConvert.DeserializeObject<dynamic>(uploadResult)["id"];
    Console.WriteLine("Uploaded");
    Console.WriteLine("Video ID:");
    Console.WriteLine(videoId);

    // wait for the video index to finish
    while (true)
    {
        await Task.Delay(10000);

        queryParams = CreateQueryString(
            new Dictionary<string, string>()
            {
                {"accessToken", accountInfo.AccessToken},
                {"language", "English"},
            });

        var videoGetIndexRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/Index?{queryParams}");
        var videoGetIndexResult = await videoGetIndexRequestResult.Content.ReadAsStringAsync();

        string processingState = JsonConvert.DeserializeObject<dynamic>(videoGetIndexResult)["state"];

        Console.WriteLine("");
        Console.WriteLine("State:");
        Console.WriteLine(processingState);

        // job is finished
        if (processingState != "Uploaded" && processingState != "Processing")
        {
            Console.WriteLine("");
            Console.WriteLine("Full JSON:");
            Console.WriteLine(videoGetIndexResult);
            break;
        }
    }

    // search for the video
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", accountInfo.AccessToken},
            {"id", videoId},
        });

    var searchRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/Search?{queryParams}");
    var searchResult = await searchRequestResult.Content.ReadAsStringAsync();
    Console.WriteLine("");
    Console.WriteLine("Search:");
    Console.WriteLine(searchResult);

    // Generate video access token (used for get widget calls)
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);
    var videoTokenRequestResult = await client.GetAsync($"{apiUrl}/auth/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/AccessToken?allowEdit=true");
    var videoAccessToken = (await videoTokenRequestResult.Content.ReadAsStringAsync()).Replace("\"", "");
    client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

    // get insights widget url
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", videoAccessToken},
            {"widgetType", "Keywords"},
            {"allowEdit", "true"},
        });
    var insightsWidgetRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/InsightsWidget?{queryParams}");
    var insightsWidgetLink = insightsWidgetRequestResult.Headers.Location;
    Console.WriteLine("Insights Widget url:");
    Console.WriteLine(insightsWidgetLink);

    // get player widget url
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", videoAccessToken},
        });
    var playerWidgetRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/PlayerWidget?{queryParams}");
    var playerWidgetLink = playerWidgetRequestResult.Headers.Location;
     Console.WriteLine("");
     Console.WriteLine("Player Widget url:");
     Console.WriteLine(playerWidgetLink);
     Console.WriteLine("\nPress Enter to exit...");
     String line = Console.ReadLine();
     if (line == "enter")
     {
         System.Environment.Exit(0);
     }

}

private string CreateQueryString(IDictionary<string, string> parameters)
{
    var queryParameters = HttpUtility.ParseQueryString(string.Empty);
    foreach (var parameter in parameters)
    {
        queryParameters[parameter.Key] = parameter.Value;
    }

    return queryParameters.ToString();
}

public class AccountContractSlim
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Location { get; set; }
    public string AccountType { get; set; }
    public string Url { get; set; }
    public string AccessToken { get; set; }
}
```

### <a name="common-errors"></a>Erros comuns

Os códigos de status listados na tabela a seguir podem ser retornados pela operação de Upload.

|Código de status|ErrorType (no corpo da resposta)|Descrição|
|---|---|---|
|409|VIDEO_INDEXING_IN_PROGRESS|O mesmo vídeo já está em curso de ser processado na conta especificada.|
|400|VIDEO_ALREADY_FAILED|O mesmo vídeo falhou ao processar na conta informada há menos de duas horas. Os clientes da API devem aguardar pelo menos duas horas antes de carregar novamente um vídeo.|
|429||Contas de teste são permitidas 5 uploads por minuto. Contas pagas são permitidas 50 uploads por minuto.|

## <a name="next-steps"></a>Próximas etapas

[Examine a saída do Indexador de Vídeo Azure produzido pela API](video-indexer-output-json-v2.md)
