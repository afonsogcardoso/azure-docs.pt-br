---
title: Gerenciar administradores de servidor no Azure Analysis Services | Microsoft Docs
description: Este artigo descreve como gerenciar administradores de servidores para um servidor azure Analysis Services usando o portal Azure, PowerShell ou APIs REST.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 10/29/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: f7c57a5751f2ff34abb26b7653070ce4ee5010fe
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "73572619"
---
# <a name="manage-server-administrators"></a>Gerenciar administradores de servidor

Os administradores do servidor devem ser um usuário ou grupo de segurança válido no Azure AD (Azure Active Directory) para o locatário no qual o servidor reside. É possível usar **Administradores do Analysis Services** para o servidor no portal do Azure, Propriedades do Servidor no SSMS, PowerShell ou API REST para gerenciar administradores do servidor. 

**Os grupos** de segurança devem `MailEnabled` ser [ativados por e-mail](https://docs.microsoft.com/exchange/recipients-in-exchange-online/manage-mail-enabled-security-groups) com a propriedade definida como `True`. Ao especificar um grupo `obj:groupid@tenantid`por uso de endereço de e-mail .

## <a name="to-add-server-administrators-by-using-azure-portal"></a>Para adicionar administradores do servidor usando o Portal do Azure

1. No portal, para o seu servidor, clique em **Administradores do Analysis Services**.
2. Em ** \<> nome do servidor - Administração de Serviços de Análise,** clique **em Adicionar**.
3. Em **Adicionar Administradores de Servidor**, selecione contas de usuário do seu Azure Active Directory ou convide usuários externos por endereço de email.

    ![Administradores de servidor no portal do Azure](./media/analysis-services-server-admins/aas-manage-users-admins.png)

## <a name="to-add-server-administrators-by-using-ssms"></a>Para adicionar administradores do servidor usando o SSMS

1. Clique com botão direito do mouse em servidor > **Propriedades**.
2. Em **Propriedades do Analysis Server**, clique em **Segurança**.
3. Clique em **Adicionar** e, em seguida, insira o endereço de email de um usuário ou grupo em seu Azure AD.
   
    ![Adicionar administradores do servidor no SSMS](./media/analysis-services-server-admins/aas-manage-users-ssms.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Use [o cmdlet New-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/new-azanalysisservicesserver) para especificar o parâmetro Administrador ao criar um novo servidor. <br>
Use [set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver) cmdlet para modificar o parâmetro Administrador para um servidor existente.

## <a name="rest-api"></a>API REST

Use [Criar](https://docs.microsoft.com/rest/api/analysisservices/servers/create) para especificar a propriedade asAdministrator ao criar um novo servidor. <br>
Use [Atualizar](https://docs.microsoft.com/rest/api/analysisservices/servers/update) para especificar a propriedade asAdministrator ao modificar um servidor existente. <br>



## <a name="next-steps"></a>Próximas etapas 

[Autenticação e permissões de usuário](analysis-services-manage-users.md)  
[Gerenciar usuários e funções de banco de dados](analysis-services-database-users.md)  
[Controle de acesso baseado em função](../role-based-access-control/overview.md)  

