# AGENTS.md

Instructions for AI coding agents working on Maat. This is the single source of agent instructions for the project: do not create a `CLAUDE.md` or any other tool-specific instruction file.

## Project

Maat is a self-hosted platform that unifies email, internal chat, projects, tasks, support tickets, calendar and documents under one login and one layout, with every item linked to the items it came from and the items it became. See [README.md](README.md) and [docs/vision-and-decisions.md](docs/vision-and-decisions.md) before making design decisions.

Current stage: design. The only runnable code is the static prototype in `prototype/index.html`. There is no application code or build system yet. The backend language is TypeScript (Node.js); the rest of the stack (frameworks, ORM, database, storage) is not decided yet. Do not introduce any of it without an explicit decision recorded in `docs/vision-and-decisions.md`.

## Language rules

- **Code is in English**: identifiers, file and folder names, comments, log messages, configuration keys, test names and commit messages.
- **Documentation is bilingual**: every document (README, files in `docs/`) contains a Portuguese section followed by an English section in the same file, with language links at the top. Portuguese is European Portuguese (pt-PT). Keep both sections in sync; a change to one language is not complete until the other is updated.
- **This file is English only.**
- **User-facing interface text** in the prototype is in Portuguese (pt-PT). How the application handles UI languages is an open question; do not hard-code a decision.

## Core design principles

- Everything is an **item** with an origin and links. Converting one item into another creates a new linked item; it never copies or moves the original.
- Every record belongs to an **organization**, even while only one exists.
- Maat is an **email client, not a mail server**. Mail stays on the IMAP server; Maat stores an index and links. Links reference the email's Message-ID, never its folder.
- Integrate existing protocols and services (IMAP/SMTP, CalDAV) instead of rebuilding them.
- Keep account authentication separate from the mail protocol code so OAuth can be added later.

## Out of scope for now

WhatsApp and Telegram integration, Git hosting, Google and Microsoft (OAuth) accounts. Do not add these unless the decision record changes.

## Repository hygiene

- This repository is **public**. Never commit real client data, real email addresses, credentials, tokens, server hostnames or IPs. Use placeholders such as `[Client A]` or `example.pt`.
- Record new decisions and open questions in `docs/vision-and-decisions.md`, in both languages.
- When unsure about a requirement, ask instead of assuming.
