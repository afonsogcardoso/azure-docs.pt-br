---
title: Prepare máquinas de origem para instalar o Serviço de Mobilidade através da instalação push para recuperação de desastres de VMware VMs e servidores físicos para o Azure | Microsoft Docs
description: Saiba como preparar seu servidor para instalar o agente de mobilidade através da instalação push para recuperação de desastres de VMware VMs e servidores físicos para o Azure usando o serviço azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: ramamill
ms.openlocfilehash: a2f4bdb96b8d1ecb23ddcec844726439ec46fff2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74186444"
---
# <a name="prepare-source-machine-for-push-installation-of-mobility-agent"></a>Prepare a máquina de origem para a instalação de push de agente de mobilidade

Quando você configura a recuperação de desastre para VMs VMware e servidores físicos usando [Azure Site Recovery](site-recovery-overview.md), instale o [Serviço Mobilidade de Recuperação do Site](vmware-physical-mobility-service-overview.md) em cada VM VMware local e servidor físico.  O serviço Mobility captura gravações de dados na máquina e as encaminha para o servidor do processo de Recuperação do Site.

## <a name="install-on-windows-machine"></a>Instalar na máquina Windows

Em cada máquina Windows que você deseja proteger, faça o seguinte:

1. Certifique-se de que haja conectividade de rede entre a máquina e o servidor de processo. Se você não tiver configurado um servidor de processos separado, por padrão, ele estará em execução no servidor de configuração.
1. Crie uma conta que o servidor de processo possa usar para acessar o computador. A conta deve ter direitos de administrador, seja local ou do domínio. Use essa conta somente para a instalação por push e atualizações de agentes.
2. Se você não usar uma conta de domínio, desative o controle de acesso de usuário remoto no computador local da seguinte maneira:
    - Na chave do Registro HKEY_LOCAL_MACHINE \ SOFTWARE \ Microsoft \ Windows \ CurrentVersion \ Policies \ System, adicione um novo DWORD: **LocalAccountTokenFilterPolicy**. Defina o valor como **1**.
    -  Para fazer isso no prompt de comando, execute o seguinte comando:  
   `REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d
3. No Firewall do Windows na máquina que você deseja proteger, selecione **Permitir um aplicativo ou recurso pelo Firewall**. Habilite **Compartilhamento de Arquivo e Impressora** e **Instrumentação de Gerenciamento do Windows (WMI)**. Em computadores pertencentes a um domínio, pode-se configurar o firewall com um objeto de Política de Grupo (GPO).

   ![Configurações de firewall](./media/vmware-azure-install-mobility-service/mobility1.png)

4. Adicione a conta criada em CSPSConfigtool. Para fazer isso, entre servidor de configuração.
5. Abra **cspsconfigtool.exe**. Ele está disponível como um atalho na área de trabalho e na pasta %ProgramData%\ASR\home\svsystems\bin.
6. Na guia **Gerenciar Contas**, selecione **Adicionar Conta**.
7. Adicione a conta que você criou.
8. Insira as credenciais usadas quando você habilitar a replicação de um computador.

## <a name="install-on-linux-machine"></a>Instalar no computador Linux

Em cada máquina Linux que você deseja proteger, faça o seguinte:

1. Assegure-se de que haja conectividade de rede entre a máquina Linux e o servidor de processo.
2. Crie uma conta que o servidor de processo possa usar para acessar o computador. A conta deve ser um usuário **raiz** no servidor Linux de origem. Use essa conta somente para a instalação por push e atualizações.
3. Verifique se o arquivo /etc/hosts no servidor Linux de origem contém entradas que mapeiam o nome do host local para os endereços IP associados a todos os adaptadores de rede.
4. Instale os pacotes openssh, openssh-server e openssl mais recentes no computador que você deseja replicar.
5. Verifique se o Secure Shell (SSH) está habilitado e em execução na porta 22.
4. Habilitar a autenticação de senha e subsistema SFTP no arquivo sshd_config. Para fazer isso, faça login como **raiz.**
5. No arquivo **/etc/ssh/sshd_config,** encontre a linha que começa com **Autenticação de senhas**.
6. Remova a marca de comentário da linha e altere o valor para **yes**.
7. Encontre a linha que começa com **Subsystem**e não comente a linha.

      ![Linux](./media/vmware-azure-install-mobility-service/mobility2.png)

8. Reinicie o serviço **sshd.**
9. Adicione a conta criada em CSPSConfigtool. Para fazer isso, entre servidor de configuração.
10. Abra **cspsconfigtool.exe**. Ele está disponível como atalho na área de trabalho e na pasta %ProgramData%\home\svsystems\bin.
11. Na guia **Gerenciar Contas**, selecione **Adicionar Conta**.
12. Adicione a conta que você criou.
13. Insira as credenciais usadas quando você habilitar a replicação de um computador.

## <a name="anti-virus-on-replicated-machines"></a>Antivírus em computadores replicados

Se as máquinas que você deseja replicar tiverem o software antivírus ativo em execução, exclua a pasta de instalação do serviço Mobility das operações de antivírus (*C:\ProgramData\ASR\agent*). Isso garante que a replicação funciona conforme o esperado.

## <a name="next-steps"></a>Próximas etapas

Depois que o Mobility Service for instalado, no portal do Azure, selecione **+ Replicate** para começar a proteger essas VMs. Saiba mais sobre como ativar a replicação de VMs e [servidores físicos](physical-azure-disaster-recovery.md#enable-replication)da [VMware](vmware-azure-enable-replication.md) .


