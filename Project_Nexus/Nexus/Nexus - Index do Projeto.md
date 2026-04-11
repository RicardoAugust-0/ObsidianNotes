---
tags:
  - nexus
  - projeto
  - index
aliases:
  - Nexus
  - Nexus CRM
created: 2026-04-03
updated: 2026-04-08
status: 🟢 Em Desenvolvimento Ativo
---

# 🧠 Nexus — Plataforma CRM Inteligente com IA

> **Nexus** é uma plataforma SaaS de CRM com inteligência artificial integrada, construída pela **RP Consultoria**. Este índice serve como o **mapa central** para navegação técnica do projeto.

---

## 🗺️ Estrutura do Vault

```
Nexus/
├── 📄 Nexus - Index do Projeto (esta nota)
│
├── 📂 01 - Contexto/
│   ├── Nexus - Visão Geral e Identidade
│   ├── Nexus - Regras de Negócio e Segurança
│   └── Nexus - Glossário
│
├── 📂 02 - Arquitetura/
│   ├── Nexus - Arquitetura
│   ├── Nexus - Stack Tecnológica
│   ├── Nexus - Modelo de Dados e Banco
│   ├── Nexus - Decisões Técnicas (ADR)
│   └── Nexus - Padrões e Convenções
│
├── 📂 03 - Código/
│   ├── Nexus - Fluxo de Dados
│   ├── Nexus - Mapeamento de Arquivos
│   └── FC - PreLeadsSection
│
├── 📂 04 - Edge Functions/
│   ├── Nexus - Edge Functions (overview)
│   ├── Nexus - Shared Helpers (_shared/)
│   ├── Nexus - nexus-assistant Arquitetura
│   └── 20 notas individuais (EF - *.md) — 22 funções no código
│
├── 📂 05 - Roadmap/
│   ├── Nexus - Melhorias e Roadmap
│   ├── Nexus - Revamp Frontend
│   └── Nexus - Sugestões de Melhoria
│
└── 📂 06 - Sprints/
    ├── Sprint 01 - Infra & Segurança
    ├── Sprint 02 - Observabilidade & Modularização
    ├── Sprint 03 - Performance & Tipos
    ├── Sprint 04 - Admin, Integrações e Desempenho
    ├── Sprint 05 - Revisão de Segurança Backend
    └── Sprint 06 - Melhorias de Qualidade Backend
```

---

## 📋 01 — Contexto de Negócio

| Documento | O que contém |
|:---|:---|
| [[Nexus - Visão Geral e Identidade]] | Proposta de valor, fluxo de usuário, diferenciais e modelo SaaS |
| [[Nexus - Regras de Negócio e Segurança]] | Economia de créditos, Gates, RLS, roles e Zero Trust |
| [[Nexus - Glossário]] | 📖 40+ termos-chave do projeto com links cruzados |

## 🏗️ 02 — Arquitetura Técnica

| Documento | O que contém |
|:---|:---|
| [[Nexus - Arquitetura]] | Diagrama de camadas, guardas de rota, hooks, backend |
| [[Nexus - Stack Tecnológica]] | Versões reais, dependências, RPCs e env vars |
| [[Nexus - Modelo de Dados e Banco]] | Schema real das 25+ tabelas, views, enums e diagrama ER |
| [[Nexus - Decisões Técnicas (ADR)]] | 10 decisões de design com contexto, trade-offs e consequências |
| [[Nexus - Padrões e Convenções]] | Design patterns, hooks, cache, naming, env vars |

## 📚 03 — Código

| Documento | O que contém |
|:---|:---|
| [[Nexus - Fluxo de Dados]] | 8 fluxos com diagramas de sequência (Auth, Chat, Leads, Créditos, etc.) |
| [[Nexus - Mapeamento de Arquivos]] | GPS do código — 100+ arquivos explicados por diretório |
| [[FC - PreLeadsSection]] | Componente de pré-leads (Google Maps + LinkedIn Scout) — status, ações, filtros |

## ⚡ 04 — Edge Functions (22 funções)

> Nota principal: [[Nexus - Edge Functions]]

| Categoria | Funções |
|:---|:---|
| 🤖 **IA & Análise** | [[EF - nexus-assistant]] · [[EF - analyze-lead-360]] · [[EF - analyze-leads-data]] · [[EF - analyze-linkedin-lead]] · [[EF - analyze-opportunities]] · [[EF - analyze-website]] · [[EF - generate-message-suggestions]] · [[EF - summarize-chat-history]] · [[EF - summarize-meeting-prep]] |
| 📝 **Conteúdo** | [[EF - generate-post-caption]] · [[EF - generate-brand-image]] · [[EF - generate-twitter-carousel]] · [[EF - generate-chat-embeddings]] |
| 🔍 **Busca** | [[EF - search-leads]] · [[EF - whatsapp-templates]] · `send-whatsapp-message` ⚠️ sem nota |
| ⚙️ **Sistema** | [[EF - process-action]] · [[EF - get-integration-config]] · [[EF - test-integration]] · [[EF - check-achievements]] · [[EF - list-users]] · `add-credits` ⚠️ sem nota |

**Infraestrutura compartilhada:**

| Documento | O que contém |
|:---|:---|
| [[Nexus - Shared Helpers]] | `_shared/auth.ts` · `logging.ts` · `credits.ts` — helpers usados por todas as funções |
| [[Nexus - nexus-assistant Arquitetura]] | Documentação dos 8 módulos do nexus-assistant refatorado |

## 🚀 05 — Roadmap & Sugestões

| Documento | O que contém |
| :--- | :--- |
| [[Nexus - Melhorias e Roadmap]] | 25 melhorias priorizadas: 🔴 4 críticas · 🟠 4 alto · 🟡 3 médio · 🟢 6 novas · 🎨 8 UI |
| [[Nexus - Revamp Frontend]] | **Planejamento de Revamp:** Nova arquitetura Atomic, Design System Pro (Glassmorphism) e Auditoria Visual |
| [[Nexus - Sugestões de Melhoria]] | **12 sugestões** de produto, infra e UX com matriz de impacto × esforço |

## 🏃 06 — Sprints (Histórico de Execução)

| Sprint                                          | Status | Foco                                                  |
| :---------------------------------------------- | :----: | :---------------------------------------------------- |
| [[Sprint 01 - Infra & Segurança]]               |   ✅    | SEC-001, SEC-002, BUG-001, BUG-002, LOG-001, PERF-002 |
| [[Sprint 02 - Observabilidade & Modularização]] |   ✅    | COST-001, LOG-001 (completo), QUAL-002, MOD-001       |
| [[Sprint 03 - Performance & Tipos]]             |   ✅    |                                                       |
| [[Sprint 04 - Admin, Integrações e Desempenho]] |   ✅    |                                                       |
| [[Sprint 05 - Revisão de Segurança Backend]]    |   ✅    |                                                       |
| [[Sprint 06 - Melhorias de Qualidade Backend]]  |   ✅    | Sanitização de inputs, mensagens de erro, seg. pontual |

---

## 🚀 Funcionalidades Principais

- **Nexus Assistant**: Chat IA com streaming SSE, busca semântica e orquestração de ações
- **CRM Completo**: Gestão de leads com tabela rica, Kanban e import CSV
- **Prospecção Ativa**: LinkedIn Scout + Email Search — orquestrados via chat
- **Análise 360°**: IA analisa leads, websites, perfis LinkedIn e sugere abordagens
- **Marketing**: Captions, imagens de marca e carrosséis com IA
- **Omnichannel**: Templates WhatsApp Business via Meta Graph API v23.0
- **Performance**: Rankings, metas, conquistas gamificadas
- **SaaS Engine**: Créditos, planos (Free/Basic/Premium/Pro/Enterprise) e multi-tenancy

---

## 🛠️ Stack em uma Linha

**Frontend:** React 18 / Vite 7 / Tailwind 3 · **Backend:** Supabase / PostgreSQL / 22 Edge Functions (Deno) · **IA:** OpenAI (primário, via Lovable Gateway) / Gemini 2.5 Flash (fallback) / Tavily

---

## 📊 Números do Projeto

|      Métrica       | Valor |
| :---------------: | :---: |
| Notas neste vault |  40+  |
|  Páginas (rotas)  |  17   |
|  Edge Functions   |  22 (search-leads com pipeline de Inteligência Geográfica)   |
| Tabelas no banco  |  25+  |
|  Migrations SQL   |  38   |
| Componentes React |  80+  |
|  Sprints Concluídas | 6   |

---

## 🔗 Links Rápidos

- **Repositório Lovable**: [Projeto no Lovable](https://lovable.dev/projects/e566b2b3-9c6f-48fb-9be7-1c45b975c14f)
- **Supabase Dashboard**: `mqnpkesyfkqknykdrgwg.supabase.co`
