---
lab:
  title: Exercício 1 – Configurar uma conexão externa e implantar esquema
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Exercício – Configurar uma conexão externa e implantar esquema

Neste exercício, você criará um conector personalizado do Microsoft Graph como um aplicativo de console. Você registrará um novo registro de aplicativo do Microsoft Entra e adicionará o código para criar uma conexão externa e implantar o esquema correspondente.

## Antes de começar

Este exercício levará cerca de **XX minutos** para ser concluído.

## Tarefa 1 – Registrar um novo registro de aplicativo do Microsoft Entra

Comece registrando um novo registro de aplicativo do Entra, que o conector personalizado do Graph usa para autenticar com o Microsoft 365.

Em um navegador da Web:

1. Acesse o **portal do Azure** em [https://portal.azure.com](https://portal.azure.com).
1. No painel de navegação, selecione **exibir**embaixo de **Microsoft Entra ID**.
1. No painel de navegação, expanda **Gerenciar** e selecione **Registros de aplicativo**.
1. No painel de navegação superior, selecione **Novo registro**.
1. Especifique os seguintes valores:
   1. **Nome:** docs do MSGraph conetor do Graph
   1. **Tipos de conta aceitos:** contas somente neste diretório organizacional (único locatário)
1. Selecione **Registrar** para confirmar a entrada
1. Na tela de visão geral, copie os valores das propriedades **ID do aplicativo** e **ID do diretório (locatário)**. Você precisará deles mais tarde.

## Tarefa 2 – Criar uma credencial

Como esse conector personalizado do Graph é executado sem interação do usuário, você precisa configurá-lo para autenticar automaticamente. Para simplificar, crie um segredo.

Continuando no navegador da Web:

1. Na navegação lateral, expanda **Gerenciar** e selecione **Certificados e segredos**.
1. Selecione a guia  **Segredos do cliente** e selecione **Novo segredo do cliente**.
1. Insira uma **descrição** do **Segredo do conector do Graph dos documentos do MSGraph**.
1. Crie o segredo selecionando **Adicionar**.
1. Copie o **valor** do segredo recém-criado. Você precisará dela mais tarde.

## Tarefa 3 – Conceder permissões de API

A etapa final da configuração do registro de aplicativo do Entra é conceder a ele permissões de API para criar a conexão externa e o esquema.

Continuando no navegador da Web:

1. No painel de navegação lateral, selecione **Permissões de API**.
1. Selecione **Adicionar permissão**.
1. Selecione **Microsoft Graph** na lista de APIs.
1. Depois, selecione **Permissões de aplicativo**.
1. Na caixa de texto de filtro, digite **externa**.
1. Expanda a seção **ExternalConnection** e selecione a permissão **ExternalConnection.ReadWrite.OwnedBy**.
1. Expanda a seção **ExternalItem** e selecione a permissão **ExternalItem.ReadWrite.OwnedBy**.
1. Para confirmar sua escolha, clique no botão **Adicionar permissões**.
1. Para concluir a configuração, conceda o consentimento do administrador clicando no botão **Conceder consentimento do administrador para (locatário)**.
1. Na caixa de diálogo de confirmação, clique em **Sim**.

## Tarefa 4 – Criar um novo aplicativo de console e instalar dependências

Depois de configurar o registro de aplicativo do Entra, a próxima etapa é criar um aplicativo de console, onde você implementará o código do conector do Graph.

Abra um terminal do Windows para criar um novo aplicativo de console:

1. Crie uma pasta digitando `mkdir documents\console_app` e, em seguida, navegue até a nova pasta digitando `cd .\documents\console_app`.
1. Crie um aplicativo de console executando `dotnet new console`
1. Adicione dependências, que você precisa para criar o conector:
   1. Para adicionar a biblioteca necessária para autenticar com o Microsoft 365, execute `dotnet add package Azure.Identity`.
   1. Para adicionar a biblioteca de cliente para se comunicar com as APIs do Graph, execute `dotnet add package Microsoft.Graph`.
   1. Para adicionar a biblioteca necessária para trabalhar com segredos do usuário, que você configurará na próxima etapa, execute `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`.
   1. Você implementará o conector do Graph como um aplicativo de console com dois comandos: um para criar a conexão externa e implantar o esquema e outro para importar o conteúdo. Para permitir a definição de comandos em seu aplicativo, execute `dotnet add package System.CommandLine --prerelease`.

## Tarefa 5 – Armazenar com segurança as informações de registro de aplicativo do Entra

Depois de criar o registro de aplicativo do Entra, você anotou suas informações, como a ID do aplicativo e do locatário e o segredo. Use essas informações no conector para autenticar com o Microsoft 365. Para disponibilizá-lo em seu código, você o armazena com segurança no projeto como segredos do usuário.

Em um terminal:

1. Verifique se o diretório de trabalho está definido para o aplicativo de console recém-criado.
1. Para iniciar segredos do usuário, execute `dotnet user-secrets init`.
1. Para armazenar com segurança as informações sobre o registro do aplicativo, substitua os tokens pelos valores reais copiados anteriormente e execute:

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## Tarefa 6 – Criar um cliente do Microsoft Graph

Os conectores personalizados do Graph usam a API do Microsoft Graph para gerenciar sua conexão externa e items. Comece criando uma instância da `GraphServiceClient` classe no pacote NuGet do **Microsoft.Graph** que você instalou no projeto.

1. Abra o projeto no Visual Studio 2022.
1. No projeto, adicione um novo arquivo de código chamado **GraphService.cs**.
1. No arquivo, comece adicionando referências aos namespaces que você usará adicionando:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. Em seguida, defina uma nova classe chamada GraphService:

   ```csharp
   class GraphService
   {
   }
   ```

1. Na classe `GraphService`, defina um singleton para armazenar uma instância de `GraphServiceClient` para se comunicar com APIs do Microsoft Graph:

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. No singleton `Client`, implemente o getter, para que ele crie uma nova instância de `GraphServiceClient` se ainda não existir.

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. Na introdução, crie uma nova instância do `GraphServiceClient`, usando uma credencial com as informações sobre o registro de aplicativo do Entra que você armazenou anteriormente:

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   Comece criando um construtor de configuração para acessar as informações sobre o registro de aplicativo do Entra armazenadas em segredos do usuário. Depois, use o construtor para recuperar as informações de registro do aplicativo. Em seguida, crie uma nova credencial de segredo do cliente passando a ID do locatário e do cliente e o segredo do cliente. Por fim, crie uma instância de `GraphServiceClient` passando a credencial recém-criada.

1. O código completo é assim:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. Salvar suas alterações

## Tarefa 7 – Definir conexão externa e configuração de esquema

A próxima etapa é definir a conexão externa e o esquema que o conector do Graph deve usar. Como o código do conector precisa de acesso ao ID da conexão externa em vários lugares, armazene-o em um local central no código.

No editor de código:

1. Crie um arquivo chamado **ConnectionConfiguration.cs**.
1. Adicione uma referência ao namespace com modelos do Microsoft Graph:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Em seguida, no mesmo arquivo, defina uma nova classe estática chamada `ConnectionConfiguration`:

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1. Na classe `ConnectionConfiguration`, adicione uma nova propriedade chamada `ExternalConnection`. Implemente-a para retornar uma instância do modelo do Microsoft Graph `ExternalConnection`:

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. Em seguida, adicione outra propriedade chamada `Schema`. Implemente-a para retornar uma instância do modelo do Graph `Schema`:

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. No esquema, defina as propriedades que você acompanha para cada item externo que você ingerir usando o conector:

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   Comece definindo a propriedade **title**, que armazena o título do item externo importado para o Microsoft 365. O título do item faz parte do índice de texto completo (`IsSearchable = true`). Os usuários também podem consultar explicitamente seu conteúdo em consultas de palavra-chave (`IsQueryable = true`). O título também pode ser recuperado e exibido nos resultados da pesquisa (`IsRetrievable = true`). A propriedade **title** representa o título do item, que você indica usando o rótulo semântico `Title`.

   Depois, defina a propriedade **description**, que armazena o resumo do conteúdo do item externo. A definição é semelhante ao título. No entanto, não há rótulo semântico para a descrição, e é por isso que você não a define.

   Em seguida, defina uma propriedade para armazenar a URL do ícone de cada item. O Copilot para Microsoft 365 requer essa propriedade e ela precisa ser mapeada usando o rótulo semântico `IconUrl`.

   Por fim, defina a propriedade **url**, que armazena a URL original do item externo. Os usuários usam essa URL para navegar até o item externo a partir dos resultados da pesquisa ou do Copilot do Microsoft 365. A URL é uma das propriedades que o Copilot para Microsoft 365 requer, e é por isso que você mapeá-la usando o rótulo semântico `Url`.

1. O código completo é assim:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. Salvar suas alterações

## Tarefa 8 – Criar conexão externa

Continue adicionando código que usa as informações sobre a conexão externa definida na seção anterior para criar a conexão externa no Microsoft 365.

No editor de código:

1. Crie um arquivo chamado **ConnectionService.cs**.
1. No arquivo, comece adicionando referências aos namespaces:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Em seguida, no mesmo arquivo, defina uma nova classe estática chamada `ConnectionService`:

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. Na classe `ConnectionService`, adicione um novo método chamado `CreateConnection`:

   ```csharp
   async static Task CreateConnection()
   {

   }
   ```

1. No método `CreateConnection`, use a instância do cliente do Microsoft Graph para chamar APIs do Microsoft Graph e criar a conexão externa usando as informações de conexão definidas anteriormente:

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. O código completo é assim:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. Salve suas alterações.

Antes de testar esse código, adicione-o para criar o esquema. Dessa forma, você pode testar o fluxo completo de criação e configuração da conexão externa.

## Tarefa 9 – Criar esquema de conexão externa

A última parte da criação de uma conexão externa é criar seu esquema.

No editor de código:

1. Abra o arquivo **ConnectionService.cs**.
1. Na classe ConnectionService, adicione um novo método chamado `CreateSchema`:

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. No método `CreateSchema`, use a instância do cliente do Microsoft Graph para chamar a API do Microsoft Graph para criar o esquema. Aguarde a criação.

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. No mesmo arquivo, adicione um novo método chamado `ProvisionConnection`. Em seu código, chame os métodos `CreateConnection` e `CreateSchema` definidos anteriormente:

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. O código completo é assim:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. Salve suas alterações.

## Tarefa 10 – Testar o código

A última etapa restante é criar um ponto de entrada no aplicativo, o que criará a conexão e o esquema dela. Para fazer isso, crie um comando que você invoca iniciando o aplicativo a partir da linha de comando.

No editor de código:

1. Abra o arquivo **Program.cs**.
1. Substitua o conteúdo pelo seguinte código:

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   Comece definindo um comando chamado `provision-connection.`; este comando invocará o método `ConnectionService.ProvisionConnection` que você definiu anteriormente. Por fim, registre o comando com o processador da linha de comando e inicie os argumentos de monitoramento do aplicativo passados na linha de comando.

1. Salvar suas alterações

Para testar o aplicativo:

1. Abra um terminal.
1. Altere o diretório de trabalho para a pasta do projeto.
1. Execute `dotnet build` para compilar o projeto.
1. Inicie o aplicativo executando `dotnet run -- provision-connection`.
1. Aguarde alguns minutos até que a conexão e o esquema sejam criados.

[Continue no próximo exercício...](./3-exercise-import-external-content.md)