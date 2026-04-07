# 🎨 Nexus - Revamp Frontend (Brainstorm)

Este documento detalha o planejamento para a reestruturação completa da interface do Nexus CRM. O objetivo é transformar um "boilerplate genérico" em uma plataforma **Premium, Data-Driven e com Estética 2025**.

---

## 🏗️ Nova Arquitetura de Frontend

Atualmente, o projeto sofre de "Componentes Monolíticos" (ex: `Logs.tsx` e `Opportunities.tsx` com +500 linhas). A nova estrutura seguirá o padrão **Atomic Design** com separação rigorosa de lógica.

### 1. Separação de Interesses (SOC)
- **Views (Pages):** Apenas montam o layout. Zero chamadas diretas ao Supabase.
- **Custom Hooks (`/hooks/api`):** Toda a lógica de `useQuery` e `useMutation` deve morar aqui.
- **Services (`/lib/api`):** Funções puras que interagem com o banco ou Edge Functions.
- **Components:** Divididos em `atoms`, `molecules` e `organisms`.

### 2. Design System: "Nexus Glass Pro"
Substituiremos o visual padrão do Shadcn por uma estética personalizada:
- **Fundo:** Dark Mesh Gradients (azul profundo para preto).
- **Cards:** Glassmorphism Real (`backdrop-filter: blur(12px)`) com bordas em gradiente sutil.
- **Tipografia:** Transição para **Inter** ou **Outfit** com pesos variados para melhor hierarquia.
- **Micro-interações:** Framer Motion para transições de página e estados de hover nos cards.

---

## 🔍 Auditoria Visual e Diagnóstico (Screenshots Reais)

Aqui está o estado atual coletado na auditoria de 03/04/2026:

### 📱 Dashboard & Métricas
![Dashboard](dashboard.png)
> [!CAUTION] **Problema:** A seção "Prioridade Inteligente" está vazia ou em estado de skeleton eterno.
> **Melhoria:** Implementar Empty States ilustrativos e animações de entrada para os cards de KPI.

### 👥 Gestão de Leads
![Leads](leads.png)
> [!WARNING] **Problema:** Layout muito utilitário. Instruções de importação ocupam muito espaço útil.
> **Melhoria:** Mover instruções para um "Help Center" ou Popover. Focar na tabela de leads com filtros laterais colapsáveis.

### 🎯 Prospecção (Opportunities)
![Opportunities](opportunities.png)
> [!IMPORTANT] **Dissonância de UX:** Esta página age como uma ferramenta de busca (Scout) mas está sob a rota de "Oportunidades".
> **Melhoria:** Renomear para "Nexus Scout" e criar uma página de Pipeline (Kanban) real para as Oportunidades.

### 📑 Logs & Analytics
![Logs](logs.png)
> [!TIP] **Ponto Forte:** Esta é a página mais rica visualmente. Devemos usar o padrão de gráficos e métricas daqui como base para o resto do app.

---

## 🛠️ Roadmap de Implementação (Fases)

### Fase 1: Fundação (Design System) - ✅ CONCLUÍDA
- [x] Atualizar `/src/index.css` com as novas variáveis de Glassmorphism e utilitários modernos (mesh gradients, text gradients, shimmers).
- [x] `MainLayout.tsx` — Adicionado mesh gradient responsivo dark/light (`bg-mesh-dark bg-mesh-light`). A Sidebar moderna já estava funcional.
- [x] Padronizar `EmptyState.tsx` — Criado componente ilustrativo com SVGs modulares (`ai`, `search`, `data`, `default`) e aplicado na `SmartPriorityPanel`.
- [x] Criado `PageHeader` padronizado.
- [x] `WelcomeHeader` refatorada — Agora extrai o nome corretamente da tabela `profiles` ("full_name") com fallback apropriado.
- [x] Melhorada a UX na página "Agente Outbound" (Leads) ocultando as instruções massivas de tabela atrás de um `Collapsible`.

### Fase 2: Componentização & Hooks - 🔄 EM ANDAMENTO
- [ ] Extrair lógica de `Logs.tsx` para `useApiLogs()`.
- [ ] Criar subcomponentes para a `Opportunities.tsx` (ScoutForm, ScoutResults, ScoutStats).

### Fase 3: UX & Novas Features
- [x] Implementar o **Kanban de Oportunidades** — Verificado: `LeadsKanban.tsx` já existe e está perfeitamente integrado no `LeadsTable.tsx` com drag-and-drop.
- [ ] Refatorar o **Chat do Nexus** para suportar streaming de resposta mais fluido.
- [ ] Tradução completa para PT-BR (corrigir termos em inglês remanescentes).

---

## 🔗 Links Internos
- [[Nexus - Arquitetura]]
- [[Nexus - Melhorias e Roadmap]]
- [[Nexus - Index do Projeto]]
