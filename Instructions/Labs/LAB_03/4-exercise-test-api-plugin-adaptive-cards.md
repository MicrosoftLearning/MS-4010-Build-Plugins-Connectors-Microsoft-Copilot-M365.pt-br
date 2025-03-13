---
lab:
  title: Exercício 3 — Testar o agente declarativo com o plug-in de API no Microsoft 365 Copilot
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Exercício 3 — Testar o agente declarativo com o plug-in de API no Microsoft 365 Copilot

A etapa final é testar o agente declarativo com o plug-in de API no Microsoft 365 Copilot.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Provisionar e iniciar a depuração

No Visual Studio Code:

1. Na **Barra de Atividades**, escolha **Kit de Ferramentas do Teams**.
1. Na seção **Contas**, verifique se você se conectou ao locatário do Microsoft 365 com o Microsoft 365 Copilot.

    ![Captura de tela da seção contas do Kit de Ferramentas do Teams no Visual Studio Code.](../media/LAB_03/3-teams-toolkit-accounts.png)

1. Na **Barra de Atividades**, escolha **Executar e Depurar**.
1. Selecione a configuração **Depurar no Copilot** e inicie a depuração usando o botão **Iniciar Depuração**.  

    ![Captura de tela da configuração Depurar no Copilot no Visual Studio Code.](../media/LAB_03/3-visual-studio-code-start-debugging.png)

1. O Visual Studio Code compila e implanta seu projeto em seu locatário do Microsoft 365 e abre uma nova janela do navegador da Web.

## Tarefa 2 — Testar e analisar os resultados

No navegador da Web:

1. Quando solicitado, entre com a conta que pertence ao locatário do Microsoft 365 com o Microsoft 365 Copilot.
1. Na barra lateral, selecione **Il Ristorante**.

    ![Captura de tela da interface do Microsoft 365 Copilot com o agente Il Ristorante selecionado.](../media/LAB_03/3-copilot-select-agent.png)

1. Escolha o início de conversa **O que vamos almoçar hoje?** e envie a solicitação.

    ![Captura de tela da interface do Microsoft 365 Copilot com o prompt de almoço.](../media/LAB_03/3-copilot-lunch-prompt.png)

1. Quando solicitado, examine os dados que o agente envia para a API e confirme usando o botão **Permitir uma vez**.

    ![Captura de tela da interface do Microsoft 365 Copilot com a confirmação do almoço.](../media/LAB_03/3-copilot-lunch-confirm.png)

1. Aguarde a resposta do agente. Observe que o pop-up em uma citação agora inclui seu Cartão Adaptável personalizado com informações adicionais da API.

    ![Captura de tela da interface do Microsoft 365 Copilot com a resposta do almoço.](../media/LAB_03/3-copilot-lunch-response.png)

1. Faça um pedido digitando na caixa de texto do prompt: **1x espaguete, 1x chá gelado** e envie o prompt.
1. Examine os dados que o agente envia para a API e continue usando o botão **Confirmar**.

    ![Captura de tela da interface do Microsoft 365 Copilot com a confirmação do pedido.](../media/LAB_03/3-copilot-order-confirm.png)

1. Aguarde até que o agente faça o pedido e retorne o resumo do pedido. Observe que, como a API retorna um único item, o agente o renderiza usando um Cartão Adaptável e inclui o cartão diretamente na resposta.

    ![Captura de tela da interface do Microsoft 365 Copilot com a resposta do pedido.](../media/LAB_03/3-copilot-order-response.png)

1. Retorne ao Visual Studio Code e interrompa a depuração.
1. Mude para a guia **Terminal** e feche todos os terminais ativos.

    ![Captura de tela da guia do terminal do Visual Studio Code com a opção de fechar todos os terminais.](../media/LAB_03/3-visual-studio-code-close-terminal.png)
