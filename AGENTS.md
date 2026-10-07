# AGENTS.md

Instructions for AI coding agents working on Maat. This is the single source of agent instructions for the project: do not create a `CLAUDE.md` or any other tool-specific instruction file.

## Project

Maat is a self-hosted platform that unifies email, internal chat, projects, tasks, support tickets, calendar and documents under one login and one layout, with every item linked to the items it came from and the items it became. See [README.md](README.md) and [docs/vision-and-decisions.md](docs/vision-and-decisions.md) before making design decisions.

Current stage: design. The only runnable code is the static prototype in `prototype/index.html`. There is no application code or build system yet. The stack is decided and described in [docs/architecture.md](docs/architecture.md); items listed there as "still open" are not decided. Do not introduce any dependency, service or tool outside that document without an explicit decision recorded in it.

Build order: follow the phases in [docs/roadmap.md](docs/roadmap.md), one at a time. Do not start work belonging to a later phase. Build only from approved parts of the specification ([docs/spec/README.md](docs/spec/README.md)); keep that index's status column up to date.

## Language rules

- **Code is in English**: identifiers, file and folder names, comments, log messages, configuration keys, test names and commit messages.
- **Documentation is bilingual**: every document (README, files in `docs/`) contains a Portuguese section followed by an English section in the same file, with language links at the top. Portuguese is European Portuguese (pt-PT). Keep both sections in sync; a change to one language is not complete until the other is updated.
- **This file is English only.**
- **User-facing interface text** is never hard-coded: every string lives in the translation files and code references it by key (keys in English). pt-PT is the only complete language and the fallback; English and possibly pt-BR come later. Dates, times, numbers and time zones are formatted with the user's locale settings.

## Core design principles

- **Organization isolation is enforced by PostgreSQL Row Level Security.** Every data table has an organization column and a policy, with `FORCE ROW LEVEL SECURITY`. All data access goes through the single access function that declares the organization per transaction. Never set it per connection, never take it from the request.

- Everything is an **item** with an origin and links. Converting one item into another creates a new linked item; it never copies or moves the original.
- Every record belongs to an **organization**, even while only one exists.
- Maat is an **email client, not a mail server**. Mail stays on the IMAP server; Maat stores an index and links. Links reference the email's Message-ID, never its folder.
- Integrate existing protocols and services (IMAP/SMTP, CalDAV, Matrix) instead of rebuilding them.
- Internal chat runs on a self-hosted Matrix server **without end-to-end encryption**; Maat is a Matrix client that stores the index and links. Server choice and login integration are decided at the start of Phase 6.
- Keep account authentication separate from the mail protocol code so OAuth can be added later.

## Users and security

- Read [docs/security.md](docs/security.md) before touching authentication, sessions, access control, RLS, credentials, files, audit or backups. A phase does not close while its controls there are unproven.
- One account belongs to one organization for now, but keep **person** and **organization membership** as separate tables so multi-organization membership can be added later without a schema rework. Nothing ties an organization to a single email domain. Design the API so scoped, revocable, expiring API keys can be added later. Departments and teams up to two levels; team heads see their team's work and shared mailboxes, never personal mailboxes.
- Never write email or message content to the audit log or to application logs.

## Out of scope for now

- Billing, Stripe, plan limits and public sign-up belong to the hosted-service phase after the first version. When built, they live in a separate module disabled by default; the core must never depend on them.

WhatsApp and Telegram integration, Git hosting, Google and Microsoft (OAuth) accounts, Active Directory/LDAP, passkeys, one account across several organizations. Do not add these unless the decision record changes.

## Repository hygiene

- This repository is **public**. Never commit real client data, real email addresses, credentials, tokens, server hostnames or IPs. Use placeholders such as `[Client A]` or `example.pt`.
- Record new decisions and open questions in `docs/vision-and-decisions.md`, in both languages.
- When unsure about a requirement, ask instead of assuming.
- This repository is self-contained. Do not reference, link to or copy code or documents from other projects, including private ones; rewrite patterns from scratch, in English.

## License

- The project is licensed under **AGPL-3.0-only** ([LICENSE](LICENSE)). Every new source file starts with the header `// SPDX-License-Identifier: AGPL-3.0-only` (or the equivalent comment syntax).
- Before adding a dependency, check that its license is compatible with AGPL-3.0-only (permissive licenses such as MIT, BSD, Apache-2.0, ISC and MPL-2.0 are fine). Flag anything else, and any proprietary or "source-available" license, instead of adding it.
- Do not accept or merge external contributions until a contributor agreement policy is decided.

## Workflow

- Work happens both in the cloud and on the owner's machine. **Never commit directly to `main`.** Each task gets its own branch and reaches `main` through a pull request.
- A pull request **merges automatically once all CI checks pass**, with one exception: a pull request that adds or upgrades a dependency waits for the owner's explicit approval.
- Always pull the latest `main` before starting a branch.
- Local development ports (fixed, so they never collide with other local projects): backend **3002**, frontend **5175**, PostgreSQL **5434**.

## Engineering practices

- Always use the latest **stable** version of each dependency, verified at the official source at install time, never from memory.
- Before adding a library, check at the source: who maintains it, how many dependencies it pulls in, its security advisory history, and who depends on it.
- **Every dependency needs the project owner's explicit approval before it is installed.** Phase 0 starts from an approved initial list; any library added after that (runtime or development) is proposed first, with the four checks and its license, and installed only once approved. Record approved dependencies and versions in `docs/architecture.md`.
- A test that has never failed proves nothing: after it passes, deliberately break what it should catch and confirm it fails for that reason. When there are two defenses, test each one on its own.
- Pin CI actions by commit SHA, not by tag.
- Run Biome (format and lint) before every commit; CI fails on any Biome error.
- Every Zod array schema for external input has `.max()`, and Fastify limits the request body size (open Zod advisory of 2026-10-01). Never run Fastify below 5.12.5.
- Comments explain why, not what.
