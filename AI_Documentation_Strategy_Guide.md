# AI Tooling Documentation Strategy Guide

> **Preparing Your Repository for Claude Code, Cursor, or GitHub Copilot**
>
> `v1.3` · Engineering & Platform Team

---

## About This Guide

AI coding assistants are only as effective as the context they can access. This guide provides a systematic, actionable framework for structuring your repository documentation so that your chosen AI tool — whether Claude Code, Cursor, or GitHub Copilot — can provide accurate, project-aware assistance from day one.

Each tool has its own instruction file format and loading behavior. This guide covers the **shared documentation practices** that benefit any AI tool (§1–2, §4–8), alongside **tool-specific instruction file guidance** for each of the three tools (§3.2–3.4). Pick the section that matches the tool your project uses.

By following this strategy, your team will reduce AI hallucinations, improve suggestion relevance, accelerate onboarding, and establish a self-maintaining documentation culture that compounds value over time.

---

## Table of Contents

1. [Why Documentation Drives AI Effectiveness](#1-why-documentation-drives-ai-effectiveness)
2. [Documentation Readiness Audit](#2-documentation-readiness-audit)
3. [AI Instruction Files (Highest Leverage)](#3-ai-instruction-files-highest-leverage)
   - [3.1 Quick Comparison](#31-quick-comparison)
   - [3.2 Claude Code — CLAUDE.md](#32-claude-code--claudemd)
   - [3.3 Cursor — .cursor/rules/*.mdc](#33-cursor--cursorrulesmdc)
   - [3.4 GitHub Copilot — copilot-instructions.md](#34-github-copilot--githubcopilot-instructionsmd)
4. [Core Repository Documentation Files](#4-core-repository-documentation-files)
   - [4.7 Data Layer Documentation](#47-data_modelsmd--data-layer-documentation)
5. [Inline Code Documentation Standards](#5-inline-code-documentation-standards)
6. [Advanced Context & RAG Strategies](#6-advanced-context--rag-strategies)
   - [6.5 Context Exclusion](#65-context-exclusion--controlling-what-ai-does-not-index)
   - [6.6 Reusable Prompt Templates](#66-reusable-prompt-templates)
   - [6.7 Model Context Protocol (MCP)](#67-model-context-protocol-mcp)
7. [Implementation Roadmap](#7-implementation-roadmap)
8. [Documentation Maintenance & Governance](#8-documentation-maintenance--governance)
   - [8.4 Measuring Documentation Effectiveness](#84-measuring-documentation-effectiveness)
   - [8.5 Multi-Language and Polyglot Repository Guidance](#85-multi-language-and-polyglot-repository-guidance)
9. [Dos and Don'ts Quick Reference](#9-dos-and-donts-quick-reference)
10. [Appendix: File Templates & Resources](#10-appendix-file-templates--resources)

---

## 1. Why Documentation Drives AI Effectiveness

AI coding tools do not infer intent from code alone. They build context from the signals they can read: file names, directory structure, comments, configuration files, and — most importantly — purpose-built documentation artefacts. A well-documented repository narrows the search space the model has to reason over, resulting in higher-confidence completions, fewer incorrect API references, and architectural suggestions that actually fit your stack.

### 1.1 The Context Hierarchy

AI tools consume context in a predictable priority order. Understanding this hierarchy lets you invest documentation effort where it has the greatest leverage:

| Priority | Source | What the AI Reads | Investment Effort |
|---|---|---|---|
| **1 — Highest** | Active file + selection | Current code, inline comments, types | Low (already exists) |
| **2** | Open editor tabs | Related files open in the IDE | Low |
| **3** | Workspace rules / prompts | `CLAUDE.md`, `.cursor/rules/`, `.github/copilot-instructions.md` | Medium — covered in §3 |
| **4** | Repo-wide indexing | All `.md`, config, source files | High — covered in §4 |
| **5 — Lowest** | External retrieval (RAG) | Docs, APIs, knowledge bases | High — covered in §6 |

> **💡 KEY:** Invest first in Priority 3 and 4 artefacts — they are universal across all AI tools and have the highest leverage-to-effort ratio.

### 1.2 Common Failure Modes Without Documentation

- AI suggests libraries or patterns from a different tech stack because the stack is undeclared
- Completions contradict your architecture because architectural decisions are not recorded
- Generated tests miss your testing conventions because no test guidelines exist
- AI duplicates helpers that already exist in utilities because no module map is available
- Security-sensitive areas receive generic code because no sensitivity markers are present

---

## 2. Documentation Readiness Audit

Before creating new documents, audit what already exists. The goal is to identify gaps, not to duplicate. Run this checklist against your repository root and each top-level module or service.

### 2.1 Quick-Scan Audit Checklist

| Artefact | Present? | Priority if Missing |
|---|---|---|
| `README.md` — project overview & quick start | ☐ Yes ☐ No | **Critical** |
| `ARCHITECTURE.md` — system design & component map | ☐ Yes ☐ No | **Critical** |
| AI instruction file for your tool (`CLAUDE.md` or `.cursor/rules/` or `.github/copilot-instructions.md`) | ☐ Yes ☐ No | **Critical** |
| `CONTRIBUTING.md` — code standards & PR process | ☐ Yes ☐ No | High |
| `docs/adr/` — architectural decision records | ☐ Yes ☐ No | High |
| `DATA_MODELS.md` — entity definitions, relationships, DB conventions | ☐ Yes ☐ No | High |
| API reference or OpenAPI spec | ☐ Yes ☐ No | Medium |
| Module-level `README` files | ☐ Yes ☐ No | Medium |
| `SECURITY.md` — sensitive areas & patterns to avoid | ☐ Yes ☐ No | **High** |
| `TESTING.md` — test strategy & how to run tests | ☐ Yes ☐ No | Medium |
| `CHANGELOG.md` — structured change history | ☐ Yes ☐ No | Low |
| `GLOSSARY.md` — domain & team terminology | ☐ Yes ☐ No | Low |
| `ONBOARDING.md` — new-developer guide | ☐ Yes ☐ No | Low (only if README quick start is insufficient for your setup) |

### 2.2 Scoring Your Readiness

| Score | Readiness Level | Recommended Action |
|---|---|---|
| 0–3 items | 🔴 Not Ready | Complete §3 and §4 before enabling AI tooling |
| 4–7 items | 🟡 Partial | Fill Critical/High gaps, then enable AI tooling |
| 8–10 items | 🟢 Good | Enable AI tooling; schedule Medium-gap sprints |
| 11–13 items | ✅ Excellent | Enable AI tooling; focus on maintenance cadence |

---

## 3. AI Instruction Files (Highest Leverage)

AI instruction files are purpose-built context injected directly into the AI tool on every request. They are the **single highest-leverage documentation investment** you can make.

### 3.1 Quick Comparison

| | **Claude Code** | **Cursor** | **GitHub Copilot** |
|---|---|---|---|
| **File** | `CLAUDE.md` | `.cursor/rules/*.mdc` | `.github/copilot-instructions.md` |
| **Size limit** | ~200 lines per file | ~500 lines per rule file | ~2 pages total |
| **Multiple files** | Yes — subdirectory hierarchy | Yes — multiple `.mdc` files | Yes — `.github/instructions/*.instructions.md` |
| **Path scoping** | Subdirectory placement + `.claude/rules/` with `paths:` frontmatter | `globs:` frontmatter per rule | `applyTo:` frontmatter per file |
| **Loading** | All files injected every request; subdirectories loaded lazily | Depends on rule type (always / intelligent / file-scoped / manual) | Automatically added to every Copilot Chat request |
| **File references** | `@path/to/file` imports content inline | `@filename` references in rule text | Not supported in instructions |
| **Auto-generate** | `/init` command | Not available | `/init` command |
| **MCP support** | Native | Native | Native (VS Code, JetBrains, Xcode, coding agent) |
| **Hierarchy** | Managed policy > Project > User > Subdirectory | Team > Project > User | Personal > Repository > Organization |
| **Enforcement** | Advisory — use hooks for hard enforcement | Advisory | Advisory |

> **💡 TIP:** Each project typically adopts **one** AI tool. Use the comparison above to identify which tool your project uses, then follow the corresponding section (§3.2, §3.3, or §3.4) for detailed guidance on writing its instruction files.

---

### 3.2 Claude Code — `CLAUDE.md`

#### How It Works

- Claude Code reads `CLAUDE.md` from disk and injects the **full content into every API request**
- It walks **upward** from the working directory to the repo root, loading all `CLAUDE.md` files it finds
- Subdirectory `CLAUDE.md` files load **lazily** — only when Claude reads files in that directory
- All levels contribute simultaneously; more specific files take priority when instructions overlap
- Instructions are **advisory context**, not enforced configuration — Claude interprets them with judgment
- Content **survives `/compact`** (re-read from disk), unlike conversational instructions

#### Size and Structure

**Target: under 200 lines per file.** Files over 200 lines cause instruction adherence to degrade — critical rules get lost in the noise. Maximum practical limit is ~500 lines.

**Structure with markdown headers and bullets, not prose:**

```markdown
# Project: PaymentCore

PaymentCore is the internal payment processing service for Acme Corp.
It handles charge authorisation, refunds, webhook dispatch, and reconciliation.
Domain: FinTech / PCI-DSS scope. Language: TypeScript (Node 20). Runtime: AWS Lambda.
Do NOT suggest UI or front-end code — this is a backend-only service.

## Stack
- Language:   TypeScript 5.3 (strict mode)
- Runtime:    Node.js 20 LTS
- Framework:  Fastify 4.x   (NOT Express)
- Database:   PostgreSQL 15 via Drizzle ORM  (NOT Prisma, NOT TypeORM)
- Queues:     AWS SQS + EventBridge
- Testing:    Vitest 1.x  (NOT Jest)
- Linting:    ESLint + Prettier (config in .eslintrc.json)

## Repository Layout
src/
  api/          # Fastify route handlers — HTTP boundary only, no business logic
  domain/       # Core business logic — pure functions, no I/O
  infra/        # DB adapters, queue clients, external HTTP clients
  shared/       # Types, constants, utilities shared across layers
tests/
  unit/         # Pure function tests (Vitest)
  integration/  # DB + queue tests (requires Docker Compose)
  e2e/          # Full API tests against local stack

## Coding Conventions
- Use Result<T, E> pattern for error handling — never throw in domain layer
- Name files kebab-case; name classes PascalCase; name functions camelCase
- Co-locate tests with source: foo.ts + foo.test.ts in the same directory
- No default exports — always use named exports
- Never use `any`; use `unknown` and narrow with type guards

## Architectural Constraints
- Domain layer must have ZERO imports from infra/ or api/
- All DB access goes through a Repository interface (never query directly in handlers)
- PCI data (card numbers, CVVs) must never be logged or stored — use token references only
- All external HTTP calls must use the HttpClient wrapper in infra/http — not raw fetch/axios

## Reuse These Before Writing New Code
- src/shared/result.ts      — Result<T,E> type + helpers (ok, err, isOk)
- src/shared/pagination.ts  — cursor-based pagination logic
- src/shared/validation.ts  — Zod schema factories for common shapes
- src/infra/db/base-repo.ts — BaseRepository with findById, save, delete
- src/shared/logger.ts      — structured logger (pino) — never use console.log

## Data Layer
- Database: PostgreSQL 15, tables snake_case plural, columns snake_case
- ORM: Drizzle — all access through Repository interfaces (see infra/db/base-repo.ts)
- Money: stored as INTEGER in cents — never use float
- Soft deletes: deleted_at column — never hard-delete user data
- Pagination: cursor-based only — never OFFSET/LIMIT for user-facing queries
- Full entity definitions and relationships: @DATA_MODELS.md

## Testing
- Every exported function in domain/ must have a unit test
- Use test.each for data-driven cases
- Mock all I/O at the infra boundary using vitest.mock()
- Integration tests must clean up DB state with afterEach(cleanDb)
- Run: pnpm test (unit), pnpm test:int (integration), pnpm test:e2e (e2e)

## Quick Reference
DO:    Follow Result<T,E> pattern · Add Zod validation at API boundary · Write tests with every domain function
DO NOT: Import from infra/ inside domain/ · Log PCI data · Use Express, Jest, Prisma, or any unlisted library
```

#### Eight Essential Sections

Every `CLAUDE.md` should cover these topics (order matters — the model reads top-down):

| # | Section | Purpose | Key Rule |
|---|---|---|---|
| A | **Project Identity** | Ground the model in the correct domain | 3–5 sentences: what, who, domain, language, what NOT |
| B | **Technology Stack** | Prevent hallucinated libraries | Exact versions + explicit "NOT X" exclusions |
| C | **Repository Structure** | Clarify module boundaries | Commented directory tree |
| D | **Coding Conventions** | Align naming, patterns, anti-patterns | Be explicit about what NOT to do |
| E | **Architectural Constraints** | Prevent the most costly mistakes | Hard rules — use "NEVER" for emphasis |
| F | **Common Patterns** | Prevent reinventing existing helpers | File paths + when-to-use descriptions |
| G | **Testing Requirements** | Align test approach | Exact run commands + coverage expectations |
| H | **Data Layer** | Prevent schema hallucinations | Entity names, DB conventions, data access rules — or reference `DATA_MODELS.md` |
| I | **Do / Do Not Summary** | Quick-scan safety net | Plain-language summary for chat-based usage |

#### Advanced Features

**Imports** — Pull in content from other files without copying:

```markdown
See @README.md for project overview.
Git workflow: @docs/git-instructions.md
```

**Path-scoped rules** — Use `.claude/rules/` with frontmatter for file-type targeting:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API endpoint rules — all handlers must validate input with Zod
```

**Hierarchical placement for monorepos:**

```
repo-root/
  CLAUDE.md                    # Cross-cutting: CI, repo structure, shared naming
  backend/
    CLAUDE.md                  # Go conventions, backend-specific constraints
  frontend/
    CLAUDE.md                  # TypeScript/React conventions
```

**`/init` command** — Run in your repo to auto-generate a starter `CLAUDE.md` by analyzing your codebase. Always start here, then refine.

#### What NOT to Put in CLAUDE.md

| Exclude | Why | Alternative |
|---|---|---|
| Standard language conventions | Claude already knows them | Only document project-specific deviations |
| Full API documentation | Too long, wastes context | Link to it or use MCP |
| Personal preferences | Should not be in team-shared file | Use `~/.claude/CLAUDE.md` for personal settings |
| Frequently changing data | Goes stale quickly | Use MCP for dynamic context |
| File-by-file descriptions | Claude can read the code | Let it explore; document boundaries only |

---

### 3.3 Cursor — `.cursor/rules/*.mdc`

#### How It Works

- Cursor loads rules from `.cursor/rules/` directory (`.mdc` or `.md` files)
- Rules are injected **at the start of the model context**, consuming tokens from the context window
- The old `.cursorrules` single-file format still works but lacks advanced features — migrate to `.cursor/rules/`
- Each rule file can have **YAML frontmatter** controlling when and how it loads

#### Size and Structure

**Target: under 500 lines per rule file.** Split larger rulesets into multiple focused files. Use the directory structure to organise by concern.

**Recommended directory layout:**

```
.cursor/rules/
├── project.mdc          # alwaysApply: true — project identity + stack
├── typescript.mdc       # globs: ["**/*.ts", "**/*.tsx"]
├── testing.mdc          # globs: ["**/*.test.ts", "**/*.spec.ts"]
├── api/
│   └── endpoints.mdc    # globs: ["src/api/**/*"]
└── security.mdc         # alwaysApply: true — security constraints
```

#### The Four Rule Types

This is Cursor's most powerful feature — **intelligent context loading** that minimises token waste:

| Type | Frontmatter | When Loaded | Best For |
|---|---|---|---|
| **Always** | `alwaysApply: true` | Every request | Project identity, stack, security constraints — use sparingly |
| **Intelligent** | `description: "..."` | Agent decides based on description match | Context-dependent conventions that apply sometimes |
| **File-scoped** | `globs: ["**/*.ts"]` | When matching files are active | Language-specific or directory-specific rules |
| **Manual** | (none needed) | Only when user types `@rule-name` | Specialized workflows, migration guides |

> **⚠️ WARNING:** Making everything `alwaysApply: true` defeats the purpose. Only project identity, stack declaration, and security constraints should always apply. Everything else should use intelligent or file-scoped loading.

#### Example `.mdc` Rule File

```yaml
---
description: "TypeScript coding conventions for source files"
alwaysApply: false
globs: ["**/*.ts", "**/*.tsx"]
---

# TypeScript Conventions

- Use `unknown` instead of `any`; narrow with type guards
- Named exports only — no default exports
- Use Result<T,E> for error handling in domain layer
- Import order: node builtins → external → internal → relative

## Anti-Patterns
- Never use `eval()` or `Function()` constructor
- Never use `as any` type assertion
- Never use default exports
```

#### Key Best Practices

- **Write clear `description` fields** — The agent uses this text to decide whether to load the rule. Write it like a search query: *"TypeScript coding conventions for source files"* not *"Rules"*
- **Reference files with `@`** — Use `@src/shared/result.ts` to show the AI real code instead of describing it in prose
- **One concern per file** — A testing rule file, a security rule file, an API conventions file. Not one monolith.
- **Use nested folders** — `.cursor/rules/api/`, `.cursor/rules/testing/` for complex projects
- **Team Rules** — Cursor supports team-level rules that apply across all team members; use for org-wide standards

#### Common Mistakes

- Making all rules `alwaysApply: true` — wastes context on every request
- Vague or missing `description` fields — agent can't match rules intelligently
- Copying entire style guides verbatim — reference files instead
- Single monolithic rule file — split by concern for better scoping
- Duplicating conventions the AI already knows — only add project-specific deviations

---

### 3.4 GitHub Copilot — `.github/copilot-instructions.md`

#### How It Works

- Copilot reads `.github/copilot-instructions.md` and **automatically adds it to every Copilot Chat request**
- Instructions activate **immediately on file save** — no restart needed
- Three levels are provided simultaneously (they don't override, they merge): Personal > Repository > Organization
- Verify your instructions are loaded by expanding the **"Used n references"** dropdown in Copilot Chat responses

#### Size and Structure

**Target: under 2 pages.** This is an official GitHub limit. Copilot's single-file approach means you must be concise — every line must earn its place.

**Use plain markdown with natural language. Keep it scannable:**

```markdown
# Project: PaymentCore

Backend payment processing service. TypeScript 5.4, Fastify 4.x, PostgreSQL 15 via Drizzle ORM.

## Conventions
- Use Fastify, NOT Express
- Use Vitest, NOT Jest
- Use Drizzle ORM, NOT Prisma or TypeORM
- Named exports only — no default exports
- Result<T,E> pattern for error handling — never throw in domain layer
- kebab-case files, PascalCase classes, camelCase functions

## Architecture
- domain/ has zero imports from infra/ or api/
- All DB access through Repository interfaces
- All HTTP calls through HttpClient wrapper in infra/http

## Security
- Never log or store PCI data (card numbers, CVVs)
- JWT validation happens at API gateway — do not re-validate in domain

## Testing
- Vitest for all tests
- Co-locate tests: foo.ts + foo.test.ts
- Mock I/O at infra boundary using vitest.mock()
- Run: pnpm test (unit), pnpm test:int (integration)
```

#### Path-Specific Instructions

For per-language or per-directory rules, create files in `.github/instructions/`:

```
.github/
├── copilot-instructions.md                    # Repo-wide (always loaded)
└── instructions/
    ├── typescript.instructions.md             # applyTo: "**/*.ts,**/*.tsx"
    ├── api-endpoints.instructions.md          # applyTo: "src/api/**/*"
    └── testing.instructions.md                # applyTo: "**/*.test.ts"
```

**Each file uses `applyTo` frontmatter:**

```yaml
---
applyTo: "**/*.ts,**/*.tsx"
---

TypeScript-specific instructions:
- Use `unknown` instead of `any`
- Prefer named exports over default exports
```

#### Priority Hierarchy

| Level | Location | Priority | Use Case |
|---|---|---|---|
| **Personal** | VS Code settings / Chat Customizations editor | Highest | Individual developer preferences |
| **Repository** | `.github/copilot-instructions.md` | Medium | Team conventions, project standards |
| **Organization** | GitHub org settings | Lowest | Company-wide policies |

All three levels are **provided simultaneously** — they don't override each other. Avoid conflicting instructions across levels.

#### Key Best Practices

- **Keep instructions general**, not task-specific — they apply to every interaction
- **Use `/init`** to auto-generate a starter file from your codebase
- **Verify activation** — check "Used n references" in chat responses
- **Use path-specific files** for per-language rules instead of cramming everything into one file
- **Coordinate with org-level** — know what your organization already provides before duplicating

#### Common Mistakes

- Exceeding 2 pages — Copilot may truncate or degrade quality
- Task-specific instructions — should be general conventions, not "fix bug #123" style
- Not verifying activation — instructions may silently not load if malformed
- Conflicting org/repo/personal instructions — all levels merge, conflicts confuse the model
- Not using `.github/instructions/` for scoped rules — cramming everything into one file

---

## 4. Core Repository Documentation Files

### 4.1 README.md — The Entry Point

The README is indexed by every AI tool. It must be scannable and structured. Include the following sections at minimum:

- Project name, one-sentence description, and badges (CI status, coverage, license)
- Quick start: clone → install → run (under 5 commands)
- Environment variable table: name, required/optional, example value, description
- Links to deeper documentation (`ARCHITECTURE.md`, `CONTRIBUTING.md`, etc.)
- Ownership: team name, Slack channel, PagerDuty rotation

> **⚠️ AVOID:** Long prose history sections in README. Move narrative context to `ARCHITECTURE.md`. README should answer *"How do I run this in 5 minutes?"*

### 4.2 ARCHITECTURE.md — System Context

This file is the most valuable for reducing AI hallucinations about system design. It should contain:

- A C4-style context diagram (ASCII art or Mermaid) showing external dependencies
- Component inventory: each major module, its responsibility, and what it must not do
- Data flow description for the 2–3 most important user journeys
- Scalability and performance design decisions (e.g., *"we use read replicas for reporting queries"*)
- Known limitations and explicit anti-patterns that were considered and rejected

**Example component map:**

```
┌─────────────┐     HTTP     ┌──────────────┐    SQS     ┌──────────────┐
│  API Gateway │ ──────────► │  PaymentCore │ ─────────► │ Notification │
└─────────────┘             │   (this svc) │            │   Service    │
                            └──────┬───────┘            └──────────────┘
                                   │ Drizzle ORM
                            ┌──────▼───────┐
                            │  PostgreSQL  │
                            │  (RDS, PG15) │
                            └──────────────┘
```

### 4.3 CONTRIBUTING.md — Conventions Contract

This document is the AI's reference for code standards. Treat it as a machine-readable spec by using consistent heading names and structured lists rather than prose.

- Branch naming: `feature/`, `fix/`, `chore/`, `docs/` prefixes
- Commit message format: Conventional Commits (`feat:`, `fix:`, `chore:`, etc.)
- PR requirements: linked issue, passing CI, one approval, no unresolved threads
- Definition of Done: tests written, docs updated, no linting errors, security scan passed
- Code review SLA: 1 business day for initial review

### 4.4 Architectural Decision Records (ADRs)

ADRs are short documents that record *why* a significant decision was made. They prevent the AI from suggesting alternatives that were already evaluated and rejected. Use the MADR template (see [Appendix B](#b-adr-template-madr)).

Store ADRs in `/docs/adr/NNNN-short-title.md`. Include an index in `/docs/adr/README.md`. The AI will surface these when generating code that touches the relevant area.

**Example ADR:**

```markdown
# ADR-0012: Use Drizzle ORM Instead of Prisma

**Status:** Accepted  |  **Date:** 2025-09-15

## Context
We needed a type-safe ORM compatible with our monorepo build pipeline.

## Decision
Adopt Drizzle ORM 0.29.x.

## Alternatives Considered
- Prisma: rejected — incompatible with our edge runtime deployment target
- TypeORM: rejected — poor TypeScript inference, decorator-heavy syntax
- Raw SQL (kysely): rejected — too verbose for team size and velocity requirements

## Consequences
Migrations are manual SQL files in /db/migrations. No automatic schema push.
```

### 4.5 SECURITY.md

Security context prevents AI tools from inadvertently generating vulnerable code. This file should define:

- PII and sensitive-data handling rules (what must be hashed, tokenised, or never stored)
- Authentication and authorisation patterns in use (e.g., *"JWT validated at the API gateway — never re-validate in domain code"*)
- Banned functions or patterns (e.g., *"never use `eval()`, `innerHTML`, `exec()`"*)
- Third-party dependency policy (approved registry, vulnerability scanning tooling)
- Incident classification and reporting path

### 4.6 TESTING.md

- Test pyramid: unit / integration / e2e ratios and rationale
- How to run each tier locally with exact commands
- Coverage thresholds and how they are enforced
- Test data management: fixtures, factories, database seeding approach
- How to write mocks for the team's core abstractions

### 4.7 DATA_MODELS.md — Data Layer Documentation

The data layer is where AI tools produce some of their **most dangerous hallucinations**: wrong column names, incorrect joins, bypassed security abstractions, and invented relationships. Explicit data layer documentation has very high leverage for AI accuracy.

#### What to Include

**Entity inventory with relationships:**

```markdown
# Data Models

## Core Entities

### User
| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | UUID | PK, generated | Never exposed externally — use `external_id` |
| external_id | UUID | unique, indexed | Public-facing identifier for API responses |
| email | VARCHAR(255) | unique, not null | Normalized to lowercase on write |
| password_hash | VARCHAR(255) | not null | bcrypt — never log or return in API responses |
| status | ENUM | not null, default 'active' | Values: active, suspended, deleted |
| created_at | TIMESTAMPTZ | not null, default now() | |
| deleted_at | TIMESTAMPTZ | nullable | Soft delete — never hard-delete user rows |

### Order
| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | UUID | PK, generated | |
| user_id | UUID | FK → User.id, indexed | CASCADE on soft-delete via application logic, not DB |
| total_amount | INTEGER | not null | Stored in **cents** — never use float for currency |
| currency | CHAR(3) | not null, default 'USD' | ISO 4217 codes only |
| status | ENUM | not null | Values: draft, pending, paid, refunded, cancelled |

### Relationships
- User → Order: one-to-many (user_id FK)
- Order → OrderItem: one-to-many (order_id FK)
- Order → Payment: one-to-one (order_id FK, unique)
```

#### Database Naming Conventions

Declare these explicitly — AI tools default to inconsistent naming without guidance:

- Table names: `snake_case`, **plural** (e.g., `users`, `order_items`)
- Column names: `snake_case` (e.g., `created_at`, `user_id`)
- Foreign keys: `<referenced_table_singular>_id` (e.g., `user_id`, `order_id`)
- Indexes: `idx_<table>_<column>` (e.g., `idx_orders_user_id`)
- Enums: defined as database-level types or application-level constants (state which one)

#### Data Access Patterns

Document how code interacts with the database — this prevents AI from bypassing your abstractions:

```markdown
## Data Access Rules

- All database access MUST go through Repository interfaces — never write raw queries in handlers or domain logic
- Use Unit of Work pattern for multi-table transactions
- Read replicas: all read-only queries use the read connection pool (see `infra/db/read-pool.ts`)
- N+1 prevention: always use eager loading / joins for known relationship traversals
- Pagination: cursor-based only — never use OFFSET/LIMIT for user-facing queries
- Soft deletes: `deleted_at IS NULL` filter is applied automatically by BaseRepository — never hard-delete
```

#### Migration Strategy

State your approach so AI generates migrations in the correct format:

```markdown
## Migrations

- Tool: Drizzle Kit (`drizzle-kit generate` → manual review → `drizzle-kit push`)
- Migration files: SQL in `/db/migrations/NNNN_description.sql`
- Naming: sequential numeric prefix + snake_case description
- Every migration must be reversible — include a `-- DOWN` section
- Never modify an existing applied migration — always create a new one
- Run: `pnpm db:migrate` (apply), `pnpm db:rollback` (revert last)
```

#### Adapting by Architecture

The sections you include in `DATA_MODELS.md` depend on your architecture. Use this as a guide:

| Section | Monolith | Microservices | Modular Monolith |
|---|---|---|---|
| Entity inventory (tables, columns, types) | Yes | Yes (per service) | Yes (per module) |
| Relationships (FKs, joins) | Yes — direct FKs | **Within service only** — no cross-service FKs | Within module only |
| Database naming conventions | Yes | Yes (per service, or shared org standard) | Yes |
| Data access patterns (Repository, pagination) | Yes | Yes (per service) | Yes (per module) |
| Migration strategy | Yes | Yes (per service) | Yes |
| Data ownership boundaries | N/A | **Yes — critical** | Yes |
| Cross-service references | N/A | **Yes — reference-by-ID, not FK** | Reference-by-module-ID |
| Published / consumed events | Optional | **Yes — critical** | Optional |
| Local projections (read-only copies) | N/A | Yes, if applicable | N/A |

**File placement by architecture:**

```
# Monolith / Modular Monolith — single repo
repo-root/
  DATA_MODELS.md              # All entities in one file

# Microservices — monorepo
repo-root/
  DATA_MODELS.md              # Cross-cutting: shared conventions, data ownership map
  services/
    order-service/
      DATA_MODELS.md           # Entities owned by this service only
    user-service/
      DATA_MODELS.md           # Entities owned by this service only

# Microservices — polyrepo
each-service-repo/
  DATA_MODELS.md              # Entities owned by this service only
```

**Microservices: document cross-service boundaries explicitly.** AI tools will attempt to create direct FKs and joins across service boundaries unless told not to:

```markdown
## Data Ownership
| Entity | Owning Service | Other Services Reference Via |
|---|---|---|
| User | user-service | user_id (UUID) — no FK, no join |
| Order | order-service | order_id (UUID) — no FK, no join |
| Product | catalog-service | product_id (UUID) — local projection in order-service |

## Cross-Service Rules
- NEVER create foreign keys to tables owned by another service
- NEVER join across service boundaries — use API calls or local projections
- Reference external entities by ID only (e.g., `user_id UUID` with no FK constraint)
- Document which events keep local projections in sync

## Published Events
| Event | Payload | Consumers |
|---|---|---|
| order.created | { order_id, user_id, total_amount, currency } | notification-service, analytics-service |
| order.paid | { order_id, paid_at } | fulfillment-service |
```

> **💡 TIP:** For AI instruction files (`CLAUDE.md`, `.cursor/rules/`, Copilot instructions), include a condensed summary of your data layer rules. Put the full `DATA_MODELS.md` at the repo root and reference it: `See @DATA_MODELS.md for entity definitions and database conventions.`

> **📏 RULE:** If the AI generates a query referencing a column name that does not exist in `DATA_MODELS.md`, the documentation has a gap. Track these as "data model hallucinations" (see §8.4) and update the file.

---

## 5. Inline Code Documentation Standards

Repository-level files provide macro context. Inline documentation provides micro context — the closest signal to where the AI is generating code. Both are necessary.

### 5.1 JSDoc / TSDoc / Type Annotations

AI tools extract type information and JSDoc comments to improve parameter suggestions and return-type inference. Prioritise annotations on:

- All exported functions and classes in public modules
- Functions with non-obvious side effects or external calls
- Complex generic types — include a usage example in the `@example` tag
- Functions that must NOT be used in certain contexts (annotate with `@internal` or a comment)

```typescript
/**
 * Authorises a payment charge against the external processor.
 *
 * @param charge - Validated charge request. Card data must be tokenised.
 * @returns Result<AuthorisationResponse, PaymentError>
 *
 * @throws Never — all errors are returned as Err variants.
 *
 * @example
 * const result = await authoriseCharge({ tokenId: 'tok_xxx', amountPence: 1000 });
 * if (isOk(result)) { ... }
 *
 * @security This function calls PCI-scope infrastructure. Do not add logging of params.
 */
export async function authoriseCharge(
  charge: ChargeRequest,
): Promise<Result<AuthorisationResponse, PaymentError>> { ... }
```

### 5.2 Module-Level README Files

For each non-trivial directory, add a `README.md` that explains:

- The single responsibility of this module
- What it imports from and what imports from it
- The 2–3 most important types or functions and when to use each
- What belongs here and what explicitly does **NOT** belong here

> **📏 RULE:** The module README test: if a new developer can answer *"where should I put X?"* by reading the README in under 30 seconds, it is good enough for AI context too.

### 5.3 TODO and FIXME Conventions

Standardise annotation keywords so AI tools can reason about technical debt:

| Keyword | Meaning | Expected AI Behaviour |
|---|---|---|
| `TODO(owner):` | Planned work with an owner | Flag if asked to complete; don't auto-complete |
| `FIXME:` | Known bug, avoid area | Warn before generating code in this block |
| `HACK:` | Temporary workaround, explain why | Surface the comment in suggestions |
| `SECURITY:` | Security-sensitive constraint | Never override; flag to developer |
| `PERF:` | Performance-critical path | Avoid allocations; prefer in-place ops |

---

## 6. Advanced Context & RAG Strategies

### 6.1 Curated Context Files for Large Repos

For large monorepos, full indexing is expensive and noisy. Create curated summary files that surface the signal a developer actually needs:

- `MODULE_MAP.md` — flat list of modules with a one-line description and entry-point file path
- `DEPENDENCIES.md` — key library decisions with version constraints and links to relevant ADRs
- `DATA_MODELS.md` — now a core artefact (see §4.7); for large repos, supplement with per-service data model files
- `API_CONTRACTS.md` — endpoint inventory with auth requirements and rate-limit notes

### 6.2 OpenAPI / AsyncAPI Specifications

Machine-readable API specifications are the most precise context you can provide for API-adjacent code generation. Ensure your specs are:

- Co-located with the service code (not in a separate docs repository that can drift)
- Committed and CI-validated against the actual running service
- Annotated with `x-internal-notes` fields for AI-specific guidance on sensitive endpoints

### 6.3 Environment and Configuration Documentation

Undocumented environment variables cause AI tools to generate hardcoded values. Add an `.env.example` file with every variable commented:

```bash
# .env.example

# --- Database ---
# Required. PostgreSQL connection string. Use connection pooling URL for production.
DATABASE_URL=postgresql://user:password@localhost:5432/paymentcore

# --- Feature Flags ---
# Optional. Set to 'true' to enable 3DS2 challenge flow in development.
FF_3DS2_ENABLED=false

# --- External Services ---
# Required. Stripe secret key (sk_test_* for local, sk_live_* injected by CI/CD only)
STRIPE_SECRET_KEY=sk_test_your_key_here
# SECURITY: Never commit sk_live_* keys. Production keys are injected via AWS Secrets Manager.
```

### 6.4 Retrieval-Augmented Generation (RAG) for Enterprise Teams

For organisations with extensive internal knowledge bases (runbooks, RFCs, post-mortems), consider connecting AI tools to a RAG pipeline:

- Ingest internal Confluence, Notion, or GitHub Wiki pages into a vector store
- Embed runbooks alongside the code that executes the relevant operations
- Index past incident reports to inform AI suggestions on error-handling paths
- Keep embeddings fresh with automated re-indexing on document change events

> **📝 NOTE:** RAG is a force-multiplier, not a foundation. Complete §3–§5 before investing in RAG — structured documentation has much faster ROI for most teams.

### 6.5 Context Exclusion — Controlling What AI Does NOT Index

Equally important to providing context is **excluding noise**. Large repositories contain generated files, vendored dependencies, and build artefacts that pollute AI indexing and cause hallucinations from non-authoritative code.

Use tool-specific ignore files to keep AI focused on your source of truth:

| File | Tool | Purpose |
|---|---|---|
| `.cursorignore` | Cursor | Prevents indexing of matched paths |
| `.gitignore` | Claude Code, Copilot, Cursor | All three tools respect `.gitignore` as a baseline filter |

> **📝 NOTE:** Claude Code and Copilot do not currently have dedicated ignore files — they rely on `.gitignore` and context window management. Cursor's `.cursorignore` is the most granular option available. Keeping `.gitignore` comprehensive is the single most effective cross-tool exclusion strategy.

**Recommended exclusion patterns:**

```gitignore
# Build outputs & generated code
dist/
build/
*.generated.ts
*.min.js

# Dependencies
node_modules/
vendor/

# Large binary assets
*.wasm
*.pb.go

# IDE & OS files
.idea/
.vscode/settings.json
.DS_Store

# Sensitive files (should never be indexed)
.env
*.pem
credentials.*
```

> **💡 TIP:** Keep `.gitignore` as your primary exclusion mechanism (respected by all three tools), and add a `.cursorignore` for Cursor-specific exclusions that go beyond what `.gitignore` covers (e.g., large source directories you want in Git but not in AI indexing).

### 6.6 Reusable Prompt Templates

For recurring AI-assisted workflows (code review, migration, endpoint scaffolding), maintain **versioned prompt templates** in the repository:

```
prompts/
  review-pr.md          # Structured code review prompt
  migrate-endpoint.md   # Step-by-step endpoint migration guide
  write-adr.md          # ADR drafting prompt with team conventions
  debug-incident.md     # Incident investigation prompt
```

Benefits:
- Team-wide consistency in how AI is directed for common tasks
- Versioned in Git — reviewable and improvable over time
- Onboarding accelerator: new developers inherit proven prompts from day one

> **📝 NOTE:** Prompt templates are a Phase 3/4 practice. Start with `CLAUDE.md` and core docs first — add reusable prompts once your team has identified recurring AI interaction patterns.

### 6.7 Model Context Protocol (MCP)

MCP (Model Context Protocol) is an emerging open standard for providing **dynamic, structured context** to AI tools at runtime. Unlike static documentation files, MCP servers can:

- Serve live data (database schemas, API state, deployment status) as AI context
- Expose project-specific tools the AI can invoke (run tests, query logs, check CI status)
- Integrate with internal knowledge bases without embedding everything in the repo

MCP complements static documentation — use `CLAUDE.md` and repo docs for stable conventions, and MCP for context that changes frequently or lives outside the repository.

> **📝 NOTE:** All three tools — Claude Code, Cursor, and GitHub Copilot — have native MCP support. Copilot supports MCP in VS Code, JetBrains, Xcode, and the Copilot coding agent. GitHub maintains an official MCP server. Evaluate MCP when your team needs AI to reason about live system state or when static docs cannot keep pace with the information the AI needs.

---

## 7. Implementation Roadmap

Use this phased roadmap to avoid paralysis. Documentation can be added iteratively — AI tooling becomes useful at Phase 1 completion and improves with each subsequent phase.

### Phase 1 — Foundation `Days 1–3` · Critical

| Task | Success Criteria |
|---|---|
| Create / update `README.md` | Quick start works end-to-end for a new developer |
| Write AI instruction file for your tool — `CLAUDE.md`, `.cursor/rules/`, or `.github/copilot-instructions.md` (see §3.2–3.4) — include tech stack with exact versions | AI suggests correct library for a test prompt; no unlisted libraries appear in completions |
| Add `.env.example` with comments | No hardcoded values in AI-generated config code |

### Phase 2 — Structure `Week 1–2` · High

| Task | Success Criteria |
|---|---|
| Write `ARCHITECTURE.md` with component diagram | AI respects module boundaries in suggestions |
| Add `CONTRIBUTING.md` with conventions | AI follows commit format and naming conventions |
| Write `SECURITY.md` | AI warns on PII-adjacent code generation |
| Write `DATA_MODELS.md` with entity definitions, relationships, and DB conventions (see §4.7) | AI generates correct column names, respects data access patterns |
| Create `/docs/adr/` with 3+ ADRs for key decisions | AI stops suggesting rejected alternatives |

### Phase 2.5 — Context Hygiene `Week 2` · High

| Task | Success Criteria |
|---|---|
| Review and extend `.gitignore` coverage | All build artefacts, secrets, and binaries are excluded from all three tools |
| Add `.cursorignore` for Cursor-specific exclusions | Cursor never indexes large vendored or generated directories that are tracked in Git |

### Phase 3 — Depth `Month 1` · Medium

1. Add module-level `README` files to top 5 most-edited directories
2. Annotate all exported domain functions with JSDoc / TSDoc
3. Write `TESTING.md` and add test factories for common domain entities
4. Create `MODULE_MAP.md` for large services; expand `DATA_MODELS.md` (created in Phase 2) with per-service data model files if needed
5. Add `CHANGELOG.md` and automate it via Conventional Commits tooling
6. Create initial `prompts/` directory with 2–3 reusable prompt templates for common team workflows

### Phase 4 — Optimisation `Ongoing`

1. Review AI suggestion quality monthly — identify gaps in documentation coverage (see §8.4 for metrics)
2. Add ADRs retroactively for every significant tech or architecture decision
3. Instrument documentation freshness: add *"Last reviewed"* dates and CI staleness checks
4. Evaluate RAG pipeline if team exceeds 10 engineers or 5 services
5. Evaluate MCP integration for dynamic context (live schemas, CI status, deployment state)
6. Contribute conventions to an org-wide documentation standard for cross-team AI consistency

---

## 8. Documentation Maintenance & Governance

Documentation that is not maintained becomes worse than no documentation — it actively misleads both humans and AI. Establish these practices to keep context accurate.

### 8.1 Documentation as Code

- Treat docs like code: require PR reviews for changes to `CLAUDE.md`, `ARCHITECTURE.md`, and ADRs
- Add CI checks that fail if key docs are not updated when their related source files change
- Use `markdownlint` and `vale` (or similar) to enforce structure and language consistency
- Block merges that delete documented modules without updating `MODULE_MAP.md`

### 8.2 Freshness Signals

Add a metadata block to critical files. Use **both** YAML frontmatter (for CI tooling) and an inline comment (for AI visibility — most AI tools read inline comments more reliably than YAML frontmatter):

**YAML frontmatter (parsed by CI and linting tools):**

```yaml
---
last_reviewed: 2025-11-01
reviewed_by: platform-team
next_review: 2026-02-01
status: current  # current | needs-review | outdated
---
```

**Inline comment (visible to AI tools in all contexts):**

```markdown
<!-- Last reviewed: 2025-11-01 | Status: current | Owner: platform-team | Next review: 2026-02-01 -->
```

> **💡 TIP:** Use both formats. The YAML frontmatter drives automated staleness checks in CI. The inline comment ensures AI tools see the freshness signal even when frontmatter is stripped or ignored.

### 8.3 Ownership Model

| Document | Owner | Review Cadence |
|---|---|---|
| `CLAUDE.md` / AI instruction files | Tech Lead | On every architectural change |
| `ARCHITECTURE.md` | Tech Lead | Quarterly |
| ADRs | Author + Tech Lead | Immutable once accepted |
| `CONTRIBUTING.md` | Engineering Manager | Semi-annually |
| `SECURITY.md` | Security Champion | On every dependency update cycle |
| Module READMEs | Module Owner | On significant refactor |
| `README.md` | Any contributor | On every significant feature |

### 8.4 Measuring Documentation Effectiveness

"Review AI suggestion quality monthly" requires concrete metrics. Track these signals to identify documentation gaps:

| Metric | How to Measure | What a Gap Looks Like |
|---|---|---|
| **AI suggestion acceptance rate** | Copilot dashboard, Cursor analytics, or manual sampling | Declining rate → AI context is stale or insufficient |
| **Hallucinated library count** | Track per sprint: AI suggests a library not in your stack | Missing or outdated `CLAUDE.md` stack section |
| **Convention violation rate** | Count AI-generated code that fails linting or review | Missing or unclear `CONTRIBUTING.md` / conventions |
| **Data model hallucination rate** | Track per sprint: AI generates wrong column names, incorrect joins, or bypasses Repository pattern | Missing or incomplete `DATA_MODELS.md` or data access rules |
| **Blind test** | Periodically ask AI to implement a known pattern and check if output matches team conventions | Docs don't describe the pattern clearly enough |
| **Onboarding time** | Measure time for new developer to land first PR with AI assistance | Gaps in `README.md`, `ONBOARDING.md`, or module READMEs |

> **💡 TIP:** Run a monthly "doc gap review" — collect the 5 worst AI suggestions from the past sprint, trace each back to the documentation artefact that should have prevented it, and fix that artefact.

### 8.5 Multi-Language and Polyglot Repository Guidance

Repositories containing multiple languages (e.g., Go backend + TypeScript frontend + Python ML pipeline) require extra care to prevent AI from mixing conventions across boundaries.

**Strategies:**

1. **Subdirectory-level `CLAUDE.md` files** — Place a `CLAUDE.md` in each language root (e.g., `backend/CLAUDE.md`, `frontend/CLAUDE.md`) with language-specific conventions. The root `CLAUDE.md` covers cross-cutting concerns only (repo structure, CI, shared naming). Claude Code automatically loads subdirectory files when working in that subtree.
2. **Explicit boundary markers** — In the root `CLAUDE.md` and `.github/copilot-instructions.md`, state clearly: *"When working in `backend/`, follow Go conventions. When working in `frontend/`, follow TypeScript conventions. Never mix."* This is especially important for Copilot, which uses a single instruction file.
3. **Separate Cursor rules per language** — `.cursor/rules/go-backend.mdc` and `.cursor/rules/ts-frontend.mdc` keep Cursor rules scoped to the relevant language boundary.
4. **Per-language `CONTRIBUTING.md`** — If conventions differ significantly, maintain a `CONTRIBUTING.md` in each language root rather than a single monolithic one.

---

## 9. Dos and Don'ts Quick Reference

### ✅ DO

- Write `CLAUDE.md` / AI instructions **before** enabling any AI tool
- Use exact library versions and explicitly name rejected alternatives
- Record architectural decisions in ADRs with context and alternatives
- Document security constraints in the file closest to the sensitive code
- Review AI instruction files as part of code review for architectural changes
- Respect per-tool size limits: **~200 lines** for `CLAUDE.md`, **~500 lines** per Cursor `.mdc` rule, **~2 pages** for Copilot instructions
- Use structured headings so AI can navigate sections predictably
- Co-locate documentation with code — avoid separate docs repositories
- Automate documentation freshness checks in CI
- Start with Phase 1 minimum — ship AI tooling early, iterate docs
- Add `.cursorignore` and extend `.gitignore` to exclude build artefacts, generated code, and vendored dependencies from AI indexing
- Use your tool's scoping mechanism in monorepos and polyglot repos — subdirectory `CLAUDE.md` files, Cursor `globs:` in `.mdc` rules, or Copilot `applyTo:` in `.github/instructions/`
- Track AI suggestion quality with concrete metrics (acceptance rate, hallucinated library count, convention violations)
- Document your data layer explicitly — entity definitions, relationships, naming conventions, data access patterns, and migration strategy in `DATA_MODELS.md`
- Include condensed data layer rules in your AI instruction file and reference the full `DATA_MODELS.md` for details

### ❌ DO NOT

- Wait until docs are "perfect" before enabling AI tooling
- Write long prose in AI instruction files — use structured lists
- Duplicate content across `README`, `ARCHITECTURE`, and your AI instruction file — link instead
- Omit "do NOT use X" instructions — negative constraints are as important as positive ones
- Store AI instructions only in your IDE settings — commit them to the repo
- Neglect documentation updates when refactoring modules
- Use vague terms like "best practices" — be specific and concrete
- Assume the AI knows your internal library names — always reference them explicitly
- Skip `SECURITY.md` for "internal" services — all services have security context
- Let ADRs become stale — mark superseded decisions as `Superseded` with a link
- Let AI tools index `node_modules/`, `dist/`, or other generated directories — this causes hallucinations from non-authoritative code
- Mix language conventions in polyglot repos — use separate instruction files per language boundary
- Leave database schema undocumented — AI will guess column names, invent relationships, and bypass data access abstractions
- Use float types for currency or assume AI knows your money representation — document it explicitly

---

## 10. Appendix: File Templates & Resources

### A. Minimal CLAUDE.md Template

```markdown
# <Project Name>

## Project Identity
<1–3 sentences: what this service does, who uses it, what domain it operates in>

## Stack
- Language:  <language + version>
- Framework: <framework> (NOT <rejected alternative>)
- Database:  <db + ORM if applicable>
- Testing:   <test framework>

## Repository Structure
src/<module>/   # <responsibility>
...

## Conventions
- <naming convention>
- <error handling pattern>
- <import rules>

## Constraints
- NEVER: <architectural hard constraint>
- NEVER: <security constraint>

## Reusable Utilities
- <path>  — <when to use it>

## Testing
- Run: <command>
- Every <scope> must have a <test type>
```

### B. Minimal Cursor Rule Template (`.cursor/rules/project.mdc`)

```yaml
---
description: "Core project identity, tech stack, and architectural constraints"
alwaysApply: true
---

# <Project Name>

<1–3 sentences: what this service does, domain, language + version>

## Stack
- Language:  <language + version>
- Framework: <framework> (NOT <rejected alternative>)
- Database:  <db + ORM>
- Testing:   <test framework>

## Constraints
- NEVER: <architectural hard constraint>
- NEVER: <security constraint>
```

### C. Minimal Copilot Instructions Template (`.github/copilot-instructions.md`)

```markdown
# <Project Name>

<1–2 sentences: what this service does, language, framework>

## Conventions
- Use <framework>, NOT <rejected alternative>
- Use <test framework>, NOT <rejected alternative>
- <naming convention>
- <error handling pattern>

## Architecture
- <key architectural constraint>
- <module boundary rule>

## Security
- <critical security constraint>

## Testing
- <test framework> for all tests
- Run: <command>
```

### D. Copilot Path-Specific Instructions Template (`.github/instructions/NAME.instructions.md`)

```yaml
---
applyTo: "**/*.ts,**/*.tsx"
---

<Language-specific conventions here>
```

### E. DATA_MODELS.md Template

```markdown
# Data Models

<!-- Last reviewed: YYYY-MM-DD | Status: current | Owner: <team> | Next review: YYYY-MM-DD -->

## Database Conventions

- Table names: snake_case, plural (e.g., `users`, `order_items`)
- Column names: snake_case (e.g., `created_at`, `user_id`)
- Foreign keys: `<referenced_table_singular>_id`
- Indexes: `idx_<table>_<column>`
- Money: stored as INTEGER in cents — never use float
- Soft deletes: `deleted_at` TIMESTAMPTZ, nullable — never hard-delete user data
- Timestamps: always TIMESTAMPTZ, never TIMESTAMP

## Core Entities

### <EntityName>
| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | UUID | PK, generated | |
| ... | ... | ... | |
| created_at | TIMESTAMPTZ | not null, default now() | |
| updated_at | TIMESTAMPTZ | not null | Auto-updated by ORM |

### Relationships
- <EntityA> → <EntityB>: one-to-many (<fk> FK)

## Data Access Rules

- All DB access through Repository interfaces — never raw queries in handlers or domain
- Pagination: cursor-based only — never OFFSET/LIMIT for user-facing queries
- Read replicas: read-only queries use the read connection pool
- N+1 prevention: use eager loading / joins for known relationship traversals

## Migrations

- Tool: <migration tool>
- Files: `<path>/NNNN_description.sql`
- Every migration must be reversible
- Never modify an applied migration — create a new one
- Run: `<migrate command>` (apply), `<rollback command>` (revert)
```

### F. ADR Template (MADR)

```markdown
# ADR-NNNN: <Short Title>

**Status:** Proposed | Accepted | Superseded by ADR-XXXX
**Date:** YYYY-MM-DD
**Deciders:** <names or teams>

## Context
<What is the situation and why does a decision need to be made?>

## Decision
<What was decided? Be specific.>

## Alternatives Considered
- <Alternative 1>: rejected because <reason>
- <Alternative 2>: rejected because <reason>

## Consequences
<What are the positive and negative consequences of this decision?>
```

### G. Recommended Tooling

| Tool | Purpose | Notes |
|---|---|---|
| `markdownlint` | Enforce MD formatting | Integrate into CI + editor |
| `vale` | Prose style linting | Catches stale terminology |
| `log4brains` | ADR management & web UI | GitHub Action available |
| `doctoc` | Auto-generate MD TOCs | Run as pre-commit hook |
| `semantic-release` | Automate `CHANGELOG.md` | Requires Conventional Commits |
| `swagger-ui` / `redoc` | Render OpenAPI specs | Co-locate with service |
| `danger.js` | PR doc-update enforcement | Fail PR if docs not updated |
| MCP servers | Dynamic AI context | Serve live schemas, CI status, logs to AI tools |

### H. Context Exclusion Template

Use this as a starting point for `.gitignore` additions and `.cursorignore`:

```gitignore
# === AI Context Exclusion ===
# Generated / build output
dist/
build/
out/
coverage/
*.generated.*
*.min.js
*.min.css

# Dependencies
node_modules/
vendor/
.pnp.*

# Binary & large assets
*.wasm
*.pb.go
*.pb.ts
*.lock

# IDE & OS
.idea/
.vscode/settings.json
.DS_Store
Thumbs.db

# Sensitive
.env
.env.*
*.pem
*.key
credentials.*
secrets.*
```

---

> *Documentation is the interface between human intent and AI capability.*
> *Invest in it deliberately — and your AI tools will compound the return.*
