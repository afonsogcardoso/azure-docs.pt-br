---
title: Vertex Execution View in Data Lake Tools for Visual Studio
description: Este artigo descreve como usar o modo de exibição de Execução de Vértice para examinar trabalhos do Data Lake Analytics.
services: data-lake-analytics
ms.service: data-lake-analytics
author: jasonwhowell
ms.author: jasonh
ms.assetid: 5366d852-e7d6-44cf-a88c-e9f52f15f7df
ms.topic: conceptual
ms.date: 10/13/2016
ms.openlocfilehash: f5adbb75e6852551976aa040a1a1c723d2e3f59b
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "71309723"
---
# <a name="use-the-vertex-execution-view-in-data-lake-tools-for-visual-studio"></a>Usar o Modo de Exibição de Execução de Vértice nas Ferramentas do Data Lake para Visual Studio
Saiba como usar o Modo de Exibição de Execução de Vértice para examinar trabalhos do Data Lake Analytics.


## <a name="open-the-vertex-execution-view"></a>Abrir o Modo de Exibição de Execução de Vértice
Abra um trabalho de U-SQL em Ferramentas do Data Lake para Visual Studio. Clique em **Exibição de Execuções de Vértice** no canto inferior esquerdo. Talvez você receba uma solicitação para carregar perfis primeiro e isso pode demorar algum tempo dependendo de sua conectividade de rede.

![Modo de Exibição de Execução de Vértice das Ferramentas do Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-open-vertex-execution-view.png)

## <a name="understand-vertex-execution-view"></a>Noções básicas do Modo de Exibição de Execução de Vértice
A Exibição de Execuções de Vértice tem três partes:

![Modo de Exibição de Execução de Vértice das Ferramentas do Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-vertex-execution-view.png)

O **seletor de vértice** à esquerda permite selecionar vértices pelos recursos (como 10 principais dados lidos ou optar por estágio). Um dos filtros mais usados é para ver os **vértices no caminho crítico**. O **Caminho crítico** é a cadeia mais longa de vértices de um trabalho U-SQL. Entender o caminho crítico é útil para otimizar seus trabalhos verificando quais vértice demoram mais.
  
![Modo de Exibição de Execução de Vértice das Ferramentas do Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-vertex-execution-view-pane2.png)

O painel central superior mostra o **status de execução de todos os vértices**.
  
![Modo de Exibição de Execução de Vértice das Ferramentas do Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-vertex-execution-view-pane3.png)

O painel central inferior mostra informações sobre cada vértice:
* Nome do Processo: o nome da instância do vértice. Ele é composto por diferentes partes em StageName|VertexName|VertexRunInstance. Por exemplo, o vértice de SV7_Split[62].v1 representa a segunda instância em execução (.v1, índice que começa em 0) do vértice número 62 no estágio SV7_Split.
* Leitura/Gravação Total de Dados: os dados foram lidos/gravados por esse vértice.
* Estado/Status de Saída: o status final quando o vértice é encerrado.
* Código de Saída/Tipo de Falha: o erro no momento da falha do vértice.
* Motivo da Criação: por que o vértice foi criado.
* Latência do Recurso/Latência do processo/ Latência da Fila de PN: o tempo necessário para o vértice aguardar recursos, processar dados e permanecer na fila.
* GUID do Processo/Criador: o GUID do vértice em execução no momento ou de seu criador.
* Versão: a enésima instância do vértice em execução (o sistema pode agendar novas instâncias de um vértice por vários motivos, por exemplo, para failover, redundância de computação etc.)
* Hora da Criação da Versão.
* Hora de Início da Criação do Processo/Hora de Enfileiramento do Processo/Hora de Início do processo/Hora de Conclusão do Processo: quando o processo do vértice começa a criação; quando o processo do vértice começa a enfileirar; quando o processo do vértice especificado começa; quando o vértice especificado é concluído.

## <a name="next-steps"></a>Próximas etapas
* Para registrar informações de diagnóstico, consulte [Acessar registros de diagnósticos do Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md)
* Para ver uma consulta mais complexa, consulte [Analisar logs de site usando a Análise Data Lake do Azure](data-lake-analytics-analyze-weblogs.md).
* Para ver detalhes do trabalho, confira [Usar o Navegador de Trabalhos e o Modo de Exibição de Trabalho para trabalhos do Azure Data Lake Analytics](data-lake-analytics-data-lake-tools-view-jobs.md)
