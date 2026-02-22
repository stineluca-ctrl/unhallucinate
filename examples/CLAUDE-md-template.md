# CLAUDE.md Template

Copy this template and fill in your project details. This file is loaded automatically by Claude Code, GSD, and most AI coding tools on every session.

**Target length:** 200-400 lines. Shorter = more stays in the AI's high-attention zone.

---

```markdown
# Project: [Your Project Name]

## Glossary

| Term | Definition |
|------|-----------|
| [term] | [one sentence, precise] |

Keep this to domain-specific terms that AI would confuse or use interchangeably. "User" vs "Account" vs "Profile" distinctions go here.

## Tech Stack

- **Frontend:** [framework + version] (SPA/SSR/static)
- **Backend:** [framework/service]
- **Database:** [engine, hosting]
- **Auth:** [provider, method]
- **Hosting:** [platform]

## Architecture Pattern

[2-3 sentences on the core pattern. Example: "Event sourcing with append-only events table and derived state tables. All writes go through the events table. State tables are updated by Postgres triggers. No direct mutations to state tables."]

## Naming Conventions

### Database
- Tables: snake_case, plural (e.g., `current_samples`)
- Columns: snake_case (e.g., `created_at`)
- Primary keys: [type] (e.g., UUID v4 via `gen_random_uuid()`)
- Foreign keys: `{entity}_id`
- Timestamps: `timestamptz` (UTC always)

### Code
- Files: [convention] (e.g., PascalCase for components, camelCase for hooks)
- Variables/functions: camelCase
- Types/interfaces: PascalCase
- Constants: SCREAMING_SNAKE_CASE
- Booleans: prefix with `is`, `has`, `can`, `should`

## Key Constraints (AI Must Follow)

- [e.g., Events table is INSERT-only. No UPDATE or DELETE ever.]
- [e.g., All state changes go through events. Never mutate state tables directly.]
- [e.g., Client-side PDF parsing only. PII never leaves the browser.]
- [e.g., Soft deletes only. No hard deletes on any user-facing data.]

## Anti-Patterns (Never Generate)

Security:
- Never put API keys, tokens, or secrets in frontend code or committed files
- Never log auth tokens, passwords, or PII to console
- Never use `dangerouslySetInnerHTML` without explicit justification

Code quality:
- No `console.log` in production code (use the project's logging utility)
- No `any` type without a comment explaining why
- No commented-out dead code committed to the repo
- No inline styles in JSX (use Tailwind / CSS modules)

UI completeness:
- Every data-fetching component MUST handle loading, empty, and error states
- Never return null from a component without an explicit empty state
- Never leave a failed API call silent

## Open Questions (Do Not Guess)

- **OQ-XX**: [Question]. [Why it matters]. DO NOT invent an answer.
- **OQ-XX**: [Question]. [Options under consideration].

If you encounter a task that requires answering one of these, flag it and stop. Do not guess.

## Document Registry

Before starting any coding task, read ONLY the docs that match the task.
Always read: CODESTYLE (for any code) + DMS (for any database work).
Read others only when the "Read When" column applies.
Do NOT read all docs on every task.

| Doc | Path | Read When | Index |
|-----|------|-----------|-------|
| DMS | docs/dms.md | Any database, schema, trigger, state machine work | docs/INDEX.md#dms |
| TRD | docs/trd.md | Auth, error handling, integrations, security | docs/INDEX.md#trd |
| CODESTYLE | docs/codestyle.md | Writing any code (always) | docs/INDEX.md#codestyle |
| ARCH | docs/arch.md | Creating files, folder structure decisions | docs/INDEX.md#arch |
| NFR | docs/nfr.md | Performance, compliance, security requirements | docs/INDEX.md#nfr |
| PRD | docs/prd.md | Workflow questions, acceptance criteria, scope | docs/INDEX.md#prd |

## Roles

| Role | Who | Key Permissions |
|------|-----|----------------|
| [role] | [name] | [summary] |
```
