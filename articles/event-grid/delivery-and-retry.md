---
title: Entrega e repetição da Grade de Eventos do Azure
description: Descreve como a Grade de Eventos do Azure entrega eventos e como ela trata mensagens não entregues.
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: conceptual
ms.date: 02/27/2020
ms.author: spelluru
ms.openlocfilehash: dda2fd98c4c0d330059156a5ec00baa97ffaf627
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77921055"
---
# <a name="event-grid-message-delivery-and-retry"></a>Entrega e repetição de mensagens da Grade de Eventos

Este artigo descreve como a Grade de Eventos do Azure manipula eventos quando a entrega não é confirmada.

A entrega proporcionada pela Grade de Eventos tem um tempo de duração. Cada mensagem é entregue pelo menos uma vez para cada assinatura. Os eventos são enviados para o ponto de extremidade registrado de cada assinatura imediatamente. Se um ponto de extremidade não confirmar o recebimento de um evento, a Grade de Eventos tentará entregá-lo novamente.

## <a name="batched-event-delivery"></a>Entrega de eventos em lote

Event Grid não envia cada evento individualmente aos assinantes. O assinante recebe uma matriz com um único evento. Você pode configurar event grid para lotes de eventos para entrega para melhor desempenho HTTP em cenários de alto rendimento.

A entrega em lote tem duas configurações:

* **Eventos máximos por lote** - O número máximo de eventos que a Event Grid entregará por lote. Esse número nunca será ultrapassado, porém menos eventos podem ser entregues se nenhum outro evento estiver disponível no momento da publicação. Event Grid não atrasa eventos para criar um lote se houver menos eventos disponíveis. Deve estar entre 1 e 5.000.
* **Tamanho do lote preferido em kilobytes** - Teto alvo para tamanho de lote em kilobytes. Semelhante aos eventos máximos, o tamanho do lote pode ser menor se mais eventos não estiverem disponíveis no momento da publicação. É possível que um lote seja maior do que o tamanho do lote preferido *se* um único evento for maior do que o tamanho preferido. Por exemplo, se o tamanho preferido for de 4 KB e um evento de 10 KB for empurrado para event grid, o evento de 10 KB ainda será entregue em seu próprio lote em vez de ser descartado.

Entrega em lote configurada por assinatura por evento através do portal, CLI, PowerShell ou SDKs.

### <a name="azure-portal"></a>Portal do Azure: 
![Configurações de entrega em lote](./media/delivery-and-retry/batch-settings.png)

### <a name="azure-cli"></a>CLI do Azure
Ao criar uma assinatura de evento, use os seguintes parâmetros: 

- **max-eventos por lote** - Número máximo de eventos em um lote. Deve ser um número entre 1 e 5000.
- **tamanho preferencial-lote-em-kilobytes** - Tamanho de lote preferido em kilobytes. Deve ser um número entre 1 e 1024.

```azurecli
storageid=$(az storage account show --name <storage_account_name> --resource-group <resource_group_name> --query id --output tsv)
endpoint=https://$sitename.azurewebsites.net/api/updates

az eventgrid event-subscription create \
  --resource-id $storageid \
  --name <event_subscription_name> \
  --endpoint $endpoint \
  --max-events-per-batch 1000 \
  --preferred-batch-size-in-kilobytes 512
```

Para obter mais informações sobre como usar o Azure CLI com event grid, consulte [eventos de armazenamento de rota para o ponto final da Web com o Azure CLI](../storage/blobs/storage-blob-event-quickstart.md).

## <a name="retry-schedule-and-duration"></a>Agendamento de nova tentativa e duração

Event Grid espera 30 segundos por uma resposta após entregar uma mensagem. Após 30 segundos, se o ponto final não tiver respondido, a mensagem será enfileirada para nova tentativa. A Grade de Eventos usa uma política de repetição de retirada exponencial para a entrega de eventos. Event Grid tenta a entrega no seguinte cronograma em uma base de esforço melhor:

- 10 segundos
- 30 segundos
- 1 minuto
- 5 minutos
- 10 minutos
- 30 minutos
- 1 hora
- Por hora por até 24 horas

Se o ponto final responder dentro de 3 minutos, event grid tentará remover o evento da fila de retentativa em uma base de esforço melhor, mas as duplicatas ainda podem ser recebidas.

Event Grid adiciona uma pequena randomização a todas as etapas de repetição e pode pular oportunicamente certas repetições se um ponto final for consistentemente insalubre, para baixo por um longo período, ou parece estar sobrecarregado.

Para comportamento determinístico, defina o tempo de evento para viver e max as tentativas de entrega nas [políticas de repetição de assinatura](manage-event-delivery.md).

Por padrão, a Grade de Eventos expira todos os eventos que não são entregues em 24 horas. Você pode [personalizar a política de repetição](manage-event-delivery.md) ao criar uma assinatura de evento. Forneça o número máximo de tentativas de entrega (o padrão é 30) e a vida útil do evento (o padrão é 1440 minutos).

## <a name="delayed-delivery"></a>Entrega atrasada

Como um ponto final experimenta falhas de entrega, event grid começará a atrasar a entrega e a repetição de eventos para esse ponto final. Por exemplo, se os primeiros 10 eventos publicados para um ponto final falharem, event grid assumirá que o ponto final está enfrentando problemas e atrasará todas as repetições subseqüentes *e novas* entregas por algum tempo - em alguns casos até várias horas.

O objetivo funcional da entrega atrasada é proteger pontos finais insalubres, bem como o sistema Event Grid. Sem o back-off e o atraso na entrega para pontos finais insalubres, a política de retentativa e os recursos de volume da Event Grid podem facilmente sobrecarregar um sistema.

## <a name="dead-letter-events"></a>Eventos de mensagens mortas

Quando a Grade de Eventos não pode fornecer um evento, ela pode enviar o evento não entregue para uma conta de armazenamento. Este processo é conhecido como armazenamento de mensagens mortas. Por padrão, a Grade de Eventos não ativa o armazenamento de mensagens mortas. Para habilitá-lo, você deve especificar uma conta de armazenamento para reter eventos que não foram entregues ao criar a assinatura do evento. Você aciona eventos dessa conta de armazenamento para resolver as entregas.

A Grade de Eventos enviará um evento ao local de mensagens mortas quando ela tiver tentado todas as suas tentativas de repetição. Se a Grade de Eventos receber um código de resposta 400 (Solicitação incorreta) ou 413 (A entidade da solicitação é grande demais), ela enviará o evento imediatamente ao ponto de extremidade de mensagens mortas. Esses códigos de resposta indicam que a entrega do evento nunca terá êxito.

Há um atraso de cinco minutos entre a última tentativa de entregar de um evento e quando ela é entregue para o local de inatividade. Esse atraso tem como objetivo reduzir o número de operações de armazenamento Blob. Se o local de mensagens mortas não estiver disponível por quatro horas, o evento será descartado.

Antes de configurar o local de mensagens mortas, você deve ter uma conta de armazenamento com um contêiner. É possível fornecer o ponto de extremidade para esse contêiner ao criar a assinatura do evento. O ponto de extremidade está no formato de: `/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>/blobServices/default/containers/<container-name>`

Talvez você queira ser notificado quando um evento tiver sido enviado para o local de mensagens mortas. Para usar a Grade de Eventos para responder a eventos não entregues, [crie uma assinatura de evento](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) para o armazenamento de blob de mensagens mortas. Toda vez que seu armazenamento de blob de mensagens mortas recebe um evento não entregue, a Grade de Eventos notifica o manipulador. O manipulador responde com ações que você deseja executar para reconciliar eventos não entregues.

Para ver um exemplo de como configurar um local de mensagens mortas, confira [Dead letter and retry policies](manage-event-delivery.md) (Políticas de mensagens mortas e repetição).

## <a name="message-delivery-status"></a>Status de entrega da mensagem

A Grade de Eventos usa códigos de resposta HTTP para confirmar o recebimento de eventos. 

### <a name="success-codes"></a>Códigos de êxito

A Event Grid considera **apenas** os seguintes códigos de resposta HTTP como entregas bem-sucedidas. Todos os outros códigos de status são considerados entregas falhadas e serão julgados ou cancelados conforme apropriado. Ao receber um código de status bem-sucedido, event grid considera a entrega concluída.

- 200 OK
- 201 Criado
- 202 Aceito
- 203 Informações Não Autorizadas
- 204 Sem Conteúdo

### <a name="failure-codes"></a>Códigos de falha

Todos os outros códigos que não estão no conjunto acima (200-204) são considerados falhas e serão julgados novamente. Alguns têm políticas específicas de repetição ligadas a eles descritas abaixo, todas as outras seguem o modelo padrão exponencial de retrocesso. É importante ter em mente que, devido à natureza altamente paraleleda da arquitetura da Event Grid, o comportamento de repetição não é determinista. 

| Código de status | Tentar comportamento novamente |
| ------------|----------------|
| 400 Solicitação Inválida | Tente novamente após 5 minutos ou mais (Deadletter imediatamente se deadletter configuração) |
| 401 Não Autorizado | Tente novamente após 5 minutos ou mais |
| 403 Proibido | Tente novamente após 5 minutos ou mais |
| 404 Não Encontrado | Tente novamente após 5 minutos ou mais |
| 408 Tempo Limite da Solicitação | Tente novamente após 2 minutos ou mais |
| Solicitação 413 entidade muito grande | Tente novamente após 10 segundos ou mais (Deadletter imediatamente se deadletter configuração) |
| 503 Serviço Indisponível | Tente novamente após 30 segundos ou mais |
| Todos os outros | Tente novamente após 10 segundos ou mais |


## <a name="next-steps"></a>Próximas etapas

* Para exibir o status de entregas de evento, consulte [Entrega de mensagens da Grade de Eventos do Monitor](monitor-event-delivery.md).
* Para personalizar as opções de entrega de eventos, confira [Dead letter and retry policies](manage-event-delivery.md) (Políticas de mensagens mortas e repetição).
