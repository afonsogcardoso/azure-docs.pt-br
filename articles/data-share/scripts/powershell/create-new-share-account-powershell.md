---
title: 'Script PowerShell: crie nova conta de compartilhamento de dados do Azure | Microsoft Docs'
description: Este script PowerShell cria uma nova conta de compartilhamento de dados.
services: data-share
author: joannapea
ms.service: data-share
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/07/2019
ms.author: joanpo
ms.openlocfilehash: c3852dd5f1d3d3df8a982716ce5dab9426782869
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "70307264"
---
# <a name="use-powershell-to-create-a-data-share-account-in-azure"></a>Use o PowerShell para criar uma conta de compartilhamento de dados no Azure

Este script PowerShell cria uma nova conta de compartilhamento de dados. 

## <a name="sample-script"></a>Exemplo de script

```powershell
# Set variables with your own values
$resourceGroupName = "<Resource group name"
$location = "<Location>"
$dataShareAccountName = "<Data share account name>"

# Create a new Azure Data Share account
New-AzDataShareAccount -ResourceGroupName $resourceGroupName -Name $dataShareAccountName -Location $location
```

## <a name="script-explanation"></a>Explicação sobre o script

Este script usa os seguintes comandos: 

| Comando | Observações |
|---|---|
| [New-AzDataShareConta](/powershell/module/az.datashare/new-azdatashareaccount?view=azps-2.6.0) | Cria uma conta de compartilhamento de dados. |
|||

## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre o Azure PowerShell, confira a [Documentação do Azure PowerShell](https://docs.microsoft.com/powershell/).

Amostras adicionais de script PowerShell do Azure Data Share podem ser encontradas nas [amostras do PowerShell do Compartilhamento de Dados do Azure](../../samples-powershell.md).
