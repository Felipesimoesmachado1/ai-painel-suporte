# 🧠 Sistema de Triagem e Suporte Inteligente com IA

## 🎯 Sobre o Projeto
Este projeto é a base de um Micro-SaaS para gestão de atendimento. Ele recebe mensagens de usuários via WhatsApp, utiliza Inteligência Artificial para interpretar, categorizar e priorizar o problema, gera um protocolo de atendimento e salva os dados de forma estruturada em um banco de dados relacional para consumo analítico (Power BI) e gestão via Dashboard web.

## 🏗️ Arquitetura e Tecnologias
- **Orquestração de Fluxos:** n8n (Self-hosted)
- **Inteligência Artificial:** Google Gemini (LLM)
- **Banco de Dados:** PostgreSQL
- **Gateway de Mensagens:** Evolution API (WhatsApp)
- **Túnel de Rede:** Ngrok
- **Análise de Dados:** Power BI

---

## 🗄️ Estrutura do Banco de Dados (PostgreSQL)
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
