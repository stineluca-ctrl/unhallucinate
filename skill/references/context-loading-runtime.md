# Context Loading at Runtime

How the **coding AI** should load project specs during implementation tasks. This is separate from context-engineering.md (which covers how to write and structure specs). This doc covers how specs get consumed efficiently at task time.

## The Problem

You wrote 8+ detailed planning docs. If the coding AI reads all of them on every task, you get:
- Context window overload (attention dilution in the middle)
- Slower task startup
- AI losing focus on the actual task buried under thousands of lines of spec

If the coding AI reads none of them, you get hallucinated schemas, wrong naming conventions, and invented state machines.

## The Solution: Document Registry + Section Index

Two components work together:

### 1. Document Registry (lives in CLAUDE.md)

A routing table that tells the coding AI WHICH docs to read for WHICH tasks. Added to CLAUDE.md so the AI sees it on every session.

```markdown
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
| PRD | docs/prd.md | Workflow questions, acceptance criteria, scope | (read full, or grep for section) |
```

### 2. Section Index (lives in docs/INDEX.md)

A single file the AI reads INSTEAD of opening every doc. Contains section headers with brief descriptions so the AI knows exactly which section of which doc to read. The AI reads the index, finds the relevant section, then uses `Read` with offset/limit to pull only that section from the source doc.

```markdown
# Document Section Index

Use this to find the exact section you need without reading entire docs.
Format: [Doc] > [Section] > [Line Range] > [What It Contains]

## DMS

| Section | Lines | Contains |
|---------|-------|----------|
| 1. Overview | 1-15 | Architecture pattern summary |
| 2. Naming Conventions | 16-45 | Table, column, PK, FK, timestamp, enum naming rules |
| 3. Entity-Relationship Diagram | 46-80 | Full ER diagram |
| 4.1 profiles | 81-105 | Auth/user profile table |
| 4.2 events | 106-145 | Append-only event log (CORE - read for any event work) |
| 4.3 current_samples | 146-190 | Sample state table |
| 4.4 current_vials | 191-230 | Vial state table |
| ... | ... | ... |
| 5. State Machines | 400-480 | All entity lifecycle diagrams |
| 6. Trigger Functions | 481-560 | handle_new_event() pseudocode |

## TRD

| Section | Lines | Contains |
|---------|-------|----------|
| 4. Auth and Authorization | 80-120 | Supabase Auth, roles, RLS summary |
| 6. Error Handling | 150-200 | Error response format, trigger failures |
| 7. Integrations | 201-250 | Label printer, PDF, barcode scanner |
| 8. Timestamp Strategy | 251-270 | UTC storage, display timezone |

## CODESTYLE

| Section | Lines | Contains |
|---------|-------|----------|
| 2. File Naming | 10-25 | Component, hook, util naming rules |
| 4. TypeScript Rules | 40-65 | Strict mode, any policy, type generation |
| 7. State Management | 100-130 | Server state, client state, form state |
| 8. Error Patterns | 131-180 | Actual code patterns (paste into task) |
| 11. Anti-Patterns | 200-250 | Never-generate list |
```

## How the AI Uses This

**Step 1:** AI reads CLAUDE.md (automatic, always-on). Sees the Document Registry.

**Step 2:** AI determines which docs are relevant to the current task based on "Read When" column.

**Step 3:** AI reads `docs/INDEX.md` to find the specific sections needed.

**Step 4:** AI uses `Read` with offset/limit to pull only those sections from the source docs.

Example: Task is "add a new trigger for vial state updates."
- Registry says: read DMS (database work) + CODESTYLE (any code)
- Index says: DMS Section 4.4 (vials, lines 191-230) + Section 6 (triggers, lines 481-560) + CODESTYLE Section 11 (anti-patterns, lines 200-250)
- AI reads ~150 lines of spec instead of 1500+

## Maintaining the Index

The INDEX.md file must be updated whenever a source doc changes. This is the tradeoff: you maintain one extra file, but every coding session loads 10x less context.

**When generating or updating any planning doc via the spec-driven-dev skill, the GENERATE workflow should include a final step: update docs/INDEX.md with the new doc's section headers and line ranges.**

If line ranges become stale (doc was edited externally), the AI can fall back to grep for the section header and read from there. Line ranges are optimization, not hard dependencies.

## Generating the Index

After creating a planning doc, generate its INDEX.md entry with this approach:

```python
# Pseudocode for generating index entries
for each doc in [DMS, TRD, CODESTYLE, ARCH, NFR]:
    scan for ## and ### headers
    record header text + starting line number
    estimate section end (next header or EOF)
    write to INDEX.md with brief description of section contents
```

The spec-driven-dev skill should run this as a final step after GENERATE mode completes.

## Fallback When Index Is Missing

If docs/INDEX.md doesn't exist yet, the AI falls back to:
1. Read the Document Registry in CLAUDE.md
2. Read the full relevant doc(s) based on "Read When" column
3. This is less efficient but still better than reading everything

The index is an optimization layer, not a hard requirement. The system works without it, just less efficiently.
