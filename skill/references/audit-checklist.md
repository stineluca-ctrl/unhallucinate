# Quick Audit Checklist

Use this as a rapid scan before diving into the full audit workflow. Each item maps to a failure mode (FM-XX) and a document type.

---

## Identity and Types
- [ ] ID format documented for every entity (FM-01 → DMS)
- [ ] Primary key type specified (UUID v4, custom, auto-increment) (FM-01 → DMS)
- [ ] Foreign key types match across all tables (FM-01 → DMS)

## Time
- [ ] Timestamp column type specified (`timestamptz` vs `timestamp`) (FM-02 → DMS)
- [ ] Timezone storage strategy documented (UTC always?) (FM-02 → TRD)
- [ ] Timezone display strategy documented (FM-02 → TRD)
- [ ] Date format for API responses documented (FM-02 → API)

## State
- [ ] Every entity with a lifecycle has a state machine diagram (FM-03 → DMS)
- [ ] Valid transitions listed (FM-03 → DMS)
- [ ] Invalid transitions explicitly listed (FM-03 → DMS)
- [ ] Transition validation enforced at database level (FM-03 → DMS)

## Immutability (if event sourcing)
- [ ] Events table marked as INSERT-only (FM-04 → DMS)
- [ ] RLS blocks UPDATE/DELETE on events (FM-04 → DMS)
- [ ] Correction pattern documented (new event, not edit) (FM-04 → ADR)
- [ ] State derivation logic documented (trigger functions) (FM-04 → DMS)

## Security
- [ ] Auth method documented (FM-05 → TRD)
- [ ] Role-permission matrix exists (FM-05 → TRD)
- [ ] RLS policies defined per table (FM-05 → DMS)
- [ ] PII handling documented (FM-05 → TRD/NFR)

## Errors
- [ ] Error response format documented (FM-06 → TRD/API)
- [ ] Trigger failure behavior documented (FM-06 → DMS)
- [ ] External service failure behavior documented (FM-06 → TRD)
- [ ] User-facing error messages specified (FM-06 → PRD)

## Naming
- [ ] Database naming convention documented (FM-07 → DMS)
- [ ] API naming convention documented (FM-07 → API)
- [ ] Conversion strategy at boundaries documented (FM-07 → TRD)

## Performance
- [ ] Performance targets per operation (FM-08 → TRD/NFR)
- [ ] Index strategy documented (FM-08 → DMS)
- [ ] Pagination strategy documented (FM-08 → API)
- [ ] N+1 query prevention documented (FM-08 → TRD)

## Relationships
- [ ] Cascade behavior per FK documented (FM-09 → DMS)
- [ ] Deletion strategy documented (hard delete, soft delete, archive) (FM-09 → DMS)

## Concurrency
- [ ] Concurrent write behavior documented (FM-10 → TRD)
- [ ] Idempotency strategy documented (FM-10 → API/TRD)
- [ ] Duplicate prevention documented (FM-10 → DMS)

## Business Rules
- [ ] All business constraints documented (FM-11 → PRD)
- [ ] Constraints enforced at database level (FM-11 → DMS)
- [ ] Violation error messages documented (FM-11 → PRD/API)

## Glossary
- [ ] Glossary exists with all domain terms (FM-12 → PRD)
- [ ] Synonyms explicitly mapped (FM-12 → PRD)
- [ ] Terms used consistently across all docs (FM-12 → ALL)

## Validation
- [ ] Input constraints per field documented (FM-13 → PRD/DMS)
- [ ] Validation enforcement layers documented (FM-13 → TRD)
- [ ] Error feedback for invalid input documented (FM-13 → PRD)

## Configuration
- [ ] Configurable values identified (FM-14 → PRD/TRD)
- [ ] Configuration storage method documented (FM-14 → TRD)
- [ ] Default values documented (FM-14 → DMS)

## Secrets and Key Exposure
- [ ] Rule documented: no secrets or API keys in client/frontend code (FM-15 → CODESTYLE/TRD)
- [ ] `.env` in `.gitignore`, `.env.example` committed with placeholder values (FM-15 → DEPLOY)
- [ ] Secret management strategy documented: where secrets live, who has access (FM-15 → DEPLOY)
- [ ] Rule documented: no logging of auth tokens, passwords, or PII (FM-15 → CODESTYLE)

## UI State Completeness
- [ ] Loading states specified for all data-fetching views (FM-16 → PRD/CODESTYLE)
- [ ] Empty states specified for all list/table views (FM-16 → PRD/CODESTYLE)
- [ ] Error states specified for all features that make API calls (FM-16 → PRD/CODESTYLE)
- [ ] Anti-pattern documented in CODESTYLE: "never return bare data render without loading/empty/error states" (FM-16 → CODESTYLE)

## Type Safety
- [ ] TypeScript strict mode setting documented (FM-17 → CODESTYLE)
- [ ] DB-to-code type mapping strategy documented: manual or generated? (FM-17 → CODESTYLE/DMS)
- [ ] `any` type policy documented and enforced (FM-17 → CODESTYLE)
- [ ] Single source of truth for types established (FM-17 → CODESTYLE)

## Coding Standards
- [ ] File naming convention documented (FM-07 → CODESTYLE)
- [ ] Code-level naming convention documented: variables, functions, booleans, events (FM-07 → CODESTYLE)
- [ ] State management pattern documented: when to use local vs shared vs server state (ARCH/CODESTYLE)
- [ ] Error handling code patterns documented: not principles, actual code structure (FM-06 → CODESTYLE)
- [ ] Anti-patterns list compiled and added to CLAUDE.md or always-on context (FM-15, FM-16 → CODESTYLE)

## Context Engineering
- [ ] CLAUDE.md (or equivalent) exists and includes: glossary, stack, naming conventions, key constraints, anti-patterns (context-engineering.md)
- [ ] CLAUDE.md is under 400 lines (longer = critical rules get buried) (context-engineering.md)
- [ ] Per-task brief template or pattern exists for coding sessions (context-engineering.md)
- [ ] CODESTYLE anti-patterns are in CLAUDE.md, not just in the CODESTYLE doc (context-engineering.md)
- [ ] Active open questions listed in CLAUDE.md so AI flags them instead of guessing (context-engineering.md)

---

## Scoring

Count checked items vs total. This gives a rough readiness score:

| Score | Readiness | Recommendation |
|-------|-----------|----------------|
| 90-100% | Ready to build | Start coding with confidence |
| 70-89% | Mostly ready | Fill gaps in critical areas before starting |
| 50-69% | Partially ready | Write DMS and TRD before any code |
| 30-49% | Not ready | Significant spec work needed first |
| 0-29% | Documentation debt | Start with PRD, then DMS, then TRD |
