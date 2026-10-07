# Visão e decisões · Vision and decisions

[Português](#português) · [English](#english)

---

## Português

Registo do que foi decidido até agora sobre a Maat e do que continua em aberto.

### O problema

Quem gere uma pequena empresa de serviços TI trabalha com a informação espalhada: emails num cliente de email, mensagens em várias apps, projetos e tickets noutras ferramentas, calendário noutro serviço. Mesmo plataformas que juntam vários módulos na mesma página (como o Nextcloud) não os **relacionam**: cada app guarda os dados à sua maneira e as ligações entre elas são superficiais.

### A ideia

Uma plataforma com um só login e um só layout (barra de módulos no topo), em que qualquer item pode dar origem a outro e as ligações ficam visíveis dos dois lados.

Exemplos:
- um email vira um ticket, uma tarefa, um projeto ou um evento;
- uma mensagem do chat vira uma tarefa;
- um anexo é guardado nos documentos e fica ligado ao ticket;
- uma resposta a um ticket sai por email e aparece na cronologia do ticket.

### Decisões tomadas

**Âmbito da primeira versão.** Sete módulos: Email, Chat interno, Projetos, Tarefas, Tickets, Calendário e Documentos.

**Modelo de item e ligações.** Tudo partilha um modelo base de item com origem e ligações. Converter cria um item novo ligado ao original, não uma cópia.

**Email por IMAP/SMTP.** A Maat é um cliente, não um servidor de email. As mensagens ficam no servidor; a Maat guarda um índice e as ligações. As ligações apontam para o Message-ID, para não se perderem se o email mudar de pasta.

**Público-alvo inicial.** Empresas que não usam Google nem Microsoft e têm email em cPanel, Mailcow ou semelhante (utilizador e palavra-passe). Google e Microsoft ficam para depois; a autenticação das contas fica separada para permitir OAuth mais tarde.

**Calendário por CalDAV.** Sincroniza com os calendários que a empresa já tem no servidor.

**Email e chat separados.** O chat é só comunicação interna da equipa.

**Chat sobre Matrix, sem cifra ponta a ponta (2026-10-06).** Um servidor Matrix próprio, no Docker Compose. A Maat é um cliente: as mensagens vivem no servidor Matrix e a Maat guarda o índice e as ligações. Ganha-se app de telemóvel (qualquer cliente Matrix) e o caminho para ligar WhatsApp e Telegram por bridges. Sem cifra ponta a ponta porque esta impediria a procura global e as ligações do lado do servidor; as mensagens continuam cifradas em trânsito (HTTPS) e ficam no servidor próprio. A escolha do servidor Matrix, o login único e a forma de impedir que as apps liguem a cifra por omissão ficam para o início da Fase 6.

**Organização no modelo de dados desde o início.** Cada conta, projeto, ticket ou conversa pertence a uma organização, mesmo que no início só exista uma.

**Self-hosted, empacotado em Docker Compose.**

**Construção por fases (2026-10-06).** Fundação, Email, Calendário, Tarefas e Projetos, Tickets, Documentos, Chat. Uma fase só está feita quando é usada no trabalho real do dia a dia. Detalhe em [roadmap.md](roadmap.md).

**Linguagem do backend: TypeScript (Node.js).** É a única linguagem com bibliotecas completas e atuais para email e calendário (imapflow, nodemailer, tsdav, ical.js) e é também a linguagem do frontend. Ver o [levantamento da stack técnica](tech-stack-research.md).

**Stack técnica (2026-10-06).** PostgreSQL com Row Level Security para isolar organizações; Fastify; esquemas Zod partilhados entre backend e frontend; SQL escrito à mão com `pg`, sem ORM; React com Vite, React Router e TanStack Query; Tailwind com o tema do protótipo; sessão no servidor com cookie `httpOnly` e Argon2; Vitest e Playwright; Docker Compose. Detalhe e razões em [architecture.md](architecture.md).

**Layout.** O desenho da interface é o do protótipo e dos mockups do projeto (barra de módulos no topo, tema escuro e claro).

**Língua da interface (2026-10-06).** A Maat lança só em português europeu (pt-PT), mas com os textos num ficheiro de traduções desde o primeiro ecrã: o código usa chaves, nunca texto escrito diretamente. No futuro: inglês e, talvez, português do Brasil (pt-BR) como língua separada. Datas, horas, números e fusos horários seguem as definições de cada utilizador.

**Licença (2026-10-07).** AGPL-3.0-only, em todo o repositório (código e documentação). Protege o modelo de serviço gerido: quem oferecer a Maat como serviço com alterações tem de as publicar. Enquanto o dono do projeto for o único titular dos direitos, pode mudar a licença das versões futuras ou vender licenças comerciais; as versões já publicadas continuam AGPL para quem as recebeu. Se houver contribuições externas, decide-se antes se é preciso um acordo de cedência (CLA).

**Línguas.** Documentação em português e inglês no mesmo ficheiro; código em inglês. As instruções para agentes de IA ficam em `AGENTS.md` (em inglês); o projeto não usa `CLAUDE.md`.

### Utilizadores, acesso e operação (2026-10-06)

**Para quem.** Uma ferramenta empresarial: uma empresa com os seus funcionários. Quando entra alguém, o administrador cria o utilizador e a pessoa entra pela primeira vez com tudo pronto.

**Estrutura da empresa.** Departamentos e equipas, em até dois níveis (por exemplo Comercial, com Equipa 1 e Equipa 2). Cada pessoa pertence a um ou mais. Cada departamento ou equipa pode ter um **chefe**, que vê as tarefas, tickets e projetos da sua equipa, atribui trabalho às pessoas dela e vê as caixas de email partilhadas da equipa. **Nunca vê caixas pessoais.**

**Administrador da plataforma.** Papel à parte (pode ser o técnico de TI): gere utilizadores, estrutura, contas de email e definições.

**Uma conta, uma empresa.** Quem trabalha para duas empresas tem duas contas. Mas o modelo de dados separa desde já a **pessoa** da sua **pertença a uma organização**, para mais tarde a mesma pessoa poder pertencer a várias organizações da mesma instalação sem refazer a base de dados (2026-10-07).

**Várias contas de email, vários domínios.** Uma pessoa pode ter contas de domínios diferentes; nada prende uma organização a um só domínio. O caso de um prestador de serviços resolve-se na primeira versão com todas as contas na organização dele.

**API desenhada para chaves com âmbito.** Mais tarde, uma Maat poderá dar a outra (por exemplo, a de um subcontratado) uma chave de API limitada, revogável e com prazo, para trocar tickets e o trabalho feito. A API nasce a pensar nisso.

**Entrada.** Email e palavra-passe **próprios da Maat**, independentes da palavra-passe do email. MFA por **TOTP**, com códigos de recuperação. Cada empresa decide se o MFA é obrigatório para todos; para administradores é **sempre obrigatório**. Passkeys ficam para mais tarde.

**Contas de email.** Configuradas pelo administrador ao criar o utilizador, **incluindo a palavra-passe da caixa**. As palavras-passe ficam cifradas; a chave é um segredo do Docker, fora da base de dados, com um comando para a trocar.

**Caixas partilhadas** (como `suporte@`). Atribuídas a departamentos ou equipas. Mais tarde, também a pessoas concretas; o modelo de dados fica preparado para isso.

**Procura.** Encontra emails pelo **assunto, conteúdo e anexos**. Se isso se faz com um índice próprio da Maat ou com a pesquisa do servidor IMAP decide-se por **medição**, no início da Fase 1, com contas reais. Uma segunda cópia do conteúdo dos emails só entra se os números a justificarem.

**Emails em HTML.** Mostrados numa área isolada, sem scripts. **Imagens externas bloqueadas por omissão**, com "mostrar imagens" e "confiar neste domínio"; a lista de domínios de confiança é **de cada pessoa**.

**Registo de auditoria.** Regista tudo o que altera dados (criar, alterar, apagar, converter) e as ações de administração e segurança, sem nunca guardar o conteúdo de emails ou mensagens. O administrador vê tudo; cada chefe vê o da sua equipa. O prazo é definido pela empresa, com 1 ano por omissão.

**Segurança.** OWASP ASVS nível 2 como base, RGPD como obrigação, NIS2 como alinhamento. Ver [security.md](security.md).

**Instalação.** Assistente no browser na primeira visita, protegido por um código de instalação que aparece no terminal. O assistente mostra a chave de cifra uma vez e pede para a guardar fora do servidor.

**HTTPS.** Proxy incluído no Docker Compose, com certificados Let's Encrypt automáticos, que se pode desligar quando o servidor já tem outro proxy.

**Cópias de segurança.** Ao estilo do Mailcow: um script faz o dump consistente da base de dados (e, a partir da Fase 6, da do Matrix), com retenção de N dias; as pastas de dados (anexos, documentos, imagens, ficheiros do chat) copiam-se com `rsync`, com um exemplo de `crontab` na documentação. A chave de cifra **não entra** no backup.

**Atualizações.** Script que faz backup antes, descarrega a versão nova, aplica as migrações e reinicia.

**Ideia para o futuro.** Ligação a um Active Directory local.

### Fora da primeira versão

- **WhatsApp e Telegram.** Ficam para uma fase posterior. O WhatsApp não tem API oficial para contas pessoais e as integrações existentes podem partir com atualizações.
- **Git.** Fica para mais tarde, como integração com um servidor Git existente em vez de ser construído de raiz.
- **Contas Google e Microsoft** (OAuth).

### Questões em aberto

- **Modelo de distribuição.** Instalação pelo próprio cliente, alojamento gerido com uma instalação por cliente, ou plataforma multi-empresa. Por decidir depois de a plataforma estar em uso.

---

## English

A record of what has been decided about Maat so far and what remains open.

### The problem

People running a small IT services company work with scattered information: emails in an email client, messages in several apps, projects and tickets in other tools, the calendar in yet another service. Even platforms that put several modules on the same page (such as Nextcloud) don't **relate** them: each app stores its data its own way, and the links between them are shallow.

### The idea

A platform with a single login and a single layout (module bar at the top), where any item can give rise to another and the links are visible from both sides.

Examples:
- an email becomes a ticket, a task, a project or an event;
- a chat message becomes a task;
- an attachment is saved to documents and linked to the ticket;
- a reply to a ticket goes out by email and shows up in the ticket's timeline.

### Decisions made

**First version scope.** Seven modules: Email, internal Chat, Projects, Tasks, Tickets, Calendar and Documents.

**Item and link model.** Everything shares a base item model with an origin and links. Converting creates a new item linked to the original, not a copy.

**Email via IMAP/SMTP.** Maat is a client, not a mail server. Messages stay on the server; Maat keeps an index and the links. Links point to the Message-ID so they survive the email moving to another folder.

**Initial target audience.** Companies that don't use Google or Microsoft and have email on cPanel, Mailcow or similar (username and password). Google and Microsoft come later; account authentication is kept separate to allow OAuth later.

**Calendar via CalDAV.** Syncs with the calendars the company already has on its server.

**Email and chat kept separate.** Chat is internal team communication only.

**Chat on Matrix, without end-to-end encryption (2026-10-06).** A self-hosted Matrix server in Docker Compose. Maat is a client: messages live on the Matrix server and Maat stores the index and links. This gives a mobile app (any Matrix client) and a path to connect WhatsApp and Telegram through bridges. No end-to-end encryption, because it would prevent global search and server-side links; messages are still encrypted in transit (HTTPS) and stay on one's own server. The choice of Matrix server, single sign-on and how to stop apps from enabling encryption by default are left for the start of Phase 6.

**Organization in the data model from day one.** Every account, project, ticket or conversation belongs to an organization, even if only one exists at first.

**Self-hosted, packaged with Docker Compose.**

**Built in phases (2026-10-06).** Foundation, Email, Calendar, Tasks and Projects, Tickets, Documents, Chat. A phase is only done when it is used in real day-to-day work. Details in [roadmap.md](roadmap.md).

**Backend language: TypeScript (Node.js).** It is the only language with complete, current libraries for email and calendar (imapflow, nodemailer, tsdav, ical.js), and it is also the frontend's language. See the [tech stack research](tech-stack-research.md).

**Tech stack (2026-10-06).** PostgreSQL with Row Level Security to isolate organizations; Fastify; Zod schemas shared between backend and frontend; hand-written SQL with `pg`, no ORM; React with Vite, React Router and TanStack Query; Tailwind with the prototype's theme; server-side sessions with an `httpOnly` cookie and Argon2; Vitest and Playwright; Docker Compose. Details and reasons in [architecture.md](architecture.md).

**Layout.** The interface design is the one in the project's prototype and mockups (module bar at the top, dark and light themes).

**Interface language (2026-10-06).** Maat launches in European Portuguese (pt-PT) only, but with interface text in a translation file from the very first screen: code uses keys, never hard-coded text. Later: English and possibly Brazilian Portuguese (pt-BR) as a separate language. Dates, times, numbers and time zones follow each user's settings.

**License (2026-10-07).** AGPL-3.0-only across the whole repository (code and documentation). It protects the managed-service model: anyone offering Maat as a service with changes must publish them. While the project owner is the sole copyright holder, they can relicense future versions or sell commercial licenses; versions already published remain AGPL for those who received them. If external contributions arrive, decide first whether a contributor agreement (CLA) is needed.

**Languages.** Documentation in Portuguese and English in the same file; code in English. Instructions for AI agents live in `AGENTS.md` (in English); the project does not use `CLAUDE.md`.

### Users, access and operations (2026-10-06)

**Who it is for.** A business tool: a company and its employees. When someone joins, the administrator creates the user and the person logs in for the first time with everything ready.

**Company structure.** Departments and teams, up to two levels (for example Sales, with Team 1 and Team 2). Each person belongs to one or more. Each department or team can have a **head**, who sees the team's tasks, tickets and projects, assigns work to its people and sees the team's shared mailboxes. **Never personal mailboxes.**

**Platform administrator.** A separate role (can be the IT technician): manages users, structure, email accounts and settings.

**One account, one company.** Someone who works for two companies has two accounts. But the data model separates the **person** from their **membership in an organization** from the start, so that later the same person can belong to several organizations in the same installation without reworking the database (2026-10-07).

**Several email accounts, several domains.** A person can have accounts on different domains; nothing ties an organization to a single domain. A service provider's case is handled in the first version with all their accounts in their own organization.

**API designed for scoped keys.** Later, one Maat may give another (for example, a subcontractor's) a limited, revocable, expiring API key to exchange tickets and the work done. The API is designed with this in mind from the start.

**Login.** Email and a **Maat-specific password**, independent of the mailbox password. MFA with **TOTP**, plus recovery codes. Each company decides whether MFA is mandatory for everyone; for administrators it is **always mandatory**. Passkeys come later.

**Email accounts.** Set up by the administrator when creating the user, **including the mailbox password**. Passwords are stored encrypted; the key is a Docker secret, outside the database, with a command to rotate it.

**Shared mailboxes** (such as `support@`). Assigned to departments or teams. Later, also to individual people; the data model is ready for that.

**Search.** Finds emails by **subject, body and attachments**. Whether this uses Maat's own index or the IMAP server's search is decided by **measurement** at the start of Phase 1, with real accounts. A second copy of email content is only added if the numbers justify it.

**HTML email.** Shown in an isolated area, without scripts. **Remote images blocked by default**, with "show images" and "trust this domain"; the trusted-domain list is **per person**.

**Audit log.** Records everything that changes data (create, update, delete, convert) and administration and security actions, never storing the content of emails or messages. The administrator sees everything; each head sees their team's. Retention is set by the company, 1 year by default.

**Security.** OWASP ASVS level 2 as the baseline, GDPR as an obligation, NIS2 as alignment. See [security.md](security.md).

**Installation.** A browser wizard on the first visit, protected by an installation code shown in the terminal. The wizard shows the encryption key once and asks for it to be stored off the server.

**HTTPS.** A reverse proxy included in Docker Compose, with automatic Let's Encrypt certificates, which can be disabled when the server already has another proxy.

**Backups.** Mailcow-style: a script makes a consistent dump of the database (and, from Phase 6, the Matrix database), with N days of retention; data folders (attachments, documents, images, chat files) are copied with `rsync`, with an example `crontab` in the documentation. The encryption key is **not** included in the backup.

**Updates.** A script that backs up first, downloads the new version, applies migrations and restarts.

**Future idea.** Integration with an on-premises Active Directory.

### Out of the first version

- **WhatsApp and Telegram.** Deferred to a later phase. WhatsApp has no official API for personal accounts, and existing integrations can break with updates.
- **Git.** Later, as an integration with an existing Git server rather than built from scratch.
- **Google and Microsoft accounts** (OAuth).

### Open questions

- **Distribution model.** Self-installed by the customer, managed hosting with one instance per customer, or a multi-tenant platform. To be decided once the platform is in use.
