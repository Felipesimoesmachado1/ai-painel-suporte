# Arquitetura de Fluxos (n8n)

## Fluxo Principal-AI: Triagem, Decisão e Resposta

**Objetivo:** Atuar como o motor cognitivo do sistema, gerenciando o ciclo de vida completo do atendimento desde a recepção da mensagem até o retorno ao usuário.

### 1. Webhook (Trigger)
*   **Função:** Ouvinte de eventos da Evolution API.
*   **Configuração:** Recebe requisições HTTP POST contendo o payload em tempo real de todas as interações no WhatsApp conectado.

### 2. IF Node (if_antiloop)
*   **Função:** Controle de redundância e prevenção de loops.
*   **Configuração:** Avalia a propriedade `{{ $json.body.data.key.fromMe }}`.
*   **Regra de Negócio:** Se `true` (mensagem enviada pelo próprio bot), o fluxo é encerrado. Se `false` (mensagem do cliente), o processamento continua.

### 3. Set Node (Extrair_Dados)
*   **Função:** Sanitização e padronização do payload.
*   **Configuração:** Isola as variáveis essenciais para o sistema (nome, telefone e mensagem), descartando os metadados desnecessários do WhatsApp para otimizar o consumo de memória nas próximas etapas.

### 4. Set Node (Gerar Protocolo)
*   **Função:** Geração de identificador único (ID).
*   **Configuração:** Executa uma função matemática para gerar um protocolo numérico aleatório de 5 dígitos (ex: 84512). Este número fica disponível como variável absoluta para ser utilizado pela IA e salvo no banco.

### 5. AI Agent (Chat model, Postgres chat memory)
*   **Função:** Processamento de Linguagem Natural (LLM) com retenção de contexto.
*   **Configuração:** Utiliza o modelo do Google Gemini atrelado a um nó de memória persistente no PostgreSQL.
*   **Regra de Negócio:** A memória permite que a IA lembre do histórico de conversa daquele usuário específico (através do número do telefone). O agente analisa a mensagem atual para determinar se o cliente está apenas conversando/tirando uma dúvida rápida ou se relatou um problema que exige a abertura de um ticket de suporte.

### 6. Parse Node (Parse IA)
*   **Função:** Estruturação de dados não-estruturados e sanitização de saída.
*   **Configuração:** Converte a resposta de texto livre gerada pelo AI Agent em um formato JSON rigoroso. Extrai parâmetros-chave definidos no prompt, como `categoria`, `prioridade`, `resposta` (texto limpo da IA) e o marcador lógico booleano de decisão (`precisa_abrir_ticket`: `true`/`false`).

### 7. IF Node (if_decisão)
*   **Função:** Roteamento de fluxo baseado na análise cognitiva.
*   **Configuração:** Avalia a intenção identificada pelo AI Agent lendo a variável booleana gerada pelo Parse IA. Em vez de checar se a variável existe, a condição verifica se a intenção requer ação do banco de dados (ex: `{{ $json.precisa_abrir_ticket }} Is True`).
*   **Ramificações:**
    *   **Caminho True (Necessita de Chamado Técnico):** O usuário relatou um problema real. O sistema deve registrar o incidente e avisar o usuário que um protocolo foi gerado.
    *   **Caminho False (Apenas Conversa):** O usuário enviou uma saudação, dúvida genérica ou agradecimento. O sistema apenas responde sem poluir o banco de dados.

### 8. Caminho TRUE (Abertura de Chamado)
*   **A. Save (PostgreSQL - Execute Query):** Executa o comando `INSERT INTO chamados_suporte` para salvar as variáveis absolutas (protocolo, telefone, mensagem original, categoria, prioridade e resposta da IA) na tabela principal, alimentando o Dashboard web e o Power BI. *(Aviso Arquitetural: Nós de banco de dados no n8n sobrescrevem o payload original, devolvendo apenas `success: true`. Variáveis em nós subsequentes devem usar referências absolutas).*
*   **B. HTTP Request_ticket (Evolution API):** Dispara uma requisição HTTP POST para a API do WhatsApp, enviando a mensagem oficial de abertura de chamado com o protocolo gerado. 
    *   *Nota Crítica de Configuração:* Os parâmetros do Body (`number` e `text`) não podem conter espaços em branco em seus nomes de chave. O valor deve buscar o texto diretamente da fonte geradora, ignorando a sobrescrita do banco de dados: `{{ $('Parse IA').first().json.resposta }}`.

### 9. Caminho FALSE (Atendimento Simples)
*   **A. HTTP Request_CHAT (Evolution API):** Dispara uma requisição HTTP POST diretamente para o WhatsApp do usuário contendo apenas a resposta conversacional gerada pela IA, mantendo o engajamento sem criar registros no banco de suporte. Assegura-se que fallback de lógicas condicionais (ex: `||`) sejam removidos para evitar o envio de payloads JSON crus (não formatados) para o cliente.

---

## Fluxo 2: API REST para o Dashboard (Frontend)

**Objetivo:** Servir os dados do banco de dados para a interface web de forma segura.

### 1. Webhook (Endpoint da API)
*   **Função:** Receber requisições do frontend (HTML/JS).
*   **Configuração:**
    *   **Método:** `GET`
    *   **Caminho (Path):** `/chamados`
    *   **Segurança (CORS):** Ativado em Options para permitir que o GitHub Pages consuma os dados sem ser bloqueado pela política de mesma origem dos navegadores.
    *   **Resposta:** Configurado para `Using Respond to Webhook Node`.

### 2. PostgreSQL (Busca)
*   **Função:** Consultar os chamados recentes.
*   **Configuração:** Executa um `SELECT * FROM chamados_suporte ORDER BY id DESC LIMIT 10`, garantindo que o painel exiba sempre os tickets mais novos primeiro.

### 3. Respond to Webhook (Resposta HTTP)
*   **Função:** Empacotar e devolver os dados.
*   **Configuração:** Transforma as linhas brutas do banco de dados em um array JSON formatado utilizando a expressão: `{{ $input.all().map(item => item.json) }}`. Esse JSON é então retornado para o `fetch()` do aplicativo web, que renderiza os cards no painel.

> **Nota Técnica Global da Arquitetura:** 
> Todo o fluxo utiliza referências de nós absolutas `($())` no n8n em vez de relativas `($json)` em nós de saída (HTTP Requests). Isso garante que a arquitetura seja escalável e não sofra perda de contexto (erros de `undefined` ou `[empty]`) quando nós intermediários (como operações de Banco de Dados, que devolvem `success: true`) sobrescreverem o JSON ao longo do processo.
