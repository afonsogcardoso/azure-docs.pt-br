---
title: Funções de modelo - implantação
description: Descreve as funções a serem usadas em um modelo do Resource Manager para recuperar informações sobre implantação.
ms.topic: conceptual
ms.date: 11/27/2019
ms.openlocfilehash: 86a1d3d7e05fedacd7a3c044ecab241ca9d059c5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80156320"
---
# <a name="deployment-functions-for-arm-templates"></a>Funções de implantação para modelos ARM 

O Resource Manager fornece as seguintes funções para obter valores relacionados à implantação atual do modelo ARM (Azure Resource Manager, gerenciador de recursos do Azure):

* [Implantação](#deployment)
* [Ambiente](#environment)
* [Parâmetros](#parameters)
* [Variáveis](#variables)

Para obter valores de recursos, de grupos de recursos ou de assinaturas, veja [Funções de recurso](template-functions-resource.md).

## <a name="deployment"></a>implantação

`deployment()`

Retorna informações sobre a operação de implantação atual.

### <a name="return-value"></a>Valor retornado

Essa função retorna o objeto que é passado durante a implantação. As propriedades no objeto retornado vão variar dependendo se o objeto de implantação for transmitido como um link ou como um objeto na linha. Quando o objeto de implantação é passado na linha, como ao usar o parâmetro **-TemplateFile** no Azure PowerShell para apontar para um arquivo local, o objeto retornado tem no seguinte formato:

```json
{
    "name": "",
    "properties": {
        "template": {
            "$schema": "",
            "contentVersion": "",
            "parameters": {},
            "variables": {},
            "resources": [
            ],
            "outputs": {}
        },
        "parameters": {},
        "mode": "",
        "provisioningState": ""
    }
}
```

Quando o objeto é passado como um link, como ao usar o parâmetro **-TemplateUri** para apontar para um objeto remoto, o objeto é retornado no seguinte formato: 

```json
{
    "name": "",
    "properties": {
        "templateLink": {
            "uri": ""
        },
        "template": {
            "$schema": "",
            "contentVersion": "",
            "parameters": {},
            "variables": {},
            "resources": [],
            "outputs": {}
        },
        "parameters": {},
        "mode": "",
        "provisioningState": ""
    }
}
```

Ao [implantar em uma assinatura do Azure](deploy-to-subscription.md), em vez de um grupo de recursos, o objeto retornado inclui uma propriedade `location`. A propriedade de localização é incluída ao implantar um modelo local ou externo.

### <a name="remarks"></a>Comentários

Você pode usar a implantação() para vincular a outro modelo com base no URI do modelo pai.

```json
"variables": {  
    "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"  
}
```  

Se você reimplantar um modelo usando o histórico de implantação no portal, o modelo será implantando como um arquivo local. A propriedade `templateLink` não é retornada na função de implantação. Se o seu modelo depender de `templateLink` para criar um link para outro modelo, não use o portal para reimplantação. Ao invés disso, emita os comandos usados para implantar originalmente o modelo.

### <a name="example"></a>Exemplo

O [modelo de exemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/deployment.json) a seguir retorna o objeto de implantação:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "subscriptionOutput": {
            "value": "[deployment()]",
            "type" : "object"
        }
    }
}
```

O exemplo anterior retorna o seguinte objeto:

```json
{
  "name": "deployment",
  "properties": {
    "template": {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [],
      "outputs": {
        "subscriptionOutput": {
          "type": "Object",
          "value": "[deployment()]"
        }
      }
    },
    "parameters": {},
    "mode": "Incremental",
    "provisioningState": "Accepted"
  }
}
```

Para um modelo no nível da assinatura, que usa a função de implantação, confira [função de implantação de assinatura](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/deploymentsubscription.json). Ele é implantado com os comandos `az deployment create` ou `New-AzDeployment`.

## <a name="environment"></a>environment

`environment()`

Retorna informações sobre o ambiente Azure usado para implantação.

### <a name="return-value"></a>Valor retornado

Esta função retorna propriedades para o ambiente Azure atual. O exemplo a seguir mostra as propriedades do Azure global. Nuvens soberanas podem retornar propriedades ligeiramente diferentes.

```json
{
  "name": "",
  "gallery": "",
  "graph": "",
  "portal": "",
  "graphAudience": "",
  "activeDirectoryDataLake": "",
  "batch": "",
  "media": "",
  "sqlManagement": "",
  "vmImageAliasDoc": "",
  "resourceManager": "",
  "authentication": {
    "loginEndpoint": "",
    "audiences": [
      "",
      ""
    ],
    "tenant": "",
    "identityProvider": ""
  },
  "suffixes": {
    "acrLoginServer": "",
    "azureDatalakeAnalyticsCatalogAndJob": "",
    "azureDatalakeStoreFileSystem": "",
    "azureFrontDoorEndpointSuffix": "",
    "keyvaultDns": "",
    "sqlServerHostname": "",
    "storage": ""
  }
}
```

### <a name="example"></a>Exemplo

O modelo de exemplo a seguir retorna o objeto do ambiente.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "environmentOutput": {
            "value": "[environment()]",
            "type" : "object"
        }
    }
}
```

O exemplo anterior retorna o seguinte objeto quando implantado no Azure global:

```json
{
  "name": "AzureCloud",
  "gallery": "https://gallery.azure.com/",
  "graph": "https://graph.windows.net/",
  "portal": "https://portal.azure.com",
  "graphAudience": "https://graph.windows.net/",
  "activeDirectoryDataLake": "https://datalake.azure.net/",
  "batch": "https://batch.core.windows.net/",
  "media": "https://rest.media.azure.net",
  "sqlManagement": "https://management.core.windows.net:8443/",
  "vmImageAliasDoc": "https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json",
  "resourceManager": "https://management.azure.com/",
  "authentication": {
    "loginEndpoint": "https://login.windows.net/",
    "audiences": [
      "https://management.core.windows.net/",
      "https://management.azure.com/"
    ],
    "tenant": "common",
    "identityProvider": "AAD"
  },
  "suffixes": {
    "acrLoginServer": ".azurecr.io",
    "azureDatalakeAnalyticsCatalogAndJob": "azuredatalakeanalytics.net",
    "azureDatalakeStoreFileSystem": "azuredatalakestore.net",
    "azureFrontDoorEndpointSuffix": "azurefd.net",
    "keyvaultDns": ".vault.azure.net",
    "sqlServerHostname": ".database.windows.net",
    "storage": "core.windows.net"
  }
}
```

## <a name="parameters"></a>parâmetros

`parameters(parameterName)`

Retorna um valor de parâmetro. O nome do parâmetro especificado deve ser definido na seção de parâmetros do modelo.

### <a name="parameters"></a>Parâmetros

| Parâmetro | Obrigatório | Type | Descrição |
|:--- |:--- |:--- |:--- |
| parameterName |Sim |string |O nome do parâmetro a retornar. |

### <a name="return-value"></a>Valor retornado

O valor do parâmetro especificado.

### <a name="remarks"></a>Comentários

Normalmente, você usa parâmetros para definir valores de recursos. O exemplo a seguir define o nome do site para o valor do parâmetro passado durante a implantação.

```json
"parameters": { 
  "siteName": {
      "type": "string"
  }
},
"resources": [
   {
      "apiVersion": "2016-08-01",
      "name": "[parameters('siteName')]",
      "type": "Microsoft.Web/Sites",
      ...
   }
]
```

### <a name="example"></a>Exemplo

O [modelo de exemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/parameters.json) a seguir mostra um uso simplificado da função parameters.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringParameter": {
            "type" : "string",
            "defaultValue": "option 1"
        },
        "intParameter": {
            "type": "int",
            "defaultValue": 1
        },
        "objectParameter": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "b"}
        },
        "arrayParameter": {
            "type": "array",
            "defaultValue": [1, 2, 3]
        },
        "crossParameter": {
            "type": "string",
            "defaultValue": "[parameters('stringParameter')]"
        }
    },
    "variables": {},
    "resources": [],
    "outputs": {
        "stringOutput": {
            "value": "[parameters('stringParameter')]",
            "type" : "string"
        },
        "intOutput": {
            "value": "[parameters('intParameter')]",
            "type" : "int"
        },
        "objectOutput": {
            "value": "[parameters('objectParameter')]",
            "type" : "object"
        },
        "arrayOutput": {
            "value": "[parameters('arrayParameter')]",
            "type" : "array"
        },
        "crossOutput": {
            "value": "[parameters('crossParameter')]",
            "type" : "string"
        }
    }
}
```

A saída do exemplo anterior com os valores padrão é:

| Nome | Type | Valor |
| ---- | ---- | ----- |
| stringOutput | String | opção 1 |
| intOutput | Int | 1 |
| objectOutput | Objeto | {"one": "a", "two": "b"} |
| arrayOutput | Array | [1, 2, 3] |
| crossOutput | String | opção 1 |

Para obter mais informações sobre o uso de parâmetros, consulte [Parâmetros no modelo do Azure Resource Manager](template-parameters.md).

## <a name="variables"></a>variáveis

`variables(variableName)`

Retorna o valor da variável. O nome do parâmetro especificado deve ser definido na seção variáveis do modelo.

### <a name="parameters"></a>Parâmetros

| Parâmetro | Obrigatório | Type | Descrição |
|:--- |:--- |:--- |:--- |
| variableName |Sim |String |O nome da variável a retornar. |

### <a name="return-value"></a>Valor retornado

O valor da variável especificada.

### <a name="remarks"></a>Comentários

Normalmente, você usa variáveis para simplificar seu modelo criando valores complexos apenas uma vez. O exemplo a seguir constrói um nome exclusivo para uma conta de armazenamento.

```json
"variables": {
    "storageName": "[concat('storage', uniqueString(resourceGroup().id))]"
},
"resources": [
    {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[variables('storageName')]",
        ...
    },
    {
        "type": "Microsoft.Compute/virtualMachines",
        "dependsOn": [
            "[variables('storageName')]"
        ],
        ...
    }
],
```

### <a name="example"></a>Exemplo

O [modelo de exemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/variables.json) a seguir retorna valores de variáveis diferentes.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {
        "var1": "myVariable",
        "var2": [ 1,2,3,4 ],
        "var3": "[ variables('var1') ]",
        "var4": {
            "property1": "value1",
            "property2": "value2"
          }
    },
    "resources": [],
    "outputs": {
        "exampleOutput1": {
            "value": "[variables('var1')]",
            "type" : "string"
        },
        "exampleOutput2": {
            "value": "[variables('var2')]",
            "type" : "array"
        },
        "exampleOutput3": {
            "value": "[variables('var3')]",
            "type" : "string"
        },
        "exampleOutput4": {
            "value": "[variables('var4')]",
            "type" : "object"
        }
    }
}
```

A saída do exemplo anterior com os valores padrão é:

| Nome | Type | Valor |
| ---- | ---- | ----- |
| exampleOutput1 | String | myVariable |
| exampleOutput2 | Array | [1, 2, 3, 4] |
| exampleOutput3 | String | myVariable |
| exampleOutput4 |  Objeto | {"property1": "value1", "property2": "value2"} |

Para obter mais informações sobre o uso de variáveis, consulte [Variáveis no modelo do Azure Resource Manager](template-variables.md).

## <a name="next-steps"></a>Próximas etapas
* Para obter uma descrição das seções em um modelo do Azure Resource Manager, consulte [Os modelos do Azure Resource Manager](template-syntax.md).
* Para mesclar diversos modelos, confira [Usar modelos vinculados com o Azure Resource Manager](linked-templates.md).
* Para iterar um número especificado de vezes ao criar um tipo de recurso, consulte [Criar várias instâncias de recursos no Azure Resource Manager](copy-resources.md).
* Para ver como implantar o modelo que você criou, consulte [Implantar um aplicativo com o Modelo do Azure Resource Manager](deploy-powershell.md).

