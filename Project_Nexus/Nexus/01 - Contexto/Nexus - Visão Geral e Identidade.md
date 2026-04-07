---
tags:
  - nexus
  - documentacao
  - contexto
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 🎯 Nexus — Visão Geral e Identidade

> **O que é o Nexus?**
> O Nexus é um ecossistema SaaS de CRM e Inteligência de Vendas. Ele não é apenas um banco de dados de contatos, mas um **Copiloto de Vendas** que automatiza a prospecção, análise de leads e geração de conteúdo via IA.

---

### Proposta de Valor

1. **Inteligência Centralizada**: Usa IA (OpenAI + Tavily) para analisar perfis de leads, sites e preparar reuniões.
2. **Prospecção Ativa**: Busca de decisores via LinkedIn Search e Email Search, orquestrada pelo chat IA.
3. **Gestão de Relacionamento**: Pipeline de vendas com visualização Kanban, tabela rica, e detalhamento profundo de cada lead.
4. **Comunicação Omnichannel**: Templates de WhatsApp Business via Meta Graph API v23.0, com CRUD completo.
5. **Conteúdo com IA**: Geração de imagens de marca, captions e carrosséis para redes sociais.

---

### Fluxo Principal de Usuário

1. **Atração**: O usuário busca leads via Nexus Chat (IA orquestra buscas LinkedIn/email) ou importa dados via CSV.
2. **Qualificação**: A IA analisa o "Lead 360" (website, perfil, informações) e sugere abordagens.
3. **Engajamento**: O usuário usa o Nexus Assistant para criar estratégias, envia templates de WhatsApp, ou usa sugestões de mensagem.
4. **Fechamento**: Atribuição a closers, gestão de reuniões (com preparação por IA) e pipeline de oportunidades.
5. **Análise**: Dashboards com KPIs, funil de conversão, performance de equipe e conquistas gamificadas.

---

### Diferenciais Técnicos

- **Zero Trust UI**: Componentes `CreditGate` e `PlanGate` (em `PlanMiddleware.tsx`) protegem tanto no frontend quanto validam no backend via `consume_credits()`.
- **Serverless First**: 20 Edge Functions (Deno) no Supabase, sem servidor próprio.
- **Realtime**: Atualizações de saldo de créditos e notificações via Supabase Realtime (`postgres_changes`).
- **Command Bus via Chat**: O Nexus Chat interpreta linguagem natural e despacha ações via marcadores (`[LINKEDIN_SEARCH_STARTED]`, `[EMAIL_SEARCH_STARTED]`).
- **Multi-tenant**: Isolamento por `user_id` com RLS (Row Level Security) em todas as tabelas.

---

### Modelo SaaS

| Aspecto | Detalhe |
|:---|:---|
| **Planos** | `free`, `basic`, `premium`, `pro`, `enterprise` (enum `plan_type`) |
| **Roles** | `admin` e `client` (enum `user_role`) |
| **Créditos** | Coluna `credits` em `profiles`. Mutações consomem créditos; leitura é gratuita |
| **Pagamentos** | Preparado para Stripe (`stripe_customer_id`, `subscription_status` em `profiles`) e MercadoPago (`mp_user_id`) |
| **Admin padrão** | CEO (`rhyanpaablo@gmail.com`) recebe `role: admin` automaticamente via trigger de signup |

---

### Notas Relacionadas

- [[Nexus - Index do Projeto]] — Hub central de navegação
- [[Nexus - Stack Tecnológica]] — Detalhes da infraestrutura
- [[Nexus - Arquitetura]] — Como as peças se conectam
