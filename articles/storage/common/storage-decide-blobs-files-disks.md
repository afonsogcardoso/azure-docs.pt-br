---
title: Decidindo quando usar Blobs do Azure, Arquivos do Azure ou Discos do Azure
description: Saiba mais sobre as diferentes maneiras de armazenar e acessar dados no Azure para ajudá-lo a decidir qual tecnologia será usada.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 11/28/2018
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: 4b1a42e25a6d8c7b4a3c24dffcb858ffe63dd10b
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "71671044"
---
# <a name="deciding-when-to-use-azure-blobs-azure-files-or-azure-disks"></a>Decidindo quando usar Blobs do Azure, Arquivos do Azure ou Discos do Azure

O Microsoft Azure fornece vários recursos no Armazenamento do Azure para armazenar e acessar seus dados na nuvem. Este artigo aborda os Arquivos, Blobs e Discos do Azure e foi elaborado para ajudá-lo a escolher entre esses recursos.

## <a name="scenarios"></a>Cenários

A tabela a seguir compara os Arquivos, Blobs e Discos, e mostra cenários de exemplo apropriados para cada um.

| Recurso | Descrição | Quando usar |
|--------------|-------------|-------------|
| **Arquivos do Azure** | Fornece uma interface SMB, bibliotecas de clientes e uma [interface REST](/rest/api/storageservices/file-service-rest-api) que permite o acesso aos arquivos armazenados em qualquer lugar. | Você deseja migrar por lift-and-shift um aplicativo para a nuvem que já usa as APIs do sistema de arquivos nativo para compartilhar dados entre ele e outros aplicativos em execução no Azure.<br/><br/>Você deseja armazenar ferramentas de desenvolvimento e depuração que precisam ser acessadas em várias máquinas virtuais. |
| **Blobs do Azure** | Fornece bibliotecas de clientes e uma [interface REST](/rest/api/storageservices/blob-service-rest-api) que permite que dados não estruturados sejam armazenados e acessados em grande escala em blobs de blocos.<br/><br/>Também dá suporte ao [Azure Data Lake Storage Gen2](../blobs/data-lake-storage-introduction.md) para soluções de análise de big data do enterprise. | Você deseja que o aplicativo dê suporte a cenários de streaming e de acesso aleatório.<br/><br/>Você deseja poder acessar dados do aplicativo em qualquer lugar.<br/><br/>Você deseja criar um lago de dados empresariais no Azure e executar análise de big data. |
| **Discos do Azure** | Fornece bibliotecas de clientes e uma [interface REST](/rest/api/compute/manageddisks/disks/disks-rest-api) que permite que os dados sejam armazenados e acessados de forma persistente e acessadas a partir de um disco rígido virtual conectado. | Você deseja migrar por lift-and-shift aplicativos que usam APIs do sistema de arquivos nativo para ler e gravar dados em discos persistentes.<br/><br/>Você deseja armazenar os dados que não precisam ser acessados fora da máquina virtual à qual o disco está anexado. |


## <a name="next-steps"></a>Próximas etapas

Ao tomar decisões sobre como os dados são armazenados e acessados, você também deve considerar os custos envolvidos. Para obter mais informações, consulte [Preços do Armazenamento do Azure](https://azure.microsoft.com/pricing/details/storage/).
  
Alguns recursos do SMB não são aplicáveis à nuvem. Para obter mais informações, consulte [Recursos sem suporte no serviço Arquivo do Azure](/rest/api/storageservices/features-not-supported-by-the-azure-file-service).
 
Para obter mais informações sobre o Azure Blobs, consulte nosso artigo, [O que é armazenamento Azure Blob?](../blobs/storage-blobs-overview.md)

Para obter mais informações sobre armazenamento de disco, consulte nossa [introdução aos discos gerenciados](../../virtual-machines/windows/managed-disks-overview.md).

Para obter mais informações sobre o Azure Files, consulte nosso artigo, [Planning for a Azure Files deployment](../files/storage-files-planning.md).