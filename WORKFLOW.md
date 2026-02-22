# unhallucinate: Complete Workflow

Step-by-step guide for using the unhallucinate framework to ship production software with AI.

> **Note:** This workflow uses [GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) as the execution engine because that's what I use. GSD is not required. The spec framework works with any AI coding tool: Cursor, Aider, Claude Code, or whatever fits your workflow. Swap "GSD" for your tool of choice wherever it appears in Phase 5.

---

## Prerequisites

- [Claude](https://claude.ai) with the spec-driven-dev skill installed
- A project folder with at least a rough idea of what you're building
- An AI coding tool for execution (I use [GSD](https://github.com/gsd-build/get-shit-done), but any tool works)

---

## Phase 1: Research (Before Any Docs)

**Goal:** Collect every piece of information that will inform your specs. No formal docs yet.

**What to gather:**
- Stakeholder interviews and meeting notes
- Existing spreadsheets, databases, or workflows being replaced
- Example files (data exports, reports, templates)
- Competitor or reference product screenshots/notes
- Domain terminology from subject matter experts
- Hardware/integration documentation (APIs, devices, file formats)
- Compliance or regulatory requirements

**Where to put it:**
```
project/
├── memory/
│   ├── people/          # Stakeholder profiles, roles, contact info
│   ├── context/         # Domain knowledge, example files, reference material
│   ├── projects/        # Project-level notes, scope, timeline
│   └── meetings/        # Meeting notes, transcripts, decisions made
```

**The rule:** Every answer in your specs should trace back to something in this folder. If you can't cite a source, it's an open question. Write it down as one. Don't invent.

---

## Phase 2: Audit

**Goal:** Understand what documentation you already have and what's missing.

**How to run:**
Tell Claude: *"Audit my project docs"* or *"Check my project readiness"*

**What the audit produces:**

1. **Coverage Matrix** - Maps each of the 12 document types to a coverage score (NONE / PARTIAL / ADEQUATE / COMPLETE) and risk level

2. **AI Decision Audit** - The most important output. For every decision AI would need to make during code generation, it checks whether the answer exists in a spec. Common decisions that must NOT be guessed:
   - ID formats (UUID v4? custom string? auto-increment?)
   - Timestamp handling (timezone, format, storage type)
   - State transitions (which states are valid? which transitions allowed?)
   - Error responses (format, codes, messages)
   - Auth model (who can access what?)
   - Naming conventions (snake_case? camelCase?)
   - Null handling (nullable columns? defaults?)
   - Cascade behavior (delete parent, what happens to children?)

3. **Context Audit** - Checks whether specs are actually reachable by AI during coding sessions. A great DMS in a folder nobody opens prevents zero bugs.

4. **Recommendations** - Ordered action items: which docs to write first, estimated effort.

---

## Phase 3: Generate Planning Documents

**Goal:** Create the specs that will govern code generation.

**How to run:**
Tell Claude: *"Create a [document type] for my project"*

The skill generates docs in dependency order. Always build on the foundation:

```
PRD first (what are we building?)
  → DMS (what does the data look like?)
  → TRD (how does the architecture work?)
    → ARCH, API, CODESTYLE, ADRs (implementation details)
  → NFR (quality requirements)
  → OQR (what don't we know yet?)
```

### Document generation rules

**Tagging is non-negotiable.** Every piece of information falls into one category:

| Tag | Meaning | Action Required |
|-----|---------|----------------|
| *(none)* | Sourced from existing docs | None, it's verified |
| `[INFERRED FROM: source]` | Logically derived | User should verify the reasoning |
| `[ASSUMPTION: reason]` | Reasonable default | User should confirm or override |
| `[DECISION NEEDED]` | Cannot be inferred | Stakeholder must decide (options provided) |
| `[OPEN QUESTION: OQ-XX]` | Tracked in OQR | Resolution needed before coding |

**Things the skill will never invent:**
- Table schemas, column names, or data types
- Performance numbers
- Security/compliance requirements
- State machine transitions
- Business rules
- Integration details
- User roles and permissions
- Field validation rules

If it doesn't know, it says so. Every document ends with a **Missing Information Summary** listing all decisions needed, assumptions made, and open questions referenced. That summary is your action list before coding begins.

### After each document

The skill automatically:
1. Updates `docs/INDEX.md` with section headers and line ranges
2. Checks if CLAUDE.md has a Document Registry section (adds one if missing)
3. Cross-references against existing docs for contradictions

---

## Phase 4: Context Engineering

**Goal:** Make specs reachable by coding AI without dumping everything into context.

### Set up CLAUDE.md

Your CLAUDE.md is the always-on context file. It should contain:

```markdown
# Project: [Name]

## Glossary
[Domain terms, one per line]

## Tech Stack
[Framework, DB, auth, hosting, one line each]

## Architecture Pattern
[2-3 sentences on the core pattern and what it constrains]

## Naming Conventions
[Paste from DMS Section 2 and CODESTYLE Section 3]

## Key Constraints (AI Must Follow)
[Immutability rules, security rules, pattern rules]

## Anti-Patterns (Never Generate)
[Paste Section 11 from CODESTYLE verbatim]

## Open Questions (Do Not Guess)
[Active unresolved decisions]

## Document Registry
[Routing table - see README for format]
```

**Keep it under 400 lines.** Every line competes for attention with every other line.

### Set up docs/INDEX.md

The Section Index maps every section in every planning doc to line ranges. The coding AI reads the index, finds what it needs, pulls only those lines.

```markdown
# Document Section Index

## DMS
| Section | Lines | Contains |
|---------|-------|----------|
| 2. Naming Conventions | 16-45 | Table, column, PK naming rules |
| 4.2 events | 106-145 | Append-only event log |
| 5. State Machines | 400-480 | All entity lifecycle diagrams |
```

The spec-driven-dev skill maintains this automatically after each GENERATE run.

---

## Phase 5: Build

**Goal:** Execute the build with AI that follows your specs.

### How the systems connect

| System | Role |
|--------|------|
| spec-driven-dev | Creates and maintains specs (source of truth) |
| CLAUDE.md | Routes AI to the right specs per task (bridge) |
| Your execution tool | Breaks features into tasks, manages coding (engine) |

### The build loop

1. **AI reads CLAUDE.md** and understands the project structure
2. **You assign a feature or task**
3. **Break it into atomic tasks** based on your specs (your tool may do this automatically)
4. **Each coding session starts fresh**, ideally with a clean context window
5. **AI reads CLAUDE.md** (automatic in Claude Code), sees the Document Registry
6. **AI loads relevant spec sections** via INDEX.md (targeted, not everything)
7. **AI writes code** grounded in spec, not guessing
8. **Verify** against acceptance criteria from the PRD
9. **Commit** per task
10. **Next task** gets a fresh session with fresh context

> **How I do it:** GSD automates steps 3-4 and 8-10. It spins up executor agents with ~200k context tokens each, runs verification loops, and manages atomic commits. But you can do the same loop manually with any coding AI. The Document Registry does the heavy lifting.

### What to watch for

- If AI is making decisions not covered by specs, **stop and update your planning docs first.** The AI guessing is a symptom of missing specs.
- If code contradicts a spec, check context engineering first. The spec might exist but not be reaching the AI. Review the Document Registry routing.
- If open questions are blocking tasks, resolve them in the OQR before continuing. Don't let AI invent answers to questions you haven't decided yet.

### Your execution tool should NOT:
- Write or maintain planning docs (that's spec-driven-dev)
- Restate specs in its own words (it references them directly)
- Make architectural decisions (those are locked in by your ADRs)

---

## Phase 6: Maintain

**Goal:** Keep specs accurate as the project evolves.

### When to update specs

- Stakeholder gives new feedback or changes requirements
- Open question gets resolved
- You discover a gap during coding (new failure mode encountered)
- PRD scope changes

### Update process

1. Update the affected planning doc(s)
2. Check for cascade effects:
   - PRD change → may need DMS, TRD, NFR updates
   - DMS change → may need API, MIGRATE updates
   - TRD change → may need ARCH, CODESTYLE updates
3. Update the OQR (mark resolved questions, add new ones)
4. Re-run `docs/INDEX.md` update (line ranges may have shifted)
5. Next coding session automatically picks up changes through CLAUDE.md

### The maintenance rule

If you're about to tell AI to do something that contradicts a spec, **update the spec first.** Specs are the source of truth. Verbal overrides in a chat session don't persist. The next agent won't know about them.

---

## Quick Reference: All 12 Document Types

| # | Doc | Abbreviation | Priority | What It Locks In |
|---|-----|-------------|----------|-----------------|
| 1 | Product Requirements Document | PRD | ESSENTIAL | Features, scope, workflows, acceptance criteria |
| 2 | Data Model Specification | DMS | ESSENTIAL | Tables, columns, constraints, triggers, state machines |
| 3 | Technical Requirements Document | TRD | ESSENTIAL | Architecture, stack, auth, security, error handling |
| 4 | Non-Functional Requirements | NFR | ESSENTIAL | Performance, security, compliance, audit |
| 5 | Architecture & Module Plan | ARCH | RECOMMENDED | Module boundaries, folder structure, fault isolation |
| 6 | Coding Standards | CODESTYLE | RECOMMENDED | Naming, TypeScript, state management, anti-patterns |
| 7 | Architecture Decision Records | ADR | RECOMMENDED | Individual decisions with reasoning and alternatives |
| 8 | API/Integration Design | API | RECOMMENDED | Endpoints, contracts, error codes |
| 9 | Open Questions Register | OQR | RECOMMENDED | Unresolved decisions, assumptions, blockers |
| 10 | Testing Strategy | TEST | OPTIONAL | Framework, coverage, test data |
| 11 | Deployment Plan | DEPLOY | OPTIONAL | CI/CD, environments, rollback |
| 12 | Data Migration Plan | MIGRATE | OPTIONAL | Import logic, transformations, validation |

Full templates for each: [skill/references/document-templates.md](skill/references/document-templates.md)
