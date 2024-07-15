---
lab:
  title: Exercício 4 – Estender e otimizar extensões de mensagem para uso com o Copilot para Microsoft 365
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercício 4 – Estender e otimizar extensões de mensagem para uso com o Copilot para Microsoft 365

Neste exercício, você estenderá e otimizará sua extensão de mensagem para uso com o Copilot para Microsoft 365. Você adicionará um novo parâmetro chamado Público-alvo e atualizará a lógica de extensão de mensagem para manipular vários parâmetros. Por último, você executará e depurará a extensão de mensagem e a testará no Copilot no Microsoft Teams.

## Tarefa 1 – Atualizar o manifesto do aplicativo

Especificar descrições concisas e precisas no manifesto do aplicativo é fundamental para garantir que o Copilot saiba quando e como invocar o seu plug-in. Atualize as descrições do aplicativo, do comando e do parâmetro no manifesto do aplicativo.

Abra o Visual Studio:

1. Na pasta **appPackage**, abra o arquivo chamado **manifest.json**
1. Atualize o objeto **descrição**

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    Como vamos adicionar outro parâmetro ao comando, atualize também a descrição do comando para incluir o novo parâmetro.

1. Na matriz **comandos**, atualize a **descrição** do comando

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    Agora adicione um novo parâmetro que o Copilot pode usar. Esse novo parâmetro ajuda os usuários a pesquisar produtos usando o Copilot voltado para diferentes públicos, como pessoas físicas e jurídicas.

1. Na matriz **parâmetros**, adicione o parâmetro **TargetAudience** após o parâmetro **ProductName**.

    ```json
    {    
        "parameters": [
            {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
            },
            {
                "name": "TargetAudience",
                "title": "Target audience",
                "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
                "inputType": "text"
            }
        ]
    }
    ```

1. Salvar suas alterações

A descrição do parâmetro **TargetAudience** descreve o que ele é e explica que o parâmetro deve aceitar **Consumidor** ou **Empresa** como valores permitidos.

## Tarefa 2 – Atualizar lógica de extensão de mensagem

Para aceitar o novo parâmetro e prompts complexos, atualize o método OnTeamsMessagingExtensionQueryAsync no Manipulador de Atividades de Bot para manipular vários parâmetros.

Digamos que um usuário insira o prompt "Localizar produtos da Contoso destinados a indivíduos com o nome Mark8". Dadas as descrições do parâmetros, "destinados a indivíduos" é traduzido para **Consumidor** e passado como o valor do parâmetro **TargetAudience**. "Mark8" é passado como o valor do parâmetro **ProductName**.

Continuando no Visual Studio:

1. Na pasta **Pesquisar**, abra o arquivo chamado **SearchApp.cs**
1. No método **OnTeamsMessagingExtensionQueryAsync**, localize o bloco de códigos abaixo:

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. Atualize o bloco de códigos para obter o valor do parâmetro **TargetAudience** e crie uma consulta de filtro para usar ao consultar a lista do SharePoint Online.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Salvar suas alterações

## Tarefa 3 – Provisionar recursos

Execute o processo Preparar dependências do aplicativo Teams para provisionar recursos.

Continuando no Visual Studio:

1. No **Gerenciador de Soluções**, clique com o botão direito do mouse no projeto **MsgExtProductSupport**
1. Expanda o menu Kit de Ferramentas** do Teams**, selecione **Preparar dependências do aplicativo Teams**
1. Na caixa de diálogo **Conta do Microsoft 365**, selecione **Continuar**
1. Na caixa de diálogo **Provisionar**, selecione **Provisionar**
1. Na caixa de diálogo de **aviso do Kit de Ferramentas do Teams**, selecione **Provisionar**
1. Na caixa de diálogo **informações do Kit de Ferramentas do Teams**, **feche** o prompt

## Tarefa 4 – Executar e depurar

Agora inicie o serviço Web e teste a extensão de mensagem no Copilot para Microsoft 365.

1. Aperte **F5** para iniciar uma sessão de depuração e abrir uma nova janela do navegador que navega até o cliente Web do Microsoft Teams.
1. Insira suas credenciais da conta do Microsoft 365 e continue no Microsoft Teams.
1. Na caixa de diálogo de instalação do aplicativo, selecione **Adicionar**
1. Abra o aplicativo **Copilot** pelo Microsoft Teams
1. Na área de redigir mensagem, abra o submenu **Plug-ins**
1. Na lista de plug-ins, alterne o plug-in de ** produtos da Contoso** para habilitá-lo
1. Insira **Localizar produtos da Contoso destinados a indivíduos** como mensagem e envie-a
1. Na resposta do Copiloto, um botão de entrar é retornado, clique no botão **Entrar** para autenticar
1. Depois de autenticar, **Digite Localizar produtos da Contoso destinados a indivíduos** como mensagem e envie-a
1. Na resposta do Copilot, os dados retornados na resposta do plug-in são exibidos e o plug-in é referenciado na resposta
1. Para exibir o Cartão Adaptável relevante para o resultado, passe o mouse sobre as referências na resposta do Copilot

Feche o navegador para interromper a sessão de depuração.

[Continuar no próximo resumo do laboratório...](./6-summary.md)