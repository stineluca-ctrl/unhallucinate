# Context Engineering Guide

How to feed your planning documents into AI coding sessions effectively. Writing great specs is half the work — getting them into AI context correctly is the other half. A spec that never reaches the context window prevents zero bugs.

---

## Why This Matters

AI has no memory between sessions. Every coding session starts fresh. If your DMS is in a folder no one opens, it doesn't exist as far as that session is concerned. The result: AI makes the same decisions it would have made without any docs — UUID vs integer, camelCase vs snake_case, loading state or not.

Context engineering is the practice of deliberately designing what AI sees, in what order, so it generates code that matches your specs instead of its training defaults.

---

## The Attention Curve

AI gives more weight to content at the beginning and end of its context window (U-shaped attention). The middle is where things get buried.

**Implication**:
- Critical rules → TOP of context (constraints, anti-patterns, naming conventions)
- Background information → middle (architecture explanation, entity descriptions)
- Acceptance criteria and examples → END of context (high attention, drives code generation)

Every context file you write should be structured: **rules first, background in the middle, expected output at the end.**

---

## The Two Context Layers

### Layer 1: Always-On (CLAUDE.md or equivalent)
Loaded in every session. Contains the information AI needs regardless of which task it's working on.

**What belongs here:**
- Project glossary (exact domain terms, 1 sentence per term)
- Tech stack summary (framework, DB, auth — 1 line each)
- Architecture pattern (event sourcing? CRUD? monolith? — 2-3 sentences)
- Naming conventions (from CODESTYLE and DMS — the actual rules, not a reference to the doc)
- Key constraints (immutable tables, RLS required, no client-side secrets — the rules AI must never violate)
- Anti-patterns list (copy Section 11 from CODESTYLE verbatim)
- Active open questions (what's currently unresolved — prevents AI from inventing answers)

**What does NOT belong here:**
- Full table definitions (too long, buries everything else)
- Raw meeting notes (unstructured, noisy)
- Acceptance criteria for specific features (belongs in per-task brief)
- Anything that changes task-to-task (it will become stale)

**Target length:** 200-400 lines. Shorter = more of it stays in the "seen" zone per session. If yours is longer, trim aggressively. Every line competes for attention with every other line.

**Structure template:**
```
# Project: [Name]

## Glossary
[Domain terms — one per line, term: definition]

## Tech Stack
Frontend: [framework + version]
Backend: [service/framework]
Database: [engine, schema strategy]
Auth: [provider, method]

## Architecture Pattern
[2-3 sentences on the core pattern and what it constrains]

## Naming Conventions
[Paste from DMS Section 2 and CODESTYLE Section 3]

## Key Constraints (AI Must Follow)
- [Immutability rules]
- [Security rules]
- [Pattern rules — e.g. "all writes go through the events table"]

## Anti-Patterns (Never Generate)
[Paste Section 11 from CODESTYLE verbatim]

## Open Questions (Do Not Guess)
[List unresolved decisions — AI must flag, not invent]
```

---

### Layer 2: Per-Task Brief
Written fresh for each coding task. Contains what AI needs for THIS specific task only.

**Template:**
```
## Task: [Feature name, e.g. "Implement order submission"]

### What to build
[1-3 sentences. Reference the PRD section if applicable.]

### Relevant data model
[Paste ONLY the table definitions from DMS that this task reads or writes]
[Include state machine if any transitions are involved]
[Include FK relationships that matter for this task]

### Business rules for this task
[Copy only the rules from PRD/NFR that apply here]
[Be explicit: "an order cannot be submitted if total is 0"]

### Constraints
- Auth required: [yes/no, which role]
- Error handling: [paste the pattern from CODESTYLE Section 8]
- [Any task-specific constraints]

### Acceptance criteria
[Paste from PRD Section 8 — the specific conditions for this task]
[Format: GIVEN / WHEN / THEN]

### Do NOT do (task-specific)
[Explicit anti-patterns for this specific task beyond the global list]
[e.g. "do not cache this response", "do not use optimistic updates here"]
```

**Target length:** Under 300 lines. If longer, the task is too big — break it into subtasks.

---

## What to Excerpt From Each Doc Type

| Source Doc | What to pull into per-task brief | What stays in CLAUDE.md |
|------------|----------------------------------|------------------------|
| DMS | Only the tables this task reads/writes + relevant state machine | Naming conventions, immutability rules |
| PRD | Specific workflow + acceptance criteria for THIS feature only | Glossary, entity definitions summary |
| TRD | Auth model + error handling pattern (always include) | Architecture pattern, tech stack |
| ARCH | Which feature folder this task belongs to + import rules | Feature boundaries summary |
| CODESTYLE | Anti-patterns (already in CLAUDE.md — don't repeat) | Full anti-patterns list |
| ADR | Only ADRs directly relevant to the decision this task involves | None (reference by number if needed) |
| NFR | Only if this task touches regulated data or has specific performance targets | Compliance summary |

---

## Rules for Effective Per-Task Context

**1. Glossary first.**
Start every task brief with the 5-10 terms most relevant to this task. AI sees this first and uses it to interpret everything that follows.

**2. Constraints before requests.**
Put "must not do X" before "build Y". Constraints that come after the request are frequently ignored. AI has already formed its approach by the time it sees a late constraint.

**3. Paste exact schema, not summaries.**
"The users table has the usual fields" is useless. Paste the actual column definitions from the DMS. AI needs column names, types, and nullability to generate correct code.

**4. Specify the exact file to create.**
Don't say "add authentication." Say "create `src/features/auth/useAuth.ts` that does X." Vague scope produces vague code.

**5. One task per session for complex features.**
Giving AI 5 unrelated tasks in one session means it holds context for all 5 poorly. One focused task per session = better output. Batch only tightly related subtasks.

**6. End with acceptance criteria.**
AI generates better code when it knows what "done" looks like, stated as specific conditions. Put acceptance criteria at the END of the brief — high attention zone.

**7. Negatives are as important as positives.**
"Do not use localStorage" prevents a bug more reliably than three paragraphs on why IndexedDB is better. Explicit negatives override AI's training defaults.

**8. Version-pin dependencies in CLAUDE.md.**
Include: "This project uses React 18.3, not 19." AI defaults to whatever version it saw most in training, which may be outdated.

---

## Connecting Specs to Your AI Workflow

If using GSD or a similar structured AI workflow framework:

| Workflow Artifact | Fed By |
|-------------------|--------|
| CLAUDE.md (always-on) | CODESTYLE Section 11, DMS Section 2, TRD Section 4, project glossary |
| Per-plan CONTEXT.md / PLANNING.md | DMS (relevant tables), PRD (relevant workflow), ADRs (relevant decisions) |
| Task-level brief | PRD acceptance criteria, DMS state machine, NFR constraints |
| Verification step | FM checklist (ai-failure-modes.md), CODESTYLE anti-patterns |

The specs don't replace the workflow framework — they feed it. Specs are the source of truth. The workflow framework is the delivery mechanism.

---

## Diagnosing Context Engineering Failures

When AI generates code that violates a documented spec, the cause is almost always one of these:

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| AI reinvents naming conventions every session | Naming conventions not in CLAUDE.md, or buried mid-file | Move to top of CLAUDE.md |
| AI violates a documented constraint | Constraint came after the task request in context | Move constraints above the task description |
| AI uses wrong library version | Version not specified in CLAUDE.md | Add to tech stack section of CLAUDE.md |
| AI generates no loading/error states | FM-16 anti-pattern not in CLAUDE.md | Add to anti-patterns section |
| AI puts secrets in frontend code | FM-15 rule not in CLAUDE.md | Add "never put secrets in client code" to constraints |
| AI schema doesn't match DMS | DMS not in task brief, or only referenced not pasted | Paste relevant table definitions directly |
| AI generates inconsistent TypeScript | TypeScript rules not in CLAUDE.md | Add strict mode setting and `any` policy |
| AI ignores an ADR | ADR not referenced in task brief | Include relevant ADR number and decision in brief |

---

## Anti-patterns in Context Engineering

Things that seem helpful but aren't:

- **Dumping the entire DMS into every task.** AI attention degrades on long context. Excerpt only what's relevant to the task.
- **"See the PRD for requirements."** AI won't go read the PRD. Paste the relevant section.
- **Putting the anti-patterns list at the end of CLAUDE.md.** They need to be near the top. After the glossary and stack summary at most.
- **Updating CLAUDE.md infrequently.** Stale context is worse than no context — it confidently misleads. Update CLAUDE.md when any spec changes.
- **One giant CLAUDE.md for the whole project.** Split by concern if it exceeds 500 lines: `CLAUDE.md` (core context) + `CLAUDE-data.md` (DMS summary) + `CLAUDE-standards.md` (CODESTYLE excerpts).
