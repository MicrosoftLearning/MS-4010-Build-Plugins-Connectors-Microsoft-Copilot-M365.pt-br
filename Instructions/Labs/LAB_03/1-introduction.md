---
lab:
  title: Introdução
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Introdução

Neste projeto, você aprenderá a usar as Extensões de Mensagem do Teams como plug-ins no Microsoft Copilot para Microsoft 365. O projeto é baseado no exemplo "Inventário da Northwind" contido neste mesmo [repositório GitHub](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/tree/main/samples/msgext-northwind-inventory-ts). Usando o respeitável [Banco de dados Northwind](https://learn.microsoft.com/dotnet/framework/data/adonet/sql/linq/downloading-sample-databases), você terá muitos dados corporativos simulados para trabalhar.

A Northwind opera um negócio de comércio eletrônico de alimentos especiais em Spokane, Washington. Neste laboratório, você trabalhará com o aplicativo do Inventário da Northwind, que fornece acesso ao inventário de produtos e informações financeiras.

Este exercício deve levar aproximadamente **60** minutos para ser concluído.

## Antes de começar

- Para [**se preparar**](./2-prepare-development-environment.md), comece configurando seu ambiente de desenvolvimento e colocando o aplicativo para executar.

- No [**Exercício 1**](./3-exercise-1-run-message-extension.md), você executará o mesmo aplicativo como uma [extensão de mensagem](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions) no Microsoft Teams e no Outlook.

- No [**Exercício 2**](./4-exercise-2-run-copilot-plugin.md), você executará o aplicativo como um plug-in para o Copilot for Microsoft 365. Você testará vários prompts e observará como o plug-in é invocado usando diferentes parâmetros. Enquanto conversa com o Copilot, você pode assistir ao console do desenvolvedor para ver as consultas sendo feitas.

- No [**Exercício 3**](./5-exercise-3-add-new-command.md), você aprenderá a adicionar um novo comando ao aplicativo, para que possa expandir os recursos do plug-in e executar mais tarefas.

  ![Captura de tela de um cartão adaptável exibindo um produto.](../media/1-00-product-card-only.png)

- Por fim, no [**Exercício 4**](./6-exercise-4-explore-plugin-source-code.md), você fará um tour pelo código para ver como ele funciona com mais profundidade. Se você ainda não tiver o Copilot, todo o restante ainda funcionará como uma extensão de mensagem para o Microsoft 365.

Quando estiver tudo pronto para começar, [continue no próximo exercício...](./2-prepare-development-environment.md)