---
tags:
  - nexus
  - glossario
  - referencia
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 📖 Nexus — Glossário de Termos

> Dicionário rápido de todos os termos, siglas e conceitos usados no projeto [[Nexus - Index do Projeto|Nexus]]. Use `Ctrl+F` para buscar.

---

## A

**Admin** — Papel de superusuário (`role: 'admin'` em `profiles`). Tem bypass total de créditos e acesso a todas as rotas. O CEO (`rhyanpaablo@gmail.com`) é admin automático. → [[Nexus - Regras de Negócio e Segurança]]

**AdminRoute** — Componente guard que protege rotas como `/posts`, `/logs`, `/whatsapp-templates`. Redireciona não-admins para `/`. → [[Nexus - Arquitetura#Guardas de Rota (Route Guards)]]

**API Logs** — Tabela `api_logs` que registra cada chamada de Edge Function: duração, modelo, tokens consumidos, custo estimado. Visualizada em `/logs`. → [[Nexus - Modelo de Dados e Banco]]

**App Role** — Enum `app_role` (`admin`, `staff`, `user`). Sistema de roles alternativo na tabela `user_roles`, verificado por `has_role()`. → [[Nexus - Regras de Negócio e Segurança#Roles e Enums]]

## B

**BaaS** — Backend-as-a-Service. O Nexus usa **Supabase** como BaaS completo (Auth, DB, Functions, Storage, Realtime). → [[Nexus - Stack Tecnológica]]

**BFF** — Backend for Frontend. Pattern usado pela Edge Function `process-action` que atua como orquestrador de ações com middleware Zero Trust. → [[Nexus - Edge Functions#process-action]]

## C

**Client** — Papel de usuário padrão (`role: 'client'` em `profiles`). Sujeito a limites de créditos e plano. → [[Nexus - Regras de Negócio e Segurança]]

**Command Bus** — Padrão arquitetural do Nexus Chat: a IA interpreta linguagem natural e despacha comandos via marcadores especiais (ex: `[LINKEDIN_SEARCH_STARTED]`). → [[Nexus - Fluxo de Dados#2. Fluxo do Nexus Chat (IA com Streaming)]]

**Consume Credits** — Função RPC `consume_credits(user_id, amount)` que debita créditos atomicamente. Retorna `boolean`. `SECURITY DEFINER`. → [[Nexus - Regras de Negócio e Segurança#Economia de Créditos]]

**CreditGate** — Componente React wrapper que verifica saldo de créditos antes de executar uma ação. Admin tem bypass. → [[Nexus - Regras de Negócio e Segurança#CreditGate]]

## D

**Deduct Credits** — RPC `deduct_credits(p_user_id, p_amount, p_action_name)`. Variante de consume_credits que também registra o nome da ação. → [[Nexus - Stack Tecnológica#Backend]]

## E

**Edge Function** — Função serverless executada no Supabase (Deno runtime). O Nexus tem **20 Edge Functions**. → [[Nexus - Edge Functions]]

**Embedding** — Vetor numérico que representa o significado semântico de um texto. Usado para busca semântica no chat via `text-embedding-ada-002`. Armazenado em `documents`. → [[Nexus - Edge Functions#nexus-assistant]]

## G

**Gate** — Componente visual de proteção no frontend. Existem 2: `CreditGate` (verifica créditos) e `PlanGate` (verifica plano). Definidos em `PlanMiddleware.tsx`. → [[Nexus - Regras de Negócio e Segurança#Sistema de Gates]]

**Serper.dev** — API de busca via SERP do Google. Usada pela Edge Function `search-leads` para encontrar emails de empresas. → [[Nexus - Edge Functions#search-leads]]

## I

**Integration Configs** — Tabela `integration_configs` que armazena credenciais de APIs externas por tenant (JSONB). Chave: `(user_id, integration_key)`. → [[Nexus - Modelo de Dados e Banco]]

## L

**Lead** — Contato comercial prospectado. Tabela `leads` com 22+ campos. Ciclo: `Novo → Primeiro contato → Qualificado → Call agendada → Em negociação → Fechado`. → [[Nexus - Fluxo de Dados#3. Fluxo de Gestão de Leads]]

**Lead 360** — Análise completa de um lead pela IA. Gera 9 campos: nextAction, leadProfile, opportunities, painPoints, maturityLevel, objections, approach, recommendedSolution, solutionFit. → [[Nexus - Edge Functions#analyze-lead-360]]

**LinkedIn Scout** — Funcionalidade de prospecção no LinkedIn. Busca perfis de decisores e permite análise + conversão para lead. Interface: `LinkedInScout.tsx`. → [[Nexus - Fluxo de Dados#5. Fluxo de Prospecção LinkedIn]]

**Lovable** — Plataforma de development/deploy usada pelo Nexus. Fornece AI Gateway (`ai.gateway.lovable.dev`) como proxy para modelos LLM. → [[Nexus - Stack Tecnológica#DevOps]]

## M

**Marcador** — Tag especial emitida pelo Nexus Chat que dispara ações no frontend. Exemplos: `[LINKEDIN_SEARCH_STARTED]`, `[SEARCH_PARAMS:{json}]`, `[EMAIL_SEARCH_STARTED]`, `[EMAIL_SEARCH_PARAMS:{json}]`. → [[Nexus - Fluxo de Dados#2. Fluxo do Nexus Chat]]

**Meta Graph API** — API da Meta (Facebook) para WhatsApp Business. Versão usada: **v23.0**. Endpoint: `graph.facebook.com/v23.0`. → [[Nexus - Edge Functions#whatsapp-templates]]

**Multi-tenant** — Modelo SaaS onde múltiplos tenants compartilham a mesma infraestrutura. No Nexus, cada `user_id` é um tenant. Isolamento via RLS. → [[Nexus - Regras de Negócio e Segurança#Isolamento Multi-Tenant]]

## N

**Nexus Assistant** — Nome do chatbot IA do sistema. Roda via Edge Function `nexus-assistant`. Interface: `Chat.tsx`. → [[Nexus - Edge Functions#nexus-assistant]]

**n8n Chat Histories** — Tabelas de histórico de chat vindas de automações n8n (WhatsApp). `n8n_chat_histories_duplicate` vincula conversas a leads específicos. → [[Nexus - Modelo de Dados e Banco]]

## P

**PlanGate** — Componente React wrapper que verifica se o plano do usuário atende o requisito mínimo. Admin tem bypass. → [[Nexus - Regras de Negócio e Segurança#PlanGate]]

**Plan Type** — Enum `plan_type`: `free`, `basic`, `premium`, `pro`, `enterprise`. Hierarquia: free(0) → basic(1) → pro/premium(2) → enterprise(3). → [[Nexus - Regras de Negócio e Segurança]]

**Pre-lead** — Lead prospectado ainda não convertido. Duas tabelas: `pre_leads` (Google Maps) e `pre_lead_linkedin` (LinkedIn Scout). → [[Nexus - Modelo de Dados e Banco#Oportunidades e Pré-leads]]

**ProtectedRoute** — Guard de rota mais básico. Verifica se há sessão ativa. Redireciona para `/auth` se não logado. → [[Nexus - Arquitetura#Guardas de Rota]]

## R

**React Query** — TanStack React Query v5. Gerencia server state com cache, staleTime e invalidação automática. Não confundir com state management global. → [[Nexus - Stack Tecnológica#Frontend]]

**RLS** — Row Level Security. Mecanismo do PostgreSQL que filtra linhas por políticas. No Nexus: admin vê tudo, client vê apenas `WHERE user_id = auth.uid()`. → [[Nexus - Regras de Negócio e Segurança#Segurança de Dados]]

**RP Consultoria** — Empresa dona do Nexus. Portfólio de soluções embarcado no prompt do `analyze-lead-360`: Financeiro, Nexus, Atendimento, Projetos Exclusivos. → [[Nexus - Edge Functions#analyze-lead-360]]

## S

**SecureRoute** — Componente combinado: `ProtectedRoute` + `VerifiedRoute`. Garante login E email verificado. → [[Nexus - Arquitetura#Guardas de Rota]]

**SSE** — Server-Sent Events. Protocolo de streaming usado pelo Nexus Chat para enviar tokens incrementais da IA ao frontend. → [[Nexus - Fluxo de Dados#2. Fluxo do Nexus Chat]]

**Streaming** — Modo de resposta do `nexus-assistant` onde tokens são enviados incrementalmente via SSE. Formato: `data: {"choices":[{"delta":{"content":"..."}}]}`. → [[Nexus - Edge Functions#nexus-assistant]]

## T

**Tavily** — API de pesquisa web em tempo real (IA). Usada para enriquecer dados de leads e análise de websites. → [[Nexus - Stack Tecnológica#IA]]

**Tenant** — No contexto Nexus, cada `user_id` é um tenant independente (single-user tenancy). **NÃO** existe `company_id`. → [[Nexus - Regras de Negócio e Segurança#Isolamento Multi-Tenant]]

## U

**UpgradeBanner** — Componente visual mostrado quando o `PlanGate` bloqueia uma funcionalidade. Exibe CTA para upgrade com link para `/pricing`. → [[Nexus - Regras de Negócio e Segurança#PlanGate]]

**User Role** — Enum `user_role`: `admin`, `client`. Atribuído em `profiles.role`. → [[Nexus - Regras de Negócio e Segurança#Roles e Enums]]

## V

**VerifiedRoute** — Guard de rota que exige email verificado (`email_confirmed_at`). Mostra tela de verificação se pendente. → [[Nexus - Arquitetura#Guardas de Rota]]

## W

**WABA ID** — WhatsApp Business Account ID. Armazenado em `integration_configs.config.waba_id`. Necessário para chamadas à Meta Graph API. → [[Nexus - Edge Functions#whatsapp-templates]]

## Z

**Zero Trust** — Princípio de segurança: "nunca confie, sempre verifique". No Nexus: frontend usa Gates para UX, mas o backend SEMPRE revalida créditos, auth e ownership. → [[Nexus - Regras de Negócio e Segurança]]

---

## Notas Relacionadas

- [[Nexus - Index do Projeto]] — Hub central
- [[Nexus - Arquitetura]] — Termos em contexto arquitetural
- [[Nexus - Edge Functions]] — Implementação detalhada das functions
