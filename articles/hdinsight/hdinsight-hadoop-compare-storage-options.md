---
title: Comparar opções de armazenamento para uso com clusters do Azure HDInsight
description: Fornece uma visão geral dos tipos de armazenamento e como eles funcionam com o Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 98d71434ac9e3f712be0cbd8c505b7d5a537e7cc
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79095540"
---
# <a name="compare-storage-options-for-use-with-azure-hdinsight-clusters"></a>Comparar opções de armazenamento para uso com clusters do Azure HDInsight

Você pode escolher entre alguns serviços de armazenamento diferentes do Azure ao criar clusters HDInsight:

* Armazenamento do Azure
* Azure Data Lake Storage Gen2
* Azure Data Lake Storage Gen1

Este artigo fornece uma visão geral desses tipos de armazenamento e de seus recursos exclusivos.

A tabela a seguir resume os serviços de armazenamento do Azure que são suportados com diferentes versões do HDInsight:

| Serviço de armazenamento | Tipo de conta | Tipo de namespace | Serviços com suporte | Níveis de desempenho compatíveis | Camadas de acesso compatíveis | Versão do HDInsight | Tipo de cluster |
|---|---|---|---|---|---|---|---|
|Azure Data Lake Storage Gen2| Uso geral V2 | Hierárquico (sistema de arquivos) | Blob | Standard | Quente, legal, arquivo | 3.6+ | Todos exceto Faísca 2.1 e 2.2|
|Armazenamento do Azure| Uso geral V2 | Objeto | Blob | Standard | Quente, legal, arquivo | 3.6+ | Todos |
|Armazenamento do Azure| Uso geral V1 | Objeto | Blob | Standard | N/D | Todos | Todos |
|Armazenamento do Azure| Armazenamento blob** | Objeto | Blob de blocos | Standard | Quente, legal, arquivo | Todos | Todos |
|Azure Data Lake Storage Gen1| N/D | Hierárquico (sistema de arquivos) | N/D | N/D | N/D | 3.6 Somente | Todos, exceto HBase |

**Para clusters HDInsight, apenas contas de armazenamento secundárias podem ser do tipo BlobStorage e Page Blob não é uma opção de armazenamento suportado.

Para obter mais informações sobre os tipos de conta do Azure Storage, consulte [a visão geral da conta de armazenamento do Azure](../storage/common/storage-account-overview.md)

Para obter mais informações sobre os níveis de acesso do Azure Storage, consulte [o armazenamento Azure Blob: Premium (visualização), níveis de armazenamento Hot, Cool e Archive](../storage/blobs/storage-blob-storage-tiers.md)

Você pode criar um cluster usando diferentes combinações de serviços para armazenamento secundário primário e opcional. A tabela a seguir resume as configurações de armazenamento de cluster que atualmente são suportadas no HDInsight:

| Versão do HDInsight | Armazenamento primário | Armazenamento secundário | Com suporte |
|---|---|---|---|
| 3.6 & 4.0 | Propósito Geral V1 , Propósito Geral V2 | Propósito Geral V1, Propósito Geral V2, BlobStorage (Blobs de bloco) | Sim |
| 3.6 & 4.0 | Propósito Geral V1 , Propósito Geral V2 | Armazenamento do Data Lake Gen2 | Não |
| 3.6 & 4.0 | Data Lake Storage Gen2* | Armazenamento do Data Lake Gen2 | Sim |
| 3.6 & 4.0 | Data Lake Storage Gen2* | Propósito Geral V1, Propósito Geral V2, BlobStorage (Blobs de bloco) | Sim |
| 3.6 & 4.0 | Armazenamento do Data Lake Gen2 | Armazenamento do Data Lake Gen1 | Não |
| 3.6 | Armazenamento do Data Lake Gen1 | Armazenamento do Data Lake Gen1 | Sim |
| 3.6 | Armazenamento do Data Lake Gen1 | Propósito Geral V1, Propósito Geral V2, BlobStorage (Blobs de bloco) | Sim |
| 3.6 | Armazenamento do Data Lake Gen1 | Armazenamento do Data Lake Gen2 | Não |
| 4,0 | Armazenamento do Data Lake Gen1 | Qualquer | Não |
| 4,0 | Propósito Geral V1 , Propósito Geral V2 | Armazenamento do Data Lake Gen1 | Não |

*=Esta pode ser uma ou várias contas do Data Lake Storage Gen2, desde que todas estejam configuradas para usar a mesma identidade gerenciada para acesso a clusters.

> [!Note] 
> O armazenamento principal do Data Lake Storage Gen2 não é suportado para clusters Spark 2.1 ou 2.2. 

## <a name="use-azure-data-lake-storage-gen2-with-apache-hadoop-in-azure-hdinsight"></a>Usar Azure Data Lake Storage Gen2 com Apache Hadoop no Azure HDInsight

O Azure Data Lake Storage Gen2 obtém os recursos de núcleo do Azure Data Lake Storage Gen1 e os integra ao armazenamento de Blobs do Azure. Esses recursos incluem um sistema de arquivos que é compatível com o Hadoop, o Azure Active Directory (Azure AD), e as listas de controle de acesso baseadas em POSIX (ACLs). Essa combinação permite usufruir do desempenho do Azure Data Lake Storage Gen1, ao mesmo tempo em que usa o gerenciamento de ciclo de vida de dados e de camada do Armazenamento de Blobs.

Para obter mais informações sobre o Azure Data Lake Storage Gen2, confira [Introdução ao Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).

### <a name="core-functionality-of-azure-data-lake-storage-gen2"></a>Funcionalidade principal do Azure Data Lake Storage Gen2

* **Acesso compatível com Hadoop:** No Azure Data Lake Storage Gen2, você pode gerenciar e acessar dados assim como faria com um Hadoop Distributed File System (HDFS). O driver do Sistema de Arquivos de Blobs do Azure (ABFS) está disponível em todos os ambientes do Apache Hadoop, incluindo o Azure HDInsight e o Azure Databricks. Use o ABFS para acessar os dados armazenados no Data Lake Storage Gen2.

* **Um superconjunto de permissões POSIX:** O modelo de segurança do Data Lake Gen2 suporta permissões ACL e POSIX, juntamente com alguma granularidade extra específica do Data Lake Storage Gen2. As configurações podem ser definidas por meio de ferramentas administrativas ou de estruturas como o Apache Hive e o Apache Spark.

* **Custo-benefício:** Data Lake Storage Gen2 oferece capacidade de armazenamento de baixo custo e transações. Recursos como o ciclo de vida de armazenamento de Blobs do Azure ajudam a reduzir os custos, ajustando as taxas de cobrança conforme a movimentação de dados no ciclo de vida.

* **Compatibilidade com ferramentas, frameworks e aplicativos blob:** O Data Lake Storage Gen2 continua a trabalhar com uma ampla gama de ferramentas, frameworks e aplicativos para armazenamento Blob.

* **Driver otimizado:** O driver ABFS é otimizado especificamente para análise de big data. As APIs REST correspondentes são exibidas por meio do ponto de extremidade do Sistema de Arquivos Distribuído (DFS), dfs.core.windows.net.

### <a name="whats-new-for-azure-data-lake-storage-gen-2"></a>Novidades no Azure Data Lake Storage Gen2

#### <a name="managed-identities-for-secure-file-access"></a>Identidades gerenciadas para acesso de arquivo seguro

O HDInsight do Azure usa identidades gerenciadas para proteger o acesso do cluster a arquivos no Azure Data Lake Storage Gen2. As identidades gerenciadas são um recurso do Azure Active Directory que fornece aos serviços do Azure um conjunto de credenciais gerenciadas automaticamente. Essas credenciais podem ser usadas para autenticar qualquer serviço com suporte para autenticação do Active Directory. O uso de identidades gerenciadas não exige que você armazene credenciais em código ou arquivos de configuração.

Para saber mais, confira [Gerenciar identidades para recursos do Azure](../active-directory/managed-identities-azure-resources/overview.md).

#### <a name="azure-blob-file-system-driver"></a>Driver do Sistema de Arquivos de Blobs do Azure

Os aplicativos do Apache Hadoop esperam nativamente ler e gravar dados do armazenamento em disco local. Um driver do sistema de arquivos do Hadoop, como o ABFS, permite que os aplicativos do Hadoop funcionem com o armazenamento em nuvem, emulando operações do sistema de arquivos regulares do Hadoop. O driver converte esses comandos recebidos do aplicativo em operações entendidas pela plataforma de armazenamento de nuvem verdadeira.

Anteriormente, o driver do sistema de arquivos do Hadoop convertia todas as operações do sistema de arquivos para chamadas à API REST do Armazenamento do Azure no lado do cliente e, em seguida, invocava a API REST. Essa conversão do lado do cliente, no entanto, resultou em várias chamadas à API REST para uma única operação do sistema de arquivos, como a renomeação de um arquivo. O ABFS moveu parte da lógica de sistema de arquivos do Hadoop do lado do cliente para o lado do servidor. A API do Azure Data Lake Storage Gen2 agora é executada em paralelo com a API de Blobs. Essa migração melhora o desempenho porque agora as operações comuns do sistema de arquivos do Hadoop podem ser executadas com uma chamada à API REST.

Para obter mais informações, consulte [O driver Azure Blob Filesystem (ABFS): Um driver de armazenamento Azure dedicado para Hadoop](../storage/blobs/data-lake-storage-abfs-driver.md).

#### <a name="uri-scheme-for-azure-data-lake-storage-gen-2"></a>Esquema de URI no Azure Data Lake Storage Gen2 

O Azure Data Lake Storage Gen2 usa um novo esquema de URI para acessar arquivos no Armazenamento do Azure a partir do HDInsight:

`abfs://<FILE_SYSTEM_NAME>@<ACCOUNT_NAME>.dfs.core.windows.net/<PATH>`

O esquema URI fornece acesso criptografado por SSL.

`<FILE_SYSTEM_NAME>` identifica o caminho do sistema de arquivos Data Lake Storage Gen2.

`<ACCOUNT_NAME>` identifica o nome da conta do Armazenamento do Azure. Um FQDN (nome de domínio totalmente qualificado) é necessário.

O `<PATH>` é o nome do caminho de HDFS do arquivo ou diretório.

Se os valores para `<FILE_SYSTEM_NAME>` e `<ACCOUNT_NAME>` não forem especificados, será usado o sistema de arquivos padrão. Para os arquivos no sistema de arquivos padrão, use um caminho absoluto ou um caminho relativo. Por exemplo, o arquivo `hadoop-mapreduce-examples.jar` que vem com clusters HDInsight pode ser referido usando um dos caminhos a seguir:

```
abfs://myfilesystempath@myaccount.dfs.core.windows.net/example/jars/hadoop-mapreduce-examples.jar
abfs:///example/jars/hadoop-mapreduce-examples.jar /example/jars/hadoop-mapreduce-examples.jar
```

> [!Note]
> O nome do arquivo é `hadoop-examples.jar` nos clusters HDInsight versões 2.1 e 1.6. Quando estiver trabalhando com arquivos fora do HDInsight, você verá que a maioria dos utilitários não reconhece o formato ABFS mas, em vez disso, esperam um formato de caminho básico, como `example/jars/hadoop-mapreduce-examples.jar`.

Para obter mais informações, consulte [Usar URI do Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction-abfs-uri.md).

## <a name="azure-storage"></a>Armazenamento do Azure

O Armazenamento do Azure é uma solução robusta de armazenamento de uso geral que se integra perfeitamente ao HDInsight. O HDInsight pode usar um contêiner de blobs no Armazenamento do Azure como o sistema de arquivos padrão para o cluster. Ao usar uma interface do HDFS, o conjunto completo de componentes no HDInsight pode operar diretamente sobre os dados estruturados ou não estruturados armazenados como blobs.

Recomendamos usar recipientes de armazenamento separados para seu armazenamento de cluster padrão e seus dados de negócios, para isolar os logs do HDInsight e arquivos temporários de seus próprios dados de negócios. Também recomendamos a exclusão do recipiente blob padrão, que contém registros de aplicativos e sistemas, após cada uso para reduzir o custo de armazenamento. Certifique-se de recuperar os logs antes de excluir o contêiner.

Se você optar por proteger sua conta de armazenamento com os **Firewalls e** as restrições de redes virtuais em **redes selecionadas,** certifique-se de ativar a exceção **Permita serviços confiáveis** da Microsoft... para que o HDInsight possa acessar sua conta de armazenamento.

### <a name="hdinsight-storage-architecture"></a>Arquitetura de armazenamento do HDInsight

O diagrama a seguir fornece uma exibição abstrata da arquitetura do HDInsight do Armazenamento do Azure:

![Arquitetura de armazenamento HDInsight](./media/hdinsight-hadoop-compare-storage-options/storage-architecture.png "Arquitetura de armazenamento HDInsight")

O HDInsight fornece acesso ao sistema de arquivos distribuídos que está anexado localmente aos nós de computação. Esse sistema de arquivos pode ser acessado usando o URI totalmente qualificado, por exemplo:

    hdfs://<namenodehost>/<path>

Por meio do HDInsight, também é possível acessar os dados no Armazenamento do Azure. A sintaxe é mostrada a seguir:

    wasb://<containername>@<accountname>.blob.core.windows.net/<path>

Considere os seguintes princípios ao usar uma conta de Armazenamento do Azure com clusters do HDInsight:

* **Contêineres nas contas de armazenamento que estão conectadas a um cluster:** como o nome e a chave da conta são associados ao cluster durante a criação, você tem acesso completo aos blobs nesses contêineres.

* **Contêineres públicos ou blobs públicos em contas de armazenamento que *não* estão conectados a um cluster:** Você tem permissão apenas para ler as bolhas nos recipientes.
  
  > [!NOTE]  
  > Os contêineres públicos permitem obter uma lista de todos os blobs disponíveis nesse contêiner e os metadados do contêiner. Um blob público somente permite acessar os blobs se você souber a URL exata. Para obter mais informações, confira [Gerenciar acesso anônimo de leitura aos contêineres e blobs](../storage/blobs/storage-manage-access-to-resources.md).

* **Contêineres privados em contas de armazenamento que *não* estão conectados a um cluster:** Você não pode acessar as bolhas nos contêineres a menos que você defina a conta de armazenamento quando você enviar os trabalhos do WebHCat. 

As contas de armazenamento definidas no processo de criação e suas chaves são armazenadas em %HADOOP_HOME%/conf/core-site.xml nos nós do cluster. Por padrão, o HDInsight usa as contas de armazenamento definidas no arquivo core-site.xml. Você pode modificar essa configuração usando o [Apache Ambari](./hdinsight-hadoop-manage-ambari.md).

Vários trabalhos do WebHCat, incluindo o Apache Hive, MapReduce, streaming do Apache Hadoop e o Apache Pig, podem conter uma descrição de contas de armazenamento e metadados. (Isso é atualmente verdade para pig com contas de armazenamento, mas não para metadados.) Para obter mais informações, [consulte Usando um cluster HDInsight com contas de armazenamento e metalojas alternativas.](https://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx)

Os blobs podem ser usados para dados estruturados e não estruturados. Os contêineres de blobs armazenam dados como pares de chave/valor e não possuem nenhuma hierarquia de diretório. No entanto, o nome da chave pode incluir o caractere de barra ( / ) para parecer que o arquivo está armazenado em uma estrutura de diretório. Por exemplo, a chave de um blob pode ser `input/log1.txt`. Não existe nenhum diretório `input`, mas por causa do caractere de barra no nome da chave, a chave se parece com um caminho de arquivo.

### <a name="benefits-of-azure-storage"></a><a id="benefits"></a>Benefícios do Armazenamento do Azure
Os clusters de cálculo e os recursos de armazenamento que não são colocados acarretam custos de desempenho. Esses custos são reduzidos dependendo da maneira que os clusters de cálculo são criados próximos aos recursos de conta de armazenamento na região do Azure. Nessa região, os nós de computação podem acessar com eficiência os dados na rede de alta velocidade dentro do Armazenamento do Azure.

Quando armazena os dados no armazenamento do Azure em vez de no HDFS, você obtém vários benefícios:

* **Compartilhamento e reutilização de dados:** os dados no HDFS estão localizados dentro do cluster de computação. Apenas os aplicativos que têm acesso ao cluster de computação podem usar os dados usando a API HDFS. Por outro lado, os dados no Armazenamento do Azure podem ser acessados por meio das APIs do HDFS ou por meio das APIs REST do Armazenamento de Blobs. Em razão dessa organização, um conjunto maior de aplicativos (incluindo outros clusters do HDInsight) e ferramentas pode ser usado para produzir e consumir os dados.

* **Arquivamento de dados:** Quando os dados são armazenados no Azure Storage, os clusters HDInsight usados para computação podem ser excluídos com segurança sem perder os dados do usuário.

* **Custo de armazenamento de dados:** Armazenar dados no DFS a longo prazo é mais caro do que armazenar os dados no Azure Storage porque o custo de um cluster de computação é mais alto do que o custo do Armazenamento Azure. Além disso, como os dados não precisam ser recarregados para cada geração de cluster de cálculo, você estará economizando também em custos de carregamento de dados.

* **Escala horizontal elástica:** embora o HDFS forneça um sistema de arquivos escalado horizontalmente, a escala é determinada pelo número de nós que você cria para o seu cluster. A alteração da escala pode ser mais complicada do que depender dos recursos de dimensionamento elástico obtidos automaticamente no Armazenamento do Azure.

* **Geo-replicação:** Seu Armazenamento Azure pode ser replicado geografamente. Embora a replicação geográfica forneça recuperação geográfica e redundância de dados, um failover para a localização com replicação geográfica afetará seriamente o desempenho e poderá gerar custos adicionais. Portanto, escolha a replicação geográfica com cuidado e somente se o valor dos dados justificar o custo adicional.

Determinados trabalhos e pacotes do MapReduce podem criar resultados intermediários que talvez você não queira realmente armazenar no contêiner de Armazenamento do Azure. Nesse caso, é possível ainda optar por armazenar os dados no HDFS local. O HDInsight usa o DFS para vários desses resultados intermediários em trabalhos do Hive e outros processos.

> [!NOTE]  
> A maioria dos comandos HDFS (por exemplo, `ls`, `copyFromLocal` e `mkdir`) ainda funciona conforme o esperado no Armazenamento do Azure. Apenas os comandos específicos à implementação nativa do HDFS (que é conhecida como DFS), como `fschk` e `dfsadmin`, apresentam um comportamento diferente no Armazenamento do Azure.

## <a name="overview-of-azure-data-lake-storage-gen1"></a>Visão Geral do Azure Data Lake Storage Gen1

O Azure Data Lake Storage Gen1 é um repositório de hiperescala em toda a empresa para cargas de trabalho de análise de big data. Com o Azure Data Lake, você pode capturar dados de qualquer tamanho, tipo e velocidade de ingestão em um único lugar para análises operacionais e exploratórias.

Acesse o Data Lake Storage Gen1 a partir do Hadoop (disponível com um cluster do HDInsight) usando as APIs REST compatíveis com WebHDFS. O Data Lake Storage Gen1 foi desenvolvido para permitir a análise dos dados armazenados e está ajustado para trabalhar em cenários de análise de dados. Proporcionando uso imediato, ele inclui os recursos que são essenciais para a utilização em casos corporativos reais. Esses recursos incluem segurança, capacidade de gerenciamento, escalabilidade, confiabilidade e disponibilidade.

Para saber mais sobre o Azure Data Lake Storage Gen1, confira a [Visão geral do Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md).

Os principais recursos do Data Lake Storage Gen1 incluem o seguinte.

### <a name="compatibility-with-hadoop"></a>Compatibilidade com o Hadoop

O Data Lake Storage Gen1 é um sistema de arquivos Apache Hadoop compatível com o HDFS e funciona com o ecossistema Hadoop.  Seus aplicativos ou serviços existentes do HDInsight que usam a API WebHDFS podem ser facilmente integrados ao Data Lake Storage Gen1. O Data Lake Storage Gen1 também exibe uma interface REST compatível com WebHDFS para aplicativos.

Os dados armazenados no Data Lake Storage Gen1 podem ser facilmente analisados usando estruturas analíticas do Hadoop, como MapReduce ou Hive. Os clusters do Azure HDInsight podem ser provisionados e configurados para acessar diretamente os dados armazenados no Data Lake Storage Gen1.

### <a name="unlimited-storage-petabyte-files"></a>Armazenamento ilimitado, arquivos em petabytes

O Data Lake Storage Gen1 fornece armazenamento ilimitado e é adequado para armazenar uma variedade de dados para análise. Ele não impõe limites de tamanho de conta, de tamanho de arquivo ou de quantidade de dados que podem ser armazenados em um Data Lake. Os arquivos individuais podem variar no tamanho, de quilobytes a petabytes, o que torna o Data Lake Storage Gen1 uma ótima opção para armazenar qualquer tipo de dados. Os dados são armazenados permanentemente por meio da criação de várias cópias, e não há limite de tempo de armazenamento dos dados no Data Lake.

### <a name="performance-tuning-for-big-data-analytics"></a>Ajustes de desempenho para a análise de big data

O Data Lake Storage Gen1 foi desenvolvido para executar sistemas analíticos de larga escala que exigem uma taxa de transferência massiva para consultar e analisar grandes quantidades de dados. O Data Lake espalha as partes de um arquivo por vários servidores de armazenamento individual. Quando você estiver analisando os dados, essa configuração melhorará a taxa de transferência de leitura quando o arquivo é lido em paralelo.

### <a name="readiness-for-enterprise-highly-available-and-secure"></a>Prontidão para a empresa: Altamente disponível e seguro

O Data Lake Storage Gen1 fornece disponibilidade e confiabilidade padrão do setor. Os ativos de dados são armazenados permanentemente: cópias redundantes protegem contra falhas inesperadas. As empresas podem usar o Data Lake Storage Gen1 em suas soluções como parte importante de sua plataforma de dados existente.

O Data Lake Storage Gen1 também fornece segurança de nível corporativo para os dados armazenados. Para obter mais informações, consulte [Protegendo dados no Armazenamento de dados do Lake Azure Gen1](#DataLakeStoreSecurity).

### <a name="flexible-data-structures"></a>Estruturas de dados flexível

O Data Lake Storage Gen1 pode armazenar qualquer dado em seu formato nativo, no estado em que estiver, sem a necessidade de transformações anteriores. O Data Lake Storage Gen1 não exige a definição de um esquema antes de os dados serem carregados. A estrutura de análise individual interpreta os dados e define um esquema no momento da análise. Como é capaz de armazenar arquivos de tamanhos e formatos arbitrários, o Data Lake Storage Gen1 pode manipular dados estruturados, semi-estruturados e não estruturados.

Os contêineres Data Lake Storage Gen1 para dados são essencialmente pastas e arquivos. Você opera nos dados armazenados usando SDKs, o portal do Azure e o Azure PowerShell. Desde que você coloque os dados no repositório usando essas interfaces e os contêineres apropriados, é possível armazenar qualquer tipo de dados. O Data Lake Storage Gen1 não executa nenhum tratamento especial de dados com base no tipo de dados que armazena.

## <a name="data-security-in-data-lake-storage-gen1"></a><a name="DataLakeStoreSecurity"></a>Segurança de dados no Data Lake Storage Gen1
O Data Lake Storage Gen1 usa o Azure Active Directory para autenticação e as listas de controle de acesso (ACLs) para gerenciar o acesso aos seus dados.

| **Recurso** | **Descrição** |
| --- | --- |
| Autenticação |O Data Lake Storage Gen1 integra-se ao Azure Active Directory (Azure AD) para gerenciamento de identidade e acesso de todos os dados armazenados no Data Lake Storage Gen1. Devido à integração, o Data Lake Storage Gen1 se beneficia de todos os recursos do Azure AD. Esses recursos incluem autenticação multifatorial, acesso condicional, controle de acesso baseado em funções, monitoramento de uso de aplicativos, monitoramento e alerta de segurança, e assim por diante. O Data Lake Storage Gen1 oferece suporte ao protocolo OAuth 2.0 para autenticação na interface REST. Confira o artigo sobre a [autenticação no Azure Data Lake Storage Gen1 usando o Azure Active Directory](../data-lake-store/data-lakes-store-authentication-using-azure-active-directory.md)|
| Controle de acesso |O Data Lake Storage Gen1 fornece controle de acesso oferecendo suporte a permissões no estilo POSIX, que são expostas pelo protocolo WebHDFS. As ACLs podem ser habilitadas na pasta raiz, nas subpastas e nos arquivos individuais. Para saber mais sobre como as ACLs funcionam no contexto do Data Lake Storage Gen1, confira o artigo sobre o [controle de acesso no Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md). |
| Criptografia |O Azure Data Lake Storage Gen1 também fornece criptografia para dados armazenados na conta. Você especifica as configurações de criptografia ao criar uma conta do Data Lake Storage Gen1. É possível optar por ter os dados criptografados ou não escolher nenhuma criptografia. Para obter mais informações, consulte [Criptografia no Data Lake Storage Gen1](../data-lake-store/data-lake-store-encryption.md). Para obter instruções sobre como fornecer uma configuração relacionada à criptografia, confira a [Introdução ao Azure Data Lake Storage Gen1 usando o portal do Azure ](../data-lake-store/data-lake-store-get-started-portal.md). |

Para saber mais sobre como proteger os dados no Data Lake Storage Gen1, confira o artigo sobre como [proteger os dados armazenados no Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-secure-data.md).

## <a name="applications-that-are-compatible-with-data-lake-storage-gen1"></a>Aplicativos compatíveis com o Data Lake Storage Gen1
O Azure Data Lake Storage Gen1 é compatível com a maioria dos componentes de software livre no ecossistema do Hadoop. Ele também se integra perfeitamente com outros serviços do Azure.  Siga os links abaixo para saber mais sobre como o Data Lake Storage Gen1 pode ser usado tanto com componentes de software livre quanto com outros serviços do Azure.

* Confira o artigo [Aplicativos de software livre de big data que funcionam com o Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-compatible-oss-other-applications.md) para obter uma lista de aplicativos de software livre interoperáveis com o Data Lake Storage Gen1.
* Confira o artigo [Integração do Azure Data Lake Storage Gen1 a outros serviços do Azure](../data-lake-store/data-lake-store-integrate-with-other-services.md) para entender como usar os serviços do Data Lake Storage Gen1 com outros serviços do Azure a fim de permitir uma gama mais ampla de cenários.
* Confira o artigo [Usando o Azure Data Lake Storage Gen1 para exigências de big data](../data-lake-store/data-lake-store-data-scenarios.md) para aprender a usar o Data Lake Storage Gen1 em cenários como a ingestão de dados, o processamento de dados, o download de dados e a visualização de dados.

## <a name="data-lake-storage-gen1-file-system-adl"></a>Sistema de arquivo do Data Lake Storage Gen1 (adl://)
Em ambientes do Hadoop (disponíveis com um cluster do HDInsight), é possível acessar o Data Lake Storage Gen1 por meio do novo sistema de arquivos, o AzureDataLakeFilesystem (adl: //). O desempenho de aplicativos e serviços que usam o adl:// pode ser otimizado de maneiras que não estão disponíveis no WebHDFS. Como resultado, ao usar o Data Lake Storage Gen1, você obtém a flexibilidade de aproveitar o melhor desempenho usando o adl:// recomendado ou de manter o código existente continuando a usar a API WebHDFS diretamente. O Azure HDInsight aproveita totalmente o AzureDataLakeFilesystem para fornecer o melhor desempenho no Data Lake Storage Gen1.

Acesse seus dados no Data Lake Storage Gen1 usando o seguinte:

`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net`

Para saber mais sobre como acessar os dados no Data Lake Storage Gen1, confira o artigo [Ações disponíveis nos dados armazenados](../data-lake-store/data-lake-store-get-started-portal.md#properties).



## <a name="next-steps"></a>Próximas etapas

* [Introdução ao Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Introdução ao Armazenamento do Azure](../storage/common/storage-introduction.md)
