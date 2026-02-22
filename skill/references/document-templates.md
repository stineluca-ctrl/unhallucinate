# Document Templates

Templates for each document type in the registry. When generating a document, use these as starting structures and fill in project-specific details.

---

## Table of Contents
1. [PRD - Product Requirements Document](#prd)
2. [DMS - Data Model Specification](#dms)
3. [TRD - Technical Requirements Document](#trd)
4. [NFR - Non-Functional Requirements](#nfr)
5. [ARCH - Architecture & Module Plan](#arch)
6. [CODESTYLE - Coding Standards](#codestyle)
7. [ADR - Architecture Decision Record](#adr)
8. [API - API/Integration Design](#api)
9. [OQR - Open Questions Register](#oqr)
10. [TEST - Testing Strategy](#test)
11. [DEPLOY - Deployment Plan](#deploy)
12. [MIGRATE - Data Migration Plan](#migrate)

---

<a id="prd"></a>
## 1. PRD - Product Requirements Document

```markdown
# [Project Name] - Product Requirements Document
Version: X.X | Date: YYYY-MM-DD | Status: Draft/Review/Approved

## 1. Problem Statement
What problem does this software solve? Who has this problem? What do they do today without it?

## 2. Goals and Success Metrics
What does "done" look like? How will you measure success?
- Goal 1: [measurable outcome]
- Goal 2: [measurable outcome]

## 3. Users and Roles
| Role | Description | Permissions Summary |
|------|-------------|-------------------|
| Admin | ... | Full access |
| Staff | ... | Read/write samples |

## 4. Glossary
| Term | Definition |
|------|-----------|
Every domain-specific term. This section prevents AI from confusing similar concepts.

## 5. Entity Definitions
For each core entity (e.g., order, user, product, shipment):
- What is it?
- What fields does it have? (name, type, required/optional, constraints)
- What is its lifecycle? (state machine: created > active > archived)
- What are its relationships? (e.g., an order has many line items)

## 6. Workflows
For each core workflow:
- Trigger: what starts this workflow?
- Steps: what happens, in order?
- Actors: who does each step?
- System actions: what does the software do at each step?
- End state: what's true when the workflow completes?
- Edge cases: what could go wrong?

## 7. Scope Boundaries
### In Scope (Phase 1)
- Feature A
- Feature B

### Out of Scope (Phase 2+)
- Feature C (reason for deferral)

## 8. Acceptance Criteria
For each feature, specific testable conditions:
- GIVEN [precondition] WHEN [action] THEN [expected result]

## 9. Assumptions
Every assumption the PRD makes. Each must be confirmed or rejected before development.

## 10. Open Questions
Link to OQR or list inline. Every unresolved decision.
```

**Sections that prevent the most AI errors**: #4 (Glossary), #5 (Entity Definitions with state machines), #6 (Workflows with edge cases), #7 (Scope Boundaries)

---

<a id="dms"></a>
## 2. DMS - Data Model Specification

This is the single most important document for preventing AI hallucinations in database-backed applications.

```markdown
# [Project Name] - Data Model Specification
Version: X.X | Date: YYYY-MM-DD | Status: Draft/Review/Approved

## 1. Overview
Brief description of the data architecture pattern (e.g., event sourcing + materialized state tables, standard CRUD, CQRS).

## 2. Naming Conventions
- Tables: [pick one: snake_case plural / PascalCase singular / other — document it]
  - Example (Postgres): snake_case plural (e.g., `orders`, `order_items`)
- Columns: [snake_case is the Postgres default; camelCase for ORMs like Prisma]
- Primary keys: [UUID v4? auto-increment bigint? custom string? pick one]
- Foreign keys: [e.g., `{entity}_id` — document the pattern]
- Timestamps: [storage type: `timestamptz` vs `timestamp`; always document timezone strategy]
- Booleans: [e.g., `is_{adjective}` — document the prefix convention]
- Enums: [native DB enum types, CHECK constraints, or lookup table — pick one]

## 3. Entity-Relationship Diagram
ASCII or mermaid diagram showing all entities and relationships.

## 4. Table Definitions
For EACH table:

### {table_name}
**Purpose**: One sentence explaining why this table exists.
**Mutability**: Immutable (append-only) | Mutable (standard CRUD) | Derived (updated by triggers)

| Column | Type | Nullable | Default | Constraints | Description |
|--------|------|----------|---------|-------------|-------------|
| id | uuid | NO | gen_random_uuid() | PRIMARY KEY | ... |
| ... | ... | ... | ... | ... | ... |

**Indexes**:
- `idx_{table}_{columns}` on (col1, col2) — reason for index

**RLS Policies** (if applicable):
- SELECT: [who can read, under what conditions]
- INSERT: [who can insert]
- UPDATE: [who can update, or "NONE - immutable"]
- DELETE: [who can delete, or "NONE - immutable"]

## 5. State Machines
For each entity with a lifecycle, a state transition diagram:

```
[state_A] --event_X--> [state_B] --event_Y--> [state_C]
                                  --event_Z--> [state_D]
```

Include:
- All valid states
- All valid transitions (event that causes each transition)
- Invalid transitions (explicitly: "an [entity] CANNOT go from [state_a] back to [state_b]")
- What happens on invalid transition attempts (error message, rejection)

## 6. Trigger Functions
For each database trigger:

### {trigger_name}
**Fires on**: INSERT/UPDATE/DELETE on {table}
**Timing**: BEFORE/AFTER
**Security**: SECURITY DEFINER | SECURITY INVOKER
**Logic**: Step-by-step pseudocode or actual SQL
**Error handling**: What happens if this trigger fails?
**Concurrency**: What happens if two events fire simultaneously?

## 7. Seed Data
Any data that must exist before the application starts (default roles, initial project, etc.)

## 8. Migration Strategy
How schema changes are applied:
- Tool: (Supabase migrations, Flyway, Prisma, raw SQL)
- Rollback approach
- Zero-downtime migration rules
```

**Sections that prevent the most AI errors**: #2 (Naming Conventions), #4 (Table Definitions with exact types), #5 (State Machines), #6 (Trigger Functions)

---

<a id="trd"></a>
## 3. TRD - Technical Requirements Document

```markdown
# [Project Name] - Technical Requirements Document
Version: X.X | Date: YYYY-MM-DD | Status: Draft/Review/Approved

## 1. Architecture Overview
High-level diagram showing all system components and how they connect.
- Frontend: [framework, hosting]
- Backend: [framework/service, hosting]
- Database: [engine, hosting]
- External services: [list each integration]

## 2. Technology Stack
| Layer | Technology | Version | Reason |
|-------|-----------|---------|--------|
| Frontend | React + Vite | 18.x | SPA, no SSR needed |
| ... | ... | ... | ... |

## 3. Architecture Pattern
Describe the core pattern (event sourcing, CQRS, microservices, monolith, etc.)
- Why this pattern was chosen
- What constraints it imposes
- What it makes easier vs harder

## 4. Authentication and Authorization
- Auth provider: [Supabase Auth, Auth0, custom]
- Auth method: [email/password, SSO, magic link]
- Session management: [JWT, cookies, refresh tokens]
- Role-based access: [how roles map to permissions]
- Row-level security: [RLS policies summary]

## 5. Security Architecture
- Data encryption: at rest [method], in transit [method]
- PII handling: [where stored, how protected, compliance requirements]
- Input validation: [where, how]
- CORS policy: [origins allowed]
- Rate limiting: [if applicable]

## 6. Error Handling Strategy
- Frontend errors: [how displayed to user, how logged]
- Backend/API errors: [response format, error codes, retry logic]
- Database errors: [trigger failures, constraint violations, connection loss]
- External service errors: [timeout handling, fallback behavior]
- Error response format:
  ```json
  {
    "error": {
      "code": "ENTITY_STATE_INVALID",
      "message": "Cannot transition [entity] from [state_a] to [state_b]",
      "details": { ... }
    }
  }
  ```

## 7. Integration Architecture
For each external system:
### {System Name}
- Protocol: [REST, WebSocket, Bluetooth, USB, file-based]
- Authentication: [API key, OAuth, none]
- Data format: [JSON, CSV, binary]
- Error handling: [what if unavailable?]
- Fallback: [graceful degradation plan]

## 8. Timestamp and Timezone Strategy
- Storage: [UTC always, with timezone, without timezone]
- Display: [user's local timezone, lab timezone, configurable]
- Format: [ISO 8601, Unix timestamp]
- Edge cases: [DST transitions, events spanning midnight]

## 9. File Handling
- Upload: [client-side, server-side, direct to storage]
- Storage: [Supabase Storage, S3, local filesystem]
- Size limits: [per file, per user]
- Allowed types: [whitelist of MIME types]
- Processing: [client-side only for PII? server-side?]

## 10. Real-time Updates
- Technology: [Supabase Realtime, WebSocket, polling]
- What updates in real-time: [which tables/events]
- Subscription management: [how connections are managed]
- Fallback: [if real-time connection drops]

## 11. Performance Targets
| Operation | Target | Measurement |
|-----------|--------|-------------|
| Page load | [e.g. < 2s] | First contentful paint |
| Search | [e.g. < 1s] | Query to results displayed |
| [Key operation] | [target] | [how measured] |

## 12. Logging and Monitoring
- Application logs: [where, format, retention]
- Error tracking: [Sentry, LogRocket, custom]
- Performance monitoring: [metrics tracked]
- Alerts: [what triggers alerts, who receives them]
```

**Sections that prevent the most AI errors**: #4 (Auth), #6 (Error Handling), #8 (Timestamps), #3 (Architecture Pattern constraints)

---

<a id="nfr"></a>
## 4. NFR - Non-Functional Requirements

```markdown
# [Project Name] - Non-Functional Requirements
Version: X.X | Date: YYYY-MM-DD

## 1. Performance
- Response time targets per operation type
- Database query time limits
- Concurrent user capacity
- Data volume projections (1 year, 3 years)

## 2. Security
- Authentication requirements
- Authorization model (RBAC, ABAC)
- Data classification (PII, PHI, public)
- Encryption requirements
- Audit trail requirements (who, what, when for every data change)
- Session timeout/management

## 3. Compliance
- Regulatory frameworks (21 CFR Part 11, HIPAA, GDPR)
- Audit trail depth (what events must be logged immutably?)
- Electronic signature requirements
- Data retention and deletion policies
- Export requirements (what formats, for whom)

## 4. Reliability
- Uptime target (99.9%? 99.5%?)
- Recovery time objective (RTO)
- Recovery point objective (RPO)
- Backup strategy
- Disaster recovery plan

## 5. Usability
- Target devices (desktop only? tablet? mobile?)
- Browser support (Chrome, Firefox, Safari, Edge)
- Accessibility standard (WCAG 2.1 AA?)
- Offline capability requirements
- Training/onboarding requirements

## 6. Scalability
- Expected growth (users, data volume, features)
- Scaling strategy (vertical, horizontal)
- Database partitioning plan (if needed)

## 7. Data Integrity
- Immutability rules (which data cannot be modified after creation?)
- Consistency model (strong, eventual)
- Conflict resolution (last-write-wins, event ordering)
- Backup and recovery testing schedule
```

---

<a id="adr"></a>
## 5. ADR - Architecture Decision Record

One file per decision. Store in `/docs/adr/` or `/architecture/decisions/`.

```markdown
# ADR-{NNN}: {Decision Title}
Date: YYYY-MM-DD | Status: Proposed | Accepted | Deprecated | Superseded by ADR-XXX

## Context
What situation or problem prompted this decision? What forces are at play?

## Decision
What did we decide? State it clearly in one sentence, then elaborate.

## Alternatives Considered
| Alternative | Pros | Cons | Why Not |
|------------|------|------|---------|
| Option A | ... | ... | ... |
| Option B (chosen) | ... | ... | Selected because... |

## Consequences
### Positive
- What becomes easier or better?

### Negative
- What becomes harder or more constrained?
- What trade-offs are we accepting?

### Neutral
- What changes but isn't clearly better or worse?

## References
Links to research, benchmarks, or discussions that informed this decision.
```

**Common ADRs for AI-assisted projects**:
- ADR-001: Architecture pattern choice (e.g., event sourcing vs CRUD vs CQRS)
- ADR-002: Database/backend platform choice and why
- ADR-003: Auth strategy (managed auth service vs custom vs session-based)
- ADR-004: Data processing location (client-side vs server-side) and why
- ADR-005: Any external integration protocol choices (REST vs WebSocket vs file-based)

---

<a id="api"></a>
## 6. API - API/Integration Design

```markdown
# [Project Name] - API Design
Version: X.X | Date: YYYY-MM-DD

## 1. Overview
- Base URL: [e.g., Supabase project URL]
- Authentication: [method, token format]
- Content type: application/json
- Error format: [standard error response shape]

## 2. Conventions
- URL naming: kebab-case
- Request body: camelCase or snake_case (pick one, document it)
- Pagination: [cursor-based, offset-based, page-based]
- Filtering: [query params, request body]
- Sorting: [format]

## 3. Endpoints
For each endpoint:

### [METHOD] /path
**Purpose**: One sentence
**Auth**: Required role(s)
**Request**:
```json
{
  "field": "type (required/optional) - description"
}
```
**Response (200)**:
```json
{
  "data": { ... }
}
```
**Errors**:
| Code | Status | Description |
|------|--------|-------------|
| ENTITY_NOT_FOUND | 404 | ... |
| INVALID_STATE_TRANSITION | 422 | ... |

## 4. Real-time Subscriptions
For each real-time channel:
- Channel: [table name or custom channel]
- Events: [INSERT, UPDATE, DELETE]
- Payload shape: [what data is sent]
- Who subscribes: [which roles/pages]

## 5. External Integrations
For each external system:
### {System Name}
- Direction: [this system calls it / it calls this system / bidirectional]
- Protocol: [REST, file-based, Bluetooth]
- Data format with examples
- Authentication
- Rate limits
- Error handling
```

---

<a id="oqr"></a>
## 7. OQR - Open Questions Register

```markdown
# [Project Name] - Open Questions Register
Last updated: YYYY-MM-DD

## Status Key
- OPEN: Needs answer before development can proceed
- BLOCKED: Waiting on specific person/information
- DECIDED: Answer confirmed, needs to be added to relevant spec
- CLOSED: Answer confirmed and added to relevant spec

## Questions

### OQ-{NNN}: {Question}
- **Priority**: BLOCKER / HIGH / MEDIUM / LOW
- **Status**: OPEN / BLOCKED / DECIDED / CLOSED
- **Assignee**: Who needs to answer this?
- **Source**: Which document surfaced this question?
- **Context**: Why does this matter? What breaks if we guess wrong?
- **Options**: If applicable, what are the possible answers?
- **Decision**: [filled in when resolved]
- **Affected docs**: Which specs need updating when this is resolved?
```

---

<a id="test"></a>
## 8. TEST - Testing Strategy

```markdown
# [Project Name] - Testing Strategy
Version: X.X | Date: YYYY-MM-DD

## 1. Test Levels
| Level | Framework | Coverage Target | What It Tests |
|-------|-----------|----------------|--------------|
| Unit | [Vitest, Jest] | 80%+ critical paths | Functions, hooks, utilities |
| Integration | [Vitest + Supabase local] | Key workflows | DB triggers, RLS, API calls |
| E2E | [Playwright, Cypress] | Happy paths + critical edges | Full user workflows |

## 2. Test Data Strategy
- Seed data: [how is test data created?]
- Fixtures: [reusable test data sets]
- Cleanup: [how is test data cleaned after tests?]
- PII in tests: [never use real PII, use faker/generated data]

## 3. Critical Test Scenarios
For each core workflow, specific test cases:
- Happy path: [normal flow works]
- Edge cases: [boundary conditions]
- Error cases: [invalid inputs, unauthorized access, system failures]
- State transition tests: [every valid and invalid state transition]

## 4. Database Testing
- Trigger function tests: [verify event > state table derivation]
- RLS tests: [verify each role can/cannot access expected data]
- Constraint tests: [verify CHECK, UNIQUE, FK constraints]
- Migration tests: [verify up + down migrations]

## 5. CI Pipeline
- When tests run: [on push, on PR, scheduled]
- Required to pass before merge: [which test levels]
- Performance regression: [how detected]
```

---

<a id="deploy"></a>
## 9. DEPLOY - Deployment Plan

```markdown
# [Project Name] - Deployment Plan
Version: X.X | Date: YYYY-MM-DD

## 1. Environments
| Environment | Purpose | URL | DB Instance |
|-------------|---------|-----|-------------|
| Local | Development | localhost:5173 | Local Supabase |
| Staging | QA/Testing | staging.example.com | Staging project |
| Production | Live | app.example.com | Production project |

## 2. Deployment Process
- Frontend: [build command, hosting platform, deploy command]
- Database: [migration tool, migration process, rollback process]
- Edge Functions: [deploy command, versioning]
- Environment variables: [where stored, how managed]

## 3. Rollback Strategy
- Frontend: [revert to previous deployment]
- Database: [down migration or restore from backup]
- When to rollback: [criteria for triggering rollback]

## 4. Secrets Management
- Where secrets are stored: [Vercel env vars, Supabase vault]
- Who has access: [list of people]
- Rotation schedule: [if applicable]
```

---

<a id="migrate"></a>
## 10. MIGRATE - Data Migration Plan

```markdown
# [Project Name] - Data Migration Plan
Version: X.X | Date: YYYY-MM-DD

## 1. Source Data
- Format: [Excel, CSV, database, manual records]
- Volume: [number of records per entity]
- Quality: [known issues, inconsistencies, missing data]
- Location: [where source data lives]

## 2. Transformation Rules
For each source > target mapping:
| Source Field | Target Table.Column | Transformation | Validation |
|-------------|--------------------|--------------:|------------|
| [Source field name] | [target_table.column] | [transformation logic] | [validation rule] |

## 3. Migration Process
- Step 1: Export source data
- Step 2: Transform (script location: /scripts/migrate/)
- Step 3: Validate transformed data
- Step 4: Import to staging
- Step 5: Verify in staging
- Step 6: Import to production
- Step 7: Verify in production

## 4. Rollback
- How to undo the migration if errors are found
- Point of no return (if any)

## 5. Validation Checklist
- [ ] Record counts match source
- [ ] No orphaned references
- [ ] All required fields populated
- [ ] State values are valid
- [ ] Timestamps in correct timezone
```

---

## 11. ARCH - Architecture & Module Plan

Defines how the codebase is organized into feature areas with fault isolation. For solo dev + AI projects, prefer pragmatic feature folders over strict modular monolith patterns (FSD, domain-per-schema). Scale up the formality if the team or codebase grows.

```markdown
# [Project Name] - Architecture & Module Plan
Version: X.X | Date: YYYY-MM-DD

## 1. Architecture Pattern
[Describe the pattern chosen: e.g., event sourcing, CRUD, CQRS, microservices, monolith, serverless.]

Why this pattern: [reason]
Why not [alternative]: [reason]
Constraints this imposes: [what AI must follow when generating code]

## 2. Tech Stack Summary
| Layer | Technology | Notes |
|-------|-----------|-------|
| Frontend | [e.g. React, Vue, Next.js, plain HTML] | [SPA? SSR? static?] |
| Backend | [e.g. Express, FastAPI, Supabase, Firebase] | [hosted? self-managed?] |
| Database | [e.g. Postgres, MySQL, MongoDB, DynamoDB] | [schema strategy?] |
| Auth | [e.g. Auth0, Supabase Auth, Clerk, custom] | [method: JWT, session, magic link?] |
| Hosting | [e.g. Vercel, Railway, AWS, self-hosted] | |

## 3. Feature Map
List every feature area in the system:

| Feature | Responsibility | Owns (tables/routes) | Key Entities |
|---------|---------------|----------------------|-------------|
| [e.g. orders] | [what it handles] | [which tables/endpoints] | [domain objects] |
| [e.g. users] | [what it handles] | [which tables/endpoints] | [domain objects] |

## 4. Feature Boundaries
For each feature:
### {feature_name}
**Owns**: Which tables/routes/API endpoints, which UI routes
**Exposes**: What other features can use (shared utilities, exported types/functions)
**Consumes**: What data from other features it reads (document the access pattern)

## 5. Frontend Structure
Adapt to the framework in use. The principles apply regardless of stack:

```
src/
  features/
    {feature}/       (components, hooks/logic, API calls for this feature)
  shared/
    components/      (reusable UI: Button, Input, Modal, Table)
    hooks/           (shared logic: useAuth, useToast)
    utils/           (formatters, validators, date utilities)
  lib/
    {client}.ts      (API/DB client initialization)
    types.ts         (shared TypeScript types or JSDoc types)
  App.tsx / main entry
```

Import rules:
- Features CAN import from shared/ and lib/
- Features SHOULD NOT import directly from other features
- If two features need the same data, lift it to shared/hooks or a service layer
- Pages/routes compose features; features don't compose each other

Alternatives for other frameworks:
- Next.js: `app/` or `pages/` routing, feature folders within `src/`
- Python/Django: `apps/` per feature, shared utilities in a `core/` app
- Node/Express: `modules/` per feature, `middleware/` for cross-cutting concerns

## 6. Backend Structure
```
[backend root]/
  [feature]/          (routes, controllers, services per feature)
  shared/             (middleware, utilities, shared types)
  [db tool]/          (migrations, seed files, schema definitions)
  [functions/lambda]/ (serverless functions, if applicable)
```

Database schema strategy: [single schema with naming prefixes? multiple schemas? ORM-managed?]

Key decision: For most small-team projects, a single default schema with clear naming prefixes is simpler than schema-per-module. Multiple schemas add API/ORM configuration overhead. Only adopt schema separation if your team size and access control requirements demand it.

## 7. Error Handling Strategy
- [Frontend]: [how UI errors are caught and displayed — framework-specific: try-catch, error boundaries, error states]
- [API layer]: [error response format, HTTP status codes, validation errors]
- [Database layer]: [constraint violations, trigger failures, connection errors]
- [External services]: [timeout handling, retry logic, fallback behavior]
- Global error logging: [Sentry / Datadog / custom / console in dev]
- Graceful degradation: [if a feature fails, what does the user see?]

Note for React projects: Error Boundaries only catch render-time errors. Async errors (API calls, event handlers, subscriptions) need explicit try-catch and error callbacks.

## 8. Feature Dependency Diagram
```
[Describe how features communicate. Examples:]

Event-driven:
  [events table] <-- all features write events
    └── triggers/workers update [state tables]
    └── pub/sub notifies frontend features

Direct API calls:
  Feature A --> API --> Feature B service --> DB
  (document the allowed dependency directions)

Message queue:
  Producer feature --> Queue --> Consumer feature
```

## 9. What's Shared (Not Feature-Specific)
- Auth/session management (global)
- [Audit trail / event log] (cross-cutting concern)
- UI component library (shared/components/)
- API/DB client (lib/)
- Error reporting (global error handler)
- [Logging, analytics, feature flags] (if applicable)
```

---

<a id="codestyle"></a>
## 12. CODESTYLE - Coding Standards

This document travels with every AI coding session. Its anti-patterns list and naming conventions should be excerpted into CLAUDE.md so AI sees them on every task. Without it, conventions drift across sessions — each session AI may make different decisions about file naming, state management, error handling, and TypeScript strictness.

```markdown
# [Project Name] - Coding Standards
Version: X.X | Date: YYYY-MM-DD

## 1. Purpose
These standards are given to AI in every coding session to prevent convention drift. AI has no memory between sessions — without explicit standards, every session produces slightly different patterns that accumulate into inconsistent, hard-to-maintain code.

## 2. File & Folder Naming
- Components/pages: [PascalCase.tsx? kebab-case.tsx? document it]
- Hooks: [always camelCase with `use` prefix — e.g. `useUserProfile.ts`]
- Utilities/helpers: [camelCase — e.g. `formatDate.ts`]
- Test files: [co-located: `UserProfile.test.tsx`? or separate `__tests__/` folder?]
- Constants files: [e.g. `constants.ts` per feature, or global `lib/constants.ts`]
- API/service files: [e.g. `userApi.ts`, `userService.ts`]

## 3. Code Naming Conventions
- Variables: camelCase
- Functions: camelCase
- Classes/Types/Interfaces: PascalCase
- Constants: [SCREAMING_SNAKE_CASE or camelCase — pick one]
- Enums: [PascalCase name, document value casing: `Status.PENDING` or `Status.Pending`]
- Boolean variables: prefix with `is`, `has`, `can`, `should` (e.g. `isLoading`, `hasError`)
- Event handlers: prefix with `on` or `handle` (e.g. `onSubmit`, `handleClick`)
- Custom hooks: always prefix with `use` — no exceptions

## 4. TypeScript Rules
- Strict mode: [enabled/disabled — paste the relevant tsconfig settings]
- `any` type: [forbidden / allowed with justification comment / test files only]
- Explicit return types: [required on all exported functions? optional? never?]
- `type` vs `interface`: [pick one for object shapes — document why]
- Type generation strategy: [manual? generated from DB schema via ORM/CLI?]
  - If generated: [which command, which output file, when to regenerate]
- `null` vs `undefined`: [document which to use for "absent" values, and where]
- `unknown` vs `any`: [prefer `unknown` for external data — document this rule]

## 5. Import Ordering
Define the exact order. AI must follow it consistently:
1. [External libraries: react, next, lodash, etc.]
2. [Internal path aliases: @/components, @/lib, @/hooks]
3. [Relative parent imports: ../utils, ../types]
4. [Relative sibling imports: ./Component, ./useHook]
5. [Type-only imports: import type { Foo } from '...']
6. [Style imports: ./Component.module.css, styles]

## 6. Component / Function Structure
For React components, enforce this order:
1. Props interface (TypeScript)
2. Component function declaration (named, not arrow function for components)
3. Hooks (state, refs, context)
4. Derived values (useMemo, useCallback)
5. Effects (useEffect)
6. Event handlers
7. Early returns (loading, error, empty states — REQUIRED for all data-fetching components)
8. Main return / JSX
9. Default export (bottom of file)

Size limits:
- Max function/component length: [e.g. 100 lines — if longer, extract helpers]
- Max file length: [e.g. 300 lines — if longer, split the file]

## 7. State Management Pattern
Document exactly where state lives at each level. AI must not invent its own pattern.

- Server/async state: [React Query / SWR / TanStack Query / custom hooks — pick one]
  - All API calls go through this layer, not raw fetch/axios in components
- Client/UI state: [useState for local / Zustand / Jotai / Context for shared]
  - Rule: default to local state. Escalate to shared only when 2+ unrelated components need it.
- Form state: [controlled components / React Hook Form / Formik — pick one]
- URL state: [query params for filterable/shareable views? document when]
- Global state: [what goes global? auth, theme, notifications? document the allowed list]

For non-React stacks, document equivalent pattern (Vuex/Pinia, NgRx, Svelte stores, etc.)

## 8. Error Handling Code Patterns
Paste the exact patterns AI must generate. Not principles — actual code structure.

### API call pattern:
```
[paste your standard try/catch or error handling wrapper]
```

### Form validation error display:
```
[paste your standard error message rendering pattern]
```

### Async useEffect with cleanup:
```
[paste your standard pattern including abort controller or cleanup]
```

### Empty / loading / error states (REQUIRED for all data-fetching components):
```
[paste your standard pattern — all three states must be handled]
```

## 9. Comment Standards
- When to comment: [only non-obvious logic / all public functions / minimal — pick one]
- JSDoc: [required for exported functions? optional? unused?]
- TODO format: `// TODO(author): description` — never commit unresolved TODOs to main
- Do NOT comment: self-explanatory code, closing braces, auto-generated sections

## 10. Linting & Formatting Config
- Formatter: [Prettier — note any non-default settings]
- Linter: [ESLint ruleset name — note project-specific overrides]
- Pre-commit hooks: [Husky / lint-staged / none — document what runs]
- AI must not suggest disabling or overriding these configs

## 11. Anti-Patterns (Explicit — AI Must Never Generate These)
This list goes into CLAUDE.md so it's present in every session.

Security:
- [ ] Never put API keys, tokens, or secrets in frontend code or committed files
- [ ] Never log auth tokens, passwords, or PII to console
- [ ] Never use `dangerouslySetInnerHTML` without explicit justification

Code quality:
- [ ] No `console.log` in production code (use the project's logging utility)
- [ ] No `any` type without a comment explaining why it can't be avoided
- [ ] No commented-out dead code committed to the repo
- [ ] No inline styles in JSX (use [Tailwind / CSS modules / styled-components])
- [ ] No direct DOM manipulation (use framework state)

UI completeness:
- [ ] Every data-fetching component MUST handle loading, empty, and error states
- [ ] Never return null from a component without an explicit empty state
- [ ] Never leave a failed API call silent — always surface something to the user

[Add project-specific anti-patterns here]
```

**Sections that prevent the most AI errors**: #4 (TypeScript rules prevent FM-17), #7 (State management prevents inconsistency), #11 (Anti-patterns prevent FM-15 and FM-16), #8 (Error patterns prevent FM-06 and FM-16)

**How to use CODESTYLE with AI**: After writing this document, copy Section 3 (naming), Section 4 (TypeScript rules), and all of Section 11 (anti-patterns) into your CLAUDE.md or per-session context file. The full CODESTYLE doc is the source of truth; CLAUDE.md is the always-visible excerpt.
