---
lab:
  title: Introdução
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Introdução

Suponha que você tenha um sistema externo onde você armazena artigos da base de dados de conhecimento. Esses artigos contêm informações sobre diferentes processos da sua organização. Você quer poder localizar e descobrir facilmente informações relevantes do Microsoft 365. Você também quer que o Copilot para Microsoft 365 inclua informações desses artigos da base de dados de conhecimento em suas respostas.

Para expor essas informações externas dentro do Microsoft 365, você criará um conector personalizado do Microsoft Graph. Os conectores do Microsoft Graph se conectam ao seu sistema externo (1) para recuperar conteúdo, usar as informações do Microsoft Entra ID para autenticar com o Microsoft 365 (2) e importar o conteúdo para o Microsoft 365 usando a API do Microsoft Graph (3).

:::image type="content" source="../media/1-graph-connector-concept.png" alt-text="Diagrama que mostra o funcionamento conceitual de um conector do Microsoft Graph.":::

Neste módulo, você aprenderá o que são conectores do Microsoft Graph e por que você deve considerar usá-los em sua organização. Você criará um conector do Microsoft Graph que importa arquivos markdown locais para o Microsoft 365. Você também aprenderá sobre como garantir que o conteúdo externo importado seja acessível apenas a indivíduos com permissões atribuídas apropriadas. Por fim, você otimizará o conector do Microsoft Graph para uso com o Copilot para Microsoft 365.

## Pré-requisitos

- Conhecimento básico do C#
- Conhecimento básico de autenticação
- Acesso a um [locatário de desenvolvedor do Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program?ocid=MSlearn)
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Quando estiver pronto para começar, [continue no próximo exercício...](./2-exercise-configure-connection-schema.md) 