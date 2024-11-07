---
lab:
  title: Exercício 1 – Criar uma extensão de mensagem
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercício 1 – Criar uma extensão de mensagem

Neste exercício, você criará uma solução de extensão de mensagem. Você usará o kit de ferramentas do Teams no Visual Studio para criar os recursos necessários, iniciará uma sessão de depuração e testará no Microsoft Teams.

![Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams.](../media/1-search-results.png)

### Duração do exercício

  - **Tempo estimado para conclusão:** 25 minutos

## Tarefa 1 – Criar um novo projeto com o Kit de Ferramentas do Teams para o Visual Studio

Comece criando um novo projeto de aplicativo do Microsoft Teams configurado com uma extensão de mensagem que contém um comando de pesquisa. Embora você possa criar um projeto usando um modelo de projeto do Kit de Ferramentas do Teams para o Visual Studio, há alterações necessárias a serem feitas no projeto com scaffold para poder concluir este módulo. Em vez disso, você usará um modelo de projeto personalizado que está disponível como um pacote NuGet. A vantagem de usar um modelo personalizado é que ele cria uma solução com os arquivos e dependências necessários, economizando seu tempo.

1. Abra uma sessão do PowerShell como Administrador.

1. Mude para um diretório de trabalho apropriado executando:

    ```Powershell
    cd ~\Documents
    ```

1. Comece instalando o pacote de modelo do NuGet executando:

    ```PowerShell
    dotnet new install M365Advocacy.Teams.Templates
    ```

1. Crie um novo projeto executando:

    ```PowerShell
    dotnet new teams-msgext-search --name "ProductsPlugin" `
      --internal-name "msgext-products" `
      --display-name "Contoso products" `
      --short-description "Product look up tool." `
      --full-description "Get real-time product information and share them in a conversation." `
      --command-id "Search" `
      --command-description "Find products by name" `
      --command-title "Products" `
      --parameter-name "ProductName" `
      --parameter-title "Product name" `
      --parameter-description "The name of the product as a keyword" `
      --allow-scripts Yes
    ```

1. Aguarde a criação do projeto.

1. Altere para o diretório do projeto executando `cd ProductsPlugin`.

1. Abra a solução no Visual Studio executando `.\ProductsPlugin.sln`.

1. Selecione **Visual Studio 2022** na janela de seleção de aplicativos e, em seguida, **Sempre**.

## Tarefa 2 – Criar um túnel de desenvolvedor

Quando o usuário interage com a sua extensão de mensagem, o serviço de bot envia solicitações para o serviço Web. Durante o desenvolvimento, o serviço Web é executado localmente no seu computador. Para permitir que o serviço de bot alcance seu serviço Web, você precisa expô-lo além do seu computador usando um Túnel do desenvolvedor.

![Captura de tela da janela Túnel do desenvolvedor no Visual Studio.](../media/14-select-dev-tunnel.png)

Continuando no Visual Studio:

1. Na barra de ferramentas, clique na lista suspensa ao lado do botão **Iniciar**, expanda o menu **Túnel do desenvolvedor (sem túnel ativo)** e clique em **Criar um túnel**.

1. Na caixa de diálogo, especifique os seguintes valores:

    1. **Conta**: entre usando a conta do Microsoft 365 fornecida a você. Selecione **Conta corporativa ou de estudante**.

    1. **Nome**: msgext-products

    1. **Tipo de túnel**: temporário

    1. **Acesso:** público

1. Crie o túnel clicando em **OK**. Um prompt será exibido informando que o novo túnel agora é o túnel ativo atual.

1. Feche o prompt clicando em **Ok**.

## Tarefa 3 – Preparar os recursos

Com tudo agora pronto, use o Kit de Ferramentas do Teams para executar o processo **Preparar dependências do aplicativo Teams** para criar os recursos necessários.

![Captura de tela do menu do Kit de Ferramentas do Teams expandido no Visual Studio Code.](../media/15-prepare-teams-app-dependencies.png)

O processo Preparar dependências de aplicativo do Teams atualizará as variáveis de ambiente **BOT_ENDPOINT** and **BOT_DOMAIN** no arquivo **TeamsApp\\env\\.env.local** usando a URL do túnel de desenvolvimento ativo e executará as ações descritas no arquivo **TeamsApp\\teamsapp.local.yml**.

Reserve um momento para explorar as etapas no arquivo **teamsapp.local.yml**.

Continuando no Visual Studio:

1. Abra o menu **Projeto** (ou clique com o botão direito no projeto **TeamsApp** no Gerenciador de Soluções), expanda o menu **Kit de Ferramentas do Teams** e selecione **Preparar dependências do aplicativo Teams**.

1. Na caixa de diálogo da **conta do Microsoft 365**, entre ou selecione uma conta existente para acessar seu locatário do Microsoft 365 e selecione **Continuar**.

1. Na caixa de diálogo **Provisionar**, faça login ou selecione uma conta já existente a ser usada para implantar recursos no Azure e especifique os seguintes valores:

      1. **Nome da assinatura**: use a lista suspensa para selecionar a assinatura.

      1. **Grupo de recursos**: selecione o grupo de recursos preenchido na lista suspensa.

1. Crie os recursos no Azure clicando em **Provisionar**.

1. No prompt de aviso do Kit de Ferramentas do Teams, clique em **Provisionar**.

1. No prompt de informações do Kit de Ferramentas do Teams, selecione **Exibir recursos provisionados** para abrir uma nova janela do navegador.

Reserve um momento para explorar os recursos criados no Azure e também exibir as variáveis de ambiente criadas no arquivo **.env.local**.

> [!NOTE]
> Quando você fechar e reabrir o Visual Studio, a URL do Túnel do desenvolvedor será alterada e não estará mais selecionada como o túnel ativo. Se isso acontecer, você precisará selecionar o túnel novamente e executar o processo **Preparar dependências do aplicativo Teams** para refletir a URL atualizada no manifesto do aplicativo.

## Tarefa 4 – Executar e depurar

O Kit de Ferramentas do Teams usa perfis de inicialização de vários projetos. Para executar o projeto, você precisa habilitar uma versão prévia do recurso no Visual Studio.

No Visual Studio:

1. Abra o menu **Ferramentas** e selecione **Opções...**

1. Na caixa de pesquisa, insira **multi-project**.

1. Em **Ambiente**, selecione **Visualizar recursos**.

1. Marque a caixa ao lado de **Habilitar perfis de inicialização de vários projetos** e clique em **OK** para salvar as alterações.

Para iniciar uma sessão de depuração e instalar o aplicativo no Microsoft Teams:

1. Pressione <kbd>F5</kbd> ou selecione **Iniciar** na barra de ferramentas.

1. Confie ou aprove quaisquer avisos de certificação SSL que apareçam ao iniciar o aplicativo pela primeira vez.

1. Aguarde até que uma janela do navegador seja aberta e a caixa de diálogo de instalação do aplicativo apareça no cliente Web do Microsoft Teams. Se solicitado, insira as credenciais da sua conta do Microsoft 365.

1. Na caixa de diálogo de instalação do aplicativo, selecione **Adicionar**.

Para testar a extensão de mensagem:

1. Abra um novo chat (<kbd>Alt+N</kbd>) e comece a digitar **Contoso** na caixa **Para**, selecionando **Suporte ao Produto da Contoso**.

    > [!NOTE]
    > Não funcionará se você conversar com sua própria conta de usuário. Precisa ser outro usuário ou grupo.

1. Na área de composição da mensagem, digite **/apps** para abrir o seletor de aplicativo.

1. Na lista de aplicativos, selecione **Produtos da Contoso** para abrir a extensão de mensagem.

1. Na caixa de texto, insira **oi**. Pode ser necessário inserir sua pesquisa várias vezes.

1. Aguarde até que os resultados da pesquisa apareçam.

1. Na lista de resultados, clique em **oi** para inserir um cartão na caixa de redigir mensagem.

![Captura de tela dos resultados da pesquisa retornados por uma extensão de mensagem baseada em pesquisa no Microsoft Teams.](../media/1-search-results.png)

Retorne ao Visual Studio e selecione **Parar** na barra de ferramentas ou pressione <kbd>Shift</kbd> + <kbd>F5</kbd> para interromper a sessão de depuração.

[Continue no próximo exercício...](./3-exercise-add-single-sign-on.md)