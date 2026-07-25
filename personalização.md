# 🚀 Prompt Principal - Hobbyist AI (Hackathon)

> **Instruções:** Copie todo o conteúdo abaixo e cole na conversa com o Claude para gerar a estrutura completa e integrada dos arquivos do projeto.

---

```text
Olá! Estou desenvolvendo o projeto "Hobbyist AI" para um hackathon/workshop de IA Serverless utilizando a infraestrutura criada pela equipe do evento (Node.js, AWS Lambda, Amazon Bedrock com Claude Haiku, DynamoDB e AWS SAM).

Abaixo está a estrutura exata do nosso repositório de arquivos:
- src/agent.mjs
- src/chat-page.mjs
- src/index.mjs
- package.json
- samconfig.toml
- template.yaml

---

### 💡 Visão Geral do Projeto: "Hobbyist AI"
O objetivo do agente é tirar as pessoas da rotina descobrindo HOBBIES DIFERENTÕES e EXÓTICOS. O agente:
1. Realiza uma entrevista inicial de 4 perguntas sobre o perfil do usuário.
2. Consulta uma base externa via protocolo MCP e cruza com os produtos do nosso catálogo no DynamoDB.
3. Gera um ROADMAP DE PROJETO EM 3 NÍVEIS (Iniciante, Intermediário e Avançado) com opção de download em PDF.
4. Conduz a FASE DE VALIDAÇÃO E GAMIFICAÇÃO para emitir um BADGE OFICIAL DE CONQUISTA.

---

### 🛠️ Requisitos Técnicos e de Negócio:

1. **Stack e SDKs:**
   - Framework do Agente: `@strands-agents/sdk` (`Agent`, `BedrockModel`, `tool`).
   - SDK da AWS para DynamoDB: `@aws-sdk/client-dynamodb` e `@aws-sdk/lib-dynamodb` (`ScanCommand`, `GetCommand`, `PutCommand`).
   - Validação de schemas das ferramentas: `zod`.
   - Variáveis de ambiente disponíveis: `process.env.SESSIONS_TABLE` e `process.env.PRODUCTS_TABLE`.

2. **Lógica e Fluxo Completo do Agente (`src/agent.mjs`):**

   - **Memória/Sessões:** Mantém o histórico via `loadHistory` e `saveHistory` na `SESSIONS_TABLE` (com expiração de 24h via `expiresAt`).
   
   - **Tools do Agente:**
     - `fetch_mcp_hobby_database`: Conecta a um servidor MCP externo (via requisição HTTP POST no padrão JSON-RPC / MCP) para consultar ideias de hobbies exóticos. Se falhar, usa a inteligência interna.
     - `search_hobby_equipment`: Busca kits e equipamentos na `PRODUCTS_TABLE` do DynamoDB para sugerir na conversa.
     - `check_shipping`: Checa estimativa de prazo de entrega para o país do usuário.
     - `generate_badge`: Gera o payload/objeto do Badge de Conquista (com Título do Badge, Descrição da Conquista, Nível Atingido e Data) após o envio da foto.

   - **System Prompt & Fluxo Obrigatório da Conversa:**
     - **ETAPA 1 (Entrevista de Perfil):** Nas primeiras mensagens, o agente NÃO sugere hobbies de imediato. Coleta as respostas destas 4 PERGUNTAS ESPECÍFICAS (conduzindo de forma amigável, 2 de cada vez):
       1. "Como você gosta de se sentir desafiado?"
       2. "Qual destas habilidades você gostaria de exercitar no novo hobby?" (Opções: Comunicação, Liderança, Colaboração, Resolução de Problemas, Interpessoal).
       3. "Qual ambiente e estilo mais te atrai?" (Ex: ao ar livre, manual em casa, digital).
       4. "Qual o seu tempo livre semanal e ritmo desejado?" (Ex: 30min diários, 2h no fim de semana).
     
     - **ETAPA 2 (Recomendação Exótica via MCP + DynamoDB):** Recomenda 2 a 3 hobbies fora do óbvio. Relaciona materiais do catálogo via `search_hobby_equipment`.
     
     - **ETAPA 3 (Roadmap em 3 Níveis):** Quando o usuário escolhe um hobby, gera o Roadmap (🟢 Nível 1 Iniciante, 🟡 Nível 2 Intermediário, 🔴 Nível 3 Avançado).
     
     - **ETAPA 4 (Gamificação, Feedback e Badge de Conquista):**
       - **4.1. Coleta de Feedback:** Após apresentar o projeto, o agente pergunta como foi a experiência ou o que o usuário achou do desafio (espera 1 resposta do usuário).
       - **4.2. Quiz Reflexivo 'Bate-Bola':** Com base no feedback recebido, o agente faz 3 perguntas curtas e diretas (cutucando os pontos trazidos pelo usuário no feedback) para fazê-lo refletir sobre o aprendizado.
       - **4.3. Solicitação de Evidência (Foto):** Após as respostas do quiz, o agente solicita que o usuário envie uma foto do projeto concluído para registrar a comprovação.
       - **4.4. Emissão do Badge:** Assim que o usuário enviar a foto (ou simular o envio), o agente parabeniza o usuário e chama a tool `generate_badge`, exibindo um BADGE/SELO DE CONQUISTA personalizado no chat (ex: "Mestre dos Terrários Herméticos - Nível 1 Concluído").

3. **Interface Web e PDF (`src/chat-page.mjs`):**
   - Exibe a interface web interativa do chat com streaming de tokens.
   - Suporta um botão de envio de arquivo/imagem para permitir o envio da foto de comprovação.
   - Inclua no HTML a biblioteca `html2pdf.js` via CDN para capturar o Roadmap e o Badge gerado na tela e permitir download em PDF com 1 clique ("📄 Baixar Guia + Badge em PDF").
   - Adicione uma estilização visual marcante (CSS) para renderizar o Badge de Conquista em destaque no chat quando for liberado.

4. **Orquestração da Lambda (`src/index.mjs`):**
   - Controla o roteamento HTTP, tratando requisições do chat (incluindo payloads de imagem/mensagens) e servindo a página `chat-page.mjs`.

5. **Dependências (`package.json`):**
   - Atualizado com todas as dependências necessárias para o projeto.

---

Por favor, forneça o código COMPLETO, limpo e pronto para uso de cada um dos arquivos (`src/agent.mjs`, `src/chat-page.mjs`, `src/index.mjs` e `package.json`), identificando claramente o caminho de cada arquivo antes do bloco de código.
