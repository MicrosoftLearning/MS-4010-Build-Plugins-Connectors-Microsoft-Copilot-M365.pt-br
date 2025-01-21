---
lab:
  title: Introdução
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Introdução

Os agentes do Microsoft 365 Copilot permitem que você crie assistentes da plataforma IA otimizados para cenários específicos. Usando instruções, você define o contexto para o agente e especifica configurações como tom de voz ou como ele deve responder. Ao configurar as habilidades do agente, você dá a ele a capacidade de interagir com sistemas externos, acionar determinado comportamento sob condições do sistema ou usar lógica de fluxo de trabalho personalizada. Um tipo de habilidade são as ações que permitem que um agente declarativo se comunique com APIs para recuperar e modificar dados.

![Diagrama que mostra a anatomia de um agente declarativo para o Microsoft 365 Copilot.](../media/LAB_02/1-anatomy-declarative-agent.png)

## Cenário de exemplo

Suponha que você trabalhe em uma organização onde você pede comida regularmente em um restaurante local. O restaurante funciona com um cardápio diário que eles publicam na Internet. Você quer ser capaz de ver rapidamente quais pratos estão disponíveis, mas também considerar alérgenos caso esteja com mais alguém. O restaurante expõe o cardápio por meio de uma API. Em vez de criar um aplicativo separado, você quer integrar as informações ao Microsoft 365 Copilot para encontrar facilmente os pratos disponíveis que você pode pedir e descobrir os ingredientes deles. Você quer usar linguagem natural para navegar pelo menu e fazer pedidos.

## O que faremos?

Neste módulo, você criará uma ação para um agente declarativo com um plug-in de API. A ação permite que o agente interaja com um sistema externo usando sua API anônima. Você aprenderá a:

- **Criar**: criar um plug-in de API que se conecta a uma API anônima.
- **Configurar**: configurar o plug-in da API para mostrar os dados da API.
- **Estender**: estender um agente declarativo com uma ação usando um plug-in da API.
- **Provisionar**: carregar o agente declarativo no Microsoft 365 Copilot e validar os resultados.

![Captura de tela de um agente declarativo que responde a um usuário com informações de uma API externa.](../media/LAB_02/1-agent-response-api-plugin.png)

## Duração do laboratório

- **Tempo estimado para conclusão:** 35 minutos

## Objetivos do aprendizado

Ao final deste módulo, você saberá como integrar agentes declarativos com plug-ins de API conectados a APIs anônimas, para permitir que eles interajam com sistemas externos em tempo real.