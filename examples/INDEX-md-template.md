# docs/INDEX.md Template

This file is the Section Index. It maps every section in every planning doc to its line range so the coding AI can load only what it needs per task.

**How the AI uses it:**
1. Reads CLAUDE.md, sees Document Registry
2. Determines which docs are relevant to the current task
3. Reads this INDEX.md to find specific sections
4. Uses `Read` with offset/limit to pull only those lines from source docs

**Maintenance:** Update this file whenever a planning doc is created or edited. The spec-driven-dev skill does this automatically as Step 8 of the GENERATE workflow.

---

```markdown
# Document Section Index

Use this to find the exact section you need without reading entire docs.
Format: [Section] > [Line Range] > [What It Contains]

## DMS (docs/dms.md)

| Section | Lines | Contains |
|---------|-------|----------|
| 1. Overview | 1-15 | Architecture pattern summary |
| 2. Naming Conventions | 16-45 | Table, column, PK, FK, timestamp, enum naming rules |
| 3. Entity-Relationship Diagram | 46-80 | Full ER diagram (mermaid or ASCII) |
| 4.1 [table_name] | 81-105 | [table purpose] |
| 4.2 [table_name] | 106-145 | [table purpose] |
| 5. State Machines | 400-480 | All entity lifecycle diagrams |
| 6. Trigger Functions | 481-560 | Trigger pseudocode and concurrency handling |

## TRD (docs/trd.md)

| Section | Lines | Contains |
|---------|-------|----------|
| 1. Architecture Overview | 1-30 | System component diagram |
| 4. Auth and Authorization | 80-120 | Auth provider, roles, RLS summary |
| 6. Error Handling | 150-200 | Error response format, trigger failures |
| 7. Integrations | 201-250 | External system protocols and fallbacks |
| 8. Timestamp Strategy | 251-270 | UTC storage, display timezone, DST handling |

## CODESTYLE (docs/codestyle.md)

| Section | Lines | Contains |
|---------|-------|----------|
| 2. File Naming | 10-25 | Component, hook, util naming rules |
| 4. TypeScript Rules | 40-65 | Strict mode, any policy, type generation |
| 7. State Management | 100-130 | Server state, client state, form state patterns |
| 8. Error Patterns | 131-180 | Actual code patterns (paste into task brief) |
| 11. Anti-Patterns | 200-250 | Never-generate list (also in CLAUDE.md) |

## PRD (docs/prd.md)

| Section | Lines | Contains |
|---------|-------|----------|
| 4. Glossary | 50-90 | All domain terms |
| 5. Entity Definitions | 91-200 | Core entities with fields and relationships |
| 6. Workflows | 201-400 | Step-by-step workflows with edge cases |
| 8. Acceptance Criteria | 500-600 | GIVEN/WHEN/THEN per feature |

## ARCH (docs/arch.md)

| Section | Lines | Contains |
|---------|-------|----------|
| 3. Feature Map | 30-60 | Feature areas and their owned tables/routes |
| 4. Feature Boundaries | 61-120 | What each feature owns, exposes, consumes |
| 5. Frontend Structure | 121-160 | Folder structure and import rules |
```

**Note:** Line numbers above are examples. Replace with actual line numbers from your generated docs. If line ranges become stale after edits, the AI can fall back to grep for the section header. Line ranges are optimization, not hard dependencies.
