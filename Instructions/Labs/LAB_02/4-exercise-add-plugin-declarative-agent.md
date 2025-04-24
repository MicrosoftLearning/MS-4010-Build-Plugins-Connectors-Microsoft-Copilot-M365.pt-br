---
lab:
  title: Exercício 3 — Conectar a definição do plug-in ao agente declarativo
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Exercício 3 — Conectar a definição do plug-in ao agente declarativo

Depois de concluir a criação da definição do plug-in da API, o próximo passo é registrá-lo com o agente declarativo. Quando os usuários interagem com o agente declarativo, ele compara o prompt do usuário com os plug-ins de API definidos e invoca as funções relevantes.

### Duração do exercício

- **Tempo estimado para conclusão:** 5 minutos

## Tarefa 1 — Conectar a definição do plug-in ao agente declarativo

No Visual Studio Code:

1. Abra o arquivo **appPackage/declarativeAgent.json**.
1. Após a propriedade **instructions**, adicione o seguinte snippet de código:

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    Usando esse snippet, você conectará o agente declarativo ao plug-in da API. Você especificará uma ID exclusiva para o plug-in e instruirá o agente onde encontrar a definição do plug-in.

1. O arquivo completo fica assim:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. Salve suas alterações.

## Tarefa 2 — Atualizar informações e instruções do agente declarativo

O agente declarativo que você está criando neste exercício ajudará os usuários a navegar no menu do restaurante italiano local e fazer pedidos. Para otimizar o agente para esse cenário, atualize seu nome, descrição e instruções.

No Visual Studio Code:

1. Atualize as informações do agente declarativo:
    1. Abra o arquivo **appPackage/declarativeAgent.json**.
    1. Atualize o valor da propriedade **name** para **Il Ristorante**.
    1. Atualize o valor da propriedade **description** para **Peça os mais deliciosos pratos italianos e bebidas no conforto da sua mesa de trabalho.**
    1. Salve as alterações.
1. Atualize as instruções do agente declarativo:
    1. Abra o arquivo **appPackage/instruction.txt**.
    1. Substitua o conteúdo dele por:

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        Observe que, nas instruções, definimos o comportamento geral do agente e instruímos do que ele é capaz. Também incluímos instruções para um comportamento específico em relação à realização de um pedido, incluindo a forma dos dados que a API espera. Incluímos essas informações para garantir que o agente funcione conforme o esperado.

    > [!NOTE]
    > Para preservar a formatação, talvez seja necessário executar várias operações de copiar/colar no Bloco de Notas antes de copiar para o Visual Studio Code.

    1. Salve as alterações.

1. Para ajudar os usuários a entender para que eles podem usar o agente, adicione iniciadores de conversa:
    1. Abra o arquivo **appPackage/declarativeAgent.json**.
    1. Após a propriedade **instructions**, adicione uma nova propriedade chamada **conversation_starters**:

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. O arquivo completo fica assim:

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. Salve suas alterações.

## Parte 3 — Atualizar a URL da API

Antes de testar o agente declarativo, você precisa atualizar o URL da API no arquivo de especificação da API. No momento, a URL está definida como `http://localhost:7071/api`, que é a URL que o Azure Functions usa ao executar localmente. No entanto, como você quer que o Copilot chame a API na nuvem, você precisa expor a API à Internet. O Kit de Ferramentas do Teams expõe automaticamente sua API local pela Internet ao criar um túnel de desenvolvimento. Cada vez que você inicia a depuração do projeto, o Kit de Ferramentas do Teams inicia um novo túnel de desenvolvimento e armazena sua URL na variável **OPENAPI_SERVER_URL**. Você pode ver como o Kit de Ferramentas do Teams inicia o túnel e armazena a URL no arquivo **.vscode/tasks.json**, na tarefa **Iniciar túnel local**:

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

Para usar esse túnel, você precisa atualizar a especificação da API para usar a variável **OPENAPI_SERVER_URL**.

No Visual Studio Code:

1. Abra o arquivo **appPackage/apiSpecificationFile/ristorante.yml**.
1. Altere o valor da propriedade **servers.url** para **${{OPENAPI_SERVER_URL}}/api**.
1. O arquivo alterado ficará assim:

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. Salve suas alterações.

Seu plug-in da API está concluído e integrado a um agente declarativo. Continue testando o agente no Microsoft 365 Copilot.