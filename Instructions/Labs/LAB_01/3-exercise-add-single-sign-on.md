---
lab:
  title: Exercício 2 – Adicionar logon único
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercício 2 – Adicionar logon único

Neste exercício, você vai atualizaa a extensão de mensagem para que os usuários sejam solicitados a entrar e autenticar. Você vai configurar o registro do aplicativo de bot do Microsoft Entra e o manifesto do aplicativo para habilitar o logon único. Você vai configurar um registro de aplicativo do Microsoft Entra para autenticar com o Microsoft Graph e atualizar a lógica de extensão de mensagem para obter um token de acesso usando o Serviço de token Bot Framework. Depois, você vai executar e depurar a extensão de mensagem para testar no Microsoft Teams.

## Tarefa 1 – Configurar o logon único

Primeiro, configure um registro do aplicativo de bot do Microsoft Entra.

No Visual Studio:

1. Na pasta **infra\entra**, abra o arquivo chamado **entra.bot.manifest.json**
1. No arquivo, atualize a matriz **identifierUris** para definir o URI da ID do aplicativo

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ],
    ```

1. No arquivo, atualize a matriz **oauth2Permissions** para criar um escopo para permitir que o Teams chame APIs da Web como administrador ou usuário:

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. No arquivo, atualize a matriz **preAuthorizedApplications** para adicionar o Microsoft Teams, Microsoft Outlook e Copilot para clientes do Microsoft 365 à lista de clientes autorizados:

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ],
    ```

1. Salvar suas alterações

Em seguida, atualize o arquivo de manifesto do aplicativo para definir o recurso que o cliente deve usar ao iniciar um fluxo de logon único no aplicativo.

Continuando no Visual Studio:

1. Na pasta **appPackage**, abra o arquivo chamado **manifest.json**
1. No arquivo, adicione o seguinte código após a **descrição**:

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    },
    ```

1. Salvar suas alterações

## Tarefa 2 – Criar arquivo de manifesto de registro do aplicativo do Microsoft Entra para o Microsoft Graph

Para autenticar com o Microsoft Graph, crie um novo arquivo de manifesto de registro do aplicativo.

Continuando no Visual Studio:

1. Na pasta **infra\entra**, crie um arquivo chamado **entra.graph.manifest.json**
2. No arquivo , adicione o seguinte código:

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [
        {
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. Salvar suas alterações

A matriz **requiredResourceAccess** define os escopos de permissão da API e a matriz **replyUrlsWithType** define os URIs de redirecionamento no registro do aplicativo.

Agora atualize o arquivo de projeto com ações para criar o registro do aplicativo.

1. Na pasta raiz do projeto, abra **teamsapp.local.yml**
1. No arquivo, localize a etapa que usa a ação **arm/deploy**
1. Antes da etapa, adicione o código a seguir:

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. Salvar suas alterações

## Tarefa 3 – Criar configuração de conexão OAuth

As configurações de conexão do Serviço de Bot de IA do Azure são usadas para gerenciar a autenticação do usuário em bots e extensões de mensagem.

Primeiro, centralize o nome da configuração de conexão OAuth usada para criar a configuração de conexão como uma variável de ambiente e, em seguida, adicione código para usar o valor da variável de ambiente em tempo de execução.

Continuando no Visual Studio:  

1. Na pasta **env**, abra **.env.local**
1. No arquivo , adicione o seguinte código:

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. Na pasta raiz do projeto, abra o arquivo chamado **teamsapp.local.yml**
1. No arquivo, localize a etapa que usa a ação **file/createOrUpdateJsonFile** direcionada ao arquivo **./appsettings.Development.json**
1. Atualize a matriz de conteúdo, adicionando a variável **CONNECTION_NAME**

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Salvar suas alterações
1. Na pasta raiz do projeto, abra o arquivo **Config.cs**
1. Na classe **ConfigOptions**, adicione uma nova propriedade de cadeia de caracteres com o nome **CONNECTION_NAME**

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Salvar suas alterações
1. Na pasta raiz do projeto, abra o arquivo chamado **Program.cs**
1. No arquivo, adicione uma nova linha para adicionar a variável de ambiente **CONNECTION_NAME** como uma definição de configuração do aplicativo:

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

Em seguida, atualize o Manipulador de Atividades do Bot para acessar a configuração do aplicativo.

1. Na pasta **Pesquisar**, abra o arquivo chamado **SearchApp.cs**
1. Na classe **SearchApp**, crie uma propriedade de cadeia de caracteres somente leitura com o nome **connectionName**.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. Crie um construtor, que injeta a configuração do aplicativo e define o valor da propriedade connectionName.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. Salvar suas alterações

Em seguida, atualize os arquivos Bicep para provisionar a configuração de conexão OAuth.

Primeiro, atualize o arquivo de parâmetros para passar as credenciais do registro do aplicativo Microsoft Entra do Microsoft Graph e o nome da configuração de conexão.

1. Na pasta **infra**, abra o arquivo chamado **azure.parameters.local.json**
1. No objeto **parameters**, adicione os parâmetros **graphEntraAppClientId**, **graphEntraAppClientSecret** e **connectionName**

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        },
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Salvar suas alterações

Depois, atualize o arquivo Bicep.

1. Na pasta **infra**, abra o arquivo chamado **azure.local.bicep**
1. No arquivo, após a declaração de parâmetro **botAppDomain**, adicione as declarações de parâmetro **graphEntraAppClientId**, **graphEntraAppClientSecret** e **connectionName**

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. No módulo **azureBotRegistration**, adicione os parâmetros **graphEntraAppClientId**, **graphEntraAppClientSecret** e **connectionName**

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Salve suas alterações.

Por fim, atualize o arquivo Bicep de registro do bot.

1. Na pasta **infra/botRegistration**, abra o arquivo chamado **azurebot.bicep**
1. No arquivo, após a declaração de parâmetro **botAppDomain**, adicione as declarações de parâmetro **graphEntraAppClientId**, **graphEntraAppClientSecret** e **connectionName**

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. Após o recurso **botServiceM365ExtensionsChannel** , adicione um novo recurso para a conexão do Microsoft Graph

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. Salvar suas alterações

## Tarefa 4 – Autenticar consultas do usuário

Em seguida, adicione código para autenticar usuários quando eles iniciarem uma pesquisa usando a extensão de mensagem.

Continuando no Visual Studio:

1. Na pasta **Pesquisar**, abra o arquivo chamado **SearchApp.cs**
1. No arquivo, comece adicionando o namespace **Autenticação** a partir do SDK do Bot Framework.

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione o seguinte código no início do método:

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

O bloco de códigos acima usa três métodos que gerenciam a autenticação do usuário.

- **GetToken** usa o cliente do serviço de token para obter um token de acesso para o usuário atual
- **HasToken** verifica se a resposta do serviço de token contém um token de acesso
- **CreateAuthResponse** é chamado se nenhum token for retornado e retorna uma resposta, que exibe um link de entrada na interface do usuário

Agora, crie os métodos na classe **SearchApp**.

- Crie o método **GetToken** usando o seguinte código:

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
{
  var magicCode = string.Empty;

  if (!string.IsNullOrEmpty(state))
  {
    if (int.TryParse(state, out var parsed)) 
    {
        magicCode = parsed.ToString();
    }
  }

  return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
}
```

Primeiro verifique se o parâmetro de estado não é nulo ou vazio. O código tentará analisá-lo como um número inteiro usando o método int.TryParse. Se a análise for bem-sucedida, o valor analisado será atribuído à variável magicCode como uma cadeia de caracteres. A variável magicCode então é passada como um argumento para o método GetUserTokenAsync junto com outros parâmetros. O método GetUserTokenAsync usa a variável magicCode para verificar a autenticidade da solicitação, garantindo que o token de usuário seja solicitado pela mesma entidade que iniciou o processo de autenticação.

- Crie o método HasToken usando o código a seguir:

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

Verifique se um token válido é obtido do serviço de token verificando se a resposta do token e a propriedade Token na resposta não é vazia ou nula.

- Crie o método **CreateAuthResponse** usando o seguinte código:

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

Primeiro, use o método **GetSignInResourceAsync** para recuperar o link de entrada do serviço de token. O link de entrada é usado para construir um objeto **MessagingExtensionResponse**. Crie um novo objeto e defina a propriedade **ComposeExtension** da resposta para um novo objeto **MessagingExtensionResult**. A propriedade type do resultado é definida como "auth", indicando que o resultado é uma resposta de autenticação. A propriedade **SuggestedActions** do resultado é definida como um novo objeto **MessagingExtensionSuggestedAction**. A propriedade Actions das ações sugeridas é definida como uma lista que contém um único objeto **CardAction**. Esse objeto **CardAction** representa uma ação que pode ser executada pelo usuário. A propriedade Type de **CardAction** é definida como **ActionTypes.OpenUrl**, indicando que é uma ação que abre uma URL. A propriedade Value é definida como o link de entrada recuperado do recurso. A propriedade Title é definida como "Entrada", que especifica o título da ação. Por fim, a resposta construída é retornada do método.

Ao seguir o link de entrada, o usuário chega a um recurso hospedado em um domínio externo. O domínio deve ser incluído no arquivo de manifesto do aplicativo. Adicione o domínio do Serviço de Token do Bot Framework ao manifesto do aplicativo.

Continuando no Visual Studio:

1. Na pasta **appPackage**, abra o arquivo **manifest.json**
1. No arquivo, atualize a matriz **validDomains** e adicione o domínio do serviço de token:

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. Salvar suas alterações

## Tarefa 5 – Provisionar os recursos

Agora com tudo pronto, execute o processo Preparar dependências do aplicativo do Teams para provisionar os recursos necessários.

Continuando no Visual Studio:

1. No **Gerenciador de Soluções**, clique com o botão direito do mouse no projeto **TeamsApp**.
1. Expanda o menu Kit de Ferramentas** do Teams**, selecione **Preparar dependências do aplicativo Teams**
1. Na caixa de diálogo **Conta do Microsoft 365**, selecione **Continuar**
1. Na caixa de diálogo **Provisionar**, selecione **Provisionar**
1. Na caixa de diálogo de **aviso do Kit de Ferramentas do Teams**, selecione **Provisionar**
1. Na caixa de diálogo de **informações do kit de ferramentas do Microsoft Teams**, selecione **Exibir recursos provisionados** para abrir uma nova janela do navegador.

Tire um momento para explorar os recursos criados e atualizados no Azure.

## Tarefa 6 – Executar e depurar

Agora inicie o serviço Web e teste a extensão de mensagem no Microsoft Teams.

Continuando no Visual Studio:

1. Aperte **F5** para iniciar uma sessão de depuração e abrir uma nova janela do navegador que navega até o cliente Web do Microsoft Teams.
1. No navegador e, se necessário, insira suas credenciais da conta do Microsoft 365 e continue no Microsoft Teams.
1. Na caixa de diálogo de instalação do aplicativo, selecione **Adicionar**
1. Abra um bate-papo novo ou já existente do Microsoft Teams
1. Na área de redigir mensagem, selecione **...** para abrir o submenu do aplicativo
1. Na lista de aplicativos, selecione **produtos da Contoso** para abrir a extensão de mensagem
1. Na caixa de texto, digite **Bot Builder** para iniciar uma pesquisa
1. A mensagem mensagem **Você precisará entrar para usar este aplicativo** será exibida
1. Selecione o **link de entrada** para abrir uma nova guia e iniciar o fluxo de autenticação
1. Na página de consentimento de permissões, revise as permissões que estão sendo solicitadas.
1. Selecione **Aceitar** para fechar a guia e retornar ao Microsoft Teams
1. Na lista de resultados, **clique em um resultado** para inserir um cartão na caixa de redigir mensagem e envie.

Feche o navegador para interromper a sessão de depuração.

[Continue no próximo exercício...](./4-exercise-retrieve-product-information-from-sharepoint-online.md)