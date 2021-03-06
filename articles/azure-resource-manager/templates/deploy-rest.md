---
title: Implantar recursos com API rest e modelo
description: Use o Azure Resource Manager e o API REST do Gerenciador de Recursos para implantar recursos no Azure. Os recursos são definidos em um modelo do Resource Manager.
ms.topic: conceptual
ms.date: 06/04/2019
ms.openlocfilehash: 9cdb7b668e5170917b41ef49639bd9a17e538766
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80153226"
---
# <a name="deploy-resources-with-arm-templates-and-resource-manager-rest-api"></a>Implantar recursos com modelos ARM e API REST do Gerenciador de Recursos

Este artigo explica como usar a API REST do Gerenciador de Recursos com modelos ARM (Azure Resource Manager) para implantar seus recursos no Azure.

Você pode incluir seu modelo no corpo da solicitação ou vinculá-lo a um arquivo. Se optar pelo arquivo, ele poderá ser local ou um arquivo externo disponível por meio de um URI. Quando seu modelo estiver em uma conta de armazenamento, você poderá restringir o acesso a ele e fornecer um token de SAS (Assinatura de Acesso Compartilhado) durante a implantação.

## <a name="deployment-scope"></a>Escopo de implantação

Você pode direcionar sua implantação para um grupo de gerenciamento, uma assinatura do Azure ou um grupo de recursos. Na maioria dos casos, você direcionará implantações para um grupo de recursos. Use o grupo de gerenciamento ou implantações de assinatura para aplicar políticas e atribuições de função em todo o escopo especificado. Você também usa implantações de assinatura para criar um grupo de recursos e implantar recursos nele. Dependendo do escopo da implantação, você usa diferentes comandos.

Para implantar em um **grupo de recursos,** use [Implantações - Criar](/rest/api/resources/deployments/createorupdate). A solicitação é enviada para:

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-05-01
```

Para implantar em uma **assinatura,** use [Implantações - Crie no escopo de assinatura.](/rest/api/resources/deployments/createorupdateatsubscriptionscope) A solicitação é enviada para:

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-05-01
```

Para obter mais informações sobre implantações no nível de assinatura, consulte [Criar grupos de recursos e recursos no nível de assinatura](deploy-to-subscription.md).

Para implantar em um **grupo de gerenciamento,** use [Implantações - Crie no escopo do grupo de gerenciamento](/rest/api/resources/deployments/createorupdateatmanagementgroupscope). A solicitação é enviada para:

```HTTP
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{groupId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-05-01
```

Para obter mais informações sobre implantações de nível de grupo de gerenciamento, consulte [Criar recursos no nível do grupo de gerenciamento](deploy-to-management-group.md).

Os exemplos deste artigo usam implantações de grupo de recursos.

## <a name="deploy-with-the-rest-api"></a>Implantar com a API REST

1. Definir [Parâmetros e cabeçalhos comuns](/rest/api/azure/), incluindo tokens de autenticação.

1. Se você não tiver um grupo de recursos existente, crie um grupo de recursos. Forneça sua ID de assinatura, o nome do novo grupo de recursos e local que você precisa para sua solução. Para obter mais informações, consulte [Criar um grupo de recursos](/rest/api/resources/resourcegroups/createorupdate).

   ```HTTP
   PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2019-05-01
   ```

   Com um corpo de solicitação como:

   ```json
   {
    "location": "West US",
    "tags": {
      "tagname1": "tagvalue1"
    }
   }
   ```

1. Valide sua implantação antes de executá-la usando a operação [Validar uma implantação do modelo](/rest/api/resources/deployments/validate) . Ao testar a implantação, forneça parâmetros exatamente como faria ao executar a implantação (mostrado na próxima etapa).

1. Para implantar um modelo, forneça seu ID de assinatura, o nome do grupo de recursos, o nome da implantação no URI de solicitação.

   ```HTTP
   PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2019-05-01
   ```

   No órgão de solicitação, forneça um link para o seu modelo e arquivo de parâmetros. Para obter mais informações sobre o arquivo parâmetro, consulte [Criar arquivo de parâmetros do Gerenciador de recursos](parameter-files.md).

   Observe que **mode** está definido como **Incremental**. Para executar uma implantação completa, defina **mode** como **Complete**. Tenha cuidado ao usar o modo completo, pois você pode excluir acidentalmente recursos que não estão em seu modelo.

   ```json
   {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental"
    }
   }
   ```

    Se você quiser registrar o conteúdo da resposta, o conteúdo da solicitação ou ambos, inclua **debugSetting** na solicitação.

   ```json
   {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental",
      "debugSetting": {
        "detailLevel": "requestContent, responseContent"
      }
    }
   }
   ```

    Você pode configurar sua conta de armazenamento para usar um token de SAS (Assinatura de Acesso Compartilhado). Para obter mais informações, consulte [Delegando Acesso com uma Assinatura de Acesso Compartilhado](/rest/api/storageservices/delegating-access-with-a-shared-access-signature).

    Se você precisar fornecer um valor confidencial para um parâmetro (como uma senha), adicione esse valor em um cofre de chaves. Recupere o cofre de chaves durante a implantação, conforme mostrado no exemplo anterior. Para obter mais informações, consulte [Passar valores seguros durante a implantação](key-vault-parameter.md).

1. Em vez de vincular os modelo e parâmetros a um arquivo, você pode incluí-los no corpo da solicitação. O exemplo a seguir mostra o corpo de solicitação com o modelo e o parâmetro inline:

   ```json
   {
      "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "storageAccountType": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "allowedValues": [
              "Standard_LRS",
              "Standard_GRS",
              "Standard_ZRS",
              "Premium_LRS"
            ],
            "metadata": {
              "description": "Storage Account type"
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          }
        },
        "variables": {
          "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
        },
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2018-02-01",
            "name": "[variables('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
              "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "properties": {}
          }
        ],
        "outputs": {
          "storageAccountName": {
            "type": "string",
            "value": "[variables('storageAccountName')]"
          }
        }
      },
      "parameters": {
        "location": {
          "value": "eastus2"
        }
      }
    }
   }
   ```

1. Para obter o status da implantação do modelo, use [Implantações - Obter](/rest/api/resources/deployments/get).

   ```HTTP
   GET https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2018-05-01
   ```

## <a name="next-steps"></a>Próximas etapas

- Para reverter para uma implantação bem-sucedida quando tiver um erro, consulte [Reversão de erro para implantação bem-sucedida](rollback-on-error.md).
- Para especificar como lidar com os recursos existentes no grupo de recursos, mas que não estão definidos no modelo, confira [Modos de implantação do Azure Resource Manager](deployment-modes.md).
- Para saber mais sobre como lidar com operações assíncronas de REST, confira [Track asynchronous Azure operations](../management/async-operations.md) (Rastrear operações assíncronas do Azure).
- Para saber mais sobre modelos, consulte [Entenda a estrutura e a sintaxe dos modelos ARM](template-syntax.md).

