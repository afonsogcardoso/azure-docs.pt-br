---
title: Tabelas de particionamento
description: Recomendações e exemplos para o uso de partições de tabela no pool Synapse SQL
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 03/18/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 368276f75128c80b8df326a26acf26c841e9f68a
ms.sourcegitcommit: bd5fee5c56f2cbe74aa8569a1a5bce12a3b3efa6
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2020
ms.locfileid: "80742681"
---
# <a name="partitioning-tables-in-synapse-sql-pool"></a>Tabelas de particionamento no pool Synapse SQL

Recomendações e exemplos para o uso de partições de tabela no pool Synapse SQL.

## <a name="what-are-table-partitions"></a>O que são as partições de tabela?

Partições de tabela permitem dividir seus dados em grupos menores de dados. Na maioria dos casos, as partições de tabela são criadas em uma coluna de data. O particionamento é suportado em todos os tipos de tabela de sinuca Synapse SQL; incluindo armazenamento de colunas agrupados, índice agrupado e pilha. O particionamento também tem suporte em todos os tipos de distribuição, incluindo hash ou round robin.  

O particionamento pode melhorar o desempenho da consulta e a manutenção de dados. Se ele beneficia ambos ou apenas um depende de como os dados são carregados e se a mesma coluna pode ser usada para ambas as finalidades, já que o particionamento só pode ser feito em uma coluna.

### <a name="benefits-to-loads"></a>Benefícios para cargas

O principal benefício do particionamento no pool Synapse SQL é melhorar a eficiência e o desempenho do carregamento de dados usando a exclusão de partição, a comutação e a fusão. Na maioria dos casos, os dados são particionados em uma coluna de data que está intimamente ligada à ordem em que os dados são carregados no banco de dados. Uma das maiores vantagens de usar partições para manter dados é evitar o registro de transações em log. Embora a simples inserção, atualização ou exclusão de dados possa ser a abordagem mais simples, com um pouco de empenho, o uso de particionamento durante o processo de carregamento pode melhorar consideravelmente o desempenho.

A alternância de partição pode ser usada para remover ou substituir uma seção de uma tabela rapidamente.  Por exemplo, uma tabela de fatos de vendas pode conter apenas dados dos últimos 36 meses. No final de cada mês, o mês de dados de vendas mais antigo é excluído da tabela.  Esses dados poderiam ser excluídos usando uma instrução delete para excluir os dados do mês mais antigo. No entanto, a exclusão de uma grande quantidade de dados linha por linha com uma declaração DELETE pode demorar muito tempo, bem como criar o risco de transações grandes, o que pode demorar muito para reverter se algo der errado. Uma abordagem ideal é remover a partição dos de dados mais antiga. A exclusão de linhas individuais pode levar horas. A exclusão de uma partição inteira pode demorar segundos.

### <a name="benefits-to-queries"></a>Vantagens para consultas

O particionamento também pode ser usado para melhorar o desempenho da consulta. Uma consulta que aplica um filtro a dados particionados pode limitar a verificação apenas para as partições qualificadas. Este método de filtragem pode evitar uma verificação de tabela completa e apenas examinar um subconjunto de dados menor. Com a introdução de índices columnstore clusterizados, os benefícios de desempenho de eliminação de predicado são menores, mas em alguns casos pode haver vantagem para as consultas. Por exemplo, se a tabela de fatos de vendas for dividida em 36 meses usando o campo de data de venda, então as consultas que filtram na data de venda podem pular a pesquisa em partições que não correspondem ao filtro.

## <a name="sizing-partitions"></a>Partições de dimensionamento

Embora o particionamento possa ser usado para melhorar o desempenho de alguns cenários, a criação de uma tabela com **muitas** partições pode prejudicar o desempenho em algumas circunstâncias.  Esses problemas são especialmente verdadeiros para tabelas columnstore clusterizadas. Para que o particionamento seja útil, é importante entender quando usar o particionamento e o número de partições a serem criadas. Não há uma regra rígida quanto à quantidade de partições que são muitas, isso depende dos seus dados e de quantas partições você carrega simultaneamente. Um esquema de particionamento bem sucedido geralmente tem dezenas a centenas de partições, não milhares.

Ao criar partições em tabelas **columnstore clusterizadas**, é importante considerar quantas linhas pertencem a cada partição. Para compactação e desempenho ideais de tabelas columnstore clusterizadas, é necessário um mínimo de um milhão de linhas por distribuição, e também é necessário haver partição. Antes de criarpartições, o pool Synapse SQL já divide cada tabela em 60 bancos de dados distribuídos. O particionamento adicionado a uma tabela é além das distribuições criadas nos bastidores. Usando este exemplo, se a tabela de fatos de vendas continha 36 partições mensais e, dado que um pool Synapse SQL tem 60 distribuições, então a tabela de fatos de vendas deve conter 60 milhões de linhas por mês, ou 2,1 bilhões de linhas quando todos os meses são preenchidos. Se uma tabela possuir menos linhas do que o mínimo recomendado, considere usar menos partições para aumentar o número de linhas por partição. Para obter mais informações, consulte o artigo [Indexação](sql-data-warehouse-tables-index.md), que inclui consultas que podem avaliar a qualidade dos índices columnstore do cluster.

## <a name="syntax-differences-from-sql-server"></a>Diferenças de sintaxe do SQL Server

O pool Synapse SQL introduz uma maneira de definir partições mais simples do que o SQL Server. Funções e esquemas de particionamento não são usados no pool Synapse SQL como estão no SQL Server. Em vez disso,tudo o que você precisa fazer é identificar a coluna particionada e os pontos delimitadores. Embora a sintaxe de particionamento possa ser ligeiramente diferente do SQL Server, os conceitos básicos são os mesmos. O pool SQL Server e synapse SQL suportam uma coluna de partição por tabela, que pode ser partição variada. Para saber mais sobre particionamento, consulte [Tabelas e índices particionados](/sql/relational-databases/partitions/partitioned-tables-and-indexes?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

O exemplo a seguir usa a instrução [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para particionar a tabela FactInternetSales na coluna OrderDateKey:

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
    [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

## <a name="migrating-partitioning-from-sql-server"></a>Migrando o particionamento do SQL Server

Para migrar definições de partição do SQL Server para o pool Synapse SQL simplesmente:

- Elimine o [esquema de partição](/sql/t-sql/statements/create-partition-scheme-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) do SQL Server.
- Adicione a definição [função de partição](/sql/t-sql/statements/create-partition-function-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para CREATE TABLE.

Se você estiver migrando uma tabela particionada de uma Instância do SQL Server, o SQL a seguir poderá ajudá-lo a descobrir o número de linhas que em cada partição. Tenha em mente que se a mesma granularidade de particionamento for usada no pool Synapse SQL, o número de linhas por partição diminui em um fator de 60.  

```sql
-- Partition information for a SQL Server Database
SELECT      s.[name]                        AS      [schema_name]
,           t.[name]                        AS      [table_name]
,           i.[name]                        AS      [index_name]
,           p.[partition_number]            AS      [partition_number]
,           SUM(a.[used_pages]*8.0)         AS      [partition_size_kb]
,           SUM(a.[used_pages]*8.0)/1024    AS      [partition_size_mb]
,           SUM(a.[used_pages]*8.0)/1048576 AS      [partition_size_gb]
,           p.[rows]                        AS      [partition_row_count]
,           rv.[value]                      AS      [partition_boundary_value]
,           p.[data_compression_desc]       AS      [partition_compression_desc]
FROM        sys.schemas s
JOIN        sys.tables t                    ON      t.[schema_id]         = s.[schema_id]
JOIN        sys.partitions p                ON      p.[object_id]         = t.[object_id]
JOIN        sys.allocation_units a          ON      a.[container_id]      = p.[partition_id]
JOIN        sys.indexes i                   ON      i.[object_id]         = p.[object_id]
                                            AND     i.[index_id]          = p.[index_id]
JOIN        sys.data_spaces ds              ON      ds.[data_space_id]    = i.[data_space_id]
LEFT JOIN   sys.partition_schemes ps        ON      ps.[data_space_id]    = ds.[data_space_id]
LEFT JOIN   sys.partition_functions pf      ON      pf.[function_id]      = ps.[function_id]
LEFT JOIN   sys.partition_range_values rv   ON      rv.[function_id]      = pf.[function_id]
                                            AND     rv.[boundary_id]      = p.[partition_number]
WHERE       p.[index_id] <=1
GROUP BY    s.[name]
,           t.[name]
,           i.[name]
,           p.[partition_number]
,           p.[rows]
,           rv.[value]
,           p.[data_compression_desc]
;
```

## <a name="partition-switching"></a>Alternância de partição

O pool Synapse SQL suporta divisão de partição, fusão e comutação. Todas essas funções são executadas usando a instrução [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Para alternar as partições entre duas tabelas, você deve garantir que as partições alinhem em seus respectivos limites e que correspondam as definições de tabela. Como restrições de verificação não estão disponíveis para impor o intervalo de valores em uma tabela, a tabela de origem deve conter os mesmos limites de partição da tabela de destino. Se os limites de partição não forem os mesmos, a alternância de partição falhará, pois os metadados da partição não serão sincronizados.

### <a name="how-to-split-a-partition-that-contains-data"></a>Como dividir uma partição que contém dados

O método mais eficiente para dividir uma partição que já contém dados é usar um `CTAS` instrução. Se a tabela de partição for um columnstore em cluster, então a partição da tabela deverá estar vazia antes que possa ser dividida.

O exemplo a seguir cria uma tabela do columnstore particionada. Ele insere uma linha em cada partição:

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101
                    )
                )
)
;

INSERT INTO dbo.FactInternetSales
VALUES (1,19990101,1,1,1,1,1,1);
INSERT INTO dbo.FactInternetSales
VALUES (1,20000101,1,1,1,1,1,1);
```

A consulta a seguir localiza a contagem de linhas utilizando a exibição do catálogo `sys.partitions`:

```sql
SELECT  QUOTENAME(s.[name])+'.'+QUOTENAME(t.[name]) as Table_name
,       i.[name] as Index_name
,       p.partition_number as Partition_nmbr
,       p.[rows] as Row_count
,       p.[data_compression_desc] as Data_Compression_desc
FROM    sys.partitions p
JOIN    sys.tables     t    ON    p.[object_id]   = t.[object_id]
JOIN    sys.schemas    s    ON    t.[schema_id]   = s.[schema_id]
JOIN    sys.indexes    i    ON    p.[object_id]   = i.[object_Id]
                            AND   p.[index_Id]    = i.[index_Id]
WHERE t.[name] = 'FactInternetSales'
;
```

O comando Dividir a seguir recebe uma mensagem de erro:

```sql
ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Msg 35346, Nível 15, Estado 1, linha 44 da cláusula SPLIT da instrução ALTER PARTITION falhou porque a partição não está vazia. somente partições vazias podem ser divididas quando existe um índice columnstore na tabela. Considere desabilitar o índice columnstore antes de emitir a instrução ALTER PARTITION, e então recriar o índice columnstore após a instrução ALTER PARTITION estar concluída.

No entanto, é possível utilizar `CTAS` para criar uma nova tabela para reter os dados.

```sql
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    FactInternetSales
WHERE   1=2
;
```

Como os limites de partição estão alinhados, uma alternância é permitida. Isso deixará a tabela de origem com uma partição vazia que você poderá dividir posteriormente.

```sql
ALTER TABLE FactInternetSales SWITCH PARTITION 2 TO  FactInternetSales_20000101 PARTITION 2;

ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Tudo o que resta é alinhar os dados aos novos limites de partição utilizando `CTAS` e, em seguida, alternar os dados de volta para a tabela principal.

```sql
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales_20000101]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

ALTER TABLE dbo.FactInternetSales_20000101_20010101 SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2;
```

Após concluir a movimentação dos dados, é recomendável atualizar as estatísticas na tabela de destino. A atualização de estatísticas garante que as estatísticas reflitam com precisão a nova distribuição dos dados em suas respectivas partições.

```sql
UPDATE STATISTICS [dbo].[FactInternetSales];
```

### <a name="load-new-data-into-partitions-that-contain-data-in-one-step"></a>Carregue novos dados em partições que contenham dados em uma etapa

Carregar dados em partições com comutação de partições é uma maneira conveniente de encenar novos dados em uma tabela que não é visível para os usuários do switch nos novos dados.  Pode ser desafiador em sistemas ocupados para lidar com a contenção de bloqueio associada à comutação de partição.  Para limpar os dados existentes em `ALTER TABLE` uma partição, costumava ser necessário alternar os dados.  Em `ALTER TABLE` seguida, outro foi necessário para mudar os novos dados.  No pool Synapse SQL, a `TRUNCATE_TARGET` opção `ALTER TABLE` é suportada no comando.  Com `TRUNCATE_TARGET` `ALTER TABLE` o comando substitui os dados existentes na partição com novos dados.  Abaixo está um `CTAS` exemplo que usa para criar uma nova tabela com os dados existentes, insere novos dados e, em seguida, alterna todos os dados de volta para a tabela de destino, substituindo os dados existentes.

```sql
CREATE TABLE [dbo].[FactInternetSales_NewSales]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

INSERT INTO dbo.FactInternetSales_NewSales
VALUES (1,20000101,2,2,2,2,2,2);

ALTER TABLE dbo.FactInternetSales_NewSales SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2 WITH (TRUNCATE_TARGET = ON);  
```

### <a name="table-partitioning-source-control"></a>Controle da origem do particionamento da tabela

Para evitar a definição da tabela de **rusting** em seu sistema de controle de origem, convém considerar a abordagem a seguir:

1. Criar a tabela como uma tabela particionada, mas sem valores de partição

    ```sql
    CREATE TABLE [dbo].[FactInternetSales]
    (
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
    )
    WITH
    (   CLUSTERED COLUMNSTORE INDEX
    ,   DISTRIBUTION = HASH([ProductKey])
    ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES () )
    )
    ;
    ```

1. `SPLIT` a tabela como parte do processo de implantação:

    ```sql
     -- Create a table containing the partition boundaries

    CREATE TABLE #partitions
    WITH
    (
        LOCATION = USER_DB
    ,   DISTRIBUTION = HASH(ptn_no)
    )
    AS
    SELECT  ptn_no
    ,       ROW_NUMBER() OVER (ORDER BY (ptn_no)) as seq_no
    FROM    (
        SELECT CAST(20000101 AS INT) ptn_no
        UNION ALL
        SELECT CAST(20010101 AS INT)
        UNION ALL
        SELECT CAST(20020101 AS INT)
        UNION ALL
        SELECT CAST(20030101 AS INT)
        UNION ALL
        SELECT CAST(20040101 AS INT)
    ) a
    ;

     -- Iterate over the partition boundaries and split the table

    DECLARE @c INT = (SELECT COUNT(*) FROM #partitions)
    ,       @i INT = 1                                 --iterator for while loop
    ,       @q NVARCHAR(4000)                          --query
    ,       @p NVARCHAR(20)     = N''                  --partition_number
    ,       @s NVARCHAR(128)    = N'dbo'               --schema
    ,       @t NVARCHAR(128)    = N'FactInternetSales' --table
    ;

    WHILE @i <= @c
    BEGIN
        SET @p = (SELECT ptn_no FROM #partitions WHERE seq_no = @i);
        SET @q = (SELECT N'ALTER TABLE '+@s+N'.'+@t+N' SPLIT RANGE ('+@p+N');');

        -- PRINT @q;
        EXECUTE sp_executesql @q;
        SET @i+=1;
    END

     -- Code clean-up

    DROP TABLE #partitions;
    ```

Com esta abordagem, o código no controle de origem permanece estático e os valores de limite de particionamento são permitidos a ser dinâmicos; evoluindo com o banco de dados ao longo do tempo.

## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre como desenvolver tabelas, consulte [Visão geral da tabela](sql-data-warehouse-tables-overview.md).
