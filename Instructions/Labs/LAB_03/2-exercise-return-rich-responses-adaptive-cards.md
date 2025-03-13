---
lab:
  title: Exercício 1 — Baixar o projeto e criar um cartão adaptável
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Exercício 1 — Baixar o projeto e criar um cartão adaptável

Vamos começar criando modelos de Cartão Adaptável para que o agente mostre os dados em suas respostas. Para criar o modelo do cartão adaptável, use as extensões do Pré-visualizador de Cartão Adaptável do Visual Studio Code para visualizar facilmente seu trabalho diretamente no Visual Studio Code. O uso da extensão nos permite criar um modelo de cartão adaptável, com referências a dados. No runtime, o agente preenche o espaço reservado com dados recuperados da API.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Baixar o projeto inicial

Comece baixando o projeto de exemplo. Em um navegador da Web:

1. Navegue até [https://github.com/microsoft/learn-declarative-agent-api-plugin-adaptive-cards-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-adaptive-cards-typescript).
  1. Siga as etapas para [baixar o código-fonte do repositório](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) no seu computador.
  1. Extraia o conteúdo do arquivo ZIP baixado na **pasta Documentos**.
  1. Abra a pasta  no Visual Studio Code.

O projeto de exemplo é um projeto do Kit de Ferramentas do Teams que inclui um agente declarativo com uma ação criada com um plug-in de API. O plug-in de API se conecta a uma API anônima em execução no Azure Functions também incluída no projeto. A API pertence a um restaurante italiano fictício e permite que você navegue pelo menu do dia e faça pedidos.

## Tarefa 2 — Criar um cartão adaptável para um prato

Primeiro, crie um Cartão Adaptável que mostre informações sobre um único prato.

No Visual Studio Code:

1. Na visualização **Explorador**, crie uma nova pasta chamada **Cartões**.
1. Na pasta **Cartões**, crie um novo arquivo chamado **dish.json**. Cole o seguinte conteúdo que representa um Cartão Adaptável vazio:

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": []
  }
  ```

1. Antes de continuar, na guia **Extensões** da barra de atividades, pesquise e instale a extensão **Visualizador do Cartão Adaptável** e crie um arquivo de dados para o Cartão Adaptável:
  1. Abra a paleta de comandos pressionando <kbd>CTRL</kbd>+<kbd>P</kbd> no teclado. Digite `>Adaptive` para localizar comandos relacionados a trabalhar com Cartões Adaptáveis.

    ![Captura de tela do Visual Studio Code mostrando comandos relacionados a trabalhar com Cartões Adaptáveis.](../media/LAB_03/3-visual-studio-code-adaptive-card-commands.png)

  1. Na lista, escolha **Cartão Adaptável: Novo Arquivo de Dados**. O Visual Studio Code criará um arquivo chamado **dish.data.json**.
  1. Substitua o conteúdo por um dado que represente um prato:

  ```json
  {
    "id": 4,
    "name": "Caprese Salad",
    "description": "Juicy vine-ripened tomatoes, fresh mozzarella, and fragrant basil leaves, drizzled with extra virgin olive oil and a touch of balsamic.",
    "image_url": "https://raw.githubusercontent.com/pnp/copilot-pro-dev-samples/main/samples/da-ristorante-api/assets/caprese_salad.jpeg",
    "price": 10.5,
    "allergens": [
    "dairy"
    ],
    "course": "lunch",
    "type": "dish"
  }
  ```

  1. Salvar suas alterações
1. Volte para o arquivo **dish.json**.
1. Na lente, selecione **Visualizar Cartão Adaptável**.

  ![Captura de tela do Visual Studio Code mostrando a visualização do Cartão Adaptável.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview.png)

  O Visual Studio Code abrirá uma exibição do cartão ao lado. À medida que você edita o cartão, as alterações ficam imediatamente visíveis na lateral.

1. Na matriz **corpo**, adicione um elemento **Contêiner** com uma referência à URL da imagem armazenada na propriedade **image_url**.

  ```json
  {
    "type": "Container",
    "items": [
    {
      "type": "Image",
      "url": "${image_url}",
      "size": "large"
    }
    ]
  }
  ```

  Observe como a visualização do cartão é atualizada automaticamente para mostrar seu cartão:

  ![Captura de tela do Visual Studio Code mostrando a prévia do Cartão Adaptável com uma imagem.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-image.png)

1. Adicione referências a outras propriedades do prato. O cartão completo é assim:

  ```json
  {
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

  ![Captura de tela do Visual Studio Code mostrando a prévia de um Cartão Adaptável de um prato.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-image-properties.png)

  Observe que, para exibir alérgenos, você usa uma função para unir os alérgenos em uma cadeia de caracteres. Se um prato não tiver alérgenos, você exibirá **nenhum**. Para garantir que os preços sejam formatados corretamente, você usa a função **formatNumber**, que nos permite especificar o número de casas decimais a serem exibidas no cartão.

## Tarefa 3 — Criar um cartão adaptável para o resumo do pedido

A API de exemplo permite que os usuários naveguem no menu e façam um pedido. Vamos criar um cartão adaptável que mostre o resumo do pedido.

No Visual Studio Code:

1. Na pasta **Cartões**, crie um novo arquivo chamado **order.json**. Cole o seguinte conteúdo que representa um Cartão Adaptável vazio:

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": []
  }
  ```

1. Crie um arquivo de dados para o Cartão Adaptável:
  1. Abra a paleta de comandos pressionando <kbd>CTRL</kbd>+<kbd>P</kbd> (<kbd>CMD</kbd>+<kbd>P</kbd> no macOS) no teclado. Digite `>Adaptive` para localizar comandos relacionados a trabalhar com Cartões Adaptáveis.

    ![Captura de tela do Visual Studio Code mostrando comandos relacionados a trabalhar com Cartões Adaptáveis.](../media/LAB_03/3-visual-studio-code-adaptive-card-commands.png)

  1. Na lista, escolha **Cartão Adaptável: Novo Arquivo de Dados**. O Visual Studio Code criará um novo arquivo chamado **order.data.json**.
  1. Substitua o conteúdo por um dado que represente o resumo do pedido:

    ```json
    {
      "order_id": 6210,
      "status": "confirmed",
      "total_price": 25.48
    }
    ```

  1. Salvar suas alterações
1. Volte para o arquivo **order.json**.
1. Na lente, selecione **Visualizar Cartão Adaptável**.
1. Depois, substitua o conteúdo do arquivo **order.json** pelo código:

  ```json
  {
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

  Assim como na seção anterior, você mapeará cada elemento no cartão para uma propriedade de dados.

  ![Captura de tela do Visual Studio Code mostrando a prévia do Cartão Adaptável de um pedido.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-order.png)

  > [!IMPORTANT]
  > Observe o espaço à direita após **${order_id}**. Isso é intencional, devido a um problema conhecido com números de renderização de Cartões Adaptáveis. Para testá-lo, remova o espaço e veja se o número desaparece da prévia.
  >
  > ![Captura de tela do Visual Studio Code mostrando uma prévia de um Cartão Adaptável sem o número do pedido.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-no-number.png)

  Restaure o espaço à direita para que o cartão seja exibido corretamente e salve as alterações.