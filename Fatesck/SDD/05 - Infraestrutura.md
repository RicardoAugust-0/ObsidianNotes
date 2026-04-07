---
title: "05 - Infraestrutura"
tags:
  - fatesck
  - infraestrutura
  - docker
---

# 05 - Infraestrutura

## Docker Compose (Planejamento)

```yaml
services:
  # API NestJS
  api:
    build: ./apps/api
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://fatesck:fatesck@db:5432/fatesck
      - JWT_SECRET=${JWT_SECRET}
      - OAUTH_CLIENT_ID=${OAUTH_CLIENT_ID}
    depends_on:
      - db
      - minio
    restart: unless-stopped

  # Banco de Dados
  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=fatesck
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=fatesck
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  # Object Storage
  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_PASSWORD}
    volumes:
      - miniodata:/data
    restart: unless-stopped

  # Reverse Proxy
  caddy:
    image: caddy:2-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddydata:/data
    depends_on:
      - api
    restart: unless-stopped

volumes:
  pgdata:
  miniodata:
  caddydata:
```

## Caddyfile (Configuração)

```
localhost:80 {
    reverse_proxy /ws/* api:3000
    reverse_proxy /api/* api:3000
    reverse_proxy /* api:3000

    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'"
        X-Frame-Options "DENY"
        X-Content-Type-Options "nosniff"
        Referrer-Policy "strict-origin-when-cross-origin"
    }
}
```

## Pipeline de CI/CD (GitHub Actions)

```
push/PR → Lint → Typecheck → Testes (unit + integration) → Build → Deploy
```

| Estágio | Ferramenta |
|---------|-----------|
| Lint | ESLint + Prettier |
| Typecheck | `tsc --noEmit` |
| Testes | Jest / Vitest |
| Build | Next.js output standalone + NestJS bundle |
| Deploy | Push da imagem Docker |

## Segurança de Infra

> [!warning]
> - **Nunca** versionar `.env` com segredos
> - Chaves de criptografia **nunca** passam pelo servidor
> - Backups automatizados com criptografia em repouso
> - Rate limiting no proxy e na aplicação

## Referências

- [[01 - Arquitetura]] para diagramas
- [[04 - Stack Tecnológica]] para tecnologias
- [[Autenticação]] para detalhes de segurança
