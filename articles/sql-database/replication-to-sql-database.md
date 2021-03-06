---
title: Replicação
description: Saiba como usar a replicação do SQL Server com bancos de dados únicos do banco de Dados SQL do Azure e bancos de dados em pools elásticos
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: seo-lt-2019
ms.devlang: ''
ms.topic: conceptual
author: allenwux
ms.author: xiwu
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: f718bc17b987926f4324635f096d5983acdb63fc
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79256465"
---
# <a name="replication-to-sql-database-single-and-pooled-databases"></a>Replicação para bancos de dados individuais e em pool do Banco de Dados SQL

A replicação do SQL Server pode ser configurada para bancos de dados únicos e em pool em um [servidor do Banco de Dados SQL](sql-database-servers.md) no Banco de Dados SQL do Azure.  

## <a name="supported-configurations"></a>**Configurações com suporte:**
  
- O SQL Server pode ser uma instância do SQL Server em execução local ou uma instância do SQL Server em execução em uma máquina virtual do Azure na nuvem. Para obter mais informações, veja [SQL Server na visão geral de Máquinas Virtuais do Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-sql-server-infrastructure-services/).  
- O Banco de Dados SQL do Azure deve ser um assinante de push de um distribuidor do SQL Server.  
- O banco de dados de distribuição e os agentes de replicação não podem ser colocados em um Banco de Dados SQL do Azure.  
- Há suporte para instantâneo e replicação transacional unidirecional. Não há suporte para replicação transacional ponto a ponto nem e replicação de mesclagem.
- A replicação está disponível para versão prévia pública na Instância Gerenciada do Banco de Dados SQL do Azure. Uma Instância Gerenciada pode hospedar bancos de dados de editores, distribuidores e assinantes. Para obter mais informações, veja [Replicação com a Instância Gerenciada do Banco de Dados SQL](replication-with-sql-database-managed-instance.md).

## <a name="versions"></a>Versões  

Os editores e distribuidores do SQL Server no local devem usar (pelo menos) uma das seguintes versões:  

- SQL Server 2016 e maior
- SQL Server 2014 [RTM CU10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) ou [SP1 CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [SP2 CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) ou [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

> [!NOTE]
> A tentativa de configurar a replicação usando uma versão não suportada pode resultar em número de erro \<MSSQL_REPL20084 (o processo não pôde se conectar ao Assinante.) e MSSQL_REPL40532 (Não é possível abrir o nome do servidor> solicitado pelo login. Houve falha no logon.).  

Para usar todos os recursos do Banco de Dados SQL do Azure, você deve estar usando as versões mais recentes do [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) e do [SQL Server Data Tools](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt).  

  
## <a name="remarks"></a>Comentários

- Replicação pode ser configurada usando o [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) ou executando instruções Transact-SQL no editor. Você não pode configurar a replicação usando o portal do Azure.  
- A replicação só pode usar logons de autenticação do SQL Server para se conectar a um Banco de Dados SQL do Azure.
- Tabelas replicadas devem ter uma chave primária.  
- Você deve ter uma assinatura do Azure existente.  
- O assinante de Banco de Dados SQL do Azure pode estar em qualquer região.  
- Uma única publicação no SQL Server pode dar suporte a assinantes do Banco de Dados SQL do Azure e do SQL Server (local e SQL Server em uma máquina virtual do Azure).  
- Gerenciamento de replicação, monitoramento e solução de problemas devem ser executados no SQL Server local.  
- Há suporte apenas para assinaturas push no Banco de Dados SQL do Azure.  
- Somente `@subscriber_type = 0` tem suporte no **sp_addsubscription** para Banco de Dados SQL.  
- O Banco de Dados SQL do Azure não oferece suporte para replicação bidirecional, imediata, atualizável ou ponto a ponto.

## <a name="replication-architecture"></a>Arquitetura de replicação  

![replication-to-sql-database](./media/replication-to-sql-database/replication-to-sql-database.png)  

## <a name="scenarios"></a>Cenários  

### <a name="typical-replication-scenario"></a>Cenário típico de replicação  

1. Crie uma publicação de replicação transacional em um banco de dados do SQL Server local.  
2. No SQL Server local, use o **Assistente para Nova Assinatura** ou instruções Transact-SQL para criar um push para assinatura para o Banco de Dados SQL do Azure.  
3. Com bancos de dados individuais e em pool no Banco de Dados SQL do Azure, o conjunto de dados inicial é um instantâneo criado pelo Agente de Instantâneo e distribuído e aplicado pelo Agente de Distribuição. Com um banco de dados de instância gerenciada, você também pode usar um backup de banco de dados para propagar o banco de dados do assinante.

### <a name="data-migration-scenario"></a>Cenário de migração de dados  

1. Use a replicação transacional para replicar dados de um Banco de Dados SQL Server local para o Banco de Dados SQL do Azure.  
2. Redirecione o cliente ou aplicativos de camada intermediária para atualizar a cópia do Banco de Dados SQL do Azure.  
3. Interrompa a atualização da versão do SQL Server da tabela e remova a publicação.  

## <a name="limitations"></a>Limitações

As opções a seguir não têm suporte para assinaturas de Banco de Dados SQL do Azure:

- Copiar associação de grupos de arquivos  
- Copiar esquemas de particionamento de tabela  
- Copiar esquemas de particionamento de índice  
- Copiar estatísticas definidas pelo usuário  
- Copiar associações padrão  
- Copiar associações de regra  
- Copiar índices de texto completo  
- Copiar XML XSD  
- Copiar índices XML  
- Permissões de cópia  
- Copiar índices espaciais  
- Copiar índices filtrados  
- Copiar atributo de compactação de dados  
- Copiar atributo de coluna esparsa  
- Converter fluxo de arquivos em tipos de dados MAX  
- Converter hierarchyid em tipos de dados MAX  
- Converter espacial em tipos de dados MAX  
- Copiar propriedades estendidas  
- Permissões de cópia  

### <a name="limitations-to-be-determined"></a>Limitações a serem determinadas

- Copiar ordenação  
- Execução em uma transação serializável do SP  

## <a name="examples"></a>Exemplos

Crie uma publicação e uma assinatura push. Para obter mais informações, consulte:
  
- [Criar uma publicação](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)
- [Criar uma Assinatura Push](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/) usando o nome do servidor do Banco de Dados SQL do Azure como assinante (por exemplo **N'azuresqldbdns.database.windows.net'**) e o nome do Banco de Dados SQL do Azure como o banco de dados de destino (por exemplo, **AdventureWorks**).  

## <a name="see-also"></a>Consulte também  

- [Replicação transacional](sql-database-managed-instance-transactional-replication.md)
- [Criar uma publicação](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)
- [Create a Push Subscription](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/)
- [Tipos de Replicação](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication)
- [Monitorando (Replicação)](https://docs.microsoft.com/sql/relational-databases/replication/monitor/monitoring-replication)
- [Inicializar uma assinatura](https://docs.microsoft.com/sql/relational-databases/replication/initialize-a-subscription)  
