---
title: Restaurar dados no Azure para um servidor Windows
description: Neste artigo, saiba como restaurar os dados armazenados no Azure em um servidor Windows ou computador Windows com o Agente microsoft Azure Recovery Services (MARS).
ms.reviewer: saurse
ms.topic: conceptual
ms.date: 09/07/2018
ms.openlocfilehash: 25ca8eecaeb615f071340188a23fae7978ddb75c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79409806"
---
# <a name="restore-files-to-windows-by-using-the-azure-resource-manager-deployment-model"></a>Restaurar arquivos no Windows usando o modelo de implantação do Azure Resource Manager

Este artigo explica como restaurar dados de um cofre de backup. Para restaurar dados, use o assistente para Recuperar Dados no agente do MARS (Serviços de Recuperação do Microsoft Azure). Você pode:

* Restaurar dados para o mesmo computador do qual os backups foram realizados.
* Restaurar dados para um computador alternativo.

Use o recurso de Restauração Instantânea para montar um instantâneo de ponto de recuperação gravável como volume de recuperação. Em seguida, é possível explorar os arquivos de volume de recuperação e de cópia em um computador local e, assim, restaurar arquivos de forma seletiva.

> [!NOTE]
> A [atualização do Backup do Azure de janeiro de 2017](https://support.microsoft.com/help/3216528/azure-backup-update-for-microsoft-azure-recovery-services-agent-januar) será necessária se você desejar usar a Restauração Instantânea para restaurar dados. Além disso, os dados de backup devem ser protegidos em cofres nas localidades listadas no artigo de suporte. Consulte a [atualização do Backup do Azure de janeiro de 2017](https://support.microsoft.com/help/3216528/azure-backup-update-for-microsoft-azure-recovery-services-agent-januar) para obter a lista mais recente de localidades que oferecem suporte à Restauração Instantânea.
>

Usar a Restauração Instantânea com os cofres dos Serviços de Recuperação no Portal do Azure. Se você tiver armazenado os dados em cofres de Backup, eles terão sido convertidos em cofres dos Serviços de Recuperação. Se desejar usar a Restauração Instantânea, baixe a atualização do MARS e siga os procedimentos que mencionam a Restauração Instantânea.

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="use-instant-restore-to-recover-data-to-the-same-machine"></a>Use a Restauração Instantânea para recuperar dados no mesmo computador

Se você excluiu acidentalmente um arquivo e deseja restaurá-lo para o mesmo computador (do qual o backup foi feito), as etapas a seguir o ajudarão a recuperar os dados.

1. Abra o snap-in do **Backup do Microsoft Azure** . Se você não souber onde o snap-in foi instalado, pesquise **Backup do Microsoft Azure** no computador ou servidor.

    O aplicativo da área de trabalho deve aparecer nos resultados da pesquisa.

2. Selecione **Recuperar Dados** para iniciar o assistente.

    ![Captura de tela do Backup do Azure, com Recuperar Dados realçado](./media/backup-azure-restore-windows-server/recover.png)

3. Na página **Como iniciar,** para restaurar os dados no mesmo servidor ou computador, selecione **Este servidor (`<server name>`)****Next** > .

    ![Captura de tela da página Introdução ao Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/samemachine_gettingstarted_instantrestore.png)

4. Na página **Selecionar Modo de Recuperação**, escolha **Pastas e arquivos individuais** > **Avançar**.

    ![Captura de tela da página Selecionar Modo de Recuperação do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/samemachine_selectrecoverymode_instantrestore.png)
   > [!IMPORTANT]
   > A opção de restauração de arquivos e pastas individuais exige o .NET Framework 4.5.2 ou posterior. Se você não vir a opção **arquivos e pastas individuais**, você deve atualizar o .NET Framework para a versão 4.5.2 ou posterior e tente novamente.
 
   > [!TIP]
   > A opção **Arquivos e pastas Individuais** permite acesso rápido aos dados do ponto de recuperação. Ela é adequada para a recuperação de arquivos individuais, com tamanhos totalizando não mais de 80 GB e ofertas de velocidades de transferência/cópia de até 6 MBps durante a recuperação. A opção **Volume** recupera todos os backups de dados em um volume especificado. Essa opção fornece velocidades de transferência mais rápidas (no máximo 60 MBps), que são ideais para a recuperação de dados grandes ou volumes inteiros.

5. Na página **Selecionar Volume e Data**, selecione o volume que contém os arquivos e pastas que você deseja restaurar.

    No calendário, selecione um ponto de recuperação. As datas em **negrito** indicam a disponibilidade de pelo menos um ponto de recuperação. Se houver vários pontos de recuperação disponíveis em uma data, escolha o ponto de recuperação específico no menu suspenso **Hora**.

    ![Captura de tela da página Selecionar Volume e Data do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/samemachine_selectvolumedate_instantrestore.png)

6. Depois de escolher o ponto de recuperação para restaurar, selecione **Montar**.

    O Backup do Azure monta o ponto de recuperação local e o usa como um volume de recuperação.

7. Na página **Procurar e Recuperar arquivos,** **selecione Procurar** para abrir o Windows Explorer e encontre os arquivos e pastas desejadas.

    ![Captura de tela da página Procurar e Recuperar Arquivos do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/samemachine_browserecover_instantrestore.png)

8. No Windows Explorer, copie os arquivos e pastas que deseja restaurar e cole-os em qualquer localização local no servidor ou computador. Você pode abrir ou transmitir os arquivos diretamente do volume de recuperação e verificar se você está recuperando as versões corretas.

    ![Captura de tela do Windows Explorer, com Cópia realçada](./media/backup-azure-restore-windows-server/samemachine_copy_instantrestore.png)

9. Após terminar, na página **Procurar e Recuperar Arquivos**, selecione **Desmontar**. Selecione **Sim** para confirmar que deseja desmontar o volume.

    ![Captura de tela da página Procurar e Recuperar Arquivos do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/samemachine_unmount_instantrestore.png)

    > [!Important]
    > Se você não selecionar **Desmontar**, o volume de recuperação permanecerá montado por seis horas a partir da hora em que foi montado. No entanto, o tempo de montagem é estendido até um máximo de 24 horas no caso de uma cópia de arquivo em andamento. Não será executada nenhuma operação de backup enquanto o volume estiver montado. Qualquer operação de backup agendada para execução durante o tempo em que o volume estiver montado será executada após o volume de recuperação ser desmontado.
    >

## <a name="use-instant-restore-to-restore-data-to-an-alternate-machine"></a>Usar a Restauração Instantânea para restaurar dados em um computador alternativo

Se o servidor inteiro for perdido, você ainda pode recuperar dados do backup do Azure para um computador diferente. As etapas a seguir ilustram o fluxo de trabalho.

Essas etapas incluem a seguinte terminologia:

* *Máquina de origem* – A máquina original da qual o backup foi retirado e que está atualmente indisponível.
* *Computador de destino* – O computador para o qual os dados estão sendo recuperados.
* *Cofre de* amostras – O cofre dos Serviços de Recuperação para o qual a máquina de origem e a máquina de destino estão registradas.

> [!NOTE]
> Os backups não podem ser restaurados em um computador de destino que esteja executando uma versão anterior do sistema operacional. Por exemplo, um backup retirado de um computador Windows 7 pode ser restaurado em um computador Windows 7 (ou posterior). Um backup feito em um computador com Windows 8 não pode ser restaurado em um computador com Windows 7.
>
>

1. Abra o snap-in **do Microsoft Azure Backup** na máquina de destino.

2. Verifique se a máquina de destino e a máquina de origem estão registradas no mesmo cofre dos Serviços de Recuperação.

3. Clique em **Recuperar Dados** para abrir o **Assistente de Recuperação de Dados**.

    ![Captura de tela do Backup do Azure, com Recuperar Dados realçado](./media/backup-azure-restore-windows-server/recover.png)

4. Na página **Introdução**, selecione **Outro servidor**.

    ![Captura de tela da página Introdução ao Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/alternatemachine_gettingstarted_instantrestore.png)

5. Forneça o arquivo de credencial de cofre que corresponde ao cofre de exemplo e selecione **Próximo**.

    Se o arquivo de credencial de cofre for inválido (ou tiver expirado), baixe um novo arquivo de credencial de cofre do cofre de exemplo no portal do Azure. Depois de fornecer uma credencial de cofre válida, o nome do cofre de backup correspondente aparecerá.

6. Na página **Selecionar Servidor de Backup**, selecione o computador de origem na lista de computadores exibidos e forneça a senha. Em seguida, selecione **Next**.

    ![Captura de tela da página Selecionar Servidor de Backup do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/alternatemachine_selectmachine_instantrestore.png)

7. Na página **Selecionar Modo de Recuperação**, selecione **Pastas e arquivos individuais** > **Próximo**.

    ![Captura de tela da página Selecionar Modo de Recuperação do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/alternatemachine_selectrecoverymode_instantrestore.png)

8. Na página **Selecionar Volume e Data**, selecione o volume que contém os arquivos e pastas que você deseja restaurar.

    No calendário, selecione um ponto de recuperação. As datas em **negrito** indicam a disponibilidade de pelo menos um ponto de recuperação. Se houver vários pontos de recuperação disponíveis em uma data, escolha o ponto de recuperação específico no menu suspenso **Hora**.

    ![Captura de tela da página Selecionar Volume e Data do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/alternatemachine_selectvolumedate_instantrestore.png)

9. Selecione **Montar** para montar localmente o ponto de recuperação como um volume de recuperação em seu computador de destino.

10. Na página **Procurar e recuperar arquivos,** **selecione Procurar** para abrir o Windows Explorer e encontre os arquivos e pastas desejadas.

    ![Captura de tela da página Procurar e Recuperar Arquivos do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/alternatemachine_browserecover_instantrestore.png)

11. No Windows Explorer, copie os arquivos e pastas do volume de recuperação e cole-os na localização de seu computador de destino. Você pode abrir ou transmitir os arquivos diretamente do volume de recuperação e verificar se as versões corretas são recuperadas.

    ![Captura de tela do Windows Explorer, com Cópia realçada](./media/backup-azure-restore-windows-server/alternatemachine_copy_instantrestore.png)

12. Após terminar, na página **Procurar e Recuperar Arquivos**, selecione **Desmontar**. Selecione **Sim** para confirmar que deseja desmontar o volume.

    ![Captura de tela da página Procurar e Recuperar Arquivos do Assistente para Recuperar Dados](./media/backup-azure-restore-windows-server/alternatemachine_unmount_instantrestore.png)

    > [!Important]
    > Se você não selecionar **Desmontar**, o volume de recuperação permanecerá montado por seis horas a partir da hora em que foi montado. No entanto, o tempo de montagem é estendido até um máximo de 24 horas no caso de uma cópia de arquivo em andamento. Não será executada nenhuma operação de backup enquanto o volume estiver montado. Qualquer operação de backup agendada para execução durante o tempo em que o volume estiver montado será executada após o volume de recuperação ser desmontado.
    >

## <a name="next-steps"></a>Próximas etapas

* Agora que você restaurou seus arquivos e pastas, poderá [gerenciar seus backups](backup-azure-manage-windows-server.md).

* Encontre [perguntas comuns sobre backup de arquivos e pastas](backup-azure-file-folder-backup-faq.md).
