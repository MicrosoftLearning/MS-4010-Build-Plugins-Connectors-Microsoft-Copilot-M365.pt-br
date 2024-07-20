---
lab:
  title: Introdução
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Introdução

As extensões de mensagem permitem que os usuários trabalhem com sistemas externos do Microsoft Teams e do Microsoft Outlook. Os usuários podem usar extensões de mensagem para procurar e alterar dados e compartilhar as informações desses sistemas em mensagens e emails como um cartão com formatação avançada.

Suponha que você tenha uma Lista do SharePoint Online com informações de produto atuais e relevantes para sua organização. Você quer pesquisar e compartilhar essas informações no Microsoft 365. Você também quer que o Copilot para Microsoft 365 use essas informações em suas respostas.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="Captura de tela da página inicial do site da equipe do SharePoint Online de suporte ao produto. Uma lista de produtos lançados recentemente é exibida." lightbox="../media/1-sharepoint-online-product-support-site.png":::

Neste módulo, você criará uma extensão de mensagem. Sua extensão de mensagem usa um bot para se comunicar com o Microsoft Teams, Microsoft Outlook e Copilot para Microsoft 365.

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams." lightbox="../media/2-search-results-nuget.png":::

Ele usa o Microsoft Entra para autenticar usuários, o que permite que retorne dados do SharePoint Online usando a API do Microsoft Graph em seu nome.

:::image type="content" source="../media/3-sign-in.png" alt-text="Captura de tela de um desafio de autenticação em uma extensão de mensagem baseada em pesquisa. Um link de entrada é exibido." lightbox="../media/3-sign-in.png":::

Depois que o usuário se autenticar, sua extensão de mensagem obterá informações sobre o produto do SharePoint Online usando a API do Microsoft Graph. Ela retornará resultados da pesquisa que podem ser incorporados em mensagens e emails como um cartão formatação avançada e, em seguida, compartilhados.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams. Os resultados da pesquisa são retornados do SharePoint Online. Cada resultado da pesquisa exibe o nome do produto, a categoria e a imagem do produto." lightbox="../media/4-search-results-sharepoint-online.png":::

:::image type="content" source="../media/5-adaptive-card.png" alt-text="Captura de tela do resultado da pesquisa incorporada em uma mensagem no Microsoft Teams. Os resultados da pesquisa são renderizados como um Cartão Adaptável com o nome do produto, a categoria, o volume de chamadas e a data de lançamento. É exibido um botão de ação com o título Exibir que os usuários podem usar para navegar até o item da lista de produtos no SharePoint Online." lightbox="../media/5-adaptive-card.png":::

Ele funciona com o Copilot para Microsoft 365 como um plug-in, permitindo que consulte a Lista do SharePoint Online em nome do usuário e use os dados retornados em suas respostas.

:::image type="content" source="../media/6-copilot-answer.png" alt-text="Captura de tela de uma resposta no Copilot para Microsoft 365 que contém informações retornadas pelo plug-in de extensão de mensagem. Um cartão adaptável é exibido mostrando informações do produto." lightbox="../media/6-copilot-answer.png":::

Ao final deste módulo, você conseguirá criar extensões de mensagem escritas em C# (em execução no .NET). Elas podem ser usadas no Microsoft Teams, Microsoft Outlook e Copilot para Microsoft 365, consultar dados por trás de APIs protegidas e retornar os resultados como cartões com formatação avançada.

## Pré-requisitos

- Conhecimento básico do C#
- Conhecimento básico do Bicep
- Conhecimento básico de autenticação
- Acesso de administrador a um locatário do Microsoft 365
- Acesso a uma assinatura do Azure
- O acesso ao Copilot para Microsoft 365 é opcional e necessário apenas para concluir um exercício
- Visual Studio 2022 17.9 com o [Kit de Ferramentas do Teams](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (componente de ferramentas de desenvolvimento do Microsoft Teams) instalado
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Quando estiver tudo pronto para começar, [continue no próximo exercício...](./2-exercise-create-a-message-extension.md)
