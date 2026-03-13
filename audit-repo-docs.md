Analyze this repository against the AI Documentation Strategy Guide to assess documentation
readiness for AI-assisted development. Perform a full audit, produce a prioritized TODO list,
and provide ready-to-use prompts for generating each missing artifact.

## Step 1: Audit Existing Documentation

Check for the presence and quality of each artefact below. For each item found, evaluate whether
its content is structured for AI consumption (scannable headings, bullet lists, explicit
constraints — not prose).

**Critical:**
- [ ] README.md — has quick start (clone → install → run in ≤5 commands), env var table, links to other docs
- [ ] ARCHITECTURE.md — has component diagram, module boundaries, data flow for key journeys, anti-patterns
- [ ] AI instruction file — CLAUDE.md, .cursor/rules/*.mdc, or .github/copilot-instructions.md (check which tool this repo uses)

**High:**
- [ ] CONTRIBUTING.md — branch naming, commit format, PR requirements, Definition of Done
- [ ] docs/adr/ — at least 3 ADRs for key decisions with alternatives considered
- [ ] SECURITY.md — PII rules, auth patterns, banned functions, dependency policy
- [ ] DATA_MODELS.md — entity definitions, relationships, DB naming conventions, data access patterns, migration strategy

**Medium:**
- [ ] API reference or OpenAPI/AsyncAPI spec — co-located and CI-validated
- [ ] Module-level README files — in top 5 most-edited directories
- [ ] TESTING.md — test pyramid, exact run commands, coverage thresholds, mock patterns

**Low:**
- [ ] CHANGELOG.md — structured, ideally automated via Conventional Commits
- [ ] GLOSSARY.md — domain-specific terminology
- [ ] .env.example — every env var commented with required/optional and example value

## Step 2: Evaluate AI Instruction File Quality (if present)

If an AI instruction file exists, check whether it covers these sections:
1. Project identity (what, who, domain, language, what NOT)
2. Technology stack with exact versions and explicit "NOT X" exclusions
3. Repository structure (commented directory tree)
4. Coding conventions (naming, patterns, anti-patterns)
5. Architectural constraints (hard rules with NEVER)
6. Common patterns / reusable utilities (file paths + when to use)
7. Data layer (DB conventions, ORM, data access rules)
8. Testing requirements (exact run commands, coverage)
9. Do / Do Not summary

Also check:
- Is it under 200 lines (CLAUDE.md) / 500 lines (.mdc) / 2 pages (Copilot)?
- Does it use structured lists, not prose?
- Are negative constraints included ("NOT Express", "NEVER use any")?

## Step 3: Check Context Hygiene

- [ ] .gitignore excludes: dist/, build/, node_modules/, vendor/, .env, *.pem, *.generated.*
- [ ] .cursorignore exists (if using Cursor) for large tracked directories AI shouldn't index
- [ ] No secrets or credentials committed

## Step 4: Produce the TODO List

Output a markdown TODO list grouped by phase, with this format:

### Phase 1 — Foundation (Days 1–3) · Critical
- [ ] **<action>** — <why this matters for AI> · Priority: Critical

### Phase 2 — Structure (Week 1–2) · High
- [ ] **<action>** — <why this matters for AI> · Priority: High

### Phase 3 — Depth (Month 1) · Medium
- [ ] **<action>** — <why this matters for AI> · Priority: Medium

### Phase 4 — Optimization (Ongoing) · Low
- [ ] **<action>** — <why this matters for AI> · Priority: Low

Rules for the TODO list:
- Only include items that are MISSING or INSUFFICIENT — do not list items that already pass
- Order items within each phase by impact (highest first)
- For each item, include a concrete one-sentence description of what "done" looks like
- If the repo already scores 8+ on the audit, say so and focus the TODO on refinements only
- At the top, include a one-line readiness score: 🔴 Not Ready / 🟡 Partial / 🟢 Good / ✅ Excellent

## Step 5: Generate Scaffold

For each **Critical** missing or insufficient artifact identified in Steps 1–3, output a ready-to-commit scaffold populated with values inferred from the repository.

se this exact format for each file:
```markdown
### 📄 ``


```

At minimum, generate scaffolds for:
- `CLAUDE.md` - Detect which AI tool the repo uses (or default to CLAUDE.md)
- `ARCHITECTURE.md` (with a component diagram based on inferred structure)
- Any missing Critical-priority file from the checklist

---