---
title: Azure Service Fabric CLI- volume de malha sfctl
description: Saiba mais sobre a sfctl, a interface de linha de comando Azure Service Fabric. Inclui uma lista de comandos para obter e excluir recursos de volume.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 0e5ba1a311bd0480c56ee40edf01fc13ec01664a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76905916"
---
# <a name="sfctl-mesh-volume"></a>sfctl mesh volume
Obter e excluir recursos do volume.

## <a name="commands"></a>Comandos

|Comando|Descrição|
| --- | --- |
| excluir | Exclui o recurso do volume. |
| list | Lista todos os recursos do volume. |
| mostrar | Obtém o recurso de volume com o nome fornecido. |

## <a name="sfctl-mesh-volume-delete"></a>sfctl mesh volume delete
Exclui o recurso do volume.

Exclui o recurso de volume identificado pelo nome.

### <a name="arguments"></a>Argumentos

|Argumento|Descrição|
| --- | --- |
| --name -n [Obrigatório] | O nome do volume. |

### <a name="global-arguments"></a>Argumentos globais

|Argumento|Descrição|
| --- | --- |
| --debug | Aumente o detalhamento do log para mostrar todos os logs de depuração. |
| --help -h | Mostrar esta mensagem de ajuda e sair. |
| --output -o | Formato de saída.  Valores permitidos\: json, jsonc, tabela, tsv.  Padrão\: json. |
| --query | Cadeia de caracteres de consulta JMESPath. Veja http\://jmespath.org/ para saber mais e obter exemplos. |
| --verbose | Aumentar o detalhamento do log. Use --debug para logs de depuração completos. |

## <a name="sfctl-mesh-volume-list"></a>sfctl mesh volume list
Lista todos os recursos do volume.

Obtém as informações sobre todos os recursos de volume em um determinado grupo de recursos. As informações incluem a descrição e outras propriedades do volume.

### <a name="global-arguments"></a>Argumentos globais

|Argumento|Descrição|
| --- | --- |
| --debug | Aumente o detalhamento do log para mostrar todos os logs de depuração. |
| --help -h | Mostrar esta mensagem de ajuda e sair. |
| --output -o | Formato de saída.  Valores permitidos\: json, jsonc, tabela, tsv.  Padrão\: json. |
| --query | Cadeia de caracteres de consulta JMESPath. Veja http\://jmespath.org/ para saber mais e obter exemplos. |
| --verbose | Aumentar o detalhamento do log. Use --debug para logs de depuração completos. |

## <a name="sfctl-mesh-volume-show"></a>sfctl mesh volume show
Obtém o recurso de volume com o nome fornecido.

Obtém as informações sobre o recurso de volume com o nome fornecido. As informações incluem a descrição e outras propriedades do volume.

### <a name="arguments"></a>Argumentos

|Argumento|Descrição|
| --- | --- |
| --name -n [Obrigatório] | O nome do volume. |

### <a name="global-arguments"></a>Argumentos globais

|Argumento|Descrição|
| --- | --- |
| --debug | Aumente o detalhamento do log para mostrar todos os logs de depuração. |
| --help -h | Mostrar esta mensagem de ajuda e sair. |
| --output -o | Formato de saída.  Valores permitidos\: json, jsonc, tabela, tsv.  Padrão\: json. |
| --query | Cadeia de caracteres de consulta JMESPath. Veja http\://jmespath.org/ para saber mais e obter exemplos. |
| --verbose | Aumentar o detalhamento do log. Use --debug para logs de depuração completos. |


## <a name="next-steps"></a>Próximas etapas
- [Configure](service-fabric-cli.md) o CLI do tecido de serviço.
- Saiba como usar a CLI do Service Fabric usando os [scripts de exemplo](/azure/service-fabric/scripts/sfctl-upgrade-application).