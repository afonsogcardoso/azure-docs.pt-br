---
title: Vários clusters HDInsight & uma conta de armazenamento do Lago de Dados Azure
description: Saiba como usar mais de um cluster HDInsight com uma única conta do Data Lake Storage
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/18/2019
ms.openlocfilehash: cc67acca11e7e0f24dc0597dcd19672a38a7bf28
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75495751"
---
# <a name="use-multiple-hdinsight-clusters-with-an-azure-data-lake-storage-account"></a>Usar múltiplos clusters HDInsight com uma conta do Azure Data Lake Storage

Começando com o HDInsight versão 3.5, você pode criar clusters HDInsight com contas de armazenamento do Azure Data Lake Storage como o sistema de arquivos padrão.
O Data Lake Storage dá suporte a armazenamento ilimitado, o que o torna ideal não apenas para hospedagem de grandes quantidades de dados, mas também para hospedar vários clusters HDInsight que compartilham uma única conta do Data Lake Storage. Para obter instruções sobre como criar um cluster HDInsight com o Data Lake Storage como armazenamento, consulte [Quickstart: Configure clusters no HDInsight](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md).

Este artigo fornece recomendações para o administrador do Data Lake Storage configurar uma única conta compartilhada do Data Lake Storage que pode ser usada em vários clusters HDInsight **ativos**. Essas recomendações se aplicam à hospedagem de vários clusters Apache Hadoop seguros e também não seguros em uma conta compartilhada do Data Lake Storage.

## <a name="data-lake-storage-file-and-folder-level-acls"></a>ACLs no nível de pasta e de arquivo do Data Lake Storage

O restante deste artigo pressupõe que você tenha um bom conhecimento de ACLs no nível de arquivo e de pasta do Azure Data Lake Storage, o que é descrito em detalhes no [Controle de acesso no Azure Data Lake Storage](../data-lake-store/data-lake-store-access-control.md).

## <a name="data-lake-storage-setup-for-multiple-hdinsight-clusters"></a>Configuração do Data Lake Storage para vários clusters HDInsight

Vamos tomar uma hierarquia de pastas de dois níveis para explicar as recomendações para usar vários clusters HDInsight com uma conta data lake storage. Considere que você tem uma conta do Data Lake Storage com a estrutura de pasta **/clusters/finance**. Com essa estrutura, todos os clusters requeridos pela organização Finance podem usar /clusters/finance como o local de armazenamento. No futuro, se outra organização, digamos, Marketing, quiser criar clusters HDInsight que usem a mesma conta do Data Lake Storage, ela poderá criar clusters/marketing. Por enquanto, usaremos apenas **/clusters/finance**.

Para habilitar essa estrutura de pasta para ser efetivamente usada pelos clusters HDInsight, o administrador do Data Lake Storage deve atribuir permissões apropriadas, conforme descrito na tabela. As permissões mostradas na tabela correspondem a ACLs de acesso e ACLs não padrão.

|Pasta  |Permissões  |Usuário proprietário  |Grupo proprietário  | Usuário nomeado | Permissões de usuário nomeado | Grupo nomeado | Permissões de grupo nomeado |
|---------|---------|---------|---------|---------|---------|---------|---------|
|/ | rwxr-x--x  |administrador |administrador  |Entidade de serviço |--x  |FINGRP   |r-x         |
|/clusters | rwxr-x--x |administrador |administrador |Entidade de serviço |--x  |FINGRP |r-x         |
|/clusters/finance | rwxr-x--t |administrador |FINGRP  |Entidade de serviço |rwx  |-  |-     |

Na tabela,

- **Admin** é o criador e o administrador da conta do Data Lake Storage.
- **Entidade de serviço** é a entidade de serviço do Azure Active Directory (AAD) associada à conta.
- **FINGRP** é um grupo de usuários criado no AAD que contém os usuários da organização Finance.

Para obter instruções sobre como criar um aplicativo do AAD (que também cria uma entidade de serviço), consulte [Criar um aplicativo do AAD](../active-directory/develop/howto-create-service-principal-portal.md#create-an-azure-active-directory-application). Para obter instruções sobre como criar um grupo de usuários no AAD, consulte [Gerenciamento de grupos no Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Veja alguns pontos principais a considerar.

- A estrutura de pastas dois níveis (**/clusters/finance/**) deve ser criada e provisionada com permissões apropriadas pelo administrador do Data Lake Storage **antes da** conta de armazenamento ser usada para clusters. Esta estrutura não é criada automaticamente durante a criação de clusters.
- O exemplo acima recomenda que se configure o grupo proprietário de **/clusters/finance** como **FINGRP** e se conceda acesso **r-x** a FINGRP para a hierarquia de pastas inteira, começando da raiz. Isso assegura que os membros de FINGRP possam navegar pela estrutura de pastas começando da raiz.
- No caso em que entidades de serviço do AAD diferentes podem criar clusters em **/clusters/finance**, o sticky bit (quando definido na pasta **finance**) assegura que as pastas criadas por uma entidade de serviço não possam ser excluídas pela outra.
- Uma vez que a estrutura da pasta e as permissões estejam no lugar, o processo de criação de clusters HDInsight cria um local de armazenamento específico para clusters em **/clusters/finance/**. Por exemplo, o armazenamento de um cluster com o nome fincluster01 poderia ser **/clusters/finance/fincluster01**. A propriedade e as permissões para as pastas criadas pelo cluster HDInsight são mostrados na tabela apresentada aqui.

    |Pasta  |Permissões  |Usuário proprietário  |Grupo proprietário  | Usuário nomeado | Permissões de usuário nomeado | Grupo nomeado | Permissões de grupo nomeado |
    |---------|---------|---------|---------|---------|---------|---------|---------|
    |/clusters/finanace/ fincluster01 | rwxr-x---  |Entidade de Serviço |FINGRP  |- |-  |-   |-  |

## <a name="recommendations-for-job-input-and-output-data"></a>Recomendações para dados de entrada e saída do trabalho

É recomendável que os dados de entrada para um trabalho e as saídas de um trabalho sejam armazenados em uma pasta fora de **/clusters**. Isso garante que, mesmo que a pasta específica de cluster seja excluída para recuperar algum espaço de armazenamento, as entradas e saídas de trabalho ainda estão disponíveis para uso futuro. Nesse caso, verifique se a hierarquia de pastas para armazenar as entradas e saídas de trabalho permite o nível de acesso apropriado para a entidade de serviço.

## <a name="limit-on-clusters-sharing-a-single-storage-account"></a>Limite de clusters compartilhando uma única conta de armazenamento

O limite no número de clusters que podem compartilhar uma única conta do Data Lake Storage depende da carga de trabalho sendo executada em um desses clusters. Ter clusters demais ou cargas de trabalho muito pesadas nos clusters que compartilham uma conta de armazenamento pode fazer com que a entrada/saída de conta de armazenamento seja limitada.

## <a name="support-for-default-acls"></a>Suporte para ACLs padrão

Ao criar uma entidade de serviço com acesso de usuário nomeado (conforme mostrado na tabela acima), é recomendável **não** adicionar o usuário nomeado com uma ACL padrão. O provisionamento de acesso de usuário nomeado usando ACLs padrão resulta na atribuição de 770 permissões para o usuário proprietário, o grupo proprietário e outros. Embora este valor padrão de 770 não retire as permissões do usuário próprio (7) ou do próprio grupo (7), ele tira todas as permissões para outros (0). Isso resulta em um problema conhecido com um caso de uso específico, que é abordado em detalhes na seção [Problemas conhecidos e soluções alternativas](#known-issues-and-workarounds).

## <a name="known-issues-and-workarounds"></a>Problemas conhecidos e limitações

Esta seção lista os problemas conhecidos para usar o HDInsight com Data Lake Storage e suas soluções alternativas.

### <a name="publicly-visible-localized-apache-hadoop-yarn-resources"></a>Recursos do Apache Hadoop YARN localizados e visíveis publicamente

Quando uma nova conta do Azure Data Lake Storage é criada, o diretório raiz é provisionado automaticamente com os bits de permissão de ACL de acesso definidos como 770. O usuário proprietário da pasta raiz é definido como o usuário que criou a conta (o administrador do Data Lake Storage) e o grupo proprietário é definido como o grupo primário do usuário que criou a conta. Nenhum acesso é fornecido para "outros".

Sabe-se que essas configurações afetam um caso de uso específico do HDInsight capturado no [YARN 247](https://hwxmonarch.atlassian.net/browse/YARN-247). Envios de trabalho podem falhar com uma mensagem de erro semelhante a esta:

    Resource XXXX is not publicly accessible and as such cannot be part of the public cache.

Conforme declarado no JIRA YARN vinculado anteriormente, ao localizar recursos públicos, o localizador valida que todos os recursos solicitados estão realmente públicos verificando as permissões deles no sistema de arquivos remoto. Qualquer Recurso Local que não se encaixe nessa condição é rejeitado para localização. Para verificar se há permissões, o acesso de leitura para o arquivo é incluído para "outros". Esse cenário não funciona fora da caixa ao hospedar clusters HDInsight no Azure Data Lake, já que o Azure Data Lake nega todo o acesso a "outros" no nível de pasta raiz.

#### <a name="workaround"></a>Solução alternativa

Defina permissões de leitura-execução **outros** por meio da hierarquia, por exemplo, em **/**, **/clusters** e **/clusters/finance**, conforme mostrado na tabela acima.

## <a name="see-also"></a>Confira também

- [Início rápido: Configurar clusters no HDInsight](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)
- [Use o Azure Data Lake Storage Gen2 com clusters Azure HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md)
