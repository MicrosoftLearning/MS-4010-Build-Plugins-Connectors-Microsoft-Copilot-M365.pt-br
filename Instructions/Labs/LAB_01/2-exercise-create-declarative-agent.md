---
lab:
  title: Exercício 1 — Criar um agente declarativo no Visual Studio Code
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Exercício 1 – Criar um agente declarativo

Neste exercício, você criará um projeto de agente declarativo a partir de um modelo, atualizará o manifesto, fará o upload do agente no Microsoft 365 e o testará no Microsoft 365 Copilot. 

Um agente declarativo será implementado em um aplicativo do Microsoft 365. Você criará um pacote de aplicativo contendo:

- app.manifest.json: o arquivo de manifesto do aplicativo descreve como seu aplicativo está configurado, incluindo os recursos.
- declarative-agent.json: o manifesto do agente declarativo descreve como o agente declarativo está configurado.
- color.png e outline.png: um ícone de cor e contorno usado para representar o agente declarativo na interface do usuário do Microsoft 365 Copilot.

### Duração do exercício

- **Tempo estimado para conclusão:** 15 minutos

## Tarefa 1 – Habilitar uploads de aplicativos personalizados no centro de administração do Teams

Para carregar agentes declarativos no Microsoft 365 por meio do Kit de Ferramentas do Teams, você precisa habilitar **Uploads de aplicativos personalizados** no centro de administração do Teams.

1. Navegue até Aplicativos do Teams > Políticas de configuração de aplicativos, no centro de administração do Teams ou vá diretamente para [Políticas de configuração de aplicativos](https://admin.teams.microsoft.com/policies/app-setup).
1. Selecione **Global (padrão em toda a organização)** na lista de políticas de configuração do aplicativo.
1. Ative **Carregar aplicativos personalizados**.
1. Selecione **Salvar**, depois **Confirmar** a sua escolha.

## Tarefa 2 — Baixar o projeto inicial

Comece baixando o projeto de exemplo no GitHub em um navegador da Web:

1. Navegue até o repositório de modelo [https://github.com/microsoft/learn-declarative-agent-vscode](https://github.com/microsoft/learn-declarative-agent-vscode).
    1. Siga as etapas para [baixar o código-fonte do repositório](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) no seu computador.
    1. Extraia o conteúdo do arquivo ZIP baixado na **pasta Documentos**.

O projeto inicial contém um projeto do Kit de Ferramentas do Teams que inclui um agente declarativo.

1. Abra a pasta do projeto  no Visual Studio Code.
1. Na pasta raiz do projeto, abra o arquivo **README.md**. Examine o conteúdo para obter mais informações sobre a estrutura do projeto.

![Captura de tela do Visual Studio Code que mostra o arquivo Leiame do projeto inicial e a estrutura de pastas na exibição do Explorer.](../media/LAB_01/create-complete.png)

## Tarefa 3 — Examinar o manifesto do agente declarativo

Vamos examinar o arquivo de manifesto do agente declarativo:

- Abra o arquivo **appPackage/declarativeAgent.json** e examine o conteúdo:

    ```json
    {
        "$schema": "https://aka.ms/json-schemas/agent/declarative-agent/v1.0/schema.json",
        "version": "v1.0",
        "name": "da-product-support",
        "description": "Declarative agent created with Teams Toolkit",
        "instructions": "$[file('instruction.txt')]"
    }
    ```

O valor da propriedade **instructions** contém uma referência a um arquivo chamado **instruction.txt**. A função **$[file(path)]** é fornecida pelo Kit de Ferramentas do Teams. O conteúdo de **instruction.txt** é incluído no arquivo de manifesto do agente declarativo quando provisionado para o Microsoft 365.

- Na pasta **appPackage**, abra o arquivo **instruction.txt** e revise o conteúdo:

    ```md
    You are a declarative agent and were created with Team Toolkit. You should start every response and answer to the user with "Thanks for using Teams Toolkit to create your declarative agent!\n\n" and then answer the questions and help the user.
    ```

## Tarefa 4 — Atualizar o manifesto do agente declarativo

Vamos atualizar as propriedades **name** e **description** para serem mais relevantes para nosso cenário.

1. Na pasta **appPackage**, abra o arquivo **declarativeAgent.json**.
1. Atualize o valor da propriedade **name** para **Microsoft 365 Knowledge Expert**.
1. Atualize o valor da propriedade **description** para o **Microsoft 365 Knowledge Expert** que pode responder a qualquer pergunta que você tenha sobre o Microsoft 365.
1. Salvar suas alterações

O arquivo atualizado deve ter o seguinte conteúdo:

```json
{
    "$schema": "https://aka.ms/json-schemas/agent/declarative-agent/v1.0/schema.json",
    "version": "v1.0",
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]"
}
```

## Tarefa 5 — Carregar o agente declarativo no Microsoft 365

Em seguida, carregue o agente declarativo no locatário do Microsoft 365.

No Visual Studio Code:

1. Na **Barra de Atividades**, abra a extensão **Kit de Ferramentas do Teams**.

    ![Captura de tela do Visual Studio Code. O ícone do Kit de Ferramentas do Teams é realçado na Barra de Atividades.](../media/LAB_01/teams-toolkit-open.png)

1. Na seção **Ciclo de vida**, selecione **Provisionar**.

    ![Captura de tela do Visual Studio Code mostrando a exibição do Kit de Ferramentas do Teams. A função "Provisionar" é destacada na seção Ciclo de vida.](../media/LAB_01/provision.png)

1. No prompt, selecione **Entrar** e siga os prompts para entrar no locatário do Microsoft 365 usando o Kit de Ferramentas do Teams. O processo de provisionamento é iniciado automaticamente depois que você entra.

    ![Captura de tela de um prompt do Visual Studio Code solicitando que o usuário entre no Microsoft 365. O botão Entrar está realçado.](../media/LAB_01/provision-sign-in.png)

    ![Captura de tela do Visual Studio Code mostrando o processo de provisionamento em andamento. A mensagem de provisionamento em andamento é realçada.](../media/LAB_01/provision-in-progress.png)

1. Aguarde o upload ser concluído antes de continuar.

    ![Captura de tela do Visual Studio Code mostrando uma notificação do sistema confirmando que o processo de provisionamento foi concluído. A notificação do sistema é realçada.](../media/LAB_01/provision-complete.png)

Em seguida, revise a saída do processo de provisionamento.

- Na pasta **appPackage/build**, abra o arquivo **declarativeAgent.dev.json**.

Observe que o valor da propriedade **instructions** contém o conteúdo do arquivo **instruction.txt**. O arquivo **declarativeAgent.dev.json** está incluso no arquivo **appPackage.dev.zip** junto com os arquivos **manifest.dev.json**, **color.png** e **outline.png**. O arquivo **appPackage.dev.zip** é carregado no Microsoft 365.

> [!IMPORTANT]
> Depois de fazer logon na sua conta do Microsoft 365, você pode ver os seguintes avisos ou mensagens de erro no Visual Studio Code. Se você acabou de habilitar uploads de aplicativos personalizados no Microsoft Teams, pode levar algum tempo para que a configuração entre em vigor.  Aguarde alguns minutos e tente novamente ou saia e faça login novamente com sua conta do Microsoft 365. A segunda mensagem sobre o acesso ao Microsoft 365 Copilot é esperada, pois o locatário não tem uma licença completa do Copilot.
> 
> ![Captura de tela do Visual Studio Code.](../media/LAB_01/ttk-login-errors.png)

## Tarefa 6 — Testar o agente declarativo no Microsoft 365 Copilot Chat

Em seguida, vamos executar o agente declarativo no Microsoft 365 Copilot Chat e validar sua funcionalidade.

1. Na **Barra de Atividades**, abra a extensão **Kit de Ferramentas do Teams**.

    ![Captura de tela do Visual Studio Code. O ícone do Kit de Ferramentas do Teams é realçado na Barra de Atividades.](../media/LAB_01/teams-toolkit-open.png)

1. Na seção **Ciclo de vida**, selecione **Publicar**. Aguarde a conclusão das ações.

1. Abra o Microsoft Edge e navegue até o Microsoft 365 Copilot Chat em [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat).

1. No **Microsoft 365 Copilot Chat**, clique no ícone no canto superior direito para expandir o painel lateral do Copilot. Observe que o painel exibe chats recentes e agentes disponíveis.

1. No painel lateral, selecione **Microsoft 365 Knowledge Expert** para entrar na experiência imersiva e conversar diretamente com o agente.

1. Pergunte ao agente **O que você pode fazer?** e envie o prompt.

    ![Captura de tela do Microsoft Edge mostrando o Microsoft 365 Copilot. O ícone para abrir o painel lateral e o agente de Suporte ao produto no painel são destacados.](../media/LAB_01/test-immersive-side-panel.png)

Prossiga para o próximo exercício.