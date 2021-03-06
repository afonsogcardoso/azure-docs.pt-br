---
title: Listar recipientes blob com .NET - Azure Storage
description: Saiba como listar recipientes blob em sua conta do Azure Storage usando a biblioteca de clientes .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 01/06/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 155b8f5d50c7b106daff8dab4df17200b844c988
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79135897"
---
# <a name="list-blob-containers-with-net"></a>Listar recipientes blob com .NET

Quando você lista os contêineres em uma conta do Azure Storage a partir do seu código, você pode especificar uma série de opções para gerenciar como os resultados são retornados do Azure Storage. Este artigo mostra como listar contêineres usando a [biblioteca cliente do Azure Storage para .NET](/dotnet/api/overview/azure/storage?view=azure-dotnet).  

## <a name="understand-container-listing-options"></a>Entenda as opções de listagem de contêineres

Para listar contêineres em sua conta de armazenamento, ligue para um dos seguintes métodos:

- [Contêineres de listasegmentados](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmented)
- [ListContainersSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmentedasync)

As sobrecargas desses métodos fornecem opções adicionais para gerenciar como os contêineres são devolvidos pela operação de listagem. Essas opções são descritas nas seções a seguir.

### <a name="manage-how-many-results-are-returned"></a>Gerencie quantos resultados são devolvidos

Por padrão, uma operação de listagem retorna até 5000 resultados por vez. Para retornar um conjunto menor de resultados, forneça `maxresults` um valor não zero para o parâmetro ao chamar um dos métodos **ListContainerSegmented.**

Se sua conta de armazenamento contiver mais de 5000 contêineres ou se você tiver especificado um valor para `maxresults` tal forma que a operação de listagem retorne um subconjunto de contêineres na conta de armazenamento, então o Azure Storage retorna um token de *continuação* com a lista de contêineres. Um token de continuação é um valor opaco que você pode usar para recuperar o próximo conjunto de resultados do Azure Storage.

Em seu código, verifique o valor do token de continuação para determinar se ele é nulo. Quando o token de continuação é nulo, então o conjunto de resultados está completo. Se o token de continuação não for nulo, ligue novamente para **ListContainersSegmented** ou **ListContainersSegmentedAsync,** passando no token de continuação para recuperar o próximo conjunto de resultados, até que o token de continuação seja nulo.

### <a name="filter-results-with-a-prefix"></a>Filtrar resultados com um prefixo

Para filtrar a lista de recipientes, especifique uma string para o `prefix` parâmetro. A seqüência de prefixo pode incluir um ou mais caracteres. O Azure Storage então retorna apenas os contêineres cujos nomes começam com esse prefixo.

### <a name="return-metadata"></a>Metadados de retorno

Para retornar metadados de contêiner com os resultados, especifique o valor **de Metadados** para a enumeração [ContainerListingDetails.](/dotnet/api/microsoft.azure.storage.blob.containerlistingdetails) O Azure Storage inclui metadados com cada contêiner retornado, portanto, você não precisa também chamar um dos **métodos FetchAttributes** para recuperar os metadados do contêiner.

## <a name="example-list-containers"></a>Exemplo: Contêineres de lista

O exemplo a seguir lista assíncronamente os contêineres em uma conta de armazenamento que começam com um prefixo especificado. O exemplo lista contêineres em incrementos de 5 resultados de cada vez e usa o token de continuação para obter o próximo segmento de resultados. O exemplo também retorna metadados de contêiner com os resultados.

```csharp
private static async Task ListContainersWithPrefixAsync(CloudBlobClient blobClient,
                                                        string prefix)
{
    Console.WriteLine("List all containers beginning with prefix {0}, plus container metadata:", prefix);

    try
    {
        ContainerResultSegment resultSegment = null;
        BlobContinuationToken continuationToken = null;

        do
        {
            // List containers beginning with the specified prefix, returning segments of 5 results each.
            // Passing null for the maxResults parameter returns the max number of results (up to 5000).
            // Requesting the container's metadata with the listing operation populates the metadata,
            // so it's not necessary to also call FetchAttributes() to read the metadata.
            resultSegment = await blobClient.ListContainersSegmentedAsync(
                prefix, ContainerListingDetails.Metadata, 5, continuationToken, null, null);

            // Enumerate the containers returned.
            foreach (var container in resultSegment.Results)
            {
                Console.WriteLine("\tContainer:" + container.Name);

                // Write the container's metadata keys and values.
                foreach (var metadataItem in container.Metadata)
                {
                    Console.WriteLine("\t\tMetadata key: " + metadataItem.Key);
                    Console.WriteLine("\t\tMetadata value: " + metadataItem.Value);
                }
            }

            // Get the continuation token. If not null, get the next segment.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0} : {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }
}
```

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Confira também

[Listar contêineres](/rest/api/storageservices/list-containers2)
[enumerando recursos blob](/rest/api/storageservices/enumerating-blob-resources)
