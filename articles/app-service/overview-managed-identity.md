---
title: Identidades gerenciadas
description: Saiba como as identidades gerenciadas funcionam no Azure App Service e nas Funções Azure, como configurar uma identidade gerenciada e gerar um token para um recurso back-end.
author: mattchenderson
ms.topic: article
ms.date: 03/04/2020
ms.author: mahender
ms.reviewer: yevbronsh
ms.openlocfilehash: 6e3169f2bfcba0a02af1490f875cbab8a14d02f6
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79280021"
---
# <a name="how-to-use-managed-identities-for-app-service-and-azure-functions"></a>Como usar identidades gerenciadas para o Serviço de Aplicativo e o Azure Functions

> [!Important] 
> Identidades gerenciadas para Serviço de Aplicativo e Azure Functions não se comportarão conforme o esperado se seu aplicativo for migrado entre assinaturas/locatários. O aplicativo precisará obter uma nova identidade, que pode ser feita ao desabilitar e reabilitar o recurso. Consulte [removendo uma identidade](#remove) abaixo. Recursos de downstream também precisará ter políticas de acesso atualizadas para usar a nova identidade.

Este tópico mostra como criar uma identidade gerenciada para aplicativos do Serviço de Aplicativo e do Azure Functions e como usá-la para acessar outros recursos. Uma identidade gerenciada do Azure Active Directory (AAD) permite que seu aplicativo acesse facilmente outros recursos protegidos por AAD, como o Azure Key Vault. A identidade é gerenciada pela plataforma do Azure e não exige provisionamento ou giro de nenhum segredo. Para obter mais informações sobre identidades gerenciadas no AAD, veja [Identidades gerenciadas para recursos do Azure](../active-directory/managed-identities-azure-resources/overview.md).

Seu aplicativo pode receber dois tipos de identidades: 
- Uma **identidade atribuída pelo sistema** é vinculada ao seu aplicativo e é excluída se o seu aplicativo for excluído. Um aplicativo só pode ter uma identidade atribuída pelo sistema.
- Uma **identidade atribuída pelo usuário** é um recurso autônomo do Azure que pode ser atribuído ao seu aplicativo. Um aplicativo pode ter várias identidades atribuídas pelo usuário.

## <a name="add-a-system-assigned-identity"></a>Adicionar uma identidade atribuída ao sistema

Criar um aplicativo com uma identidade designada pelo sistema requer uma propriedade adicional a ser definida no aplicativo.

### <a name="using-the-azure-portal"></a>Usando o portal do Azure

Para configurar uma identidade gerenciada no portal, primeiro, crie um aplicativo como normal e, em seguida, habilite o recurso.

1. Crie um aplicativo no portal, como você faria normalmente. Navegue até ele no portal.

2. Se você estiver usando um aplicativo de funções, navegue até os **recursos da Plataforma**. Para outros tipos de aplicativo, role para baixo até o grupo **Configurações** no painel de navegação à esquerda.

3. Selecione **Identidade**.

4. Na guia **Sistema atribuído**, alterne o **Status** para **Ligado**. Clique em **Salvar**.

    ![Identidade gerenciada no Serviço de Aplicativo](media/app-service-managed-service-identity/system-assigned-managed-identity-in-azure-portal.png)

### <a name="using-the-azure-cli"></a>Usando a CLI do Azure

Para configurar uma identidade gerenciada usando a CLI do Azure, será preciso usar o comando `az webapp identity assign` em um aplicativo existente. Você tem três opções para executar os exemplos nesta seção:

- Usar o [Azure Cloud Shell](../cloud-shell/overview.md) do portal do Azure.
- Use o Azure Cloud Shell incorporado através do botão "Try It", localizado no canto superior direito de cada bloco de código abaixo.
- [Instale a versão mais recente da CLI do Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) (2.0.31 ou mais recente) se você preferir usar um console da CLI local. 

As etapas a seguir o guiarão na criação de um aplicativo Web e na atribuição de uma identidade a ele usando a CLI:

1. Se você estiver usando a CLI do Azure em um console local, primeiro entre no Azure usando o [logon az](/cli/azure/reference-index#az-login). Use uma conta associada à assinatura do Azure a qual você gostaria de implantar o aplicativo:

    ```azurecli-interactive
    az login
    ```
2. Crie um aplicativo Web usando a CLI. Para ver mais exemplos de como usar a CLI com o Serviço de Aplicativo, consulte [Exemplos de CLI do Serviço de Aplicativo](../app-service/samples-cli.md):

    ```azurecli-interactive
    az group create --name myResourceGroup --location westus
    az appservice plan create --name myPlan --resource-group myResourceGroup --sku S1
    az webapp create --name myApp --resource-group myResourceGroup --plan myPlan
    ```

3. Execute o comando `identity assign` para criar a identidade para este aplicativo:

    ```azurecli-interactive
    az webapp identity assign --name myApp --resource-group myResourceGroup
    ```

### <a name="using-azure-powershell"></a>Usando o PowerShell do Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

As etapas a seguir guiarão você pela criação de um aplicativo da Web e pela atribuição de uma identidade usando o Azure PowerShell:

1. Se necessário, instale o Azure PowerShell usando as instruções encontradas `Login-AzAccount` no [guia Azure PowerShell](/powershell/azure/overview)e execute para criar uma conexão com o Azure.

2. Crie um aplicativo da Web usando o Azure PowerShell. Para obter mais exemplos de como usar o Azure PowerShell com o Serviço de Aplicativo, consulte [ Amostras do PowerShell do Serviço de Aplicativo ](../app-service/samples-powershell.md):

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name myResourceGroup -Location $location
    
    # Create an App Service plan in Free tier.
    New-AzAppServicePlan -Name $webappname -Location $location -ResourceGroupName myResourceGroup -Tier Free
    
    # Create a web app.
    New-AzWebApp -Name $webappname -Location $location -AppServicePlan $webappname -ResourceGroupName myResourceGroup
    ```

3. Execute o comando `Set-AzWebApp -AssignIdentity` para criar a identidade para este aplicativo:

    ```azurepowershell-interactive
    Set-AzWebApp -AssignIdentity $true -Name $webappname -ResourceGroupName myResourceGroup 
    ```

### <a name="using-an-azure-resource-manager-template"></a>Usando um modelo do Azure Resource Manager

Um modelo do Azure Resource Manager pode ser usado para automatizar a implantação de recursos do Azure. Para saber mais sobre a implantação do Serviço de Aplicativo e do Azure Functions, consulte [Automatizar a implantação de recursos no Serviço de Aplicativo](../app-service/deploy-complex-application-predictably.md) e [Automatizar a implantação de recursos no Azure Functions](../azure-functions/functions-infrastructure-as-code.md).

Qualquer recurso do tipo `Microsoft.Web/sites` pode ser criado com uma identidade, incluindo a propriedade a seguir na definição de recurso:
```json
"identity": {
    "type": "SystemAssigned"
}    
```

> [!NOTE] 
> Um aplicativo pode ter identidades atribuídas pelo sistema e atribuídas pelo usuário ao mesmo tempo. Nesse caso, a `type` propriedade seria `SystemAssigned,UserAssigned`

Adicionar o tipo atribuído pelo sistema diz ao Azure para criar e gerenciar a identidade do seu aplicativo.

Por exemplo, um aplicativo Web pode ser semelhante ao seguinte:
```json
{
    "apiVersion": "2016-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "SystemAssigned"
    },
    "properties": {
        "name": "[variables('appName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "hostingEnvironment": "",
        "clientAffinityEnabled": false,
        "alwaysOn": true
    },
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
    ]
}
```

Quando o site é criado, ele tem as seguintes propriedades adicionais:
```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

A propriedade tenantId identifica a qual locatário do AAD a identidade pertence. O principalId é um identificador exclusivo para a nova identidade do aplicativo. No AAD, a entidade de serviço tem o mesmo nome que você deu à sua instância do Serviço de Aplicativo ou das Funções do Azure.


## <a name="add-a-user-assigned-identity"></a>Adicionar uma identidade atribuída pelo usuário

Criar um aplicativo com uma identidade atribuída pelo usuário exige que você crie a identidade e, em seguida, adicione seu identificador de recursos à configuração do aplicativo.

### <a name="using-the-azure-portal"></a>Usando o portal do Azure

Primeiro, você precisará criar um recurso de identidade atribuído pelo usuário.

1. Crie um recurso de identidade gerenciado atribuído pelo usuário de acordo com [estas instruções](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

2. Crie um aplicativo no portal, como você faria normalmente. Navegue até ele no portal.

3. Se você estiver usando um aplicativo de funções, navegue até os **recursos da Plataforma**. Para outros tipos de aplicativo, role para baixo até o grupo **Configurações** no painel de navegação à esquerda.

4. Selecione **Identidade**.

5. Na guia **atribuída ao Usuário,** clique **em Adicionar**.

6. Procure a identidade que você criou anteriormente e selecione-a. Clique em **Adicionar**.

    ![Identidade gerenciada no Serviço de Aplicativo](media/app-service-managed-service-identity/user-assigned-managed-identity-in-azure-portal.png)

### <a name="using-an-azure-resource-manager-template"></a>Usando um modelo do Azure Resource Manager

Um modelo do Azure Resource Manager pode ser usado para automatizar a implantação de recursos do Azure. Para saber mais sobre a implantação do Serviço de Aplicativo e do Azure Functions, consulte [Automatizar a implantação de recursos no Serviço de Aplicativo](../app-service/deploy-complex-application-predictably.md) e [Automatizar a implantação de recursos no Azure Functions](../azure-functions/functions-infrastructure-as-code.md).

Qualquer recurso do tipo `Microsoft.Web/sites` pode ser criado com uma identidade incluindo o seguinte bloco na definição do recurso, substituindo `<RESOURCEID>` pelo ID do recurso da identidade desejada:
```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}    
```

> [!NOTE] 
> Um aplicativo pode ter identidades atribuídas pelo sistema e atribuídas pelo usuário ao mesmo tempo. Nesse caso, a `type` propriedade seria `SystemAssigned,UserAssigned`

A adição do tipo atribuído pelo usuário diz ao Azure para usar a identidade atribuída pelo usuário especificada para o seu aplicativo.

Por exemplo, um aplicativo Web pode ser semelhante ao seguinte:
```json
{
    "apiVersion": "2016-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
        }
    },
    "properties": {
        "name": "[variables('appName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "hostingEnvironment": "",
        "clientAffinityEnabled": false,
        "alwaysOn": true
    },
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
    ]
}
```

Quando o site é criado, ele tem as seguintes propriedades adicionais:
```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

O principalId é um identificador único para a identidade usada para a administração AAD. O clientId é um identificador exclusivo para a nova identidade do aplicativo que é usado para especificar qual identidade usar durante chamadas de tempo de execução.


## <a name="obtain-tokens-for-azure-resources"></a>Obter tokens para recursos do Azure

Um aplicativo pode usar sua identidade gerenciada para obter tokens para acessar outros recursos protegidos pela AAD, como o Azure Key Vault. Esses tokens representam o acesso do aplicativo ao recurso e não um usuário específico do aplicativo. 

Talvez seja necessário configurar o recurso de destino para permitir o acesso do aplicativo. Por exemplo, se você solicitar um token para acessar o Key Vault, você precisa ter certeza de que adicionou uma política de acesso que inclua a identidade do seu aplicativo. Caso contrário, as chamadas para o Key Vault serão rejeitadas, mesmo se elas incluírem o token. Para saber mais sobre os recursos que oferecem suporte a tokens do Azure Active Directory, veja [Serviços do Azure que dão suporte à autenticação do Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

> [!IMPORTANT]
> Os serviços back-end para identidades gerenciadas mantêm um cache por recurso URI por cerca de 8 horas. Se você atualizar a política de acesso de um recurso de destino específico e recuperar imediatamente um token para esse recurso, você poderá continuar a obter um token em cache com permissões desatualizadas até que esse token expire. No momento, não há como forçar uma atualização de tokens.

Há um protocolo REST simples para obter um token no Serviço de Aplicativo e no Azure Functions. Isso pode ser usado para todas as aplicações e idiomas. Para .NET e Java, o Azure SDK fornece uma abstração sobre este protocolo e facilita uma experiência de desenvolvimento local.

### <a name="using-the-rest-protocol"></a>Usar o protocolo REST

Um aplicativo com uma identidade gerenciada tem duas variáveis de ambiente definidas:

- MSI_ENDPOINT: a URL para o serviço de token local.
- MSI_SECRET: um cabeçalho usado para ajudar a reduzir os ataques de falsificação da solicitação do lado do servidor (SSRF). O valor é trocado pela plataforma.

A **MSI_ENDPOINT** é uma URL local da qual o aplicativo pode solicitar tokens. Para obter um token para um recurso, solicite uma HTTP GET para esse ponto de extremidade, incluindo os seguintes parâmetros:

> |Nome do parâmetro|No|Descrição|
> |-----|-----|-----|
> |recurso|Consulta|O URI do recurso do AAD do recurso para o qual um token deve ser obtido. Pode ser um dos [serviços do Azure que dão suporte à autenticação do Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) ou a qualquer outro URI de recurso.|
> |api-version|Consulta|A versão da API do token a ser usada. Atualmente, a única versão com suporte é 2017-09-01.|
> |segredo|Cabeçalho|O valor da variável de ambiente MSI_SECRET. Esse cabeçalho é usado para ajudar a reduzir os ataques de falsificação da solicitação do lado do servidor (SSRF).|
> |clientid|Consulta|(Opcional, a menos que seja atribuído pelo usuário) O ID da identidade atribuída pelo usuário a ser usada. Se omitido, a identidade atribuída pelo sistema é usada.|

> [!IMPORTANT]
> Se você está tentando obter tokens para identidades atribuídas `clientid` pelo usuário, você deve incluir a propriedade. Caso contrário, o serviço de token tentará obter um token para uma identidade atribuída ao sistema, que pode ou não existir.

Uma resposta bem-sucedida de 200 OK inclui um corpo JSON com as seguintes propriedades:

> |Nome da propriedade|Descrição|
> |-------------|----------|
> |access_token|O token de acesso solicitado. O serviço Web de chamada pode usar esse token para se autenticar no serviço Web de recebimento.|
> |expires_on|A hora de expiração do token de acesso. A data é representada como o número de segundos de 1970-01-01T0:0:0Z UTC até a hora de expiração. Esse valor é usado para determinar o tempo de vida de tokens em cache.|
> |recurso|O URI da ID de aplicativo do serviço Web de recebimento.|
> |token_type|Indica o valor do tipo de token. O único tipo com suporte do Azure AD é Portador Para saber mais sobre os tokens de portador, consulte [Estrutura de Autorização do OAuth 2.0: Uso do Token de Portador (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt).|

Essa resposta é igual a [resposta à solicitação do AAD de token de acesso de serviço para serviço](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md#get-a-token).

> [!NOTE]
> Variáveis de ambiente são configuradas quando o processo é iniciado pela primeira vez, então após habilitar a identidade gerenciada para o seu aplicativo, você pode precisar reiniciar o aplicativo ou reimplantar o código dele antes que `MSI_ENDPOINT` e `MSI_SECRET` estejam disponíveis para seu código.

### <a name="rest-protocol-examples"></a>Exemplos de protocolo REST

Uma solicitação de exemplo pode ser semelhante ao seguinte:

```
GET /MSI/token?resource=https://vault.azure.net&api-version=2017-09-01 HTTP/1.1
Host: localhost:4141
Secret: 853b9a84-5bfa-4b22-a3f3-0b9a43d9ad8a
```

Uma resposta de exemplo pode ser semelhante ao seguinte:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "access_token": "eyJ0eXAi…",
    "expires_on": "09/14/2017 00:00:00 PM +00:00",
    "resource": "https://vault.azure.net",
    "token_type": "Bearer"
}
```

### <a name="code-examples"></a>Exemplos de código

# <a name="net"></a>[.NET](#tab/dotnet)

> [!TIP]
> Para as linguagens .NET, também é possível usar [Microsoft.Azure.Services.AppAuthentication](#asal) em vez de criar essa solicitação por conta própria.

```csharp
private readonly HttpClient _client;
// ...
public async Task<HttpResponseMessage> GetToken(string resource)  {
    var request = new HttpRequestMessage(HttpMethod.Get, 
        String.Format("{0}/?resource={1}&api-version=2017-09-01", Environment.GetEnvironmentVariable("MSI_ENDPOINT"), resource));
    request.Headers.Add("Secret", Environment.GetEnvironmentVariable("MSI_SECRET"));
    return await _client.SendAsync(request);
}
```

# <a name="javascript"></a>[Javascript](#tab/javascript)

```javascript
const rp = require('request-promise');
const getToken = function(resource, cb) {
    let options = {
        uri: `${process.env["MSI_ENDPOINT"]}/?resource=${resource}&api-version=2017-09-01`,
        headers: {
            'Secret': process.env["MSI_SECRET"]
        }
    };
    rp(options)
        .then(cb);
}
```

# <a name="python"></a>[Python](#tab/python)

```python
import os
import requests

msi_endpoint = os.environ["MSI_ENDPOINT"]
msi_secret = os.environ["MSI_SECRET"]

def get_bearer_token(resource_uri):
    token_auth_uri = f"{msi_endpoint}?resource={resource_uri}&api-version=2017-09-01"
    head_msi = {'Secret':msi_secret}

    resp = requests.get(token_auth_uri, headers=head_msi)
    access_token = resp.json()['access_token']

    return access_token
```

# <a name="powershell"></a>[Powershell](#tab/powershell)

```powershell
$resourceURI = "https://<AAD-resource-URI-for-resource-to-obtain-token>"
$tokenAuthURI = $env:MSI_ENDPOINT + "?resource=$resourceURI&api-version=2017-09-01"
$tokenResponse = Invoke-RestMethod -Method Get -Headers @{"Secret"="$env:MSI_SECRET"} -Uri $tokenAuthURI
$accessToken = $tokenResponse.access_token
```

---

### <a name="using-the-microsoftazureservicesappauthentication-library-for-net"></a><a name="asal"></a>Usar a biblioteca de Microsoft.Azure.Services.AppAuthentication do .NET

Para aplicativos e funções .NET, a maneira mais simples de trabalhar com uma identidade gerenciada é por meio do pacote Microsoft.Azure.Services.AppAuthentication. Essa biblioteca também permitirá que você teste seu código localmente em sua máquina de desenvolvimento, usando sua conta de usuário do Visual Studio, a [Azure CLI](/cli/azure) ou a Autenticação Integrada do Active Directory. Para obter mais informações sobre as opções de desenvolvimento local com essa biblioteca, consulte a [Referência Microsoft.Azure.Services.AppAuthentication]. Esta seção mostra a você como começar a usar a biblioteca no seu código.

1. Adicione referências a [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) e a qualquer outro pacote NuGet necessário para seu aplicativo. O exemplo abaixo também usa [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault).

2. Adicione o seguinte código ao seu aplicativo, modificando-o para ter como destino o recurso correto. Este exemplo mostra duas maneiras de trabalhar com o Azure Key Vault:

    ```csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;
    // ...
    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://vault.azure.net");
    // OR
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
    ```

Para saber mais sobre o Microsoft.Azure.Services.AppAuthentication e as operações que ele expõe, consulte a [Referência Microsoft.Azure.Services.AppAuthentication] e [Serviço de Aplicativo e KeyVault com a amostra MSI .NET](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

### <a name="using-the-azure-sdk-for-java"></a>Usando o Azure SDK para Java

Para aplicações e funções Java, a maneira mais simples de trabalhar com uma identidade gerenciada é através do [Azure SDK for Java](https://github.com/Azure/azure-sdk-for-java). Esta seção mostra a você como começar a usar a biblioteca no seu código.

1. Adicione uma referência à [biblioteca Azure SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure). Para projetos Maven, você pode adicionar `dependencies` este trecho à seção do arquivo POM do projeto:

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure</artifactId>
        <version>1.23.0</version>
    </dependency>
    ```

2. Use `AppServiceMSICredentials` o objeto para autenticação. Este exemplo mostra como esse mecanismo pode ser usado para trabalhar com o Azure Key Vault:

    ```java
    import com.microsoft.azure.AzureEnvironment;
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.keyvault.Vault
    //...
    Azure azure = Azure.authenticate(new AppServiceMSICredentials(AzureEnvironment.AZURE))
            .withSubscription(subscriptionId);
    Vault myKeyVault = azure.vaults().getByResourceGroup(resourceGroup, keyvaultName);

    ```


## <a name="remove-an-identity"></a><a name="remove"></a>Remova uma identidade

Uma identidade atribuída pelo sistema pode ser removida desabilitando o recurso usando o portal, PowerShell ou CLI da mesma forma que foi criado. As identidades atribuídas pelo usuário podem ser removidas individualmente. Para remover todas as identidades, defina o tipo como "Nenhum" no [modelo ARM](#using-an-azure-resource-manager-template):

```json
"identity": {
    "type": "None"
}
```

Remover uma identidade atribuída pelo sistema dessa maneira também a excluirá do AAD. As identidades atribuídas pelo sistema também são automaticamente removidas do AAD quando o recurso do aplicativo é excluído.

> [!NOTE]
> Há também uma configuração de aplicativo que pode ser definida, WEBSITE_DISABLE_MSI, que apenas desativa o serviço de token local. No entanto, ele deixa a identidade no local e ferramentas ainda mostrará a identidade gerenciada como "ligada" ou "habilitada". Como resultado, o uso dessa configuração não é recomendado.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Acesse o Banco de Dados SQL com segurança usando uma identidade gerenciada](app-service-web-tutorial-connect-msi.md)

[Referência Microsoft.Azure.Services.AppAuthentication]: https://go.microsoft.com/fwlink/p/?linkid=862452
