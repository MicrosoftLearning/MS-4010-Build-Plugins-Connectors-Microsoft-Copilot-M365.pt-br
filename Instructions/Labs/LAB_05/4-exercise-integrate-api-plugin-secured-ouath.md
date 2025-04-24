---
lab:
  title: Exercício 3 — Integrar um plug-in de API a uma API protegida com o OAuth
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Exercício 3 — Integrar um plug-in de API a uma API protegida com o OAuth

Os plug-ins de API para Microsoft 365 Copilot permitem que você se integre a APIs protegidas com OAuth. Você mantém a ID do cliente e o segredo do aplicativo que protege sua API seguros, registrando-os no cofre do Teams. Em runtime, o Microsoft 365 Copilot executa seu plug-in, obtém as informações do cofre e as usa para obter um token de acesso e chamar a API. Seguindo esse processo, o ID do cliente e o segredo permanecem seguros e nunca são expostos ao cliente.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 – abrir o projeto de exemplo e examinar a configuração de autenticação

Comece baixando o projeto de exemplo:

1. Em um navegador da Web, acesse [https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs). Você recebe uma solicitação para baixar um arquivo ZIP com o projeto de exemplo.
1. Salve o arquivo ZIP no seu computador.
1. Extraia o conteúdo do arquivo ZIP.
1. Abra a pasta  no Visual Studio Code.

O projeto de exemplo é um projeto do Kit de Ferramentas do Teams que inclui um agente declarativo, um plug-in de API e uma API protegida com o Microsoft Entra ID. A API está em execução no Azure Functions e implementa segurança usando os recursos internos de autenticação e autorização do Azure Functions, às vezes chamados de Easy Auth.

## Tarefa 2 — examinar a configuração de autorização do OAuth2

Antes de continuar, examine a configuração de autorização do OAuth2 no projeto de exemplo.

### Examinar a definição de API

Primeiro, dê uma olhada na configuração de segurança da definição de API incluída no projeto.

No Visual Studio Code:

1. Abra o arquivo **appPackage/apiSpecificationFile/repair.yml**.
1. Na seção **components.securitySchemes**, observe a propriedade **oAuth2AuthCode**:

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  A propriedade define um esquema de segurança OAuth2 e inclui informações sobre as URLs a serem chamadas para obter um token de acesso e quais escopos a API usa.

  > **IMPORTANTE** Observe que o escopo é totalmente qualificado com a URI da ID do aplicativo (**api://...**). Ao trabalhar com o Microsoft Entra, você precisa qualificar totalmente os escopos personalizados. Quando o Microsoft Entra vê um escopo não qualificado, ele pressupõe que pertence ao Microsoft Graph, o que leva a erros de fluxo de autorização.

1. Localize a propriedade **paths./repairs.get.security**. Observe que ele faz referência ao esquema de segurança **oAuth2AuthCode** e ao escopo de que o cliente precisa para executar a operação.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **IMPORTANTE** Listar os escopos necessários na especificação da API é puramente informativo. Ao implementar a API, você é responsável por validar o token e verificar se ele contém os escopos necessários.

### Examinar a implementação da API

Em seguida, dê uma olhada na implementação da API.

No Visual Studio Code:

1. Abra o arquivo **src/functions/repairs.ts**.
1. Na função manipuladora de **reparos**, localize a seguinte linha, que verifica se a solicitação contém um token de acesso com os escopos necessários:

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. A função **hasRequiredScopes** é implementada ainda mais no arquivo **repairs.ts**:

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  A função começa extraindo o token de portador do cabeçalho da solicitação de autorização. Em seguida, ele usa o pacote **jwt-decode** para decodificar o token e obter a lista de escopos da declaração **scp**. Por fim, ele verifica se a declaração **scp** contém todos os escopos necessários.

  Observe que a função não está validando o token de acesso. Em vez disso, ele verifica apenas se o token de acesso contém os escopos necessários. Neste modelo, a API está em execução no Azure Functions e implementa a segurança usando o Easy Auth, que é responsável por validar o token de acesso. Se a solicitação não contiver um token de acesso válido, o runtime do Azure Functions o rejeitará antes que ele chegue ao seu código. Embora o Easy Auth valide o token, ele não verifica os escopos necessários. Você mesmo tem que fazer isso.

### Examinar a configuração da tarefa do cofre

Neste projeto, você usa o Kit de Ferramentas do Teams para adicionar as informações do OAuth ao cofre. O Kit de Ferramentas do Teams registra as informações do OAuth no cofre usando uma tarefa especial na configuração do projeto.

No Visual Studio Code:

1. Abra o arquivo **./teampsapp.local.yml**.
1. Na seção de **provisionamento**, localize a tarefa **oauth/register**.

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  A tarefa pega os valores das variáveis de projeto **TEAMS_APP_ID**, **AAD_APP_CLIENT_ID** e **SECRET_AAD_APP_CLIENT_SECRET** , armazenados nos arquivos **env/.env.local** e **env/.env.local.user** e os registra no cofre. Também habilita a chave de prova do Code Exchange (PKCE) como medida de segurança extra. Em seguida, ele pega o ID de entrada do cofre e o grava no arquivo de ambiente **env/.env.local**. O resultado dessa tarefa é uma variável de ambiente chamada **OAUTH2AUTHCODE_CONFIGURATION_ID**. O Kit de Ferramentas do Teams grava o valor dessa variável no arquivo **appPackages/ai-plugin.json** que contém a definição do plug-in. No runtime, o agente declarativo que carrega o plug-in da API usa essa ID para obter as informações do OAuth do cofre e iniciar um fluxo de autenticação para obter um token de acesso.

  > **IMPORTANTE** A tarefa **oauth/register** só é responsável por registrar as informações do OAuth no cofre se elas ainda não existirem. Se as informações já existirem, o Kit de Ferramentas do Teams ignora a execução dessa tarefa.

1. Em seguida, localize a tarefa **oauth/update** .

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  A tarefa mantém as informações do OAuth no cofre sincronizadas com o seu projeto. É necessário que seu projeto funcione corretamente. Uma das principais propriedades é a URL na qual seu plug-in de API está disponível. Cada vez que você inicia o seu projeto, o Kit de Ferramentas do Teams abre um túnel de desenvolvimento em uma nova URL. As informações do OAuth no cofre precisam fazer referência a essa URL para que o Copilot possa acessar a sua API.

### Examinar a configuração de autenticação e autorização

A próxima parte a ser explorada são as configurações de autenticação e autorização do Azure Functions. A API neste exercício usa os recursos internos de autenticação e autorização do Azure Functions. O Kit de Ferramentas do Teams configura esses recursos durante o provisionamento do Azure Functions para o Azure.

No Visual Studio Code:

1. Abra o arquivo **infra/azure.bicep**.
1. Localize o recurso **authSettings**:

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  Esse recurso habilita os recursos internos de autenticação e autorização no aplicativo Azure Functions. Primeiro, na seção **globalValidation**, ele define que o aplicativo permite apenas solicitações autenticadas. Se o aplicativo receber uma solicitação não autenticada, ele a rejeita com um erro HTTP 401. Em seguida, na seção **identityProviders**, a configuração define que ele usa o Microsoft Entra ID (anteriormente conhecido como Azure Active Directory) para autorizar solicitações. Especifica qual registro de aplicativo do Microsoft Entra ele usa para proteger a API e quem tem permissão para chamar a API.

### Examinar registro de aplicativo do Microsoft Entra

A parte final a ser examinada é o registro de aplicativo do Microsoft Entra que o projeto usa para proteger a API. Ao usar o OAuth, você protege o acesso aos recursos usando um aplicativo. O aplicativo normalmente define as credenciais necessárias para obter um token de acesso, tais como um segredo do cliente ou um certificado. Também especifica as diferentes permissões (também conhecidas como escopos) que o cliente pode solicitar ao chamar a API. O registro de aplicativo do Microsoft Entra representa um aplicativo na nuvem da Microsoft e define um aplicativo para uso com fluxos de autorização OAuth.

No Visual Studio Code:

1. Abra o arquivo **./aad.manifest.json**.
1. Localize a propriedade **oauth2Permissions**.

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  A propriedade define um escopo personalizado chamado **repairs_read** para conceder permissão ao cliente para ler reparos a partir da API de reparos.

1. Localize a propriedade **identifierUris**.

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  A propriedade **identifierUris** define um identificador que é usado para qualificar totalmente o escopo.
