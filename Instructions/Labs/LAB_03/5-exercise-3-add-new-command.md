---
lab:
  title: Exercício 3 - Adicionar um novo comando
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Exercício 3 - Adicionar um novo comando

Neste exercício, você aprimorará o plug-in de extensão de mensagem do Teams e do Copilot adicionando um novo comando. Embora a extensão de mensagem atual forneça informações sobre produtos no banco de dados do inventário da Northwind com eficácia, ela não fornece informações relacionadas aos clientes da Northwind. Você apresentará um novo comando associado a uma chamada de API que recupera produtos ordenados por um nome de cliente especificado pelo usuário. Este exercício pressupõe que você tenha feito pelo menos os exercícios 1, 2 e 3. Não tem problema pular o Exercício 4 caso você não tenha uma licença do Copilot para Microsoft 365.

Para isso, passaremos pelas seguintes tarefas:

1. **Estender a extensão de mensagem/ interface do usuário do plug-in** modificando o manifesto do aplicativo Teams. Isso inclui introduzir um novo comando: **"companySearch"**. Observe que a interface do usuário da extensão de mensagem é um cartão adaptável, enquanto para o Copilot é a entrada e saída de texto no chat do Copilot.

1. **Cria um manipulador para o comando "companySearch"**. Ele analisará a cadeia de caracteres de consulta passada do código de roteamento da mensagem, validará a entrada e chamará a API de pesquisa de produtos por empresa. Essa tarefa também preencherá um cartão adaptável com a lista de produtos retornada, que será exibida na mensagem ou na interface do usuário do chat do Copilot.

1. Atualize o código de **roteamento** do comando para rotear o novo comando para o manipulador criado na tarefa anterior. Você fará isso estendendo o método chamado pelo Bot Framework quando os usuários consultarem o banco de dados da Northwind (**handleTeamsMessagingExtensionQuery**). 

1. **Implementar a Pesquisa de Produtos por Empresa** que retorna uma lista de produtos encomendados por essa empresa.

1. **Executar o aplicativo** e pesquisar produtos que foram comprados por uma empresa especificada.

## Tarefa 1 – Estender a extensão de mensagem / interface do usuário do plugin 

1. No Visual Studio Code, na **pasta de trabalho**, abra **manifest.json** e adicione o seguinte JSON imediatamente após o comando `discountSearch`. Com essas informações adicionais, você está adicionando à matriz `commands` que define a lista de comandos aceitos pelo plug-in.

   ```json
   {
       "id": "companySearch",
       "context": [
           "compose",
           "commandBox"
       ],
       "description": "Given a company name, search for products ordered by that company",
       "title": "Customer",
       "type": "query",
       "parameters": [
           {
               "name": "companyName",
               "title": "Company name",
               "description": "The company name to find products ordered by that company",
               "inputType": "text"
           }
       ]
   }
   ```

> [!NOTE] 
> A **ID** é a conexão entre a IU e o código. Esse valor é definido como **COMMAND_ID** nos arquivos **discount\product\SearchCommand.ts** . Veja como cada um desses arquivos tem um **COMMAND_ID** exclusivo que corresponde ao valor de **id**.

## Tarefa 2 – Criar um manipulador para o comando "companySearch"

Neste exercício, copiaremos parte do código existente para criar novos manipuladores para nossos comandos. 

1. No Visual Studio Code, no seu **diretório de trabalho**, navegue até **.\src\messageExtensions**, copie '**productSearchCommand.ts**' e cole na mesma pasta para criar uma cópia. Renomeie este arquivo **customerSearchCommand.ts**.

1. Mude a linha 7 para:

    ```typescript
    import { searchProductsByCustomer } from "../northwindDB/products";
    ```

1. Mude a linha 10 para:

   ```javascript
   const COMMAND_ID = "companySearch";
   ```



1. Substitua o conteúdo de **handleTeamsMessagingExtensionQuery** por:

   ```javascript
    {
       let companyName;
   
       // Validate the incoming query, making sure it's the 'companySearch' command
       // The value of the 'companyName' parameter is the company name to search for
       if (query.parameters.length === 1 && query.parameters[0]?.name === "companyName") {
           [companyName] = (query.parameters[0]?.value.split(','));
       } else { 
           companyName = cleanupParam(query.parameters.find((element) => element.name === "companyName")?.value);
       }
       console.log(`🍽️ Query #${++queryCount}:\ncompanyName=${companyName}`);    
   
       const products = await searchProductsByCustomer(companyName);
   
       console.log(`Found ${products.length} products in the Northwind database`)
       const attachments = [];
       products.forEach((product) => {
           const preview = CardFactory.heroCard(product.ProductName,
               `Customer: ${companyName}`, [product.ImageUrl]);
   
           const resultCard = cardHandler.getEditCard(product);
           const attachment = { ...resultCard, preview };
           attachments.push(attachment);
       });
       return {
           composeExtension: {
               type: "result",
               attachmentLayout: "list",
               attachments: attachments,
           },
       };
   }
   ```

> [!NOTE]
> Você implementará `searchProductsByCustomer` na Tarefa 4.

## Tarefa 3 – Atualizar o roteamento de comandos

Nesta tarefa, você roteará o comando `companySearch` para o manipulador implementado na tarefa anterior.

1. Abra **searchApp.ts** e adicione o seguinte. Abaixo desta linha:

   ```javascript
   import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
   ```

1. Adicione esta linha:

   ```javascript
   import customerSearchCommand from "./messageExtensions/customerSearchCommand";
   ```

1. Abaixo desta instrução:

   ```javascript
         case discountedSearchCommand.COMMAND_ID: {
           return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

1. Adicione esta instrução:

   ```javascript
         case customerSearchCommand.COMMAND_ID: {
           return customerSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

> [!NOTE]
> A operação baseada na interface do usuário do plug-in é explicitamente chamada. No entanto, quando invocado pelo Microsoft 365 Copilot, o comando é acionado pelo orquestrador do Copilot.

## Tarefa 4 – Implementar a Pesquisa de Produtos por Empresa

Nesta tarefa, você implementará uma pesquisa de produtos **por nome da empresa** e retornará uma lista dos produtos solicitados pela empresa. A saída da tabela da consulta será assim:

| Tabela         | Localizar        | Pesquisar por    |
| ------------- | ----------- | ------------- |
| Customer      | ID do Cliente | Nome do cliente |
| Ordens        | ID da Ordem    | ID do Cliente   |
| OrderDetail   | Product     | ID da Ordem      |

A consulta funciona desta forma: 

1. Use a tabela **Cliente** para localizar a **ID do cliente** com o **Nome do cliente**. 

1. Consulte a tabela **Pedidos** com a **ID do cliente** para recuperar as **IDs do pedido** associadas. 

1. Para cada **ID do pedido**, localize os produtos associados na tabela **OrderDetail**. 

1. Por fim, retorne uma lista de produtos ordenados pelo nome da empresa especificada.

Agora, vamos modificar o arquivo **products.ts** para adicionar a nova consulta de pesquisa.

1. Abra **.\src\northwindDB\products.ts**

1. Atualize a instrução `import` na linha 1 para incluir OrderDetail, Pedido e Cliente. Ele deve ter a seguinte aparência:

   ```javascript
   import {
       TABLE_NAME, Product, ProductEx, Supplier, Category, OrderDetail,
       Order, Customer
   } from './model';
   ```

1. Adicione a nova função `searchProductsByCustomer()`

   Abaixo desta linha:

   ```javascript
   import { getInventoryStatus } from '../adaptiveCards/utils';
   ```

   Adicione a função :

   ```javascript
   export async function searchProductsByCustomer(companyName: string): Promise<ProductEx[]> {

       let result = await getAllProductsEx();
   
       let customers = await loadReferenceData<Customer>(TABLE_NAME.CUSTOMER);
       let customerId="";
       for (const c in customers) {
           if (customers[c].CompanyName.toLowerCase().includes(companyName.toLowerCase())) {
               customerId = customers[c].CustomerID;
               break;
           }
       }
       
       if (customerId === "") 
           return [];

       let orders = await loadReferenceData<Order>(TABLE_NAME.ORDER);
       let orderdetails = await loadReferenceData<OrderDetail>(TABLE_NAME.ORDER_DETAIL);
       // build an array orders by customer id
       let customerOrders = [];
       for (const o in orders) {
           if (customerId === orders[o].CustomerID) {
               customerOrders.push(orders[o]);
           }
       }
       
       let customerOrdersDetails = [];
       // build an array order details customerOrders array
       for (const od in orderdetails) {
           for (const co in customerOrders) {
               if (customerOrders[co].OrderID === orderdetails[od].OrderID) {
                   customerOrdersDetails.push(orderdetails[od]);
               }
           }
       }

       // Filter products by the ProductID in the customerOrdersDetails array
       result = result.filter(product => 
           customerOrdersDetails.some(order => order.ProductID === product.ProductID)
       );

       return result;
   }
   ```

## Tarefa 5 – Executar o aplicativo! Buscar produto pelo nome da empresa

Agora você já pode testar o exemplo como um plug-in para o Copilot para Microsoft 365.

1. Remova o aplicativo **Northwest Inventory** no Teams. Essa tarefa é necessária, pois você está atualizando o manifesto. As atualizações de manifesto requerem que o aplicativo seja reinstalado. A maneira mais limpa de fazer isso é primeiro removê-lo do Teams.

    1. Na barra lateral do Teams, selecione os três pontos (...) 1️⃣. Você verá **Northwind Inventory** 2️⃣ na lista de aplicativos.

    1. Clique com o botão direito do mouse no ícone **Northwest Inventory** e clique em desinstalar 3️⃣.

        ![Captura de tela de como desinstalar o Northwind Inventory.](../media/3-01-uninstall-app.png)

1. Inicie o aplicativo no Visual Studio Code usando o perfil **Depurar no Teams (Edge)** pressionando **F5**.

1. No Teams, selecione **Chat** e, em seguida, **Copilot**. O Copilot é a melhor opção.

1. Selecione o **ícone de Plugin** e depois **Northwind Inventory** para ativar o plugin.

1. Digite o prompt: 

   ```console
   What are the products ordered by 'Consolidated Holdings' in Northwind Inventory?
   ```

   A saída do Terminal mostra que o Copilot entendeu a consulta e executou o comando `companySearch`, passando o nome da empresa extraído pelo Copilot.

   ![Captura de tela do Copilot entendendo a consulta e executando o comando "companySearch".](../media/3-08-terminal-query-output.png)

   Aqui está a saída no Copilot:

    ![Captura de tela do Copilot produzindo os resultados a partir do comando.](../media/3-07-response-customer-search.png)

Você também pode tentar estes prompts:

```console
What are the products ordered by 'Consolidated Holdings' in Northwind Inventory? Please list the product name, price and supplier in a table.
```

Claro, você também pode testar esse novo comando usando o exemplo como uma extensão de mensagem, como fizemos no exercício anterior. 

1. Na barra lateral do Teams, vá para a seção **Chat** e escolha um chat ou inicie um novo chat com um colega.

1. Selecione o sinal **+** para acessar o menu Aplicativos.

1. Escolha o aplicativo **Northwind Inventory**.

1. Observe como agora você pode ver uma nova guia chamada **Cliente**.

1. Pesquise por **Consolidated Holdings** e veja os produtos pedidos por esta empresa. Eles serão os mesmos que o Copilot retornou na tarefa anterior.

    ![Captura de tela do novo comando usado como uma extensão de mensagem.](../media/3-08-customer-message-extension.png)

## Verifique seu trabalho

Depois de concluir o exercício, você terá um novo comando para pesquisar pedidos por empresa no aplicativo Northwind Inventory. Você também poderá usar o plug-in com o Copilot e como uma extensão de mensagem em outros aplicativos. 

No próximo exercício, você explorará o código-fonte do plug-in e os cartões adaptáveis para aprender mais sobre como os aplicativos são criados e como personalizá-los ainda mais.

[Continue no próximo exercício...](./6-exercise-4-explore-plugin-source-code.md)