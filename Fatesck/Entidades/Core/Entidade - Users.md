---
title: "Entidade - Users"
tags:
  - fatesck
  - entidade
  - database
aliases:
  - Users
  - Usuários
---

# Entidade — Users

Dados do usuário e perfil de acesso.

## Campos

| Campo | Tipo | Null | Descrição |
|-------|------|------|-----------|
| `id` | UUID (PK) | — | Identificador único |
| `email` | VARCHAR(255) | — | E-mail de login (único) |
| `emailVerified` | BOOLEAN | — | Se o e-mail foi verificado |
| `passwordHash` | TEXT | — | Hash Argon2id da senha |
| `firstName` | VARCHAR(100) | — | Primeiro nome |
| `lastName` | VARCHAR(100) | — | Sobrenome |
| `avatarUrl` | TEXT | ✓ | URL do avatar (não criptografado) |
| `role` | ENUM(`admin`, `professor`, `aluno`) | — | Papel do usuário |
| `recoveryPhraseHash` | TEXT | ✓ | Hash da frase de recuperação |
| `isActive` | BOOLEAN | — | Usuário ativo ou desativado |
| `createdAt` | TIMESTAMP | — | Data de criação |
| `updatedAt` | TIMESTAMP | — | Última atualização |

## Relacionamentos

```
Users
├─── Enrollment[]        (N:M com Classes)
├─── Class[]             (1:N como professor criador)
├─── UserKey[]           (1:1 efetivo, chave ativa)
├─── RefreshToken[]      (1:N sessões)
├─── Message[] (sender)  (1:N mensagens enviadas)
├─── Message[] (receiver)(1:N mensagens recebidas)
├─── Submission[]        (1:N entregas de atividades)
├─── File[]              (1:N uploads)
└─── AuditLog[]          (1:N logs de auditoria)
```

## Índices

- Único em `email`

## Referências

- [[03 - Modelo de Dados]]
- [[Modelo de Chaves]] para `passwordHash` e chaves criptográficas
- [[Entidade - Enrollments]] para matrículas em turmas
