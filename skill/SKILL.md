---
name: spec-driven-dev
description: >
  Audit, generate, and maintain software planning documents that prevent AI hallucinations during development.
  Use this skill whenever: starting a new project, about to begin coding, reviewing project readiness,
  the user says "check my docs", "what docs do I need", "audit my project", "create a TRD",
  "write a spec", "planning documents", "project setup", "architecture review", or any variation of
  "am I ready to build this". Also trigger when a user mentions they're vibe-coding, using AI to build,
  or worried about code quality from AI-generated code. This skill is about PLANNING documents,
  not feature specs (use feature-spec for individual features).
---

# Spec-Driven Development

You are helping a developer (likely solo or small team, likely using AI to generate code) ensure their project has the planning documents needed to build reliable software. Your job is to prevent the specific failure modes that occur when AI generates code without proper specifications.

## Why This Exists

AI code generation fails in predictable ways when specs are missing:
- Type mismatches (string IDs in one place, numeric in another)
- Mutability violations (UPDATE on an append-only table)
- Security shortcuts (no auth, no RLS, plaintext storage)
- Timezone bugs (assumed UTC, actual timezone not documented)
- Missing state transitions (what happens when an entity moves between lifecycle states?)
- Concurrency gaps (two users perform the same write operation simultaneously)
- Error recovery holes (database goes down mid-transaction, then what?)

These aren't theoretical. They're the top causes of rework in AI-assisted projects. Good specs eliminate 60-80% of these bugs before a single line of code is written.

## Two Modes

### MODE 1: AUDIT
Check existing project docs against the document registry. Output a gap analysis showing what's covered, what's missing, and what's at risk.

### MODE 2: GENERATE
Create missing documents from templates, populated with project-specific details.

Always start with AUDIT unless the user explicitly asks to generate a specific document.

---

## Document Registry

These are the documents a production software project needs. Not all are required for every project. The registry is ordered by criticality for AI-assisted development.

| Priority | Document | Abbreviation | What It Locks In | Skip Risk |
|----------|----------|-------------|------------------|-----------|
| **ESSENTIAL** | Product Requirements Document | PRD | Features, scope, user workflows, acceptance criteria | Scope creep, AI builds wrong features |
| **ESSENTIAL** | Data Model Specification | DMS | Tables, columns, constraints, triggers, state machines | Schema bugs, state corruption, broken event sourcing |
| **ESSENTIAL** | Technical Requirements Document | TRD | Architecture, stack, integrations, security model, error handling | Wrong patterns, security holes, integration failures |
| **ESSENTIAL** | Non-Functional Requirements | NFR | Performance, security, compliance, audit, data retention | Audit trail missing, security gaps, compliance failures |
| **RECOMMENDED** | Architecture & Module Plan | ARCH | Module boundaries, fault isolation, communication rules | Spaghetti code, cascade failures, tight coupling |
| **RECOMMENDED** | Coding Standards | CODESTYLE | File naming, TypeScript rules, state management pattern, anti-patterns, error handling code patterns | AI generates inconsistent conventions across sessions, exposes secrets, omits UI states |
| **RECOMMENDED** | Architecture Decision Records | ADR | Individual high-stakes decisions with reasoning | Future changes violate past decisions |
| **RECOMMENDED** | API/Integration Design | API | Endpoints, request/response contracts, error codes | Frontend-backend mismatch, broken integrations |
| **RECOMMENDED** | Open Questions Register | OQR | Unresolved decisions, assumptions, blockers | Late discovery of scope issues |
| **OPTIONAL** | Testing Strategy | TEST | Test framework, coverage targets, test data | Untested edge cases, regression bugs |
| **OPTIONAL** | Deployment Plan | DEPLOY | CI/CD, environments, rollback strategy | Broken deployments, no rollback path |
| **OPTIONAL** | Data Migration Plan | MIGRATE | Import logic, transformation rules, validation | Data corruption during import |

Read `references/document-templates.md` for the full template of each document type.
Read `references/ai-failure-modes.md` for the specific AI hallucination patterns each document prevents.
Read `references/context-engineering.md` for how to feed these documents into AI coding sessions effectively — specs that never reach the context window don't prevent bugs.
Read `references/context-loading-runtime.md` for how to set up the Document Registry and Section Index so the coding AI loads only relevant context per task — prevents context window overload during implementation.

---

## AUDIT Workflow

When auditing a project, follow these steps:

### Step 1: Discover
Scan the project folder for existing documentation. Look for:
- .md files in root and subdirectories
- CLAUDE.md or similar memory files
- /memory, /docs, /specs, /architecture directories
- Meeting notes, transcripts
- Existing PRDs, TRDs, schema files
- SQL migration files
- OpenAPI/Swagger specs
- ADR directories

### Step 2: Classify
Map each discovered document to the Document Registry. A single document may cover multiple registry items (e.g., a PRD that includes data model sections). Track coverage at the SECTION level, not just the document level.

### Step 3: Gap Analysis
For each registry item, assess:

**Coverage score** (per document type):
- NONE: Document doesn't exist
- PARTIAL: Document exists but missing critical sections
- ADEQUATE: Core sections present, some gaps in detail
- COMPLETE: All sections present with sufficient detail for AI code generation

**Risk assessment** (what breaks if this gap isn't filled):
- CRITICAL: AI will generate wrong code that's hard to detect
- HIGH: AI will make assumptions that cause rework
- MEDIUM: Some manual correction needed but catchable in review
- LOW: Nice to have, won't cause major issues

### Step 4: AI Decision Audit
This is the most important step. Read `references/ai-failure-modes.md` and check each failure mode against the existing docs. For every decision that AI would need to make during code generation, verify that the answer exists in a spec somewhere.

Common decisions AI must NOT be left to guess:
- ID formats (UUID v4? custom format? auto-increment?)
- Timestamp handling (timezone, format, storage type)
- State transitions (which states are valid? which transitions are allowed?)
- Error responses (format, codes, messages)
- Auth model (who can access what?)
- Naming conventions (snake_case? camelCase? for tables, columns, variables, endpoints)
- Null handling (nullable columns? default values? required fields?)
- Cascade behavior (delete a parent entity, what happens to its children?)
- Concurrency model (optimistic locking? pessimistic? event sourcing?)
- Idempotency (can the same event fire twice safely?)

### Step 5: Context Audit
Writing specs is half the work. Verify they're actually reachable by AI during coding sessions:

- Is there a CLAUDE.md or equivalent always-on context file?
- Does it include: project glossary, tech stack, naming conventions, key constraints, anti-patterns?
- Is there a CODESTYLE document? Are its anti-patterns excerpted into CLAUDE.md?
- Is there a per-task brief template or pattern (GSD PLANNING.md, CONTEXT.md, or similar)?
- Are the full spec docs (DMS, TRD) accessible to reference during coding, or buried in a folder no one opens?

Flag "CONTEXT GAP" for any spec that exists but has no path into AI context.

### Step 6: Report
Generate the audit report with this structure:

```
# Project Documentation Audit

## Summary
- Documents found: X
- Registry coverage: X/12 document types
- Overall readiness: [NOT READY / PARTIALLY READY / READY TO BUILD]

## Coverage Matrix
[Table mapping each registry item to coverage score and risk]

## Critical Gaps
[Ordered list of missing specs that will cause the most AI hallucinations]

## AI Decision Gaps
[Specific decisions that have no documented answer - these WILL be hallucinated]

## Context Gaps
[Specs that exist but have no path into AI context — invisible specs don't help]

## Recommendations
[Ordered action items: which docs to write first, estimated effort]
```

---

## GENERATE Workflow

When generating a document, follow these steps:

### Step 1: Choose Document Type
If the user hasn't specified, recommend based on the audit results (highest-risk gaps first).

### Step 2: Gather Context
Before writing, collect:
- Existing docs (read them all, don't duplicate what's already specified)
- CLAUDE.md or memory files (project context, decisions, terminology)
- Meeting notes (stakeholder preferences, domain knowledge)
- Open questions (what's unresolved? flag it, don't guess)

### Step 3: Identify What You Don't Know (BEFORE Drafting)

This step exists because the number one failure mode in AI-generated specs is filling gaps with plausible-sounding fiction. A wrong spec is worse than a missing spec, because wrong specs generate wrong code with confidence.

After gathering context, build a **knowledge inventory** for the document you're about to write. For every section in the template, categorize each required piece of information:

**KNOWN** = explicitly stated in existing docs, meeting notes, or user input. You can cite the source.
**INFERRABLE** = not stated directly but logically follows from known facts. Mark with `[INFERRED FROM: source]`.
**UNKNOWN** = not in any source. You cannot write this without guessing.

If the UNKNOWN list has more than 3 items, STOP and ask the user before drafting. Use the AskUserQuestion tool to present the unknowns as specific, answerable questions. Group related unknowns into a single question where possible.

If the UNKNOWN list has 1-3 items, you may draft with those sections marked `[DECISION NEEDED]`, but call them out prominently at the top of the document.

If the UNKNOWN list is empty, proceed to drafting.

**Examples of what counts as UNKNOWN (never invent these):**
- Database column types not specified anywhere
- Performance targets not mentioned by anyone
- Auth method not confirmed by stakeholders
- State transitions not described in meeting notes
- Error handling behavior not discussed
- Field names, ID formats, or naming conventions not established
- Integration protocols (API, Bluetooth, USB) not documented
- Compliance requirements not confirmed
- Any number, metric, or threshold (response times, user counts, data volumes)

**Examples of what counts as INFERRABLE:**
- If the PRD says "Supabase" then auth is likely Supabase Auth (but mark as inferred)
- If event sourcing is chosen, events table is append-only (follows from the pattern)
- If the team uses React, frontend state uses hooks (standard practice)

The difference: INFERRABLE has a logical chain from a known fact. UNKNOWN is a gap where you'd be making something up to fill space.

### Step 4: Draft

Use the template from `references/document-templates.md`. Fill in project-specific details from gathered context.

**Tagging rules (non-negotiable):**

Every piece of information in the document must come from one of these sources:
- **Direct source**: Stated in an existing doc, meeting note, or user message. No tag needed.
- **`[INFERRED FROM: source]`**: Logically derived from a known fact. Tag it so the user can verify the reasoning.
- **`[ASSUMPTION: reason]`**: A reasonable default where the user hasn't specified. The user should confirm or override.
- **`[DECISION NEEDED]`**: A stakeholder decision that cannot be inferred. Include 2-3 options with trade-offs so the user can decide quickly.
- **`[OPEN QUESTION: OQ-XX]`**: Maps to a known open question from the OQR. Reference the OQ number.

**What to do when you hit an unknown mid-draft:**
1. Do NOT invent a plausible answer. Write `[DECISION NEEDED]` with context.
2. If the unknown blocks multiple downstream sections, stop drafting and ask the user.
3. If you realize the document requires prerequisite docs that don't exist (e.g., writing a DMS without a PRD), warn the user and ask how to proceed.

**Things that are NEVER acceptable to invent:**
- Table schemas, column names, or data types (ask or mark unknown)
- Performance numbers (response times, user counts, data volumes)
- Security/compliance requirements (HIPAA, 21 CFR Part 11, audit depth)
- State machine transitions (which states exist, which transitions are valid)
- Business rules (how many items per order, who can approve what)
- Integration details (API endpoints, protocols, auth methods for external systems)
- User roles and permissions
- Field validation rules (min/max, format, required/optional)

### Step 5: Cross-Reference
After drafting, verify:
- Does this doc contradict anything in existing docs?
- Are all terms consistent with the project glossary?
- Are all entity names, field names, and conventions consistent?
- Does every `[DECISION NEEDED]` have enough context for the stakeholder to decide?
- Are there any unmarked assumptions? (Read the draft with skepticism. If a statement isn't sourced from existing docs, it needs a tag.)

### Step 6: Missing Info Summary

After the document is complete, append a **Missing Information Summary** at the bottom:

```
## Missing Information Summary

### Decisions Needed (X items)
These require stakeholder input before this spec can be used for code generation:
1. [DECISION NEEDED] Section X.X: [question] — Options: A, B, C
2. ...

### Assumptions Made (X items)
These are reasonable defaults. Confirm or override:
1. [ASSUMPTION] Section X.X: [what was assumed and why]
2. ...

### Open Questions Referenced (X items)
These map to existing open questions:
1. [OQ-XX] Section X.X: [brief description]
2. ...

### Prerequisite Gaps
Documents or information that should exist before this spec is finalized:
1. [what's missing and why it matters]
```

This summary is the user's action list. It tells them exactly what to resolve before code generation begins. Present it clearly when sharing the document.

### Step 7: Present
Save the document and summarize:
- What was created
- How many decisions need user input (with the specific questions)
- How many assumptions were made
- What prerequisite gaps exist
- Recommended next action

### Step 8: Update Section Index

After generating or updating any planning doc, update `docs/INDEX.md` (create it if it doesn't exist). This file allows the coding AI to load only the specific sections it needs per task instead of reading entire documents. See `references/context-loading-runtime.md` for the full pattern.

For the newly created/updated document:
1. Scan for all `##` and `###` section headers
2. Record each header's line number and a brief description of what it contains
3. Add or update ONLY this doc's section in `docs/INDEX.md`. Do NOT replace or remove entries for other docs. Each doc gets its own `## DocName` section in the index. If the index file already exists with entries for other docs, append or update only the section for the doc you just created/modified.

Also check: does CLAUDE.md have a Document Registry section? If not, add one following the template in `references/context-loading-runtime.md`. This registry tells the coding AI which docs to read for which tasks.

---

## Document Interdependencies

Documents build on each other. Respect this order when generating:

```
PRD (what)
 ├── DMS (data model derived from PRD entities)
 ├── TRD (architecture to support PRD requirements)
 │    ├── ARCH (code organization from TRD architecture)
 │    │    └── CODESTYLE (conventions AI must follow per coding session)
 │    ├── API (contracts from TRD architecture)
 │    └── ADRs (decisions made during TRD)
 ├── NFR (quality attributes for PRD features)
 └── OQR (gaps found during any doc creation)
      ├── TEST (strategy from PRD acceptance criteria + TRD architecture)
      ├── DEPLOY (plan from TRD architecture)
      └── MIGRATE (plan from DMS + existing data)
```

CODESTYLE is the document that travels with every coding session. Once written, excerpt the anti-patterns list and naming conventions into CLAUDE.md or equivalent context file so AI sees them on every task.

If generating a DMS without a PRD, warn the user. If generating an API spec without a TRD, warn the user. Always build on the foundation.

---

## Calibrating for Project Size

Not every project needs all 12 documents at full depth. Calibrate based on:

**Solo dev + AI, MVP/prototype (< 3 months)**
- ESSENTIAL: PRD (light), DMS (full), TRD (light)
- RECOMMENDED: ADRs for the 3-5 biggest decisions
- Skip: formal API spec (use inline comments), deployment plan, testing strategy doc

**Solo dev + AI, production system (3-12 months)**
- ESSENTIAL: PRD, DMS, TRD, NFR (all full)
- RECOMMENDED: ADRs, API spec, OQR
- Consider: Testing strategy, deployment plan

**Small team (2-5 devs)**
- All ESSENTIAL and RECOMMENDED at full depth
- Testing strategy and deployment plan become essential (team coordination)

**The rule**: If AI is generating code, the DMS and TRD are always essential regardless of project size. CODESTYLE is always recommended regardless of project size — convention drift compounds across every session without it. These three documents prevent the most hallucinations per hour of writing.

---

## Updating Documents

Specs are living documents. When the user asks to update docs based on new information (meeting notes, stakeholder feedback, resolved open questions):

1. Identify which documents are affected
2. Show the specific sections that need changes
3. Make the changes with clear diff visibility
4. Update the OQR (mark resolved questions, add new ones)
5. Check for cascade effects (a PRD change may require DMS and TRD updates)

---

## Key Principles

1. **Never invent what you don't know.** A wrong spec is worse than a missing spec. Wrong specs generate wrong code with confidence. If you don't have the answer, ask the user or mark it `[DECISION NEEDED]`. This is the single most important rule in this skill.

2. **Specs exist to prevent AI hallucinations, not for bureaucratic completeness.** Every section should answer a question that AI would otherwise guess wrong.

3. **Explicit beats implicit.** If a decision isn't written down, AI will make it for you. You probably won't like the result.

4. **Mark unknowns, don't hide them.** `[DECISION NEEDED]` is infinitely better than a silent wrong assumption buried in generated code.

5. **State machines prevent state bugs.** Every entity with a lifecycle needs a state transition diagram. No exceptions.

6. **The glossary is the cheapest insurance.** Term confusion between similar domain concepts causes bugs that are expensive to find.

7. **Write for the AI that will read it.** Clear, unambiguous, with examples. If a human would need to ask a follow-up question, AI will just guess instead.

8. **When in doubt, ask.** Use the AskUserQuestion tool to gather missing info. A 30-second question saves hours of rework from a wrong assumption. Group related questions to minimize interruptions (max 3-4 questions at a time).

9. **Source everything.** If you write "the system uses UUID v4" in a spec, you should be able to point to where in the existing docs that decision was made. If you can't, tag it.
