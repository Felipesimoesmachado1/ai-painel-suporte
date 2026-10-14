#  Sistema de Triagem e Suporte Inteligente com IA

##  Sobre o Projeto
Este projeto é a base de um Micro-SaaS para gestão de atendimento. Ele recebe mensagens de usuários via WhatsApp, utiliza Inteligência Artificial para interpretar, categorizar e priorizar o problema, gera um protocolo de atendimento e salva os dados de forma estruturada em um banco de dados relacional para consumo analítico (Power BI) e gestão via Dashboard web.

##  Arquitetura e Tecnologias
- **Infraestrutura:** Docker & Docker Compose
- **Orquestração de Fluxos:** n8n (Self-hosted)
- **Inteligência Artificial:** Google Gemini (LLM)
- **Banco de Dados:** PostgreSQL
- **Gateway de Mensagens:** Evolution API (WhatsApp)
- **Túnel de Rede:** Ngrok
- **Análise de Dados:** Power BI
- **Frontend:** HTML/JS com Tailwind CSS (Hospedado via GitHub Pages)

---

## Infraestrutura e Containerização (Docker🐳)
Todo o ecossistema de back-end opera em um ambiente isolado e conteinerizado, gerenciado através do **Docker Compose**. Essa abordagem garante portabilidade, fácil replicação do ambiente em qualquer servidor (Cloud ou On-Premise) e independência de sistema operacional.

Os seguintes serviços rodam simultaneamente na mesma rede lógica (`docker-network`):
1. **Container PostgreSQL:** Hospeda o banco de dados relacional e a memória de contexto (Chat Memory) da Inteligência Artificial.
2. **Container n8n:** Executa a orquestração de dados e APIs na porta `5678`.
3. **Container Evolution API:** Gerencia a sessão do WhatsApp via instâncias locais, conectando-se ao n8n via Webhooks.

---

##  Estrutura do Banco de Dados (PostgreSQL)
A tabela `chamados_suporte` atua como a *Single Source of Truth* (Fonte Única de Verdade) do sistema.

```sql
CREATE TABLE chamados_suporte (
    id SERIAL PRIMARY KEY,
    protocolo VARCHAR(20) UNIQUE NOT NULL,
    telefone_cliente VARCHAR(20),
    nome_cliente VARCHAR(100),
    mensagem_original TEXT,
    categoria VARCHAR(50),
    prioridade VARCHAR(20),
    resposta_ia TEXT,
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 📊 Analytics e Business Intelligence (Power BI)
O sistema não apenas resolve o fluxo operacional, mas gera inteligência para a tomada de decisão gerencial. Um painel no Power BI foi conectado nativamente ao PostgreSQL (via modo Import/DirectQuery) para consumir a tabela `chamados_suporte`.

**Processo de ETL e Modelagem:**
- Tratamento de tipos de dados (Datas e Textos) via Power Query.
- Criação de medidas em DAX para contagem dinâmica e filtragem de severidade.

**Métricas e KPIs acompanhados no Dashboard:**
- **Volume Geral:** Contagem total de chamados abertos.
- **Distribuição por Categoria:** Gráficos mostrando quais áreas (Sistema, Hardware, Dúvida) demandam mais esforço do suporte.


*<img width="896" height="467" alt="image" src="https://github.com/user-attachments/assets/6954f4ce-b44d-4db2-b0ff-382c6fd6621b" />*

