---
title: Amostras da CLI
description: Encontre amostras da CLI do Azure para alguns dos cenários comuns do Serviço de Aplicativo. Saiba como automatizar suas tarefas de implantação ou gerenciamento do Serviço de Aplicativo.
tags: azure-service-management
ms.assetid: 53e6a15a-370a-48df-8618-c6737e26acec
ms.topic: sample
ms.date: 12/12/2017
ms.custom: mvc
ms.openlocfilehash: 0b3acb1b421962cde7d90398f42bdfeefda578e3
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/24/2020
ms.locfileid: "74971495"
---
# <a name="cli-samples-for-azure-app-service"></a>Exemplos de CLI para o Serviço de Aplicativo do Azure

A tabela a seguir inclui links para bash scripts criados usando a CLI do Azure.

| | |
|-|-|
|**Como criar o aplicativo**||
| [Criar um aplicativo e implantar arquivos com FTP](./scripts/cli-deploy-ftp.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo de Serviço de Aplicativo e implanta um arquivo usando FTP. |
| [Como criar um aplicativo e implantar o código do GitHub](./scripts/cli-deploy-github.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo de Serviço de Aplicativo e implanta o código de um repositório do GitHub público. |
| [Criar um aplicativo com a implantação contínua do GitHub](./scripts/cli-continuous-deployment-github.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo de Serviço de Aplicativo com a publicação contínua de um repositório do GitHub que você tem. |
| [Criar um aplicativo e implantar o código de um repositório Git local](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo e configura o push de código de um repositório Git local. |
| [Criar um aplicativo e implantar o código em um ambiente de preparo](./scripts/cli-deploy-staging-environment.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo com um slot de implantação para alterações de código de preparo. |
| [Criar um aplicativo do ASP.NET Core em um contêiner do Docker](./scripts/cli-linux-docker-aspnetcore.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo de Serviço de Aplicativo no Linux e carrega uma imagem do Docker do Docker Hub. |
|**Como configurar o aplicativo**||
| [Mapear um domínio personalizado para um aplicativo](./scripts/cli-configure-custom-domain.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo do Serviço de Aplicativo e mapeia um nome de domínio personalizado para ele. |
| [Associar um certificado SSL personalizado a um aplicativo](./scripts/cli-configure-ssl-certificate.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo do Serviço de Aplicativo e associa o certificado SSL de um nome de domínio personalizado a ele. |
|**Como escalar um aplicativo**||
| [Dimensionar manualmente um aplicativo](./scripts/cli-scale-manual.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo e dimensiona-o em duas instâncias. |
| [Escalar um aplicativo em todo o mundo com uma arquitetura de alta disponibilidade](./scripts/cli-scale-high-availability.md?toc=%2fcli%2fazure%2ftoc.json) | Cria dois aplicativos do Serviço de Aplicativo em duas regiões geográficas diferentes e disponibiliza-os por meio de um único ponto de extremidade usando o Gerenciador de Tráfego do Azure. |
|**Proteger o aplicativo**||
| [Integrar com o Gateway de Aplicativo do Azure](./scripts/cli-integrate-app-service-with-application-gateway.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo e o integra com o Gateway de Aplicativo usando restrições de acesso e ponto de extremidade de serviço. |
|**Como conectar o aplicativo aos recursos**||
| [Conectar um aplicativo a um Banco de Dados SQL](./scripts/cli-connect-to-sql.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo do Serviço de Aplicativo e um Banco de Dados SQL e, em seguida, adiciona a cadeia de conexão de banco de dados às configurações do aplicativo. |
| [Conectar um aplicativo a uma conta de armazenamento](./scripts/cli-connect-to-storage.md?toc=%2fcli%2fazure%2ftoc.json)| Cria um aplicativo do Serviço de Aplicativo e uma conta de armazenamento e, em seguida, adiciona a cadeia de conexão de armazenamento às configurações do aplicativo. |
| [Conectar um aplicativo a um Cache do Azure para Redis](./scripts/cli-connect-to-redis.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo de Serviço de Aplicativo e um Cache do Azure para Redis e, em seguida, adiciona detalhes da conexão redis às configurações do aplicativo. |
| [Conectar um aplicativo ao Cosmos DB](./scripts/cli-connect-to-documentdb.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo de Serviço de Aplicativo e um Cosmos DB e, em seguida, adiciona os detalhes da conexão do BD Cosmos às configurações do aplicativo. |
|**Aplicativo de backup e restauração**||
| [Fazer backup de um aplicativo](./scripts/cli-backup-onetime.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo e um backup avulso para ele. |
| [Criar um backup agendado para um aplicativo](./scripts/cli-backup-scheduled.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo e um backup agendado para ele. |
| [Restaura um aplicativo usando um backup](./scripts/cli-backup-restore.md?toc=%2fcli%2fazure%2ftoc.json) | Restaura um aplicativo de Serviço de Aplicativo usando um backup. |
|**Como monitorar o aplicativo**||
| [Monitorar um aplicativo com logs do servidor Web](./scripts/cli-monitor.md?toc=%2fcli%2fazure%2ftoc.json) | Cria um aplicativo do Serviço de Aplicativo, habilita o log para ele e baixa os logs no computador local. |
| | |