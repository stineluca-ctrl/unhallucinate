# unhallucinate

**Make the decisions before AI makes them for you.**

A planning-first framework for AI-coded software. You lock in the architecture, data model, security, and edge cases upfront - so AI executes your decisions instead of inventing its own.

**Created by [Luca Stine](https://github.com/stineluca-ctrl)** - built from shipping production software with AI and turning hard-won lessons into a repeatable system. Includes a Claude Skill called **spec-driven-dev** that automates the auditing and document generation.

---

## The Problem

AI code generation is powerful but unguarded: **the first 80% looks incredible.** Routes work, UI renders, data flows. Then you hit the last 20% - the error handling, the edge cases, the security - and the gaps show up.

The research tells us why:

- **45% of AI-generated code contains security vulnerabilities** (Veracode, 2025)
- AI-assisted PRs have **1.7x more issues** than human-authored PRs (CodeRabbit, 2025)
- When given rich context, hallucination rates drop from **54% to 16%** - proving the problem is context, not capability (Qodo, 2025)

The pattern is clear: AI guesses when specs are missing, and it loses track of specs when context isn't managed. This framework solves both problems - specs that lock in every decision AI would otherwise guess, and context engineering that makes sure AI actually follows them.

---

## How It Works

Two systems working together:

| System | Role | What It Does |
|--------|------|-------------|
| **spec-driven-dev** (Claude Skill) | Planning | Audits, generates, and maintains planning documents |
| **CLAUDE.md** | Bridge | Always-on context file with Document Registry routing AI to the right specs |

The spec-driven-dev skill creates the source of truth, and CLAUDE.md makes it reachable. You bring whatever execution tool you prefer.

> **My setup:** I use [GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) by GlitterCowboy (Taches) as my execution engine. The workflow below reflects that. GSD is not required. The spec framework works with any AI coding tool or workflow: GSD, Cursor, Aider, Claude Code directly, whatever you like. I'm just showing how I plug the pieces together and the problems I found along the way.

---

## Full Workflow

### Phase 1: Research

Gather everything before writing a single planning doc. Stakeholder interviews, meeting notes, existing spreadsheets, example files, competitor references, workflow observations. Dump it all into a `memory/` folder organized by type (people, context, projects, meetings). Nothing formal yet. Just raw material.

The goal: when you sit down to write specs, every answer should come from a source you can point to. If you can't point to a source, it's an open question, not something you invent.

### Phase 2: Audit

Run the spec-driven-dev skill in AUDIT mode against your project folder. It scans existing docs, maps them against the Document Registry, and outputs a gap analysis: what's covered, what's missing, and what's at risk. This tells you which documents to write first (highest-risk gaps).

The audit includes an **AI Decision Audit** that checks every decision AI would need to make during code generation and verifies the answer exists in a spec. If it doesn't, that decision *will* be hallucinated.

### Phase 3: Generate Planning Documents

Write documents in dependency order using the spec-driven-dev skill. Each builds on the last:

**ESSENTIAL (every project):**

| # | Doc | What It Locks In |
|---|-----|-----------------|
| 1 | **PRD** (Product Requirements Document) | Features, scope, user workflows, acceptance criteria |
| 2 | **DMS** (Data Model Specification) | Tables, columns, constraints, triggers, state machines |
| 3 | **TRD** (Technical Requirements Document) | Architecture, stack, integrations, security, error handling |
| 4 | **NFR** (Non-Functional Requirements) | Performance, security, compliance, audit, data retention |

**RECOMMENDED (production systems):**

| # | Doc | What It Locks In |
|---|-----|-----------------|
| 5 | **ARCH** (Architecture & Module Plan) | Module boundaries, folder structure, fault isolation |
| 6 | **CODESTYLE** (Coding Standards) | Naming, TypeScript rules, state management, anti-patterns |
| 7 | **ADR** (Architecture Decision Records) | Individual high-stakes decisions with reasoning |
| 8 | **API** (API/Integration Design) | Endpoints, contracts, error codes |
| 9 | **OQR** (Open Questions Register) | Unresolved decisions, assumptions, blockers |

**OPTIONAL (when relevant):**

| # | Doc | What It Locks In |
|---|-----|-----------------|
| 10 | **TEST** (Testing Strategy) | Framework, coverage targets, test data |
| 11 | **DEPLOY** (Deployment Plan) | CI/CD, environments, rollback |
| 12 | **MIGRATE** (Data Migration Plan) | Import logic, transformation rules, validation |

Every generated document follows strict tagging rules:

- **No tag** = sourced from existing docs (citable)
- `[INFERRED FROM: source]` = logically derived from a known fact
- `[ASSUMPTION: reason]` = reasonable default, needs confirmation
- `[DECISION NEEDED]` = stakeholder input required, includes options and trade-offs
- `[OPEN QUESTION: OQ-XX]` = maps to a tracked open question

Nothing is invented. If the AI doesn't know it, the doc says so explicitly. A wrong spec is worse than a missing spec, because wrong specs generate wrong code with confidence.

### Phase 4: Context Engineering

This is where most AI-coded projects fail silently. Great specs in a folder nobody opens prevent zero bugs.

**CLAUDE.md** gets a **Document Registry**:

```markdown
## Document Registry

Before starting any coding task, read ONLY the docs that match the task.
Always read: CODESTYLE (for any code) + DMS (for any database work).

| Doc | Path | Read When |
|-----|------|-----------|
| DMS | docs/dms.md | Any database, schema, trigger, state machine work |
| TRD | docs/trd.md | Auth, error handling, integrations, security |
| CODESTYLE | docs/codestyle.md | Writing any code (always) |
| ARCH | docs/arch.md | Creating files, folder structure decisions |
| NFR | docs/nfr.md | Performance, compliance, security requirements |
| PRD | docs/prd.md | Workflow questions, acceptance criteria, scope |
```

This routing table tells any coding AI which docs to read for which tasks. It reads 2-3 relevant docs per task instead of all 12.

**docs/INDEX.md** maps every section header in every planning doc to its line range and a brief description. The coding AI reads the index, finds the relevant section, then pulls only those lines. 150 lines of targeted spec instead of 1,500+.

**CLAUDE.md also includes:** project glossary, tech stack, naming conventions, key constraints, CODESTYLE anti-patterns (excerpted, not just referenced), and active open questions so AI flags them instead of guessing.

Keep CLAUDE.md under 400 lines. Longer means critical rules get buried in the attention dead zone.

### Phase 5: Build

Your execution tool takes over. The planning documents are the source of truth. Your coding workflow references them.

**How the systems connect:**

- **Planning docs** = what to build (specs, source of truth)
- **Your execution tool** = how to build it (task breakdown, coding, verification)
- **CLAUDE.md** = the bridge (Document Registry routes AI to the right spec sections per task)

The key property of any good execution workflow: each coding session starts with CLAUDE.md, sees the Document Registry, and loads only the spec sections relevant to its current task. No single session tries to hold the entire project in memory. Each one gets targeted context and writes focused code.

> **How I do it:** I use GSD, which handles this naturally. It breaks features into atomic tasks, spins up executor agents with fresh context windows (~200k tokens each), runs verification loops against acceptance criteria, and manages state files tracking progress. But the pattern works the same if you're just opening Claude Code and pointing it at a task. The Document Registry in CLAUDE.md does the routing regardless of what tool wraps it.

**What your execution tool should NOT do:** It shouldn't write or maintain your planning docs. That's the spec-driven-dev skill's job. Execution references specs, never restates them. If your coding AI needs information that isn't in the specs, that's a signal to update your planning docs first, not to let the AI guess.

### Phase 6: Maintain

Specs are living documents. When requirements change, stakeholders give feedback, or open questions get resolved:

1. Update the affected planning docs (using the spec-driven-dev skill)
2. Check for cascade effects (a PRD change may require DMS and TRD updates)
3. Update the OQR (mark resolved questions, add new ones)
4. Update docs/INDEX.md with new line ranges
5. The next coding session automatically picks up the changes through CLAUDE.md

---

## Document Dependency Tree

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

Always build on the foundation. Don't write a DMS without a PRD. Don't write an API spec without a TRD.

---

## Calibrating for Project Size

Not every project needs all 12 documents at full depth.

**Solo dev + AI, MVP/prototype (< 3 months)**
- ESSENTIAL: PRD (light), DMS (full), TRD (light)
- RECOMMENDED: ADRs for the 3-5 biggest decisions

**Solo dev + AI, production system (3-12 months)**
- ESSENTIAL: PRD, DMS, TRD, NFR (all full)
- RECOMMENDED: ADRs, API spec, OQR

**Small team (2-5 devs)**
- All ESSENTIAL and RECOMMENDED at full depth
- Testing strategy and deployment plan become essential

**The rule:** If AI is generating code, the DMS and TRD are always essential regardless of project size. CODESTYLE is always recommended. These three documents prevent the most hallucinations per hour of writing.

---

## Project Structure

```
unhallucinate/
├── README.md                          # This file
├── WORKFLOW.md                        # Step-by-step workflow guide
├── LICENSE
├── .github/
│   └── CONTRIBUTING.md
├── skill/
│   ├── SKILL.md                       # The Claude Skill definition
│   └── references/
│       ├── document-templates.md      # Full templates for all 12 doc types
│       ├── ai-failure-modes.md        # 17 cataloged failure modes
│       ├── context-engineering.md     # How to feed specs into AI context
│       ├── context-loading-runtime.md # Document Registry + Section Index pattern
│       └── audit-checklist.md         # Quick-scan checklist for audits
└── examples/
    ├── CLAUDE-md-template.md          # Template for CLAUDE.md with Document Registry
    └── INDEX-md-template.md           # Template for docs/INDEX.md Section Index
```

---

## Using the Skill

The spec-driven-dev skill is a [Claude Skill](https://code.claude.com/docs). Install it in your Claude environment and invoke it with natural language:

- *"Audit my project docs"* - runs AUDIT mode
- *"Create a DMS for my project"* - runs GENERATE mode for a specific doc
- *"What docs do I need?"* - runs AUDIT and recommends
- *"Check my project readiness"* - full audit with scoring

The skill handles the templating, tagging, cross-referencing, and Section Index updates. You focus on answering the questions it surfaces.

---

## The Core Principle

> Every `[DECISION NEEDED]` tag you resolve before coding starts is a bug that never gets written.

AI generates correct code when specs remove the need to guess. The spec-driven-dev skill ensures the specs exist and stay honest. Your execution workflow ensures the code follows them.

---

## Installation

### Use the skill

Copy the `skill/` folder into your Claude skills directory:

```bash
cp -r skill/ ~/.claude/skills/spec-driven-dev/
```

That's it. The SKILL.md reads its own `references/` folder at runtime.

### Use the framework without the skill

Read the [README](README.md) and [WORKFLOW.md](WORKFLOW.md). Use the templates in `examples/` to set up your `CLAUDE.md` and `docs/INDEX.md`. The concepts work with any AI coding tool.

---

## The Research

The data behind this framework. Everything above is built on these findings.

### The numbers

- **45% of AI-generated code contains security vulnerabilities** (Veracode, 2025)
- AI-assisted PRs have **1.7x more issues** than human-authored PRs (CodeRabbit, 2025)
- XSS vulnerabilities are **2.74x more likely** in AI-generated code (CodeRabbit, 2025)
- Change failure rate **increased ~30%** with AI adoption (Cortex, 2026)
- **65% of developers say AI misses relevant context** during refactoring, testing, and code review (Qodo, 2025)
- **Just 3% of developers highly trust** the accuracy of AI-generated code (Stack Overflow, 2025)
- When given rich context, hallucination rates drop from **54% to 16%** - proving the problem is context, not capability (Qodo, 2025)

### Why does AI fail at the last 20%?

Because the last 20% requires **decisions**, not just code. Error handling, state transitions, security boundaries, naming conventions, cascade behavior, concurrency, type safety. These are all decisions. When specs are missing, AI doesn't stop and ask. It guesses confidently. And those guesses compound into architectures that look functional until they aren't.

The research confirms this: there is **no direct correlation between functional performance (Pass@1 rate) and overall code quality or security.** Code that passes tests can still have vulnerabilities, type drift, and architectural debt - because those are all decisions AI made for you when no spec existed. This framework's core job is making those decisions upfront so AI executes them instead of inventing them.

### Context rot makes it worse

Even when you write good specs, there's a second problem: **getting them into AI's context window correctly.** Stanford research found that with just 20 retrieved documents (~4,000 tokens), accuracy drops 15-20 percentage points when important information sits in the middle of the context versus at the beginning or end. AI has a U-shaped attention curve. Critical rules buried in the middle of a massive context dump get ignored.

As one research team put it: *"The challenge isn't just crafting the perfect prompt. It's thoughtfully curating what information enters the model's limited attention budget at each step."*

### AI Failure Modes This Prevents

This framework catalogs 17 specific failure modes (FM-01 through FM-17) that occur when AI generates code without proper specs. Each maps to a document type:

| FM | Failure Mode | Severity | Prevented By |
|----|-------------|----------|-------------|
| FM-01 | ID Format Inconsistency | HIGH | DMS |
| FM-02 | Timestamp Chaos | HIGH | TRD |
| FM-03 | State Transition Violations | CRITICAL | DMS |
| FM-04 | Mutable Event Log | CRITICAL | DMS + ADR |
| FM-05 | Security by Absence | CRITICAL | TRD + DMS |
| FM-06 | Error Swallowing | HIGH | TRD |
| FM-07 | Naming Convention Drift | MEDIUM | DMS + CODESTYLE |
| FM-08 | N+1 Queries / Perf Cliffs | MEDIUM | TRD + DMS |
| FM-09 | Cascade Confusion | HIGH | DMS |
| FM-10 | Concurrency Blindness | CRITICAL | TRD + DMS |
| FM-11 | Implicit Business Rules | MEDIUM | PRD + DMS |
| FM-12 | Glossary Confusion | LOW | PRD |
| FM-13 | Missing Validation Boundaries | MEDIUM | DMS + PRD |
| FM-14 | Hardcoded Configuration | LOW | PRD + TRD |
| FM-15 | Secret/Key Exposure | CRITICAL | CODESTYLE + TRD |
| FM-16 | UI State Gaps | MEDIUM | CODESTYLE + PRD |
| FM-17 | Type Drift | HIGH | CODESTYLE + DMS |

See [skill/references/ai-failure-modes.md](skill/references/ai-failure-modes.md) for detailed descriptions, detection checks, and common wrong defaults for each.

---

## Sources and Further Reading

**Research cited in this README:**

- [Veracode - AI-Generated Code Security Risks (2025)](https://www.veracode.com/blog/ai-generated-code-security-risks/)
- [CodeRabbit - State of AI vs Human Code Generation (2025)](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report)
- [Qodo - State of AI Code Quality (2025)](https://www.qodo.ai/reports/state-of-ai-code-quality/)
- [Cortex - Engineering in the Age of AI Benchmark Report (2026)](https://www.cortex.io/post/ai-is-making-engineering-faster-but-not-better-state-of-ai-benchmark-2026)
- [Stack Overflow - 2025 Developer Survey, AI Section](https://survey.stackoverflow.co/2025/ai/)
- [Stanford - Lost in the Middle: How Language Models Use Long Contexts (2023)](https://arxiv.org/abs/2307.03172)
- [Georgetown CSET - Cybersecurity Risks of AI-Generated Code](https://cset.georgetown.edu/wp-content/uploads/CSET-Cybersecurity-Risks-of-AI-Generated-Code.pdf)
- [Chroma Research - Context Rot](https://research.trychroma.com/context-rot)
- [Andrej Karpathy on Vibe Coding](https://x.com/karpathy/status/1886192184808149383)
- [Simon Willison - Not All AI-Assisted Programming Is Vibe Coding](https://simonw.substack.com/p/not-all-ai-assisted-programming-is)

**Related tools:**

- [GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) by GlitterCowboy/Taches - Development workflow framework
- [Claude Code](https://code.claude.com/docs) - Anthropic's CLI coding agent
- [Claude Code Skills](https://code.claude.com/docs/skills) - Skill system documentation

---

## License

MIT
