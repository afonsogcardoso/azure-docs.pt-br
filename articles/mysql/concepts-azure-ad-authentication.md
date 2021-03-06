---
title: Autenticação de diretório ativo - Banco de dados Azure para MySQL
description: Conheça os conceitos do Azure Active Directory para autenticação com o Banco de Dados Azure para MySQL
author: lfittl-msft
ms.author: lufittl
ms.service: mysql
ms.topic: conceptual
ms.date: 01/22/2019
ms.openlocfilehash: 960536c3f80aa7870d6f2056d8e95cd1a4338dfe
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79299015"
---
# <a name="use-azure-active-directory-for-authenticating-with-mysql"></a>Use o Azure Active Directory para autenticar com o MySQL

A autenticação do Microsoft Azure Active Directory (Azure AD) é um mecanismo de conexão ao Banco de Dados Azure para MySQL usando identidades definidas no Azure AD.
Com a autenticação do Azure AD, você pode gerenciar identidades de usuários de banco de dados e outros serviços da Microsoft em um local central, o que simplifica o gerenciamento de permissões.

> [!IMPORTANT]
> A autenticação Azure AD para O Banco de Dados Azure para MySQL está atualmente em visualização pública.
> Essa versão prévia é fornecida sem um contrato de nível de serviço e não é recomendada para cargas de trabalho de produção. Alguns recursos podem não ter suporte ou podem ter restrição de recursos.
> Para obter mais informações, consulte [Termos de Uso Suplementares para Visualizações do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Os benefícios do uso do Azure AD incluem:

- Autenticação de usuários em todos os Serviços Azure de forma uniforme
- Gerenciamento de políticas de senha e rotação de senhas em um único lugar
- Várias formas de autenticação suportadas pelo Azure Active Directory, que podem eliminar a necessidade de armazenar senhas
- Os clientes podem gerenciar permissões de banco de dados usando grupos (Azure AD) externos.
- A autenticação Azure AD usa usuários do banco de dados MySQL para autenticar identidades no nível do banco de dados
- Suporte à autenticação baseada em tokens para aplicativos conectados ao Banco de Dados Do Azure para MySQL

Para configurar e usar a autenticação do Azure Active Directory, use o seguinte processo:

1. Crie e preencha o Azure Active Directory com as identidades dos usuários conforme necessário.
2. Associar ou alterar opcionalmente o Active Directory atualmente associado à sua assinatura do Azure.
3. Crie um administrador Azure AD para o banco de dados Azure para o servidor MySQL.
4. Crie usuários de banco de dados em seu banco de dados mapeados para as identidades do Azure AD.
5. Conecte-se ao seu banco de dados recuperando um token para uma identidade Azure AD e login.

> [!NOTE]
> Para saber como criar e preencher o Azure AD e, em seguida, configurar o Azure AD com o Azure Database para MySQL, consulte [Configurar e fazer login com o Azure AD para O Banco de Dados Azure para MySQL](howto-configure-sign-in-azure-ad-authentication.md).

## <a name="architecture"></a>Arquitetura

O diagrama de alto nível a seguir resume como a autenticação funciona usando a autenticação Azure AD com o Azure Database for MySQL. As setas indicam caminhos para comunicação.

![fluxo de autenticação][1]

## <a name="administrator-structure"></a>Estrutura do administrador

Ao usar a autenticação Azure AD, existem duas contas de administrador para o servidor MySQL; o administrador Original MySQL e o administrador Azure AD. Somente o administrador com base em uma conta do AD do Azure pode criar o primeiro usuário de banco de dados do AD do Azure contido em um banco de dados de usuário. O logon de administrador do AD do Azure pode ser um usuário ou um grupo do AD do Azure. Quando o administrador é uma conta de grupo, ele pode ser usado por qualquer membro do grupo, permitindo vários administradores azure AD para o servidor MySQL. O uso de uma conta de grupo como administrador melhora a capacidade de gerenciamento, permitindo que você adicione e remova centralmente os membros do grupo no Azure AD sem alterar os usuários ou permissões no servidor MySQL. Somente um administrador do AD do Azure (um usuário ou grupo) pode ser configurado por vez, a qualquer momento.

![estrutura de administrador][2]

## <a name="permissions"></a>Permissões

Para criar novos usuários que possam autenticar com o Azure AD, você deve ser o administrador Azure AD projetado. Esse usuário é atribuído configurando a conta Administrador Ad do Azure para um banco de dados Azure específico para o servidor MySQL.

Para criar um novo usuário de banco de dados Azure AD, você deve se conectar como o administrador do Azure AD. Isso é demonstrado no [Configure and Login with Azure AD for Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md).

Qualquer autenticação Azure AD só é possível se o administrador do Azure AD foi criado para o Azure Database for MySQL. Se o administrador do Azure Active Directory foi removido do servidor, os usuários existentes do Azure Active Directory criados anteriormente não poderão mais se conectar ao banco de dados usando suas credenciais do Azure Active Directory.

## <a name="connecting-using-azure-ad-identities"></a>Conectar-se usando as identidades do Azure AD

A autenticação do Active Directory do Azure dá suporte aos seguintes métodos de conexão a um banco de dados usando identidades do AD do Azure:

- Senha do Azure Active Directory
- Integrada do Azure Active Directory
- Universal do Azure Active Directory com o MFA
- Usando certificados de aplicativo de diretório ativo ou segredos de cliente

Depois de autenticar contra o Active Directory, você recupera um token. Este token é sua senha para fazer login.

> [!NOTE]
> Para obter mais detalhes sobre como se conectar com um token Active Directory, consulte [Configurar e fazer login com o Azure AD para O Banco de Dados Azure para MySQL](howto-configure-sign-in-azure-ad-authentication.md).

## <a name="additional-considerations"></a>Considerações adicionais

- Apenas um administrador Azure AD pode ser configurado para um banco de dados Azure para servidor MySQL a qualquer momento.
- Apenas um administrador Azure AD do MySQL pode inicialmente se conectar ao Banco de Dados Azure para MySQL usando uma conta do Azure Active Directory. O administrador do Active Directory pode configurar os próximos usuários do banco de dados do Azure AD.
- Se um usuário for excluído do Azure AD, esse usuário não poderá mais autenticar com o Azure AD e, portanto, não será mais possível adquirir um token de acesso para esse usuário. Neste caso, embora o usuário correspondente ainda esteja no banco de dados, não será possível se conectar ao servidor com esse usuário.
> [!NOTE]
> O login com o usuário Azure AD excluído ainda pode ser feito até que o token expire (até 60 minutos após a emissão do token).  Se você também remover o usuário do Banco de Dados Do Azure para MySQL, esse acesso será revogado imediatamente.
- Se o administrador do Azure AD for removido do servidor, o servidor não será mais associado a um inquilino Azure AD e, portanto, todos os logins do Azure AD serão desativados para o servidor. A adição de um novo administrador azure AD do mesmo inquilino reativará os logins do Azure AD.
- O Banco de Dados Azure para MySQL corresponde aos tokens de acesso ao Banco de Dados Azure para o usuário MySQL usando o ID de usuário Azure AD exclusivo do usuário, em vez de usar o nome de usuário. Isso significa que, se um usuário do Azure AD for excluído no Azure AD e um novo usuário criado com o mesmo nome, o Azure Database for MySQL considera que um usuário diferente. Portanto, se um usuário for excluído do Azure AD e, em seguida, um novo usuário com o mesmo nome adicionado, o novo usuário não poderá se conectar com o usuário existente.

## <a name="next-steps"></a>Próximas etapas

- Para saber como criar e preencher o Azure AD e, em seguida, configurar o Azure AD com o Azure Database para MySQL, consulte [Configurar e fazer login com o Azure AD para O Banco de Dados Azure para MySQL](howto-configure-sign-in-azure-ad-authentication.md).
- Para obter uma visão geral dos logins e usuários de banco de dados para o Azure Database for MySQL, consulte Criar usuários no Banco de [Dados Azure para MySQL](howto-create-users.md).

<!--Image references-->

[1]: ./media/concepts-azure-ad-authentication/authentication-flow.png
[2]: ./media/concepts-azure-ad-authentication/admin-structure.png
