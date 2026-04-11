---
tags:
  - nexus
  - backend
  - edge-functions
  - whatsapp
  - supabase
created: 2026-04-08
updated: 2026-04-08
refactored: 2026-04-08
parent: "[[Nexus - Edge Functions]]"
---

# EF — `send-whatsapp-message`

> Envia uma mensagem de texto livre via WhatsApp usando a API proprietária da RP Consultoria. **Não usa o sistema de templates** — é envio direto de texto.

---

## Resumo

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/send-whatsapp-message/index.ts` |
| **Custo** | 0 créditos (não chama `consumeCredits`) |
| **Auth** | JWT obrigatório (`requireAuth`) |
| **Método HTTP** | `POST` |
| **API Externa** | `https://api.rpconsultoriaone.shop/message/sendText/RPOne` |

> [!important] API proprietária, não Meta
> Esta função **não usa a Meta Graph API**. Ela chama uma API própria da RP Consultoria (`rpconsultoriaone.shop`), autenticada por `WHATSAPP_API_KEY` (env secret). Contrasta com [[EF - whatsapp-templates]], que usa Meta Graph API v23.0 para gerenciar templates.

---

## Contrato de API

**Request body:**
```json
{ "phone": "5511999990000", "message": "Olá! Tudo bem?" }
```

**Validação:** `phone` e `message` são obrigatórios. Ausência retorna `400 Bad Request`.

**Response (sucesso):**
```json
{ "success": true, "data": { ...resposta_da_api_whatsapp } }
```

**Response (erro de envio):**
```json
{ "error": "Falha ao enviar mensagem via WhatsApp" }
```

---

## Fluxo de Execução

```
1. CORS preflight (OPTIONS → 200)
2. requireAuth(req) → { user, supabase } — Zero Trust
3. Validar phone + message no body
4. Ler WHATSAPP_API_KEY de Deno.env (nunca exposta ao frontend)
   └─ Se ausente → 500 "WHATSAPP_API_KEY not configured"
5. POST para https://api.rpconsultoriaone.shop/message/sendText/RPOne
   Headers: { "Content-Type": "application/json", "apikey": apiKey }
   Body:    { "number": phone, "text": message }
6. Se response não-OK → logger.error + retorna status da API externa
7. logger.success({ duration_ms, phone })
8. Retornar { success: true, data: resultado }
```

> [!success] Refatorado em 2026-04-08
> Função migrada da Meta WhatsApp Cloud API (que lia credenciais de `integration_configs`) para a API proprietária da RP Consultoria. Autenticação agora via `WHATSAPP_API_KEY` env secret.

---

## Segurança

- **Zero Trust:** `userId` extraído do JWT — nunca confiado no body
- **API Key protegida:** `WHATSAPP_API_KEY` é um env secret do Supabase, nunca bundlada no frontend
- A função **não valida formato do telefone** além de checar se o campo está presente — o número é passado diretamente para a API externa

> [!warning] Sem validação de formato de telefone
> O campo `phone` não é validado contra E.164 ou qualquer regex. Número inválido resultará em erro da API externa (código HTTP não-200), que é logado e retornado ao cliente.

---

## Diferença entre send-whatsapp-message e whatsapp-templates

| Aspecto | `send-whatsapp-message` | `whatsapp-templates` |
|:---|:---|:---|
| **Propósito** | Envio de texto livre | CRUD de templates aprovados |
| **API** | `rpconsultoriaone.shop` (proprietária) | Meta Graph API v23.0 |
| **Auth externa** | `WHATSAPP_API_KEY` (env secret) | `integration_configs` (por tenant) |
| **Custo crédito** | 0 créditos | 0 créditos (CRUD) |
| **Notas** | Envio direto, sem template | Gerencia templates, não envia |

Para envio de mensagens **via templates aprovados pela Meta**, veja [[EF - whatsapp-templates]].

---

## Interação com o Ecossistema

Esta função é chamada pelo frontend quando o usuário dispara uma mensagem manual para um lead via interface do Nexus. Não passa pelo `process-action` (BFF) — é chamada diretamente.

```
Frontend (LeadDetail / Chat) → send-whatsapp-message → API rpconsultoriaone.shop → WhatsApp
```

---

## Notas Relacionadas

- [[Nexus - Edge Functions]] — Índice de todas as functions
- [[EF - whatsapp-templates]] — CRUD de templates Meta
- [[Nexus - Shared Helpers]] — `requireAuth`, `createLogger`, `corsHeaders`
- [[Nexus - Regras de Negócio e Segurança]] — Modelo Zero Trust
