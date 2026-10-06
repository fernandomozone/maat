# Levantamento da stack técnica · Tech stack research

[Português](#português) · [English](#english)

Relacionado com / Related to: [issue #3](https://github.com/fernandomozone/maat/issues/3)

---

## Português

Levantamento feito a 2026-10-06 para escolher a linguagem do backend da Maat. Objetivo: ver, módulo a módulo, o que já existe para reaproveitar em cada linguagem candidata (TypeScript/Node.js, Python, Go, Rust, PHP), antes de escolher. **Este documento não toma a decisão**; serve de base para ela.

Limites: as datas vêm dos registos de pacotes (npm, PyPI, crates.io, pkg.go.dev, Packagist) e do Docker Hub. A API do GitHub não estava acessível, por isso há datas e licenças marcadas como não verificadas.

### Como evitar o "Frankenstein"

A regra proposta: **o núcleo da Maat (API, modelo de item e ligações, workers de sincronização) numa só linguagem**. Tudo o que for de outra linguagem só entra como **serviço separado, ligado por protocolo** (IMAP, CalDAV, S3, OIDC, Matrix), nunca como código misturado. Assim, a base de dados, o armazenamento de ficheiros ou um servidor Matrix podem ser escritos em qualquer linguagem sem afetar o código que mantemos.

### 1. Existe um projeto que sirva de base?

Não. Nenhum projeto open source combina as duas escolhas centrais da Maat: ser **cliente** IMAP (o email fica no servidor) e permitir que **qualquer item se ligue a qualquer outro**, nos dois sentidos.

| Projeto | Linguagem | Licença | Porque não serve de base |
|---|---|---|---|
| [Huly](https://github.com/hcengineering/platform) | TypeScript | EPL-2.0 | Sem cliente IMAP nem CalDAV (só Gmail e Google Calendar); stack pesada (8–16 GB RAM); a empresa deixou de financiar o serviço alojado em julho de 2026 |
| [Twenty](https://github.com/twentyhq/twenty) | TypeScript | AGPLv3 + partes comerciais | É um CRM. O código de sincronização IMAP e CalDAV serve de **referência** |
| [Frappe Helpdesk / ERPNext](https://github.com/frappe/helpdesk) | Python | AGPL-3.0 | O mais próximo de um "framework" com tudo ligado, mas prende ao Python e ao modelo de dados do Frappe |
| [Zammad](https://github.com/zammad/zammad), [FreeScout](https://github.com/freescout-help-desk/freescout) | Ruby, PHP | AGPLv3 | Tickets por email bem feitos, mas **tomam conta da caixa de correio** (podem apagar o email), o que contraria a Maat |
| [Nextcloud](https://apps.nextcloud.com/apps/mail), [SOGo](https://github.com/Alinto/sogo) | PHP, Objective-C | AGPL, GPL | Úteis como **serviços laterais** de CalDAV ou WebDAV, não como base |
| EGroupware, Group-Office, tine | PHP | GPL/AGPL | Código antigo; no Group-Office os projetos e tickets são pagos; o tine está limitado a 5 utilizadores sem licença |

Conclusão: **núcleo próprio**, com integrações ao nível do protocolo.

### 2. Serviço de email pronto (em vez de biblioteca)

| Serviço | Licença | Estado |
|---|---|---|
| [EmailEngine](https://github.com/postalsys/emailengine) | Proprietária: 14 dias de teste, depois exige chave paga | Muito ativo |
| [RustMailer](https://github.com/rustmailer/rustmailer) | Proprietária, com teste de 14 dias | Mesmo problema de licença |
| [Nylas sync-engine](https://github.com/nylas/sync-engine) | AGPL | Arquivado, sem manutenção |
| [Mailspring-Sync](https://github.com/Foundry376/Mailspring-Sync) | GPL-3.0 | Motor de desktop, um processo por conta |

Não há alternativa gratuita, mantida e com licença permissiva ao EmailEngine. Para um projeto público, o caminho realista é usar uma **biblioteca** IMAP dentro do nosso backend.

### 3. Tabela por módulo e linguagem

Legenda: **✔ maduro** · **◐ parcial** (existe, mas com lacunas ou ainda antes da versão 1.0) · **✘ lacuna** (não há opção mantida)

| Área | TypeScript/Node | Python | Go | Rust | PHP |
|---|---|---|---|---|---|
| Cliente IMAP | ✔ [imapflow](https://github.com/postalsys/imapflow) (MIT; IDLE, CONDSTORE, QRESYNC) | ◐ [IMAPClient](https://pypi.org/project/IMAPClient/), [imap-tools](https://pypi.org/project/imap-tools/): síncronos, sem QRESYNC; a opção assíncrona é GPL | ◐ [go-imap v2](https://pkg.go.dev/github.com/emersion/go-imap/v2) ainda em beta | ◐ [async-imap](https://crates.io/crates/async-imap): sem QRESYNC | ◐ [Horde Imap_Client](https://github.com/horde/Imap_Client) (LGPL, com QRESYNC), [ImapEngine](https://github.com/DirectoryTree/ImapEngine); ligações longas exigem processo à parte |
| Envio SMTP | ✔ [nodemailer](https://www.npmjs.com/package/nodemailer) | ✔ smtplib, [aiosmtplib](https://pypi.org/project/aiosmtplib/) | ✔ [go-mail](https://pkg.go.dev/github.com/wneessen/go-mail) | ✔ [lettre](https://crates.io/crates/lettre) | ✔ [symfony/mailer](https://packagist.org/packages/symfony/mailer) |
| Ler e compor MIME | ✔ [mailparser](https://www.npmjs.com/package/mailparser), [postal-mime](https://www.npmjs.com/package/postal-mime) | ✔ módulo `email` da biblioteca padrão | ✔ [enmime](https://pkg.go.dev/github.com/jhillyerd/enmime/v2) | ✔ [mail-parser](https://crates.io/crates/mail-parser) (Stalwart) | ✔ [zbateson](https://packagist.org/packages/zbateson/mail-mime-parser) |
| CalDAV / CardDAV | ✔ [tsdav](https://www.npmjs.com/package/tsdav) (os dois) + [ical.js](https://www.npmjs.com/package/ical.js) | ◐ [caldav](https://pypi.org/project/caldav/) sim; sem biblioteca CardDAV | ◐ [go-webdav](https://pkg.go.dev/github.com/emersion/go-webdav) v0.7; RRULE sem versões desde 2023 | ◐ [libdav](https://crates.io/crates/libdav) v0.x | ✘ sem cliente CalDAV mantido ([sabre/vobject](https://packagist.org/packages/sabre/vobject) é forte para .ics) |
| Chat em tempo real | ✔ socket.io, ws | ✔ python-socketio, websockets | ✔ [centrifuge](https://pkg.go.dev/github.com/centrifugal/centrifuge) | ✔ axum, socketioxide | ◐ Laravel Reverb, em processo à parte |
| SDK Matrix (se o chat usar Matrix) | ✔ matrix-js-sdk | ✔ matrix-nio, mautrix | ✔ mautrix-go | ✔ matrix-rust-sdk | ✘ sem SDK estável |
| Ficheiros (S3) e pré-visualizações | ✔ AWS SDK, sharp, pdfjs | ✔ boto3, pypdfium2 (evitar PyMuPDF, que é AGPL) | ✔ AWS SDK; imagens via cgo | ✔ aws-sdk-s3, pdfium-render | ✔ AWS SDK; PDF via Ghostscript |
| Tarefas em segundo plano | ✔ pg-boss, BullMQ | ✔ Celery, Procrastinate e outros | ✔ River (Postgres, antes da v1) | ◐ apalis em RC | ✔ Laravel Queues |
| Framework web + ORM | ✔ NestJS / Fastify; ORMs ainda a mudar (Drizzle antes da v1, Prisma a passar para a v8) | ✔ Django (tudo incluído) | ✔ chi + pgx + sqlc (montado à mão) | ◐ axum + sqlx/SeaORM, muito antes da v1 | ✔ Laravel, Symfony |
| Autenticação / OIDC | ✔ openid-client, better-auth | ✔ Authlib, django-allauth | ✔ go-oidc, zitadel/oidc | ◐ openidconnect | ◐ bom dentro do Laravel/Symfony, fraco fora deles |

### 4. Serviços de infraestrutura (independentes da linguagem)

- **Base de dados:** PostgreSQL, que também dá pesquisa de texto (com configuração para português), sem serviço extra.
- **Armazenamento de ficheiros:** o **MinIO Community foi arquivado em abril de 2026**, por isso fica de fora. As alternativas são [Garage](https://garagehq.deuxfleurs.fr/) (AGPL, leve), [SeaweedFS](https://github.com/seaweedfs/seaweedfs) (Apache-2.0) ou o [Object Storage da Hetzner](https://docs.hetzner.com/storage/object-storage/overview/).
- **Pesquisa (opcional):** [Meilisearch](https://github.com/meilisearch/meilisearch) (edição comunitária em MIT), se o Postgres não chegar.
- **Login externo (opcional):** Authentik, Zitadel (tem multi-organização), Kanidm ou Keycloak.
- **Matrix (se for a escolha do chat):** o [Synapse passou a AGPL em 2023](https://element.io/blog/element-to-adopt-agplv3/); o Dendrite está só em manutenção de segurança; [Tuwunel](https://github.com/matrix-construct/tuwunel) é Apache-2.0.

### 5. Leitura do levantamento

- **TypeScript/Node** é a única linguagem com o email e o calendário completos e atuais (imapflow, nodemailer, tsdav, ical.js). Também é a linguagem do frontend, o que dá **uma só linguagem em todo o projeto**. Ponto fraco: os ORMs mudam muito.
- **Python** é o segundo mais completo e o Django traz muita coisa feita. As lacunas estão precisamente no email (sem QRESYNC, IMAP assíncrono só em GPL) e no CardDAV.
- **Go** é ótimo para workers de longa duração, mas o cliente IMAP ainda está em beta.
- **Rust** tem todas as peças, mas muitas antes da versão 1.0, e é o que dá mais trabalho a juntar.
- **PHP** é muito produtivo com Laravel, mas não tem cliente CalDAV mantido nem SDK Matrix, e as ligações IMAP longas exigem processos à parte.

Falta a decisão. O fator que este levantamento não mede é **em que linguagem o projeto vai ser mantido com mais conforto**.

---

## English

Research done on 2026-10-06 to choose Maat's backend language. Goal: see, module by module, what already exists to reuse in each candidate language (TypeScript/Node.js, Python, Go, Rust, PHP) before choosing. **This document does not make the decision**; it is the basis for it.

Limits: dates come from package registries (npm, PyPI, crates.io, pkg.go.dev, Packagist) and Docker Hub. The GitHub API was not reachable, so some dates and licenses are marked as unverified.

### Avoiding the "Frankenstein"

Proposed rule: **Maat's core (API, item and link model, sync workers) in a single language**. Anything in another language only comes in as a **separate service, connected by protocol** (IMAP, CalDAV, S3, OIDC, Matrix), never as mixed code. That way the database, file storage or a Matrix server can be written in any language without affecting the code we maintain.

### 1. Is there a project that could be the base?

No. No open-source project combines Maat's two core choices: being an IMAP **client** (mail stays on the server) and letting **any item link to any other**, both ways.

| Project | Language | License | Why it can't be the base |
|---|---|---|---|
| [Huly](https://github.com/hcengineering/platform) | TypeScript | EPL-2.0 | No IMAP or CalDAV client (Gmail and Google Calendar only); heavy stack (8–16 GB RAM); the company stopped funding its hosted service in July 2026 |
| [Twenty](https://github.com/twentyhq/twenty) | TypeScript | AGPLv3 + commercial parts | It's a CRM. Its IMAP and CalDAV sync code is a useful **reference** |
| [Frappe Helpdesk / ERPNext](https://github.com/frappe/helpdesk) | Python | AGPL-3.0 | The closest "everything linked" framework, but locks in Python and Frappe's data model |
| [Zammad](https://github.com/zammad/zammad), [FreeScout](https://github.com/freescout-help-desk/freescout) | Ruby, PHP | AGPLv3 | Good email-threaded tickets, but they **take over the mailbox** (can delete mail), which contradicts Maat |
| [Nextcloud](https://apps.nextcloud.com/apps/mail), [SOGo](https://github.com/Alinto/sogo) | PHP, Objective-C | AGPL, GPL | Useful as CalDAV or WebDAV **side services**, not as a base |
| EGroupware, Group-Office, tine | PHP | GPL/AGPL | Older codebases; Group-Office's projects and tickets are paid; tine is limited to 5 users without a license |

Conclusion: **our own core**, with protocol-level integrations.

### 2. Ready-made email service (instead of a library)

| Service | License | Status |
|---|---|---|
| [EmailEngine](https://github.com/postalsys/emailengine) | Proprietary: 14-day trial, then a paid key is required | Very active |
| [RustMailer](https://github.com/rustmailer/rustmailer) | Proprietary, 14-day trial | Same licensing problem |
| [Nylas sync-engine](https://github.com/nylas/sync-engine) | AGPL | Archived, unmaintained |
| [Mailspring-Sync](https://github.com/Foundry376/Mailspring-Sync) | GPL-3.0 | Desktop engine, one process per account |

There is no free, maintained, permissively licensed alternative to EmailEngine. For a public project, the realistic path is an IMAP **library** inside our backend.

### 3. Table by module and language

Legend: **✔ mature** · **◐ partial** (exists, but with gaps or pre-1.0) · **✘ gap** (no maintained option)

| Area | TypeScript/Node | Python | Go | Rust | PHP |
|---|---|---|---|---|---|
| IMAP client | ✔ [imapflow](https://github.com/postalsys/imapflow) (MIT; IDLE, CONDSTORE, QRESYNC) | ◐ [IMAPClient](https://pypi.org/project/IMAPClient/), [imap-tools](https://pypi.org/project/imap-tools/): synchronous, no QRESYNC; the async option is GPL | ◐ [go-imap v2](https://pkg.go.dev/github.com/emersion/go-imap/v2) still beta | ◐ [async-imap](https://crates.io/crates/async-imap): no QRESYNC | ◐ [Horde Imap_Client](https://github.com/horde/Imap_Client) (LGPL, with QRESYNC), [ImapEngine](https://github.com/DirectoryTree/ImapEngine); long-lived connections need a separate process |
| SMTP sending | ✔ [nodemailer](https://www.npmjs.com/package/nodemailer) | ✔ smtplib, [aiosmtplib](https://pypi.org/project/aiosmtplib/) | ✔ [go-mail](https://pkg.go.dev/github.com/wneessen/go-mail) | ✔ [lettre](https://crates.io/crates/lettre) | ✔ [symfony/mailer](https://packagist.org/packages/symfony/mailer) |
| MIME parse and compose | ✔ [mailparser](https://www.npmjs.com/package/mailparser), [postal-mime](https://www.npmjs.com/package/postal-mime) | ✔ stdlib `email` | ✔ [enmime](https://pkg.go.dev/github.com/jhillyerd/enmime/v2) | ✔ [mail-parser](https://crates.io/crates/mail-parser) (Stalwart) | ✔ [zbateson](https://packagist.org/packages/zbateson/mail-mime-parser) |
| CalDAV / CardDAV | ✔ [tsdav](https://www.npmjs.com/package/tsdav) (both) + [ical.js](https://www.npmjs.com/package/ical.js) | ◐ [caldav](https://pypi.org/project/caldav/) yes; no CardDAV library | ◐ [go-webdav](https://pkg.go.dev/github.com/emersion/go-webdav) v0.7; RRULE without releases since 2023 | ◐ [libdav](https://crates.io/crates/libdav) v0.x | ✘ no maintained CalDAV client ([sabre/vobject](https://packagist.org/packages/sabre/vobject) is strong for .ics) |
| Real-time chat | ✔ socket.io, ws | ✔ python-socketio, websockets | ✔ [centrifuge](https://pkg.go.dev/github.com/centrifugal/centrifuge) | ✔ axum, socketioxide | ◐ Laravel Reverb, as a separate process |
| Matrix SDK (if chat uses Matrix) | ✔ matrix-js-sdk | ✔ matrix-nio, mautrix | ✔ mautrix-go | ✔ matrix-rust-sdk | ✘ no stable SDK |
| Files (S3) and previews | ✔ AWS SDK, sharp, pdfjs | ✔ boto3, pypdfium2 (avoid PyMuPDF, which is AGPL) | ✔ AWS SDK; images via cgo | ✔ aws-sdk-s3, pdfium-render | ✔ AWS SDK; PDF via Ghostscript |
| Background jobs | ✔ pg-boss, BullMQ | ✔ Celery, Procrastinate and others | ✔ River (Postgres, pre-1.0) | ◐ apalis in RC | ✔ Laravel Queues |
| Web framework + ORM | ✔ NestJS / Fastify; ORMs still churning (Drizzle pre-1.0, Prisma moving to v8) | ✔ Django (batteries included) | ✔ chi + pgx + sqlc (assembled by hand) | ◐ axum + sqlx/SeaORM, much pre-1.0 | ✔ Laravel, Symfony |
| Authentication / OIDC | ✔ openid-client, better-auth | ✔ Authlib, django-allauth | ✔ go-oidc, zitadel/oidc | ◐ openidconnect | ◐ good inside Laravel/Symfony, thin outside them |

### 4. Infrastructure services (language-independent)

- **Database:** PostgreSQL, which also provides full-text search (with a Portuguese configuration) without an extra service.
- **File storage:** **MinIO Community was archived in April 2026**, so it is out. Alternatives are [Garage](https://garagehq.deuxfleurs.fr/) (AGPL, lightweight), [SeaweedFS](https://github.com/seaweedfs/seaweedfs) (Apache-2.0) or [Hetzner Object Storage](https://docs.hetzner.com/storage/object-storage/overview/).
- **Search (optional):** [Meilisearch](https://github.com/meilisearch/meilisearch) (community edition is MIT), if Postgres isn't enough.
- **External login (optional):** Authentik, Zitadel (multi-organization built in), Kanidm or Keycloak.
- **Matrix (if chosen for chat):** [Synapse moved to AGPL in 2023](https://element.io/blog/element-to-adopt-agplv3/); Dendrite is in security-only maintenance; [Tuwunel](https://github.com/matrix-construct/tuwunel) is Apache-2.0.

### 5. Reading the research

- **TypeScript/Node** is the only language with complete, current email and calendar support (imapflow, nodemailer, tsdav, ical.js). It is also the frontend's language, giving **one language across the whole project**. Weak spot: ORMs change a lot.
- **Python** is the second most complete, and Django brings a lot ready-made. Its gaps are precisely in email (no QRESYNC, async IMAP only under GPL) and CardDAV.
- **Go** is great for long-running workers, but its IMAP client is still beta.
- **Rust** has every piece, but much of it is pre-1.0, and it takes the most work to assemble.
- **PHP** is very productive with Laravel, but has no maintained CalDAV client or Matrix SDK, and long-lived IMAP connections need separate processes.

The decision is still open. The factor this research does not measure is **which language the project will be most comfortably maintained in**.
