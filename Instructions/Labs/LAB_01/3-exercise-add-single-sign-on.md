---
lab:
  title: Exercício 2 – Adicionar logon único
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercício 2 – Adicionar logon único

Neste exercício, você adicionará o logon único à extensão de mensagem para autenticar consultas de usuário.

![Captura de tela de um desafio de autenticação em uma extensão de mensagem baseada em pesquisa. Um link para entrar é exibido.](../media/2-sign-in.png)

### Duração do exercício

  - **Tempo estimado para conclusão:** 40 minutos

## Tarefa 1 – Configurar o registro do aplicativo da API de back-end

Primeiro, crie um registro de aplicativo do Microsoft Entra para a API de back-end. Para os fins deste exercício, você criará um novo; no entanto, em um ambiente de produção, você usaria um registro de aplicativo existente.

Em uma janela do navegador:

1. Navegue até o [Portal do Azure](https://portal.azure.com).

1. Abra o menu do portal e selecione **Microsoft Entra ID**.

1. Escolha **Gerenciar > Registros de aplicativo** e selecione **Novo registro**.

1. No formulário Registrar um aplicativo, especifique os seguintes valores:

    1. **Nome**: API de produtos

    1. **Tipos de conta de suporte**: contas em qualquer diretório organizacional (Qualquer locatário do Microsoft Entra ID – Multilocatário)

1. Selecione **Registrar** para criar o registro do aplicativo.

1. No menu de registro do aplicativo à esquerda, selecione **Gerenciar > Expor uma API**.

1. Ao lado de **URI da ID do Aplicativo**, selecione **Adicionar** e **Salvar** para criar um novo URI da ID do Aplicativo.

1. Na seção Escopos definidos por esta API, selecione **Adicionar um escopo**.

1. No formulário Adicionar um escopo, especifique os seguintes valores.

    1. **Nome do escopo**: Product.Read

    1. **Quem pode consentir?**: Administradores e usuários

    1. **Nome de exibição do consentimento do administrador**: Ler produtos

    1. **Descrição do consentimento do administrador**: Permite que o aplicativo leia os dados do produto

    1. **Nome de exibição do consentimento do usuário**: Ler produtos

    1. **Descrição do consentimento do usuário**: permite que o aplicativo leia os dados do produto

    1. **Estado**: Habilitado

1. Clique em **Adicionar escopo** para criar o escopo.

Em seguida, anote a ID de registro do aplicativo e a ID do escopo. Você precisa desses valores para configurar o registro do aplicativo usado para obter um token de acesso para a API de back-end.

1. No menu de registro de aplicativo à esquerda, selecione **Manifesto**.

1. Copie o valor da propriedade **appId** e salve-o para uso posterior.

1. Copie o valor da propriedade **oauth2Permissions.id** e salve-o para uso posterior.

Como precisamos desses valores no projeto, adicione-os ao arquivo de ambiente.

No Visual Studio e no projeto **TeamsApp**:

1. Na pasta **env**, abra **.env.local**.

1. No arquivo, adicione as seguintes variáveis de ambiente e defina os valores para a **ID de registro do aplicativo** e a **ID do escopo** que você salvou anteriormente:

    ```text
    BACKEND_API_ENTRA_APP_ID=<app-registration-id>
    BACKEND_API_ENTRA_APP_SCOPE_ID=<scope-id>
    ```

1. Salve suas alterações.

## Tarefa 2 – Criar um arquivo de manifesto de registro de aplicativo para autenticação com a API de back-end

Para autenticar com a API de back-end, você precisa de um registro de aplicativo para obter um token de acesso para chamar a API.

Em seguida, crie um arquivo de manifesto de registro de aplicativo. O manifesto define os escopos de permissão da API e o URI de redirecionamento no registro do aplicativo.

No Visual Studio e no projeto **TeamsApp**:

1. Na pasta **infra\entra**, crie um novo arquivo (<kbd>Ctrl+Shift+A</kbd>) chamado **entra.products.api.manifest.json**.

1. No arquivo , adicione o seguinte código:

    ```json
    {
      "id": "${{PRODUCTS_API_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{PRODUCTS_API_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-product-api-${{TEAMSFX_ENV}}",
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
          "resourceAppId": "${{BACKEND_API_ENTRA_APP_ID}}",
          "resourceAccess": [
            {
              "id": "${{BACKEND_API_ENTRA_APP_SCOPE_ID}}",
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

1. Salve suas alterações.

A propriedade **requiredResourceAccess** especifica a ID de registro do aplicativo e a ID de escopo da API de back-end.

A propriedade **replyUrlsWithType** especifica o URI de redirecionamento usado pelo Serviço de Token do Bot Framework para retornar o token de acesso ao serviço de token após o usuário se autenticar.

Em seguida, atualize o fluxo de trabalho automatizado para criar e atualizar o registro do aplicativo.

No projeto **TeamsApp**:

1. Abra **teamsapp.local.yml**.

1. No arquivo, localize a etapa que usa a ação **aadApp/update**.

1. Após a ação, adicione as ações **aadApp/create** e **aadApp/update** para criar e atualizar o registro do aplicativo (começando na **linha 31**):

    ```yml
      - uses: aadApp/create
        with:
            name: ${{APP_INTERNAL_NAME}}-products-api-${{TEAMSFX_ENV}}
            generateClientSecret: true
            signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
            clientId: PRODUCTS_API_ENTRA_APP_ID
            clientSecret: SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET
            objectId: PRODUCTS_API_ENTRA_APP_OBJECT_ID
            tenantId: PRODUCTS_API_ENTRA_APP_TENANT_ID
            authority: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY
            authorityHost: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
            manifestPath: "./infra/entra/entra.products.api.manifest.json"
            outputFilePath : "./infra/entra/build/entra.products.api.${{TEAMSFX_ENV}}.json"
    ```

1. Salvar suas alterações

A ação **aadApp/create** criará um novo registro de aplicativo com o nome e o público-alvo especificados e gerará um segredo do cliente. A propriedade **writeToEnvironmentFile** gravará a ID de registro do aplicativo, o segredo do cliente, a ID do objeto, a ID do locatário, a autoridade e o host de autoridade nos arquivos do ambiente. O segredo do cliente será criptografado e armazenado com segurança no arquivo **env.local.user**. O nome da variável de ambiente do segredo do cliente é prefixado com **SECRET_**; ele informará ao Kit de ferramenta do Teams para não gravar o valor nos logs.

A ação **aadApp/update** atualizará o registro do aplicativo com o arquivo de manifesto especificado.

## Tarefa 3 – Centralizar o nome da configuração de conexão

Em seguida, centralize o nome da configuração de conexão no arquivo de ambiente e atualize a configuração do aplicativo para acessar o valor da variável de ambiente no runtime.

Continuando no Visual Studio e no projeto **TeamsApp**:

1. Na pasta **env**, abra **.env.local**.

1. No arquivo , adicione o seguinte código:

    ```text
    CONNECTION_NAME=ProductsAPI
    ```

1. Abra **teamsapp.local.yml**.

1. No arquivo, localize a etapa que usa a ação **file/createOrUpdateJsonFile** direcionada ao arquivo **./appsettings.Development.json**. Atualize a matriz de conteúdo para incluir a variável de ambiente **CONNECTION_NAME** e grave o valor no arquivo **appsettings.Development.json**.

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../ProductsPlugin/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Salve suas alterações.

Em seguida, atualize a configuração do aplicativo para acessar a variável de ambiente **CONNECTION_NAME**.

No projeto **ProductsPlugin**:

1. Abra **Config.cs**.

1. Na classe **ConfigOptions**, adicione uma nova propriedade chamada **CONNECTION_NAME**:

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Salve suas alterações.

1. Abra **Program.cs**.

1. No arquivo, atualize o código que lê a configuração do aplicativo para incluir a propriedade **CONNECTION_NAME**:

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["ConnectionName"] = config.CONNECTION_NAME;
    ```

1. Salve suas alterações.

Em seguida, atualize o código do bot para usar o nome da configuração de conexão no tempo de execução.

1. Na pasta **Pesquisar**, abra **SearchApp.cs**.

1. No início da classe **SearchApp**, (mais ou menos na linha 14), crie um construtor que aceite um objeto **IConfiguration** e atribua o valor da propriedade **CONNECTION_NAME** a um campo privado chamado **connectionName**:

    ```csharp
    private readonly string connectionName;
    public SearchApp(IConfiguration configuration)
    {
      connectionName = configuration["CONNECTION_NAME"];
    }  
    ```

1. Salve suas alterações.

## Tarefa 4 – Definir a configuração de conexão da API de Produtos

Para autenticar com a API de back-end, você precisa definir uma configuração de conexão no recurso do Bot do Azure.

Continuando no Visual Studio e no projeto **TeamsApp**:

1. Na pasta **infra**, abra **azure.parameters.local.json**.

1. No arquivo, adicione os parâmetros **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** e **connectionName**:

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
        "backendApiEntraAppClientId": {
          "value": "${{BACKEND_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientId": {
          "value": "${{PRODUCTS_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientSecret": {
          "value": "${{SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Salve suas alterações.

Em seguida, atualize o arquivo Bicep para incluir os novos parâmetros e passá-los para o recurso de Bot do Azure.

1. Na pasta **infra**, abra o arquivo chamado **azure.local.bicep**.

1. No arquivo, após a declaração do parâmetro **botAppDomain**, adicione as declarações dos parâmetros **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** e **connectionName**:

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. Na declaração do módulo **azureBotRegistration**, adicione os novos parâmetros:

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botEntraAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        backendApiEntraAppClientId: backendApiEntraAppClientId
        productsApiEntraAppClientId: productsApiEntraAppClientId
        productsApiEntraAppClientSecret: productsApiEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Salve suas alterações.

Por fim, atualize o arquivo Bicep de registro do bot para incluir a nova configuração de conexão.

1. Na pasta **infra/botRegistration**, abra **azurebot.bicep**.

1. No arquivo, após a declaração do parâmetro **botAppDomain**, adicione as declarações dos parâmetros **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** e **connectionName** :

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. No arquivo, adicione um novo recurso chamado **botServicesProductsApiConnection** ao final do arquivo:

    ```bicep
    resource botServicesProductsApiConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: productsApiEntraAppClientId
        clientSecret: productsApiEntraAppClientSecret
        scopes: 'api://${backendApiEntraAppClientId}/Product.Read'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${botEntraAppClientId}'
          }
        ]
      }
    }
    ```

1. Salve suas alterações.

## Tarefa 5 – Configurar a autenticação na extensão de mensagem

Para autenticar consultas de usuário na extensão de mensagem, use o SDK do Bot Framework para obter um token de acesso para o usuário pelo Serviço de Token do Bot Framework. O token de acesso pode ser usado para acessar dados a partir de um serviço externo.

Para simplificar o código, crie uma classe auxiliar que processe a autenticação do usuário.

Continuando no Visual Studio e no projeto **ProductsPlugin**:

1. Crie uma pasta chamada **Auxiliares**.

1. Na pasta **Auxiliares**, crie um novo arquivo de classe chamado **AuthHelpers.cs**

1. No arquivo , adicione o seguinte código:

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    using Microsoft.Bot.Schema.Teams;
    internal static class AuthHelpers
    {
        internal static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
        {
            var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);
            return new MessagingExtensionResponse
            {
                ComposeExtension = new MessagingExtensionResult
                {
                    Type = "auth",
                    SuggestedActions = new MessagingExtensionSuggestedAction
                    {
                        Actions = [
                            new() {
                                Type = ActionTypes.OpenUrl,
                                Value = resource.SignInLink,
                                Title = "Sign In",
                            },
                        ],
                    },
                },
            };
        }
        internal static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
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
        internal static bool HasToken(TokenResponse tokenResponse) => tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
    }
    ```

1. Salve suas alterações.

Os três métodos auxiliares na classe **AuthHelpers** manipulam a autenticação do usuário na extensão de mensagem.

- O método **CreateAuthResponse** constrói uma resposta que renderiza um link de entrada na interface do usuário. O link de entrada é recuperado no serviço de token usando o método **GetSignInResourceAsync**.

- O método **GetToken** usa o cliente do serviço de token para obter um token de acesso para o usuário atual. O método usa um código mágico para verificar a autenticidade da solicitação.

- O método **HasToken** verifica se a resposta do serviço de token contém um token de acesso. Se o token não for nulo ou vazio, o método retornará true.

Em seguida, atualize o código de extensão de mensagem para usar os métodos auxiliares para autenticar consultas de usuário.

1. Na pasta **Pesquisar**, abra **SearchApp.cs**.

1. Na parte superior do arquivo, adicione a seguinte instrução using:

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione o seguinte código no início do método:

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

1. Salve suas alterações.

Em seguida, adicione o domínio do Serviço de Token ao arquivo de manifesto do aplicativo para garantir que o cliente possa confiar no domínio ao iniciar um fluxo de logon único.

No projeto **TeamsApp**:

1. Na pasta **appPackage**, abra o arquivo **manifest.json**.

1. No arquivo, atualize a matriz **validDomains** e adicione o domínio do serviço de token:

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]
    ```

1. Salve suas alterações.

## Tarefa 6 – Criar e atualizar recursos

Feito tudo isso, execute o processo **Preparar dependências do aplicativo Teams** para criar novos recursos e atualizar os já existentes.

> [!NOTE]
> Se o provisionamento não conseguir preparar as dependências, verifique se você tem os valores corretos para **BACKEND_API_ENTRA_APP_ID** e **BACKEND_API_ENTRA_APP_SCOPE_ID** em **env.local**.

Continuando no Visual Studio:

1. No **Gerenciador de Soluções**, clique com o botão direito do mouse no projeto **TeamsApp**.

1. Expanda o menu **Kit de Ferramentas do Teams** e selecione **Preparar dependências do aplicativo Teams**.

1. Na caixa de diálogo **Conta do Microsoft 365**, clique em **Continuar**.

1. Na caixa de diálogo **Provisionar**, clique em **Provisionar**.

1. Na caixa de diálogo de **aviso do Kit de Ferramentas do Teams**, clique em **Provisionar**.

1. Na caixa de diálogo de **informações do Kit de Ferramentas do Teams**, clique no ícone de cruz para fechar a caixa de diálogo.

## Tarefa 7 – Executar e depurar

Com os recursos provisionados, inicie uma sessão de depuração para testar a extensão da mensagem.

1. Para iniciar uma nova sessão de depuração, pressione <kbd>F5</kbd> ou clique em **Iniciar** na barra de ferramentas.

1. Aguarde até que uma janela do navegador seja aberta e a caixa de diálogo de instalação do aplicativo apareça no cliente Web do Microsoft Teams. Se solicitado, insira as credenciais da sua conta do Microsoft 365.

1. Na caixa de diálogo de instalação do aplicativo, selecione **Adicionar**.

1. Abra um bate-papo novo ou já existente do Microsoft Teams.

1. Na área de composição da mensagem, comece a digitar **/apps** para abrir o seletor de aplicativo.

1. Na lista de aplicativos, selecione **Produtos da Contoso** para abrir a extensão de mensagem.

1. Na caixa de texto, insira **oi**. Pode ser necessário inserir sua pesquisa várias vezes.

1. A mensagem **Você precisará entrar para usar este aplicativo** será exibida:

    ![Captura de tela de um desafio de autenticação em uma extensão de mensagem baseada em pesquisa. Um link para entrar é exibido.](../media/2-sign-in.png)

1. Siga o link de **login** para iniciar o fluxo de autenticação.

1. Concorde com as permissões solicitadas e retorne ao Microsoft Teams:

    ![Captura de tela da caixa de diálogo de consentimento de permissão da API do Microsoft Entra.](../media/18-api-permission-consent.png)

1. Aguarde a conclusão da pesquisa e a exibição dos resultados.

1. Na lista de resultados, clique em **oi** para inserir um cartão na caixa de redigir mensagem.

Retorne ao Visual Studio e selecione **Parar** na barra de ferramentas ou pressione <kbd>Shift</kbd> + <kbd>F5</kbd> para interromper a sessão de depuração.

[Continue no próximo exercício...](./4-exercise-return-data-api.md)