---
tags:
  - nexus
  - fluxo-de-dados
  - dados
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 🔄 Nexus — Fluxo de Dados

> Esta nota documenta como a informação percorre o [[Nexus - Index do Projeto|Nexus]] em cada funcionalidade principal, desde a interação do usuário até a persistência e retorno visual.

---

## 1. Fluxo de Autenticação & Onboarding

```mermaid
sequenceDiagram
    actor U as Usuário
    participant FE as Frontend (React)
    participant Auth as Supabase Auth
    participant Trigger as DB Trigger
    participant DB as PostgreSQL

    U->>FE: Acessa /signup
    FE->>Auth: signUp(email, password)
    Auth-->>Trigger: AFTER INSERT on auth.users
    Trigger->>DB: INSERT profiles (role=client, plan=free, credits=0)
    Auth-->>FE: Session + JWT
    FE->>FE: Redireciona para /auth (verificar email)
    U->>U: Confirma email
    U->>FE: Acessa /
    FE->>FE: ProtectedRoute ✅ → VerifiedRoute ✅
    FE->>DB: SELECT profiles WHERE id=user.id
    DB-->>FE: { role, plan_type, credits }
    FE->>FE: Renderiza Chat (rota padrão)
```

**Detalhes importantes:**
- O trigger `handle_new_user()` no banco cria automaticamente o perfil
- Se o email é `rhyanpaablo@gmail.com`, o role é `admin` automaticamente
- O frontend escuta mudanças em `profiles` via **Realtime** para atualizar créditos/role em tempo real

---

## 2. Fluxo do Nexus Chat (IA com Streaming)

Este é o fluxo principal da aplicação — o Nexus Chat funciona como um **hub de comandos** onde a IA interpreta intenções e despacha ações.

```mermaid
sequenceDiagram
    actor U as Usuário
    participant FE as Chat.tsx
    participant Hook as useStreamingChat
    participant EF as Edge: nexus-assistant
    participant AI as OpenAI API
    participant DB as PostgreSQL

    U->>FE: Digita mensagem
    FE->>FE: Cria/seleciona conversa
    FE->>Hook: streamMessage({ message, history })

    Hook->>EF: POST /functions/v1/nexus-assistant
    Note over Hook,EF: Headers: Authorization Bearer + stream:true

    EF->>AI: Chat Completion (streaming)
    AI-->>EF: SSE chunks (delta.content)
    EF-->>Hook: data: {"choices":[{"delta":{"content":"..."}}]}
    Hook-->>FE: onDelta(chunk) → atualiza streamingMessage
    
    Note over FE: Rendering incremental do texto

    AI-->>EF: data: [DONE]
    EF-->>Hook: Stream finalizado
    Hook-->>FE: onDone()

    FE->>FE: Detecta marcadores especiais na resposta
    
    alt "[LINKEDIN_SEARCH_STARTED]"
        FE->>FE: parseLinkedInParams()
        FE->>FE: startLinkedInSearch(params)
    else "[EMAIL_SEARCH_STARTED]"
        FE->>FE: parseEmailSearchParams()
        FE->>FE: startEmailSearch(segmento, localizacao)
    end

    FE->>DB: UPDATE nexus_conversations (messages)
    DB-->>FE: Conversa persistida
```

**Marcadores especiais na resposta da IA:**

| Marcador | Ação Disparada |
|:---|:---|
| `[LINKEDIN_SEARCH_STARTED]` | Inicia prospecção no LinkedIn |
| `[SEARCH_PARAMS:{ json }]` | Parâmetros extraídos para a busca |
| `[EMAIL_SEARCH_STARTED]` | Inicia busca por e-mail |
| `[EMAIL_SEARCH_PARAMS:{ json }]` | Parâmetros da busca por email |

> [!important] Padrão Command via Chat
> A IA não é passiva — ela interpreta a intenção do usuário e **orquestra ações**. O chat é essencialmente um **command bus** natural language → action.

---

## 3. Fluxo de Gestão de Leads

```mermaid
flowchart TD
    A["Fontes de Entrada"] --> B
    
    subgraph B["Entrada de Leads"]
        B1["CSV Import (manual)"]
        B2["LinkedIn Scout (IA)"]
        B3["Email Search (IA)"]
        B4["Cadastro Manual (AddLeadDialog)"]
    end

    B --> C["Tabela `leads` no PostgreSQL"]
    
    C --> D["Visualizações"]
    
    subgraph D["Views de Leads"]
        D1["LeadsTable (tabela rica)"]
        D2["LeadsKanban (kanban board)"]
        D3["LeadsListView (lista simples)"]
        D4["LeadsTableView (tabela condensada)"]
    end

    D --> E["Ações sobre o Lead"]
    
    subgraph E["Ações"]
        E1["Qualificar (QualificationModal)"]
        E2["Atribuir Closer (AssignCloserDialog)"]
        E3["Análise IA 360° (analyze-lead-360)"]
        E4["Chat com Lead (LeadChat)"]
        E5["Análise de Website (analyze-website)"]
        E6["Sugestões de Mensagem (generate-message-suggestions)"]
    end

    E1 --> F["Status atualizado no DB"]
    E3 --> G["Edge Function → OpenAI → Histórico salvo"]
    E4 --> H["Mensagens persistidas + Templates favoritos"]
```

**Ciclo de vida de um lead:**
```
Novo → Primeiro Contato → Qualificado → Proposta → Negociação → Fechado (Ganho/Perdido)
```

---

## 4. Fluxo de Créditos (Zero Trust)

```mermaid
sequenceDiagram
    actor U as Usuário
    participant FE as CreditGate
    participant Hook as useUserRole
    participant DB as profiles
    participant EF as Edge Function
    participant RPC as consume_credits()

    U->>FE: Clica em ação (ex: "Analisar Lead")
    FE->>Hook: Verifica isAdmin e credits
    
    alt É Admin
        FE->>EF: Executa diretamente (bypass)
    else Tem créditos >= custo
        FE->>EF: Executa ação
        EF->>RPC: consume_credits(user_id, 1)
        RPC->>DB: UPDATE profiles SET credits = credits - 1
        RPC-->>EF: true
        EF-->>FE: Resultado da ação
        Note over FE: React Query invalida cache → saldo atualiza
    else Sem créditos
        FE->>FE: Toast "Créditos insuficientes"
        FE->>FE: CTA → redireciona para /pricing
    end
```

**Regras de crédito:**
- **Ações de leitura** (listar leads, ver dashboard, ler histórico): **0 créditos**
- **Ações de mutação** (enviar mensagem, analisar com IA, gerar conteúdo): **1 crédito**
- **Admin**: bypass total (sem consumo de créditos)

---

## 5. Fluxo de Prospecção LinkedIn

```mermaid
sequenceDiagram
    actor U as Usuário
    participant Chat as Nexus Chat
    participant AI as nexus-assistant
    participant LS as useLinkedInSearch
    participant EF as analyze-linkedin-lead
    participant DB as PostgreSQL

    U->>Chat: "Busque decisores de TI em São Paulo"
    Chat->>AI: Envia mensagem + histórico
    AI-->>Chat: "[LINKEDIN_SEARCH_STARTED][SEARCH_PARAMS:{...}]"
    Chat->>LS: startLinkedInSearch(params)
    LS->>EF: search-leads(segmento, localização)
    EF-->>LS: Lista de perfis encontrados
    LS-->>Chat: Resultados exibidos (LinkedInResultsCard)

    Note over U: Usuário seleciona perfil para análise

    U->>Chat: Analisa perfil específico
    Chat->>EF: analyze-linkedin-lead(profileData)
    EF-->>DB: Salva oportunidade como pré-lead
    EF-->>Chat: Análise detalhada do perfil

    Note over U: Conversão para lead qualificado

    U->>Chat: Converte para lead
    Chat->>DB: INSERT leads (dados do perfil)
```

---

## 6. Fluxo de WhatsApp Templates

```mermaid
flowchart LR
    A["Admin abre /whatsapp-templates"] --> B["useWhatsappTemplates()"]
    B --> C["supabase.functions.invoke('whatsapp-templates')"]
    C --> D["Edge Function"]
    
    D --> E{"Action?"}
    E -- "list/sync" --> F["GET Meta API → Lista templates"]
    E -- "create" --> G["POST Meta API → Cria template"]
    E -- "update" --> H["POST Meta API → Atualiza componentes"]
    E -- "delete" --> I["DELETE Meta API → Remove template"]
    
    F & G & H & I --> J["Response normalizada"]
    J --> K["React Query cache atualizado"]
    K --> L["UI re-renderiza (TemplateListTable)"]
```

**Fluxo de credenciais:**
1. Frontend chama Edge Function
2. Edge Function busca `integration_configs WHERE user_id AND integration_key='whatsapp'`
3. Extrai `waba_id` + `access_token` do campo JSON `config`
4. Usa credenciais para chamar a Meta Graph API

---

## 7. Fluxo de Estado (State Management)

```mermaid
graph TD
    subgraph "Server State (React Query)"
        A["queryKey: dashboard-leads"]
        B["queryKey: nexus-conversations"]
        C["queryKey: whatsapp-templates"]
        D["Invalidação automática após mutations"]
    end

    subgraph "Client State (React useState)"
        E["selectedConversationId"]
        F["currentMessages"]
        G["streamingMessage"]
        H["navCollapsed / chatSidebarOpen"]
    end

    subgraph "Auth State"
        I["Supabase Session (localStorage)"]
        J["UserProfile (useUserRole)"]
        K["Realtime channel: profiles UPDATE"]
    end

    D --> A & B & C
    K --> J
    I --> J
```

**Estratégia de cache:**
- `staleTime: 5 min` para dados do dashboard
- `gcTime: 10 min` para garbage collection do React Query
- Invalidação imediata após qualquer mutation (create, update, delete)

---

## 8. Fluxo Realtime — Atualização de Perfil

```mermaid
sequenceDiagram
    participant Admin as Admin (Painel)
    participant DB as PostgreSQL
    participant RT as Supabase Realtime
    participant FE as Frontend (useUserRole)

    Admin->>DB: UPDATE profiles SET credits=100 WHERE id=user_x
    DB-->>RT: postgres_changes (UPDATE, profiles)
    RT-->>FE: Channel notification
    FE->>FE: fetchUserProfile() automaticamente
    FE->>FE: UI atualiza saldo em tempo real
```

---

## Notas Relacionadas

- [[Nexus - Arquitetura]] — Diagramas de camadas e conexões
- [[Nexus - Mapeamento de Arquivos]] — Onde cada fluxo vive no código
- [[Nexus - Index do Projeto]] — Visão geral do projeto
