---
lab:
  title: Exercício 1 – Integrar um plug-in de API a uma API protegida com uma chave
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Exercício 1 – Integrar um plug-in de API a uma API protegida com uma chave

Os plug-ins de API para Microsoft 365 Copilot permitem que você se integre a APIs protegidas com uma chave. Você mantém a chave de API segura registrando-a no cofre do Teams. Em runtime, o Microsoft 365 Copilot executa o seu plug-in, obtém a chave de API do cofre e a usa para chamar a API. Seguindo esse processo, a chave de API permanece segura e nunca é exposta ao cliente.

Neste exercício, você criará um novo agente declarativo com um plug-in de API que é autenticado com uma chave de API gerada por você.

### Duração do exercício

- **Tempo estimado para conclusão:** 10 minutos

## Tarefa 1 — Criar um novo projeto

Comece criando um novo plug-in de API para o Microsoft 365 Copilot. Abra o Visual Studio Code.

No Visual Studio Code:

1. Na **Barra de Atividades** (barra lateral), abra a extensão Kit de Ferramentas do Teams.
1. No painel de extensão do **Kit de Ferramentas do Teams**, escolha **Criar um Novo Aplicativo**.
1. Na lista de modelos de projeto, escolha **Agente do Copilot**.
1. Na lista de recursos do aplicativo, escolha **Agente declarativo**.
1. Escolha a opção **Adicionar plug-in**.
1. Escolha a opção **Iniciar com uma nova API**.
1. Na lista de tipos de autenticação, escolha **Chave de API (Autenticação de Token de Portador)**.
1. Para a linguagem de programação, selecione **TypeScript**.
1. Escolha uma pasta para armazenar o seu projeto.
1. Nomeie o seu projeto **da-repairs-key**.

O Kit de Ferramentas do Teams cria um novo projeto que inclui um agente declarativo, um plug-in de API e uma API protegida com uma chave.

## Tarefa 2 — Examinar a configuração de autenticação da chave de API

Antes de continuar, examine a configuração de autenticação de chave de API no projeto gerado.

### Examinar a definição de API

Primeiro, dê uma olhada em como a autenticação de chave de API é definida na definição da API.

No Visual Studio Code:

1. Abra o arquivo **appPackage/apiSpecificationFile/repair.yml**. Esse arquivo contém a definição OpenAPI para a API.
1. Na seção **components.securitySchemes**, observe a propriedade **apiKey**:

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  A propriedade define um esquema de segurança que usa a chave de API como token de portador no cabeçalho da solicitação de autorização.

1. Localize a propriedade **paths./repairs.get.security**. Observe que ele faz referência ao esquema de segurança **apiKey**.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### Examinar a implementação da API

Em seguida, veja como a API valida a chave de API em cada solicitação.

No Visual Studio Code:

1. Abra o arquivo **src/functions/repairs.ts**.
1. Na função manipuladora de **reparos**, localize a seguinte linha, que rejeita todas as solicitações não autorizadas:

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. A função **isApiKeyValid** é implementada ainda mais no arquivo repairs.ts:

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  A função verifica se o cabeçalho de autorização contém um token de portador e o compara com a chave de API definida na variável de ambiente **API_KEY**.

Esse código mostra uma implementação simplista da segurança da chave de API, mas ilustra como a segurança da chave de API funciona na prática.

### Examinar a configuração da tarefa do cofre

Neste projeto, você usa o Kit de Ferramentas do Teams para adicionar a chave de API ao cofre. O Kit de Ferramentas do Teams registra a chave de API no cofre usando uma tarefa especial na configuração do projeto.

No Visual Studio Code:

1. Abra o arquivo **./teampsapp.local.yml**.
1. Na seção de **provisionamento**, localize a tarefa **apiKey/register**.

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  A tarefa pega o valor da variável de projeto **SECRET_API_KEY**, armazenada no arquivo **env/.env.local.user** e o registra no cofre. Em seguida, ele pega o ID de entrada do cofre e o grava no arquivo de ambiente **env/.env.local**. O resultado dessa tarefa é uma variável de ambiente chamada **APIKEY_REGISTRATION_ID**. O Kit de Ferramentas do Teams grava o valor dessa variável no arquivo **appPackages/ai-plugin.json** que contém a definição do plug-in. No runtime, o agente declarativo que carrega o plug-in da API usa essa ID para recuperar a chave de API do cofre e chamar a API com segurança.

## Tarefa 3 — Configurar a chave de API para desenvolvimento local

Antes de testar o projeto, é preciso definir uma chave de API para a sua API. Em seguida, armazene a chave de API no cofre e registre o ID de entrada do cofre em seu plug-in de API. Para desenvolvimento local, armazene a chave de API em seu projeto e use o Kit de Ferramentas do Teams para registrá-la no cofre para você.

No Visual Studio Code:

1. Abra o painel **Terminal** (Ctrl + ').
1. Em uma linha de comando:
  1. Restaure as dependências do projeto, executando`npm install`.
  1. Gere uma nova chave de API executando: `npm run keygen`.
  1. Copie a chave gerada para a área de transferência.
1. Abra o arquivo **env/.env.local.user**.
1. Atualize a propriedade **SECRET_API_KEY** para a chave de API recém-gerada. A propriedade atualizada é a seguinte:

  ```text
  SECRET_API_KEY='your_key'
  ```

1. Salve suas alterações.

Cada vez que você cria o projeto, o Kit de Ferramentas do Teams atualiza automaticamente a chave de API no cofre e atualiza o seu projeto com a ID de entrada do cofre.