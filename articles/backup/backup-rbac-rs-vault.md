---
title: Gerenciar backups com controle de acesso baseado em função
description: Use o Controle de Acesso baseado em função para gerenciar o acesso a operações de gerenciamento de backups em um cofre dos Serviços de Recuperação.
ms.reviewer: utraghuv
ms.topic: conceptual
ms.date: 06/24/2019
ms.openlocfilehash: e2e32ac6981635e3b9885119fdf397783ac32cc9
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79273196"
---
# <a name="use-role-based-access-control-to-manage-azure-backup-recovery-points"></a>Use o Controle de Acesso baseado em função para gerenciar pontos de recuperação de Backup do Azure

O RBAC (controle de acesso baseado em função) do Azure permite o gerenciamento de acesso refinado para o Azure. Com o RBAC, você pode separar as tarefas dentro de sua equipe e conceder somente a quantidade de acesso que os usuários precisam para realizar seus trabalhos.

> [!IMPORTANT]
> As funções fornecidas pelo Azure Backup estão limitadas a ações que podem ser executadas no portal Azure ou através de API rest ou cofre de serviços de recuperação PowerShell ou CMDlets CLI. As ações executadas na interface de usuário do Cliente do Agente de backup do Azure, na interface de usuário do System Center Data Protection Manager ou no Servidor de Backup do Azure estão fora do controle dessas funções.

O Azure Backup fornece três funções incorporadas para controlar as operações de gerenciamento de backup. Saiba mais sobre as [funções internas do RBAC do Azure](../role-based-access-control/built-in-roles.md)

* [Contribuinte de backup](../role-based-access-control/built-in-roles.md#backup-contributor) - Esta função tem todas as permissões para criar e gerenciar backup, exceto excluir o cofre dos Serviços de Recuperação e dar acesso a outros. Imagine essa função como administrador do gerenciamento de backups, que pode executar todas as operações de gerenciamento de backups.
* [Operador de Backup](../role-based-access-control/built-in-roles.md#backup-operator): essa função tem permissões para fazer tudo que um colaborador faz, exceto remover backups e gerenciar políticas de backup. Essa função é equivalente à de colaborador, com exceção de que não é possível executar operações destrutivas, como interromper backups com exclusão de dados ou remover registro de recursos locais.
* [Leitor de Backup](../role-based-access-control/built-in-roles.md#backup-reader): essa função tem permissões para exibir todas as operações de gerenciamento de backups. Imagine essa função como uma pessoa de monitoramento.

Se você está procurando definir seus próprios papéis para obter ainda mais controle, veja como [construir funções personalizadas](../role-based-access-control/custom-roles.md) no Azure RBAC.

## <a name="mapping-backup-built-in-roles-to-backup-management-actions"></a>Mapeando funções internas de backup para ações de gerenciamento de backups

A tabela a seguir captura as ações de gerenciamento de backups e a função RBAC mínima correspondente necessária para executar essa operação.

| Operação de gerenciamento | Função RBAC mínima necessária | Escopo exigido |
| --- | --- | --- |
| Criar cofre de Serviços de Recuperação | Colaborador de Backup | Grupo de recursos contendo o cofre |
| Habilitar backup de VMs do Azure | Operador de Backup | Grupo de recursos contendo o cofre |
| | Colaborador de Máquina Virtual | Recurso de VM |
| Backup sob demanda de VM | Operador de Backup | Cofre dos Serviços de Recuperação |
| Restaurar VM | Operador de Backup | Cofre dos Serviços de Recuperação |
| | Colaborador | Grupo de recursos no qual a VM será implantada |
| | Colaborador de Máquina Virtual | VM fonte que foi backup |
| Restaurar backup de VM de discos não gerenciados | Operador de Backup | Cofre dos Serviços de Recuperação |
| | Colaborador de Máquina Virtual | VM fonte que foi backup |
| | Colaborador da Conta de Armazenamento | Recurso de conta de armazenamento no qual os discos serão restaurados |
| Restaurar discos gerenciados do backup da VM | Operador de Backup | Cofre dos Serviços de Recuperação |
| | Colaborador de Máquina Virtual | VM fonte que foi backup |
| | Colaborador da Conta de Armazenamento | Conta de armazenamento temporária selecionada como parte da restauração para manter dados do cofre antes de convertê-los em discos gerenciados |
| | Colaborador | Grupo de recursos para o qual o disco gerenciado, ou discos, será restaurado |
| Restaurar arquivos individuais do backup da VM | Operador de Backup | Cofre dos Serviços de Recuperação |
| | Colaborador de Máquina Virtual | VM fonte que foi backup |
| Criar política de backup para backup da VM do Azure | Colaborador de Backup | Cofre dos Serviços de Recuperação |
| Modificar a política de backup da VM do Azure | Colaborador de Backup | Cofre dos Serviços de Recuperação |
| Excluir a política de backup da VM do Azure | Colaborador de Backup | Cofre dos Serviços de Recuperação |
| Interromper o backup (com retenção de dados ou exclusão de dados) no backup da VM | Colaborador de Backup | Cofre dos Serviços de Recuperação |
| Registrar-se no Windows Server/cliente/SCDPM local ou no Servidor de Backup do Azure | Operador de Backup | Cofre dos Serviços de Recuperação |
| Excluir o Windows Server/cliente/SCDPM local registrado ou o Servidor de Backup do Azure | Colaborador de Backup | Cofre dos Serviços de Recuperação |

> [!IMPORTANT]
> Se você especificar o Colaborador de VM em um escopo de recurso da VM e clicar em Backup como parte das configurações da VM, ele abrirá a tela 'Habilitar Backup', mesmo que o backup da VM já tenha sido realizado, pois a chamada para verificar o status do backup funciona apenas no nível de assinatura. Para evitar isso, vá para o cofre e abra a exibição de item de backup da VM ou especifique a função de Colaborador de VM em um nível de assinatura.

## <a name="minimum-role-requirements-for-the-azure-file-share-backup"></a>Requisitos mínimos de função para o backup do compartilhamento de arquivos do Azure

A tabela a seguir captura as ações de gerenciamento de backup e a função correspondente necessária para executar a operação de compartilhamento de arquivos do Azure.

| Operação de gerenciamento | Função necessária | Recursos |
| --- | --- | --- |
| Habilitar o backup das ações do Azure File | Colaborador de Backup |Cofre dos Serviços de Recuperação |
| |Conta de Armazenamento | Recurso de conta de armazenamento de contribuintes |
| Backup sob demanda de VM | Operador de Backup | Cofre dos Serviços de Recuperação |
| Restaurar compartilhamento de arquivos | Operador de Backup | Cofre dos Serviços de Recuperação |
| | Colaborador da Conta de Armazenamento | Recursos da conta de armazenamento onde a fonte de restauração e os compartilhamentos de arquivos target estão presentes |
| Restaurar arquivos individuais | Operador de Backup | Cofre dos Serviços de Recuperação |
| |Colaborador da Conta de Armazenamento|Recursos da conta de armazenamento onde a fonte de restauração e os compartilhamentos de arquivos target estão presentes |
| Parar a proteção |Colaborador de Backup | Cofre dos Serviços de Recuperação |
| Cancelar o registro da conta de armazenamento do cofre |Colaborador de Backup | Cofre dos Serviços de Recuperação |
| |Colaborador da Conta de Armazenamento | Recurso da conta de armazenamento|

## <a name="next-steps"></a>Próximas etapas

* [Controle de Acesso Baseado em Função](../role-based-access-control/role-assignments-portal.md): introdução ao RBAC no portal do Azure.
* Saiba como gerenciar o acesso com:
  * [Powershell](../role-based-access-control/role-assignments-powershell.md)
  * [Azure CLI](../role-based-access-control/role-assignments-cli.md)
  * [REST API](../role-based-access-control/role-assignments-rest.md)
* [Solução de problemas de Controle de Acesso Baseado em Função](../role-based-access-control/troubleshooting.md): obtenha sugestões para corrigir problemas comuns.
