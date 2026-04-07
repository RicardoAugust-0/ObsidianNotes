---
tags:
  - nexus
  - padroes
  - convencoes
  - clean-code
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 🧩 Nexus — Padrões e Convenções de Código

> Guia de referência dos padrões recorrentes identificados no código do [[Nexus - Index do Projeto|Nexus]]. Útil para manter consistência ao contribuir ou estender o projeto.

---

## 1. Design Patterns Identificados

### Command Bus via Chat
O Nexus Chat não é apenas um chatbot — é um **orquestrador de ações via linguagem natural**.

```
Usuário: "Busque decisores de TI em São Paulo"
IA → Emite: [LINKEDIN_SEARCH_STARTED][SEARCH_PARAMS:{...}]
Frontend → Detecta marcador → Dispara useLinkedInSearch
```

**Pattern:** Natural Language → LLM → Marcador → Frontend Action Handler

### Router Pattern (Edge Functions)
A função `whatsapp-templates` implementa um **action router** — uma única Edge Function que roteia para múltiplos handlers via campo `action`:

```typescript
// Input: { action: "list" | "create" | "update" | "delete", ...params }
switch (action) {
  case "list": return handleList(wabaId, token, params);
  case "create": return handleCreate(wabaId, token, params);
  // ...
}
```

### BFF (Backend for Frontend)
A função `process-action` atua como **BFF** com middleware chain:
```
Auth → Email Verified → Has Credits → Consume Credits → Execute Action → Return Result
```

### Gate Pattern (Visual Middleware)
Componentes wrapper que controlam renderização baseados em estado:

```jsx
// CreditGate → verifica créditos antes de executar ação
<CreditGate cost={1} actionLabel="Analisar" onProceed={fn}>
  <Button>Analisar Lead</Button>
</CreditGate>

// PlanGate → verifica plano antes de renderizar conteúdo
<PlanGate requiredPlan="basic" featureName="Pipeline">
  <PipelineComponent />
</PlanGate>
```

### Lazy Loading (Code Splitting)
O Dashboard (`Index.tsx`) usa `React.lazy()` para todos os gráficos pesados:

```typescript
const ConversionFunnelChart = lazy(() => 
  import("@/components/dashboard/ConversionFunnelChart")
    .then(m => ({ default: m.ConversionFunnelChart }))
);
```

### Dual Provider com Fallback (IA)
Todas as funções de IA tentam um provedor primário (Lovable/Gemini) e, se falhar, caem para fallback (OpenAI):

```typescript
const useLovable = !!lovableApiKey;
const apiUrl = useLovable 
  ? 'https://ai.gateway.lovable.dev/v1/chat/completions'
  : 'https://api.openai.com/v1/chat/completions';
```

---

## 2. Padrões de Hooks

### Padrão Dominante: Edge Function → React Query → Cache

```typescript
// Exemplo: useWhatsappTemplates
const { data, isLoading } = useQuery({
  queryKey: ['whatsapp-templates'],
  queryFn: () => supabase.functions.invoke('whatsapp-templates', { 
    body: { action: 'list' } 
  }),
});

const createMutation = useMutation({
  mutationFn: (data) => supabase.functions.invoke('whatsapp-templates', { 
    body: { action: 'create', ...data } 
  }),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['whatsapp-templates'] });
    toast.success("Template criado!");
  },
});
```

### Padrão de State + Realtime

```typescript
// Exemplo: useUserRole
const [userRole, setUserRole] = useState(null);

useEffect(() => {
  fetchUserProfile(); // Busca inicial
  
  const channel = supabase
    .channel('profile-changes')
    .on('postgres_changes', { 
      event: 'UPDATE', 
      schema: 'public', 
      table: 'profiles',
      filter: `id=eq.${userId}` 
    }, () => fetchUserProfile()) // Re-fetch automático
    .subscribe();
    
  return () => { supabase.removeChannel(channel); };
}, [userId]);
```

### Padrão de Streaming SSE

```typescript
// Exemplo: useStreamingChat
const response = await fetch(url, { method: 'POST', body, headers });
const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const chunk = decoder.decode(value);
  // Parse SSE: "data: {...}\n\n"
  onDelta(parsedContent);
}
```

---

## 3. Convenções de Nomenclatura

### Arquivos e Diretórios

| Tipo | Convenção | Exemplo |
|:---|:---|:---|
| Componentes React | PascalCase | `ChatConversation.tsx` |
| Hooks | camelCase com prefixo `use` | `useStreamingChat.ts` |
| Páginas | PascalCase | `LeadDetail.tsx` |
| Edge Functions | kebab-case (diretórios) | `analyze-lead-360/` |
| Migrations SQL | Timestamp + UUID/nome | `20260328142844_saas_multitenant.sql` |

### Colunas do Banco

| Convenção | Exemplo |
|:---|:---|
| snake_case | `user_id`, `plan_type`, `created_at` |
| Português para campos de negócio | `segmento`, `cargo_lead`, `oportunidade` |
| Inglês para campos técnicos | `status`, `email`, `assigned_to` |

### Estado na URL

| Padrão | Exemplo |
|:---|:---|
| Rotas em inglês, kebab-case | `/whatsapp-templates`, `/leads/:id` |
| Params em camelCase | Sem query params — estado em React state |

---

## 4. Padrões de Componentes

### Estrutura de Página Padrão

```tsx
const ExamplePage = () => {
  // 1. Hooks (queries, mutations, state)
  const { data, isLoading } = useQuery({...});
  
  // 2. Event handlers
  const handleAction = useCallback(() => {...}, []);
  
  // 3. Render
  return (
    <MainLayout>
      <div className="space-y-8">
        <Header />
        <ContentGrid />
      </div>
    </MainLayout>
  );
};
```

### Padrão de Dialog (CRUD)

```
[Página] → [Botão Adicionar] → [AddXxxDialog] → mutation → invalidateQueries → toast
[Página] → [Linha na tabela] → [EditXxxDialog] → mutation → invalidateQueries → toast
[Página] → [Botão Excluir] → [DeleteXxxDialog] → mutation → invalidateQueries → toast
```

### Tamanho de Componentes (Referencial)

| Componente | Tamanho | Complexidade |
|:---|:---:|:---|
| `LeadsTable.tsx` | 42KB | Tabela rica com filtros, bulk actions, import CSV |
| `LeadChat.tsx` | 36KB | Chat direto com lead, sugestões IA, templates |
| `LeadDetailsSheet.tsx` | 36KB | Sheet lateral com detalhes completos |
| `nexus-assistant/index.ts` | 36KB | Motor principal de IA (751 linhas) |
| `LinkedInScout.tsx` | 23KB | Interface de prospecção completa |
| `TemplateCreateDialog.tsx` | 21KB | Formulário complexo com variáveis dinâmicas |

---

## 5. Padrão de Cache (React Query)

| Configuração | Valor | Onde |
|:---|:---|:---|
| `staleTime` | 5 minutos | Dashboard (`dashboard-leads`) |
| `gcTime` | 10 minutos | Dashboard |
| Invalidação | Imediata após mutation | Todos os hooks com mutations |
| `queryKey` | Array descritivo | `['dashboard-leads']`, `['nexus-conversations']`, `['whatsapp-templates']` |

---

## 6. Padrão de Error Handling

### Edge Functions
```typescript
try {
  // ... lógica
} catch (error) {
  console.error('[NOME-FUNCAO] Error:', error);
  logData = { ...logData, status: 'error', error_message: error.message };
  await supabase.from('api_logs').insert([logData]);
  return new Response(JSON.stringify({ error: error.message }), { status: 500 });
}
```

### Frontend (Hooks)
```typescript
onError: (error) => {
  toast.error("Erro ao executar ação");
  console.error(error);
}
```

---

## 7. Variáveis de Ambiente

| Variável | Onde | Uso |
|:---|:---|:---|
| `SUPABASE_URL` | Edge Functions | URL do projeto Supabase |
| `SUPABASE_SERVICE_ROLE_KEY` | Edge Functions | Service role (bypass RLS) |
| `SUPABASE_ANON_KEY` | Edge Functions + Frontend | Anon key (respects RLS) |
| `OPENAI_API_KEY` | Edge Functions | Fallback para LLM |
| `LOVABLE_API_KEY` | Edge Functions | LLM primário (Gemini via Gateway) |
| `SERPER_API_KEY` | Edge Functions | Serper.dev (busca de leads) |

> [!note] Multi-tenant API Keys
> Tenants podem sobrescrever `OPENAI_API_KEY` e `SERPER_API_KEY` via `integration_configs`. O sistema sempre tenta a chave do tenant primeiro, depois cai para env vars.

---

## Notas Relacionadas

- [[Nexus - Arquitetura]] — Diagramas de onde os padrões são aplicados
- [[Nexus - Edge Functions]] — Implementações dos patterns no backend
- [[Nexus - Mapeamento de Arquivos]] — Localização de cada arquivo
- [[Nexus - Glossário]] — Definição de cada termo usado aqui
