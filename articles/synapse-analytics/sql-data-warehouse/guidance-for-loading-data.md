---
title: Práticas recomendadas de carregamento de dados para pool Synapse SQL
description: Recomendações e otimizações de desempenho para o carregamento de dados usando o pool Synapse SQL.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 02/04/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: e170a789727fb0de36705895245cc638d30ee3d7
ms.sourcegitcommit: bd5fee5c56f2cbe74aa8569a1a5bce12a3b3efa6
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2020
ms.locfileid: "80745502"
---
# <a name="best-practices-for-loading-data-using-synapse-sql-pool"></a>Práticas recomendadas para carregar dados usando o pool Synapse SQL

Neste artigo, você aprenderá recomendações e otimizações de desempenho para carregar dados usando o pool SQL.

## <a name="preparing-data-in-azure-storage"></a>Preparando dados no Armazenamento do Azure

Para minimizar a latência, colocalize sua camada de armazenamento e seu pool SQL.

Ao exportar dados em um formato de arquivo ORC, você poderá receber erros de falta de memória Java quando houver colunas de texto grandes. Para contornar essa limitação, exporte apenas um subconjunto das colunas.

O PolyBase não pode carregar linhas com mais de 1.000.000 bytes de dados. Ao colocar dados nos arquivos de texto no armazenamento de blob do Azure ou do Azure Data Lake Store, eles devem ter menos de 1.000.000 de bytes de dados. Essa limitação de bytes é verdadeira, independentemente do esquema de tabela.

Todos os formatos de arquivo têm características diferentes de desempenho. Para o carregamento mais rápido, use arquivos de texto delimitados compactados. A diferença entre o desempenho de UTF-16 e UTF-8 é mínima.

Dividir arquivos compactados grandes em arquivos compactados menores.

## <a name="running-loads-with-enough-compute"></a>Executando carregamentos com computação suficiente

Para uma velocidade mais alta de carregamento, execute apenas uma carga de trabalho por vez. Se isso não for viável, execute um número mínimo de cargas simultaneamente. Se você espera um grande trabalho de carregamento, considere aumentar seu pool SQL antes da carga.

Para executar cargas com recursos de computação apropriados, crie usuários de carregamento designados para executar cargas. Atribuir cada usuário de carregamento a uma classe de recursos específica ou grupo de carga de trabalho. Para executar uma carga, faça login como um dos usuários de carregamento e, em seguida, execute a carga. A carga é executada com a classe de recurso do usuário.  

> [!NOTE]
> Esse método é mais simples do que tentar alterar a classe de recurso do usuário para se ajustar à necessidade de classe de recurso atual.

### <a name="example-of-creating-a-loading-user"></a>Exemplo de como criar um usuário de carregamento

Este exemplo cria um usuário de carregamento para a classe de recurso staticrc20. A primeira etapa é **conectar-se ao mestre** e criar um logon.

```sql
   -- Connect to master
   CREATE LOGIN LoaderRC20 WITH PASSWORD = 'a123STRONGpassword!';
```

Conecte-se ao pool SQL e crie um usuário. O código a seguir pressupõe que você está conectado ao banco de dados chamado mySampleDataWarehouse. Ele mostra como criar um usuário chamado LoaderRC20 e dá ao usuário permissão de controle em um banco de dados. Em seguida, adiciona o usuário como membro da função de banco de dados staticrc20.  

```sql
   -- Connect to the database
   CREATE USER LoaderRC20 FOR LOGIN LoaderRC20;
   GRANT CONTROL ON DATABASE::[mySampleDataWarehouse] to LoaderRC20;
   EXEC sp_addrolemember 'staticrc20', 'LoaderRC20';
```

Para executar uma carga com recursos para as classes de recursos staticRC20, faça login como LoaderRC20 e execute a carga.

Execute cargas sob classes de recursos estáticas em vez de dinâmicas. Usar as classes de recursos estáticas garante os mesmos recursos, independentemente de suas [unidades de data warehouse](what-is-a-data-warehouse-unit-dwu-cdwu.md). Se você usar uma classe de recursos dinâmicos, os recursos variam de acordo com seu nível de serviço.

Para classes dinâmicas, um nível inferior do serviço significa que você provavelmente precisa usar uma classe de recursos maior para seu usuário de carregamento.

## <a name="allowing-multiple-users-to-load"></a>Permitindo que vários usuários façam o carregamento

Muitas vezes há a necessidade de ter vários usuários de carregar dados em um pool SQL. Carregar com [CREATE TABLE AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) requer permissões CONTROL do banco de dados.  A permissão CONTROL dá acesso de controle para todos os esquemas.

Talvez você não queira que todos os usuários de carregamento tenham acesso de controle em todos os esquemas. Para limitar as permissões, use a instrução DENY CONTROL.

Por exemplo: considere os esquemas de banco de dados, schema_A para o departamento A e schema_B para o departamento B. Os usuários de banco de dados user_A e user_B serão usuários de carregamento de PolyBase nos departamentos A e B respectivamente. Ambos receberam permissões de banco de dados CONTROL. Os criadores dos esquemas A e B agora bloquearam seus esquemas usando DENY:

```sql
   DENY CONTROL ON SCHEMA :: schema_A TO user_B;
   DENY CONTROL ON SCHEMA :: schema_B TO user_A;
```

User_A e user_B estão agora bloqueados do esquema do outro departamento.

## <a name="loading-to-a-staging-table"></a>Carregando uma tabela de preparo

Para obter a velocidade de carregamento mais rápida para mover dados em uma tabela de pool SQL, carregue os dados em uma tabela de preparação.  Defina a tabela de preparo como um heap e use o round-robin para a opção de distribuição.

Considere que o carregamento é geralmente um processo de duas etapas no qual você primeiro carrega para uma tabela de estadiamento e, em seguida, insere os dados em uma tabela de pool SQL de produção. Caso a tabela de produção use uma distribuição hash, o tempo total para carregar e inserir pode ser mais rápido se você definir a tabela de preparo com a distribuição de hash.

Carregar para a tabela de preparo demora mais, mas a segunda etapa de inserção das linhas na tabela de produção não incorrerá em movimentação de dados entre as distribuições.

## <a name="loading-to-a-columnstore-index"></a>Carregando para um índice columnstore

Os índices columnstore exigem grandes quantidades de memória para compactar dados em rowgroups de alta qualidade. Para obter a melhor compactação e eficiência de índice, o índice columnstore precisa compactar um máximo de 1.048.576 linhas em cada rowgroup.

Quando há pressão de memória, o índice columnstore pode não ser capaz de alcançar as taxas máximas de compactação. Este cenário, por sua vez, afeta o desempenho da consulta. Para uma análise profunda, consulte [Otimizações de memória columnstore](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).

- Para garantir que o usuário de carregamento tenha memória suficiente para alcançar as taxas máximas de compactação, use usuários de carregamento que sejam membros de uma classe de recursos média ou grande.
- Carregue linhas suficientes para preencher completamente novos rowgroups. Durante um carregamento em massa, a cada 1.048.576 linhas são compactadas diretamente para o columnstore como um rowgroup completo. Carregamentos com menos de 102.400 linhas enviam as linhas para o deltastore, onde as linhas são mantidas em um índice de árvore b.

> [!NOTE]
> Se você carregar poucas linhas, todas elas podem ir para o deltastore e não ser compactadas imediatamente no formato de columnstore.

## <a name="increase-batch-size-when-using-sqlbulkcopy-api-or-bcp"></a>Aumente o tamanho do lote ao usar a API ou bcp sqLBulkCopy

O carregamento com o PolyBase fornecerá o maior throughput com o pool SQL. Se você não puder usar o PolyBase para carregar e usar a API ou [bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) [SqLBulkCopy,](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) você deve considerar aumentar o tamanho do lote para uma melhor produção.

> [!TIP]
> Um tamanho de lote entre linhas de 100 K a 1M é a linha de base recomendada para determinar a capacidade ideal de tamanho do lote.

## <a name="handling-loading-failures"></a>Tratamento de falhas de carregamento

Um carregamento usando uma tabela externa pode falhar com o erro *“Consulta anulada – o limite de rejeição máximo foi atingido durante a leitura de uma fonte externa”*. Essa mensagem indica que os dados externos contêm registros sujos.

Um registro de dados é considerado sujo se atender a uma das seguintes condições:

- Os tipos de dados e o número de colunas não correspondem às definições da coluna da tabela externa.
- Os dados não estão em conformidade com o formato de arquivo externo especificado.

Para corrigir os registros sujos, verifique se a tabela externa e as definições de formato de arquivo externo estão corretas e se os dados externos são compatíveis com essas definições.

Se um subconjunto de registros de dados externos estiver sujo, você pode optar por rejeitar esses registros para suas consultas usando as opções de rejeição em [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="inserting-data-into-a-production-table"></a>Inserindo dados em uma tabela de produção

Uma única carga para uma pequena tabela com uma [instrução INSERT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), ou mesmo uma recarga periódica de uma pesquisa pode ser boa o suficiente com uma instrução como `INSERT INTO MyLookup VALUES (1, 'Type 1')`.  Porém, inserções únicas de toneladas não são tão eficientes como executar um carregamento em massa.

Se você tiver milhares ou mais de inserções únicas ao longo do dia, crie lotes para as inserções para que possa carregá-las em massa.  Desenvolva seus processos para acrescentar as inserções únicas para um arquivo e depois crie outro processo que periodicamente carrega o arquivo.

## <a name="creating-statistics-after-the-load"></a>Criando estatísticas após o carregamento

Para melhorar o desempenho de suas consultas, é importante que as estatísticas sejam criadas em todas as colunas de todas as tabelas após o primeiro carregamento ou ocorrem alterações significativas nos dados. A criação de estatísticas pode ser feita manualmente ou você pode habilitar [AUTO_CREATE_STATISTICS](sql-data-warehouse-tables-statistics.md#automatic-creation-of-statistic).

Para obter uma explicação detalhada das estatísticas, confira [Estatísticas](sql-data-warehouse-tables-statistics.md). O exemplo a seguir mostra como criar manualmente estatísticas em cinco colunas da tabela Customer_Speed.

```sql
create statistics [SensorKey] on [Customer_Speed] ([SensorKey]);
create statistics [CustomerKey] on [Customer_Speed] ([CustomerKey]);
create statistics [GeographyKey] on [Customer_Speed] ([GeographyKey]);
create statistics [Speed] on [Customer_Speed] ([Speed]);
create statistics [YearMeasured] on [Customer_Speed] ([YearMeasured]);
```

## <a name="rotate-storage-keys"></a>Girar as chaves de armazenamento

É uma boa prática de segurança alterar a chave de acesso para seu armazenamento de blob regularmente. Você tem duas chaves de armazenamento para sua conta de armazenamento de blob, o que permite a transição das chaves.

Para girar chaves de conta de armazenamento do Azure:

Para cada conta de armazenamento cuja chave foi alterada, execute [ALTER DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/alter-database-scoped-credential-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Exemplo:

A chave original é criada

```sql
CREATE DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key1'
```

Altere a chave 1 para a chave 2

```sql
ALTER DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key2'
```

Não é necessária nenhuma outra alteração nas fontes de dados externas subjacentes.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre o PolyBase e a criação de um processo de Extrair, carregar e transformar (ELT), consulte [Projetar ELT para o SQL Data Warehouse](design-elt-data-loading.md).
- Para ver um tutorial de carregamento, [Usar o PolyBase para carregar dados do armazenamento de blobs do Azure para o SQL Data Warehouse do Azure](load-data-from-azure-blob-storage-using-polybase.md).
- Para monitorar os carregamentos de dados, consulte [Monitorar sua carga de trabalho usando DMVs](sql-data-warehouse-manage-monitor.md).
