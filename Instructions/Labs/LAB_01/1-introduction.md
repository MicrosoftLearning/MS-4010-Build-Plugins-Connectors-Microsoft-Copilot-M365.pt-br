---
lab:
  title: Introdução
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Introdução

As extensões de mensagem permitem que os usuários trabalhem com sistemas externos do Microsoft Teams e do Microsoft Outlook. Os usuários podem usar extensões de mensagem para procurar, alterar e compartilhar dados desses sistemas em mensagens e emails como um cartão com formatação avançada.

Suponha que você tenha uma API personalizada que usa para acessar informações do produto que são atuais e relevantes para sua organização. Você quer pesquisar e compartilhar essas informações no Microsoft 365. Você também quer que o Microsoft 365 Copilot use essas informações em suas respostas.

Neste módulo, você criará uma extensão de mensagem. Sua extensão de mensagem usa um bot para se comunicar com o Microsoft Teams, Microsoft Outlook e o Microsoft 365 Copilot.

![Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams.](../media/1-search-results.png)

Ele usa o Microsoft Entra para autenticar usuários, o que permite que retorne dados da API em seu nome.

Depois que o usuário se autenticar, sua extensão de mensagem obterá dados da API e retornará os resultados de pesquisa que podem ser inseridos em mensagens e emails como um cartão formatado avançado e, em seguida, compartilhados.

![Captura de tela dos resultados da pesquisa que usam dados de uma API externa no Microsoft Teams.](../media/3-search-results-api.png)

![Captura de tela do resultado da pesquisa inserido em uma mensagem no Microsoft Teams.](../media/4-adaptive-card.png)

Ele funciona com o Microsoft 365 Copilot como um plug-in, permitindo que consulte os dados do produto em nome do usuário e use os dados retornados em suas respostas.

![Captura de tela de uma resposta no Microsoft 365 Copilot que contém informações retornadas pelo plug-in de extensão de mensagem. Um cartão adaptável é exibido mostrando informações do produto.](../media/5-copilot-answer.png)

Ao final deste módulo, você conseguirá criar extensões de mensagem escritas em C# (em execução no .NET). Elas podem ser usadas no Microsoft Teams, Microsoft Outlook e no Microsoft 365 Copilot. consultar dados por trás de APIs protegidas e retornar os resultados como cartões com formatação avançada.

## Pré-requisitos

- Conhecimento básico do C#
- Conhecimento básico do Bicep
- Conhecimento básico de autenticação
- Acesso de administrador a um locatário do Microsoft 365
- Acesso a uma assinatura do Azure
- O acesso ao Microsoft 365 Copilot é opcional e necessário apenas para concluir o **Exercício 4: Tarefa 5**.
- Visual Studio 2022 17.10+ com o [Kit de Ferramentas do Teams](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (componente de ferramentas de desenvolvimento do Microsoft Teams) instalado
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Proxy de desenvolvimento 0.19.1+](https://aka.ms/devproxy)

> [!NOTE]
> O único exercício neste laboratório que requer uma licença do Microsoft 365 Copilot é o **Exercício 4: Tarefa 5**. Tudo até esse ponto deve ser feito, independentemente de seu locatário ter o Copilot ou não.

## Duração do laboratório

  - **Tempo estimado para conclusão:** 150 minutos

## Objetivos do aprendizado

Ao final deste módulo, você estará apto a:

- Entender o que são extensões de mensagem e como criá-las.
- Criar uma extensão de mensagem.
- Entender como autenticar usuários usando logon único e chamar uma API personalizada protegida com autenticação do Microsoft Entra.
- Entender como estender e otimizar extensões de mensagem para uso com o Microsoft 365 Copilot.

Quando estiver tudo pronto para começar, [continue para o primeiro exercício...](./2-exercise-create-a-message-extension.md)
