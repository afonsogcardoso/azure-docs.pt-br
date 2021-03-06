---
title: POWER in Azure Cosmos DB consulta idioma
description: Saiba mais sobre a função do sistema SQL POWER no Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 683c53c369f136ad4b917b93e9a92a71072d05e0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "71349640"
---
# <a name="power-azure-cosmos-db"></a>PODER (Azure Cosmos DB)
 Retorna o valor da expressão especificada para a potência indicada.  
  
## <a name="syntax"></a>Sintaxe
  
```sql
POWER (<numeric_expr1>, <numeric_expr2>)  
```  
  
## <a name="arguments"></a>Argumentos
  
*numeric_expr1*  
   É uma expressão numérica.  
  
*numeric_expr2*  
   É o poder para criar *numeric_expr1.*  
  
## <a name="return-types"></a>Tipos de retorno
  
  Retorna uma expressão numérica.  
  
## <a name="examples"></a>Exemplos
  
  O exemplo a seguir demonstra a elevação de um número à potência 3 (o cubo do número).  
  
```sql
SELECT POWER(2, 3) AS pow1, POWER(2.5, 3) AS pow2  
```  
  
 Este é o conjunto de resultados.  
  
```json
[{pow1: 8, pow2: 15.625}]  
```  

## <a name="next-steps"></a>Próximas etapas

- [Funções matemáticas Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Funcionamento do sistema Azure Cosmos DB](sql-query-system-functions.md)
- [Introdução ao Azure Cosmos DB](introduction.md)
