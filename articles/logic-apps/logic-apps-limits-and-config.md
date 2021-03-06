---
title: Limites e configuração
description: Limites de serviço, como duração, throughput e capacidade, além de valores de configuração, como endereços IP para permitir, para aplicativos azure logic
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 03/12/2020
ms.openlocfilehash: 418be090e7ff78ec0089c115c9884ffeffdda871
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79284012"
---
# <a name="limits-and-configuration-information-for-azure-logic-apps"></a>Informações de limites e configuração para os Aplicativos Lógicos do Azure

Este artigo descreve os limites e os detalhes de configuração para criar e executar fluxos de trabalho automatizados com os Aplicativos Lógicos do Azure. Para automatizar o poder, consulte [Limites e configuração no Power Automate](https://docs.microsoft.com/flow/limits-and-config).

<a name="definition-limits"></a>

## <a name="definition-limits"></a>Limites de definição

Estes são os limites de definição de um único aplicativo lógico:

| Nome | Limite | Observações |
| ---- | ----- | ----- |
| Ações por fluxo de trabalho | 500 | Para estender esse limite, adicione fluxos de trabalho aninhados conforme necessário. |
| Profundidade de aninhamento permitido para ações | 8 | Para estender esse limite, adicione fluxos de trabalho aninhados conforme necessário. |
| Fluxos de trabalho por região e assinatura | 1,000 | |
| Gatilhos por fluxo de trabalho | 10 | Ao trabalhar no modo de exibição de código, não no designer |
| Limite de casos de escopo do comutador | 25 | |
| Variáveis por fluxo de trabalho | 250 | |
| Caracteres por expressão | 8.192 | |
| Tamanho máximo para `trackedProperties` | 16.000 caracteres |
| Nome para `action` ou `trigger` | 80 caracteres | |
| Comprimento de `description` | 256 caracteres | |
| Máximo `parameters` | 50 | |
| Máximo `outputs` | 10 | |
||||

<a name="run-duration-retention-limits"></a>

## <a name="run-duration-and-retention-limits"></a>Limites de retenção e duração da execução

Estes são os limites de execução de um único aplicativo lógico:

| Nome | Limite de vários inquilinos | Limite do ambiente de serviço de integração | Observações |
|------|--------------------|---------------------------------------|-------|
| Duração da execução | 90 dias | 366 dias | A duração da execução é calculada usando o tempo de início de uma execução e o limite especificado *no momento* de início pela configuração do fluxo de trabalho, [**Executar retenção de histórico em dias**](#change-duration). <p><p>Para alterar o limite padrão, que é de 90 dias, consulte [a duração da execução](#change-duration)da alteração . |
| Executar retenção no armazenamento | 90 dias | 366 dias | A retenção de execução é calculada usando o tempo de início de uma execução e o limite especificado *no momento atual* pela configuração do fluxo de trabalho, Executar [**retenção de histórico em dias**](#change-retention). Se uma corrida é concluída ou cronometrada, o cálculo de retenção sempre usa o tempo de início da execução. Quando a duração de uma execução excede o limite de retenção *atual,* a execução é removida do histórico de execuçãos. <p><p>Se você alterar essa configuração, o limite atual é sempre usado para calcular a retenção, não importa o limite anterior. Por exemplo, se você reduzir o limite de retenção de 90 dias para 30 dias, uma corrida de 60 dias é removida do histórico de execuçãos. Se você aumentar o período de retenção de 30 dias para 60 dias, uma corrida que é de 20 dias de idade permanece na história da corrida por mais 40 dias. <p><p>Para alterar o limite padrão, que é de 90 dias, consulte [alterar a retenção de execução no armazenamento](#change-retention). |
| Intervalo de recorrência mínimo | 1 segundo | 1 segundo ||
| Intervalo de recorrência máximo | 500 dias | 500 dias ||
|||||

<a name="change-duration"></a>
<a name="change-retention"></a>

### <a name="change-run-duration-and-run-retention-in-storage"></a>Alterar a duração da execução e executar a retenção no armazenamento

Para alterar o limite padrão para duração da execução e executar a retenção no armazenamento, siga estas etapas. Para aumentar o limite máximo, [entre em contato com a equipe da Logic Apps](mailto://logicappsemail@microsoft.com) para obter ajuda com suas necessidades.

> [!NOTE]
> Para aplicativos lógicos no Azure multi-inquilino, o limite padrão de 90 dias é o mesmo que o limite máximo. Você só pode diminuir esse valor.
> Para aplicativos lógicos em um ambiente de serviço de integração, você pode diminuir ou aumentar o limite de padrão de 90 dias.

1. Vá para o [portal Azure.](https://portal.azure.com) Na caixa de pesquisa do portal, encontre e selecione **aplicativos Logic**.

1. Selecione e abra seu aplicativo lógico no Logic App Designer.

1. No menu do aplicativo lógico, selecione **configurações de fluxo de trabalho**.

1. Em **opções de tempo de execução,** na **lista executar o histórico em dias,** selecione **Personalizado**.

1. Arraste o controle deslizante para alterar o número de dias que você deseja.

1. Quando terminar, na barra de ferramentas de **configurações do fluxo de trabalho,** **selecione Salvar**.

<a name="looping-debatching-limits"></a>

## <a name="concurrency-looping-and-debatching-limits"></a>Simultaneidade, um loop e debatching de limites

Estes são os limites de execução de um único aplicativo lógico:

| Nome | Limite | Observações |
| ---- | ----- | ----- |
| Simultaneidade do gatilho | - Ilimitado quando o controle de simultuário é desligado <p><p>- 25 é o limite padrão quando o controle de simultuário é ligado, o que não pode ser desfeito depois que você liga o controle. Você pode alterar o padrão para um valor entre 1 e 50, inclusive. | Esse limite descreve o número mais alto de instâncias de aplicativo lógico que podem ser executados ao mesmo tempo, ou em paralelo. <p><p>**Nota:** Quando a simulta é ligada, o limite SplitOn é reduzido para 100 itens para [matrizes de desbatching](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch). <p><p>Para alterar o limite padrão para um valor entre 1 e 50, inclusive, consulte [Alterar o limite de simultaneidade do gatilho](../logic-apps/logic-apps-workflow-actions-triggers.md#change-trigger-concurrency) ou [Disparar instâncias sequencialmente](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-trigger). |
| Execuções de espera máximo | - Sem concorrência, o número mínimo de corridas de espera é 1, enquanto o número máximo é de 50. <p><p>- Com a concorrência, o número mínimo de corridas de espera é de 10 mais o número de corridas simultâneas (gatilho de concorrência). Você pode alterar o número máximo até 100, inclusive. | Esse limite descreve o maior número de instâncias do aplicativo lógico que pode aguardar para ser executado quando o aplicativo lógico já está em execução o número máximo de instâncias simultâneo. <p><p>Para alterar o limite padrão, consulte [execuções de espera da alteração limitam](../logic-apps/logic-apps-workflow-actions-triggers.md#change-waiting-runs). |
| Itens da matriz de foreach | 100.000 | Esse limite descreve o maior número de itens de matriz que um loop "para cada" pode processar. <p><p>Para filtrar matrizes maiores, você pode usar o [ação de consulta](logic-apps-perform-data-operations.md#filter-array-action). |
| Simultaneidade de foreach | 20 é o limite padrão quando o controle de simultaneidade é desativado. Você pode alterar o padrão para um valor entre 1 e 50, inclusive. | Esse limite é o maior número de iterações de loop "for each" que podem ser executadas ao mesmo tempo ou em paralelo. <p><p>Para alterar o limite padrão para um valor entre 1 e 50 inclusive, consulte [Alterar o limite de simultaneidade “para cada”](../logic-apps/logic-apps-workflow-actions-triggers.md#change-for-each-concurrency) ou [Executar loops "para cada" sequencialmente](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-for-each). |
| Itens SplitOn | - 100.000 sem contra-corrente de gatilho <p><p>- 100 com a concorrência do gatilho | Para gatilhos que retornam uma matriz, você pode especificar uma expressão que usa uma propriedade 'SplitOn' que [divide ou retira de lote os itens da matriz em várias instâncias de fluxo de trabalho](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch) para processamento, em vez de usar um loop "Foreach". Essa expressão referencia a matriz a ser usada para criar e executar uma instância de fluxo de trabalho para cada item da matriz. <p><p>**Nota:** Quando a simulta é ligada, o limite SplitOn é reduzido para 100 itens. |
| Iterações Until | - Padrão: 60 <p><p>- Máximo: 5.000 | |
||||

<a name="throughput-limits"></a>

## <a name="throughput-limits"></a>Limites de taxa de transferência

Estes são os limites de definição de um único aplicativo lógico:

### <a name="multi-tenant-logic-apps-service"></a>Serviço de aplicativos lógicos multilocatários

| Nome | Limite | Observações |
| ---- | ----- | ----- |
| Ação: Execuções a cada 5 minutos | 100.000 é o limite padrão, mas 300.000 é o limite máximo. | Para alterar o limite padrão, consulte [Execute seu aplicativo lógico no modo "alto rendimento"](../logic-apps/logic-apps-workflow-actions-triggers.md#run-high-throughput-mode), que está na visualização. Ou, você pode distribuir a carga de trabalho entre mais de um aplicativo lógico conforme necessário. |
| Ação: Saída de chamadas simultâneas | ~2.500 | Você pode reduzir o número de solicitações simultâneas ou reduzir a duração conforme necessário. |
| Ponto de extremidade de runtime: chamadas de entrada simultâneas | ~1,000 | Você pode reduzir o número de solicitações simultâneas ou reduzir a duração conforme necessário. |
| Ponto de extremidade de runtime: lê chamadas por 5 minutos  | 60.000 | Você pode distribuir a carga de trabalho entre mais de um aplicativo, conforme necessário. |
| Ponto de extremidade de runtime: invoca chamadas por 5 minutos | 45,000 | Você pode distribuir a carga de trabalho entre mais de um aplicativo, conforme necessário. |
| Taxa de transferência de conteúdo por 5 minutos | 600 MB | Você pode distribuir a carga de trabalho entre mais de um aplicativo, conforme necessário. |
||||

### <a name="integration-service-environment-ise"></a>Ambiente de serviço de integração (ISE)

Aqui estão os limites de throughput para o SKU Premium:

| Nome | Limite | Observações |
|------|-------|-------|
| Limite de execução da unidade base | Sistema estrangulado quando a capacidade da infra-estrutura atinge 80% | Fornece ~4.000 execuções de ação por minuto, o que equivale a ~160 milhões de execuções de ação por mês | |
| Limite de execução da unidade de escala | Sistema estrangulado quando a capacidade da infra-estrutura atinge 80% | Cada unidade de escala pode fornecer ~2.000 execuções de ação adicionais por minuto, o que é ~80 milhões de execuções de ação por mês | |
| Unidades de escala máxima que você pode adicionar | 10 | |
||||

Para ultrapassar esses limites no processamento normal ou executar um teste de carga que possa ultrapassar esses limites, [entre em contato com a equipe de Aplicativos Lógicos](mailto://logicappsemail@microsoft.com) para que possam ajudá-lo com suas necessidades.

> [!NOTE]
> O [SKU do desenvolvedor](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) não tem limites publicados, pois este SKU não tem nenhum contrato de nível de serviço (SLA) ou recursos para o dimensionamento.
> Use este SKU apenas para experimentos, desenvolvimento e testes, não testes de produção ou desempenho.

<a name="gateway-limits"></a>

## <a name="gateway-limits"></a>Limites de gateway

O Azure Logic Apps suporta operações de gravação, incluindo inserções e atualizações, através do gateway. No entanto, essas operações têm [limites em seu tamanho de carga.](https://docs.microsoft.com/data-integration/gateway/service-gateway-onprem#considerations)

<a name="request-limits"></a>

## <a name="http-limits"></a>Limites do HTTP

Aqui estão os limites para uma única chamada HTTP de saída ou de entrada:

#### <a name="timeout"></a>Tempo limite

Algumas operações de conector fazem chamadas assíncronas ou escutam solicitações de webhook, portanto o tempo limite para essas operações pode ter mais do que esses limites. Para obter mais informações, consulte os detalhes técnicos para o conector específico e também [Gatilhos e ações de fluxo de trabalho](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action).

| Nome | Limite de vários inquilinos | Limite do ambiente de serviço de integração | Observações |
|------|--------------------|---------------------------------------|-------|
| Solicitação de saída | 120 segundos <br>(2 minutos) | 240 segundos <br>(4 minutos) | Exemplos de solicitações de saída incluem chamadas feitas por gatilhos HTTP. <p><p>**Dica:** Para operações mais longas, use um [padrão de votação assíncrono](../logic-apps/logic-apps-create-api-app.md#async-pattern) ou um [loop de até .](../logic-apps/logic-apps-workflow-actions-triggers.md#until-action) |
| Solicitação de entrada | 120 segundos <br>(2 minutos) | 240 segundos <br>(4 minutos) | Exemplos de solicitações de entrada incluem chamadas recebidas por gatilhos de solicitação e gatilhos de webhook. <p><p>**Nota**: Para que o chamador original obtenha a resposta, todas as etapas da resposta devem ser concluídas dentro do limite, a menos que você chame outro aplicativo lógico como um fluxo de trabalho aninhado. Para obter mais informações, consulte [Chamar, acionar ou aninhar aplicativos lógicos](../logic-apps/logic-apps-http-endpoint.md). |
|||||

<a name="message-size-limits"></a>

#### <a name="message-size"></a>Tamanho da mensagem

| Nome | Limite de vários inquilinos | Limite do ambiente de serviço de integração | Observações |
|------|--------------------|---------------------------------------|-------|
| Tamanho da mensagem | 100 MB | 200 MB | Os conectores com etiqueta ISE usam o limite ISE, não os limites do conector não ISE. <p><p>Para contornar esse limite, consulte [Tratar mensagens grandes com agrupamento](../logic-apps/logic-apps-handle-large-messages.md). No entanto, alguns conectores e APIs podem não oferecer suporte a agrupamento ou até o limite padrão. |
| Tamanho da mensagem com agrupamento | 1 GB | 5 GB | Esse limite se aplica a ações que suportam fatiamento nativo ou permitem que você habilite o emparcelamento em sua configuração de tempo de execução. <p><p>Para o ambiente de serviço de integração, o mecanismo Logic Apps suporta esse limite, mas os conectores têm seus próprios limites de recompeamento até o limite do motor, por exemplo, veja a [referência de API do conector Azure Blob Storage](https://docs.microsoft.com/connectors/azureblob/). Para obter mais informações sobre o emalargamento, consulte [Manusear mensagens grandes com o chunking](../logic-apps/logic-apps-handle-large-messages.md). |
|||||

#### <a name="character-limits"></a>Limites de caracteres

| Nome | Observações |
|------|-------|
| Limite de avaliação da expressão | 131.072 caracteres | As expressões `@concat()`, `@base64()`, `@string()` não podem ser maiores do que esse limite. |
| Solicitar limite de caractere suinolo | 16.384 caracteres |
|||

#### <a name="retry-policy"></a>Política de repetição

| Nome | Limite | Observações |
| ---- | ----- | ----- |
| Tentativas de repetição | 90 | O padrão é 4. Para alterar o padrão, use o [Parâmetro de política de repetição](../logic-apps/logic-apps-workflow-actions-triggers.md). |
| Atraso máximo de nova tentativa | 1 dia | Para alterar o padrão, use o [Parâmetro de política de repetição](../logic-apps/logic-apps-workflow-actions-triggers.md). |
| Atraso mínimo de nova tentativa | 5 segundos | Para alterar o padrão, use o [Parâmetro de política de repetição](../logic-apps/logic-apps-workflow-actions-triggers.md). |
||||

<a name="custom-connector-limits"></a>

## <a name="custom-connector-limits"></a>Limites do conector personalizado

Aqui estão os limites para conectores personalizados que você pode criar de APIs da Web.

| Nome | Limite de vários inquilinos | Limite do ambiente de serviço de integração | Observações |
|------|--------------------|---------------------------------------|-------|
| Número de conectores personalizados | 1.000 por assinatura do Azure | 1.000 por assinatura do Azure ||
| Número de solicitações por minuto para um conector personalizado | 500 solicitações por minuto por conexão | 2.000 solicitações por minuto por *conector personalizado* ||
|||

<a name="managed-identity"></a>

## <a name="managed-identities"></a>Identidades gerenciadas

| Nome | Limite |
|------|-------|
| Identidades gerenciadas por aplicativo lógico | Ou a identidade atribuída ao sistema ou 1 identidade atribuída pelo usuário |
| Número de aplicativos lógicos que têm uma identidade gerenciada em uma assinatura do Azure por região | 250 |
|||

<a name="integration-account-limits"></a>

## <a name="integration-account-limits"></a>Limites da conta de integração

Cada assinatura do Azure tem esses limites de conta de integração:

* Uma conta de integração [de nível gratuito](../logic-apps/logic-apps-pricing.md#integration-accounts) por região do Azure

* 1.000 contas de integração total, incluindo contas de integração em quaisquer [ambientes de serviço de integração (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) em [Tanto O Desenvolvedor quanto o Premium SKUs](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level).

* Cada ISE, seja [Desenvolvedor ou Premium,](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level)está limitado a 5 contas de integração total:

  | ISE SKU | Limites da conta de integração |
  |---------|----------------------------|
  | **Premium** | 5 total - [Contas padrão](../logic-apps/logic-apps-pricing.md#integration-accounts) apenas, incluindo uma conta Standard gratuitamente. Não são permitidas contas gratuitas ou básicas. |
  | **Desenvolvedor** | 5 total - [Grátis](../logic-apps/logic-apps-pricing.md#integration-accounts) (limitado a 1 conta) e [Standard](../logic-apps/logic-apps-pricing.md#integration-accounts) combinado, ou todas as contas Standard. Não são permitidas contas básicas. Use o [SKU do desenvolvedor](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) para experimentos, desenvolvimento e testes, mas não para testes de produção ou desempenho. |
  |||

Os custos adicionais se aplicam às contas de integração que você adiciona além das contas de integração que estão incluídas em um ISE. Para saber como funcionam os preços e o faturamento dos ISEs, consulte o [modelo de preços da Logic Apps](../logic-apps/logic-apps-pricing.md#fixed-pricing). Para obter preços, consulte [preços da Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

<a name="artifact-number-limits"></a>

### <a name="artifact-limits-per-integration-account"></a>Limites de artefato por conta de integração

Aqui estão os limites do número de artefatos para cada nível de conta de integração.
Para obter preços, consulte [preços da Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/). Para saber como funcionam os preços e o faturamento das contas de integração, consulte o [modelo de preços da Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts).

> [!NOTE]
> Use o nível Free apenas para cenários exploratórios, não cenários de produção. Esta camada restringe o uso e a taxa de transferência e não tem nenhum SLA (Contrato de Nível de Serviço).

| Artefato | Grátis | Basic | Standard |
|----------|------|-------|----------|
| Contratos comerciais de EDI | 10 | 1 | 1,000 |
| Parceiros comerciais de EDI | 25 | 2 | 1,000 |
| Mapas | 25 | 500 | 1,000 |
| Esquemas | 25 | 500 | 1,000 |
| Assemblies | 10 | 25 | 1,000 |
| Certificados | 25 | 2 | 1,000 |
| Configurações de lote | 5 | 1 | 50 |
||||

<a name="artifact-capacity-limits"></a>

### <a name="artifact-capacity-limits"></a>Limites de capacidade do artefato

| Artefato | Limite | Observações |
| -------- | ----- | ----- |
| Assembly | 8 MB | Para carregar arquivos maiores que 2 MB, use um [contêiner de blob e uma conta de armazenamento do Azure](../logic-apps/logic-apps-enterprise-integration-schemas.md). |
| Mapa (arquivo XSLT) | 8 MB | Para carregar arquivos maiores que 2 MB, use a [API REST de Aplicativos Lógicos do Azure – Mapas](https://docs.microsoft.com/rest/api/logic/maps/createorupdate). <p><p>**Nota**: A quantidade de dados ou registros que um mapa pode processar com sucesso é baseada no tamanho da mensagem e nos limites de tempo limite de ação nos aplicativos azure logic. Por exemplo, se você usar uma ação HTTP, com base no [tamanho da mensagem HTTP e nos limites de tempo limite,](#request-limits)um mapa pode processar dados até o limite de tamanho da mensagem HTTP se a operação for concluída dentro do limite de tempo HTTP. |
| Esquema | 8 MB | Para carregar arquivos maiores que 2 MB, use um [contêiner de blob e uma conta de armazenamento do Azure](../logic-apps/logic-apps-enterprise-integration-schemas.md). |
||||

<a name="integration-account-throughput-limits"></a>

### <a name="throughput-limits"></a>Limites de taxa de transferência

| Ponto de extremidade de runtime | Grátis | Basic | Standard | Observações |
|------------------|------|-------|----------|-------|
| Chamadas de leitura a cada 5 minutos | 3.000 | 30,000 | 60.000 | Você pode distribuir a carga de trabalho entre mais de uma conta conforme necessário. |
| Chamadas de invocação a cada 5 minutos | 3.000 | 30,000 | 45,000 | Você pode distribuir a carga de trabalho entre mais de uma conta conforme necessário. |
| Chamadas de acompanhamento a cada 5 minutos | 3.000 | 30,000 | 45,000 | Você pode distribuir a carga de trabalho entre mais de uma conta conforme necessário. |
| Chamadas simultâneas de bloqueio | ~1,000 | ~1,000 | ~1,000 | O mesmo para todas as SKUs. Você pode reduzir o número de solicitações simultâneas ou reduzir a duração conforme necessário. |
||||

<a name="b2b-protocol-limits"></a>

### <a name="b2b-protocol-as2-x12-edifact-message-size"></a>Tamanho da mensagem do protocolo B2B (AS2, X12, EDIFACT)

Aqui estão os limites de tamanho de mensagem que se aplicam aos protocolos B2B:

| Nome | Limite de vários inquilinos | Limite do ambiente de serviço de integração | Observações |
|------|--------------------|---------------------------------------|-------|
| AS2 | v2 - 100 MB<br>v1 - 50 MB | v2 - 200 MB <br>v1 - 50 MB | Aplicável ao decodificar e codificar |
| X12 | 50 MB | 50 MB | Aplicável ao decodificar e codificar |
| EDIFACT | 50 MB | 50 MB | Aplicável ao decodificar e codificar |
||||

<a name="disable-delete"></a>

## <a name="disabling-or-deleting-logic-apps"></a>Desabilitar ou excluir aplicativos lógicos

Ao desabilitar um aplicativo lógico, nenhuma nova execução será instanciada.
Todas as corridas em andamento e pendentes continuam até que terminem, o que pode levar tempo para ser concluído.

Ao excluir um aplicativo lógico, nenhuma nova execução será instanciada.
Todas as execuções em andamento e pendentes serão canceladas.
Se você tiver milhares de execuções, o cancelamento poderá demorar um tempo significativo até ser concluído.

<a name="configuration"></a>

## <a name="firewall-configuration-ip-addresses-and-service-tags"></a>Configuração do firewall: endereços IP e tags de serviço

Os endereços IP que o Azure Logic Apps usa para chamadas de entrada e saída dependem da região onde seu aplicativo lógico existe. *Todos os* aplicativos lógicos da mesma região usam as mesmas faixas de endereço IP. Algumas chamadas [do Power Automate,](https://docs.microsoft.com/power-automate/getting-started) como as solicitações **HTTP** e HTTP **+ OpenAPI,** passam diretamente pelo serviço Azure Logic Apps e vêm dos endereços IP que estão listados aqui. Para obter mais informações sobre endereços IP usados pelo Power Automate, consulte [Limites e configuração no Power Automate](https://docs.microsoft.com/flow/limits-and-config#ip-address-configuration).

> [!TIP]
> Para ajudar a reduzir a complexidade ao criar regras de segurança, você pode usar opcionalmente [tags de serviço,](../virtual-network/service-tags-overview.md)em vez de especificar os endereços IP do Logic Apps para cada região, descritos posteriormente nesta seção. Essas tags funcionam em todas as regiões onde o serviço Logic Apps está disponível:
>
> * **LogicAppsManagement**: Representa os prefixos de endereço IP de entrada para o serviço Logic Apps.
> * **LogicApps**: Representa os prefixos de endereço IP de saída para o serviço Logic Apps.

* Para suportar as chamadas que seus aplicativos lógicos fazem diretamente com [HTTP](../connectors/connectors-native-http.md), [HTTP + Swagger](../connectors/connectors-native-http-swagger.md), e outras solicitações HTTP, configure seu firewall com todos os endereços IP de [entrada](#inbound) *e* [saída](#outbound) que são usados pelo serviço Logic Apps, com base nas regiões onde seus aplicativos lógicos existem. Esses endereços são exibidos sob os títulos **De entrada** e **De saída** nesta seção e são classificados por região.

* Para dar suporte às chamadas que [conectores gerenciados pela Microsoft](../connectors/apis-list.md) fazem, configure seu firewall com *todos* os endereços IP [de saída](#outbound) usados por esses conectores, com base nas regiões em que seus aplicativos lógicos existem. Esses endereços são exibidos sob o título **De saída** nesta seção e são classificados por região.

* Para habilitar a comunicação para aplicativos lógicos que são executados em um ambiente de serviço de integração (ISE), certifique-se de [abrir essas portas](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#network-ports-for-ise).

* Se seus aplicativos de lógica tiverem problemas para acessar contas de armazenamento do Azure que usam [firewalls e regras de firewall,](../storage/common/storage-network-security.md)você tem [várias opções para habilitar o acesso](../connectors/connectors-create-api-azureblobstorage.md#access-storage-accounts-behind-firewalls).

  Por exemplo, os aplicativos lógicos não podem acessar diretamente contas de armazenamento que usam regras de firewall e existem na mesma região. No entanto, se você permitir que os [endereços IP de saída para conectores gerenciados em sua região,](../logic-apps/logic-apps-limits-and-config.md#outbound)seus aplicativos lógicos podem acessar contas de armazenamento que estão em uma região diferente, exceto quando você usa os conectores Azure Table Storage ou Azure Queue Storage. Para acessar seu armazenamento de mesa ou armazenamento de filas, você pode usar o gatilho HTTP e as ações em vez disso. Para outras opções, consulte [Contas de armazenamento access atrás de firewalls](../connectors/connectors-create-api-azureblobstorage.md#access-storage-accounts-behind-firewalls).

* Para conectores personalizados, [o Azure Government](../azure-government/documentation-government-overview.md)e [o Azure China 21Vianet,](https://docs.microsoft.com/azure/china/)endereços IP fixos ou reservados não estão disponíveis.

<a name="inbound"></a>

### <a name="inbound-ip-addresses"></a>Endereços IP de entrada

Esta seção lista os endereços IP de entrada apenas para o serviço Azure Logic Apps. Para ajudar a reduzir a complexidade ao criar regras de segurança, você pode usar opcionalmente a [tag de serviço,](../virtual-network/service-tags-overview.md) **LogicAppsManagement,** em vez de especificar prefixos de endereço IP do Logic Apps de entrada para cada região. Esta tag funciona em todas as regiões onde o serviço Logic Apps está disponível. Se você tem o Governo Azure, consulte [Azure Government - Endereços IP de entrada](#azure-government-inbound).

<a name="multi-tenant-inbound"></a>

#### <a name="multi-tenant-azure---inbound-ip-addresses"></a>Endereços IP de entrada multi-inquilinos

| Região multi-inquilino | IP |
|---------------------|----|
| Leste da Austrália | 13.75.153.66, 104.210.89.222, 104.210.89.244, 52.187.231.161 |
| Sudeste da Austrália | 13.73.115.153, 40.115.78.70, 40.115.78.237, 52.189.216.28 |
| Sul do Brasil | 191.235.86.199, 191.235.95.229, 191.235.94.220, 191.234.166.198 |
| Canadá Central | 13.88.249.209, 52.233.30.218, 52.233.29.79, 40.85.241.105 |
| Leste do Canadá | 52.232.129.143, 52.229.125.57, 52.232.133.109, 40.86.202.42 |
| Índia Central | 52.172.157.194, 52.172.184.192, 52.172.191.194, 104.211.73.195 |
| Centro dos EUA | 13.67.236.76, 40.77.111.254, 40.77.31.87, 104.43.243.39 |
| Leste da Ásia | 168.63.200.173, 13.75.89.159, 23.97.68.172, 40.83.98.194 |
| Leste dos EUA | 137.135.106.54, 40.117.99.79, 40.117.100.228, 137.116.126.165 |
| Leste dos EUA 2 | 40.84.25.234, 40.79.44.7, 40.84.59.136, 40.70.27.253 |
| França Central | 52.143.162.83, 20.188.33.169, 52.143.156.55, 52.143.158.203 |
| Sul da França | 52.136.131.145, 52.136.129.121, 52.136.130.89, 52.136.131.4 |
| Leste do Japão | 13.71.146.140, 13.78.84.187, 13.78.62.130, 13.78.43.164 |
| Oeste do Japão | 40.74.140.173, 40.74.81.13, 40.74.85.215, 40.74.68.85 |
| Coreia Central | 52.231.14.182, 52.231.103.142, 52.231.39.29, 52.231.14.42 |
| Sul da Coreia | 52.231.166.168, 52.231.163.55, 52.231.163.150, 52.231.192.64 |
| Centro-Norte dos EUA | 168.62.249.81, 157.56.12.202, 65.52.211.164, 65.52.9.64 |
| Norte da Europa | 13.79.173.49, 52.169.218.253, 52.169.220.174, 40.112.90.39 |
| Norte da África do Sul | 102.133.228.4, 102.133.224.125, 102.133.226.199, 102.133.228.9 |
| África do Sul Ocidental | 102.133.72.190, 102.133.72.145, 102.133.72.184, 102.133.72.173 |
| Centro-Sul dos Estados Unidos | 13.65.98.39, 13.84.41.46, 13.84.43.45, 40.84.138.132 |
| Sul da Índia | 52.172.9.47, 52.172.49.43, 52.172.51.140, 104.211.225.152 |
| Sudeste Asiático | 52.163.93.214, 52.187.65.81, 52.187.65.155, 104.215.181.6 |
| Sul do Reino Unido | 51.140.79.109, 51.140.78.71, 51.140.84.39, 51.140.155.81 |
| Oeste do Reino Unido | 51.141.48.98, 51.141.51.145, 51.141.53.164, 51.141.119.150 |
| Centro-Oeste dos EUA | 52.161.26.172, 52.161.8.128, 52.161.19.82, 13.78.137.247 |
| Europa Ocidental | 13.95.155.53, 52.174.54.218, 52.174.49.6, 52.174.49.6 |
| Oeste da Índia | 104.211.164.112, 104.211.165.81, 104.211.164.25, 104.211.157.237 |
| Oeste dos EUA | 52.160.90.237, 138.91.188.137, 13.91.252.184, 157.56.160.212 |
| Oeste dos EUA 2 | 13.66.224.169, 52.183.30.10, 52.183.39.67, 13.66.128.68 |
|||

<a name="azure-government-inbound"></a>

#### <a name="azure-government---inbound-ip-addresses"></a>Governo Azure - Endereços IP de entrada

| Azure | IP |
|-------------------------|----|
| Governo dos EUA do Arizona | 52.244.67.164, 52.244.67.64, 52.244.66.82 |
| Governo dos EUA do Texas | 52.238.119.104, 52.238.112.96, 52.238.119.145 |
| Gov. dos EUA – Virgínia | 52.227.159.157, 52.227.152.90, 23.97.4.36 |
| DoD Central dos EUA | 52.182.49.204, 52.182.52.106 |
|||

<a name="outbound"></a>

### <a name="outbound-ip-addresses"></a>Endereços IP de saída

Esta seção lista os endereços IP de saída para o serviço Azure Logic Apps e conectores gerenciados. Para ajudar a reduzir a complexidade ao criar regras de segurança, você pode usar opcionalmente a [tag de serviço](../virtual-network/service-tags-overview.md), **LogicApps,** em vez de especificar prefixos de endereço IP de aplicativos lógicos de saída para cada região. Esta tag funciona em todas as regiões onde o serviço Logic Apps está disponível. Para conectores gerenciados, use os endereços IP. Se você tem o Governo Azure, consulte [Azure Government - Endereços IP de saída](#azure-government-outbound).

<a name="multi-tenant-outbound"></a>

#### <a name="multi-tenant-azure---outbound-ip-addresses"></a>Endereços IP multi-inquilinos - Outbound

| Região | IP de Aplicativos Lógicos | IP de conectores gerenciados |
|--------|---------------|-----------------------|
| Leste da Austrália | 13.75.149.4, 104.210.91.55, 104.210.90.241, 52.187.227.245, 52.187.226.96, 52.187.231.184, 52.187.229.130, 52.187.226.139 | 13.70.72.192 - 13.70.72.207, 13.72.243.10, 40.126.251.213, 52.237.214.72 |
| Sudeste da Austrália | 13.73.114.207, 13.77.3.139, 13.70.159.205, 52.189.222.77, 13.77.56.167, 13.77.58.136, 52.189.214.42, 52.189.220.75 | 13.70.136.174, 13.77.50.240 - 13.77.50.255, 40.127.80.34, 52.255.48.202 |
| Sul do Brasil | 191.235.82.221, 191.235.91.7, 191.234.182.26, 191.237.255.116, 191.234.161.168, 191.234.162.178, 191.234.161.28, 191.234.162.131 | 104.41.59.51, 191.232.38.129, 191.233.203.192 - 191.233.203.207, 191.232.191.157 |
| Canadá Central | 52.233.29.92, 52.228.39.241, 52.228.39.244, 40.85.250.135, 40.85.250.212, 13.71.186.1, 40.85.252.47, 13.71.184.150 | 13.71.170.208 - 13.71.170.223, 52.228.33.76, 52.228.34.13, 52.228.42.205, 52.233.31.197, 52.237.24.126, 52.237.32.212 |
| Leste do Canadá | 52.232.128.155, 52.229.120.45, 52.229.126.25, 40.86.203.228, 40.86.228.93, 40.86.216.241, 40.86.226.149, 40.86.217.241 | 40.69.106.240 - 40.69.106.255, 52.229.120.52, 52.229.120.178, 52.229.123.98, 52.229.126.202, 52.242.35.152, 52.242.30.112 |
| Índia Central | 52.172.154.168, 52.172.186.159, 52.172.185.79, 104.211.101.108, 104.211.102.62, 104.211.90.169, 104.211.90.162, 104.211.74.145 | 52.172.211.12, 104.211.81.192 - 104.211.81.207, 104.211.98.164, 52.172.212.129 |
| Centro dos EUA | 13.67.236.125, 104.208.25.27, 40.122.170.198, 40.113.218.230, 23.100.86.139, 23.100.87.24, 23.100.87.56, 23.100.82.16 | 13.89.171.80 - 13.89.171.95, 40.122.49.51, 52.173.245.164, 52.173.241.27 |
| Leste da Ásia | 13.75.94.173, 40.83.127.19, 52.175.33.254, 40.83.73.39, 65.52.175.34, 40.83.77.208, 40.83.100.69, 40.83.75.165 | 13.75.36.64 - 13.75.36.79, 23.99.116.181, 52.175.23.169, 13.75.110.131 |
| Leste dos EUA | 13.92.98.111, 40.121.91.41, 40.114.82.191, 23.101.139.153, 23.100.29.190, 23.101.136.201, 104.45.153.81, 23.101.132.208 | 40.71.11.80 - 40.71.11.95, 40.71.249.205, 40.114.40.132, 40.71.249.139 |
| Leste dos EUA 2 | 40.84.30.147, 104.208.155.200, 104.208.158.174, 104.208.140.40, 40.70.131.151, 40.70.29.214, 40.70.26.154, 40.70.27.236 | 40.70.146.208 - 40.70.146.223, 52.232.188.154, 104.208.233.100, 104.209.247.23, 52.225.129.144 |
| França Central | 52.143.164.80, 52.143.164.15, 40.89.186.30, 20.188.39.105, 40.89.191.161, 40.89.188.169, 40.89.186.28, 40.89.190.104 | 40.79.130.208 - 40.79.130.223, 40.89.135.2, 40.89.186.239 |
| Sul da França | 52.136.132.40, 52.136.129.89, 52.136.131.155, 52.136.133.62, 52.136.139.225, 52.136.130.144, 52.136.140.226, 52.136.129.51 | 40.79.178.240 - 40.79.178.255, 52.136.133.184, 52.136.142.154 |
| Leste do Japão | 13.71.158.3, 13.73.4.207, 13.71.158.120, 13.78.18.168, 13.78.35.229, 13.78.42.223, 13.78.21.155, 13.78.20.232 | 13.71.153.19, 13.78.108.0 - 13.78.108.15, 40.115.186.96, 13.73.21.230 |
| Oeste do Japão | 40.74.140.4, 104.214.137.243, 138.91.26.45, 40.74.64.207, 40.74.76.213, 40.74.77.205, 40.74.74.21, 40.74.68.85 | 40.74.100.224 - 40.74.100.239, 40.74.130.77, 104.215.61.248, 104.215.27.24 |
| Coreia Central | 52.231.14.11, 52.231.14.219, 52.231.15.6, 52.231.10.111, 52.231.14.223, 52.231.77.107, 52.231.8.175, 52.231.9.39 | 52.231.18.208 - 52.231.18.223, 52.141.36.214, 52.141.1.104 |
| Sul da Coreia | 52.231.204.74, 52.231.188.115, 52.231.189.221, 52.231.203.118, 52.231.166.28, 52.231.153.89, 52.231.155.206, 52.231.164.23 | 52.231.147.0 - 52.231.147.15, 52.231.163.10, 52.231.201.173 |
| Centro-Norte dos EUA | 168.62.248.37, 157.55.210.61, 157.55.212.238, 52.162.208.216, 52.162.213.231, 65.52.10.183, 65.52.9.96, 65.52.8.225 | 52.162.107.160 - 52.162.107.175, 52.162.242.161, 65.52.218.230, 52.162.126.4 |
| Norte da Europa | 40.113.12.95, 52.178.165.215, 52.178.166.21, 40.112.92.104, 40.112.95.216, 40.113.4.18, 40.113.3.202, 40.113.1.181 | 13.69.227.208 - 13.69.227.223, 52.178.150.68, 104.45.93.9, 94.245.91.93, 52.169.28.181 |
| Norte da África do Sul | 102.133.231.188, 102.133.231.117, 102.133.230.4, 102.133.227.103, 102.133.228.6, 102.133.230.82, 102.133.231.9, 102.133.231.51 | 13.65.86.57, 104.214.19.48 - 104.214.19.63, 104.214.70.191, 102.133.168.167 |
| África do Sul Ocidental | 102.133.72.98, 102.133.72.113, 102.133.75.169, 102.133.72.179, 102.133.72.37, 102.133.72.183, 102.133.72.132, 102.133.75.191 | 13.65.86.57, 104.214.19.48 - 104.214.19.63, 104.214.70.191, 102.133.72.85 |
| Centro-Sul dos Estados Unidos | 104.210.144.48, 13.65.82.17, 13.66.52.232, 23.100.124.84, 70.37.54.122, 70.37.50.6, 23.100.127.172, 23.101.183.225 | 13.65.86.57, 104.214.19.48 - 104.214.19.63, 104.214.70.191, 52.171.130.92 |
| Sul da Índia | 52.172.50.24, 52.172.55.231, 52.172.52.0, 104.211.229.115, 104.211.230.129, 104.211.230.126, 104.211.231.39, 104.211.227.229 | 13.71.125.22, 40.78.194.240 - 40.78.194.255, 104.211.227.225, 13.71.127.26 |
| Sudeste Asiático | 13.76.133.155, 52.163.228.93, 52.163.230.166, 13.76.4.194, 13.67.110.109, 13.67.91.135, 13.76.5.96, 13.67.107.128 | 13.67.8.240 - 13.67.8.255, 13.76.231.68, 52.187.68.19, 52.187.115.69 |
| Sul do Reino Unido | 51.140.74.14, 51.140.73.85, 51.140.78.44, 51.140.137.190, 51.140.153.135, 51.140.28.225, 51.140.142.28, 51.140.158.24 | 51.140.80.51, 51.140.148.0 - 51.140.148.15, 51.140.61.124, 51.140.74.150 |
| Oeste do Reino Unido | 51.141.54.185, 51.141.45.238, 51.141.47.136, 51.141.114.77, 51.141.112.112, 51.141.113.36, 51.141.118.119, 51.141.119.63 | 51.140.211.0 - 51.140.211.15, 51.141.47.105, 51.141.124.13, 51.141.52.185 |
| Centro-Oeste dos EUA | 52.161.27.190, 52.161.18.218, 52.161.9.108, 13.78.151.161, 13.78.137.179, 13.78.148.140, 13.78.129.20, 13.78.141.75 | 13.71.195.32 - 13.71.195.47, 52.161.102.22, 13.78.132.82, 52.161.101.204 |
| Europa Ocidental | 40.68.222.65, 40.68.209.23, 13.95.147.65, 23.97.218.130, 51.144.182.201, 23.97.211.179, 104.45.9.52, 23.97.210.126 | 13.69.64.208 - 13.69.64.223, 40.115.50.13, 52.174.88.118, 40.91.208.65, 52.166.78.89 |
| Oeste da Índia | 104.211.164.80, 104.211.162.205, 104.211.164.136, 104.211.158.127, 104.211.156.153, 104.211.158.123, 104.211.154.59, 104.211.154.7 | 104.211.146.224 - 104.211.146.239, 104.211.161.203, 104.211.189.218, 104.211.189.124 |
| Oeste dos EUA | 52.160.92.112, 40.118.244.241, 40.118.241.243, 157.56.162.53, 157.56.167.147, 104.42.49.145, 40.83.164.80, 104.42.38.32 | 40.112.243.160 - 40.112.243.175, 104.40.51.248, 104.42.122.49, 40.112.195.87, 13.93.148.62 |
| Oeste dos EUA 2 | 13.66.210.167, 52.183.30.169, 52.183.29.132, 13.66.210.167, 13.66.201.169, 13.77.149.159, 52.175.198.132, 13.66.246.219 | 13.66.140.128 - 13.66.140.143, 52.183.78.157, 52.191.164.250 |
||||

<a name="azure-government-outbound"></a>

#### <a name="azure-government---outbound-ip-addresses"></a>Governo Azure - Endereços IP de saída

| Região | IP de Aplicativos Lógicos | IP de conectores gerenciados |
|--------|---------------|-----------------------|
| Governo dos EUA do Arizona | 52.244.67.143, 52.244.65.66, 52.244.65.190 | 52.127.2.160 - 52.127.2.175, 52.244.69.0, 52.244.64.91 |
| Governo dos EUA do Texas | 52.238.114.217, 52.238.115.245, 52.238.117.119 | 52.127.34.160 - 52.127.34.175, 40.112.40.25, 52.238.161.225 |
| Gov. dos EUA – Virgínia | 13.72.54.205, 52.227.138.30, 52.227.152.44 | 52.127.42.128 - 52.127.42.143, 52.227.143.61, 52.227.162.91 |
| DoD Central dos EUA | 52.182.48.215, 52.182.92.143 | 52.127.58.160 - 52.127.58.175, 52.182.54.8, 52.182.48.136 |
||||

## <a name="next-steps"></a>Próximas etapas

* Saiba como [criar seu primeiro aplicativo lógico](../logic-apps/quickstart-create-first-logic-app-workflow.md)  
* Saiba mais sobre [exemplos e cenários comuns](../logic-apps/logic-apps-examples-and-scenarios.md)
