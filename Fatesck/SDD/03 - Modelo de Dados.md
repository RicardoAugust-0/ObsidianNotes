---
title: "03 - Modelo de Dados"
tags:
  - fatesck
  - banco-dados
  - sdd
  - entidades
---

# 03 - Modelo de Dados

## Visão

- **PostgreSQL** via **Prisma ORM**
- IDs **UUIDv4** por padrão
- Todas as tabelas incluem `createdAt` e `updatedAt`

```mermaid
erDiagram
    USER ||--o{ CLASS : "cria (professor)"
    USER ||--o{ ENROLLMENT : ""
    CLASS ||--o{ ENROLLMENT : ""
    CLASS ||--o{ ASSIGNMENT : ""
    ASSIGNMENT ||--o{ SUBMISSION : ""
    USER ||--o{ SUBMISSION : "faz (aluno)"
    USER ||--o| USERKEY : ""
    USER ||--o{ MESSAGE : "envia/recebe"
    CLASS ||--o{ MESSAGE : ""
    MESSAGE o|--o{ MESSAGE : "resposta"
```

## Entidades

### Core

| Entidade | Responsabilidade | Nota |
|----------|-----------------|------|
| **Users** | Dados do usuário, perfil, papel no sistema | [[Entidade - Users]] |
| **Classes** | Turmas/disciplinas, código de acesso, configuração de chat | [[Entidade - Classes]] |
| **Enrollments** | Associação N:M entre users e classes | [[Entidade - Enrollments]] |

### Acadêmico

| Entidade | Responsabilidade | Nota |
|----------|-----------------|------|
| **Assignments** | Atividades/avaliações com prazos e pesos | [[Entidade - Assignments]] |
| **Submissions** | Entregas de atividades, notas, feedback | [[Entidade - Submissions]] |

### Segurança e Suporte

| Entidade | Responsabilidade |
|----------|-----------------|
| **UserKeys** | Chaves criptográficas RSA (1 par ativo por usuário) |
| **RefreshTokens** | Tokens de sessão (rotação por uso) |
| **Messages** | Mensagens criptografadas (blob opaco) |
| **Files** | Metadados de arquivos no Object Storage |
| **AuditLogs** | Registro de auditoria com JSONB |

## Resumo de Cardinalidades

| Entidade A | Card. | Entidade B | Detalhe |
|------------|-------|------------|---------|
| `User` | 1:N | `Class` | Professor cria turmas |
| `User` | N:M | `Class` | Via `Enrollment` |
| `Class` | 1:N | `Assignment` | Turma com atividades |
| `Assignment` | 1:N | `Submission` | Atividade com entregas |
| `User` | 1:1 | `UserKey` | Par de chaves ativo |
| `User` | 1:N | `RefreshToken` | Múltiplas sessões |
| `Class` | 1:N | `Message` | Chat de turma |
| `Message` | 1:N | `Message` | Respostas encadeadas |

## Referências

- Arquitetura: [[01 - Arquitetura]]
- Segurança E2E: [[02 - Segurança E2E]]
- Visão geral: [[00 - Visão Geral]]
