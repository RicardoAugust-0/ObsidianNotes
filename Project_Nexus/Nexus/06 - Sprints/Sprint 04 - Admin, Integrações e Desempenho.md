---
tags:
  - nexus
  - sprint
  - historico
created: 2026-04-04
status: ✅ Concluída
parent: "[[Nexus - Index do Projeto]]"
---

# 🏃 Sprint 04 — Revamp Performance, Segurança Admin & UX de Integrações

> **Objetivo:** Reformulação completa do dashboard de "**Meu Desempenho**" para o modelo Solo-SaaS, correção crítica garantindo Isolamento Multi-Tenant e resolvendo a Recursão RLS do Painel Admin, além da otimização do fluxo (UX/UI) obrigatório de Integrações.

---

## 📋 Escopo

| Item | Tipo | Status |
|:---|:---:|:---:|
| Refatoração Dashboard "Meu Desempenho" | UI/UX | ✅ |
| Segurança Multi-Tenant (RankingsChart) | SEC | ✅ |
| Correção RLS Recursivo no Painel Admin | SEC/DB | ✅ |
| Testes Bateria (Produção e APIs externas) | QA/QA | ✅ |
| Validação de Integrações no Scout | UX | ✅ |
| Refatoração Modal de Integrações | UX/DX | ✅ |

---

## ✅ Entregáveis

### 1. Dashboard "Meu Desempenho" Totalmente Revampado
A antiga página de performance foi transformada em um verdadeiro Dashboard Solo-SaaS contendo:
- **`PerformanceKPIs`:** Reorganizado para 4 colunas no Desktop, acrescentado cálculo dinâmico de pipeline total (`value`).
- **`LeadsEvolutionChart`:** Área visual configurada para renderizar grid e eixos mesmo quando os dados são 0.
- **`QuickWinsPanel`:** Novo balcão de visualização de Hot Leads (com `close_probability` >= 50%) com formatação de cores e alertas integrados, focando em Fechamento.
- **Goals Panel Hacking:** Atualização no controle para **Hard-Delete** (no `DELETE` SQL) corrigindo um bug em que exclusões (soft-deletes) retornavam ao reiniciar a página.
- **`TeamFunnelChart` / `RankingsChart`:** Ajustado todos os mapeamentos de status do estágio Inglês antigo (new/contacted) para os equivalentes em português ("Novo", "Qualificado", etc). *Privacidade corrigida:* Listagem de usuários no ranking removida do `list-users` Global para a tabela `profiles` restrita (evitando vazamentos entre Tenants).

---

### 2. Painel Admin: Correção RLS Recursion (Segurança DB)
- **Problema:** Um segundo admin (Ricardo) não enxergava outros usuários administradores ou clientes além de si próprio, e no limite isso poderia derrubar chamadas da tabela. A política RLS antiga gerava um "Loop infinito" (RLS Recursion) porque a função `is_admin()` lia `profiles` ao mesmo tempo que estava restrita pela própria política de `profiles`.
- **Solução (`SQL via Supabase Editor`):**
  Criamos uma nova checagem direta blindada (bypassing the evaluation layer do profiles):
  ```sql
  CREATE OR REPLACE FUNCTION public.is_admin_check() RETURNS BOOLEAN
  LANGUAGE sql SECURITY DEFINER SET search_path = public AS $$
    SELECT EXISTS (SELECT 1 FROM profiles WHERE id = auth.uid() AND role = 'admin'::user_role);
  $$;
  ALTER FUNCTION public.is_admin_check() OWNER TO postgres;
  ```
  Resultando na visão clara (e controlada de verdade) do `Admin.tsx` a todos que tiverem o Role "admin" preenchido, garantindo que o painel mostre o time Rhyan/Ricardo 100%.

---

### 3. Integrações & Scopes da Aplicação (UX e Regras de Negócio)
- **Problema de UX:** Se a aba Oportunidades for utilizada sem chaves configuradas, o usuário receberia erros complexos de webhooks `500` e a tela ficava morta girando carregamento. Outro ponto era um confusão de "Ativar" switches sem salvar manualmente.
- **UX de Bloqueio em Oportunidades (`Opportunities.tsx` e `LinkedInScout.tsx`):**
  Agregado o hook `useIntegrationConfigs`.
  A tela agora exige (bloqueio por aviso em toast) as chaves ativas do **"Google API"** (para buscar no maps) e do **"Apollo.io"** (para o LinkedIn).
- **UX do Modal de Configuração (`IntegrationConfigDialog.tsx`):**
  Criado processo dinâmico e inteligente unificando os botões. Agora temos 1 botão `Conectar`. 
  - Ao clicar, ele invoca a Edge Function interna `test-integration`.
  - Em caso de SUCESSO (conectado de fato), ele já Salva a credencial por trás, torna o Switch ativo (invisível em frontend) e sobe um belo Toast confirmando!
  - Para as integradas, surgiu um botão visível e óbvio de cor vermelha `Desconectar`.
- **Textos de Menus:** Deixamos as descrições da pág Integrações mais detalhadas sobre O QUE elas liberam (`Necessário para o Nexus Scout...`).

---

### 4. Insights de Produção Obtidos no "Teste Bairro" (Script MJS)
- Realizado script conectando na Base em Prod para bater endpoints simulando login dos admins.
- Foi atestado o recebimento do código de repasse **status 403 Google Search API.** Assim, o time tomou ciência de que o motor de oportunidade real (API nativa Google via painel) precisa de uma substituição urgente de Access Key (o Google bloqueou/esgotou por limitação).

---
## 🔗 Referências e PRs/Arquivos Toucados
- `src/pages/Opportunities.tsx`
- `src/components/opportunities/LinkedInScout.tsx`
- `src/components/integrations/IntegrationConfigDialog.tsx`
- `src/pages/Integrations.tsx`
- `src/components/performance/*.tsx`
- Edge Function `search-leads` (verificada)
