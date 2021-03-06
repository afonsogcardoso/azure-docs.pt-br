---
title: Envie alertas de saúde do serviço Azure com opsGenie usando webhooks
description: Obtenha notificações personalizadas sobre eventos de integridade do serviço na instância do OpsGenie.
ms.topic: conceptual
ms.date: 06/10/2019
ms.openlocfilehash: def12d5e7b1b93b8370cd7be61538fca53531ae1
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77654130"
---
# <a name="send-azure-service-health-alerts-with-opsgenie-using-webhooks"></a>Envie alertas de saúde do serviço Azure com opsGenie usando webhooks

Este artigo mostra como configurar alertas de integridade do serviço do Azure com o OpsGenie usando um webhook. Usando a Integração de Integridade do Serviço do Azure do [OpsGenie](https://www.opsgenie.com/), você pode encaminhar alertas de Integridade do Serviço do Azure para o OpsGenie. O OpsGenie pode determinar as pessoas certas a serem notificadas com base em plantões, usando email, mensagens de texto (SMS), chamadas telefônicas, notificações por push do iOS e Android e escalonando alertas até que o alerta seja confirmado ou fechado.

## <a name="creating-a-service-health-integration-url-in-opsgenie"></a>Criando uma URL de integração de integridade do serviço no OpsGenie
1.  Verifique se você se inscreveu e entrou em sua conta do [OpsGenie](https://www.opsgenie.com/).

1.  Navegue para a seção **Integrações** no OpsGenie.

    ![A seção “Integração” do OpsGenie](./media/webhook-alerts/opsgenie-integrations-section.png)

1.  Selecione botão de integração **Integridade do Serviço do Azure**.

    ![O “botão Integridade do Serviço do Azure” do OpsGenie](./media/webhook-alerts/opsgenie-azureservicehealth-button.png)

1.  **Nomeie** o alerta e especifique o campo **Atribuído à Equipe**.

1.  Preencha os outros campos como **Destinatários**, **Habilitado** e **Suprimir Notificações**.

1.  Copie e salve a **URL de Integração**, que já deve conter a `apiKey` acrescentada ao final.

    ![A “URL de Integração” do OpsGenie](./media/webhook-alerts/opsgenie-integration-url.png)

1.  Selecione **Salvar Integração**

## <a name="create-an-alert-using-opsgenie-in-the-azure-portal"></a>Criar um alerta usando o OpsGenie no portal do Azure
### <a name="for-a-new-action-group"></a>Para um novo grupo de ação:
1. Siga as etapas 1 a 8 de [Criar um alerta em uma notificação de integridade do serviço para um novo grupo de ação usando o portal do Azure](../azure-monitor/platform/alerts-activity-log-service-notifications.md).

1. Defina na lista de **Ações**:

    a. **Tipo de Ação:** *Webhook*

    b. **Detalhes:** a **URL de Integração** do OpsGenie salva anteriormente.

    c. **Nome:** o nome, alias ou identificador do webhook.

1. Selecione **Salvar** quando concluir a criação do alerta.

### <a name="for-an-existing-action-group"></a>Para um grupo de ação existente:
1. No [portal do Azure](https://portal.azure.com/), selecione **Monitor**.

1. Na seção **Configurações**, selecione **Grupos de ação**.

1. Encontre e selecione o grupo de ação que você deseja editar.

1. Adicione à lista de **Ações**:

    a. **Tipo de Ação:** *Webhook*

    b. **Detalhes:** a **URL de Integração** do OpsGenie salva anteriormente.

    c. **Nome:** o nome, alias ou identificador do webhook.

1. Selecione **Salvar** quando concluir a atualização do grupo de ação.

## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>Testando a integração de webhook por meio de uma solicitação HTTP POST
1. Crie o conteúdo de integridade do serviço que você deseja enviar. Você pode encontrar um exemplo de carga útil de webhook webhook em [Webhooks para alertas de registro de atividade do Azure](../azure-monitor/platform/activity-log-alerts-webhook.md).

1. Crie uma solicitação HTTP POST, da seguinte maneira:

    ```
    POST        https://api.opsgenie.com/v1/json/azureservicehealth?apiKey=<APIKEY>

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
1. Você deverá receber uma resposta `200 OK` com a mensagem de status “êxito”.

1. Acesse o [OpsGenie](https://www.opsgenie.com/) para confirmar se a integração foi configurada com êxito.

## <a name="next-steps"></a>Próximas etapas
- Saiba como [configurar notificações de webhook para sistemas de gerenciamento de problemas existentes.](service-health-alert-webhook-guide.md)
- Revise o [esquema de webhook](../azure-monitor/platform/activity-log-alerts-webhook.md)de alerta de log de atividade . 
- Conheça as [notificações de saúde dos serviços](../azure-monitor/platform/service-notifications.md).
- Saiba mais sobre [grupos de ação](../azure-monitor/platform/action-groups.md).