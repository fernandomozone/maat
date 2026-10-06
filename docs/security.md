# Segurança · Security

[Português](#português) · [English](#english)

---

## Português

O padrão de segurança da Maat, decidido a 2026-10-06. Ainda não há código: todos os controlos estão **por fazer**. O documento serve de lista do que cada fase tem de cumprir e provar.

### 1. Referências

- **OWASP ASVS, nível 2** como base. É o nível recomendado para aplicações que guardam dados pessoais e de negócio. A versão em vigor é verificada na fonte oficial quando se começar a mapear requisito a requisito.
- **RGPD** como obrigação: a Maat guarda emails e dados de clientes, que são dados pessoais.
- **NIS2** como alinhamento: a obrigação é das empresas que usam a Maat, e a ferramenta deve ajudá-las a cumprir (rastreabilidade, controlo de acessos, cópias de segurança).

### 2. Como se acompanha

Cada controlo tem um **estado** e uma **prova**:

- **Estado:** por fazer, feito, provado.
- **Prova:** um teste automático (o preferido), uma configuração verificável, ou uma verificação manual descrita passo a passo.

Um controlo só passa a "provado" quando a prova existe e foi vista a falhar ao partir-se o controlo de propósito. Nenhuma fase do [roteiro](roadmap.md) fecha com controlos dessa fase por provar.

### 3. Controlos já decididos

| Área | Controlo | Fase | Estado |
|---|---|---|---|
| Isolamento | RLS com `FORCE` em todas as tabelas; utilizador da aplicação sem posse das tabelas nem `BYPASSRLS`; organização declarada por transação; teste ao catálogo; prova de acesso cruzado entre duas organizações. Ver [architecture.md](architecture.md) | 0 | Por fazer |
| Autenticação | Palavras-passe da Maat com Argon2; MFA por TOTP com códigos de recuperação; obrigatório para administradores; obrigatório para todos se a empresa o decidir | 0 | Por fazer |
| Sessões | Sessão no servidor, identificador opaco em cookie `httpOnly`; "sair" invalida a sessão no servidor | 0 | Por fazer |
| Controlo de acessos | Chefes veem só a sua equipa; caixas pessoais nunca visíveis a terceiros; administrador da plataforma como papel separado | 0–1 | Por fazer |
| Instalação | Assistente protegido por código de instalação mostrado no terminal | 0 | Por fazer |
| Transporte | HTTPS com certificados automáticos no proxy incluído | 0 | Por fazer |
| Auditoria | Registo de todas as alterações de dados e ações de administração e segurança, sem conteúdo de emails ou mensagens; prazo definido pela empresa (1 ano por omissão) | 0 | Por fazer |
| Cópias de segurança | Script com dump consistente e retenção; restauro testado; chave de cifra fora do backup | 0 | Por fazer |
| Credenciais de terceiros | Palavras-passe de email cifradas; chave em segredo do Docker; comando de troca da chave | 1 | Por fazer |
| Email | HTML mostrado em área isolada, sem scripts; imagens externas bloqueadas por omissão | 1 | Por fazer |
| Ficheiros | Servidos pela aplicação depois de verificar a organização; nomes que não se adivinham; limites de tamanho e tipo | 1 e 5 | Por fazer |
| Cadeia de fornecimento | Quatro verificações antes de cada biblioteca; ações de CI fixadas por commit; auditoria de dependências no CI | 0 | Por fazer |

### 4. RGPD

A detalhar antes de a Fase 1 entrar em uso real. Pontos já conhecidos:
- Saber que dados pessoais a Maat guarda, onde e porquê.
- Apagar quando é pedido, incluindo cópias e índices.
- Se a procura usar um índice próprio, o que for apagado no servidor de email é apagado também na Maat.

---

## English

Maat's security standard, decided on 2026-10-06. There is no code yet: every control is **to do**. This document is the list of what each phase must deliver and prove.

### 1. References

- **OWASP ASVS, level 2** as the baseline. It is the level recommended for applications that hold personal and business data. The current version is checked at the official source when requirements are mapped one by one.
- **GDPR** as an obligation: Maat stores emails and client data, which are personal data.
- **NIS2** as alignment: the obligation falls on the companies using Maat, and the tool should help them comply (traceability, access control, backups).

### 2. How it is tracked

Each control has a **status** and a **proof**:

- **Status:** to do, done, proven.
- **Proof:** an automated test (preferred), a verifiable configuration, or a manual check described step by step.

A control only becomes "proven" when the proof exists and has been seen to fail when the control is deliberately broken. No phase of the [roadmap](roadmap.md) closes with that phase's controls unproven.

### 3. Controls already decided

| Area | Control | Phase | Status |
|---|---|---|---|
| Isolation | RLS with `FORCE` on every table; application user that owns no tables and has no `BYPASSRLS`; organization declared per transaction; catalog test; cross-access proof between two organizations. See [architecture.md](architecture.md) | 0 | To do |
| Authentication | Maat passwords hashed with Argon2; TOTP MFA with recovery codes; mandatory for administrators; mandatory for everyone if the company decides so | 0 | To do |
| Sessions | Server-side session, opaque identifier in an `httpOnly` cookie; "log out" invalidates the session on the server | 0 | To do |
| Access control | Heads see only their team; personal mailboxes never visible to others; platform administrator as a separate role | 0–1 | To do |
| Installation | Wizard protected by an installation code shown in the terminal | 0 | To do |
| Transport | HTTPS with automatic certificates on the included proxy | 0 | To do |
| Audit | Log of all data changes and administration and security actions, without email or message content; retention set by the company (1 year by default) | 0 | To do |
| Backups | Script with consistent dump and retention; restore tested; encryption key kept out of the backup | 0 | To do |
| Third-party credentials | Mailbox passwords encrypted; key as a Docker secret; key rotation command | 1 | To do |
| Email | HTML shown in an isolated area, without scripts; remote images blocked by default | 1 | To do |
| Files | Served by the application after checking the organization; unguessable names; size and type limits | 1 and 5 | To do |
| Supply chain | Four checks before each library; CI actions pinned by commit; dependency audit in CI | 0 | To do |

### 4. GDPR

To be detailed before Phase 1 goes into real use. Points already known:
- Know what personal data Maat stores, where and why.
- Delete on request, including copies and indexes.
- If search uses its own index, whatever is deleted on the mail server is also deleted in Maat.
