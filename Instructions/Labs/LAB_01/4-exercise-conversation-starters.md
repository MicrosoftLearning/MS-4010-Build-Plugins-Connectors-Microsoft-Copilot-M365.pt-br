---
lab:
  title: Exercício 3 — Adicionar iniciadores de conversa ao seu agente declarativo
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Exercício 3 — Adicionar iniciadores de conversa ao seu agente declarativo

Neste exercício, você atualizará o agente declarativo para incluir iniciadores de conversas que fornecem aos usuários exemplos de prompts para ajudá-los a entender os tipos de perguntas que eles podem fazer.

### Duração do exercício

- **Tempo estimado para conclusão:** 5 minutos

## Tarefa 1 — Adicionar iniciadores de conversa

No Visual Studio Code:

1. Na pasta **appPackage**, abra o arquivo **declarativeAgent.json**.
1. Adicione o seguinte snippet de código ao arquivo :

   ```json
   "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
   ]
   ```

1. Salve suas alterações.

O arquivo **declarativeAgent.json** ficará assim:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
    "version": "v1.0",
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]",
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "https://learn.microsoft.com/microsoft-365/"
                }
            ]
        }
    ],
  "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
  ]
}
```

## Tarefa 2 — testar o agente declarativo no Microsoft 365 Copilot

Em seguida, faça o upload das alterações e inicie uma sessão de depuração.

No Visual Studio Code:

1. Na **Barra de Atividades**, abra a extensão **Kit de Ferramentas do Teams**.
1. Na seção **Ciclo de vida**, selecione **Provisionar**, e então **Publicar**.
1. **Confirme** que deseja enviar uma atualização para o aplicativo.
1. Aguarde a conclusão do upload.

Continuando no navegador da Web:

1. No **Microsoft 365 Copilot**, clique no ícone no canto superior direito para **expandir o painel lateral do Copilot**.
1. Encontre **Suporte ao produto** na lista de agentes e selecione-o para entrar na experiência imersiva para conversar diretamente com o agente. Observe que os iniciadores de conversa que você definiu no manifesto são exibidos na interface do usuário.

![Captura de tela do Microsoft Edge mostrando o agente declarativo Microsoft 365 Knowledge Expert na experiência imersiva com iniciadores de conversa personalizados.](../media/LAB_01/test-conversation-starters.png)

Feche o navegador para interromper a sessão de depuração no Visual Studio Code.