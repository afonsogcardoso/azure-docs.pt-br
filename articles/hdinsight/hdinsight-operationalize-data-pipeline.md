---
title: Operacionalize um pipeline de análise de dados - Azure
description: Configure e execute um exemplo de pipeline de dados que é disparado por novos dados e produz resultados concisos.
author: ashishthaps
ms.author: ashishth
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/25/2019
ms.openlocfilehash: 16c7af4d66bd550eb4a286de7c86c436b1fe10e2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75922657"
---
# <a name="operationalize-a-data-analytics-pipeline"></a>Operacionalize um pipeline de análise de dados

Os *pipelines de dados* são subjacentes a muitas soluções de análise de dados. Como o nome sugere, um pipeline de dados coleta dados brutos, limpa e remodela-os conforme necessário e, em seguida, normalmente realiza cálculos ou agregações antes de armazenar os dados processados. Os dados processados são consumidos pelos clientes, relatórios ou APIs. Um pipeline de dados deve fornecer resultados repetíveis, quer seja em uma agenda ou disparado por novos dados.

Este artigo descreve como operacionalizar seus pipelines de dados para repetição usando Oozie em execução em clusters Hadoop do HDInsight. O cenário de exemplo orienta você por um pipeline de dados que prepara e processa dados de série temporal de voos de uma companhia aérea.

No cenário a seguir, os dados de entrada estão em um arquivo simples que contém um lote de dados de voo para um mês. Esses dados de voo incluem informações como os aeroportos de origem e de destino, as milhas percorridas, horários de saída e de chegada etc. O objetivo deste pipeline é resumir o desempenho diário da companhia aérea, e cada companhia aérea tem uma linha por dia, com a média de atraso na partida e na chegada em minutos e o total de milhas percorridas nesse dia.

| YEAR | MONTH | DAY_OF_MONTH | CARRIER |AVG_DEP_DELAY | AVG_ARR_DELAY |TOTAL_DISTANCE |
| --- | --- | --- | --- | --- | --- | --- |
| 2017 | 1 | 3 | AA | 10.142229 | 7.862926 | 2644539 |
| 2017 | 1 | 3 | AS | 9,435449 | 5,482143 | 572289 |
| 2017 | 1 | 3 | DL | 6,935409 | -2,1893024 | 1909696 |

O pipeline de exemplo aguarda a chegada de novos dados de voo, depois, armazena essas informações detalhadas do voo em seu data warehouse do Apache Hive para análise de longo prazo. O pipeline também cria um conjunto de dados bem menor que resume apenas os dados de voos diários. Esses dados diários de resumo de voo são enviados para um banco de dados SQL para fornecer relatórios, como para um site.

O diagrama a seguir ilustra o pipeline de exemplo.

![Visão geral do pipeline de dados do exemplo de voo HDI](./media/hdinsight-operationalize-data-pipeline/flight-pipeline-overview.png)

## <a name="apache-oozie-solution-overview"></a>Visão geral da solução Apache Oozie

Esse pipeline usa o Apache Oozie em execução em um cluster Hadoop do HDInsight.

O Oozie descreve seus pipelines em termos de *ações*, *fluxos de trabalho* e *coordenadores*. As ações determinam o trabalho real a ser executado, como a execução de uma consulta do Hive. Os fluxos de trabalho definem a sequência de ações. Os coordenadores definem a agenda de execução do fluxo de trabalho. Os coordenadores também podem esperar novos dados serem disponibilizados antes de iniciar uma instância do fluxo de trabalho.

O diagrama a seguir mostra o design de alto nível desse exemplo de pipeline do Oozie.

![Oozie Flight exemplo De Pipeline de Dados](./media/hdinsight-operationalize-data-pipeline/pipeline-overview-oozie.png)

## <a name="provision-azure-resources"></a>Provisionar recursos do Azure

Esse pipeline exige um Banco de Dados SQL do Azure e um cluster Hadoop do HDInsight no mesmo local. O Banco de Dados Azure SQL armazena os dados de resumo produzidos pelo pipeline e o Oozie Metadata Store.

### <a name="provision-azure-sql-database"></a>Provisionar um Banco de Dados SQL do Azure

1. Crie um Banco de Dados SQL do Azure. Consulte [Criar um banco de dados SQL do Azure no portal Azure](../sql-database/sql-database-single-database-get-started.md).

1. Para garantir que seu cluster HDInsight possa acessar o banco de dados Azure SQL conectado, configure as regras de firewall do Banco de Dados SQL do Azure para permitir que os serviços e recursos do Azure acessem o servidor. Você pode habilitar essa opção no portal Azure selecionando **Set server firewall**e selecionando **ON** abaixo Permitir que os serviços e recursos do **Azure acessem esse servidor** para o servidor ou banco de dados Do Azure SQL. Para obter mais informações, confira [Criar e gerenciar regras de firewall de IP](../sql-database/sql-database-firewall-configure.md#use-the-azure-portal-to-manage-server-level-ip-firewall-rules).

1. Use [o editor de consulta](../sql-database/sql-database-single-database-get-started.md#query-the-database) para executar as `dailyflights` seguintes instruções SQL para criar a tabela que armazenará os dados resumidos de cada execução do pipeline.

    ```sql
    CREATE TABLE dailyflights
    (
        YEAR INT,
        MONTH INT,
        DAY_OF_MONTH INT,
        CARRIER CHAR(2),
        AVG_DEP_DELAY FLOAT,
        AVG_ARR_DELAY FLOAT,
        TOTAL_DISTANCE FLOAT
    )
    GO

    CREATE CLUSTERED INDEX dailyflights_clustered_index on dailyflights(YEAR,MONTH,DAY_OF_MONTH,CARRIER)
    GO
    ```

Seu Banco de dados SQL do Azure está pronto.

### <a name="provision-an-apache-hadoop-cluster"></a>Provisão de um Aglomerado Apache Hadoop

Crie um cluster Apache Hadoop com uma metaloja personalizada. Durante a criação de cluster saem do portal, a partir da guia **Armazenamento,** certifique-se de selecionar seu banco de dados SQL nas **configurações do Metastore**. Para obter mais informações sobre a seleção de um metastore, consulte [Selecionar um metastore personalizado durante a criação do cluster](./hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation). Para obter mais informações sobre criação de clusters, consulte [Get Started com HDInsight on Linux](hadoop/apache-hadoop-linux-tutorial-get-started.md).

## <a name="verify-ssh-tunneling-set-up"></a>Verifique a configuração do túnel SSH

Para usar o Console da Web do Oozie a fim de exibir o status de suas instâncias de coordenador e de fluxo de trabalho, configure um túnel SSH para seu cluster do HDInsight. Para saber mais, confira [Túnel SSH](hdinsight-linux-ambari-ssh-tunnel.md).

> [!NOTE]  
> Você também pode usar o Chrome com a extensão [Foxy Proxy](https://getfoxyproxy.org/) para procurar os recursos da Web de seu cluster no túnel SSH. Configure-o para usar um proxy em as solicitações por meio do host `localhost` na porta 9876 do túnel. Essa abordagem é compatível com o Subsistema do Windows para Linux, também conhecido como Bash no Windows 10.

1. Execute o seguinte comando para abrir um túnel `CLUSTERNAME` SSH para o seu cluster, onde está o nome do seu cluster:

    ```cmd
    ssh -C2qTnNf -D 9876 sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Verifique se o túnel está funcionando navegando até o Ambari em seu nó principal procurando:

    `http://headnodehost:8080`

1. Para acessar o **Oozie Web Console** de dentro da Ambari, navegue até **o Oozie** > **Quick Links** > [servidor ativo] > **Oozie Web UI**.

## <a name="configure-hive"></a>Configurar o Hive

### <a name="upload-data"></a>Carregar dados

1. Baixe um arquivo CSV de exemplo que contenha dados de voo de um mês inteiro. Baixe o arquivo ZIP `2017-01-FlightData.zip` do [repositório do GitHub para HDInsight](https://github.com/hdinsight/hdinsight-dev-guide) e descompacte-o no arquivo CSV `2017-01-FlightData.csv`.

1. Copie esse arquivo CSV na conta de Armazenamento do Azure anexada ao seu cluster HDInsight e coloque-o na pasta `/example/data/flights`.

    1. Use o SCP para copiar os arquivos de seu computador local para o armazenamento local de seu nó principal do cluster HDInsight.

        ```cmd
        scp ./2017-01-FlightData.csv sshuser@CLUSTERNAME-ssh.azurehdinsight.net:2017-01-FlightData.csv
        ```

    1. Use [o comando ssh](./hdinsight-hadoop-linux-use-ssh-unix.md) para se conectar ao seu cluster. Edite o comando abaixo substituindo `CLUSTERNAME` pelo nome do cluster e, em seguida, digite o comando:

        ```cmd
        ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
        ```

    1. A partir da sessão ssh, use o comando HDFS para copiar o arquivo do seu nó principal de armazenamento local para o Armazenamento Azure.

        ```bash
        hadoop fs -mkdir /example/data/flights
        hdfs dfs -put ./2017-01-FlightData.csv /example/data/flights/2017-01-FlightData.csv
        ```

### <a name="create-tables"></a>Criar tabelas

Os dados de exemplo agora disponíveis. No entanto, o pipeline exige duas tabelas do Hive para o processamento, uma para os dados de entrada (`rawFlights`) e outra para os dados resumidos (`flights`). Crie essas tabelas no Ambari da seguinte maneira.

1. Faça logon no Ambari navegando para `http://headnodehost:8080`.

2. Na lista de serviços, selecione **Hive**.

    ![Lista de serviços Apache Ambari selecionando Colmeia](./media/hdinsight-operationalize-data-pipeline/hdi-ambari-services-hive.png)

3. Selecione **Ir para Exibição** ao lado do rótulo Exibição do Hive 2.0.

    ![Lista de resumos da Colmeia Apache Ambari](./media/hdinsight-operationalize-data-pipeline/hdi-ambari-services-hive-summary.png)

4. Na área de texto de consulta, cole as instruções a seguir para criar a tabela `rawFlights`. A tabela `rawFlights` fornece um esquema durante a leitura dos arquivos CSV dentro da pasta `/example/data/flights` no Armazenamento do Azure.

    ```sql
    CREATE EXTERNAL TABLE IF NOT EXISTS rawflights (
        YEAR INT,
        MONTH INT,
        DAY_OF_MONTH INT,
        FL_DATE STRING,
        CARRIER STRING,
        FL_NUM STRING,
        ORIGIN STRING,
        DEST STRING,
        DEP_DELAY FLOAT,
        ARR_DELAY FLOAT,
        ACTUAL_ELAPSED_TIME FLOAT,
        DISTANCE FLOAT)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
    WITH SERDEPROPERTIES
    (
        "separatorChar" = ",",
        "quoteChar"     = "\""
    )
    LOCATION '/example/data/flights'
    ```

5. Selecione **Executar** para criar a tabela.

    ![hdi ambari serviços hive consulta](./media/hdinsight-operationalize-data-pipeline/hdi-ambari-services-hive-query.png)

6. Para criar a tabela `flights`, substitua o texto na área de texto de consulta pelas instruções a seguir. A `flights` tabela é uma tabela gerenciada pela Colmeia que particiona dados carregados nela por ano, mês e dia do mês. Essa tabela conterá todos os dados históricos de voo, com a granularidade mais baixa presente nos dados de origem de uma linha por voo.

    ```sql
    SET hive.exec.dynamic.partition.mode=nonstrict;

    CREATE TABLE flights
    (
        FL_DATE STRING,
        CARRIER STRING,
        FL_NUM STRING,
        ORIGIN STRING,
        DEST STRING,
        DEP_DELAY FLOAT,
        ARR_DELAY FLOAT,
        ACTUAL_ELAPSED_TIME FLOAT,
        DISTANCE FLOAT
    )
    PARTITIONED BY (YEAR INT, MONTH INT, DAY_OF_MONTH INT)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
    WITH SERDEPROPERTIES 
    (
        "separatorChar" = ",",
        "quoteChar"     = "\""
    );
    ```

7. Selecione **Executar** para criar a tabela.

## <a name="create-the-oozie-workflow"></a>Criar o fluxo de trabalho do Oozie

Pipelines normalmente processam dados em lotes de acordo com um intervalo determinado. Nesse caso, o pipeline processa os dados de voo diários. Essa abordagem permite o recebimento diário, semanal, mensal ou anual de arquivos CSV.

O fluxo de trabalho de exemplo processa os dados de voo diariamente, em três etapas principais:

1. Execução de uma consulta do Hive para extrair os dados do intervalo de datas do arquivo CSV de origem representado pela tabela `rawFlights` e inserção dos dados na tabela `flights`.
2. Execução de uma consulta de Hive para criar dinamicamente uma tabela de preparo no Hive do dia, que contém uma cópia dos dados de voo resumidos por dia e companhia aérea.
3. Uso do Apache Sqoop para copiar todos os dados da tabela de preparo diária no Hive para a tabela de destino `dailyflights` no Banco de Dados SQL do Azure. O Sqoop lê as linhas de origem dos dados por trás da tabela Hive que reside no Armazenamento do Azure e os carrega no Banco de Dados SQL usando uma conexão JDBC.

Essas três etapas são coordenadas por um fluxo de trabalho do Oozie.

1. A partir de sua estação `job.properties`de trabalho local, crie um arquivo chamado . Use o texto abaixo como o conteúdo inicial do arquivo.
Em seguida, atualize os valores para o seu ambiente específico. A tabela abaixo do texto resume cada uma das propriedades e indica onde você pode encontrar os valores para o seu próprio ambiente.

    ```text
    nameNode=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net
    jobTracker=[ACTIVERESOURCEMANAGER]:8050
    queueName=default
    oozie.use.system.libpath=true
    appBase=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie
    oozie.wf.application.path=${appBase}/load_flights_by_day
    hiveScriptLoadPartition=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-load-flights-partition.hql
    hiveScriptCreateDailyTable=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-create-daily-summary-table.hql
    hiveDailyTableName=dailyflights${year}${month}${day}
    hiveDataFolder=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/example/data/flights/day/${year}/${month}/${day}
    sqlDatabaseConnectionString="jdbc:sqlserver://[SERVERNAME].database.windows.net;user=[USERNAME];password=[PASSWORD];database=[DATABASENAME]"
    sqlDatabaseTableName=dailyflights
    year=2017
    month=01
    day=03
    ```

    | Propriedade | Origem do valor |
    | --- | --- |
    | nameNode | O caminho completo até o Contêiner do Armazenamento do Azure anexado ao seu cluster HDInsight. |
    | jobTracker | O nome do host interno do nó principal YARN de seu cluster ativo. Na home page do Ambari, selecione YARN na lista de serviços e escolha o Gerenciador de Recursos Ativo. O URI do nome do host é exibido na parte superior da página. Acrescente a porta 8050. |
    | queueName | O nome da fila YARN usada ao programar as ações de Hive. Mantenha o padrão. |
    | oozie.use.system.libpath | Deixe como true. |
    | appBase | O caminho até a subpasta no Armazenamento do Azure onde você implanta o fluxo de trabalho e os arquivos de suporte do Oozie. |
    | oozie.wf.application.path | O local do fluxo de trabalho do Oozie `workflow.xml` para execução. |
    | hiveScriptLoadPartition | O caminho no Armazenamento do Azure até o arquivo de consulta de Hive `hive-load-flights-partition.hql`. |
    | hiveScriptCreateDailyTable | O caminho no Armazenamento do Azure até o arquivo de consulta de Hive `hive-create-daily-summary-table.hql`. |
    | hiveDailyTableName | O nome gerado dinamicamente a ser usado para a tabela de preparo. |
    | hiveDataFolder | O caminho no Armazenamento do Azure até os dados contidos na tabela de preparo. |
    | sqlDatabaseConnectionString | A cadeia de conexão da sintaxe JDBC para o seu Banco de Dados SQL do Azure. |
    | sqlDatabaseTableName | O nome da tabela no Banco de Dados SQL do Azure na qual os resumos de linha são inseridos. Deixe como `dailyflights`. |
    | year | O componente ano do dia para o qual os resumos de voo são computados. Deixe como está. |
    | month | O componente mês do dia para o qual os resumos de voo são computados. Deixe como está. |
    | dia | O componente dia do mês do dia para o qual os resumos de voo são computados. Deixe como está. |

1. A partir de sua estação `hive-load-flights-partition.hql`de trabalho local, crie um arquivo chamado . Use o código abaixo como o conteúdo do arquivo.

    ```sql
    SET hive.exec.dynamic.partition.mode=nonstrict;

    INSERT OVERWRITE TABLE flights
    PARTITION (YEAR, MONTH, DAY_OF_MONTH)
    SELECT 
        FL_DATE,
        CARRIER,
        FL_NUM,
        ORIGIN,
        DEST,
        DEP_DELAY,
        ARR_DELAY,
        ACTUAL_ELAPSED_TIME,
        DISTANCE,
        YEAR,
        MONTH,
        DAY_OF_MONTH
    FROM rawflights
    WHERE year = ${year} AND month = ${month} AND day_of_month = ${day};
    ```

    Variáveis do Oozie usam a sintaxe `${variableName}`. Essas variáveis estão definidas no `job.properties` arquivo. O Oozie substitui os valores reais no runtime.

1. A partir de sua estação `hive-create-daily-summary-table.hql`de trabalho local, crie um arquivo chamado . Use o código abaixo como o conteúdo do arquivo.

    ```sql
    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}
    (
        YEAR INT,
        MONTH INT,
        DAY_OF_MONTH INT,
        CARRIER STRING,
        AVG_DEP_DELAY FLOAT,
        AVG_ARR_DELAY FLOAT,
        TOTAL_DISTANCE FLOAT
    )
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE TABLE ${hiveTableName}
    SELECT  year, month, day_of_month, carrier, avg(dep_delay) avg_dep_delay, 
            avg(arr_delay) avg_arr_delay, sum(distance) total_distance 
    FROM flights
    GROUP BY year, month, day_of_month, carrier 
    HAVING year = ${year} AND month = ${month} AND day_of_month = ${day};
    ```

    Essa consulta cria uma tabela de preparo que armazenará apenas os dados resumidos de um dia. Observe a instrução SELECT que calcula a média de atrasos e a distância total percorrida pela companhia aérea por dia. Os dados inseridos nessa tabela são armazenados em um local conhecido (o caminho indicado pela variável hiveDataFolder), para que possam ser usados como a origem para o Sqoop na próxima etapa.

1. A partir de sua estação `workflow.xml`de trabalho local, crie um arquivo chamado . Use o código abaixo como o conteúdo do arquivo. Essas etapas acima são expressas como ações separadas no arquivo de fluxo de trabalho Oozie.

    ```xml
    <workflow-app name="loadflightstable" xmlns="uri:oozie:workflow:0.5">
        <start to = "RunHiveLoadFlightsScript"/>
        <action name="RunHiveLoadFlightsScript">
            <hive xmlns="uri:oozie:hive-action:0.2">
                <job-tracker>${jobTracker}</job-tracker>
                <name-node>${nameNode}</name-node>
                <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
                </configuration>
                <script>${hiveScriptLoadPartition}</script>
                <param>year=${year}</param>
                <param>month=${month}</param>
                <param>day=${day}</param>
            </hive>
            <ok to="RunHiveCreateDailyFlightTableScript"/>
            <error to="fail"/>
        </action>
    
        <action name="RunHiveCreateDailyFlightTableScript">
            <hive xmlns="uri:oozie:hive-action:0.2">
                <job-tracker>${jobTracker}</job-tracker>
                <name-node>${nameNode}</name-node>
                <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
                </configuration>
                <script>${hiveScriptCreateDailyTable}</script>
                <param>hiveTableName=${hiveDailyTableName}</param>
                <param>year=${year}</param>
                <param>month=${month}</param>
                <param>day=${day}</param>
                <param>hiveDataFolder=${hiveDataFolder}/${year}/${month}/${day}</param>
            </hive>
            <ok to="RunSqoopExport"/>
            <error to="fail"/>
        </action>
    
        <action name="RunSqoopExport">
            <sqoop xmlns="uri:oozie:sqoop-action:0.2">
                <job-tracker>${jobTracker}</job-tracker>
                <name-node>${nameNode}</name-node>
                <configuration>
                <property>
                    <name>mapred.compress.map.output</name>
                    <value>true</value>
                </property>
                </configuration>
                <arg>export</arg>
                <arg>--connect</arg>
                <arg>${sqlDatabaseConnectionString}</arg>
                <arg>--table</arg>
                <arg>${sqlDatabaseTableName}</arg>
                <arg>--export-dir</arg>
                <arg>${hiveDataFolder}/${year}/${month}/${day}</arg>
                <arg>-m</arg>
                <arg>1</arg>
                <arg>--input-fields-terminated-by</arg>
                <arg>"\t"</arg>
                <archive>mssql-jdbc-7.0.0.jre8.jar</archive>
                </sqoop>
            <ok to="end"/>
            <error to="fail"/>
        </action>
        <kill name="fail">
            <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
        </kill>
        <end name="end"/>
    </workflow-app>
    ```

As duas consultas da Colmeia são acessadas por seu caminho no Azure Storage, e os valores variáveis restantes são fornecidos pelo `job.properties` arquivo. Este arquivo configura o fluxo de trabalho a ser executado para a data de 3 de janeiro de 2017.

## <a name="deploy-and-run-the-oozie-workflow"></a>Implantar e executar o fluxo de trabalho do Oozie

Use o SCP da sua sessão de bash`workflow.xml`para implantar seu`hive-load-flights-partition.hql` fluxo `hive-create-daily-summary-table.hql`de trabalho Oozie (), as consultas da Colmeia (e), e a configuração do trabalho ().`job.properties`  No Oozie, somente o arquivo `job.properties` pode existir no armazenamento local do nó principal. Todos os outros arquivos devem ser armazenados em HDFS, nesse caso, o Armazenamento do Azure. A ação de Sqoop usada pelo fluxo de trabalho depende de um driver JDBC para se comunicar com seu Banco de Dados SQL, que deve ser copiado do nó principal para o HDFS.

1. Crie a subpasta `load_flights_by_day` no caminho do usuário no armazenamento local do nó principal. A partir da sessão ssh aberta, execute o seguinte comando:

    ```bash
    mkdir load_flights_by_day
    ```

1. Copie todos os arquivos no diretório atual (os arquivos `workflow.xml` e `job.properties`) até a subpasta `load_flights_by_day`. A partir de sua estação de trabalho local, execute o seguinte comando:

    ```cmd
    scp ./* sshuser@CLUSTERNAME-ssh.azurehdinsight.net:load_flights_by_day
    ```

1. Copie os arquivos do fluxo de trabalho no HDFS. A partir da sessão ssh aberta, execute os seguintes comandos:

    ```bash
    cd load_flights_by_day
    hadoop fs -mkdir -p /oozie/load_flights_by_day
    hdfs dfs -put ./* /oozie/load_flights_by_day
    ```

1. Copie `mssql-jdbc-7.0.0.jre8.jar` do nó principal local para a pasta de fluxo de trabalho no HDFS. Revise o comando conforme necessário se o cluster contiver um arquivo de frasco diferente. Revise `workflow.xml` conforme necessário para refletir um arquivo de jardiferente. A partir da sessão ssh aberta, execute o seguinte comando:

    ```bash
    hdfs dfs -put /usr/share/java/sqljdbc_7.0/enu/mssql-jdbc*.jar /oozie/load_flights_by_day
    ```

1. Execute o fluxo de trabalho. A partir da sessão ssh aberta, execute o seguinte comando:

    ```bash
    oozie job -config job.properties -run
    ```

1. Observe o status usando o Console da Web do Oozie. No Ambari, selecione **Oozie**, **Links Rápidos** e, em seguida, **Console da Web do Oozie**. Na guia **Trabalhos de Fluxo de Trabalho**, selecione **Todos os Trabalhos**.

    ![hdi oozie web console fluxos de trabalho](./media/hdinsight-operationalize-data-pipeline/hdi-oozie-web-console-workflows.png)

1. Quando o status for bem sucedido, consulte a tabela SQL Database para visualizar as linhas inseridas. No portal do Azure, navegue até o painel de seu Banco de Dados SQL, selecione **Ferramentas** e abra o **Editor de Consulta**.

        SELECT * FROM dailyflights

Agora que o fluxo de trabalho está em execução para o dia de teste único, você pode encapsular esse fluxo de trabalho com um coordenador que agenda a execução diária do fluxo de trabalho.

## <a name="run-the-workflow-with-a-coordinator"></a>Executar o fluxo de trabalho com um coordenador

Para agendar a execução diária desse fluxo de trabalho (ou a execução durante os dias de um intervalo de datas), use um coordenador. Um coordenador é definido por um arquivo XML, por exemplo `coordinator.xml`:

```xml
<coordinator-app name="daily_export" start="2017-01-01T00:00Z" end="2017-01-05T00:00Z" frequency="${coord:days(1)}" timezone="UTC" xmlns="uri:oozie:coordinator:0.4">
    <datasets>
        <dataset name="ds_input1" frequency="${coord:days(1)}" initial-instance="2016-12-31T00:00Z" timezone="UTC">
            <uri-template>${sourceDataFolder}${YEAR}-${MONTH}-FlightData.csv</uri-template>
            <done-flag></done-flag>
        </dataset>
    </datasets>
    <input-events>
        <data-in name="event_input1" dataset="ds_input1">
            <instance>${coord:current(0)}</instance>
        </data-in>
    </input-events>
    <action>
        <workflow>
            <app-path>${appBase}/load_flights_by_day</app-path>
            <configuration>
                <property>
                    <name>year</name>
                    <value>${coord:formatTime(coord:nominalTime(), 'yyyy')}</value>
                </property>
                <property>
                    <name>month</name>
                    <value>${coord:formatTime(coord:nominalTime(), 'MM')}</value>
                </property>
                <property>
                    <name>day</name>
                    <value>${coord:formatTime(coord:nominalTime(), 'dd')}</value>
                </property>
                <property>
                    <name>hiveScriptLoadPartition</name>
                    <value>${hiveScriptLoadPartition}</value>
                </property>
                <property>
                    <name>hiveScriptCreateDailyTable</name>
                    <value>${hiveScriptCreateDailyTable}</value>
                </property>
                <property>
                    <name>hiveDailyTableNamePrefix</name>
                    <value>${hiveDailyTableNamePrefix}</value>
                </property>
                <property>
                    <name>hiveDailyTableName</name>
                    <value>${hiveDailyTableNamePrefix}${coord:formatTime(coord:nominalTime(), 'yyyy')}${coord:formatTime(coord:nominalTime(), 'MM')}${coord:formatTime(coord:nominalTime(), 'dd')}</value>
                </property>
                <property>
                    <name>hiveDataFolderPrefix</name>
                    <value>${hiveDataFolderPrefix}</value>
                </property>
                <property>
                    <name>hiveDataFolder</name>
                    <value>${hiveDataFolderPrefix}${coord:formatTime(coord:nominalTime(), 'yyyy')}/${coord:formatTime(coord:nominalTime(), 'MM')}/${coord:formatTime(coord:nominalTime(), 'dd')}</value>
                </property>
                <property>
                    <name>sqlDatabaseConnectionString</name>
                    <value>${sqlDatabaseConnectionString}</value>
                </property>
                <property>
                    <name>sqlDatabaseTableName</name>
                    <value>${sqlDatabaseTableName}</value>
                </property>
            </configuration>
        </workflow>
    </action>
</coordinator-app>
```

Como você pode ver, grande parte do coordenador está apenas passando informações de configuração para a instância de fluxo de trabalho. No entanto, alguns itens importantes merecem destaque.

* Ponto 1: os atributos `start` e `end` no próprio elemento `coordinator-app` controlam o intervalo de tempo durante o qual o coordenador é executado.

    ```
    <coordinator-app ... start="2017-01-01T00:00Z" end="2017-01-05T00:00Z" frequency="${coord:days(1)}" ...>
    ```

    Um coordenador é responsável por agendar ações dentro do intervalo de datas `start` e `end`, de acordo com o intervalo especificado pelo atributo `frequency`. Cada ação agendada, por sua vez, executa o fluxo de trabalho conforme configurado. Na definição do coordenador acima, o coordenador está configurado para executar ações de 1º de janeiro de 2017 a 5 de janeiro de 2017. A freqüência é definida como um dia `${coord:days(1)}`pela expressão [de freqüência da Linguagem de Expressão Oozie](https://oozie.apache.org/docs/4.2.0/CoordinatorFunctionalSpec.html#a4.4._Frequency_and_Time-Period_Representation) . Isso é resultado do agendamento por parte do coordenador de uma ação (e, portanto, do fluxo de trabalho), uma vez por dia. Para intervalos de datas que estão no passado, como neste exemplo, a ação será agendada para execução sem atraso. O início da data a partir da qual a execução de uma ação é programada é chamada de *tempo nominal*. Por exemplo, para processar os dados de 1º de janeiro de 2017, o coordenador agendará a ação com um tempo nominal de 2017-01-01T00:00:00 GMT.

* Ponto 2: dentro do intervalo de datas do fluxo de trabalho, o elemento `dataset` especifica onde procurar no HDFS os dados de um determinado intervalo de datas, e configura como o Oozie determina se os dados ainda estão disponíveis para processamento.

    ```xml
    <dataset name="ds_input1" frequency="${coord:days(1)}" initial-instance="2016-12-31T00:00Z" timezone="UTC">
        <uri-template>${sourceDataFolder}${YEAR}-${MONTH}-FlightData.csv</uri-template>
        <done-flag></done-flag>
    </dataset>
    ```

    O caminho até os dados no HDFS é criado dinamicamente de acordo com a expressão fornecida no elemento `uri-template`. Neste coordenador, uma frequência de um dia também é usada com o conjunto de dados. Embora as datas de início e de término no elemento coordenador controlem quando as ações são agendados (e definem seus tempos nominais), `initial-instance` e `frequency` no conjunto de dados controlam o cálculo da data usada para construir o `uri-template`. Nesse caso, defina a instância inicial como um dia antes do início do coordenador, a fim de garantir que ele seleciona o primeiro dia (01/01/2017) de dados. O cálculo da data do conjunto de `initial-instance` dados avança a partir do valor de (31/12/2016) avançando em incrementos de frequência de conjunto de dados (um dia) até encontrar a data mais recente que não passa o tempo nominal definido pelo coordenador (2017-01-01T00:00:00 GMT para a primeira ação).

    O elemento `done-flag` vazio indica que quando o Oozie verifica a presença de dados de entrada na hora indicada, o Oozie determina se os dados estão disponíveis pela presença de um arquivo ou diretório. Neste caso, é a presença de um arquivo csv. Se houver um arquivo csv, o Oozie assumirá que os dados estão prontos e iniciará uma instância do fluxo de trabalho para processar o arquivo. Se não houver nenhum arquivo csv presente, Oozie assume que os dados ainda não estão prontos e que a execução do fluxo de trabalho entra em um estado de espera.

* Ponto 3: o elemento `data-in` especifica o carimbo de hora específico a ser usado como o tempo nominal ao substituir os valores em `uri-template` para o conjunto de dados associado.

    ```xml
    <data-in name="event_input1" dataset="ds_input1">
        <instance>${coord:current(0)}</instance>
    </data-in>
    ```

    Nesse caso, defina a instância como a expressão `${coord:current(0)}`, que é convertida no uso do tempo nominal da ação, conforme agendado originalmente pelo coordenador. Em outras palavras, quando o coordenador agenda a ação a ser executada com um tempo nominal de 01/01/2017, 01/01/2017 será usado para substituir as variáveis YEAR (2017) e MONTH (01) no modelo de URI. Após a computação do modelo de URI para essa instância, o Oozie verifica se o diretório ou arquivo esperado está disponível e agenda adequadamente a próxima execução do fluxo de trabalho.

Os três pontos anteriores são combinados para produzir uma situação na qual o coordenador agenda diariamente o processamento dos dados de origem.

* Ponto 1: o coordenador começa com uma data nominal de 2017-01-01.

* Ponto 2: o Oozie procura os dados disponíveis em `sourceDataFolder/2017-01-FlightData.csv`.

* Ponto 3: quando o Oozie encontra esse arquivo, ele agenda uma instância do fluxo de trabalho que processará os dados para 2017-01-01. Depois, o Oozie continua o processamento de 2017-01-02. Essa avaliação se repete até, mas sem incluir, 2017-01-05.

Assim como ocorre com os fluxos de trabalho, a configuração de um coordenador é definida em um arquivo `job.properties`, que tem um superconjunto das configurações usadas pelo fluxo de trabalho.

```text
nameNode=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net
jobTracker=[ACTIVERESOURCEMANAGER]:8050
queueName=default
oozie.use.system.libpath=true
appBase=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie
oozie.coord.application.path=${appBase}
sourceDataFolder=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/example/data/flights/
hiveScriptLoadPartition=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-load-flights-partition.hql
hiveScriptCreateDailyTable=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-create-daily-summary-table.hql
hiveDailyTableNamePrefix=dailyflights
hiveDataFolderPrefix=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/example/data/flights/day/
sqlDatabaseConnectionString="jdbc:sqlserver://[SERVERNAME].database.windows.net;user=[USERNAME];password=[PASSWORD];database=[DATABASENAME]"
sqlDatabaseTableName=dailyflights
```

As únicas propriedades novas introduzidas nesse arquivo `job.properties` são:

| Propriedade | Origem do valor |
| --- | --- |
| oozie.coord.application.path | Indica o local do arquivo `coordinator.xml` contendo o coordenador do Oozie para execução. |
| hiveDailyTableNamePrefix | O prefixo usado ao criar dinamicamente o nome da tabela de preparo. |
| hiveDataFolderPrefix | O prefixo do caminho no qual todas as tabelas de preparo serão armazenadas. |

## <a name="deploy-and-run-the-oozie-coordinator"></a>Implantar e executar o Coordenador do Oozie

Para executar o pipeline com um coordenador, faça mais ou menos como no caso do fluxo de trabalho, com exceção de trabalhar em uma pasta um nível acima da pasta que contém seu fluxo de trabalho. Essa convenção de pasta separa os coordenadores dos fluxos de trabalho no disco, para que você possa associar um coordenador com fluxos de trabalho filho diferentes.

1. Use o SCP de sua máquina local para copiar os arquivos do coordenador no armazenamento local do nó principal de seu cluster.

    ```bash
    scp ./* sshuser@CLUSTERNAME-ssh.azurehdinsight.net:~
    ```

2. Use SSH para o nó principal.

    ```bash
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Copie os arquivos do coordenador no HDFS.

    ```bash
    hdfs dfs -put ./* /oozie/
    ```

4. Execute o coordenador.

    ```bash
    oozie job -config job.properties -run
    ```

5. Verifique o status usando o Console da Web do Oozie, desta vez selecionando a guia **Trabalhos do Coordenador** e, em seguida, **Todos os trabalhos**.

    ![Trabalhos do Coordenador no Console da Web do Oozie](./media/hdinsight-operationalize-data-pipeline/hdi-oozie-web-console-coordinator-jobs.png)

6. Selecione uma instância do coordenador para exibir a lista de ações agendadas. Nesse caso, você deverá ver quatro ações com tempos nominais no intervalo de 01/01/2017 a 04/01/2017.

    ![Trabalho do Coordenador no Console da Web do Oozie](./media/hdinsight-operationalize-data-pipeline/hdi-oozie-web-console-coordinator-instance.png)

    Cada ação nesta lista corresponde a uma instância do fluxo de trabalho que processa um dia inteiro de dados, em que o início do dia é indicado pelo tempo nominal.

## <a name="next-steps"></a>Próximas etapas

[Documentação do Apache Oozie](https://oozie.apache.org/docs/4.2.0/index.html)
