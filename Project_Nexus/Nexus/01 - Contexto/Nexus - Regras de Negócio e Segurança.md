---
tags:
  - nexus
  - seguranca
  - logica
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 🛡️ Nexus — Regras de Negócio e Segurança

> **Conceito Fundamental:**
> O Nexus opera sob um modelo **Zero Trust** onde o backend é o árbitro final de permissões. O frontend fornece UX de bloqueio visual, mas a validação real acontece nas Edge Functions e RPCs do banco.

---

## 💳 Economia de Créditos

### Como funciona

O sistema usa a coluna `credits` na tabela `profiles` como saldo transacional.

| Tipo de Ação | Custo | Exemplo |
|:---|:---:|:---|
| **Leitura** | 0 | Ver dashboard, listar leads, ler histórico de conversas |
| **Mutação/IA** | 1+ | Analisar lead com IA, gerar caption, buscar email, enviar template |

### Fluxo de consumo (backend)

1. Edge Function recebe a requisição
2. Chama `consume_credits(user_id, amount)` — RPC `SECURITY DEFINER` (bypassa RLS)
3. Se `is_admin(user_id)` → retorna `true` sem debitar (bypass total)
4. Se `credits >= amount` → debita e retorna `true`
5. Se saldo insuficiente → retorna `false` (ação bloqueada)

### Fluxo de consumo (frontend)

O hook `useUserCredits` expõe:
- `deduct_credits(p_user_id, p_amount, p_action_name)` — deduz via RPC e registra a ação
- `hasEnoughCredits(amount)` — check local para UX imediata
- `refreshBalance()` — força re-fetch do saldo

---

## 🚪 Sistema de Gates (Portões Visuais)

> Definidos em `src/components/auth/PlanMiddleware.tsx`.

### CreditGate

Envolve botões/ações e verifica créditos **antes** de executar.

```
<CreditGate cost={1} actionLabel="Analisar Lead" onProceed={handleAnalyze}>
  <Button>Analisar</Button>
</CreditGate>
```

**Comportamento:**
- Admin → executa direto (bypass)
- Créditos suficientes → executa `onProceed`
- Sem créditos → Toast de erro + CTA "Fazer Upgrade" → redireciona para `/pricing`

### PlanGate

Bloqueia funcionalidades inteiras baseado no plano do usuário.

**Hierarquia de planos:**
```
free (0) → basic (1) → pro/premium (2) → enterprise (3)
```

```
<PlanGate requiredPlan="basic" featureName="Pipeline de Vendas">
  <PipelineComponent />
</PlanGate>
```

- Admin → sempre vê o conteúdo
- Plano suficiente → renderiza children
- Plano insuficiente → mostra `UpgradeBanner` com CTA para `/pricing`

### AdminRoute (Route-Level Guard)

Protege rotas inteiras. Componente em `src/components/auth/AdminRoute.tsx`.
- Lê `isAdmin` do hook `useUserRole()`
- Se não é admin → redireciona para `/`
- Rotas protegidas: `/posts`, `/logs`, `/whatsapp-templates`

---

## 🔐 Segurança de Dados (RLS)

### Isolamento Multi-Tenant

> ⚠️ O Nexus **NÃO** usa `company_id`. O isolamento é por `user_id` (cada usuário é um tenant independente).

**Políticas RLS na tabela `profiles`:**

| Policy | Nível | Condição |
|:---|:---|:---|
| Admins can view all profiles | SELECT | `is_admin(auth.uid())` |
| Admins can update all profiles | UPDATE | `is_admin(auth.uid())` |
| Users can view own profile | SELECT | `id = auth.uid()` |
| Users can update own profile | UPDATE | `id = auth.uid()` |

**Políticas RLS nas tabelas de negócio (ex: `integration_configs`):**

| Policy | Condição |
|:---|:---|
| Admins have full access | `is_admin(auth.uid())` — ALL operations |
| Users see own data | `user_id = auth.uid()` |

### Função helper `is_admin()`

```sql
CREATE OR REPLACE FUNCTION public.is_admin(user_id UUID) RETURNS BOOLEAN AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM public.profiles 
    WHERE id = user_id AND role = 'admin'::public.user_role
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## 🔄 Fluxo de Processamento de IA (Edge Functions)

1. **Entrada**: O usuário solicita uma análise (ex: "Analisar Website do Lead") via Chat ou botão direto
2. **Autenticação**: A Edge Function extrai o JWT do header `Authorization: Bearer`
3. **Validação de créditos**: Chama `consume_credits(user_id, 1)` — bloqueia se saldo insuficiente
4. **Busca de config**: Se necessário, busca credenciais em `integration_configs` (ex: WhatsApp → `waba_id` + `access_token`)
5. **Processamento**: Chama OpenAI/Tavily com os dados
6. **Resultado**: Salva no banco (ex: `lead_ai_analyses`) e retorna ao frontend
7. **Cache**: React Query invalida a query afetada → UI atualiza automaticamente

---

## 👤 Roles e Enums

### Enums no banco (PostgreSQL)

| Enum | Valores |
|:---|:---|
| `user_role` | `admin`, `client` |
| `plan_type` | `free`, `basic`, `premium`, `pro`, `enterprise` |
| `app_role` | `admin`, `staff`, `user` (tabela `user_roles` separada) |

> [!note] Dois sistemas de roles coexistem
> - `profiles.role` (enum `user_role`: admin/client) → usado pelo `useUserRole` hook
> - `user_roles.role` (enum `app_role`: admin/staff/user) → tabela separada, verificada por `has_role()` RPC

### Auto-criação de perfil

Trigger `handle_new_user()` no banco:
- Disparado após `INSERT` em `auth.users`
- Cria `profiles` com `role='client'`, `plan_type='free'`, `credits=0`
- **Exceção**: email `rhyanpaablo@gmail.com` recebe `role='admin'`

---

## Notas Relacionadas

- [[Nexus - Arquitetura]] — Diagramas de camadas e middleware visual
- [[Nexus - Modelo de Dados e Banco]] — Schema real das tabelas
- [[Nexus - Fluxo de Dados]] — Fluxos detalhados com diagramas de sequência
- [[Nexus - Index do Projeto]] — Hub central
