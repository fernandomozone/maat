# Roteiro · Roadmap

[Português](#português) · [English](#english)

---

## Português

Ordem de construção da primeira versão, decidida a 2026-10-06 ([issue #4](https://github.com/fernandomozone/maat/issues/4)).

### Regras

- **Uma fase de cada vez, por esta ordem.** A fase seguinte só começa quando a anterior estiver feita.
- **"Feito" quer dizer uso real.** Uma fase só está feita quando o dono do projeto a usa no seu trabalho do dia a dia, com as suas contas e dados verdadeiros. Testes com dados de exemplo são necessários, mas não chegam.
- **Cada fase acrescenta pelo menos uma ligação entre módulos**, porque é esse o diferencial da Maat.

### Fase 0: Fundação

- Projeto em Docker Compose, PostgreSQL, integração contínua.
- Organizações, utilizadores, login com sessão no servidor.
- Isolamento entre organizações (RLS), com as provas automáticas descritas em [architecture.md](architecture.md).
- O modelo de item e ligações.
- A estrutura da interface: barra de módulos no topo, tema claro e escuro, procura global.
- Departamentos e equipas, chefes, administrador da plataforma; MFA por TOTP; registo de auditoria.
- Instalação: assistente no browser com código de instalação, proxy com HTTPS automático, scripts de backup e de atualização.

**Feito quando:** a Maat se instala de raiz com o assistente, se entra com MFA, a barra de módulos aparece, um backup é restaurado com sucesso, e a prova de isolamento passa (e falha quando se parte de propósito).

### Fase 1: Email

- **Primeiro passo: a medição da procura** (índice próprio ou pesquisa IMAP, incluindo anexos) com contas reais em Mailcow e cPanel. A decisão fica registada antes de construir a procura.
- O administrador configura as contas IMAP/SMTP de cada pessoa, com credenciais cifradas; caixas partilhadas atribuídas a equipas.
- Processo de sincronização com IDLE.
- Pastas, lista, leitura, anexos, escrever, responder, arquivar.
- Email novo aparece no ecrã sem recarregar.
- HTML isolado; imagens externas bloqueadas, com "confiar neste domínio" por pessoa.
- **Convites recebidos aparecem legíveis** (título, data, hora, local, organizador), em vez de um anexo `.ics` solto.

**Feito quando:** uma conta real é trabalhada só na Maat, sem abrir outro cliente de email.

### Fase 2: Calendário

- Sincronização CalDAV com os calendários que já existem no servidor.
- Vista de semana, dia e mês; criar e editar eventos.
- **Convites completos:** aceitar, recusar ou "talvez"; a resposta segue por email ao organizador e o evento entra no calendário.
- **Ligação:** converter um email em evento.

**Feito quando:** os convites e eventos do dia a dia são tratados só na Maat.

### Fase 3: Tarefas e Projetos

- Lista de tarefas; projetos em kanban.
- **Ligações:** converter um email em tarefa ou projeto, com a origem visível dos dois lados; reservar tempo no calendário para uma tarefa.

**Feito quando:** um email vira tarefa, a tarefa mostra o email, e o email mostra a tarefa, com trabalho real.

### Fase 4: Tickets

- Email convertido em ticket; resposta ao cliente na mesma conversa; notas internas; cronologia.
- **Ligações:** ticket com tarefas, eventos e emails.

**Feito quando:** um pedido de suporte real é tratado do princípio ao fim dentro da Maat.

### Fase 5: Documentos

- Pastas, carregar e pré-visualizar ficheiros.
- **Ligações:** guardar anexos de email; ligar documentos a tickets, tarefas e projetos.

**Feito quando:** os documentos de trabalho deixam de viver em pastas soltas.

### Fase 6: Chat interno

- Matrix sem cifra ponta a ponta ([issue #2](https://github.com/fernandomozone/maat/issues/2)). No início da fase: escolher o servidor Matrix, desenhar o login único e impedir que as apps liguem a cifra por omissão.
- **Ligações:** converter uma mensagem em tarefa ou ticket.

**Feito quando:** a equipa conversa na Maat em vez de noutra aplicação.

---

## English

Build order for the first version, decided on 2026-10-06 ([issue #4](https://github.com/fernandomozone/maat/issues/4)).

### Rules

- **One phase at a time, in this order.** The next phase only starts when the previous one is done.
- **"Done" means real use.** A phase is only done when the project owner uses it in their daily work, with their own accounts and real data. Tests with sample data are necessary but not enough.
- **Every phase adds at least one link between modules**, because that is what sets Maat apart.

### Phase 0: Foundation

- Docker Compose project, PostgreSQL, continuous integration.
- Organizations, users, login with server-side sessions.
- Isolation between organizations (RLS), with the automated proofs described in [architecture.md](architecture.md).
- The item and link model.
- The interface shell: module bar at the top, light and dark themes, global search.
- Departments and teams, heads, platform administrator; TOTP MFA; audit log.
- Installation: browser wizard with an installation code, reverse proxy with automatic HTTPS, backup and update scripts.

**Done when:** Maat installs from scratch with the wizard, you log in with MFA, the module bar shows, a backup is restored successfully, and the isolation proof passes (and fails when deliberately broken).

### Phase 1: Email

- **First step: the search measurement** (own index or IMAP search, including attachments) with real accounts on Mailcow and cPanel. The decision is recorded before building search.
- The administrator sets up each person's IMAP/SMTP accounts, with encrypted credentials; shared mailboxes assigned to teams.
- Sync process with IDLE.
- Folders, list, reading, attachments, compose, reply, archive.
- New mail appears on screen without reloading.
- Isolated HTML; remote images blocked, with per-person "trust this domain".
- **Received invitations are shown readably** (title, date, time, location, organizer) instead of as a bare `.ics` attachment.

**Done when:** a real account is handled only in Maat, without opening another email client.

### Phase 2: Calendar

- CalDAV sync with the calendars already on the server.
- Week, day and month views; create and edit events.
- **Full invitations:** accept, decline or "maybe"; the reply goes by email to the organizer and the event is added to the calendar.
- **Link:** convert an email into an event.

**Done when:** day-to-day invitations and events are handled only in Maat.

### Phase 3: Tasks and Projects

- Task list; projects as kanban.
- **Links:** convert an email into a task or project, with the origin visible on both sides; reserve calendar time for a task.

**Done when:** an email becomes a task, the task shows the email, and the email shows the task, with real work.

### Phase 4: Tickets

- Email converted into a ticket; reply to the customer in the same thread; internal notes; timeline.
- **Links:** ticket with tasks, events and emails.

**Done when:** a real support request is handled from start to finish inside Maat.

### Phase 5: Documents

- Folders, upload and preview files.
- **Links:** save email attachments; link documents to tickets, tasks and projects.

**Done when:** work documents no longer live in loose folders.

### Phase 6: Internal chat

- Matrix without end-to-end encryption ([issue #2](https://github.com/fernandomozone/maat/issues/2)). At the start of the phase: choose the Matrix server, design single sign-on and stop apps from enabling encryption by default.
- **Links:** convert a message into a task or ticket.

**Done when:** the team talks in Maat instead of another app.
