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

**Organização no modelo de dados desde o início.** Cada conta, projeto, ticket ou conversa pertence a uma organização, mesmo que no início só exista uma.

**Self-hosted, empacotado em Docker Compose.**

**Construção por fases.** Todos os módulos estão no plano, mas são construídos por ordem para a plataforma ser usável cedo.

**Línguas.** Documentação em português e inglês no mesmo ficheiro; código em inglês. As instruções para agentes de IA ficam em `AGENTS.md` (em inglês); o projeto não usa `CLAUDE.md`.

### Fora da primeira versão

- **WhatsApp e Telegram.** Ficam para uma fase posterior. O WhatsApp não tem API oficial para contas pessoais e as integrações existentes podem partir com atualizações.
- **Git.** Fica para mais tarde, como integração com um servidor Git existente em vez de ser construído de raiz.
- **Contas Google e Microsoft** (OAuth).

### Questões em aberto

- **Modelo de distribuição.** Instalação pelo próprio cliente, alojamento gerido com uma instalação por cliente, ou plataforma multi-empresa. Por decidir depois de a plataforma estar em uso.
- **Tecnologia do chat interno.** Construído de raiz ou sobre um protocolo existente (por exemplo Matrix, o que facilitaria ligar WhatsApp e Telegram mais tarde).
- **Stack técnica** (linguagem, framework, base de dados, armazenamento de ficheiros).
- **Ordem exata das fases** de construção.
- **Língua da interface** e suporte a traduções.
- **Licença** do projeto.

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

**Organization in the data model from day one.** Every account, project, ticket or conversation belongs to an organization, even if only one exists at first.

**Self-hosted, packaged with Docker Compose.**

**Built in phases.** All modules are in the plan, but they are built in order so the platform becomes usable early.

**Languages.** Documentation in Portuguese and English in the same file; code in English. Instructions for AI agents live in `AGENTS.md` (in English); the project does not use `CLAUDE.md`.

### Out of the first version

- **WhatsApp and Telegram.** Deferred to a later phase. WhatsApp has no official API for personal accounts, and existing integrations can break with updates.
- **Git.** Later, as an integration with an existing Git server rather than built from scratch.
- **Google and Microsoft accounts** (OAuth).

### Open questions

- **Distribution model.** Self-installed by the customer, managed hosting with one instance per customer, or a multi-tenant platform. To be decided once the platform is in use.
- **Internal chat technology.** Built from scratch or on an existing protocol (for example Matrix, which would make connecting WhatsApp and Telegram easier later).
- **Tech stack** (language, framework, database, file storage).
- **Exact order of build phases.**
- **Interface language** and translation support.
- **Project license.**
