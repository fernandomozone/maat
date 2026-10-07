# Arquitetura · Architecture

[Português](#português) · [English](#english)

---

## Português

O **como** da Maat. O **porquê** do produto está em [vision-and-decisions.md](vision-and-decisions.md); o levantamento que levou a estas escolhas está em [tech-stack-research.md](tech-stack-research.md).

Fixado a 2026-10-06. As versões concretas de cada peça são verificadas na fonte oficial no momento da instalação, sempre a última versão **estável**, nunca de memória.

### 1. Princípios, por ordem de precedência

Quando duas escolhas colidem, ganha a que está mais acima.

1. **Nenhuma organização vê dados de outra.** Garantido pela base de dados, não pela disciplina de quem programa.
2. **A Maat é um cliente de email, não um servidor.** O email fica no servidor IMAP; a Maat guarda o índice e as ligações.
3. **Uma linguagem no núcleo.** Tudo o que for de outra linguagem entra só como serviço separado, ligado por protocolo.
4. **Uma pessoa consegue manter isto.** Cada peça acrescentada é manutenção para sempre; o que não tiver uso real hoje não entra.
5. **Última versão estável, verificada na fonte.**

### 2. A stack

| Camada | Escolha | Porquê |
|---|---|---|
| Linguagem | TypeScript nos dois lados | Os tipos do ecrã são os mesmos da API |
| Base de dados | PostgreSQL com Row Level Security | É a base de dados que garante o isolamento entre organizações |
| Backend | Node.js com Fastify | O OpenAPI nasce das próprias rotas: valida, tipa e documenta a partir de um só esquema |
| Esquemas | Zod, um por recurso, **importados pelo backend e pelo frontend** | Validação, tipos e formulários sem cópias. O OpenAPI é o contrato para terceiros |
| Acesso a dados | `pg` com **SQL escrito à mão**, Zod a validar cada linha, migrações SQL numeradas | Sem ORM: o esquema da base de dados é a fonte, e o código só o lê. Evita a instabilidade atual dos ORMs em TypeScript |
| Frontend | React com Vite, React Router, TanStack Query | Maduro; o TanStack Query dá a sensação de resposta imediata |
| Interface | Tailwind, com as **variáveis do protótipo como tema**; modais e menus com o `<dialog>` do browser; sem biblioteca de componentes | O desenho é o do protótipo; o Tailwind é a forma de o aplicar |
| Traduções | Textos da interface em ficheiros por língua, com chaves no código; pt-PT completo, as outras línguas recaem nele quando falta uma tradução. A biblioteca escolhe-se na Fase 0, com as quatro verificações | Acrescentar uma língua é traduzir um ficheiro, sem mexer no código. O pt-BR só precisa de traduzir o que difere do pt-PT |
| Formulários | React Hook Form com os esquemas partilhados | A validação do ecrã é a mesma da API |
| Autenticação | **Sessão no servidor**: identificador opaco em cookie `httpOnly`, palavras-passe com Argon2, MFA por TOTP com códigos de recuperação | "Sair" tem de ser verdade; não há serviços distribuídos que justifiquem JWT |
| Email | imapflow (IMAP), nodemailer (SMTP), mailparser (MIME) | IDLE, CONDSTORE e QRESYNC; o ecossistema mais completo |
| Calendário | tsdav (CalDAV e CardDAV), ical.js (iCalendar e recorrências) | |
| Tarefas em segundo plano | pg-boss | Corre sobre o próprio PostgreSQL: sem Redis, um serviço a menos |
| Tempo real no browser | WebSocket no Fastify, alimentado por `LISTEN/NOTIFY` do PostgreSQL | Email novo e mensagens de chat chegam ao ecrã sem recarregar |
| Credenciais IMAP/SMTP | Cifradas na base de dados; a chave é um segredo do Docker, fora da base de dados e fora do backup, com comando para a trocar | Guardamos credenciais de terceiros: uma cópia da base de dados sozinha não as revela |
| Ficheiros | Disco do servidor, num volume Docker, atrás de uma interface própria de armazenamento (guardar, ler, apagar) | Como o Mailcow faz com o email: zero serviços extra, e a cópia de segurança é o volume mais o dump da base de dados. Se um dia for preciso S3, acrescenta-se outra implementação da interface sem mexer no resto |
| Testes | Vitest e Playwright | |
| Operação | Docker Compose, instalado num servidor próprio; proxy com HTTPS automático (desligável); scripts de backup e de atualização | Como o Mailcow: um projeto com todos os serviços em contentores. Sem armazenamento nem serviços geridos de terceiros |

### 3. Isolamento entre organizações

Uma só base de dados, com a coluna da organização em todas as tabelas de dados. A defesa não depende de nos lembrarmos do filtro:

1. **O PostgreSQL recusa.** RLS ativo e `FORCE ROW LEVEL SECURITY` em todas as tabelas. A aplicação liga-se com um utilizador que **não é dono das tabelas** nem tem `BYPASSRLS`; caso contrário, as políticas são ignoradas.
2. **A organização vem da sessão**, verificada no servidor. Nunca de um parâmetro do pedido.
3. **Declarada por transação, nunca por ligação.** Com um pool, um valor colado à ligação passaria para o pedido seguinte. Usa-se `set_config(..., true)` dentro da transação, que aceita parâmetros ligados (o `SET LOCAL` não aceita).
4. **Uma função única de acesso a dados** abre a transação, declara a organização e só expõe a função de consulta. Não há caminho alternativo.
5. **Nenhuma tabela escapa.** Um teste percorre o catálogo do PostgreSQL e falha se aparecer uma tabela sem a coluna e a política.
6. **Os ficheiros também.** São servidos pela aplicação, depois de verificar a organização, com nomes que não se adivinham.
7. **A prova.** Um teste cria duas organizações e tenta ler, alterar e apagar dados (incluindo ficheiros) de uma a partir da outra. Tem de falhar tudo.

### 4. Processos

O mesmo código, com dois pontos de entrada:

- **API**: o Fastify que serve o frontend e a API.
- **Sincronização**: um processo à parte que mantém as ligações IMAP abertas (IDLE), sincroniza calendários e envia email. Assim, uma conta lenta ou com erro não pesa na API.

### 4-B. Instalação de referência

A instalação usada no desenvolvimento (Fases 0 a 6), que pode crescer depois para o serviço alojado:

- Uma máquina virtual **só para a Maat**, separada do servidor de email, e cujo processamento se pode aumentar sem reinstalar.
- **Os dados num disco à parte** (volume de blocos montado na VM): a base de dados e as pastas de ficheiros. Assim cresce-se o disco e a máquina de forma independente.

### 5. Por decidir

Nada nesta camada. As questões de produto em aberto estão em [vision-and-decisions.md](vision-and-decisions.md).

### 6. Boas práticas

- **Cada decisão fica escrita, com data e com o porquê**, neste documento ou em [vision-and-decisions.md](vision-and-decisions.md).
- **Antes de escolher uma biblioteca**, ver na fonte quem a mantém, quantas dependências arrasta, o historial de avisos de segurança e quem depende dela.
- **Um teste que nunca falhou não prova nada.** Depois de passar, parte-se de propósito aquilo que ele devia apanhar e confirma-se que acusa. Quando há duas defesas, a prova tem de acusar cada uma por si.
- **CI com ações fixadas por commit**, não por etiqueta.
- **O código explica o porquê**, não o quê.

---

## English

Maat's **how**. The product's **why** is in [vision-and-decisions.md](vision-and-decisions.md); the research behind these choices is in [tech-stack-research.md](tech-stack-research.md).

Set on 2026-10-06. Exact versions of each piece are checked at the official source at install time, always the latest **stable** release, never from memory.

### 1. Principles, in order of precedence

When two choices collide, the higher one wins.

1. **No organization sees another's data.** Enforced by the database, not by programmer discipline.
2. **Maat is an email client, not a server.** Mail stays on the IMAP server; Maat stores the index and links.
3. **One language in the core.** Anything in another language comes in only as a separate service, connected by protocol.
4. **One person can maintain this.** Every added piece is maintenance forever; anything without real use today stays out.
5. **Latest stable version, verified at the source.**

### 2. The stack

| Layer | Choice | Why |
|---|---|---|
| Language | TypeScript on both sides | The UI's types are the API's types |
| Database | PostgreSQL with Row Level Security | The database enforces isolation between organizations |
| Backend | Node.js with Fastify | OpenAPI is generated from the routes themselves: one schema validates, types and documents |
| Schemas | Zod, one per resource, **imported by both backend and frontend** | Validation, types and forms without copies. OpenAPI is the contract for third parties |
| Data access | `pg` with **hand-written SQL**, Zod validating every row, numbered SQL migrations | No ORM: the database schema is the source and the code only reads it. Avoids today's churn in TypeScript ORMs |
| Frontend | React with Vite, React Router, TanStack Query | Mature; TanStack Query makes the UI feel instant |
| Interface | Tailwind, with the **prototype's variables as the theme**; modals and menus with the browser's `<dialog>`; no component library | The design is the prototype's; Tailwind is how it is applied |
| Translations | Interface text in per-language files, with keys in the code; pt-PT is complete and other languages fall back to it when a translation is missing. The library is chosen in Phase 0, with the four checks | Adding a language means translating a file, without touching code. pt-BR only needs to translate what differs from pt-PT |
| Forms | React Hook Form with the shared schemas | UI validation is the same as the API's |
| Authentication | **Server-side session**: opaque identifier in an `httpOnly` cookie, passwords hashed with Argon2, TOTP MFA with recovery codes | "Log out" must be true; there are no distributed services that would justify JWT |
| Email | imapflow (IMAP), nodemailer (SMTP), mailparser (MIME) | IDLE, CONDSTORE and QRESYNC; the most complete ecosystem |
| Calendar | tsdav (CalDAV and CardDAV), ical.js (iCalendar and recurrence) | |
| Background jobs | pg-boss | Runs on PostgreSQL itself: no Redis, one service fewer |
| Real-time in the browser | WebSocket in Fastify, fed by PostgreSQL `LISTEN/NOTIFY` | New mail and chat messages reach the screen without reloading |
| IMAP/SMTP credentials | Encrypted in the database; the key is a Docker secret, outside the database and outside the backup, with a rotation command | We store third-party credentials: a copy of the database alone does not reveal them |
| Files | Server disk, in a Docker volume, behind our own storage interface (put, get, delete) | Like Mailcow does with mail: no extra service, and backup is the volume plus the database dump. If S3 is ever needed, another implementation of the interface is added without touching the rest |
| Tests | Vitest and Playwright | |
| Operations | Docker Compose, installed on one's own server; reverse proxy with automatic HTTPS (can be disabled); backup and update scripts | Like Mailcow: one project with every service in containers. No managed third-party storage or services |

### 3. Isolation between organizations

A single database, with the organization column on every data table. The defense does not depend on remembering the filter:

1. **PostgreSQL refuses.** RLS enabled with `FORCE ROW LEVEL SECURITY` on every table. The application connects as a user that **does not own the tables** and has no `BYPASSRLS`; otherwise policies are ignored.
2. **The organization comes from the session**, verified on the server. Never from a request parameter.
3. **Declared per transaction, never per connection.** With a pool, a value stuck to the connection would leak into the next request. It uses `set_config(..., true)` inside the transaction, which accepts bound parameters (`SET LOCAL` does not).
4. **A single data-access function** opens the transaction, declares the organization and exposes only the query function. There is no other path.
5. **No table escapes.** A test walks the PostgreSQL catalog and fails if a table appears without the column and the policy.
6. **Files too.** They are served by the application, after checking the organization, with unguessable names.
7. **The proof.** A test creates two organizations and tries to read, change and delete data (including files) of one from the other. Everything must fail.

### 4. Processes

The same codebase, with two entry points:

- **API**: the Fastify server for the frontend and the API.
- **Sync**: a separate process that keeps IMAP connections open (IDLE), syncs calendars and sends mail. A slow or failing account does not weigh on the API.

### 4-B. Reference installation

The installation used during development (Phases 0 to 6), which can later grow into the hosted service:

- A virtual machine **for Maat only**, separate from the mail server, whose compute can be resized without reinstalling.
- **Data on a separate disk** (a block volume attached to the VM): the database and the file folders. Disk and machine grow independently.

### 5. Still open

Nothing at this layer. Open product questions are in [vision-and-decisions.md](vision-and-decisions.md).

### 6. Good practices

- **Every decision is written down, with its date and its why**, in this document or in [vision-and-decisions.md](vision-and-decisions.md).
- **Before choosing a library**, check at the source who maintains it, how many dependencies it pulls in, its security advisory history and who depends on it.
- **A test that has never failed proves nothing.** After it passes, deliberately break what it should catch and confirm it fails. When there are two defenses, the proof must catch each one on its own.
- **CI with actions pinned by commit**, not by tag.
- **Code explains the why**, not the what.
