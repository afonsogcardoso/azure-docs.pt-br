---
title: Funções do sistema no idioma de consulta azure Cosmos DB
description: Saiba mais sobre funções do sistema SQL incorporada e definida pelo usuário no Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 6f41adbb726313ef095084d079dc7852736e0c06
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74870523"
---
# <a name="system-functions-azure-cosmos-db"></a>Funções do sistema (Azure Cosmos DB)

 O Cosmos DB fornece muitas funções SQL internas. As categorias de funções internas estão listadas abaixo.  
  
|Grupo de funções|Descrição|Operações|  
|--------------|-----------------|-----------------| 
|[Funções de matriz](sql-query-array-functions.md)|As funções de matriz executam uma operação em um valor de matriz de entrada e retornam um valor numérico, booliano ou um valor de matriz. | [ARRAY_CONCAT](sql-query-array-concat.md) [ARRAY_CONTAINS](sql-query-array-contains.md) [ARRAY_SLICE](sql-query-array-slice.md) [ARRAY_LENGTH](sql-query-array-length.md) |
|[Funções de data e hora](sql-query-date-time-functions.md)|As funções de data e hora permitem obter a data e a hora atuals do UTC em duas formas; um carimbo de tempo numérico cujo valor é a época unix em milissegundos ou como uma string que está em conformidade com o formato ISO 8601. | [GetCurrentDateTime](sql-query-getcurrentdatetime.md), [GetCurrentTimestamp](sql-query-getcurrenttimestamp.md) |
|[Funções matemáticas](sql-query-mathematical-functions.md)|As funções matemáticas executam um cálculo, normalmente com base em valores de entrada que são fornecidos como argumentos, e retornam um valor numérico. | [ABS,](sql-query-abs.md) [ACOS,](sql-query-acos.md) [ASIN,](sql-query-asin.md) [ATAN,](sql-query-atan.md) [ATN2,](sql-query-atn2.md) [TETO,](sql-query-ceiling.md) [COS,](sql-query-cos.md) [COT,](sql-query-cot.md) [GRAUS,](sql-query-degrees.md) [EXP,](sql-query-exp.md) [PISO,](sql-query-floor.md) [LOG,](sql-query-log.md) [LOG10,](sql-query-log10.md) [PI,](sql-query-pi.md) [ENERGIA,](sql-query-power.md) [RADIANS,](sql-query-radians.md) [RAND,](sql-query-rand.md) [ROUND,](sql-query-round.md) [SIGN,](sql-query-sign.md) [SIN,](sql-query-sin.md) [SQRT,](sql-query-sqrt.md) [QUADRADO,](sql-query-square.md) [TAN](sql-query-tan.md), [TRUNC](sql-query-trunc.md) |
|[Funções espaciais](sql-query-spatial-functions.md)|As funções espaciais executam uma operação em um valor de entrada de objeto espacial e retornam um valor numérico ou booliano. | [ST_DISTANCE](sql-query-st-distance.md) [ST_INTERSECTS](sql-query-st-intersects.md)ST_INTERSECTS , [ST_ISVALID](sql-query-st-isvalid.md), [ST_ISVALIDDETAILED](sql-query-st-isvaliddetailed.md) [ST_WITHIN](sql-query-st-within.md) |
|[Funções de cadeia de caracteres](sql-query-string-functions.md)|As funções de cadeia de caracteres executam uma operação em um valor de cadeia de caracteres de entrada e retornam uma cadeia de caracteres, um valor numérico ou um valor booliano. | [CONCAT](sql-query-concat.md), [CONTÉM](sql-query-contains.md), [ENDSWITH](sql-query-endswith.md), [INDEX_OF](sql-query-index-of.md), [ESQUERDA](sql-query-left.md), [COMPRIMENTO](sql-query-length.md), [INFERIOR](sql-query-lower.md), [LTRIM](sql-query-ltrim.md), [SUBSTITUIR](sql-query-replace.md), [REPLICAR](sql-query-replicate.md), [INVERTER](sql-query-reverse.md), [DIREITA](sql-query-right.md), [RTRIM](sql-query-rtrim.md), [STARTSWITH](sql-query-startswith.md), [StringToArray](sql-query-stringtoarray.md), [StringToBOOLEAN,](sql-query-stringtoboolean.md) [StringTonull](sql-query-stringtonull.md), [StringToNumber](sql-query-stringtonumber.md), [StringToObject](sql-query-stringtoobject.md), [SUBSTRING](sql-query-substring.md), [Tostring](sql-query-tostring.md), [TRIM](sql-query-trim.md), [UPPER](sql-query-upper.md) |
|[Funções de verificação de tipo](sql-query-type-checking-functions.md)|As funções de verificação de tipo permitem que você verifique o tipo de uma expressão em consultas SQL. | [IS_ARRAY](sql-query-is-array.md) [, IS_BOOL,](sql-query-is-bool.md) [IS_DEFINED,](sql-query-is-defined.md) [IS_NULL](sql-query-is-null.md) [IS_NUMBER](sql-query-is-number.md), [IS_OBJECT](sql-query-is-object.md), [IS_PRIMITIVE](sql-query-is-primitive.md), [IS_STRING](sql-query-is-string.md) |

## <a name="built-in-versus-user-defined-functions-udfs"></a>Funções incorporadas versus UDFs (UDFs) incorporadas

Se você estiver usando uma função definida pelo usuário (UDF) para a qual uma função incorporada está agora disponível, a função incorporada correspondente será mais rápida de ser executada e mais eficiente.

## <a name="built-in-versus-ansi-sql-functions"></a>Funções incorporadas versus SQL ANSI

A principal diferença entre as funções cosmos DB e as funções ANSI SQL é que as funções do Cosmos DB são projetadas para funcionar bem com dados sem esquema e esquemas mistos. Por exemplo, se uma propriedade estiver faltando ou `unknown`tiver um valor não numérico como, o item é ignorado em vez de retornar um erro.

## <a name="next-steps"></a>Próximas etapas

- [Introdução ao Azure Cosmos DB](introduction.md)
- [Funções de matriz](sql-query-array-functions.md)
- [Funções de data e hora](sql-query-date-time-functions.md)
- [Funções matemáticas](sql-query-mathematical-functions.md)
- [Funções espaciais](sql-query-spatial-functions.md)
- [Funções de cadeia de caracteres](sql-query-string-functions.md)
- [Funções de verificação de tipo](sql-query-type-checking-functions.md)
- [Funções definidas pelo usuário](sql-query-udfs.md)
- [Agregações](sql-query-aggregates.md)
