# Fase 0, parte 1: estrutura e dependências · Phase 0, part 1: structure and dependencies

[Português](#português) · [English](#english)

Estado / Status: **aprovado / approved** (2026-10-07)

---

## Português

### 1. Estrutura do repositório

Um só repositório, com espaços de trabalho do npm (incluídos no próprio npm, sem ferramenta extra):

```
maat/
├── apps/
│   ├── api/            Backend Fastify. Dois pontos de entrada:
│   │                   src/server.ts (API) e src/sync.ts (sincronização, a partir da Fase 1)
│   └── web/            Frontend React + Vite
├── packages/
│   └── schemas/        Esquemas Zod partilhados pela API e pelo frontend
├── database/
│   └── migrations/     Migrações SQL numeradas (001_…, 002_…), aplicadas por um script nosso
├── deploy/             docker-compose.yml, configuração do proxy, scripts de backup,
│                       restauro, atualização e troca de chave
├── docs/               Documentação (PT e EN)
└── prototype/          Protótipo estático (referência visual)
```

- **Node a correr TypeScript diretamente**, sem passo de compilação no backend. O `tsc` só verifica tipos.
- **Migrações sem biblioteca:** um script pequeno aplica os ficheiros SQL por ordem e regista-os numa tabela.
- **Sem Redis:** as tarefas em segundo plano usam o pg-boss sobre o PostgreSQL.

### 2. Plataforma

| Peça | Versão proposta | Notas |
|---|---|---|
| Node.js | **26** (26.10.0) | Decidido a 2026-10-07. Passa a LTS a 2026-10-28, segundo o calendário oficial; manutenção até 2029-04-30 |
| PostgreSQL | 18 (imagem oficial alpine) | Versão exata da imagem fixada na montagem |
| Proxy | Caddy 2 (imagem oficial) | Certificados Let's Encrypt automáticos; desligável |

### 3. Dependências de execução

Verificado a 2026-10-07 no npm (versão, data, responsáveis, dependências diretas, licença) e nas bases de avisos de segurança (GitHub Advisories, osv.dev, Snyk). `npm audit` sem vulnerabilidades em todo o conjunto.

**Backend** (103 pacotes no total, incluindo indiretos):

| Pacote | Versão | Para quê | Responsáveis | Dep. diretas | Avisos de segurança |
|---|---|---|---|---|---|
| fastify | 5.12.5 | Servidor HTTP | equipa Fastify (6) | 15 | Vários em 2026, todos corrigidos; **nunca abaixo de 5.12.5**. Ver nota 1 |
| @fastify/cookie | 11.1.2 | Cookie de sessão | equipa Fastify | 2 | Nenhum |
| @fastify/helmet | 13.1.1 | Cabeçalhos de segurança | equipa Fastify | 2 | Nenhum |
| @fastify/rate-limit | 11.2.0 | Limitar tentativas de login | equipa Fastify | 4 | Nenhum |
| @fastify/swagger | 9.9.1 | Gerar o OpenAPI a partir das rotas | equipa Fastify | 5 | Nenhum |
| fastify-type-provider-zod | 7.0.0 | Ligar os esquemas Zod ao Fastify | 2 pessoas | 1 | Nenhum |
| zod | 4.6.5 | Esquemas e validação | 1 pessoa | 0 | **Um aviso alto em aberto** (2026-10-01). Ver nota 2 |
| pg | 8.23.1 | Cliente PostgreSQL | 1 pessoa | 6 | Um, de 2018, corrigido |
| argon2 | 0.45.1 | Hash de palavras-passe | 1 pessoa | 4 | Nenhum |
| otpauth | 9.5.2 | TOTP (MFA) | 1 pessoa | 1 | Um, de 2020, corrigido |
| pg-boss | 12.37.0 | Tarefas em segundo plano (na Fase 0: limpeza da auditoria e das sessões) | 1 pessoa | 4 | Nenhum |

**Frontend** (18 pacotes no total):

| Pacote | Versão | Para quê | Avisos de segurança |
|---|---|---|---|
| react, react-dom | 19.3.0 | Interface | Nenhum relevante |
| react-router | 8.4.0 | Navegação | Vários, todos nas funções de renderização no servidor, que não usamos; a versão atual está limpa |
| @tanstack/react-query | 5.104.1 | Dados do servidor | Nenhum |
| react-hook-form | 7.89.0 | Formulários | Nenhum |
| @hookform/resolvers | 5.9.1 | Ligar formulários aos esquemas Zod | Nenhum |
| i18next | 26.4.2 | Traduções | Dois antigos (2017–18), corrigidos |
| react-i18next | 17.0.16 | Traduções no React | Nenhum |
| lean-qr | 2.7.4 | Código QR para ativar o MFA, gerado no browser | Nenhum; zero dependências |

### 4. Dependências de desenvolvimento (73 pacotes no total)

| Pacote | Versão | Para quê |
|---|---|---|
| typescript | 7.0.2 | Verificação de tipos |
| vite, @vitejs/plugin-react | 8.3.3, 6.1.2 | Compilar o frontend |
| tailwindcss, @tailwindcss/vite | 4.3.3 | Estilos, com o tema do protótipo |
| vitest | 5.0.3 | Testes |
| @playwright/test | 1.63.0 | Testes no browser |
| @types/node, @types/pg, @types/react, @types/react-dom | 26.6.4, 8.23.1, 19.3.0, 19.3.0 | Tipos |
| @biomejs/biome | 2.5.15 | Formatação e regras de qualidade do código, numa só ferramenta sem dependências; verificado no CI. Aprovado porque quase todo o código é escrito por agentes, em várias sessões, e os PRs entram sozinhos |

### 5. Alternativas que ficaram de fora

- **otplib** (TOTP): 6 dependências, contra 1 do otpauth.
- **qrcode**: sem versão nova desde 2024; trocado pelo lean-qr.
- **@fastify/swagger-ui**: a página de documentação interativa da API acrescenta superfície de ataque; o OpenAPI fica disponível como ficheiro JSON.
- **Bibliotecas de migrações** (node-pg-migrate e outras): um script próprio chega.
- **WebSocket** (@fastify/websocket): só é preciso na Fase 1.

### 6. Notas de segurança

1. **Fastify:** teve vários avisos em 2026, corrigidos depressa. Alguns tocaram precisamente na validação de pedidos e nos cabeçalhos de proxy (`trustProxy`), que é como a Maat corre atrás do Caddy. Regra: acompanhar as versões de perto e nunca ficar atrás.
2. **Zod:** há um aviso alto, publicado a 2026-10-01, ainda sem versão corrigida: uma lista enorme enviada para um esquema de lista sem tamanho máximo pode esgotar a memória. Regra obrigatória: **toda a lista que venha de fora tem `.max()`**, e o tamanho do corpo dos pedidos é limitado no Fastify. Acompanhar a correção.
3. **Responsáveis únicos:** zod, pg, argon2, otpauth e pg-boss são mantidos por uma pessoa cada. São bibliotecas muito usadas, mas é um risco a vigiar.

### 7. Aprovação

Aprovado a 2026-10-07: a estrutura, o Node 26 e todas as dependências das secções 3 e 4, incluindo o Biome.

---

## English

### 1. Repository structure

A single repository, using npm workspaces (built into npm, no extra tool):

```
maat/
├── apps/
│   ├── api/            Fastify backend. Two entry points:
│   │                   src/server.ts (API) and src/sync.ts (sync, from Phase 1)
│   └── web/            React + Vite frontend
├── packages/
│   └── schemas/        Zod schemas shared by the API and the frontend
├── database/
│   └── migrations/     Numbered SQL migrations (001_…, 002_…), applied by our own script
├── deploy/             docker-compose.yml, proxy configuration, backup, restore,
│                       update and key-rotation scripts
├── docs/               Documentation (PT and EN)
└── prototype/          Static prototype (visual reference)
```

- **Node runs TypeScript directly**, with no build step for the backend. `tsc` only type-checks.
- **Migrations without a library:** a small script applies the SQL files in order and records them in a table.
- **No Redis:** background jobs use pg-boss on PostgreSQL.

### 2. Platform

| Piece | Proposed version | Notes |
|---|---|---|
| Node.js | **26** (26.10.0) | Decided on 2026-10-07. Becomes LTS on 2026-10-28 per the official schedule; supported until 2029-04-30 |
| PostgreSQL | 18 (official alpine image) | Exact image version pinned at setup |
| Proxy | Caddy 2 (official image) | Automatic Let's Encrypt certificates; can be disabled |

### 3. Runtime dependencies

Checked on 2026-10-07 on npm (version, date, maintainers, direct dependencies, license) and in security advisory databases (GitHub Advisories, osv.dev, Snyk). `npm audit` reports no vulnerabilities across the whole set.

**Backend** (103 packages in total, including transitive ones):

| Package | Version | Purpose | Maintainers | Direct deps | Security advisories |
|---|---|---|---|---|---|
| fastify | 5.12.5 | HTTP server | Fastify team (6) | 15 | Several in 2026, all fixed; **never below 5.12.5**. See note 1 |
| @fastify/cookie | 11.1.2 | Session cookie | Fastify team | 2 | None |
| @fastify/helmet | 13.1.1 | Security headers | Fastify team | 2 | None |
| @fastify/rate-limit | 11.2.0 | Throttle login attempts | Fastify team | 4 | None |
| @fastify/swagger | 9.9.1 | Generate OpenAPI from routes | Fastify team | 5 | None |
| fastify-type-provider-zod | 7.0.0 | Wire Zod schemas into Fastify | 2 people | 1 | None |
| zod | 4.6.5 | Schemas and validation | 1 person | 0 | **One open high-severity advisory** (2026-10-01). See note 2 |
| pg | 8.23.1 | PostgreSQL client | 1 person | 6 | One, from 2018, fixed |
| argon2 | 0.45.1 | Password hashing | 1 person | 4 | None |
| otpauth | 9.5.2 | TOTP (MFA) | 1 person | 1 | One, from 2020, fixed |
| pg-boss | 12.37.0 | Background jobs (in Phase 0: audit and session cleanup) | 1 person | 4 | None |

**Frontend** (18 packages in total):

| Package | Version | Purpose | Security advisories |
|---|---|---|---|
| react, react-dom | 19.3.0 | UI | None relevant |
| react-router | 8.4.0 | Routing | Several, all in server-side rendering features we do not use; the current version is clean |
| @tanstack/react-query | 5.104.1 | Server data | None |
| react-hook-form | 7.89.0 | Forms | None |
| @hookform/resolvers | 5.9.1 | Connect forms to Zod schemas | None |
| i18next | 26.4.2 | Translations | Two old ones (2017–18), fixed |
| react-i18next | 17.0.16 | Translations in React | None |
| lean-qr | 2.7.4 | QR code to enable MFA, generated in the browser | None; zero dependencies |

### 4. Development dependencies (73 packages in total)

| Package | Version | Purpose |
|---|---|---|
| typescript | 7.0.2 | Type checking |
| vite, @vitejs/plugin-react | 8.3.3, 6.1.2 | Build the frontend |
| tailwindcss, @tailwindcss/vite | 4.3.3 | Styles, with the prototype's theme |
| vitest | 5.0.3 | Tests |
| @playwright/test | 1.63.0 | Browser tests |
| @types/node, @types/pg, @types/react, @types/react-dom | 26.6.4, 8.23.1, 19.3.0, 19.3.0 | Types |
| @biomejs/biome | 2.5.15 | Code formatting and quality rules, in one tool with no dependencies; checked in CI. Approved because almost all code is written by agents, across several sessions, and PRs merge automatically |

### 5. Alternatives left out

- **otplib** (TOTP): 6 dependencies, against 1 for otpauth.
- **qrcode**: no new release since 2024; replaced by lean-qr.
- **@fastify/swagger-ui**: the interactive API documentation page adds attack surface; OpenAPI is available as a JSON file.
- **Migration libraries** (node-pg-migrate and others): our own script is enough.
- **WebSocket** (@fastify/websocket): only needed in Phase 1.

### 6. Security notes

1. **Fastify:** several advisories in 2026, fixed quickly. Some touched exactly request validation and proxy headers (`trustProxy`), which is how Maat runs behind Caddy. Rule: track releases closely and never fall behind.
2. **Zod:** a high-severity advisory published on 2026-10-01, still without a fixed release: a huge array sent to an array schema with no maximum length can exhaust memory. Mandatory rule: **every array that comes from outside has `.max()`**, and request body size is limited in Fastify. Watch for the fix.
3. **Single maintainers:** zod, pg, argon2, otpauth and pg-boss are each maintained by one person. They are widely used, but it is a risk to watch.

### 7. Approval

Approved on 2026-10-07: the structure, Node 26 and every dependency in sections 3 and 4, including Biome.
