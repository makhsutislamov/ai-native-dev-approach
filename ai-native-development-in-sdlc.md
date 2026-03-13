# AI-Native Development in SDLC — General Approach

## Core Principle

Treat AI as a development team member, not a tool. This means it needs the same things any new developer needs: context, conventions, clear expectations, and review.

---

## The 3 Pillars

1. **AI-Readable Codebase** — AI can only be as good as the context it has
2. **Human-in-the-Loop Workflow** — AI proposes, human decides
3. **Continuous Feedback** — Refine what AI knows based on what it gets wrong

---

## Phase 1: Make Your Project AI-Ready

Before using AI in any meaningful way, your project needs structured context that AI can consume.

### What to create

- **AI instruction file** at project root (e.g., `CLAUDE.md`, `.cursorrules`, `copilot-instructions.md`) containing:
  - Tech stack and architecture overview
  - Build / test / run commands
  - Critical rules ("never do X", "always do Y")
  - Link to detailed pattern guides
- **Pattern guides** — structured documents covering:
  - Architecture and directory conventions
  - API/endpoint patterns
  - Testing strategy and examples
  - Code quality standards
  - Git/branching workflow

### Writing principles for AI-consumed docs

- **Pattern-first:** show code before explanation
- **Example-driven:** 3+ real examples per pattern
- **Dense:** tables and lists over paragraphs
- **Structured:** consistent headings, code blocks with language tags
- **Reference source files** with line numbers

---

## Phase 2: Introduce AI by SDLC Stage

Roll out in order of risk — lowest risk first:

| Order | SDLC Stage     | AI Activity                          | Risk        |
|-------|----------------|--------------------------------------|-------------|
| 1     | Testing        | Generate unit/integration tests      | Low         |
| 2     | Code Review    | AI reviews PRs against standards     | Low         |
| 3     | Refactoring    | Identify and execute cleanups        | Low-Medium  |
| 4     | Implementation | Generate features with plan approval | Medium      |
| 5     | Architecture   | Propose solution designs             | Medium-High |
| 6     | CI/CD & Ops    | Generate pipelines, IaC, scripts     | Medium      |

---

## Phase 3: The Plan-First Workflow

For any non-trivial task, follow this loop:

```
Requirement → AI reads context → AI proposes plan → Human reviews plan
     ↓                                                      ↓
   (reject / adjust)                                  (approve)
                                                         ↓
                                                  AI implements
                                                         ↓
                                                  Human reviews code
                                                         ↓
                                                  Standard CI/CD pipeline
```

**Never skip the plan step.** This is the single biggest differentiator between teams that succeed with AI and those that don't.

---

## Phase 4: Continuous Refinement

- When AI gets something wrong, update the docs so it won't repeat the mistake
- Track what types of tasks AI handles well vs. poorly
- Build a library of reusable prompt patterns / agent definitions for recurring tasks
- Keep documentation in sync with evolving codebase

---

## Best Practices

### Documentation

- Keep AI docs next to the code they describe
- Update docs when patterns change — stale docs produce stale output
- Use real code examples, never hypothetical ones
- One source of truth — don't duplicate conventions in multiple places

### Workflow

- AI-generated code must pass the same CI pipeline as human code
- Never disable linting, tests, or hooks for AI output
- Start small — one team, one project, one SDLC stage
- Expand based on measured results, not assumptions

### Team Adoption

- One champion documents patterns and leads the first rollout
- Share concrete wins early (time saved, bugs caught)
- Make AI optional initially — forced adoption creates resistance
- Create shared prompt/agent libraries so the team doesn't reinvent the wheel

### Quality Gates

- AI proposes → human approves → AI implements → human reviews
- AI should self-check against documented patterns before presenting output
- Treat AI suggestions the same as junior developer PRs — review everything

---

## Common Pitfalls

| Pitfall                                 | Consequence                                     |
|-----------------------------------------|-------------------------------------------------|
| No structured documentation             | AI generates generic, convention-breaking code   |
| Trusting AI output without review       | Subtle bugs, security issues, wrong patterns     |
| Starting with high-risk tasks           | Broken features, lost confidence in AI approach  |
| Outdated docs                           | AI follows stale patterns                        |
| Skipping the plan step                  | Wasted effort when AI goes the wrong direction   |
| Trying to automate everything at once   | Overwhelm, poor results, team pushback           |

---

## Recommended Rollout Timeline

| Week | Action                                              |
|------|-----------------------------------------------------|
| 1    | Create AI instruction file + core pattern guides    |
| 2-3  | AI-assisted test generation + code review           |
| 3-4  | AI-assisted implementation (plan-first)             |
| 5+   | Expand to more stages, refine docs, measure results |

---

## The Formula

> **Structured context + Plan-first workflow + Human review + Iterative refinement**

Start with documentation. Introduce AI at the lowest-risk stage. Expand as confidence builds. Keep docs current. Always keep humans in the decision loop.
