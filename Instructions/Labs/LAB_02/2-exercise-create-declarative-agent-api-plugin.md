---
lab:
  title: Exercício 1 — Baixar arquivos de projeto e examinar
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Exercício 1 — Baixar arquivos de projeto e examinar

Estender um agente declarativo com ações permite que ele recupere e atualize dados armazenados em sistemas externos em tempo real. Usando plug-ins de API, você pode se conectar a sistemas externos por meio de suas APIs para recuperar e atualizar informações.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Baixar o projeto inicial

Comece baixando o projeto de exemplo. Em um navegador da Web:

1. Navegue até [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript).
    1. Siga as etapas para [baixar o código-fonte do repositório](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) no seu computador.
    1. Extraia o conteúdo do arquivo ZIP baixado na **pasta Documentos**.
    1. Abra a pasta  no Visual Studio Code.

O projeto de exemplo é um projeto do Kit de Ferramentas do Teams que inclui um agente declarativo e uma API anônima em execução no Azure Functions. O agente declarativo é idêntico a um agente declarativo recém-criado usando o Kit de Ferramentas do Teams. A API pertence a um restaurante italiano fictício e permite que você navegue pelo menu do dia e faça pedidos.

## Tarefa 2 — Examinar a definição de API

Primeiro, observe a definição de API da API do restaurante italiano.

No Visual Studio Code:

1. Na exibição **Explorer**, abra o arquivo **appPackage/apiSpecificationFile/ristorante.yml**. O arquivo é uma especificação OpenAPI que descreve a API do restaurante italiano.
1. Localize a propriedade **servers.url**

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    Observe que ela está apontando para uma URL local que corresponde à URL padrão ao executar o Azure Functions localmente.

1. Localize a propriedade **paths**, que contém duas operações: **/dishes** para recuperar o menu de hoje e **/orders** para fazer um pedido.

    > [!IMPORTANT]
    > Observe que cada operação contém a propriedade **operationId** que identifica exclusivamente a operação na especificação da API. O Copilot exige que cada operação tenha uma ID exclusiva para saber qual API deve chamar para prompts de usuário específicos.

## Tarefa 3 — Examinar a implementação da API

Em seguida, examine a API de exemplo que você usará neste exercício.

No Visual Studio Code:

1. Na exibição **Explorer**, abra o arquivo **src/data.json**. O arquivo contém um item de menu fictício do restaurante italiano. Cada prato é composta por:

    - nome,
    - descrição,
    - link para uma imagem,
    - preço,
    - em que prato é servido,
    - tipo (prato ou bebida),
    - opcionalmente, uma lista de alérgenos

    Neste exercício, as APIs usam esse arquivo como fonte de dados.
1. Em seguida, expanda a pasta **src/functions**. Observe dois arquivos chamados **dishes.ts** e **placeOrder.ts**. Esses arquivos contêm a implementação das duas operações definidas na especificação da API.
1. Abra o arquivo **src/functions/dishes.ts**. Reserve um momento para revisar como a API está trabalhando. Ela começa carregando os dados de amostra do arquivo **src/functions/data.json**.

    ```typescript
    import data from "../data.json";
    ```

    Em seguida, ela procura nos diferentes parâmetros de cadeia de caracteres de consulta possíveis filtros que o cliente que chama a API pode passar.

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    Com base nos filtros especificados na solicitação, a API filtra o conjunto de dados e retorna uma resposta.

1. Em seguida, examine a API para fazer pedidos definidos no arquivo **src/functions/placeOrder.ts**. A API começa referenciando os dados de exemplo. Em seguida, ela define a forma do pedido que o cliente envia no corpo da solicitação.

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    Quando a API processa a solicitação, ela primeiro verifica se a solicitação contém um corpo e se tem a forma correta. Caso contrário, ela rejeitará a solicitação com um erro 400 Bad Request.

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    Em seguida, a API resolve a solicitação em pratos no menu e calcula o preço total.

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > Observe como a API espera que o cliente especifique o prato por uma parte de seu nome em vez de sua ID. Isso ocorre propositalmente porque grandes modelos de linguagem funcionam melhor com palavras do que com números. Além disso, antes de chamar a API para fazer o pedido, o Copilot tem o nome do prato prontamente disponível como parte do prompt do usuário. Se o Copilot tivesse que referenciar um prato por sua ID, primeiro precisaria recuperar, o que requer solicitações de API adicionais e o que o Copilot não pode fazer agora.

    Quando a API está pronta, ela retorna uma resposta com um preço total, uma ID de pedido inventada e um status.

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

