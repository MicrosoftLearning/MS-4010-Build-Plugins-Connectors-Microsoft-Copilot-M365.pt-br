---
lab:
  title: Exercício 3 – Retornar dados do produto da API protegida do Microsoft Entra
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercício 3 – Retornar dados do produto da API protegida do Microsoft Entra

Neste exercício, você atualizará a extensão de mensagem para recuperar dados de uma API personalizada. Você obterá dados da API personalizada com base na consulta do usuário e retornará dados nos resultados da pesquisa para o usuário.

![Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams.](../media/3-search-results-api.png)

### Duração do exercício

  - **Tempo estimado para conclusão:** 50 minutos

## Tarefa 1 – Instalar e configurar o proxy de desenvolvimento

Neste exercício, você usará o proxy de desenvolvimento, uma ferramenta de linha de comando que pode simular APIs. É útil quando você quer testar seu aplicativo sem precisar criar uma API real.

Para concluir este exercício, você precisa instalar a [versão mais recente do proxy de desenvolvimento](/microsoft-cloud/dev/dev-proxy/get-started) e baixar a predefinição dele para este módulo.

A predefinição simula uma API CRUD (Criar, Ler, Atualizar, Excluir) com um armazenamento de dados na memória, que o Microsoft Entra protege. Isso significa que você pode testar seu aplicativo como se ele estivesse chamando uma API real que requer autenticação.

1. Para instalar o proxy de desenvolvimento, abra uma nova **janela do prompt de comando como Administrador**:

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. Para baixar a predefinição, execute o seguinte texto do comando:

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. Mantenha a janela do prompt de comando aberta para usá-la depois.

## Tarefa 2 – Obter o valor de consulta do usuário

Crie um método que obtenha o valor da consulta do usuário pelo nome do parâmetro.

No Visual Studio e no **ProductsPlugin**:

1. Na pasta **Auxiliares**, crie um novo arquivo chamado **MessageExtensionHelpers.cs**.

1. Substitua o código no arquivo por:

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. Salve suas alterações.

Em seguida, atualize o método **OnTeamsMessagingExtensionQueryAsync** na classe SearchApp para usar o novo método auxiliar.

1. Na pasta **Pesquisar**, abra **SearchApp.cs**.

1. No método **OnTeamsMessagingExtensionQueryAsync**, substitua o código:

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   por

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. Mova o cursor para a variável de **texto**, use `Ctrl + R`, `Ctrl + R` e renomeie a variável para **nome**.

1. Pressione **Enter** para renomear a variável em três arquivos.

1. Salve suas alterações.

O método **OnTeamsMessagingExtensionQueryAsync** agora estará assim:

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## Tarefa 3 – Obter dados da API personalizada

Para obter dados da API personalizada, você precisa enviar o token de acesso no Cabeçalho de autorização da solicitação e desserializar a resposta em um modelo que represente os dados do produto.

Primeiro, crie um modelo que represente os dados do produto retornados da API personalizada.

No Visual Studio e no **ProductsPlugin**:

1. Crie uma pasta chamada **Modelos**.

1. Na pasta **Modelos**, crie um novo arquivo chamado **Product.cs**.

1. No arquivo, substitua o código existente pelo seguinte:

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. Salve suas alterações.

Em seguida, crie uma classe de serviço que recupere os dados do produto pela API personalizada.

No Visual Studio e no **ProductsPlugin**:

1. Crie uma pasta chamada **Serviços**.

1. Na pasta **Serviços**, crie um arquivo chamado **ProductService.cs**

1. No arquivo, substitua o código existente pelo seguinte:

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. Salve suas alterações.

A classe **ProductsService** contém métodos para obter dados do produto pela API personalizada. O construtor de classe usa um token de acesso como parâmetro e configura uma instância **HttpClient** com o token de acesso no cabeçalho de autorização.

Em seguida, atualize o método **OnTeamsMessagingExtensionQueryAsync** para usar a classe **ProductsService** para obter dados do produto a partir da API personalizada.

1. Na pasta **Pesquisar**, abra **SearchApp.cs**.

1. No método **OnTeamsMessagingExtensionQueryAsync**, adicione o seguinte código após a declaração da variável **nome** para obter dados do produto da API personalizada:

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. Salve suas alterações.

## Tarefa 4 – Criar os resultados da pesquisa

Agora que você tem os dados do produto, poderá incluí-los nos resultados da pesquisa que são retornados ao usuário.

Primeiro, vamos atualizar o modelo de Cartão Adaptável existente para exibir as informações do produto.

Continuando no Visual Studio e no projeto **ProductsPlugin**:

1. Na pasta **Recursos**, renomeie **card.json** para **Product.json**.

1. Na pasta **Recursos**, crie um arquivo chamado **Product.data.json**. Esse arquivo contém dados de exemplo que o Visual Studio usa para gerar uma visualização do modelo de Cartão Adaptável.

1. Adicione o seguinte JSON ao arquivo:

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. Salve suas alterações.

1. Na pasta **Recursos**, abra **Product.json**.

1. No arquivo, substitua o conteúdo pelo seguinte JSON:

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
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
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. Salve suas alterações.

O modelo de Cartão Adaptável usa expressões de associação para exibir as informações do produto. As expressões **\$\{name\}**, **\$\{category\}**, **\$\{imageUrl\}**, **\$\{callVolume\}** e **\$\{releaseDate\}** são substituídas pelos valores correspondentes dos dados do produto. As funções de modelo **formatNumber** e **formatDateTime** são usadas para formatar os valores **callVolume** e **releaseDate** em um número e data, respectivamente.

Reserve um momento para explorar a visualização do Cartão Adaptável no Visual Studio. A versão prévia mostra a aparência do modelo de Cartão Adaptável quando os dados do produto são associados ao modelo. Ele usa os dados de exemplo do arquivo **Product.data.json** para gerar a visualização.

Em seguida, atualize a propriedade **validDomains** no manifesto do aplicativo para incluir o domínio **raw.githubusercontent.com**, para que as imagens no modelo de Cartão Adaptável possam ser exibidas no Microsoft Teams.

No projeto **TeamsApp**:

1. Na pasta **appPackage**, abra o arquivo **manifest.json**.

1. No arquivo, adicione o domínio GitHub à propriedade **validDomains**:

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. Salve suas alterações.

Em seguida, atualize o método **OnTeamsMessagingExtensionQueryAsync** para criar uma lista de anexos que contêm as informações do produto.

No projeto **ProductsPlugin**:

1. Na pasta **Pesquisar**, abra **SearchApp.cs**.

1. Atualize **card.json** para **Product.json** para refletir a alteração no nome do arquivo. Na linha 34, substitua o código a seguir:

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   por

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. Adicione o seguinte código após a declaração da variável de **modelo** para criar uma lista de anexos:

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. Atualize a instrução **return** para incluir a variável **anexos** :

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

1. Salvar as alterações

## Tarefa 5 – Criar e atualizar os recursos

Feito tudo isso, execute o processo **Preparar dependências do aplicativo Teams** para criar novos recursos e atualizar os já existentes.

Continuando no Visual Studio:

1. No **Gerenciador de Soluções**, clique com o botão direito do mouse no projeto **TeamsApp**.

1. Expanda o menu **Kit de Ferramentas do Teams** e selecione **Preparar dependências do aplicativo Teams**.

1. Na caixa de diálogo **Conta do Microsoft 365**, clique em **Continuar**.

1. Na caixa de diálogo **Provisionar**, clique em **Provisionar**.

1. Na caixa de diálogo de **aviso do Kit de Ferramentas do Teams**, clique em **Provisionar**.

1. Na caixa de diálogo de **informações do Kit de Ferramentas do Teams**, clique no ícone de cruz para fechar a caixa de diálogo.

## Tarefa 6 – Executar e depurar

Com os recursos provisionados, inicie uma sessão de depuração para testar a extensão da mensagem.

Primeiro, inicie o proxe de desenvolvimento para simular a API personalizada.

1. Na **janela do prompt de comando** que ainda está aberta, execute o seguinte comando para iniciar o proxy de desenvolvimento:

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. Se solicitado, aceite os avisos de certificado.

> [!NOTE]
> Quando o proxy de desenvolvimento estiver em execução, ele atuará como um proxy de todo o sistema.

Em seguida, inicie uma sessão de depuração no Visual Studio:

1. Para iniciar uma nova sessão de depuração, pressione <kbd>F5</kbd> ou clique em **Iniciar** na barra de ferramentas.

1. Aguarde até que uma janela do navegador seja aberta e a caixa de diálogo de instalação do aplicativo apareça no cliente Web do Microsoft Teams. Se solicitado, insira as credenciais da sua conta do Microsoft 365.

1. Na caixa de diálogo de instalação do aplicativo, selecione **Adicionar**.

1. Abra um bate-papo novo ou já existente do Microsoft Teams.

1. Na área de composição da mensagem, digite **/apps** para abrir o seletor de aplicativo.

1. Na lista de aplicativos, selecione **Produtos da Contoso** para abrir a extensão de mensagem.

1. Digite **mark8** na caixa de texto. Pode ser necessário inserir sua consulta de pesquisa várias vezes.

1. Aguarde a conclusão da pesquisa e a exibição dos resultados.

    ![Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams.](../media/3-search-results-api.png)

1. Na lista de resultados, selecione um resultado de pesquisa para inserir um cartão na caixa de redigir mensagem

Retorne ao Visual Studio e selecione **Parar** na barra de ferramentas ou pressione <kbd>Shift</kbd> + <kbd>F5</kbd> para interromper a sessão de depuração. Além disso, encerre o proxy de desenvolvimento pressionando <kbd>Ctrl</kbd> + <kbd>C</kbd>.

[Continue no próximo exercício...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)