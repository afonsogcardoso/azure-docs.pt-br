---
title: Gerenciar permissões de banco de dados no Azure Data Explorer
description: Este artigo descreve os controles de acesso baseados em função para bancos de dados e tabelas no Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: b4d5e56e990c0353f44209c6b19ae2d1727de27a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76030096"
---
# <a name="manage-azure-data-explorer-database-permissions"></a>Gerenciar permissões do banco de dados do Azure Data Explorer

O Azure Data Explorer permite controlar o acesso a bancos de dados e tabelas, usando um modelo de *controle de acesso baseado em função*. Nesse modelo, as *entidades de segurança* (usuários, grupos e aplicativos) são mapeadas para *funções*. As entidades de segurança podem acessar os recursos de acordo com as funções atribuídas.

Este artigo descreve as funções disponíveis e como atribuir as entidades a essas funções usando o portal do Azure e os comandos de gerenciamento do Data Explorer do Azure.

## <a name="roles-and-permissions"></a>Funções e permissões

O Azure Data Explorer tem as seguintes funções:

|Função                       |Permissões                                                                        |
|---------------------------|-----------------------------------------------------------------------------------|
|Administrador do banco de dados             |Pode fazer qualquer ação no escopo de um banco de dados específico.|
|Usuário de banco de dados              |Pode ler todos os dados e metadados no banco de dados. Além disso, podem criar tabelas (tornando-se o administrador da tabela para essa tabela) e funções no banco de dados.|
|Visualizador do banco de dados            |Pode ler todos os dados e metadados no banco de dados.|
|Ingestor de banco de dados          |Pode ingerir dados para todas as tabelas existentes no banco de dados, mas não consultar os dados.|
|Monitor do banco de dados           |Pode executar comandos '.show ...' no contexto do banco de dados e suas entidades filhas.|
|Administrador de tabela                |Pode fazer qualquer ação no escopo de uma tabela específica. |
|Ingestor de tabela             |Pode ingerir dados no escopo de uma tabela específica, mas não consultar os dados.|

## <a name="manage-permissions-in-the-azure-portal"></a>Gerenciar permissões no portal do Azure

1. Faça login no [portal Azure](https://portal.azure.com/).

1. Navegue até o cluster do Azure Data Explorer.

1. Na seção **Visão geral**, selecione o banco de dados onde você quer gerenciar as permissões.

    ![Selecionar banco de dados](media/manage-database-permissions/select-database.png)

1. Selecione **Permissões** e, em seguida, **Adicionar**.

    ![Permissões de banco de dados](media/manage-database-permissions/database-permissions.png)

1. Em **Adicionar permissões de banco de dados**, selecione a função à qual você quer atribuir a entidade de segurança e, em seguida **Selecionar entidades de segurança**.

    ![Adicionar permissões de banco de dados](media/manage-database-permissions/add-permission.png)

1. Pesquise a entidade de segurança, selecione-a e, em seguida, **Selecionar**.

    ![Gerenciar permissões no portal do Azure](media/manage-database-permissions/new-principals.png)

1. Selecione **Salvar**.

    ![Gerenciar permissões no portal do Azure](media/manage-database-permissions/save-permission.png)

## <a name="manage-permissions-with-management-commands"></a>Gerenciar permissões com comandos de gerenciamento

1. Faça login [https://dataexplorer.azure.com](https://dataexplorer.azure.com)e adicione seu cluster se ele ainda não estiver disponível.

1. No painel esquerdo, selecione o banco de dados apropriado.

1. Use o comando `.add` para atribuir entidades de segurança a funções: `.add database databasename rolename ('aaduser | aadgroup=user@domain.com')`. Para adicionar um usuário à função de usuário do banco de dados, execute o seguinte comando, substituindo o nome do banco de dados e o usuário.

    ```Kusto
    .add database <TestDatabase> users ('aaduser=<user@contoso.com>')
    ```

    A saída do comando mostra a lista de usuários existentes e as funções às quais eles estão atribuídos no banco de dados.
    
    Para exemplos relativos ao Azure Active Directory e ao modelo de autorização kusto, consulte [Princípios e Provedores de Identidade](https://docs.microsoft.com/azure/kusto/management/access-control/principals-and-identity-providers)

## <a name="next-steps"></a>Próximas etapas

[Gravar consultas](write-queries.md)
