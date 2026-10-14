##  Painel de Chamados (Frontend)

O Painel de Chamados é a interface administrativa web projetada para o Nível 1 de suporte. Desenvolvida focando em velocidade, clareza e conformidade com a LGPD.

###  Tecnologias Utilizadas
- **HTML5 & Vanilla JavaScript:** Estrutura leve sem necessidade de frameworks pesados para garantir performance e facilidade de deploy.
- **Tailwind CSS:** Utilizado via CDN para estilização responsiva, componentes modulares e design system consistente.
- **Hospedagem:** GitHub Pages, configurado para consumir dados via requisição de uma origem externa (Ngrok) através da configuração de CORS.

###  Funcionalidades Principais e UX
- **Consumo de API Dinâmica:** Realiza requisições assíncronas ao endpoint do n8n para atualizar a lista de chamados, garantindo uma interface fluida.
- **Conformidade com a LGPD (Data Masking):** 
  - Todos os números de telefone (dados sensíveis) são carregados do banco de dados e imediatamente mascarados no frontend via JavaScript.
  - Implementado um "Toggle Button" (ícone de olho) que permite ao atendente revelar temporariamente o número apenas quando estritamente necessário para o contato, garantindo a privacidade do cliente (*Privacy by Default*).
- **Agrupamento Inteligente (Group By):** A interface processa o array de dados e agrupa os tickets por cliente. A tela inicial (lista principal) exibe apenas o chamado mais recente e urgente de cada usuário, evitando poluição visual.
- **Sistema de Modal (Dossiê do Cliente):**
  - Ao clicar em um ticket, um modal em overlay é aberto.
  - O modal é dividido em 3 áreas funcionais para agilizar o atendimento: Detalhes do Usuário/Problema, Resposta Gerada pela IA e Ações Rápidas.
  - O modal também renderiza, na parte inferior, o **Histórico Completo** do cliente, listando todos os chamados anteriores abertos por aquele mesmo número, permitindo que o atendente compreenda as recorrências.

---

##  Artefatos do Repositório
Este repositório é focado na arquitetura de backend, automação e inteligência de dados. Abaixo estão os artefatos técnicos inclusos:

- `[]
`: Exportação completa do fluxo do n8n para importação e auditoria da lógica de automação e integração com a IA.
- ``: Arquivo fonte do Power BI layout do dashboard de inteligência.

> 💡 **Demonstração em Tempo Real:** Abaixo, um fluxo completo demonstrando o recebimento da mensagem no WhatsApp, o processamento da IA e o reflexo imediato no Painel Web.
>
> *(Insira o seu GIF aqui)*
