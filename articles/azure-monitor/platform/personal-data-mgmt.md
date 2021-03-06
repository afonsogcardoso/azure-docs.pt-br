---
title: Diretrizes para dados pessoais armazenados no Azure Log Analytics| Microsoft Docs
description: Este artigo descreve como gerenciar dados pessoais armazenados no Azure Log Analytics e os métodos para identificá-los e removê-los.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/18/2018
ms.openlocfilehash: a720627e1783d2e29ef180b7855132ea59444cab
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79248743"
---
# <a name="guidance-for-personal-data-stored-in-log-analytics-and-application-insights"></a>Diretrizes para dados pessoais armazenados no Log Analytics e no Application Insights

O Log Analytics é um armazenamento de dados no qual é possível localizar dados pessoais. O Application Insights armazena seus dados em uma partição do Log Analytics. Este artigo abordará onde, no Log Analytics e no Application Insights, esses dados normalmente estão localizados, bem como os recursos disponíveis para tratar os dados.

> [!NOTE]
> Para os fins deste artigo, _dados de log_ referem-se aos dados enviados para um espaço de trabalho do Log Analytics, enquanto _dados de aplicativo_ referem-se aos dados coletados pelo Application Insights.

[!INCLUDE [gdpr-dsr-and-stp-note](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="strategy-for-personal-data-handling"></a>Estratégia para tratamento de dados pessoais

Enquanto couber a você e à sua empresa determinar qual estratégia será adotada para o tratamento de dados privados (se for o caso), algumas abordagens possíveis são apresentadas a seguir. As abordagens estão listadas em ordem de preferência, do ponto de vista técnico, da mais para a menos preferencial:

* Sempre que possível, pare de coletar, ocultar, anonimizar ou ajustar os dados que estão sendo coletados para excluí-lo de ser considerado "privado". Isso é _sem dúvida_ a abordagem preferencial, evitando a necessidade de criar uma estratégia de tratamento de dados muito cara e impactante.
* Quando não for possível, tente normalizar os dados para reduzir o impacto na plataforma e no desempenho dos dados. Por exemplo, em vez de registrar uma ID de usuário explícita, crie um dado de pesquisa que correlacionará o nome de usuário e os respectivos detalhes a uma ID interna que poderá ser registrada em outro lugar. Dessa forma, se um dos usuários solicitar que você exclua as informações pessoais, é possível que somente excluir a linha na tabela de consulta correspondente ao usuário seja suficiente. 
* Por fim, se os dados privados precisarem ser coletados, compile um processo em torno do caminho de API de limpeza e do caminho de API de consulta existente para atender a qualquer obrigação necessária para exportar e excluir dados privados associados a um usuário. 

## <a name="where-to-look-for-private-data-in-log-analytics"></a>Onde pesquisa dados privados no Log Analytics?

O Log Analytics é um armazenamento flexível que, enquanto prescreve um esquema aos dados, permite substituir todos os campos por valores personalizados. Além disso, qualquer esquema personalizado pode ser processado. Assim sendo, é impossível dizer exatamente onde os dados privados estarão localizados no workspace específico. No entanto, os locais a seguir são bons pontos iniciais no inventário:

### <a name="log-data"></a>Dados de log

* *Endereços IP*: O Log Analytics coleta uma variedade de informações de IP em muitas tabelas diferentes. Por exemplo, a consulta a seguir mostra todas as tabelas nas quais os endereços IPv4 foram coletados nas últimas 24 horas:
    ```
    search * 
    | where * matches regex @'\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b' //RegEx originally provided on https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp
    | summarize count() by $table
    ```
* *IDs de usuário*: as IDs de usuário são localizadas em uma grande variedade de soluções e tabelas. É possível pesquisar um nome de usuário específico em todo o conjunto de dados usando o comando de pesquisa:
    ```
    search "[username goes here]"
    ```
  Lembre-se de procurar não apenas nomes de usuários legíveis, mas também GUIDs que podem ser rastreados diretamente para um usuário em particular!
* *IDs de dispositivo*: como as IDs de usuário, as IDs de dispositivo algumas vezes são consideradas "privadas". Use a mesma abordagem listada acima para IDs de usuário para identificar tabelas nas quais isso possa ser uma preocupação. 
* *Dados personalizados*: o Log Analytics permite a coleção em uma variedade de métodos: logs personalizados e campos personalizados, a [API do coletor de dados HTTP](../../azure-monitor/platform/data-collector-api.md) e dados personalizados coletados como parte dos logs de eventos do sistema. Todos são suscetíveis a conter dados privados e devem ser examinados para verificar se esses dados existem.
* *Dados capturados pela solução*: como o mecanismo da solução é aberto, é recomendável revisar todas as tabelas geradas pelas soluções para garantir a conformidade.

### <a name="application-data"></a>Dados do aplicativo

* *Endereços IP*: enquanto o Application Insights ofuscará, por padrão, todos os campos de endereço IP para "0.0.0.0", é um padrão bastante comum substituir esse valor pelo IP do usuário atual para manter as informações de sessão. A consulta de análise abaixo pode ser usada para encontrar qualquer tabela que contém os valores na coluna de endereço IP diferentes de "0.0.0.0" nas últimas 24 horas:
    ```
    search client_IP != "0.0.0.0"
    | where timestamp > ago(1d)
    | summarize numNonObfuscatedIPs_24h = count() by $table
    ```
* *IDs de usuário*: por padrão, o Application Insights usará IDs geradas aleatoriamente para acompanhamento de usuário e de sessão. No entanto, é comum substituir esses campos para armazenar uma ID mais relevante para o aplicativo. Por exemplo: nomes de usuário, GUIDs AAD, etc. Esses IDs são muitas vezes considerados no escopo como dados pessoais e, portanto, devem ser tratados adequadamente. Nossa recomendação é sempre tentar ofuscar essas IDs ou torná-las anônimas. Os campos em que esses valores são geralmente encontrados incluem session_Id, user_Id, user_AuthenticatedId, user_AccountId, bem como customDimensions.
* *Dados personalizados*: o Application Insights permite acrescentar um conjunto de dimensões personalizadas para qualquer tipo de dados. Essas dimensões podem ser *quaisquer* dados. Use a consulta a seguir para identificar quaisquer dimensões personalizadas coletadas nas últimas 24 horas:
    ```
    search * 
    | where isnotempty(customDimensions)
    | where timestamp > ago(1d)
    | project $table, timestamp, name, customDimensions 
    ```
* *Dados na memória e em trânsito*: o Application Insights acompanhará solicitações, exceções, chamadas de dependência e rastreamentos. Dados privados podem frequentemente ser coletados no código e no nível de chamada HTTP. Examine as exceções, solicitações, dependências e tabelas de rastreamento para identificar quaisquer dados desse tipo. Use [inicializadores de telemetria](https://docs.microsoft.com/azure/application-insights/app-insights-api-filtering-sampling) sempre que possível para ofuscar esses dados.
* *Capturas do Depurador de Instantâneos*: o recurso [Depurador de Instantâneos](https://docs.microsoft.com/azure/application-insights/app-insights-snapshot-debugger) do Application Insights permite coletar instantâneos de depuração, sempre que uma exceção é detectada na instância de produção do seu aplicativo. Instantâneos vão expor o rastreamento de pilha completo levando às exceções, bem como os valores de variáveis locais em cada etapa na pilha. Infelizmente, esse recurso não permite a exclusão seletiva de pontos de alinhamento, nem o acesso programático aos dados no instantâneo. Portanto, se a taxa de retenção de instantâneo padrão não atende a seus requisitos de conformidade, a recomendação é desativar o recurso.

## <a name="how-to-export-and-delete-private-data"></a>Como exportar e excluir dados privados

Conforme mencionado na seção [Estratégia para tratamento de dados pessoais](#strategy-for-personal-data-handling) anterior, é __altamente__ recomendável, se possível, reestruturar a política de coleta de dados para desabilitar a coleta de dados privados, ofuscando ou anonimizando-os, ou modificando-os de alguma outra forma, para evitar que sejam considerados "privados". O tratamento dos dados resultará, principalmente, em custos para você e sua equipe para definir e automatizar uma estratégia, compilar uma interface para seus clientes interagirem com os dados, e custos de manutenção contínuos. Além disso, é computacionalmente exigente para o Log Analytics e o Application Insights, e um grande volume de consultas simultâneas ou chamadas de API de limpeza têm o potencial de impactar negativamente todas as outras interações com a funcionalidade do Log Analytics. Dito isso, há de fato alguns cenários válidos em que dados privados devem ser coletados. Para esses casos, os dados devem ser tratados conforme descrito nesta seção.

[!INCLUDE [gdpr-intro-sentence](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-and-export"></a>Exibir e exportar

Para exibir e exportar solicitações de dados, a [API de consulta do Log Analytics](https://dev.loganalytics.io/) ou a [API de consulta do Application Insights](https://dev.applicationinsights.io/quickstart) devem ser usadas. A lógica para converter a forma dos dados em uma forma apropriada para entregar aos usuários será de sua responsabilidade. [Azure Functions](https://azure.microsoft.com/services/functions/) é um ótimo local para hospedar essa lógica.

> [!IMPORTANT]
>  Embora a grande maioria das operações de expurgo possa ser concluída muito mais rapidamente do que o SLA, **o SLA formal para a conclusão das operações de purga é definido em 30 dias** devido ao seu forte impacto na plataforma de dados utilizada. Este é um processo automatizado; não há como solicitar que uma operação seja tratada mais rapidamente.

### <a name="delete"></a>Excluir

> [!WARNING]
> As exclusões no Log Analytics são destrutivas e irreversíveis! Tenha extremo cuidado na execução delas.

Disponibilizamos como parte de uma privacidade que trata um caminho de API de *limpeza*. Esse caminho deve ser utilizado com moderação devido ao risco associado, o potencial impacto no desempenho e o potencial para distorcer agregações, medidas e outros aspectos dos dados do Log Analytics. Consulte a seção [Estratégia para tratamento de dados pessoais](#strategy-for-personal-data-handling) para obter abordagens alternativas para tratar dados privados.

A limpeza é uma operação altamente privilegiada que nenhum aplicativo ou usuário no Azure (incluindo até mesmo o proprietário do recurso) terá permissões para executar sem que seja concedida explicitamente uma função no Azure Resource Manager. Essa função é _Limpador de Dados_ e deve ser cuidadosamente delegada devido ao potencial de perda de dados. 

> [!IMPORTANT]
> Para gerenciar os recursos do sistema, as solicitações de purga são estranguladas a 50 solicitações por hora. Você deve emuar a execução de solicitações de expurgo enviando um único comando cujo predicado inclui todas as identidades de usuário que requerem purgação. Use o [operador in](/azure/kusto/query/inoperator) para especificar várias identidades. Você deve executar a consulta antes de executar a solicitação de expurgo para verificar se os resultados são esperados. 



Depois que a função do Azure Resource Manager for atribuída, dois novos caminhos de API estarão disponíveis: 

#### <a name="log-data"></a>Dados de log

* [Limpeza POST](https://docs.microsoft.com/rest/api/loganalytics/workspaces%202015-03-20/purge) – obtém um objeto especificando parâmetros de dados para excluir e retorna um GUID de referência 
* Status de limpeza GET – a chamada de limpeza POST retornará um cabeçalho 'x-ms-status-location' que incluirá uma URL que você poderá chamar para determinar o status da API de limpeza. Por exemplo: 

    ```
    x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/Microsoft.OperationalInsights/workspaces/[WorkspaceName]/operations/purge-[PurgeOperationId]?api-version=2015-03-20
    ```

> [!IMPORTANT]
>  Enquanto esperamos que a grande maioria das operações de expurgo completem muito mais rápido do que o nosso SLA, devido ao seu forte impacto na plataforma de dados usada pela Log Analytics, **o SLA formal para a conclusão das operações de purga é definido em 30 dias**. 

#### <a name="application-data"></a>Dados do aplicativo

* [Limpeza POST](https://docs.microsoft.com/rest/api/application-insights/components/purge) – obtém um objeto especificando parâmetros de dados para excluir e retorna um GUID de referência
* Status de limpeza GET – a chamada de limpeza POST retornará um cabeçalho 'x-ms-status-location' que incluirá uma URL que você poderá chamar para determinar o status da API de limpeza. Por exemplo: 

   ```
   x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/microsoft.insights/components/[ComponentName]/operations/purge-[PurgeOperationId]?api-version=2015-05-01
   ```

> [!IMPORTANT]
>  Embora a grande maioria das operações de expurgo possa ser concluída muito mais rapidamente do que o SLA, devido ao seu forte impacto na plataforma de dados usada pelo Application Insights, **o SLA formal para a conclusão das operações de expurgo é definido em 30 dias**.

## <a name="next-steps"></a>Próximas etapas
- Para saber mais sobre como os dados do Log Analytcs são coletados, processados e protegidos, veja [Segurança de dados do Log Analytics](../../azure-monitor/platform/data-security.md).
- Para saber mais sobre como os dados do Application Insights são coletados, processados e protegidos, confira [Segurança de dados do Application Insights](../../azure-monitor/app/data-retention-privacy.md).
