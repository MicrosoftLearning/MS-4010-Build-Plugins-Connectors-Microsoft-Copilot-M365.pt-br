---
lab:
  title: Introdução
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Introdução

Os agentes do Microsoft 365 Copilot permitem que você crie assistentes da plataforma IA otimizados para cenários específicos. Usando instruções, você define o contexto para o copiloto e configura o tom de voz ou como ele deve responder. Ao configurar o conhecimento do agente, você concede a ele acesso a dados externos que não fazem parte do LLM (Grande Modelo de Linguagem), para que ele possa responder com mais precisão. 

## Cenário de exemplo

Suponha que você trabalhe em um departamento de TI de uma grande organização. Sua organização padroniza a TI por meio de diferentes políticas que armazena em um sistema especializado. Você e seus colegas do departamento de TI recebem regularmente perguntas cujas respostas são previstas pelas políticas. Procurar por respostas no sistema de gerenciamento de políticas é demorado. Você gostaria de fornecer à sua organização um assistente com tecnologia de IA capaz de responder as perguntas de seus colegas usando informações embasadas das políticas.

## Objetivos do aprendizado

Ao final deste módulo, você poderá criar agentes declarativos para o Microsoft 365 Copilot. Você entenderá como configurar suas instruções a fim de otimizá-las para um cenário específico. Você também saberá como integrá-los aos conectores do Microsoft Graph, para dar a eles acesso a dados externos, que não fazem parte do LLM do Microsoft 365 Copilot.

## Pré-requisitos

- Conhecimento do que é o Microsoft 365 Copilot e como ele funciona no nível iniciante
- Conhecimento de como criar um agente declarativo do Microsoft 365 Copilot
- Conhecimento de como se criar um conector do Graph
- Locatário do Microsoft 365 no Microsoft 365 Copilot e privilégios de administrador de locatários
- [Visual Studio Code](https://code.visualstudio.com/) com a extensão do [Kit de Ferramentas do Teams](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) instalada
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
