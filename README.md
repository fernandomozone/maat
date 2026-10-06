# Maat

[Português](#português) · [English](#english)

---

## Português

Uma plataforma self-hosted que junta, no mesmo login e no mesmo layout, o email, o chat interno, os projetos, as tarefas, os tickets de suporte, o calendário e os documentos de uma pequena empresa. E, mais importante do que juntar, **liga** tudo: um email pode virar um ticket, uma conversa pode virar uma tarefa, um anexo pode ser guardado nos documentos, e cada item mostra sempre de onde veio e em que se transformou.

> Estado: fase de conceção. Para já há um protótipo interativo e o registo das decisões tomadas. Ainda não há código da aplicação.

### Porquê "Maat"

Maat (também escrito *Ma'at*) era, no antigo Egito, a deusa e o princípio da **ordem, do equilíbrio e da verdade**. O seu oposto era *isfet*: o caos, a desordem. Para os egípcios, manter Maat era uma tarefa de todos os dias, não algo que se conquistava uma vez.

Era representada com uma pena de avestruz na cabeça. Segundo a tradição, depois da morte, o coração de cada pessoa era pesado numa balança contra essa pena, para ver se tinha vivido em equilíbrio.

O nome foi escolhido porque descreve o problema que este projeto quer resolver. O trabalho de quem gere uma pequena empresa de serviços está espalhado: emails num programa, mensagens noutro, tarefas num terceiro, calendário noutro sítio, ficheiros em pastas soltas. Nada fala com nada e é fácil perder o fio. A Maat quer pôr ordem nesse caos, com tudo num só sítio e tudo ligado.

### O que vai ter

| Módulo | Ideia |
|---|---|
| Email | Contas externas ligadas por IMAP/SMTP (cPanel, Mailcow e afins). O email continua no servidor; a Maat guarda o índice e as ligações. |
| Chat | Comunicação interna da equipa, com canais e mensagens diretas. |
| Projetos | Kanban, roadmap e lista. Cada cartão mostra a sua origem. |
| Tarefas | Lista pessoal e da equipa, com prazos, checklist e comentários. |
| Tickets | Suporte a clientes, com cronologia completa e resposta por email na mesma conversa. |
| Calendário | Sincronizado por CalDAV com o que a empresa já tem no servidor. |
| Documentos | Ficheiros partilhados (PDFs, etc.), organizados por organização, projeto e cliente. |

O conceito central: tudo é um **item** com origem e ligações. Converter um email num ticket não copia nada; cria um item novo ligado ao original.

### Protótipo

Em [`prototype/index.html`](prototype/index.html) há um protótipo interativo com os sete módulos e dados fictícios. Basta abrir o ficheiro no navegador. As alterações ficam guardadas no próprio navegador, e o botão "Repor demo" volta ao início. A interface do protótipo está em português.

### Documentação

- [Visão e decisões](docs/vision-and-decisions.md): âmbito da primeira versão, decisões tomadas e questões em aberto.
- [Roteiro](docs/roadmap.md): as fases de construção e o que conta como feito em cada uma.
- [Arquitetura](docs/architecture.md): a stack, o isolamento entre organizações e as boas práticas.
- [Levantamento da stack técnica](docs/tech-stack-research.md): o que existe para reaproveitar, módulo a módulo, em cada linguagem candidata.
- [AGENTS.md](AGENTS.md): convenções para agentes de IA que trabalhem no projeto (em inglês).

---

## English

A self-hosted platform that brings a small company's email, internal chat, projects, tasks, support tickets, calendar and documents together under one login and one layout. More important than bringing them together, it **links** everything: an email can become a ticket, a conversation can become a task, an attachment can be saved to documents, and every item always shows where it came from and what it turned into.

> Status: design phase. For now there is an interactive prototype and a record of the decisions made. There is no application code yet.

### Why "Maat"

Maat (also written *Ma'at*) was, in ancient Egypt, the goddess and principle of **order, balance and truth**. Her opposite was *isfet*: chaos, disorder. For the Egyptians, upholding Maat was an everyday task, not something achieved once.

She was depicted with an ostrich feather on her head. According to tradition, after death each person's heart was weighed on a scale against that feather, to see whether they had lived in balance.

The name was chosen because it describes the problem this project wants to solve. The work of someone running a small services company is scattered: emails in one program, messages in another, tasks in a third, the calendar somewhere else, files in loose folders. Nothing talks to anything, and it is easy to lose track. Maat aims to bring order to that chaos, with everything in one place and everything linked.

### What it will include

| Module | Idea |
|---|---|
| Email | External accounts connected via IMAP/SMTP (cPanel, Mailcow and similar). Email stays on the server; Maat keeps the index and the links. |
| Chat | Internal team communication, with channels and direct messages. |
| Projects | Kanban, roadmap and list. Each card shows its origin. |
| Tasks | Personal and team lists, with due dates, checklists and comments. |
| Tickets | Customer support, with a full timeline and replies sent by email in the same thread. |
| Calendar | Synced via CalDAV with what the company already has on its server. |
| Documents | Shared files (PDFs, etc.), organized by organization, project and client. |

The core concept: everything is an **item** with an origin and links. Converting an email into a ticket copies nothing; it creates a new item linked to the original.

### Prototype

[`prototype/index.html`](prototype/index.html) is an interactive prototype with all seven modules and fictional data. Just open the file in a browser. Changes are saved in the browser itself, and the "Repor demo" button resets everything. The prototype's interface is in Portuguese.

### Documentation

- [Vision and decisions](docs/vision-and-decisions.md): scope of the first version, decisions made and open questions.
- [Roadmap](docs/roadmap.md): the build phases and what counts as done in each.
- [Architecture](docs/architecture.md): the stack, isolation between organizations and good practices.
- [Tech stack research](docs/tech-stack-research.md): what exists to reuse, module by module, in each candidate language.
- [AGENTS.md](AGENTS.md): conventions for AI agents working on the project.
