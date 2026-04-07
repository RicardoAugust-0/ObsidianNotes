---
tags:
  - nexus
  - stack
  - arquitetura
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 🛠️ Nexus — Stack Tecnológica

> **Resumo da Infraestrutura:**
> O Nexus é um projeto Modern Full-Stack (Serverless), desacoplado entre Frontend (React SPA) e Backend (Supabase BaaS + Edge Functions).

---

### Frontend (User Interface)

| Tecnologia | Versão | Uso |
|:---|:---|:---|
| [React](https://react.dev/) | 18.3 | Biblioteca de UI |
| [Vite](https://vitejs.dev/) | 7.3 | Bundler e dev server (com plugin SWC) |
| [TypeScript](https://www.typescriptlang.org/) | 5.8 | Tipagem estática em todo o projeto |
| [Tailwind CSS](https://tailwindcss.com/) | 3.4 | Estilização utility-first |
| [shadcn/ui](https://ui.shadcn.com/) + Radix | — | Componentes acessíveis e modulares (~30 primitivos Radix) |
| [TanStack React Query](https://tanstack.com/query/latest) | 5.83 | Cache, fetching, mutations e invalidação de dados |
| [React Router DOM](https://reactrouter.com/) | 6.30 | Roteamento SPA com 17 rotas |
| [React Hook Form](https://react-hook-form.com/) | 7.61 | Gestão de formulários |
| [Zod](https://zod.dev/) | 3.25 | Validação de schemas (templates WhatsApp, etc.) |
| [Recharts](https://recharts.org/) | 2.15 | Gráficos e visualização de dados |
| [Lucide React](https://lucide.dev/) | 0.462 | Ícones |
| [next-themes](https://github.com/pacocoursey/next-themes) | 0.3 | Toggle dark/light mode |
| [Sonner](https://sonner.emilkowal.dev/) | 1.7 | Notificações toast |
| [date-fns](https://date-fns.org/) | 3.6 | Formatação de datas |
| [react-markdown](https://github.com/remarkjs/react-markdown) | 10.1 | Renderização de markdown (chat IA) |
| [xlsx](https://sheetjs.com/) | 0.18 | Import/export de planilhas CSV/Excel |
| [cmdk](https://cmdk.paco.me/) | 1.1 | Command palette |

> [!tip] Padrão de State Management
> O Nexus **NÃO** usa Redux, Zustand ou Context API para estado global. O padrão é:
> - **Server state**: TanStack React Query (cache + invalidação de queries)
> - **Auth/Profile state**: Custom hook `useUserRole` com `useState` + Supabase Realtime
> - **Local UI state**: `useState` por componente (ex: `navCollapsed`, `chatSidebarOpen`)

---

### Backend (Backend-as-a-Service)

| Tecnologia | Uso |
|:---|:---|
| [Supabase](https://supabase.com/) | BaaS completo (Auth, DB, Functions, Storage, Realtime) |
| **PostgreSQL** | Banco relacional com RLS nativo e funções RPC |
| **Supabase Auth** | Email/Senha com verificação. JWT. Sessão persistente em `localStorage` |
| **Supabase Edge Functions (Deno)** | 20 funções serverless (TypeScript/Deno) |
| **Supabase Realtime** | Subscrições `postgres_changes` para updates em `profiles` |
| **Supabase Storage** | Upload de logos, avatares e documentos |

**RPCs no banco (functions SQL):**

| Função | Assinatura | Retorno |
|:---|:---|:---|
| `consume_credits` | `(user_id, amount)` | `boolean` |
| `deduct_credits` | `(p_user_id, p_amount, p_action_name)` | `boolean` |
| `add_credits` | `(p_user_id, p_amount)` | `number` |
| `get_user_credits` | `(p_user_id)` | `number` |
| `is_admin` | `(user_id)` | `boolean` |
| `is_admin_or_staff` | `()` | `boolean` |
| `has_role` | `(_user_id, _role)` | `boolean` |
| `handle_user_action` | `(action, payload)` | `json` |
| `match_documents` | `(query_embedding, match_count, filter)` | `array` |
| `require_billing_or_credits` | `(action_cost)` | `boolean` |

---

### Inteligência Artificial & APIs Externas

| Serviço | Uso no Nexus | Conexão |
|:---|:---|:---|
| [OpenAI API](https://openai.com/api/) | Nexus Chat (streaming SSE), análises 360°, geração de captions, imagens, sugestões de mensagem | Edge Function → env vars |
| [Tavily AI](https://tavily.com/) | Pesquisa web em tempo real para enriquecimento de leads e análise de websites | Edge Function → env vars |
| [Meta Graph API v23.0](https://developers.facebook.com/) | WhatsApp Business Templates (CRUD) | Edge Function → `integration_configs` (por tenant) |

> [!note] Pagamentos (Futuro / Parcial)
> O schema tem colunas preparadas para **Stripe** (`stripe_customer_id`, `subscription_status`) e **MercadoPago** (`mp_user_id`).
> A tabela `subscriptions` já existe com campos `plan_id`, `preference_id`, `payment_method`, `external_reference`.

---

### DevOps & Ferramentas

| Ferramenta | Detalhe |
|:---|:---|
| **Linguagem** | TypeScript (frontend + edge functions) |
| **Package Manager** | `npm` (com `bun.lock` também presente) |
| **Versionamento** | Git (GitHub) |
| **Deployment** | [Lovable.dev](https://lovable.dev) (Frontend + auto-deploy) + Supabase (Edge Functions e DB) |
| **Linting** | ESLint 9 + eslint-plugin-react-hooks + react-refresh |
| **Container** | Dockerfile presente para deploy containerizado alternativo |

---

### Notas Relacionadas

- [[Nexus - Index do Projeto]] — Hub central de navegação
- [[Nexus - Visão Geral e Identidade]] — Proposta de valor e diferenciais
- [[Nexus - Arquitetura]] — Diagramas de camadas
