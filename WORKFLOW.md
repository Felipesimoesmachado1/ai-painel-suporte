# Arquitetura do Fluxo Principal (n8n)

Este documento detalha a engenharia do fluxo `Fluxo Principal-AI`, responsável por receber mensagens do WhatsApp, manter contexto conversacional, tomar decisões lógicas e persistir dados no banco relacional.

##  Fluxo Principal-AI: Triagem, Decisão e Resposta
**Objetivo:** Atuar como o motor cognitivo do sistema, gerenciando o ciclo de vida completo do atendimento desde a recepção da mensagem até o retorno ao usuário.

### 1. Webhook (Trigger)
- **Função:** Ouvinte de eventos da Evolution API.
- **Configuração:** Recebe requisições HTTP POST contendo o payload em tempo real de todas as interações no WhatsApp conectado.

### 2. IF Node (`if_antiloop`)
- **Função:** Controle de redundância e prevenção de loops.
- **Configuração:** Avalia a propriedade `{{ $json.body.data.key.fromMe }}`.
- **Regra de Negócio:** Se `true` (mensagem enviada pelo próprio bot), o fluxo é encerrado. Se `false` (mensagem do cliente), o processamento continua.

### 3. Set Node (`Extrair_Dados`)
- **Função:** Sanitização e padronização do payload.
- **Configuração:** Isola as variáveis essenciais para o sistema (`nome`, `telefone` e `mensagem`), descartando os metadados desnecessários do WhatsApp para otimizar o consumo de memória nas próximas etapas.

### 4. Set Node (`Gerar Protocolo`)
- **Função:** Geração de identificador único (ID).
- **Configuração:** Executa uma função matemática para gerar um protocolo numérico aleatório de 5 dígitos (ex: 84512). Este número fica disponível como variável absoluta para ser utilizado pela IA e salvo no banco.

### 5. AI Agent (`Chat model`, `Postgres chat memory`)
- **Função:** Processamento de Linguagem Natural (LLM) com retenção de contexto.
- **Configuração:** Utiliza o modelo do Google Gemini atrelado a um nó de memória persistente no PostgreSQL. 
- **Regra de Negócio:** A memória permite que a IA lembre do histórico de conversa daquele usuário específico (através do número do telefone). O agente analisa a mensagem atual para determinar se o cliente está apenas conversando/tirando uma dúvida rápida ou se relatou um problema que exige a abertura de um ticket de suporte.

### 6. Parse Node (`Parse IA`)
- **Função:** Estruturação de dados não-estruturados.
- **Configuração:** Converte a resposta de texto livre gerada pelo AI Agent em um formato JSON rigoroso. Extrai parâmetros-chave definidos no prompt, como `categoria`, `prioridade`, `resposta_texto` e o marcador de decisão (ex: `abrir_chamado: true/false`).

### 7. IF Node (`if_decisão`)
- **Função:** Roteamento de fluxo baseado na análise cognitiva.
- **Configuração:** Avalia a intenção identificada pelo AI Agent através dos dados parseados.
- **Ramificações:**
  - **Caminho `True` (Necessita de Chamado Técnico):** O usuário relatou um problema real. O sistema deve registrar o incidente.
  - **Caminho `False` (Apenas Conversa):** O usuário enviou uma saudação, dúvida genérica ou agradecimento. O sistema apenas responde sem poluir o banco de dados.

### 8. Caminho TRUE (Abertura de Chamado)
- **A. `Save` (PostgreSQL - Execute Query):** Executa o comando `INSERT INTO chamados_suporte` para salvar as variáveis absolutas (protocolo, telefone, mensagem original, categoria, prioridade e resposta da IA) na tabela principal, alimentando o Dashboard web e o Power BI.
- **B. `HTTP Request_ticket` (Evolution API):** Dispara uma requisição HTTP POST para a API do WhatsApp, enviando a mensagem oficial de abertura de chamado, informando a categoria, prioridade e o número do protocolo gerado.

### 9. Caminho FALSE (Atendimento Simples)
- **A. `HTTP Request_CHAT` (Evolution API):** Dispara uma requisição HTTP POST diretamente para o WhatsApp do usuário contendo apenas a resposta conversacional gerada pela IA, mantendo o engajamento sem criar registros no banco de suporte.
