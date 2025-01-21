---
lab:
  title: Exercício 2 — Criar definição de plug-in de API
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Exercício 2 — Criar definição de plug-in de API

A próxima etapa é adicionar a definição do plug-in ao projeto. A definição do plug-in contém as informações a seguir:

- Quais ações o plug-in pode executar.
- Qual é a forma dos dados que ele espera e retorna.
- Como o agente declarativo deve chamar a API subjacente.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Adicionar a estrutura básica de definição de plug-in

No Visual Studio Code:

1. Na pasta **appPackage**, adicione um novo arquivo chamado **ai-plugin.json**.
1. Cole o seguinte conteúdo:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    O arquivo contém uma estrutura básica para um plug-in de API com uma descrição para o humano e o modelo. A propriedade **description_for_model** inclui informações detalhadas sobre o que o plug-in pode fazer para ajudar o agente a entender quando deve considerar invocá-lo.
1. Salve suas alterações.

## Tarefa 2 — Definir funções

Um plug-in de API define uma ou mais funções que são mapeadas para operações de API definidas na especificação da API. Cada função consiste em um nome, uma descrição e uma definição de resposta que instrui o agente sobre como exibir os dados aos usuários.

### Definir uma função para recuperar o menu

Comece definindo uma função para recuperar as informações sobre o menu de hoje.

No Visual Studio Code:

1. Abra o arquivo **appPackage/ai-plugin.json**.
1. Na matriz **funções**, adicione o seguinte snippet:

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    Comece definindo uma função que invoca a operação **getDishes** na especificação da API. Em seguida, forneça uma descrição da função. Essa descrição é importante porque o Copilot a usa para decidir qual função invocar para o prompt de um usuário.

    Na propriedade response_semantics, você especifica como o Copilot deve exibir os dados que recebe da API. Como a API retorna as informações sobre os pratos no menu na propriedade **dishes**, você definirá a propriedade **data_path** como a expressão `$.dishes` JSONPath.

    Em seguida, na seção **propriedades**, você mapeará quais propriedades da resposta da API representam o título, a descrição e a URL. Como neste caso os pratos não têm uma URL, você mapeará apenas o **título** e a **descrição**.

1. O snippet de código completo é assim:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Salve suas alterações.

### Definir uma função para fazer o pedido

Em seguida, defina uma função para fazer o pedido.

No Visual Studio Code:

1. Abra o arquivo **appPackage/ai-plugin.json**.
1. Adicione o seguinte snippet ao final da matriz **funções**:

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    Comece consultando a operação da API com a ID **placeOrder**. Em seguida, forneça uma descrição que o Copilot usará para corresponder essa função ao prompt de um usuário. Em seguida, instrua o Copilot sobre como retornar os dados. A seguir estão os dados que a API retorna após fazer um pedido:

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    Como os dados que você quer mostrar estão localizados diretamente na raiz do objeto de resposta, defina **data_path** como **$**, o que indica o nó superior do objeto JSON. Defina o título para mostrar o número do pedido e o subtítulo o preço.

1. O arquivo completo fica assim:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Salve suas alterações.

## Tarefa 3 — Definir runtimes

Depois de definir as funções que o Copilot invocará, a próxima etapa é instruí-lo sobre como chamá-las. Você fará isso na seção **runtimes** da definição do plug-in.

No Visual Studio Code:

1. Abra o arquivo **appPackage/ai-plugin.json**.
1. Na matriz runtimes, adicione o seguinte código:

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    Comece instruindo o Copilot que você fornecerá informações de OpenAPI sobre a API (**tipo: OpenApi**) a ser chamada e que ela é anônima (**auth.type: None**). Em seguida, na seção **especificação**, especifique o caminho relativo para a especificação da API localizada em seu projeto. Por fim, na propriedade **run_for_functions**, liste todas as funções que pertencem a essa API.

1. O arquivo completo fica assim:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Salve suas alterações.

