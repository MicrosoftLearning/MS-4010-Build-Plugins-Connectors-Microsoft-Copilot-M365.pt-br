---
lab:
  title: Preparar seu ambiente de desenvolvimento
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Preparar seu ambiente de desenvolvimento

Primeiro, vamos preparar seu ambiente de desenvolvimento, contas e software. Antes de começar, você deverá concluir as seguintes tarefas.

## Tarefa 1 – Pré-requisitos de instalação

> [!IMPORTANT]
> Para concluir este projeto com êxito, você precisará de uma conta do Microsoft 365 com permissão para carregar aplicativos. Para concluir o **Exercício 2**, a conta também deve ser licenciada para o Microsoft Copilot para Microsoft 365.

Se você estiver usando um novo locatário, é uma boa ideia fazer logon na [página do Microsoft 365](https://office.com) em [https://office.com](https://office.com) antes de começar. Dependendo de como o locatário está configurado, talvez seja solicitada a configuração da autenticação multifator. Confirme se você consegue acessar o Microsoft Teams e o Microsoft Outlook antes de continuar.

As ferramentas a seguir já foram instaladas no laboratório no **MS-4010-CLIENT01**. Certifique-se de que estão instalados e operáveis:

1. [Visual Studio Code](https://code.visualstudio.com/) (versão mais recente)

1. [Gerenciador de Armazenamento do Azure](https://azure.microsoft.com/products/storage/storage-explorer/) – Baixe se você quiser ver e editar o banco de de dados da Northwind utilizado neste exemplo.

<!--## Task 2 - Install nvm-windows

You'll use this tool to install Node.js and optionally switch Node versions as needed for your projects.

1. In a web browser, navigate to [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases).
2. Locate the latest release version and select the **nvm-setup.zip** file to download.  The file will be downloaded to your machine.
3. Open the file folder and **extract** the contents of the zip folder to a folder on your machine.
4. From the new folder, select **nvm-setup.exe** to open the setup file.
5. Follow the prompts in the installer to install the tool using the default options.
6. Nvm for Windows will be installed on your machine.

## Task 3 - Install Node.js

Install Node.js version 18.18.2, which is compatible with all of the solutions in this course.

1. Open the **Command Prompt** application.
2. Enter the command `nvm install 18.18` to install Node.js.
3. The nvm output should confirm that installation is complete.
4. Run the command `nvm use 18.18` to use this version of Node.js.
5. Run the command `node -v` to confirm that you have version 18.18.2 installed.

You have now installed and configured Node.js version 18.18.2-->

## Tarefa 2 – Baixar o código de exemplo

[Baixe](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/) o repositório de exemplo como um arquivo ZIP e extraia-o na pasta **Documentos**:

```text
https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/
```

No repositório, navegue até a pasta **samples/msgext-northwind-inventory-ts**. Esses laboratórios se referirão a essa pasta como sua "**pasta de trabalho**", já que é onde você trabalhará.

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

1. Abra a **pasta de trabalho** no Visual Studio Code. Pode ser solicitado que você confie nos autores desta pasta; se isso acontecer, confie. No **menu Arquivo do Visual Studio Code**, selecione **Abrir pasta** e abra diretamente a pasta **msgext-northwind-inventory-ts**.

1. Agora selecione o ícone do **Kit de Ferramentas do Teams** à esquerda 1️⃣. Se oferecer opções para criar um novo projeto, você provavelmente está na pasta errada.  Você verá as seções de Contas, Meio Ambiente, etc., conforme mostrado abaixo.

1. Em **Contas**, selecione **Entrar no Microsoft 365** 2️⃣ e entre com sua conta do Microsoft 365.

    ![Captura de tela da entrada no Microsoft 365 a partir do Kit de Ferramentas do Teams.](../media/1-04-setup-teams-toolkit-01.png)

1. Uma janela do navegador será aberta oferecendo o login no Microsoft 365. Quando aparecer **Você entrou e fechar esta página**, feche-a-.

1. Por fim, verifique se uma marca de seleção verde aparece ao lado de **Upload de aplicativo personalizado habilitado**. Se não aparecer, isso significa que sua conta de usuário não tem permissão para carregar aplicativos do Teams. Essa permissão está "desativada" por padrão; confira estas [instruções para permitir que os usuários carreguem aplicativos personalizados](https://learn.microsoft.com/microsoftteams/teams-custom-app-policies-and-settings#allow-users-to-upload-custom-apps)

    ![Captura de tela mostrando que o sideload está habilitado.](../media/1-04-setup-teams-toolkit-03.png)

## Verifique seu trabalho

Depois de seguir todas as tarefas acima, você terá instalado e baixado no seu computador:

- [Visual Studio Code](https://code.visualstudio.com/) (versão mais recente)

- [NodeJS versão 18.x](https://nodejs.org/download/release/v18.18.2/)

- [Gerenciador de Armazenamento do Azure](https://azure.microsoft.com/products/storage/storage-explorer/) (OPCIONAL)

- [Kit de Ferramentas do Teams para Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5)

- Repositório de exemplo: [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/)

Se tudo tiver sido preparado corretamente, agora está tudo pronto para executar o aplicativo de exemplo como uma extensão de mensagem. 

[Continue no próximo exercício...](./3-exercise-1-run-message-extension.md)