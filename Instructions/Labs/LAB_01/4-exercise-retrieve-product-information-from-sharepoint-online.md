---
lab:
  title: Exercício 3 – Recuperar informações do produto no SharePoint Online
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercício 3 – Recuperar informações do produto no SharePoint Online

Neste exercício, você vai provisionar e configurar um site do SharePoint Online, que armazena informações sobre produtos como itens em uma lista. Você vai atualizar o código de extensão de mensagem para recuperar os itens de lista no SharePoint Online usando o SDK do Microsoft Graph e retornar os dados de item de lista nos resultados da pesquisa. Por fim, você vai executar e depurar sua extensão de mensagem e testá-la no Microsoft Teams.

![Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams. Os resultados da pesquisa são retornados do SharePoint Online. Cada resultado da pesquisa exibe o nome, a categoria e a imagem do produto.](../media/4-search-results-sharepoint-online.png)

## Tarefa 1 – Provisionar e configurar o site do SharePoint de marketing de produto

Comece criando um site do SharePoint Online usando o serviço de livro de aparência do SharePoint.

Em um navegador da Web:

1. Vá para **livro de aparência do SharePoint** em [https://lookbook.microsoft.com](https://lookbook.microsoft.com)
1. Na navegação superior, expanda **Exibir os designs**
1. No menu **Exibir os designs**, expanda **Equipe** e selecione **Suporte ao Produto**
1. Selecione **Adicionar locatário**
1. Se solicitado, entre no seu locatário.
1. Na tela de consentimento de permissão, revise as permissões necessárias e escolha **Aceitar** para retornar ao serviço de livro de aparência do SharePoint.
1. No formulário, aceite os padrões e selecione **Provisar**

Você receberá um email no seu endereço de e-mail notificando quando o provisionamento do site for concluído. A conclusão desse processo pode levar alguns minutos.

![Captura de tela da página inicial do site da equipe do SharePoint Online de suporte ao produto. Uma lista de produtos lançados recentemente é mostrada.](../media/1-sharepoint-online-product-support-site.png)

Para habilitar a filtragem nas colunas Título e Categoria de varejo ao consultar a lista usando a API do Microsoft Graph, crie índices na lista.

Continuando no navegador da Web:

1. Vá para o site de **Suporte ao produto** em **<https://tenant.sharepoint.com/sites/productmarketing>** e substitua **locatário** pelo nome da sua instância do SharePoint Online
1. Na barra do **pacote do Microsoft 365**, selecione a **engrenagem de configurações** para abrir o painel lateral Configurações.
1. No cabeçalho **SharePoint**, selecione **Conteúdos do site**
1. Na lista de Listas e Bibliotecas, passe o mouse sobre a lista **Produtos**, selecione o ícone **vertical de três pontos** para expandir o menu **Mostrar ações** e selecione **Configurações**.
1. Na seção **Colunas**, na lista de colunas, selecione **Colunas indexadas**
1. Selecione **Criar um novo índice**

## Tarefa 2 – Adicionar variáveis de ambiente do nome do host  do SharePoint e URL do site

Em seguida, centralizamos o nome do host da instância do SharePoint Online e a URL do site de suporte ao produto como variáveis de ambiente. Em seguida, exponha os valores como uma variável de ambiente a ser usada em tempo de execução e atualize o código para ler o valor.

Abra o Visual Studio:

1. Na pasta **env**, abra o arquivo chamado **.env.local**
1. No arquivo, adicione as variáveis de ambiente **SPO_HOSTNAME** e **SPO_SITE_URL**, substituindo **locatário** pelo nome da instância do SharePoint Online:

    ```text
    SPO_HOSTNAME=tenant.sharepoint.com
    SPO_SITE_URL=sites/productmarketing
    ```

1. Salvar suas alterações

Em seguida, atualize a ação para gravar as variáveis de ambiente no arquivo de configurações do aplicativo.

1. Na pasta raiz do projeto, abra o arquivo chamado **teamsapp.local.yml**
1. Localize a etapa que usa a ação **file/createOrUpdateJsonFile**, que tem como alvo o arquivo **./appsettings.Development.json**
1. No arquivo, atualize a matriz de **conteúdo**, adicionando as variáveis **SPO_HOSTNAME** e **SPO_SITE_URL** :

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
            SPO_HOSTNAME: ${{SPO_HOSTNAME}}
            SPO_SITE_URL: ${{SPO_SITE_URL}}
    ```

1. Salvar suas alterações

Agora, atualize a classe ConfigOptions para incluir as novas variáveis de ambiente

1. Na pasta raiz do projeto, abra o arquivo Config.cs
1. Na classe ConfigOptions, adicione novas propriedades de cadeia de caracteres com os nomes SPO_HOSTNAME e SPO_SITE_URL

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
      public string SPO_HOSTNAME { get; set; }
      public string SPO_SITE_URL { get; set; }
    }
    ```

1. Salvar suas alterações

Em seguida, atualize a configuração do aplicativo com as duas variáveis de ambiente.

1. No pasta raiz do projeto, abra Program.cs.
1. Adicione novas linhas para adicionar as variáveis de ambiente SPO_HOSTNAME e SPO_SITE_URL como definições de configuração do aplicativo.

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    builder.Configuration["SPO_HOSTNAME"] = config.SPO_HOSTNAME;
    builder.Configuration["SPO_SITE_URL"] = config.SPO_SITE_URL;
    ```

1. Salvar suas alterações

A última etapa é atualizar o Manipulador de Atividades do Bot para ler os valores da configuração do aplicativo e armazená-los em propriedades de somente leitura.

1. Na pasta Pesquisar, abra o arquivo chamado SearchApp.cs
1. Na classe SearchApp, crie propriedades de cadeia de caracteres somente leitura com os nomes spoHostname e spoSiteUrl

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
      private readonly string spoHostname;
      private readonly string spoSiteUrl;
    }
    ```

1. Atualize o construtor para definir os valores de propriedade usando a configuração do aplicativo injetado:

    ```csharp
    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    } 
    ```

1. Salve suas alterações.

## Tarefa 3 – Atualizar o comando de pesquisa

À medida que a extensão de mensagem retorna informações do produto, atualize o título e a descrição do comando de pesquisa, o nome do parâmetro e sua descrição.

Continuando no Visual Studio:

1. Na pasta **appPackage**, abra o arquivo chamado **manifest.json**
1. Na matriz **composeExtensions**, atualize o objeto de comando com:

    ```json
    "composeExtensions": [
      {
        "botId": "${{BOT_ID}}",
        "commands": [
          {
            "id": "Search",
            "type": "query",
            "title": "Products",
            "description": "Find products by name",
            "initialRun": false,
            "fetchTask": false,
            "context": [
              "commandBox",
              "compose",
              "message"
            ],
            "parameters": [
              {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
              }
            ]
          }
        ]
      }
    ],
    ```

1. Salvar suas alterações

## Tarefa 4 – Obter o valor de consulta do usuário

Quando o método OnTeamsMessagingExtensionQueryAsync é executado, a primeira coisa que queremos fazer é entender o que o usuário inseriu na caixa de pesquisa.

Primeiro, vamos remover o código existente.

Continuando no Visual Studio:

1. Na pasta **Pesquisar**, abra o arquivo chamado **SearchApp.cs**
1. No método **OnTeamsMessagingExtensionQueryAsync**, remova todo o código **após** a instrução **if** que verifica se há um token de acesso
1. Na classe **SearchApp**, remova o método **FindPackages** e a propriedade **_adaptiveCardFilePath**.

Após remover o código, a classe **SearchApp** ficará semelhante ao seguinte trecho de código:

```csharp
public class SearchApp : TeamsActivityHandler
{
    private readonly string connectionName;
    private readonly string spoHostname;
    private readonly string spoSiteUrl;

    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    }

    protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
    {
        var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
        var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);

        if (!HasToken(tokenResponse))
        {
            return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
        }
    }
}
```

Agora vamos escrever o código para obter o valor do parâmetro **ProductName**.

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione código para recuperar o valor do parâmetro **ProductName** da matriz **Parâmetros** no objeto **MessagingExtensionQuery**

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    ```

1. Na classe **SearchApp**, implemente o método **GetQueryData**

    ```csharp
    private static string GetQueryData(IList<MessagingExtensionParameter> parameters, string key)
    {
      if (parameters.Any() != true)
      {
        return string.Empty;
      }
    
      var foundPair = parameters.FirstOrDefault(pair => pair.Name == key);
      return foundPair?.Value?.ToString() ?? string.Empty;
    }
    ```

1. Salvar suas alterações

O método **GetQueryData** é usado para recuperar o valor associado a uma chave específica de uma lista de objetos **MessagingExtensionParameter**. Ele oferece uma maneira conveniente de extrair dados da matriz de parâmetros no objeto **MessagingExtensionQuery**.

## Tarefa 5 – Criar o filtro OData de consulta de lista do SharePoint

Agora que o usuário passou o valor, use-o valor para criar um filtro de consulta OData. O filtro é usado para consultar a lista do SharePoint Online pela coluna Title, que contém o nome do produto.

Continuando no Visual Studio:

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione código para criar a variável **filterQuery**.

    ```csharp
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Salvar suas alterações

O código construirá uma consulta de filtro com base no parâmetro **name**. Se o parâmetro name for fornecido, ele criará uma expressão de filtro para procurar itens com um campo **Título** começando com o nome fornecido. Se o parâmetro name não for fornecido, ele atribuirá uma cadeia de caracteres vazia como a consulta de filtro. A consulta de filtro resultante será usada posteriormente no código para recuperar itens filtrados de um site do SharePoint.

## Tarefa 6 – Instalar e configurar o SDK do Microsoft Graph

Para executar solicitações autenticadas para o Microsoft Graph, use o **SDK do Microsoft Graph**.

Instale o pacote do SDK do Microsoft Graph a partir do NuGet, crie uma classe **TokenProvider**, que permite usar o token de acesso obtido do serviço de token e, em seguida, inicialize um novo **GraphServiceClient**.

Continuando no Visual Studio:

1. No Gerenciador de Soluções, clique com o botão direito do mouse no projeto **MsgExtProductSupport**
1. Selecione **Gerenciar pacotes NuGet...**
1. Selecione a guia **Procurar** e pesquise **Microsoft.Grapf**
1. Na lista de resultados, selecione **Microsoft.Graph**
1. Na lista suspensa **Versão**, escolha **5.42.0**
1. Selecionar **Instalar**
1. Na caixa de diálogo **Aceitação da Licença**, selecione **Aceito** para instalar o SDK.

Depois de instalar o pacote, crie um provedor de token para o SDK do Microsoft Graph.

1. Na pasta **Pesquisar**, crie um arquivo chamado **TokenProvider.cs**
1. No arquivo , adicione o seguinte código:

    ```csharp
    using Microsoft.Kiota.Abstractions.Authentication;
    
    namespace MsgExtProductSupport.Search
    {
       public class TokenProvider : IAccessTokenProvider
        {
            public string Token { get; set; }
            public AllowedHostsValidator AllowedHostsValidator => throw new NotImplementedException();
    
            public Task<string> GetAuthorizationTokenAsync(Uri uri, Dictionary<string, object>? additionalAuthenticationContext = null, CancellationToken cancellationToken = default)
            {
                return Task.FromResult(Token);
            }
        }
    }
    ```

1. Salvar suas alterações

Agora crie um método para criar uma nova instância **GraphServiceClient**.

1. Na pasta **Pesquisar**, abra o arquivo chamado **SearchApp.cs**
1. No arquivo, importe os namespaces necessários:

    ```csharp
    using Microsoft.Graph;
    using Microsoft.Kiota.Abstractions.Authentication;
    ```

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione código para criar um novo Graph Client, que você usa para enviar solicitações ao Microsoft Graph

    ```csharp
    var graphClient = CreateGraphClient(tokenResponse);
    ```

1. Na classe **SearchApp**, implemente o método **CreateGraphClient**

    ```csharp
    private static GraphServiceClient CreateGraphClient(TokenResponse tokenResponse)
    {
      TokenProvider provider = new() { Token = tokenResponse.Token };
      var authenticationProvider = new BaseBearerTokenAuthenticationProvider(provider);
      var graphClient = new GraphServiceClient(authenticationProvider);
      return graphClient;
    }
    ```

1. Salvar suas alterações

Esse código configura o provedor de autenticação e cria um objeto cliente que pode ser usado para interagir com a API do Microsoft Graph usando o token de acesso fornecido.

## Tarefa 7 – Consultar lista de produtos

Para consultar os itens na lista Produtos e, posteriormente, criar os resultados da pesquisa, use o GraphServiceClient para enviar solicitações para obter dados do produto do SharePoint Online.

Continuando no Visual Studio:

No método **OnTeamsMessagingExtensionQueryAsync**, adicione código para obter os dados do SharePoint:

  ```csharp
  var site = await GetSharePointSite(graphClient, spoHostname, spoSiteUrl, cancellationToken);
  var drive = await GetSharePointDrive(graphClient, site.SharepointIds.SiteId, "Product Imagery", cancellationToken);
  var items = await GetProducts(graphClient, site.SharepointIds.SiteId, filterQuery, cancellationToken);
  ```

Este código irá:

- **Mostrar o site de Marketing do produto**; o objeto de site contém a ID do site do SharePoint, que é usado para retornar e consultar objetos no site
- **Mostrar a unidade de Imagens do produto**; a unidade representa a biblioteca de documentos que contém as imagens do produto. Você usará a unidade posteriormente para ver as imagens do produto exibidas nos resultados da pesquisa
- **Mostrar os Produtos**, que você usa para consultar a lista Produtos com base na consulta do usuário

Implemente os três métodos na classe **SearchApp**.

- Implemente o método **GetSharePointSite**

    ```csharp
    private static async Task<Site> GetSharePointSite(GraphServiceClient graphClient, string hostName, string siteUrl, CancellationToken cancellationToken)
    {
        return await graphClient.Sites[$"{hostName}:/{siteUrl}"].GetAsync(r => r.QueryParameters.Select = new string[] { "sharePointIds" }, cancellationToken);
    }
    ```

Esse método usa o GraphServiceClient para enviar uma solicitação ao Microsoft Graph para retornar o objeto de site usando um caminho. O caminho é criado a partir da combinação do nome de host do SharePoint Online e da URL do site. Como você só precisa dos valores da propriedade sharePointIds, o parâmetro de consulta Select é configurado para retornar somente essa propriedade na resposta.

- Implemente o método **GetSharePointDrive**

    ```csharp
    private static async Task<Drive> GetSharePointDrive(GraphServiceClient graphClient, string siteId, string name, CancellationToken cancellationToken)
    {
        var drives = await graphClient.Sites[siteId].Drives.GetAsync(r => r.QueryParameters.Select = new string[] { "id", "name" }, cancellationToken);
        var drive = drives.Value.Find(d => d.Name == name);
        return drive;
    }
    ```

Esse método usa o GraphServiceClient e a ID do site para retornar uma coleção de bibliotecas de documentos do site, retornando as propriedades ID e name de cada biblioteca de documentos. A coleção de bibliotecas é então filtrada para retornar a unidade, que tem o mesmo nome que o parâmetro do método name.

- Implemente o método **GetProducts**

    ```csharp
    private static async Task<SiteCollectionResponse> GetProducts(GraphServiceClient graphClient, string siteId, string filterQuery, CancellationToken cancellationToken)
    {
        var fields = new string[]
        {
            "fields/Id",
            "fields/Title",
            "fields/RetailCategory",
            "fields/PhotoSubmission",
            "fields/CustomerRating",
            "fields/ReleaseDate"
        };
    
        var request = graphClient.Sites.WithUrl($"https://graph.microsoft.com/v1.0/sites/{siteId}/lists/Products/items?expand={string.Join(",", fields)}&$filter={filterQuery}");
        return await request.GetAsync(null, cancellationToken);
    }
    ```

Esse método usa o GraphServiceClient para retornar itens de lista filtrados da lista Produtos usando a consulta de filtro passada e retorna os dados de item de lista definidos na matriz de campos.

## Tarefa 8 – Criar resultados de pesquisa

Depois de obter os produtos do SharePoint, você criará os resultados da pesquisa, que serão retornados ao usuário.

A criação dos resultados da pesquisa consiste em iterar da matriz de itens, para cada item, você cria um MessagingExtensionAttachment, que contém os cartões de visualização e conteúdo.

Antes de iterar nos itens, crie um modelo de Cartão Adaptável, que você usa em seu loop.

Continuando no Visual Studio:

1. Na pasta **Recursos**, crie um arquivo chamado **Product.json**

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.6",
      "body": [
        {
          "type": "TextBlock",
          "text": "${Product.Title}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${Product.RetailCategory}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${ProductImage}",
              "altText": "${Product.Title}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(Product.CustomerRating,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(Product.ReleaseDate,'dd/MM/yyyy')}"
            }
          ]
        },
        {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View",
              "url": "https://${SPOHostname}/${SPOSiteUrl}/Lists/Products/DispForm.aspx?ID=${Product.Id}"
            }
          ]
        }
      ]
    }
    ```

O modelo usa expressões de vinculação de dados, que são substituídas por valores reais ao renderizar o Cartão Adaptável. Quando o cartão for renderizado, ele conterá algumas informações do produto, uma imagem do produto e um botão de ação. O botão de ação abrirá um navegador que navega até o formulário de exibição de Lista do SharePoint do item do produto.

Em seguida, adicione o código para transformar o arquivo JSON em um modelo de Cartão Adaptável.

1. Na pasta **Pesquisar**, abra **SearchApp.cs**
1. No arquivo, importe os namespaces necessários:

    ```csharp
    using AdaptiveCards.Templating;
    ```

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione o código para ler o conteúdo do arquivo JSON e criar um novo objeto **AdaptiveCardTemplate**.

    ```csharp
    var card = File.ReadAllText(@"Resources\Product.json");
    var template = new AdaptiveCardTemplate(card);
    ```

1. Salvar suas alterações

Em seguida, crie um loop para iterar nos itens da lista. Cada iteração irá:

- Desserializar os dados do item atual em um objeto Produto
- Mostrar a miniatura da imagem do produto
- Criar um cartão de conteúdo
- Criar um cartão de visualização
- Criar um MessagingExtensionAttachment combinando o conteúdo e os cartões de visualização
- Adicionar o MessagingExtensionAttachment à lista

Quando o loop terminar, você terá uma lista de anexos que podem ser retornados ao usuário.

1. Na pasta **Pesquisar**, abra **SearchApp.cs**
1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione código para criar uma nova Lista para armazenar os objetos **MessagingExtensionAttachment**

    ```csharp
    var attachments = new List<MessagingExtensionAttachment>();
    ```

1. Crie um loop foreach para iterar nos itens da lista.

    ```csharp
    foreach (var item in items.Value) { 
            
    }
    ```

1. Adicione o código a seguir ao loop foreach.

    ```csharp
    var product = JsonConvert.DeserializeObject<Product>(item.AdditionalData["fields"].ToString());
    product.Id = item.Id;
    
    var thumbnails = await GetThumbnails(graphClient, drive.Id, product.PhotoSubmission, cancellationToken);
    
    var resultCard = template.Expand(new
    {
      Product = product,
      ProductImage = thumbnails.Large.Url,
      SPOHostname = spoHostname,
      SPOSiteUrl = spoSiteUrl,
    });
    
    var previewcard = new ThumbnailCard
    {
      Title = product.Title,
      Subtitle = product.RetailCategory,
      Images = new List<CardImage> { new() { Url = thumbnails.Small.Url } }
    }.ToAttachment();
    
    var attachment = new MessagingExtensionAttachment
    {
      Content = JsonConvert.DeserializeObject(resultCard),
      ContentType = AdaptiveCard.ContentType,
      Preview = previewcard
    };
    
    attachments.Add(attachment);
    ```

1. Salvar suas alterações

Para tipar fortemente os dados do item de lista, crie um modelo que represente o **Produto**.

1. Na pasta raiz do projeto, crie uma nova pasta chamada **Modelos**
1. Na pasta **Modelos**, crie um novo arquivo chamado **Product.cs**

    ```csharp
    namespace MsgExtProductSupport.Models
    {
        public class Product
        {
            public string Title { get; set; }
            public string RetailCategory { get; set; }
            public Link Specguide { get; set; }
            public string PhotoSubmission { get; set; }
            public double CustomerRating { get; set; }
            public DateTime ReleaseDate { get; set; }
            public string Id { get; set; }
            public string ContentType { get; set; }
            public DateTime Modified { get; set; }
            public DateTime Created { get; set; }
        }
    
        public class Link
        {
            public string Description { get; set; }
            public string Url { get; set; }
        }
    }
    ```

1. Salvar suas alterações

Em seguida, implemente o método **GetThumbnails** para recuperar imagens em miniatura do Microsoft Graph de um produto.

1. Na pasta **Pesquisar**, abra o arquivo chamado **SearchApp.cs**
1. Na classe **SearchApp**, crie o método **GetThumbnails**

    ```csharp
    private static async Task<ThumbnailSet> GetThumbnails(GraphServiceClient graphClient, string driveId, string photoUrl, CancellationToken cancellationToken)
    {
        var fileName = photoUrl.Split('/').Last();
        var driveItem = await graphClient.Drives[driveId].Root.ItemWithPath(fileName).GetAsync(null, cancellationToken);
        var thumbnails = await graphClient.Drives[driveId].Items[driveItem.Id].Thumbnails["0"].GetAsync(r => r.QueryParameters.Select = new string[] { "small", "large" }, cancellationToken);
        return thumbnails;
    }
    ```

1. Salvar suas alterações

O método **GetThumbnails** usa o ponto de extremidade Miniaturas na API do Microsoft Graph para retornar miniaturas pequenas e grandes da imagem do produto armazenada no SharePoint.

## Tarefa 9 – Retornar resultados da pesquisa

Agora que temos uma coleção de objetos MessagingExtensionResult, podemos retorná-los ao usuário como resultados da pesquisa.

- No método **OnTeamsMessagingExtensionQueryAsync**, adicione código para retornar os resultados da pesquisa como a resposta da extensão de mensagem.

    ```csharp
    return new MessagingExtensionResponse
    {
      ComposeExtension = new MessagingExtensionResult
      {
        Type = "result",
        AttachmentLayout = "list",
        Attachments = attachments
      }
    };
    ```

## Tarefa 10 – Provisionar os recursos

Execute o processo Preparar dependências do aplicativo Teams para provisionar recursos.

Continuando no Visual Studio:

1. No **Gerenciador de Soluções**, clique com o botão direito do mouse no projeto **MsgExtProductSupport**
1. Expanda o menu Kit de Ferramentas** do Teams**, selecione **Preparar dependências do aplicativo Teams**
1. Na caixa de diálogo **Conta do Microsoft 365**, selecione **Continuar**
1. Na caixa de diálogo **Provisionar**, selecione **Provisionar**
1. Na caixa de diálogo de **aviso do Kit de Ferramentas do Teams**, selecione **Provisionar**
1. Na caixa de diálogo **informações do Kit de Ferramentas do Teams**, **feche** o prompt

## Tarefa 11 – Executar e depurar

Agora inicie o serviço Web e teste a extensão de mensagem no Microsoft Teams.

Continuando no Visual Studio:

1. Aperte **F5** para iniciar uma sessão de depuração e abrir uma nova janela do navegador que navega até o cliente Web do Microsoft Teams.
1. Insira suas credenciais da conta do Microsoft 365 e continue no Microsoft Teams.
1. Na caixa de diálogo de instalação do aplicativo, selecione **Adicionar**
1. Abra um bate-papo novo ou já existente do Microsoft Teams
1. Na área de redigir mensagem, selecione **...** para abrir o submenu do aplicativo
1. Na lista de aplicativos, selecione **produtos da Contoso** para abrir a extensão de mensagem
1. Digite **Mark8** na caixa de texto. Dois resultados são mostrados, **Mark8** e **controlador Mark8**
1. Escolha **Mark8** para inserir um cartão na caixa de redigir mensagem
1. **Envie a mensagem** contendo o cartão
1. No cartão enviado, selecione o **botão Exibir** para exibir o item de lista do SharePoint do produto na lista Produtos em uma nova guia

Feche o navegador para interromper a sessão de depuração.

[Continue no próximo exercício...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)