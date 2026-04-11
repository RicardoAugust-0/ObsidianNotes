---
tags:
  - nexus
  - arquivos
  - mapeamento
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 📁 Nexus — Mapeamento de Arquivos

> Referência completa de cada arquivo e diretório importante do [[Nexus - Index do Projeto|Nexus]]. Use esta nota como um "GPS" do código-fonte.

---

## Estrutura Raiz

```
nexus-project-main/
├── index.html              # HTML entry point (SPA root)
├── package.json            # Dependências e scripts npm
├── vite.config.ts          # Configuração do Vite (bundler)
├── tailwind.config.ts      # Tema e tokens do TailwindCSS
├── tsconfig.json           # Config base do TypeScript
├── tsconfig.app.json       # Config TS para código da aplicação
├── tsconfig.node.json      # Config TS para código Node (vite config)
├── postcss.config.js       # PostCSS (autoprefixer)
├── eslint.config.js        # Regras de linting
├── components.json         # Configuração do shadcn/ui
├── Dockerfile              # Container para deploy
├── docs/                   # Documentação de planejamento
├── public/                 # Assets estáticos
├── src/                    # Código-fonte principal
└── supabase/               # Backend (migrations + functions)
```

---

## `src/` — Código-Fonte Principal

### Arquivos de Entrada

| Arquivo | Responsabilidade |
|:---|:---|
| `main.tsx` | Bootstrap da aplicação. Cria root React e importa CSS global |
| `App.tsx` | **Ponto central**. Configura providers (Query, Theme, Tooltip, Router) e define todas as 17 rotas com seus guardas |
| `App.css` | Estilos específicos da aplicação |
| `index.css` | CSS global (TailwindCSS directives + custom properties) |
| `vite-env.d.ts` | Declarações de tipos para variáveis de ambiente Vite |

---

### `src/pages/` — Páginas da Aplicação

Cada arquivo representa uma **rota** completa da SPA.

| Página | Rota | Proteção | Descrição |
|:---|:---|:---|:---|
| `Auth.tsx` | `/auth` | Pública | Login e confirmação de email |
| `SignUp.tsx` | `/signup` | Pública | Cadastro de novo usuário (onboarding SaaS) |
| `Pricing.tsx` | `/pricing` | ProtectedRoute | Página de planos e preços |
| `Chat.tsx` | `/` | SecureRoute | **Página principal**. Nexus Chat com IA, layout especial com sidebar de conversas |
| `Index.tsx` | `/dashboard` | SecureRoute | Dashboard com KPIs, gráficos e insights de ML |
| `Leads.tsx` | `/leads` | SecureRoute | Agente Outbound — tabela de leads com import CSV |
| `LeadDetail.tsx` | `/leads/:id` | SecureRoute | Detalhe de um lead (chat, timeline, projetos, anexos) |
| `Opportunities.tsx` | `/opportunities` | SecureRoute | Pipeline de oportunidades e LinkedIn Scout |
| `PreLeadDetail.tsx` | `/opportunities/:id` | SecureRoute | Detalhe de pré-lead / oportunidade do LinkedIn |
| `Analytics.tsx` | `/analytics` | SecureRoute | Analytics avançado (funil, fontes, tokens, conversações) |
| `Performance.tsx` | `/performance` | SecureRoute | Rankings, metas, conquistas e alertas de equipe |
| `Meetings.tsx` | `/meetings` | SecureRoute | Gestão de reuniões com preparação por IA |
| `Integrations.tsx` | `/integrations` | SecureRoute | Configuração de integrações (WhatsApp, etc.) |
| `Posts.tsx` | `/posts` | AdminRoute | Kanban de postagens de marketing com IA |
| `Logs.tsx` | `/logs` | AdminRoute | Logs de chamadas de API e Edge Functions |
| `WhatsappTemplates.tsx` | `/whatsapp-templates` | AdminRoute | CRUD de templates WhatsApp Business |
| `NotFound.tsx` | `*` | Nenhuma | Página 404 |

---

### `src/components/` — Componentes

#### `auth/` — Autenticação e Controle de Acesso

| Componente | Função |
|:---|:---|
| `ProtectedRoute.tsx` | Guarda de rota: redireciona para `/auth` se não logado. Escuta `onAuthStateChange` |
| `VerifiedRoute.tsx` | Guarda de rota: exige email verificado. Mostra tela de verificação se pendente |
| `AdminRoute.tsx` | Guarda de rota: redireciona para `/` se não é admin. Usa [[Nexus - Arquitetura#2. Camada de Lógica de Negócio (Hooks)\|useUserRole]] |
| `PlanMiddleware.tsx` | Middlewares visuais: `CreditGate`, `PlanGate`, `UpgradeBanner`. Controla acesso por créditos e plano |

#### `layout/` — Estrutura Visual

| Componente | Função |
|:---|:---|
| `MainLayout.tsx` | Layout wrapper: Sidebar + conteúdo principal em container responsivo |
| `AppSidebar.tsx` | Sidebar de navegação colapsável. 11 itens, filtra por role. Inclui logo, subscription status e logout |
| `ThemeToggle.tsx` | Toggle dark/light mode usando `next-themes` |

#### `chat/` — Sistema de Chat com IA

| Componente | Função |
|:---|:---|
| `ChatConversation.tsx` | Container principal da conversa (mensagens + input) |
| `ChatInput.tsx` | Input de mensagem com envio |
| `ChatMessageItem.tsx` | Renderizador de mensagem individual (user/assistant). Limpa marcadores da IA |
| `ChatSidebar.tsx` | Lista de conversas passadas, renomear, deletar, limpar tudo |
| `ChatWelcome.tsx` | Tela de boas-vindas quando nenhuma conversa está selecionada |
| `ChatTrigger.tsx` | Botão trigger para abrir chat |
| `FloatingChat.tsx` | Chat flutuante (modal overlay) |
| `NexusChatModal.tsx` | Modal completo do Nexus Chat |
| `SemanticSearchButton.tsx` | Botão de busca semântica no chat |
| `LinkedInResultsCard.tsx` | Card de resultado de busca LinkedIn no chat |
| `LinkedInSearchProgress.tsx` | Indicador de progresso da busca LinkedIn |
| `EmailSearchResultsCard.tsx` | Card de resultado de busca por email |

#### `leads/` — Gestão de Leads

| Componente | Função |
|:---|:---|
| `LeadsTable.tsx` | **Componente mais complexo** (42KB). Tabela rica com filtros, bulk actions, import CSV |
| `LeadsKanban.tsx` | Visualização Kanban dos leads por status |
| `LeadsListView.tsx` | Visualização em lista simplificada |
| `LeadsTableView.tsx` | Visualização compacta em tabela |
| `AddLeadDialog.tsx` | Dialog para adicionar lead manualmente |
| `EditLeadDialog.tsx` | Dialog para editar dados do lead |
| `LeadDetailsDialog.tsx` | Dialog com detalhes completos do lead |
| `LeadDetailsSheet.tsx` | Sheet lateral com detalhes (36KB) |
| `LeadContextPanel.tsx` | Painel de contexto lateral (23KB) — timeline, projetos, anexos |
| `LeadChat.tsx` | **Chat direto com o lead** (36KB). Envio de mensagens, sugestões IA, templates |
| `LeadTimeline.tsx` | Timeline de interações com o lead |
| `LeadProjects.tsx` | Projetos associados ao lead |
| `LeadAttachments.tsx` | Anexos do lead |
| `QualificationModal.tsx` | Modal para qualificar o lead |
| `AssignCloserDialog.tsx` | Dialog para atribuir closer ao lead |
| `CallScheduleModal.tsx` | Modal de agendamento de ligação |
| `ChatAnalysisDialog.tsx` | Dialog de análise de conversas com IA |
| `AIAnalysisHistory.tsx` | Histórico de análises feitas pela IA |
| `FavoriteTemplates.tsx` | Templates de mensagem favoritos |

#### `opportunities/` — Oportunidades e LinkedIn Scout

| Componente | Função |
|:---|:---|
| `LinkedInScout.tsx` | **LinkedIn Scout** (23KB). Interface de prospecção no LinkedIn |
| `LinkedInAnalysisCards.tsx` | Cards de análise de perfis LinkedIn |
| `PreLeadsSection.tsx` | Seção de pré-leads (20KB) |
| `PreLeadsKanban.tsx` | Kanban de pré-leads |
| `PreLeadsTableView.tsx` | Tabela de pré-leads |
| `PreLeadCard.tsx` | Card de pré-lead individual |
| `PreLeadDetailsModal.tsx` | Modal de detalhes do pré-lead (23KB) |
| `LeadsExportReport.tsx` | Exportação de relatório de leads |
| `ProspectingMetricsDashboard.tsx` | Dashboard de métricas de prospecção |

#### `dashboard/` — Dashboard Principal

| Componente | Função |
|:---|:---|
| `MetricCard.tsx` | Card de métrica individual (KPI) |
| `WelcomeHeader.tsx` | Header de boas-vindas personalizado |
| `ConversionFunnelChart.tsx` | Gráfico de funil de conversão |
| `StatusDistributionChart.tsx` | Gráfico de distribuição de status |
| `LeadsBySegmentChart.tsx` | Gráfico de leads por segmento |
| `FunnelChart.tsx` | Componente genérico de funil |
| `MarketingMetrics.tsx` | Métricas de marketing |
| `MLInsights.tsx` | Insights de Machine Learning |
| `SmartPriorityPanel.tsx` | Painel de priorização inteligente |

#### `analytics/` — Analytics Avançado

| Componente | Função |
|:---|:---|
| `AnalyticsKPIs.tsx` | KPIs do módulo de analytics |
| `ConversationAnalytics.tsx` | Análise de conversações (7.8KB) |
| `PreLeadsAnalytics.tsx` | Análise de pré-leads (7KB) |
| `SalesFunnelChart.tsx` | Funil de vendas |
| `StatusDonutChart.tsx` | Donut de distribuição de status |
| `LeadSourceChart.tsx` | Gráfico de fontes de leads |
| `LeadsTrendChart.tsx` | Tendência de leads ao longo do tempo |
| `LeadVelocityRate.tsx` | Velocidade do pipeline |
| `CallsTimelineChart.tsx` | Timeline de ligações |
| `FunctionCostChart.tsx` | Custo das Edge Functions |
| `TokensDistributionChart.tsx` | Distribuição de tokens de IA |
| `TopRolesChart.tsx` | Top cargos prospectados |
| `TopSegmentsChart.tsx` | Top segmentos |
| `PainPointsAnalysis.tsx` | Análise de pain points |
| `MLInsightsPanel.tsx` | Painel de insights ML |

#### `posts/` — Postagens de Marketing com IA

| Componente | Função |
|:---|:---|
| `MarketingPostsKanban.tsx` | Kanban de postagens (17KB) |
| `AddMarketingPostDialog.tsx` | Dialog para criar postagem |
| `EditMarketingPostDialog.tsx` | Dialog para editar postagem |
| `PostDetailDialog.tsx` | Detalhes da postagem |
| `AutomatedPostForm.tsx` | Formulário de postagem automatizada |
| `BrandIdentityConfig.tsx` | Configuração de identidade visual |
| `CarouselGenerator.tsx` | Gerador de carrosséis |
| `TwitterCarouselGenerator.tsx` | Gerador de carrosséis para Twitter |
| `ImageGenerator.tsx` | Gerador de imagens com IA |

#### `performance/` — Performance e Gamificação

| Componente | Função |
|:---|:---|
| `PerformanceKPIs.tsx` | KPIs de performance da equipe |
| `PerformanceFilters.tsx` | Filtros de período e equipe |
| `PerformanceAlerts.tsx` | Alertas de performance |
| `RankingsChart.tsx` | Ranking de membros da equipe (11KB) |
| `SalesTable.tsx` | Tabela de vendas (11KB) |
| `TeamFunnelChart.tsx` | Funil por equipe |
| `GoalsPanel.tsx` | Painel de metas |
| `CreateGoalDialog.tsx` | Criar nova meta |
| `AchievementsPanel.tsx` | Painel de conquistas |

#### `whatsapp-templates/` — Templates WhatsApp

| Componente | Função |
|:---|:---|
| `TemplateListTable.tsx` | Tabela de listagem de templates |
| `TemplateCreateDialog.tsx` | Dialog de criação (21KB). Form com header, body, variáveis e buttons |
| `TemplateEditDialog.tsx` | Dialog de edição (19KB). Respeita restrições da Meta API |
| `TemplateDeleteDialog.tsx` | Confirmação de exclusão com aviso de cooldown |
| `TemplateStatusBadge.tsx` | Badge visual por status (Approved, Pending, Rejected, etc.) |
| `TemplatePreview.tsx` | Preview visual do template |

#### `credits/` — Economia de Créditos (UI)

> Componentes visuais que expõem o saldo de créditos do usuário. Consomem o hook [[#`src/hooks/` — Custom Hooks|useUserCredits]] e são independentes de qualquer lógica de dedução.

| Componente | Função |
|:---|:---|
| `CreditsBadge.tsx` | Badge compacto exibido no header/sidebar. Mostra saldo com ícone de moeda (`Coins`). Cor muda conforme saldo: **verde** ≥ 5, **âmbar** ≥ 2, **vermelho** < 2. Inclui `Tooltip` com aviso de saldo baixo. Props: `className`, `showLabel` |

> [!tip] Onde usar
> `CreditsBadge` é plug-and-play: adicione onde quiser exibir o saldo sem carregar nenhum contexto adicional — ele faz o próprio fetch via `useUserCredits`.

---

#### `subscription/` — Status de Plano e Assinatura (UI)

> Card compacto que combina **plano atual** + **saldo de créditos** em um único componente, ideal para Sidebar e painéis de perfil. Consome `useUserRole` (que já traz `planType`, `credits` e `isAdmin` num único fetch).

| Componente | Função |
|:---|:---|
| `SubscriptionStatus.tsx` | Card com badge do plano atual (`Iniciante` / `Basic` / `Pro` / `Enterprise` / `ADMIN`) e display de créditos. Admins veem `∞`. Se o plano for `free`, exibe botão CTA "Fazer Upgrade" que navega para `/pricing` |

**Mapeamento de planos → labels:**

| `plan_type` | Label exibido | Cor |
|:---|:---|:---|
| `free` | Iniciante | Zinc |
| `basic` | Basic | Azul |
| `pro` / `premium` | Pro | Primária |
| `enterprise` | Enterprise | Verde esmeralda |
| *(admin)* | ADMIN | Destrutiva (vermelho) |

> [!info] Diferença entre `CreditsBadge` e `SubscriptionStatus`
> - `CreditsBadge` → exibe **só créditos**, leve, uso inline no header
> - `SubscriptionStatus` → exibe **plano + créditos + CTA**, uso em sidebars e painéis

---

#### Outros Componentes

| Componente | Função |
|:---|:---|
| `ai/AISuggestions.tsx` | Sugestões inteligentes da IA no dashboard |
| `integrations/IntegrationConfigDialog.tsx` | Dialog para configurar credenciais de integrações |
| `ui/` | **Componentes shadcn/ui** — biblioteca completa de primitivos UI (botões, inputs, dialogs, etc.) |

---

### `src/hooks/` — Custom Hooks

| Hook | Linhas | Padrão | Função |
|:---|:---:|:---|:---|
| `useUserRole.tsx` | 92 | State + Realtime | Perfil do usuário: expõe `role`, `planType`, `credits`, `isAdmin`, `loading`. Escuta `postgres_changes` na tabela `profiles` para manter créditos em tempo real |
| `useUserCredits.ts` | 125 | State + RPC | Saldo de créditos em tempo real. Expõe `balance`, `loading`, `deduct_credits(amount)`, `hasEnoughCredits(amount)`, `refreshBalance()`. Chama `consume_credits` RPC |
| `useStreamingChat.ts` | 118 | Fetch + SSE | Streaming de mensagens via `EventSource` com a Edge Function `nexus-assistant`. Parseia marcadores especiais (`[LINKEDIN_SEARCH_STARTED]` etc.) |
| `useNexusConversations.ts` | 144 | React Query | CRUD completo de conversas do Nexus Chat. Queries com cache, mutations com `invalidateQueries` automático |
| `useWhatsappTemplates.ts` | ~100 | React Query | CRUD de templates WhatsApp via Edge Function `whatsapp-templates`. Cache e refetch automático |
| `useLinkedInSearch.ts` | ~100 | State Machine | Prospecção LinkedIn com estados `idle → loading → results → error`. Tracking de progresso por etapa |
| `useEmailSearch.ts` | ~50 | State | Busca por e-mail de decisores via Edge Function `search-leads`. Estado simples com `loading` + `results` |
| `useIntegrationConfigs.ts` | ~60 | React Query | Configurações de integrações por tenant (`user_id`). Abstrai fetch da tabela `integration_configs` |
| `useLeadAssignmentNotifications.tsx` | ~50 | Realtime | Escuta `postgres_changes` para atribuição de leads. Dispara notificação nativa do browser quando o lead é atribuído ao usuário logado |
| `usePushNotifications.tsx` | ~200 | Browser API | Gerencia Web Push Notifications via service worker. Solicita permissão, salva subscription e envia para o backend |
| `useDebounce.ts` | ~15 | Utility | Debounce genérico — retorna valor estabilizado após delay (ms). Usado em campos de busca |
| `use-mobile.tsx` | ~20 | Media Query | Detecta viewport mobile via `window.matchMedia`. Retorna `boolean` |
| `use-toast.ts` | ~100 | State | Sistema de toasts baseado no padrão shadcn/ui. Expõe `toast()` para exibir notificações efêmeras |

> [!note] Hooks de dados vs. hooks de UI
> - **Dados** (Supabase/Edge): `useUserRole`, `useUserCredits`, `useNexusConversations`, `useWhatsappTemplates`, `useIntegrationConfigs`, `useLeadAssignmentNotifications`
> - **Ações externas**: `useStreamingChat`, `useLinkedInSearch`, `useEmailSearch`, `usePushNotifications`
> - **Utilitários**: `useDebounce`, `use-mobile`, `use-toast`

---

### `src/lib/` — Utilitários

| Arquivo | Função |
|:---|:---|
| `utils.ts` | Função `cn()` — merge de classes TailwindCSS (`clsx` + `tailwind-merge`) |
| `whatsapp-types.ts` | Tipos TypeScript + Zod schemas para WhatsApp Templates (9KB). Cobre Template, Component, Button, Status, Category |

---

### `src/integrations/supabase/` — Integração Supabase

| Arquivo | Função |
|:---|:---|
| `client.ts` | Cria e exporta o client Supabase com URL e anon key. Auth persistente em localStorage |
| `types.ts` | **Tipos gerados automaticamente** (42KB) — definições TypeScript de todas as tabelas, views e functions do banco |

---

## `supabase/` — Backend

### `supabase/functions/` — 20 Edge Functions (Deno)

#### IA & Análise
| Função | Descrição |
|:---|:---|
| `nexus-assistant/` | **Motor principal do chat**. Integra com OpenAI para streaming SSE. Detecta intenções e despacha ações |
| `analyze-lead-360/` | Análise completa de um lead (360°) usando IA |
| `analyze-leads-data/` | Análise em batch de dados de leads |
| `analyze-linkedin-lead/` | Análise de perfil LinkedIn específico |
| `analyze-opportunities/` | Análise de pipeline de oportunidades |
| `analyze-website/` | Análise de website de empresa para enriquecer lead |
| `generate-message-suggestions/` | Gera sugestões de mensagens personalizadas para leads |
| `summarize-chat-history/` | Sumariza histórico de conversas |
| `summarize-meeting-prep/` | Prepara briefing para reuniões com IA |

#### Conteúdo & Mídia
| Função | Descrição |
|:---|:---|
| `generate-post-caption/` | Gera captions para postagens de marketing |
| `generate-brand-image/` | Gera imagens de marca via IA |
| `generate-twitter-carousel/` | Gera carrosséis para Twitter/X |
| `generate-chat-embeddings/` | Gera embeddings para busca semântica |

#### Busca & Integração
| Função | Descrição |
|:---|:---|
| `search-leads/` | Busca de leads (LinkedIn, email, etc.) |
| `whatsapp-templates/` | **CRUD completo** de templates WhatsApp via Meta Graph API v23.0. Router por action |

#### Sistema
| Função | Descrição |
|:---|:---|
| `process-action/` | Processador genérico de ações (command handler) |
| `get-integration-config/` | Busca config de integração por `user_id` + `integration_key` |
| `test-integration/` | Testa conectividade de uma integração |
| `check-achievements/` | Verifica e atribui conquistas ao usuário |
| `list-users/` | Lista usuários (admin only) |

#### `_shared/` — Código Compartilhado
Utilitários compartilhados entre Edge Functions (CORS headers, auth helpers, etc.)

---

### `supabase/migrations/` — 32 Migrations SQL

As migrations estão em ordem cronológica. As mais importantes para entender o schema:

| Migration | Conteúdo |
|:---|:---|
| `20260328142844_saas_multitenant.sql` | **Fundação SaaS**: tabela `profiles`, enums `user_role`/`plan_type`, RLS, trigger de signup, função `consume_credits()` |
| `20260328160000_onboarding_and_mp_prep.sql` | Onboarding e preparação de Marketplace |
| `20260328210000_admin_fix_and_rls.sql` | Correções de RLS e permissões admin |
| `20260328220000_rpc_handle_action.sql` | RPCs para processar ações |
| `20260401131730_*.sql` | Migration mais recente |

---

## `docs/` — Documentação de Planejamento

| Arquivo | Conteúdo |
|:---|:---|
| `PLAN-saas-multitenant.md` | Plano de refatoração para SaaS Multi-tenant. Define schema de `profiles`, RLS, fluxo de onboarding, middleware Zero Trust |
| `PLAN-whatsapp-templates.md` | Plano detalhado de integração WhatsApp Templates. Task breakdown, critérios de sucesso, estrutura de arquivos |

---

## Arquivos de Configuração

| Arquivo | O que configura |
|:---|:---|
| `vite.config.ts` | Bundler Vite: plugin React SWC, resolução de aliases (`@/` → `src/`) |
| `tailwind.config.ts` | Tema customizado: cores, animações, breakpoints, plugins |
| `components.json` | Configuração do shadcn/ui: estilo, path dos componentes, aliases |
| `Dockerfile` | Build multi-stage para deploy containerizado |
| `supabase/config.toml` | Configuração local do Supabase CLI |

---

## Notas Relacionadas

- [[Nexus - Index do Projeto]] — Visão geral e status
- [[Nexus - Arquitetura]] — Como as peças se conectam
- [[Nexus - Fluxo de Dados]] — Como a informação percorre o sistema
