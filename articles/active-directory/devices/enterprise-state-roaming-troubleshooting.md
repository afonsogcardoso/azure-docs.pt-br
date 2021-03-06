---
title: Solucionar problemas do Enterprise State Roaming no Azure Active Directory
description: Responde a algumas dúvidas que os administradores de TI podem ter sobre as configurações e a sincronização de dados do aplicativo.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 02/12/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: tanning
ms.collection: M365-identity-device-management
ms.openlocfilehash: ae8ce24aeb665a7f99326e83dbe18d020e1b6196
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78672345"
---
# <a name="troubleshooting-enterprise-state-roaming-settings-in-azure-active-directory"></a>Solucionando problemas de configurações do Enterprise State Roaming no Azure Active Directory

Este tópico fornece informações sobre como diagnosticar e solucionar problemas com o Enterprise State Roaming, além de fornecer uma lista de problemas conhecidos.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!NOTE]
> Este artigo se aplica ao navegador baseado em HTML Microsoft Edge Legacy, lançado com o Windows 10 em julho de 2015. O artigo não se aplica ao novo navegador baseado no Microsoft Edge Chromium, lançado em 15 de janeiro de 2020. Para obter mais informações sobre o comportamento do Sync para o novo Microsoft Edge, consulte o artigo [Microsoft Edge Sync](/deployedge/microsoft-edge-enterprise-sync).

## <a name="preliminary-steps-for-troubleshooting"></a>Etapas preliminares para a solução de problemas 

Antes de iniciar a solução de problemas, verifique se o usuário e o dispositivo foram configurados corretamente e se todos os requisitos do Enterprise State Roaming foram atendidos pelo dispositivo e pelo usuário. 

1. O Windows 10, com as atualizações mais recentes e a versão mínima 1511 (compilação de SO 10586 ou posterior), está instalado no dispositivo. 
1. O dispositivo é adicionado ao Azure AD ou ao Azure AD híbrido. Para obter mais informações, consulte [como colocar um dispositivo sob controle do Azure AD](overview.md).
1. Verifique se **Enterprise State Roaming** está habilitado para o locatário no Azure AD conforme descrito em [Para habilitar Enterprise State Roaming](enterprise-state-roaming-enable.md). Você pode habilitar roaming para todos os usuários ou apenas um grupo selecionado de usuários.
1. O usuário é atribuído a uma licença Azure Active Directory Premium.  
1. O dispositivo deve ser reiniciado e o usuário deve entrar novamente para acessar recursos de Enterprise State Roaming.

## <a name="information-to-include-when-you-need-help"></a>Informações a serem incluídas quando precisar de ajuda
Se você não conseguir solucionar seu problema com as orientações abaixo, pode entrar em contato com nossos engenheiros de suporte. Ao entrar em contato com eles, inclua as seguintes informações:

* **Descrição geral do erro**: o usuário viu mensagens de erro? Se não havia nenhuma mensagem de erro, descreva o comportamento inesperado observado em detalhes. Quais recursos estão habilitados para sincronização e o que o usuário espera sincronizar? Muitos recursos não estão sendo sincronizados ou apenas um?
* **Usuários afetados** – A sincronização está funcionando/falhando para um usuário ou vários usuários? Quantos dispositivos estão envolvidos por usuário? Nenhum deles está sincronizando ou alguns estão e outros não?
* **Informações sobre o usuário** – qual identidade o usuário está usando para entrar no dispositivo? Como o usuário está entrando no dispositivo? Eles são parte de um grupo de segurança selecionado autorizado a fazer a sincronização? 
* **Informações sobre o dispositivo** – esse dispositivo está adicionado ao Azure AD ou ao domínio? Em que compilação o dispositivo está? Quais são as atualizações mais recentes?
* **Data / Hora / Fuso horário** – Qual foi a data e hora precisas que você viu o erro (inclua o fuso horário)?

A inclusão dessas informações nos ajudará a resolver seu problema o mais rápido possível.

## <a name="troubleshooting-and-diagnosing-issues"></a>Solução de problemas e diagnósticos

Esta seção fornece sugestões sobre como solucionar e diagnosticar problemas relacionados ao Enterprise State Roaming.

## <a name="verify-sync-and-the-sync-your-settings-settings-page"></a>Verifique a sincronização e a página de configurações "Sincronize suas configurações" 

1. Após associar seu PC com Windows 10 a um domínio que esteja configurado para permitir o Enterprise State Roaming, entre com sua conta do trabalho. Vá para **Configurações** > **Contas** > **Sincronizar suas configurações** e confirmar que a sincronização e as configurações individuais estão em funcionamento, e que a parte superior da página de configurações indica que você está sincronizando com sua conta de trabalho. Confirme que a mesma conta também é usada como sua conta de login em **Configurações** > **Contas** > **Suas Informações**. 
1. Verifique se a sincronização funciona em várias máquinas fazendo algumas alterações na máquina original, como mover a barra de tarefas para o lado direito ou superior da tela. Observe a alteração ser propagada para o segundo computador em cinco minutos. 

   * Bloquear e desbloquear a tela (Win + L) pode ajudar a disparar uma sincronização.
   * Você deve entrar com a mesma conta nos dois computadores para que a sincronização funcione, pois o Enterprise State Roaming está associado à conta de usuário e não à conta do computador.

**Possível problema**: se os controles na página **Configurações** não estiverem disponíveis e você visualizar a mensagem "Alguns recursos do Windows só estarão disponíveis se você estiver usando uma conta da Microsoft ou uma conta corporativa". Esse problema pode surgir para dispositivos que tenham sido configurados para serem adicionados ao domínio e registrados no Azure AD, mas o dispositivo ainda não foi autenticado com êxito para o Azure AD. Uma possível causa é que a política do dispositivo deve ser aplicada, mas essa aplicação ocorre de maneira assíncrona e pode ser atrasada por algumas horas. 

### <a name="verify-the-device-registration-status"></a>Verificar o status de registro do dispositivo

O Enterprise State Roaming requer que o dispositivo seja registrado com o AD do Azure. Embora não seja específico ao Enterprise State Roaming, seguir as instruções abaixo poderá ajudar a confirmar se o Windows 10 Client está registrado, bem como confirmar a impressão digital, a URL das configurações do Azure AD, o status do NGC e outras informações.

1. Abra o prompt de comando sem privilégios elevados. Para fazer isso no Windows, abra o inicializador de Execução (Win + R) e digite “cmd” para abrir.
1. Depois de aberto o prompt de comando, digite “*dsregcmd.exe /status*”.
1. Para o resultado esperado, o valor do campo **AzureAdJoined** deve ser “YES”, o valor do campo **WamDefaultSet** deve ser “YES” e o valor do campo **WamDefaultGUID** deve ser um GUID com “(AzureAd)” no final.

**Problema potencial**: **WamDefaultSet** e **AzureAdJoined** ambos têm "NÃO" no valor de campo, o dispositivo foi acompanhado por domínio e registrado no Azure AD, e o dispositivo não sincroniza. Se estiver mostrando isso, o dispositivo pode precisar esperar que a política seja aplicada ou a autenticação do dispositivo falhou ao se conectar ao Azure AD. O usuário pode precisar aguardar algumas horas para que a política a seja aplicada. Outras etapas de solução de problemas podem incluir a tentativa de regravar o registro automático, fazendo a retirada e a volta, ou iniciando a tarefa no Task Scheduler. Em alguns casos, executar "*dsregcmd.exe /leave*" em uma janela de prompt de comando elevado, reinicializar e tentar se registrar novamente pode ajudar com esse problema.

**Problema potencial**: O campo para **ConfiguraçõesUrl** está vazio e o dispositivo não sincroniza. O usuário pode ter logado pela última vez no dispositivo antes do Enterprise State Roaming ser ativado no Portal do Diretório Ativo do Azure. Reinicie o dispositivo e faça o logon do usuário. Opcionalmente, no portal, tente fazer com que o administrador de TI navegue até **o Azure Active Directory** > **Devices** > Enterprise State**Roaming** desativar e reativar **Os usuários podem sincronizar configurações e dados do aplicativo entre dispositivos**. Uma vez reativado, reinicie o dispositivo e faça o login do usuário. Se isso não resolver o problema, **configuraçõesUrl** pode estar vazia se houver um certificado de dispositivo ruim. Nesse caso, executar “*dsregcmd.exe /leave*” em uma janela de prompt de comandos com privilégios elevados, reinicializar e tentar se registrar novamente pode ajudar com esse problema.

## <a name="enterprise-state-roaming-and-multi-factor-authentication"></a>Enterprise State Roaming e Autenticação Multifator 

Em determinadas condições, o Roaming de Estado de Empresa poderá não sincronizar dados se a Autenticação Multifator do Azure estiver configurada. Para obter mais informações sobre esses sintomas, consulte o documento de suporte [KB3193683](https://support.microsoft.com/kb/3193683). 

**Possível problema**: se o dispositivo estiver configurado para exigir Autenticação Multifator no portal do Azure Active Directory, você talvez não consiga sincronizar configurações ao entrar em um dispositivo com Windows 10 usando uma senha. Esse tipo de configuração de Autenticação Multifator destina-se a proteger uma conta de administrador do Azure. Os usuários administradores talvez ainda possam executar a sincronização entrando em seus dispositivos Windows 10 com o PIN do Microsoft Passport for Work ou concluindo a Autenticação Multifator durante o acesso a outros serviços do Azure, como o Office 365.

**Problema potencial**: O sincronização pode falhar se o admin configurar a política de acesso condicional de autenticação de vários fatores da Federação de Diretórioativo Ativo e o token de acesso no dispositivo expirar. Certifique-se de entrar e sair usando o PIN do Microsoft Passport for Work ou conclua a Autenticação Multifator durante o acesso a outros serviços do Azure, como o Office 365.

### <a name="event-viewer"></a>Visualizador de Eventos

Para a solução de problemas avançada, o Visualizador de Eventos pode ser usado para localizar erros específicos. Eles estão documentados na tabela abaixo. Os eventos podem ser encontrados em Event Viewer > Applications and Services Logs > **Microsoft** > **Windows** > **SettingSync-Azure** e para problemas relacionados à identidade com sincronização **Microsoft** > **Windows** > **AAD**.

## <a name="known-issues"></a>Problemas conhecidos

### <a name="sync-does-not-work-on-devices-that-have-apps-side-loaded-using-mdm-software"></a>A sincronização não funciona em dispositivos que possuem aplicativos com sideloaded usando o software do MDM

Afeta os dispositivos que executam a Atualização de Aniversário do Windows 10 (versão 1607). No Visualizador de Eventos, em logs do Azure SettingSync, a ID de evento 6013 com erro 80070259 é vista com frequência.

**Ação recomendada**  
Verifique se o cliente v1607 do Windows 10 tem a atualização cumulativa de 23 de agosto de 2016 ([KB3176934](https://support.microsoft.com/kb/3176934) compilação do sistema operacional 14393.82). 

---

### <a name="internet-explorer-favorites-do-not-sync"></a>Os favoritos do Internet Explorer não sincronizam

Afeta os dispositivos que executam a atualização de novembro do Windows 10 (versão 1511).

**Ação recomendada**  
Verifique se o cliente v1511 do Windows 10 tem a atualização cumulativa de julho de 2016 ([KB3172985](https://support.microsoft.com/kb/3172985) compilação do sistema operacional 10586.494).

---

### <a name="theme-is-not-syncing-as-well-as-data-protected-with-windows-information-protection"></a>O tema não está sincronizando, assim como os dados protegidos com a Proteção de Informações do Windows 

Para evitar o vazamento de dados, os dados protegidos pelo [Windows Information Protection](https://technet.microsoft.com/itpro/windows/keep-secure/protect-enterprise-data-using-wip) não serão sincronizados através do Enterprise State Roaming para dispositivos que usam a Atualização de Aniversário do Windows 10.

**Ação recomendada**  
Nenhum. Atualizações futuras do Windows poderão resolver esse problema.

---

### <a name="date-time-and-region-settings-do-not-sync-on-domain-joined-device"></a>As configurações de data, hora e região não sincronizam no dispositivo associado ao domínio 
  
Dispositivos associados ao domínio não passarão pela sincronização para a configuração de data, hora e região: horário automático. O uso do horário automático pode substituir as configurações de data, hora e região e fazer com que essas configurações não sincronizem. 

**Ação recomendada**  
Nenhum. 

---

### <a name="uac-prompts-when-syncing-passwords"></a>O UAC é ativado ao sincronizar senhas

Afeta os dispositivos que executam a atualização de novembro do Windows 10 (versão 1511) com uma NIC sem fio que está configurada para sincronizar senhas.

**Ação recomendada**  
Verifique se o cliente v1511 do Windows 10 tem a atualização cumulativa ([KB3140743](https://support.microsoft.com/kb/3140743) compilação do sistema operacional 10586.494).

---

### <a name="sync-does-not-work-on-devices-that-use-smart-card-for-login"></a>A sincronização não funciona em dispositivos que usam cartões inteligentes para login

Se você tentar entrar em seu dispositivo Windows usando um cartão inteligente ou um cartão inteligente Virtual, a sincronização das configurações vai parar de funcionar.     

**Ação recomendada**  
Nenhum. Atualizações futuras do Windows poderão resolver esse problema.

---

### <a name="domain-joined-device-is-not-syncing-after-leaving-corporate-network"></a>O dispositivo associado ao domínio não está sincronizando após deixar a rede corporativa     

Dispositivos associados ao domínio no AD do Azure podem apresentar falhas de sincronização se o dispositivo estiver fora por longos períodos de tempo, e podem não concluir a autenticação de domínio.

**Ação recomendada**  
Conecte o computador a uma rede corporativa para que a sincronização possa ser retomada.

---

### <a name="azure-ad-joined-device-is-not-syncing-and-the-user-has-a-mixed-case-user-principal-name"></a>O dispositivo Ingressado no Azure AD não está sincronizando e o usuário tem um nome UPN com letras maiúsculas e minúsculas.

Se o usuário tiver um UPN de caixa mista (por exemplo, UserName em vez de nome de usuário) e o usuário estiver em um dispositivo Azure AD Joined, que foi atualizado do Windows 10 Build 10586 a 14393, o dispositivo do usuário pode não sincronizar. 

**Ação recomendada**  
O usuário precisará remover o dispositivo do domínio e reingressá-lo na nuvem. Para isso, faça login como usuário administrador local e desconecte o dispositivo indo para O**Sistema** >  **de Configurações** > **Sobre** e selecione "Gerenciar ou desconectar-se do trabalho ou da escola". Limpe os arquivos abaixo e, em seguida, a azure AD Junte-se novamente ao dispositivo no**Sistema** >  **de Configurações** > **Sobre** e selecionando "Conectar-se ao trabalho ou à escola". Continue ingressando o dispositivo no Azure Active Directory e conclua o fluxo.

Na etapa de limpeza, limpe os seguintes arquivos:
- Settings.dat em `C:\Users\<Username>\AppData\Local\Packages\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\Settings\`
- Todos os arquivos na pasta `C:\Users\<Username>\AppData\Local\Packages\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\AC\TokenBroker\Account`

---

### <a name="event-id-6065-80070533-this-user-cant-sign-in-because-this-account-is-currently-disabled"></a>ID de evento 6065: 80070533 Este usuário não pode fazer login porque esta conta está desabilitada no momento  

No Visualizador de Eventos nos logs de SettingSync/Debug, esse erro pode ser visto quando as credenciais do usuário expiram. Além disso, ele pode ocorrer quando o locatário não provisiona o AzureRMS automaticamente. 

**Ação recomendada**  
No primeiro caso, solicite ao usuário que atualize suas credenciais e faça logon no dispositivo com as novas credenciais. Para resolver o problema do AzureRMS, continue com as etapas listadas em [KB3193791](https://support.microsoft.com/kb/3193791). 

---

### <a name="event-id-1098-error-0xcaa5001c-token-broker-operation-failed"></a>ID do evento 1098: Erro: Falha na operação do agente de Token 0xCAA5001C  

No Visualizador de Eventos, em logs do AAD/Operacionais, esse erro pode ser visto com o evento 1104: chamada do plugin do AP da nuvem AAD Obter erro retornado do token: 0xC000005F. Esse problema ocorre se estiverem faltando permissões ou atributos de propriedade.  

**Ação recomendada**  
Prossiga com as etapas listadas em [KB3196528](https://support.microsoft.com/kb/3196528).  

## <a name="next-steps"></a>Próximas etapas

Para uma visão geral, consulte [visão geral do estado corporativo](enterprise-state-roaming-overview.md).
