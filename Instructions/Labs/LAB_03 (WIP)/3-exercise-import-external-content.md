---
lab:
  title: Exercício 2 – Importar conteúdo externo
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Exercício – Importar conteúdo externo

Neste exercício, você estenderá o conector personalizado do Microsoft Graph com o código para importar arquivos markdown locais para o Microsoft 365.

## Antes de começar

Este exercício levará cerca de **XX minutos** para ser concluído.

## Tarefa 1 – Baixar conteúdo externo

Para seguir este exercício, copie os arquivos de conteúdo de exemplo usados neste exercício no [GitHub](https://pnp.github.io/download-partial/?url=https://github.com/pnp/graph-connectors-samples/tree/main/samples/dotnet-csharp-graphdocs/content) e armazene-os em seu projeto, em uma pasta chamada **conteúdo**.

![Captura de tela de um editor de código mostrando os arquivos de conteúdo usados neste exercício.](../media/6-content-files.png)

Para que o código funcione corretamente, a pasta **conteúdo** e o conteúdo nela devem ser copiados para a pasta de saída da compilação.

No editor de código:

1. Abra o arquivo **.csproj** e adicione o seguinte código antes da tag `</Project>`:

   ```xml
   <ItemGroup>
     <ContentFiles Include="content\**"    CopyToOutputDirectory="PreserveNewest" />
   </ItemGroup>
   
   <Target Name="CopyContentFolder" AfterTargets="Build">
     <Copy SourceFiles="@(ContentFiles)" DestinationFiles="@   (ContentFiles->'$(OutputPath)\content\%(RecursiveDir)%(Filename)%   (Extension)')" />
   </Target>
   ```

1. Salve suas alterações.

## Tarefa 2 – Adicionar bibliotecas para analisar Markdown e YAML

O conector do Microsoft Graph que você está criando importa arquivos markdown locais para o Microsoft 365. Cada um desses arquivos contém um cabeçalho com metadados no formato YAML, também conhecido como frontmatter. Além disso, o conteúdo de cada arquivo é gravado em Markdown. Para extrair metadados e converter o corpo em HTML, use bibliotecas personalizadas:

1. Abra um terminal e altere o diretório de trabalho para o projeto.
1. Para adicionar a biblioteca de processamento Markdown, execute o seguinte comando: `dotnet add package Markdig`.
1. Para adicionar a biblioteca de processamento YAML, execute este comando: `dotnet add package YamlDotNet`.

## Tarefa 3 – Definir a classe para representar o arquivo importado

Para simplificar o trabalho com arquivos markdown importados e seu conteúdo, vamos definir uma classe com as propriedades necessárias.

No editor de código:

1. Crie um arquivo chamado **ContentService.cs**.
1. Adicione os códigos a seguir:

   ```csharp
   using YamlDotNet.Serialization;
   
   public interface IMarkdown
   {
     string? Markdown { get; set; }
   }
   
   class DocsArticle : IMarkdown
   {
     [YamlMember(Alias = "title")]
     public string? Title { get; set; }
     [YamlMember(Alias = "description")]
     public string? Description { get; set; }
     public string? Markdown { get; set; }
     public string? Content { get; set; }
     public string? RelativePath { get; set; }
   }
   ```

   A interface `IMarkdown` representa o conteúdo do arquivo markdown local. Ele precisa ser definido separadamente para aceitar a desserialização do conteúdo do arquivo. A classe `DocsArticle` representa o documento final com propriedades YAML e conteúdo HTML analisados. Os atributos `YamlMember` mapeiam propriedades para metadados no cabeçalho de cada documento.

1. Salve suas alterações.

## Tarefa 4 – Definir a classe `ContentService`

Em seguida, crie uma classe que contenha o código para carregar arquivos Markdown locais, transformando-os em itens externos e carregando-os no Microsoft 365.

No editor de código:

1. Verifique se você está editando o arquivo **ContentService.cs**.
1. Adicione a seguinte instrução using na parte superior do arquivo:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Depois, ao final do arquivo, adicione o seguinte código:

   ```csharp
   static class ContentService
   {
     static IEnumerable<DocsArticle> Extract()
     {}
   
     static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
     {}
   
     static async Task Load(IEnumerable<ExternalItem> items)
     {}
   
     public static async Task LoadContent()
     {
       var content = Extract();
       var transformed = Transform(content);
       await Load(transformed);
     }
   }
   ```

   A classe `ContentService` define três métodos que representam o processo de manipulação de conteúdo:

   1. `Extract`, que carrega arquivos markdown locais e os analisa em instâncias da classe `DocsArticle` para facilitar a manipulação.
   1. `Transform`, que converte objetos `DocsArticle` em instâncias da classe `ExternalItems`, que faz parte do SDK do Microsoft Graph .NET e que representa itens externos a serem carregados no Microsoft 365.
   1. `Load`, que carrega itens externos para o Microsoft 365 usando a API do Microsoft Graph.

   Esses métodos são chamados nessa ordem específica a partir do método `LoadContent`.

1. Salve suas alterações.

## Tarefa 5 – Configurar o processamento de markdown

Vamos começar com a extração do conteúdo dos arquivos markdown locais.

Primeiro, adicione métodos auxiliares para usar as bibliotecas `Markdig` e `YamlDotNet` de forma fácil..

No editor de código:

1. Crie um arquivo chamado **MarkdownExtensions.cs**.
1. No arquivo , adicione o seguinte código:

   ```csharp
   // from: https://khalidabuhakmeh.com/parse-markdown-front-matter-with-csharp
   using Markdig;
   using Markdig.Extensions.Yaml;
   using Markdig.Syntax;
   using YamlDotNet.Serialization;
   
   public static class MarkdownExtensions
   {
     private static readonly IDeserializer YamlDeserializer =
         new DeserializerBuilder()
         .IgnoreUnmatchedProperties()
         .Build();
         
     private static readonly MarkdownPipeline Pipeline
         = new MarkdownPipelineBuilder()
         .UseYamlFrontMatter()
         .Build();
   }
   ```

   A propriedade `YamlDeserializer` define um novo desserializador para o bloco YAML em cada um dos arquivos markdown que você está extraindo. Você configurará o desserializador para ignorar todas as propriedades que não fazem parte da classe para a qual o arquivo é desserializado.

   A propriedade `Pipeline` define um pipeline de processamento para o analisador de markdown. Você o configura para analisar o cabeçalho YAML. Sem essa configuração, as informações do cabeçalho seriam descartadas.

1. Depois, estenda a classe `MarkdownExtensions` com o seguinte código:

   ```csharp
   public static T GetContents<T>(this string markdown) where T :    IMarkdown, new()
   {
     var document = Markdown.Parse(markdown, Pipeline);
     var block = document
         .Descendants<YamlFrontMatterBlock>()
         .FirstOrDefault();
   
     if (block == null)
       return new T { Markdown = markdown };
   
     var yaml =
         block
         // this is not a mistake
         // we have to call .Lines 2x
         .Lines // StringLineGroup[]
         .Lines // StringLine[]
         .OrderByDescending(x => x.Line)
         .Select(x => $"{x}\n")
         .ToList()
         .Select(x => x.Replace("---", string.Empty))
         .Where(x => !string.IsNullOrWhiteSpace(x))
         .Aggregate((s, agg) => agg + s);
   
     var t = YamlDeserializer.Deserialize<T>(yaml);
     t.Markdown = markdown.Substring(block.Span.End + 1);
     return t;
   }
   ```

   O método `GetContents` converte uma cadeia de caracteres Markdown com metadados YAML no cabeçalho no tipo especificado, que implementa a interface `IMarkdown`. A partir da cadeia de caracteres de marcação, ele extrai o cabeçalho YAML e o desserializa no tipo especificado. Depois, ele extrai o corpo do artigo e o define como a propriedade `Markdown` para processamento posterior.

1. Salve suas alterações.

## Tarefa 6 – Extrair conteúdo markdown e YAML

Com os métodos auxiliares prontos, implemente o método de extração para carregar os arquivos Markdown locais e extrair informações deles.

No editor de código:

1. Abra o arquivo **ContentService.cs**.
1. Na parte superior do arquivo, adicione a seguinte instrução using:

   ```csharp
   using Markdig;
   ```

1. Depois, na classe `ContentService`, implemente o método `Extract` usando o código:

   ```csharp
   static IEnumerable<DocsArticle> Extract()
   {
     var docs = new List<DocsArticle>();
   
     var contentFolder = "content";
     var contentFolderPath = Path.Combine(Directory.GetCurrentDirectory(),    contentFolder);
     var files = Directory.GetFiles(contentFolder, "*.md", SearchOption.   AllDirectories);
   
     foreach (var file in files)
     {
       try
       {
         var contents = File.ReadAllText(file);
         var doc = contents.GetContents<DocsArticle>();
         doc.Content = Markdown.ToHtml(doc.Markdown ?? "");
         doc.RelativePath = Path.GetRelativePath(contentFolderPath, file);
         docs.Add(doc);
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   
     return docs;
   }
   ```

   O método começa carregando arquivos markdown a partir da pasta **conteúdo**. Para cada arquivo, ele carrega seu conteúdo como uma cadeia de caracteres. Ele converte a cadeia de caracteres em um objeto com os metadados e o conteúdo armazenados em propriedades separadas usando o método de extensão `GetContents` definido anteriormente na classe `MarkdownExtensions`. Em seguida, ele converte a cadeia de caracteres markdown em HTML. Por fim, ele armazena o caminho relativo para o arquivo e adiciona o objeto a uma coleção para processamento posterior.

1. Salve suas alterações.

## Tarefa 7 – Transformar conteúdo em itens externos

Depois de ler o conteúdo externo, a próxima etapa é transformá-lo em itens externos, que serão carregados no Microsoft 365.

Comece adicionando um método auxiliar que gera uma ID exclusiva para cada item externo com base em seu caminho de arquivo relativo.

No editor de código:

1. Confirme se você está editando o arquivo **ContentService.cs**.
1. Na classe `ContentService` , adicione o seguinte método:

   ```csharp
   static string GetDocId(string relativePath)
   {
     var id = relativePath.Replace(Path.DirectorySeparatorChar.ToString(),    "__").Replace(".md", "");
     return id;
   }
   ```

   O método `GetDocId` usa o caminho de arquivo relativo e substitui todos os separadores de diretório por um sublinhado duplo. Isso é necessário porque os caracteres separadores de caminho não podem ser usados em uma ID de item externo.

1. Salve suas alterações.

Agora, implemente o método `Transform`, que converte objetos que representam arquivos markdown locais em itens externos do Microsoft Graph.

No editor de código:

1. Confirme se você está no arquivo **ContentService.cs**.
1. Implemente o método `Transform` com o seguinte código:

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var docId = GetDocId(a.RelativePath ?? '');
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             new()
             {
               Type = AclType.Everyone,
               Value = "everyone",
               AccessType = AccessType.Grant
             }
         }
       };
     });
   }
   ```

   Primeiro, defina uma URL base. Use essa URL para criar uma URL completa para cada item, para que, quando o item for exibido aos usuários, eles possam navegar até o item original. Em seguida, transforme cada item de um `DocsArticle` em um `ExternalItem`. Comece obtendo uma ID exclusiva para cada item com base em seu caminho de arquivo relativo. Em seguida, crie uma nova instância de `ExternalItem` e preencha as propriedades com informações do `DocsArticle`. Depois, defina o conteúdo do item para o conteúdo HTML extraído do arquivo local e defina o tipo de conteúdo do item como HTML. Por fim, configure a permissão do item para que ele fique visível para todo mundo na organização.

1. Salve suas alterações.

## Tarefa 8 – Carregar itens externos no Microsoft 365

A última etapa do processamento do conteúdo é carregar os itens externos transformados no Microsoft 365.

No editor de código:

1. Verifique se você está editando o arquivo **ContentService.cs**.
1. Na classe `ContentService`, implemente o método `Load` usando o código:

   ```csharp
   static async Task Load(IEnumerable<ExternalItem> items)
   {
     foreach (var item in items)
     {
       Console.Write(string.Format("Loading item {0}...", item.Id));
   
       try
       {
         await GraphService.Client.External
           .Connections[Uri.EscapeDataString(ConnectionConfiguration.   ExternalConnection.Id!)]
           .Items[item.Id]
           .PutAsync(item);
   
         Console.WriteLine("DONE");
       }
       catch (Exception ex)
       {
         Console.WriteLine("ERROR");
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

   Para cada item externo, use o SDK do Microsoft Graph .NET para chamar a API do Microsoft Graph e fazer upload do item. Na solicitação, especifique a ID da conexão externa criada anteriormente, a ID do item a ser carregado e o conteúdo completo do item.

1. Salve suas alterações.

## Tarefa 9 – Adicionar o comando content load

Antes de testar o código, você precisa estender o aplicativo de console com um comando que invoca a lógica de carregamento de conteúdo.

No editor de código:

1. Abra o arquivo **Program.cs**.
1. Adicione um novo comando para carregar conteúdo usando o código:

    ```csharp
    var loadContentCommand = new Command("load-content", "Loads content   into the external connection");
    loadContentCommand.SetHandler(ContentService.LoadContent);
    ```

1. Registre o comando recém-definido com o comando raiz para que ele possa ser chamado, usando o código:

     ```csharp
     rootCommand.AddCommand(loadContentCommand);
     ```

1. Salve suas alterações.

## Tarefa 10 – Testar o código

Só falta testar se o conector do Microsoft Graph importa corretamente conteúdo externo.

1. Abra um terminal.
1. Altere o diretório de trabalho do seu projeto.
1. Compile o projeto executando o comando `dotnet build`.
1. Comece a carregar o conteúdo executando o comando `dotnet run -- load-content`.
1. Aguarde até que o comando seja concluído e carregue o conteúdo.

[Continue no próximo exercício...](./4-exercise-ensure-secure-access.md)