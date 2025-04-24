---
lab:
  title: Exercício 2 — Criar agente declarativo e integrar o conector do Graph
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Exercício 2 — Criar agente declarativo e integrar o conector do Graph

Neste exercício, você criará um novo agente declarativo do zero e o configurará para usar a conexão externa como sua fonte de fundamento.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Criar um novo agente declarativo

Uma maneira de criar um agente declarativo é usando o Kit de Ferramentas do Teams. O Kit de Ferramentas do Teams oferece um projeto de modelo para criar agentes declarativos, o que oferece um ótimo lugar para se começar a definir as configurações do agente e incluir recursos extras.

Para criar um novo agente declarativo, abra o Visual Studio Code.

No Visual Studio Code:

1. Na **Barra de Atividades** (barra lateral), abra a extensão **Kit de Ferramentas do Teams**.
1. No painel do Kit de Ferramentas do Teams, selecione o botão **Criar um novo aplicativo**.
1. Na caixa de diálogo **Novo projeto**, escolha a opção **Agente**.
1. Na próxima caixa de diálogo, escolha a opção **Agente declarativo**.
1. Não adicione um plug-in selecionando a opção **Sem plug-in**.
1. Selecione uma pasta onde você deseja armazenar o projeto em seu computador.
1. Nomeie o projeto `da-it-policies`.

## Tarefa 2 — Configurar instruções do agente declarativo

O Kit de Ferramentas do Teams cria um novo projeto de agente declarativo. Para definir o escopo do seu cenário, atualize a descrição e as instruções do agente.

No Visual Studio Code:

1. Abra o arquivo **appPackage/declarativeAgent.json**.
1. Atualize o valor da propriedade **name** para `Contoso IT policies`
1. Atualize o valor da propriedade **description** para `Assistant specialized in Contoso IT policies`.
1. Salve suas alterações.
1. O conteúdo do arquivo atualizado fica da seguinte maneira:

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. Abra o arquivo **appPackage/instruction.txt**.
1. Atualize o conteúdo para:

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. Salve suas alterações.

## Tarefa 3 — Integrar o conector do Microsoft Graph a um agente declarativo

Depois de criar um agente declarativo, a próxima etapa é integrá-lo a um conector do Microsoft Graph, para que ele possa acessar dados externos.

No Visual Studio Code:

1. Abra o arquivo **appPackage/declarativeAgent.json**.
1. Após a propriedade **instructions**, adicione uma nova propriedade chamada **capabilities** com o seguinte código:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. Na propriedade **connection_id**, especifique `policieslocal` como ID da conexão externa. `policieslocal` é a ID da conexão externa que o conector do Graph criou nas etapas anteriores.
1. O conteúdo do arquivo atualizado fica da seguinte maneira:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. Salve suas alterações.
