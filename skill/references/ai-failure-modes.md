# AI Failure Modes in Code Generation

This reference catalogs the specific ways AI hallucinates or makes wrong decisions when generating code without proper specifications. Use this during the AUDIT workflow (Step 4: AI Decision Audit) to check if each failure mode is addressed by existing documentation.

---

## How to Use This File

For each failure mode below, check:
1. Is there a documented answer in the project's specs?
2. If yes, is it clear enough that AI won't misinterpret it?
3. If no, flag it as an AI Decision Gap in the audit report.

---

## FM-01: ID Format Inconsistency

**What happens**: AI generates UUIDs in one file, auto-increment integers in another, and string IDs in a third. The types don't match across joins and foreign keys.

**Prevented by**: DMS Section 2 (Naming Conventions) specifying exact ID type and generation method for every table.

**Check**: Is there ONE documented answer for "what type is an entity ID?" that applies everywhere?

**Common wrong defaults**:
- Auto-increment integers (leaks information about record count)
- UUID without specifying v4 vs v7
- String IDs without specifying format (is "ORD-001" a valid order_id? or is it a UUID?)

---

## FM-02: Timestamp Chaos

**What happens**: AI uses `Date.now()` in JavaScript (Unix milliseconds), `CURRENT_TIMESTAMP` in SQL (depends on server timezone), `new Date().toISOString()` (UTC string), and `TIMESTAMP WITHOUT TIME ZONE` in Postgres (no timezone info). Events appear to happen at wrong times.

**Prevented by**: TRD Section 8 (Timestamp and Timezone Strategy) specifying storage format, display format, and timezone handling.

**Check**: Can you answer ALL of these from existing docs?
- What Postgres type is used for timestamps? (`timestamptz` or `timestamp`?)
- What timezone is stored? (UTC always? local?)
- How are timestamps displayed to users? (user's timezone? lab's timezone?)
- What format is used in API responses? (ISO 8601? Unix?)
- How are DST transitions handled?

**Common wrong defaults**:
- `timestamp` without timezone (loses timezone info forever)
- Storing local time instead of UTC (breaks if lab moves or has remote users)
- Displaying UTC to users (confusing)

---

## FM-03: State Transition Violations

**What happens**: AI generates code that allows any state to transition to any other state (e.g., a closed order going back to draft, a deleted account being reactivated). No validation on transitions.

**Prevented by**: DMS Section 5 (State Machines) with explicit valid/invalid transitions.

**Check**: For every entity with a lifecycle, is there a state machine diagram showing:
- All valid states?
- All valid transitions (and what event triggers each)?
- Explicitly invalid transitions?
- What error is returned for invalid transitions?

**Common wrong defaults**:
- No state validation at all (status field accepts any string)
- CHECK constraint on valid states but no transition validation
- Validation in frontend only (bypassable)

---

## FM-04: Mutable Event Log

**What happens**: AI generates UPDATE or DELETE statements on an append-only events table, breaking the immutability guarantee. Audit trail becomes unreliable. State replay produces different results than the original execution.

**Prevented by**: DMS Section 4 (Table Definitions with Mutability field) + DMS Section 6 (Trigger Functions) + ADR explaining event sourcing pattern.

**Check**: Is it explicitly documented that:
- The events table is INSERT-only?
- RLS policies block UPDATE/DELETE on events?
- Corrections are new events, not edits to old events?
- There's a specific correction event type?

**Common wrong defaults**:
- AI generates `UPDATE events SET ... WHERE id = ...` for corrections
- AI generates a "soft delete" pattern with `is_deleted` flag (violates immutability)
- No RLS policy preventing UPDATE/DELETE

---

## FM-05: Security by Absence

**What happens**: AI generates no authentication, no authorization checks, no RLS policies. Or generates them inconsistently (some endpoints protected, others wide open). Defaults to "anyone can do anything."

**Prevented by**: TRD Section 4 (Auth) + TRD Section 5 (Security) + DMS Section 4 (RLS Policies per table).

**Check**: For every table and endpoint:
- Is there a documented answer for "who can access this?"
- Are RLS policies defined (not just mentioned)?
- Is there a role-permission matrix?

**Common wrong defaults**:
- `anon` role can read all data (Supabase default if RLS not enabled)
- No RLS policies created (tables are publicly readable)
- Auth token checked in frontend but not enforced server-side

---

## FM-06: Error Swallowing

**What happens**: AI generates try/catch blocks that log errors to console and return generic "Something went wrong" messages. Trigger failures are silently ignored. Users see blank screens instead of error messages.

**Prevented by**: TRD Section 6 (Error Handling Strategy) specifying error format, error codes, and behavior for each error type.

**Check**: Is there a documented answer for:
- What format do error responses use?
- What happens if a database trigger fails?
- What happens if an external service (printer, shipping) is unavailable?
- How are errors communicated to the user?
- Are errors logged for debugging?

**Common wrong defaults**:
- `catch(e) { console.log(e) }` with no user feedback
- Trigger failures silently ignored (event inserted but state not updated)
- Generic error messages with no actionable information

---

## FM-07: Naming Convention Drift

**What happens**: AI uses camelCase for some columns, snake_case for others, PascalCase for some tables. Frontend sends `orderId`, backend expects `order_id`, database has `OrderID`. Everything breaks at integration boundaries.

**Prevented by**: DMS Section 2 (Naming Conventions) + API Section 2 (Conventions).

**Check**: Is there ONE documented convention for:
- Database table names? (snake_case, plural)
- Database column names? (snake_case)
- API request/response field names? (camelCase? snake_case?)
- Frontend variable names? (camelCase)
- Conversion strategy at boundaries? (automatic via ORM? manual mapping?)

**Common wrong defaults**:
- AI follows whatever convention appeared most recently in its training data
- Mixed conventions within the same file
- No conversion layer between DB snake_case and JS/frontend camelCase

---

## FM-08: N+1 Queries and Performance Cliffs

**What happens**: AI generates code that loads a list of parent records, then for each parent loads its children, then for each child loads related data. 100 parents = potentially 100s of queries. Page takes 30 seconds to load.

**Prevented by**: TRD Section 11 (Performance Targets) + DMS Section 4 (Indexes) + API design with explicit data loading patterns.

**Check**: For every list/table view in the app:
- Is there a documented query strategy? (JOIN, separate queries, denormalized view, GraphQL fragments?)
- Are there indexes supporting the expected query patterns?
- Is pagination specified? (cursor-based? offset? limit?)
- Are there performance targets for common queries?

**Common wrong defaults**:
- Separate query per related entity (N+1)
- No pagination (loads all 50,000 records)
- No indexes beyond primary keys
- Full table scans on search

---

## FM-09: Cascade Confusion

**What happens**: AI generates `ON DELETE CASCADE` on a foreign key, so deleting a parent record deletes all its children, grandchildren, and related records. Or AI generates no cascade behavior, so deleting a parent leaves orphaned child records that cause null reference errors everywhere.

**Prevented by**: DMS Section 4 (Table Definitions with explicit FK constraints and cascade behavior).

**Check**: For every foreign key relationship:
- What happens when the parent is deleted? (CASCADE, SET NULL, RESTRICT, or "never deleted")
- Is deletion even allowed for this entity? (event sourcing often means "archive, never delete")
- What happens to child records?

**Common wrong defaults**:
- `ON DELETE CASCADE` everywhere (dangerous — one delete destroys related data unintentionally)
- No cascade behavior specified (orphaned records accumulate)
- Soft delete in some tables, hard delete in others (inconsistent behavior across the app)

---

## FM-10: Concurrency Blindness

**What happens**: Two users perform the same write operation simultaneously. Both succeed, resulting in duplicate records or a corrupted state. Or two users update the same record at the same time, and one update is silently lost.

**Prevented by**: TRD Section 3 (Architecture Pattern constraints) + DMS Section 6 (Trigger Functions with concurrency handling).

**Check**: For every write operation:
- What happens if two users do this simultaneously?
- Is there optimistic locking (version numbers)?
- Is there a database-level constraint preventing duplicates?
- For event sourcing: is there an idempotency key?

**Common wrong defaults**:
- No concurrency handling at all (last write wins, silently)
- Optimistic locking in frontend only (bypassable)
- No idempotency key on events (duplicate events possible)

---

## FM-11: Implicit Business Rules

**What happens**: A rule like "an order can only be submitted once" or "you can't fulfill a request until it's approved" exists in someone's head but not in the code. AI doesn't know the rule exists, generates code that violates it.

**Prevented by**: PRD Section 6 (Workflows with edge cases) + DMS Section 5 (State Machines) + DMS Section 4 (CHECK constraints).

**Check**: For every entity:
- Are there business rules that constrain what can happen?
- Are those rules documented?
- Are they enforced at the database level (not just frontend)?
- What happens when a rule is violated? (error message, prevention, warning)

---

## FM-12: Glossary Confusion

**What happens**: AI uses two domain terms interchangeably when they mean different things (e.g., "user" and "account", "order" and "transaction"). Or generates code for an entity name that doesn't appear in the data model. Terminology mismatch between layers causes bugs that are hard to trace.

**Prevented by**: PRD Section 4 (Glossary) used consistently across all documents.

**Check**:
- Is there a glossary covering all domain terms?
- Does every document use the same terms for the same concepts?
- Are synonyms and near-synonyms explicitly mapped?
- Does code use the same terms as specs? (no silent renaming between DB, API, and frontend)

---

## FM-13: Missing Validation Boundaries

**What happens**: AI generates a form that accepts any string for an entity ID, any number for a quantity field (including 0, negative, or 999999), any date format. Invalid data enters the database and causes downstream failures.

**Prevented by**: DMS Section 4 (constraints) + PRD Section 5 (field constraints) + API Section 3 (request validation).

**Check**: For every user input:
- What are the valid values? (min, max, format, allowed characters)
- Where is validation enforced? (frontend, API, database, all three?)
- What feedback does the user get for invalid input?

---

## FM-14: Hardcoded Configuration

**What happens**: AI hardcodes values that should be configurable (page size = 20, currency = "USD", timezone = "UTC"). Changing the value later requires code changes instead of configuration changes.

**Prevented by**: PRD Section 5 (Entity Definitions noting which fields are configurable) + TRD (configuration management section).

**Check**: Which values might change per tenant, per deployment, or over time? Are they configurable or hardcoded?

Common things that should NOT be hardcoded:
- Pagination limits
- Currency or locale
- Timezone
- Default role permissions
- Status names and allowed transitions
- Environment-specific URLs, keys, or thresholds
- Any business rule numeric value (quantities, limits, counts)

---

## FM-15: Secret and Key Exposure

**What happens**: AI puts API keys, database connection strings, or auth tokens directly in frontend code or source files. These get committed to git, pushed to public repos, and become a security incident. Or AI logs auth tokens to the console for debugging, exposing them in production logs.

**Prevented by**: CODESTYLE Section 11 (Anti-patterns) + DEPLOY Section 4 (Secrets Management) + TRD Section 5 (Security Architecture).

**Check**:
- Is there a documented rule that secrets never appear in client/frontend code?
- Is `.env` in `.gitignore`? Is `.env.example` committed with placeholder values?
- Is there a documented secret management strategy? (Vercel env vars, AWS Secrets Manager, etc.)
- Is there a rule against logging auth tokens, passwords, or PII?

**Common wrong defaults**:
- `const API_KEY = "sk-live-..."` hardcoded in a React component
- DB connection string in a frontend config file
- `console.log(user)` where `user` contains auth tokens
- No `.gitignore` entry for `.env`

---

## FM-16: UI State Gaps

**What happens**: AI generates the happy path only. Components fetch data and render it — but there's no loading state, no empty state, and no error state. Users see a blank screen while loading, an empty div when there's no data, and a white screen crash when the API fails. These aren't edge cases; they're the normal experience before data loads.

**Prevented by**: CODESTYLE Section 8 (Error Handling Patterns) + PRD Section 6 (Workflows with edge cases) + CODESTYLE Section 11 (Anti-patterns).

**Check**: For every component that fetches data, are these explicitly specified?
- Loading state: what does the user see while waiting?
- Empty state: what does the user see when the response is empty?
- Error state: what does the user see when the request fails?
- Are these in the acceptance criteria? Are they in the CODESTYLE anti-patterns list?

**Common wrong defaults**:
```jsx
// AI generates this:
return <div>{data.map(item => <Row key={item.id} {...item} />)}</div>

// What it should generate:
if (isLoading) return <LoadingSpinner />
if (error) return <ErrorMessage message={error.message} />
if (!data?.length) return <EmptyState message="No items found" />
return <div>{data.map(item => <Row key={item.id} {...item} />)}</div>
```

**Severity note**: FM-16 rarely corrupts data but consistently ships broken UX. It's the most common visible defect in AI-generated frontend code.

---

## FM-17: Type Drift

**What happens**: The database schema says a column is `uuid`. The ORM returns it as `string`. A Zod schema somewhere says it's `number`. A TypeScript interface says `any`. Each layer made its own type decision independently. TypeScript stops catching errors at integration boundaries because types don't match — but the mismatch only surfaces at runtime.

**Prevented by**: CODESTYLE Section 4 (TypeScript Rules) + DMS Section 2 (Naming Conventions with type decisions) + DMS Section 4 (Table Definitions with exact column types).

**Check**:
- Is TypeScript strict mode documented and enabled?
- Is there a documented strategy for mapping DB types to TypeScript types?
- Is type generation automated (ORM CLI, Supabase CLI) or manual?
- Is there a `no-any` rule documented and enforced?
- Do the types in the codebase actually match the DMS column types?

**Common wrong defaults**:
- `id: string` in TypeScript when DB generates UUID (fine, but must be consistent)
- `id: number` in TypeScript when DB uses UUID (breaks everything)
- `any` typed API responses (TypeScript can't catch mismatches)
- `interface User { [key: string]: any }` (completely defeats the type system)
- Different type definitions for the same entity in different files

---

## Severity Rankings

When prioritizing which failure modes to address first:

**CRITICAL (data corruption or security breach)**:
FM-04 (Mutable Event Log), FM-05 (Security), FM-03 (State Transitions), FM-10 (Concurrency), FM-15 (Secret Exposure)

**HIGH (significant rework)**:
FM-01 (ID Format), FM-02 (Timestamps), FM-06 (Error Handling), FM-09 (Cascades), FM-17 (Type Drift)

**MEDIUM (annoying but visibly broken)**:
FM-07 (Naming), FM-08 (Performance), FM-11 (Business Rules), FM-13 (Validation), FM-16 (UI States)

**LOW (quality of life)**:
FM-12 (Glossary), FM-14 (Hardcoded Config)
