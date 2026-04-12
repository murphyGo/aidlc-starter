# Adopt Skill

Onboard an existing codebase into the AI-DLC workflow by analyzing the current system and generating a brownfield-formatted IDEA.md.

## Arguments

- `$ARGUMENTS` - One of the following:
  - (empty) - Full guided flow: scan codebase, dialogue, generate IDEA.md
  - `scan` - Scan only: analyze codebase and show findings, no IDEA.md
  - `"description of what to add"` - Start with a known goal, skip the "What We Are Adding" dialogue

## Objective

Help users apply AI-DLC to a project that already has source code. Scan the existing codebase to build context, then guide the user through a brownfield-specific dialogue to produce an IDEA.md with Current State, What We Are Adding, and What Must Not Change sections.

---

## Execution Steps

### Step 1: Validate Environment

1. **Check for existing source code**:
   - Scan for source files (.java, .py, .js, .ts, .go, .rs, .rb, .php, .c, .cpp, .cs, etc.)
   - Scan for build files (package.json, pom.xml, build.gradle, Cargo.toml, go.mod, Makefile, etc.)
   - If NO source code found: Suggest `/ideate` instead — this project looks greenfield

2. **Check for existing IDEA.md**:
   - If IDEA.md exists: Ask user whether to overwrite or refine
   - If IDEA.md exists with brownfield format (has "Current State" section): Offer to update

3. **Verify AI-DLC rules available**:
   - Check `aidlc-workflows/aidlc-rules/` exists
   - If missing, provide setup instructions (copy from aidlc-starter)

### Step 2: Codebase Scan

Analyze the workspace and collect:

```
Scan Targets:
├── Languages & Frameworks (source file extensions, imports, configs)
├── Build System (package manager, build tool, scripts)
├── Project Structure (monolith / microservices / monorepo / library)
├── Services & Components (directories, entry points, API definitions)
├── Data Stores (DB configs, migration files, ORM models)
├── Infrastructure (Docker, CDK, Terraform, CI/CD configs)
├── Test Setup (test framework, test directories, coverage config)
└── Key Dependencies (from lock files or dependency manifests)
```

**Keep scanning lightweight** — read configs and directory structure, don't parse every source file.

### Step 3: Present Scan Results

```
## Codebase Analysis

| Aspect | Finding |
|--------|---------|
| Languages | [e.g., TypeScript 5.x, Python 3.11] |
| Frameworks | [e.g., Express 4.x, React 18] |
| Build System | [e.g., npm, Maven] |
| Structure | [e.g., Monorepo with 3 services] |
| Data Stores | [e.g., PostgreSQL via node-postgres] |
| Infrastructure | [e.g., AWS ECS Fargate, CDK] |
| Tests | [e.g., Jest 29.x, __tests__/ pattern] |

### Services / Components Found
- [service-a]: [brief description from README or entry point]
- [service-b]: [brief description]
- ...

### Does this look right?

- **yes** - Proceed to describe what you want to add
- **adjust** - Correct any findings
- **details [component]** - Show more detail about a specific component
```

**Wait for user response.**

If `$ARGUMENTS` is `scan`, stop here after presenting results.

### Step 4: What We Are Adding (The Goal)

If `$ARGUMENTS` contains a description, use it as starting point. Otherwise ask:

```
## What Would You Like to Add?

Now that I understand the current system, tell me:
- What new feature, module, or capability do you want to build?
- Or what existing part needs significant rework?

Describe it however feels natural — a sentence or a paragraph.
```

**Wait for user response.**

Follow up with 1-2 clarifying questions based on what's missing:

| If Missing | Ask |
|------------|-----|
| Scope clarity | "Which of these are in scope for this iteration vs. future phases?" |
| Integration points | "Which existing services/components will this interact with?" |
| User-facing vs internal | "Is this customer-facing, internal tooling, or infrastructure?" |

**Maximum 2 rounds of questions.** Accept partial answers — `/init-project` will refine further.

### Step 5: What Must Not Change (Constraints)

```
## Constraints

Based on the codebase scan, I'd suggest these as "must not change":

| Category | Suggested Constraint |
|----------|---------------------|
| [e.g., API contracts] | [e.g., order-service REST API — do not modify existing endpoints] |
| [e.g., Database schema] | [e.g., Existing tables — additive migrations only] |
| [e.g., Infrastructure] | [e.g., Existing CDK stacks — add new, don't modify] |
| [e.g., Patterns] | [e.g., Raw SQL with typed helpers — no ORM introduction] |

### Are these right?

- **yes** - Use these constraints
- **add [constraint]** - Add another constraint
- **remove [number]** - Remove a suggested constraint
- **skip** - No constraints to specify (not recommended)
```

**Wait for user response.**

### Step 6: Present Draft IDEA.md

Synthesize scan results + dialogue into brownfield format:

```markdown
## Here's Your Brownfield IDEA.md

---

# [Project Name - from repo name or user input]

> **Brownfield project.** This document describes a change to an existing system.
> The Current State section gives AI-DLC the context it needs to understand
> what already exists before generating requirements and design.

---

## Current State

[System description synthesized from Step 2 scan: what the system does,
tech stack, services/components, data stores, deployment]

---

## What We Are Adding

[From Step 4 dialogue]

---

## Features In Scope (this iteration)

- [Feature 1]
- [Feature 2]
- ...

## Features Explicitly Out of Scope (this iteration)

- [Anything user mentioned as future/later]

---

## What Must Not Change

- [Constraint 1 from Step 5]
- [Constraint 2]
- ...

---

## Open Questions

- [Any unresolved points from dialogue]

---

### How does this look?

- **save** - Create IDEA.md with this content
- **adjust [section]** - Modify a specific section
- **add [detail]** - Add something I missed
- **restart** - Start over
```

### Step 7: Handle Feedback Loop

| User Response | Action |
|---------------|--------|
| "save" / "looks good" / "yes" | Proceed to Step 8 |
| "adjust [section]" | Ask what to change, update draft, re-present |
| "add [detail]" | Incorporate, re-present draft |
| "restart" | Go back to Step 4 (keep scan results) |

### Step 8: Save and Guide Next Steps

1. **Write IDEA.md** to project root

2. **Present completion message**:

```
## IDEA.md Created (Brownfield)

Your existing system context and planned changes have been saved to IDEA.md.

### What's Next

Run `/init-project` to:
1. Refine requirements through interactive dialogue
2. AI-DLC will auto-detect your existing codebase (brownfield mode)
3. Reverse engineering stage will analyze your code in detail
4. Generate specifications for the new work

### What to Expect

The `/init-project` process includes a **reverse engineering stage** for brownfield
projects. This stage:
- Analyzes your codebase structure, architecture, and business logic
- Generates detailed documentation of the existing system
- Requires your review and approval before proceeding
- Typically takes longer than greenfield initialization

Shall I run `/init-project` now? (yes/no)
```

If yes, invoke `/init-project`.

---

## Dialogue Guidelines

### Principles

1. **Leverage the scan** — Don't ask the user what they already know from the code
2. **Suggest, don't interrogate** — Present findings and let user correct
3. **Keep it short** — 3-4 exchanges maximum for the full flow
4. **Brownfield-aware defaults** — Default constraints favor preserving existing patterns

### Tone

- Confident about scan findings, open to corrections
- Pragmatic — focus on what's changing, not documenting everything
- Concise — the user already knows their codebase

---

## Error Handling

| Situation | Response |
|-----------|----------|
| No source code found | Suggest `/ideate` for greenfield projects |
| Very large monorepo | Focus scan on top-level structure, ask user which area to focus on |
| IDEA.md already exists (greenfield format) | Offer to convert to brownfield format preserving existing content |
| IDEA.md already exists (brownfield format) | Offer to update specific sections |
| User wants to start fresh instead | Redirect to `/ideate` |
| Scan finds ambiguous structure | Present options and ask user to clarify |

---

## Example Invocations

Full guided flow:
```
/adopt
```

Scan only (no IDEA.md generation):
```
/adopt scan
```

Start with a known goal:
```
/adopt Add a returns and refunds module to the existing e-commerce platform
```
