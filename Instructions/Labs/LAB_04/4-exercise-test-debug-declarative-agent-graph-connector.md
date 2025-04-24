---
lab:
  title: Exercício 3 — Testar e depurar
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Exercício 3 — Testar e depurar

Neste exercício, você testará e implantará seu agente declarativo no Microsoft 365 e o testará usando o Microsoft 365 Copilot Chat.

### Duração do exercício

- **Tempo estimado para conclusão:** 5 minutos

## Tarefa 1 — Testar o agente declarativo no Microsoft 365 Copilot

Para testar seu agente declarativo, implante-o como um aplicativo em seu locatário do Microsoft 365. Depois de abri-lo no Microsoft 365 Copilot, verifique se ele funciona conforme o esperado.

No Visual Studio Code:

1. Na **Barra de Atividades** (barra lateral), abra a extensão **Kit de Ferramentas do Teams**.
1. No painel **Ciclo de vida**, escolha **Provisionar**. O Kit de Ferramentas do Teams empacota o projeto do agente declarativo como um aplicativo e faz seu upload no Microsoft 365.
1. Abra um navegador da Web.

Em um navegador da Web:

1. Navegue até [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat).
1. Entre com sua conta de trabalho de locatário do Microsoft 365.
1. No Microsoft 365 Copilot, no painel lateral, selecione o agente de **Políticas de TI da Contoso** para ativá-lo.
1. Na caixa de texto de bate-papo, pergunte `What's the acceptable use policy at Contoso?`.
1. Aguarde a resposta do agente. Observe como a resposta inclui referências ao conteúdo externo que o conector do Graph ingeriu. A URL em cada referência aponta para a localização no sistema externo onde o conteúdo está armazenado.

    ![Captura de tela do Microsoft 365 Copilot respondendo ao prompt de um usuário.](../media/LAB_04/3-copilot-response.png)