---
lab:
  title: Preparar seu ambiente de desenvolvimento
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Preparar seu ambiente de desenvolvimento

Primeiro, vamos preparar seu ambiente de desenvolvimento, contas e software. Antes de começar, você deverá concluir as seguintes tarefas.

## Tarefa 1 – Pré-requisitos de instalação

> [!IMPORTANT]
> Para concluir este projeto com êxito, você precisará de uma conta do Microsoft 365 com permissão para carregar aplicativos. Para concluir o **Exercício 2**, a conta também deve ser licenciada para o Microsoft Copilot para Microsoft 365.

Se você estiver usando um novo locatário, é uma boa ideia fazer logon na [página do Microsoft 365](https://office.com) em [https://office.com](https://office.com) antes de começar. Dependendo de como o locatário está configurado, talvez seja solicitada a configuração da autenticação multifator. Confirme se você consegue acessar o Microsoft Teams e o Microsoft Outlook antes de continuar.

As ferramentas a seguir já foram instaladas no laboratório no **MS-4010-DEVBOX.** Certifique-se de que estão instalados e operáveis:

1. [Visual Studio Code](https://code.visualstudio.com/) (versão mais recente)

1. [Gerenciador de Armazenamento do Azure](https://azure.microsoft.com/products/storage/storage-explorer/) – Baixe se você quiser ver e editar o banco de de dados da Northwind utilizado neste exemplo.

## Tarefa 2 – Instalar nvm-windows

Você usará essa ferramenta para instalar o Node.js e, se necessário, alternar as versões do Node para seus projetos.

1. Em um navegador da Web, acesse [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases).
2. Localize a versão mais recente e selecione o arquivo **nvm-setup.zip** para baixar.  O arquivo será baixado no seu computador.
3. Abra a pasta do arquivo e **extraia** o conteúdo da pasta zip em um diretório no seu computador.
4. Na nova pasta, abra o arquivo **nvm-setup.exe** para iniciar a instalação.
5. Siga as solicitações do instalador para instalar a ferramenta usando as opções padrão.
6. O nvm para Windows será instalado no seu computador.

## Tarefa 3 – Instalar Node.js

Instale a versão 18.18.2 do Node.js, que é compatível com todas as soluções neste curso.

1. Abra o aplicativo **Prompt de Comando**.
2. Insira o comando `nvm install 18.18` para instalar o Node.js.
3. A saída do nvm deve confirmar a conclusão da instalação.
4. Execute o comando `nvm use 18.18` para usar esta versão do Node.js.
5. Execute o comando `node -v` para confirmar se você tem a versão 18.18.2 instalada.

Você instalou e configurou a versão 18.18.2 do Node.js

## Tarefa 4 – Baixar o código de exemplo

[Clone](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples.git) ou [baixe](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples.git) o repositório de exemplo: [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/).

No repositório clonado ou baixado, navegue até a pasta **samples/msgext-northwind-inventory-ts**. Esses laboratórios se referirão a essa pasta como sua "**pasta de trabalho**", já que é onde você trabalhará.

## Tarefa 3 – Copiar documentos de exemplo para o OneDrive

O aplicativo de exemplo inclui alguns documentos para o Copilot referenciar durante os laboratórios. Nesta tarefa, você copiará esses arquivos para o OneDrive do usuário para que o Copilot possa encontrá-los. Dependendo de como o locatário estiver configurado, talvez seja solicitado que você configure a autenticação multifator como parte desse processo.

1. Abra seu navegador da Web e navegue até o Microsoft 365 ([https://www.office.com/](https://www.office.com/)). Entre usando a conta do Microsoft 365 que você usará no laboratório. Talvez você precise configurar a autenticação multifator.

1. Usando o menu de aplicativos no canto superior esquerdo da página 1️⃣, navegue até o aplicativo OneDrive no Microsoft 365 2️⃣.

    ![Captura de tela da navegação até o aplicativo OneDrive no Microsoft 365.](../media/1-02-copy-sample-files-01.png)

1. No OneDrive, navegue até **Meus arquivos** 1️⃣. Se houver uma pasta de documentos, navegue até ela também. Caso contrário, você pode trabalhar diretamente no local **Meus arquivos**.

    ![Captura de tela da navegação até os documentos no OneDrive.](../media/1-02-copy-sample-files-02.png)

1. Agora selecione **Adicionar novo** 1️⃣ e **Pasta** 2️⃣ para criar uma nova pasta.

    ![Captura de tela mostrando a adição de uma nova pasta no OneDrive.](../media/1-02-copy-sample-files-03.png)

1. Nomeie a pasta **contratos da Northwind** e clique em **Criar**.

    ![Captura de tela da nomeação da nova pasta "contratos da Northwind".](../media/1-02-copy-sample-files-03-b.png)

1. Agora, dentro desta nova pasta, clique em **Adicionar novo** 1️⃣ novamente, mas desta vez clique em **Upload de arquivos** 2️⃣.

    ![Captura de tela da adição de novos arquivos à nova pasta.](../media/1-02-copy-sample-files-04.png)

1. Agora navegue até a pasta **sampleDocs** dentro da sua **pasta de trabalho**. Realce todos os arquivos 1️⃣ e clique em **Ok** 2️⃣ para carregar todos.

    ![Captura de tela do upload dos arquivos de exemplo desse repositório para a pasta.](../media/1-02-copy-sample-files-05.png)

Ao fazer essa tarefa com antecedência, há uma boa chance de que o mecanismo de pesquisa do Microsoft 365 os tenha descoberto quando você estiver tudo pronto para eles.

## Tarefa 4: Configurar o Kit de Ferramentas do Teams para Visual Studio Code

Nesta tarefa, você instalará a versão atual do [Kit de Ferramentas do Teams para Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5). A maneira mais fácil de fazer isso é diretamente no Visual Studio Code.

1. Abra a **pasta de trabalho** no Visual Studio Code. Pode ser solicitado que você confie nos autores desta pasta; se isso acontecer, confie.

1. Agora selecione o ícone do **Kit de Ferramentas do Teams** à esquerda 1️⃣. Se oferecer opções para criar um novo projeto, você provavelmente está na pasta errada. No **menu Arquivo do Visual Studio Code**, selecione **Abrir pasta** e abra diretamente a pasta **msgext-northwind-inventory-ts**. Você verá as seções de Contas, Meio Ambiente, etc., conforme mostrado abaixo.

1. Em **Contas**, selecione **Entrar no Microsoft 365** 2️⃣ e entre com sua conta do Microsoft 365.

    ![Captura de tela da entrada no Microsoft 365 a partir do Kit de Ferramentas do Teams.](../media/1-04-setup-teams-toolkit-01.png)

1. Uma janela do navegador será aberta oferecendo o login no Microsoft 365. Quando aparecer **Você entrou e fechar esta página**, feche-a-.

1. Por fim, verifique se uma marca de seleção verde aparece ao lado de **Upload de aplicativo personalizado habilitado**. Se não aparecer, isso significa que sua conta de usuário não tem permissão para carregar aplicativos do Teams. Essa permissão está "desativada" por padrão; confira estas [instruções para permitir que os usuários carreguem aplicativos personalizados](https://learn.microsoft.com/microsoftteams/teams-custom-app-policies-and-settings#allow-users-to-upload-custom-apps)

    ![Captura de tela mostrando que o sideload está habilitado.](../media/1-04-setup-teams-toolkit-03.png)

> [!NOTE]
> O Programa para Desenvolvedores do Microsoft 365 não inclui licenças do Copilot para Microsoft 365. Dessa forma, se você decidir usar um locatário de desenvolvedor, poderá testar o exemplo apenas como uma Extensão de Mensagem.

## Verifique seu trabalho

Depois de seguir todas as tarefas acima, você terá instalado e baixado no seu computador:

- [Visual Studio Code](https://code.visualstudio.com/) (versão mais recente)

- [NodeJS versão 18.x](https://nodejs.org/download/release/v18.18.2/)

- [Gerenciador de Armazenamento do Azure](https://azure.microsoft.com/products/storage/storage-explorer/) (OPCIONAL)

- [Kit de Ferramentas do Teams para Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5)

- Repositório de exemplo: [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/)

Se tudo tiver sido preparado corretamente, agora está tudo pronto para executar o aplicativo de exemplo como uma extensão de mensagem. 

[Continue no próximo exercício...](./3-exercise-1-run-message-extension.md)