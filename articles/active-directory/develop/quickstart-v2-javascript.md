---
title: Conectar usuários em aplicativos de página única JavaScript | Azure
titleSuffix: Microsoft identity platform
description: Saiba como um aplicativo JavaScript pode chamar uma API que exige tokens de acesso usando a plataforma de identidade da Microsoft.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 04/11/2019
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:JavaScript
ms.openlocfilehash: 2649d885d31bcd0fef403e7e4eddbc828fd17b93
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/26/2020
ms.locfileid: "80050251"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-a-javascript-spa"></a>Início Rápido: Conectar usuários e obter um token de acesso em um JavaScript SPA

Neste início rápido, você usará um exemplo de código para aprender como um SPA (aplicativo de página única) do JavaScript pode conectar usuários de contas pessoais, contas corporativas e de estudante. Um SPA JavaScript também pode obter um token de acesso para chamar a API do Microsoft Graph ou qualquer API Web. (Confira [Como o exemplo funciona](#how-the-sample-works) para ver uma ilustração.)

## <a name="prerequisites"></a>Pré-requisitos

* Assinatura do Azure - [criar uma gratuitamente](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) (para editar arquivos de projeto)


> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Registrar e baixar o aplicativo de início rápido
> Para iniciar seu aplicativo de início rápido, use qualquer uma das opções a seguir.
>
> ### <a name="option-1-express-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Opção 1 (Expresso): Registrar e configurar o aplicativo automaticamente e, em seguida, baixar seu exemplo de código
>
> 1. Entre no [portal do Azure](https://portal.azure.com) usando uma conta corporativa ou de estudante, ou uma conta pessoal da Microsoft.
> 1. Se sua conta fornecer acesso a mais de um locatário, selecione a conta na parte superior direita e defina sua sessão do portal para o locatário do Azure Active Directory que deseja usar.
> 1. Acesse o novo painel do [portal do Azure – Registros de aplicativo](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs).
> 1. Insira um nome para seu aplicativo. 
> 1. Em **Tipos de conta com suporte**, selecione **Contas em qualquer diretório organizacional e contas pessoais da Microsoft**.
> 1. Selecione **Registrar**.
> 1. Siga as instruções para baixar e configurar automaticamente o novo aplicativo.
>
> ### <a name="option-2-manual-register-and-manually-configure-your-application-and-code-sample"></a>Opção 2 (Manual): Registrar e configurar manualmente o aplicativo e o exemplo de código
>
> #### <a name="step-1-register-your-application"></a>Etapa 1: Registre seu aplicativo
>
> 1. Entre no [portal do Azure](https://portal.azure.com) usando uma conta corporativa ou de estudante, ou uma conta pessoal da Microsoft.
>
> 1. Se sua conta fornecer acesso a mais de um locatário, selecione sua conta na parte superior direita e defina sua sessão do portal para o locatário Azure AD que deseja usar.
> 1. Vá até a página [Registros de aplicativo](https://go.microsoft.com/fwlink/?linkid=2083908) da plataforma de identidade da Microsoft para desenvolvedores.
> 1. Selecione **Novo registro**.
> 1. Quando a página **Registrar um aplicativo** aparecer, insira um nome para o seu aplicativo.
> 1. Em **Tipos de conta com suporte**, selecione **Contas em qualquer diretório organizacional e contas pessoais da Microsoft**.
> 1. Selecione **Registrar**. Na página **Visão geral** do aplicativo, anote o valor de **ID do aplicativo (cliente)** para uso posterior.
> 1. Este início rápido requer que o [fluxo de concessão implícita](v2-oauth2-implicit-grant-flow.md) seja ativado. No painel esquerdo do aplicativo registrado, selecione **Autenticação**.
> 1. Em **Configurações da Plataforma**, selecione **Adicionar uma plataforma**. Um painel será aberto à esquerda. Nele, selecione a região **Aplicativos Web**.
> 1. Ainda à esquerda, defina o valor do **URI de Redirecionamento** como `http://localhost:3000/`. Em seguida, selecione **Token de Acesso** e **Token de ID**.
> 1. Selecione **Configurar**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Etapa 1: Configurar seu aplicativo no portal do Azure
> Para que o exemplo de código deste início rápido funcione, você precisará adicionar um `redirectUri` como `http://localhost:3000/` e habilitar a **Concessão implícita**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Fazer essas alterações para mim]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Já configurado](media/quickstart-v2-javascript/green-check.png) Seu aplicativo já está configurado com esses atributos.

#### <a name="step-2-download-the-project"></a>Etapa 2: Baixe o projeto

> [!div renderon="docs"]
> Para executar o projeto com um servidor Web usando Node.js, [baixe os arquivos de projeto de núcleo](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip).

> [!div renderon="portal"]
> Executar o projeto com um servidor Web usando o Node.js

> [!div renderon="portal" id="autoupdate" class="nextstepaction"]
> [Baixe o exemplo de código](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-javascript-app"></a>Etapa 3: Configurar o aplicativo JavaScript
>
> Na pasta *JavaScriptSPA*, edite *authConfig.js* e defina os valores `clientID`, `authority` e `redirectUri` em `msalConfig`.
>
> ```javascript
>
>  // Config object to be passed to Msal on creation
>  const msalConfig = {
>    auth: {
>      clientId: "Enter_the_Application_Id_Here",
>      authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
>      redirectUri: "Enter_the_Redirect_Uri_Here",
>    },
>    cache: {
>      cacheLocation: "sessionStorage", // This configures where your cache will be stored
>      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
>    }
>  };  
>
>```

> [!div renderon="portal"]
> > [!NOTE]
> > Enter_the_Supported_Account_Info_Here

> [!div renderon="docs"]
>
> Em que:
> - *\<Insira_a_ID_do_Aplicativo_Aqui>* é a **ID do Aplicativo (cliente)** do aplicativo registrado.
> - *\<Insira_a_ID_da_Instância_de_Nuvem_Aqui>* é a instância da nuvem do Azure. Para a nuvem principal ou global do Azure, basta inserir *https://login.microsoftonline.com* . Para nuvens **nacionais** (por exemplo, China), confira [Nuvens nacionais](https://docs.microsoft.com/azure/active-directory/develop/authentication-national-cloud).
> - *\<Insira_as_informações_de_Locatário_aqui>* é definido para uma das seguintes opções:
>    - Se o seu aplicativo for compatível com as *contas neste diretório organizacional*, substitua esse valor pela **ID do locatário** ou **Nome do locatário** (por exemplo, *contoso.microsoft.com*).
>    - Se o aplicativo for compatível com as *contas em qualquer diretório organizacional*, substitua esse valor por **organizações**.
>    - Se o seu aplicativo for compatível com as *contas em qualquer diretório organizacional e contas pessoais da Microsoft*, substitua esse valor por **comum**. Para restringir o suporte a *contas pessoais da Microsoft*, substitua esse valor por **consumidores**.
>
> > [!TIP]
> > Para encontrar os valores de **ID do aplicativo (cliente)** , **ID de diretório (locatário)** e **Tipos de conta com suporte**, vá para a página **Visão Geral** do aplicativo no portal do Azure.
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Etapa 3: seu aplicativo está configurado e pronto para ser executado
> Configuramos seu projeto com os valores das propriedades do seu aplicativo. 

> [!div renderon="docs"]
> 
> Em seguida, ainda na mesma pasta, edite o arquivo *graphConfig.js* para definir o `graphMeEndpoint` e o `graphMeEndpoint` para o objeto `apiConfig`.
> ```javascript
>   // Add here the endpoints for MS Graph API services you would like to use.
>   const graphConfig = {
>     graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me",
>     graphMailEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me/messages"
>   };
>
>   // Add here scopes for access token to be used at MS Graph API endpoints.
>   const tokenRequest = {
>       scopes: ["Mail.Read"]
>   };
> ```
>

> [!div renderon="docs"]
>
> Em que:
> - *\<Insira_o_Ponto_de_Extremidade_Aqui>* é o ponto de extremidade no qual as chamadas à API serão feitas. Para o serviço principal ou global da API do Microsoft Graph, basta inserir `https://graph.microsoft.com`. Para obter mais informações, confira [Implantação em uma nuvem nacional](https://docs.microsoft.com/graph/deployments)
>
> #### <a name="step-4-run-the-project"></a>Etapa 4: Executar o projeto

Execute o projeto com um servidor Web usando o [Node.js](https://nodejs.org/en/download/):

1. Para iniciar o servidor, execute o seguinte comando no diretório do projeto:
    ```batch
    npm install
    npm start
    ```
1. Abra um navegador da Web e vá para `http://localhost:3000/`.

1. Selecione **Entrar** para iniciar a conexão e chame a API do Microsoft Graph.

Depois que o navegador carregar o aplicativo, selecione **Entrar**. Na primeira vez em que entrar, será solicitado que você forneça seu consentimento para permitir que o aplicativo acesse seu perfil e faça logon. Depois que você se conectar com êxito, as informações do seu perfil de usuário deverão ser exibidas na página.

## <a name="more-information"></a>Mais informações

### <a name="how-the-sample-works"></a>Como o exemplo funciona

![Como funciona o aplicativo de exemplo neste início rápido](media/quickstart-v2-javascript/javascriptspa-intro.svg)

### <a name="msaljs"></a>msal.js

A biblioteca MSAL conecta usuários e solicita os tokens que são usados para acessar uma API protegida pela plataforma de identidade da Microsoft. O arquivo *index.html* do início rápido contém uma referência à biblioteca:

```html
<script type="text/javascript" src="https://alcdn.msftauth.net/lib/1.2.1/js/msal.js" integrity="sha384-9TV1245fz+BaI+VvCjMYL0YDMElLBwNS84v3mY57pXNOt6xcUYch2QLImaTahcOP" crossorigin="anonymous"></script>
```
> [!TIP]
> É possível substituir a versão anterior pela mais recente em [versões MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases).

Como alternativa, se tiver instalado o Node.js, você poderá baixar a versão mais recente por meio do npm (Gerenciador de Pacotes do Node.js):

```batch
npm install msal
```

### <a name="msal-initialization"></a>Inicialização da MSAL

O código de início rápido também mostra como inicializar a biblioteca MSAL:

```javascript
  // Config object to be passed to Msal on creation
  const msalConfig = {
    auth: {
      clientId: "75d84e7a-40bx-f0a2-91b9-0c82d4c556aa", // this is a fake id
      authority: "https://login.microsoftonline.com/common",
      redirectUri: "http://localhost:3000/",
    },
    cache: {
      cacheLocation: "sessionStorage", // This configures where your cache will be stored
      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
    }
  };  

const myMSALObj = new Msal.UserAgentApplication(msalConfig);
```

> |Where  |  |
> |---------|---------|
> |`clientId`     | A ID do aplicativo que está registrado no portal do Azure.|
> |`authority`    | (Opcional) A URL da autoridade que dá suporte aos tipos de conta, conforme descrito na seção de configuração. A autoridade padrão é `https://login.microsoftonline.com/common`. |
> |`redirectUri`     | O URI de reposta/redirecionamento configurado do registro de aplicativo. Nesse caso, `http://localhost:3000/`. |
> |`cacheLocation`  | (Opcional) Define o armazenamento do navegador para o estado de autenticação. O padrão é sessionStorage.   |
> |`storeAuthStateInCookie`  | (Opcional) A biblioteca que armazena o estado de solicitação da autenticação exigido para validação dos fluxos de autenticação nos cookies do navegador. Esse cookie é definido para os navegadores IE e Microsoft Edge reduzirem determinados [problemas conhecidos](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues). |

Para saber mais sobre opções configuráveis disponíveis, confira [Inicializar aplicativos cliente](msal-js-initializing-client-applications.md).

### <a name="sign-in-users"></a>Conectar usuários

O snippet de código a seguir mostra como realizar a conexão de usuários:

```javascript
// Add scopes for the id token to be used at Microsoft identity platform endpoints.
const loginRequest = {
    scopes: ["openid", "profile", "User.Read"],
};

myMSALObj.loginPopup(loginRequest)
    .then((loginResponse) => {
    //Login Success callback code here
}).catch(function (error) {
    console.log(error);
});
```

> |Where  |  |
> |---------|---------|
> | `scopes`   | (Opcional) Contém os escopos que estão sendo solicitados para o consentimento do usuário no momento da conexão. Por exemplo, `[ "user.read" ]` para o Microsoft Graph ou `[ "<Application ID URL>/scope" ]` para as APIs Web personalizadas (ou seja, `api://<Application ID>/access_as_user`). |

> [!TIP]
> Como alternativa, talvez você queira usar o método `loginRedirect` para redirecionar a página atual para a página de entrada, em vez de uma janela pop-up.

### <a name="request-tokens"></a>Solicitar tokens

A MSAL usa três métodos para adquirir tokens: `acquireTokenRedirect`, `acquireTokenPopup` e `acquireTokenSilent`

#### <a name="get-a-user-token-silently"></a>Obter um token de usuário no modo silencioso

O método `acquireTokenSilent` manipula as aquisições e a renovação de tokens sem nenhuma interação do usuário. Após o método `loginRedirect` ou `loginPopup` ser executado pela primeira vez, `acquireTokenSilent` é o método normalmente usado para obter tokens usados para acessar recursos protegidos nas próximas chamadas. As chamadas para solicitar ou renovar tokens são feitas no modo silencioso.

```javascript

const tokenRequest = {
    scopes: ["Mail.Read"]
};

myMSALObj.acquireTokenSilent(tokenRequest)
    .then((tokenResponse) => {
        // Callback code here
        console.log(tokenResponse.accessToken);
    }).catch((error) => {
        console.log(error);
    });
```

> |Where  |  |
> |---------|---------|
> | `scopes`   | Contém os escopos solicitados para retorno no token de acesso da API. Por exemplo, `[ "mail.read" ]` para o Microsoft Graph ou `[ "<Application ID URL>/scope" ]` para as APIs Web personalizadas (ou seja, `api://<Application ID>/access_as_user`).|

#### <a name="get-a-user-token-interactively"></a>Obter um token de usuário interativamente

Há situações em que é necessário forçar os usuários a interagir com o ponto de extremidade da plataforma de identidade da Microsoft. Por exemplo:
* Os usuários podem precisar reinserir as credenciais porque a senha expirou.
* Seu aplicativo está solicitando acesso a escopos de recursos adicionais com os quais o usuário precisa concordar.
* A autenticação de dois fatores é necessária.

O padrão usual recomendado para a maioria dos aplicativos é chamar `acquireTokenSilent` primeiro, depois capturar a exceção e, em seguida, chamar `acquireTokenPopup` (ou `acquireTokenRedirect`) para iniciar uma solicitação interativa.

Chamar o `acquireTokenPopup` resulta em uma janela pop-up para entrada. (Ou `acquireTokenRedirect` resulta no redirecionamento dos usuários para o ponto de extremidade da plataforma de identidade da Microsoft). Nessa janela, os usuários precisam interagir confirmando suas credenciais, dando consentimento ao recurso necessário ou concluindo a autenticação de dois fatores.

```javascript
// Add here scopes for access token to be used at MS Graph API endpoints.
const tokenRequest = {
    scopes: ["Mail.Read"]
};

myMSALObj.acquireTokenPopup(requestObj)
    .then((tokenResponse) => {
        // Callback code here
        console.log(tokenResponse.accessToken);
    }).catch((error) => {
        console.log(error);
    });
```

> [!NOTE]
> Este início rápido usa os métodos `loginRedirect` e `acquireTokenRedirect` com o Microsoft Internet Explorer devido a um [problema conhecido](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) relacionado ao tratamento de janelas pop-up pelo Internet Explorer.

## <a name="next-steps"></a>Próximas etapas

Para obter um guia passo a passo mais detalhado sobre como criar aplicativo para este início rápido, confira:

> [!div class="nextstepaction"]
> [Tutorial para entrar e chamar o MS Graph](https://docs.microsoft.com/azure/active-directory/develop/guidedsetups/active-directory-javascriptspa)

Para navegar pelo repositório da MSAL em busca de documentos, perguntas frequentes, problemas e muito mais, confira:

> [!div class="nextstepaction"]
> [Repositório GitHub do MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)

Ajude-nos a melhorar a plataforma de identidade da Microsoft. Deixe sua opinião respondendo a uma breve pesquisa de duas perguntas.

> [!div class="nextstepaction"]
> [Pesquisa da plataforma de identidade da Microsoft](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyKrNDMV_xBIiPGgSvnbQZdUQjFIUUFGUE1SMEVFTkdaVU5YT0EyOEtJVi4u)
