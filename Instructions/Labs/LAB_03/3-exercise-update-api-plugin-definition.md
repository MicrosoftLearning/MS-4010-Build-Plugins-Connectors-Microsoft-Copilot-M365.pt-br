---
lab:
  title: Exercício 2 — Atualizar a definição do plug-in da API
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Exercício 2 — Atualizar a definição do plug-in da API

A próxima etapa é atualizar a definição do plug-in da API com cartões adaptáveis que o Copilot deve usar para exibir dados da API para os usuários.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Adicionar cartão adaptável para exibir um prato

No Visual Studio Code:

1. Abra o arquivo **cards/dish.json** e copie o conteúdo.
1. Abra o arquivo **appPackage/ai-plugin.json**.
1. Para a propriedade **functions.getDishes.capabilities.response_semantics**, adicione uma nova propriedade chamada **static_template** e defina o valor do **corpo** como o conteúdo de **dish.json**.
1. O snippet de código completo é assim:

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "Image",
                "url": "${image_url}",
                "size": "large"
              },
              {
                "type": "TextBlock",
                "text": "${name}",
                "weight": "Bolder"
              },
              {
                "type": "TextBlock",
                "text": "${description}",
                "wrap": true
              },
              {
                "type": "TextBlock",
                "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
                "weight": "Lighter"
              },
              {
                "type": "TextBlock",
                "text": "**Price:** €${formatNumber(price, 2)}",
                "weight": "Lighter",
                "spacing": "None"
              }
            ]
          }
      ]
    }
    ```

1. Salve suas alterações.

## Tarefa 2 — Adicionar modelo de cartão adaptável para exibir o resumo do pedido

No Visual Studio Code:

1. Abra o arquivo **cards/order.json** e copie o conteúdo.
1. Abra o arquivo **appPackage/ai-plugin.json**.
1. Para a propriedade **functions.placeOrder.capabilities.response_semantics**, adicione uma nova propriedade chamada **static_template** e defina o conteúdo como Cartão Adaptável.
1. O arquivo completo fica assim:

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "TextBlock",
            "text": "Order Confirmation 🤌",
            "size": "Large",
            "weight": "Bolder",
            "horizontalAlignment": "Center"
          },
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "Your order has been successfully placed!",
                "weight": "Bolder",
                "spacing": "Small"
              },
              {
                "type": "FactSet",
                "facts": [
                  {
                    "title": "Order ID:",
                    "value": "${order_id} "
                  },
                  {
                    "title": "Status:",
                    "value": "${status}"
                  },
                  {
                    "title": "Total Price:",
                    "value": "€${formatNumber(total_price, 2)}"
                  }
                ]
              }
            ]
          }
        ]
    }
    ```

1. Salve suas alterações.
