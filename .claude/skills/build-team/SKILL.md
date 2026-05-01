# Build-Team Skill

Meta-skill that analyzes the current project, proposes an agent team (one team-lead plus role-appropriate specialists with parallelism), and on user approval generates Claude Code subagents under `.claude/agents/` and a `/team-lead` orchestrator skill.

## Arguments

- `$ARGUMENTS` — One of:
  - (empty) — Full guided flow: analyze → propose → approve → generate
  - `propose` — Analyze and propose only, do not generate files
  - `regenerate` — Force regeneration even if `.claude/agents/` already populated
  - `roster` — Show the current team (if a team already exists)
  - `add <role>` — Add a single role to an existing team (e.g., `add security-engineer`)

## Objective

Produce a project-tailored agent team that can collaborate under one team-lead. The team-lead can either find work autonomously (scan project state, propose a plan, delegate) or execute a user-given task by decomposing it across specialists. All roles are AIDLC-aligned: each agent's prompt names which AIDLC artifacts it produces or consumes.

The skill is opinionated about *team-lead is mandatory and singleton* but lets the project shape the rest of the roster. It must justify each suggested role with concrete signals from the project (units, tech stack, NFR concerns, brownfield constraints) — not boilerplate.

---

## Execution Steps

### Step 1: Validate Prerequisites

1. **Detect existing team**:
   - List `.claude/agents/*.md` and `.claude/skills/team-lead/SKILL.md`
   - If team exists and `$ARGUMENTS` is not `regenerate` / `roster` / `add ...`:
     ```
     ## Team Already Exists

     Found agents in .claude/agents/:
     [list with one-line description from each agent's frontmatter]

     Options:
     - **roster** — Show full team detail
     - **add <role>** — Add a new role
     - **regenerate** — Replace the existing team (will overwrite)
     - **cancel** — Keep current team, exit
     ```
     Wait for response.

2. **Verify project context available** (degrade gracefully):

   | Signal | Source | If missing |
   |--------|--------|------------|
   | Idea | `IDEA.md` or `docs/PROJECT-VISION.md` | Warn — proposal will be generic |
   | Requirements | `docs/requirements.md` or `aidlc-docs/inception/requirements/` | Continue — fewer signals |
   | Units of work | `aidlc-docs/aidlc-state.md`, `aidlc-docs/inception/plans/execution-plan.md` | Continue — default to 1 developer |
   | Tech stack | `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc. | Continue — ask user inline |
   | Codebase | source dirs (`src/`, `app/`, `cmd/`, `lib/`) | Continue — flag as pre-code |
   | Extensions | `aidlc-docs/aidlc-state.md` Extension Configuration | Continue — security/compliance roles only suggested if extensions opted in |

   If all signals are missing, suggest `/start` first and exit.

### Step 2: Project Analysis (Extract Signals)

Read each available source and extract structured signals. **Do not narrate the read** — just collect.

```
Signals to extract:
├── Project type (web app / API / CLI / library / ML / data pipeline / infra / mobile / multi)
├── Greenfield vs Brownfield (presence of substantial pre-existing code)
├── Tech stack (languages, frameworks, runtimes)
├── Number of units of work (from aidlc-state.md units list)
├── Unit dependencies (sequential vs parallelizable)
├── NFR profile (security, performance, compliance, observability — from NFR Requirements stage if present)
├── Infrastructure footprint (Docker, IaC files, deploy targets)
├── External integrations (APIs, queues, data stores)
├── User-facing surface (UI, CLI, public API → doc burden)
├── Test infrastructure (test framework present, coverage tooling)
└── Active extensions (security-baseline, compliance, etc.)
```

These signals drive the proposal — never invent a role without a signal to back it.

### Step 3: Propose Team Composition

Present the proposal in this exact shape:

```
## Proposed Agent Team

Based on analysis of [N signals from M sources], here is a team tailored to this project.

### Project Snapshot
| Aspect | Finding |
|--------|---------|
| Project type | [type] |
| Mode | [greenfield / brownfield] |
| Tech stack | [primary stack] |
| Units of work | [N units, P parallelizable] |
| NFR focus | [top 1–3 concerns, or "standard"] |
| Active extensions | [list, or "none"] |

### Team Roster

| # | Role | Count | Why this project needs it | Triggers (when team-lead delegates) |
|---|------|-------|---------------------------|-------------------------------------|
| 1 | **team-lead** | 1 | Mandatory orchestrator | Always entry point |
| 2 | planner | 1 | [signal-based reason] | Inception artifacts: intent / requirements / user stories |
| 3 | architect | 1 | [signal-based reason] | Construction design stages: functional / NFR / infra |
| 4 | developer | [N] | [N units identified — P parallelizable] | Code Generation per unit |
| 5 | qa | [N] | [reason] | Build & Test, acceptance validation |
| 6 | reviewer | 1 | [reason] | Post-implementation review per Step 5.1 of dev-skill |
| 7 | operator | [0/1] | [reason or "skipped — no IaC detected"] | CI/CD, deploy, infra changes |
| ... | [project-specific] | ... | ... | ... |

### Project-Specific Suggestions

[Roles proposed beyond the core set, each with explicit signal-based justification.
Examples:
- **frontend-specialist** — React + complex UI surface detected
- **ml-researcher** — training pipeline + model artifacts in IDEA.md
- **security-engineer** — security-baseline extension is active
- **data-engineer** — Kafka + S3 + Glue references in inception
Mark as **suggested** if 50–80% confident, **strongly recommended** if signal is unambiguous.]

### Parallelism Plan

| Role | Why N | Coordination |
|------|-------|--------------|
| developer × N | [units that can run in parallel from execution-plan deps] | team-lead assigns one unit per developer |
| qa × N | [test independence] | one per developer, or 1 if shared QA suffices |
| ... | ... | ... |

### Approval

- **approve** — Generate this team as-is
- **adjust** — Tell me what to change (counts, roles, additions, removals)
- **details <role>** — Show the full prompt that would be generated for a role
- **cancel** — Don't generate
```

**Wait for user response.**

### Step 4: Refinement Dialogue

| Response | Action |
|----------|--------|
| `approve` / `yes` / `generate` | Proceed to Step 5 |
| `adjust ...` | Apply change, re-present roster |
| `details <role>` | Show generated agent prompt preview, return to approval |
| `cancel` / `no` | Exit, no files written |

Accept partial approvals: e.g., "approve but drop operator" → drop, re-confirm in one line.

### Step 5: Generate Agent Files

For each approved role, write `.claude/agents/<role>.md` using the **Subagent Template** below. Substitute placeholders with project-specific content drawn from Step 2 signals.

Mandatory frontmatter fields:
- `name`: kebab-case, must match filename stem
- `description`: one sentence describing **when** the team-lead should delegate to this agent (the Agent tool uses this for routing)
- `tools`: comma-separated tool list scoped to the role (see catalog below)
- `model`: `sonnet` for routine work, `opus` for team-lead and architect

**Tool scoping by role:**

| Role | Tools |
|------|-------|
| team-lead | All tools (it spawns others via Agent) |
| planner | Read, Write, Edit, Glob, Grep, WebFetch, WebSearch |
| architect | Read, Write, Edit, Glob, Grep, WebFetch |
| developer | Read, Write, Edit, Glob, Grep, Bash, NotebookEdit |
| qa | Read, Write, Edit, Glob, Grep, Bash |
| reviewer | Read, Glob, Grep, Bash (read-only Bash for tests/linters) |
| operator | Read, Write, Edit, Glob, Grep, Bash |
| doc-writer | Read, Write, Edit, Glob, Grep |
| researcher | Read, Write, Edit, Glob, Grep, WebFetch, WebSearch |
| security-engineer | Read, Edit, Glob, Grep, Bash |

For multi-instance roles (e.g., `developer × 3`), generate **one** agent file. The team-lead spawns N parallel `Agent(subagent_type="developer", ...)` calls, each given a different unit. Don't create `developer-1.md`, `developer-2.md` — that splits the prompt source unnecessarily.

### Step 6: Generate Team-Lead Skill

Write `.claude/skills/team-lead/SKILL.md` using the **Team-Lead Skill Template** below. The team-lead is invoked by the user (`/team-lead`), reads project state, and decides between autonomous vs directed mode.

Also write `.claude/agents/team-lead.md` — the team-lead also exists as a subagent so other tooling could invoke it via the Agent tool, but the **primary entry point is the skill**.

### Step 7: Summary and Next Steps

```
## Agent Team Created

### Generated
| File | Purpose |
|------|---------|
| .claude/agents/team-lead.md | Orchestrator |
| .claude/agents/<role>.md | [N specialist files] |
| .claude/skills/team-lead/SKILL.md | User entry point — `/team-lead` |
| docs/AGENT-TEAM.md | Roster + delegation map (reference) |

### How to Use

| Mode | Command | What happens |
|------|---------|--------------|
| Autonomous | `/team-lead` | Lead scans state, proposes plan, asks for approval, delegates |
| Directed | `/team-lead <task>` | Lead decomposes the task, delegates to specialists in parallel |
| Status | `/team-lead status` | Recent activity, in-flight delegations |
| Roster | `/team-lead roster` | Show team |

### Modifying the Team

- Add a role: `/build-team add <role>`
- Replace team: `/build-team regenerate`
- Edit an agent: edit `.claude/agents/<role>.md` directly — frontmatter `description` controls delegation routing

### Next Step

Try it: `/team-lead` (autonomous) or `/team-lead "fix the failing CI on main"` (directed).
```

Also write `docs/AGENT-TEAM.md` with the full roster + delegation map (table from Step 3) so the team-lead can re-read it.

---

## Agent Role Catalog

Reference for which roles to consider and their default responsibilities. **Always justify with project signals.**

### Always considered

| Role | Default responsibilities | AIDLC alignment |
|------|--------------------------|-----------------|
| team-lead | Orchestration, decomposition, delegation, synthesis, approval gates | Cross-cuts all stages |
| planner | Requirements clarification, user stories, acceptance criteria | Inception |
| architect | Functional design, NFR design, infrastructure design, ADRs | Construction (design stages) |
| developer | Code generation, unit tests, refactors per unit-of-work | Construction (code-generation) |
| qa | Test plans, integration/e2e tests, cross-check execution | Construction (build-and-test), cross-check |
| reviewer | Fresh-eyes code review using protocols in `.claude/skills/code-review/protocols/` | Cross-cuts construction |

### Conditionally suggested (require a signal)

| Role | Signal | Skip if |
|------|--------|---------|
| operator | IaC files / Dockerfile / .github/workflows / deploy/ present | No infra footprint |
| doc-writer | Public API, CLI tool, large user-facing surface, or docs/ has many files | Internal-only library |
| researcher | ML/research project, novel algorithm, exploration phase explicit in IDEA | Standard CRUD |
| frontend-specialist | React/Vue/Svelte/etc. + non-trivial UI surface | API-only project |
| backend-specialist | Multiple services + complex backend, or backend is dominant | Frontend-only |
| data-engineer | Pipeline tooling (Kafka, Spark, Airflow, dbt) or large data movement | No data plane |
| ml-engineer | Model training/serving pipeline distinct from research | No ML inference |
| security-engineer | `security-baseline` extension active, or security-sensitive domain | No security extension |
| compliance-officer | Compliance extension active, regulated domain | None |
| dba | Heavy schema work, migrations, performance-tuned queries | Light persistence |
| sre | Production SLOs declared, observability NFRs, on-call docs | Pre-production |

If a project signal points to a role not in this catalog, propose it with a custom name and clear justification — the catalog is a starting point, not a cap.

---

## Subagent Template

Generate each `.claude/agents/<role>.md` with this structure. Replace `{{...}}` placeholders.

```markdown
---
name: {{role-name}}
description: {{One sentence describing WHEN team-lead should delegate to this agent. The Agent tool uses this for routing — be specific about triggers.}}
tools: {{comma-separated tool list per role-tool table}}
model: {{sonnet | opus}}
---

You are the **{{role-name}}** for the {{project-name}} project.

## Project Context

{{One paragraph: what the project is, derived from IDEA.md / PROJECT-VISION.md.}}

**Tech stack:** {{from package manifest}}
**Mode:** {{greenfield | brownfield}}
**AIDLC artifacts location:** `aidlc-docs/`

## Your Responsibilities

{{Role-specific bullets, e.g., for developer:
- Implement code-generation steps from `aidlc-docs/construction/plans/{unit-name}-code-generation-plan.md`
- One unit per invocation (the team-lead assigns the unit via prompt)
- Application code goes to workspace root, never under `aidlc-docs/`
- Write unit tests alongside implementation
- Reference requirement IDs (FR-/NFR-) in code comments
}}

## AIDLC Artifacts You Produce

{{List with paths, e.g.:
- `aidlc-docs/construction/{unit-name}/code/implementation-summary.md` — what was implemented and why
- Source files in workspace per project structure
- Updated checkbox state in the per-stage plan file
}}

## AIDLC Artifacts You Consume

{{List with paths, e.g.:
- `docs/requirements.md` — functional requirements
- `aidlc-docs/construction/{unit-name}/functional-design/` — design decisions
- `aidlc-docs/construction/plans/{unit-name}-code-generation-plan.md` — current step list
- `CLAUDE.md` — project conventions
}}

## How You Are Invoked

The team-lead delegates to you with a prompt that includes:
- The unit name and stage step you should execute
- Pointers to relevant artifacts
- Acceptance criteria for the delegation

You must:
1. Read the named artifacts before acting
2. Execute exactly the requested scope — do not expand work
3. Report back with: files changed, tests added/run, decisions made, blockers
4. If blocked, surface the blocker — do not improvise around missing context

## Project-Specific Rules

{{From CLAUDE.md "Important Conventions" + any extension rules active in aidlc-state.md.
e.g.:
- Skills don't auto-commit — show changes, wait for approval
- Use Markdown tables for status, not free text
- Reference requirement IDs in code comments
}}

## Output Format

When you finish a delegation, return a structured report:

```
### {{role-name}} Report

**Task**: <what was delegated>
**Status**: complete | blocked | partial
**Files**: <created / modified>
**Tests**: <added / run / passing>
**Decisions**: <key choices and rationale>
**Blockers**: <if any>
**Handoff**: <what should happen next>
```
```

---

## Team-Lead Skill Template

Generate `.claude/skills/team-lead/SKILL.md` with this structure:

```markdown
# Team-Lead Skill

Orchestrator for the {{project-name}} agent team. Operates in autonomous (find work) or directed (execute task) mode and delegates to specialist subagents in parallel.

## Arguments

- `$ARGUMENTS`:
  - (empty) — **Autonomous mode**: scan project, propose work, await approval, delegate
  - `<task description>` — **Directed mode**: decompose the task and delegate
  - `status` — Show recent team activity from session logs
  - `roster` — Show the team (read `docs/AGENT-TEAM.md`)
  - `replan` — Re-derive the autonomous plan even if one exists in this session

## Objective

Act as the team-lead. Decide what to do, delegate to specialists, synthesize results, and present approval gates to the user. Never silently expand scope. Always treat the user as the final authority.

---

## Execution Steps

### Step 1: Mode Detection

| `$ARGUMENTS` | Mode | Next |
|--------------|------|------|
| empty | Autonomous | Step 2 |
| `status` | Status | Read recent `docs/sessions/*.md`, summarize, exit |
| `roster` | Roster | Read `docs/AGENT-TEAM.md`, present, exit |
| `replan` | Autonomous (force) | Step 2 |
| anything else | Directed | Step 3 |

### Step 2: Autonomous Mode — Find Work

Read in order, stop early if a high-priority item is found:

1. `aidlc-docs/aidlc-state.md` — incomplete unit/stage = highest priority
2. `docs/TECH-DEBT.md` — Critical-aged or High-aged-over-threshold items
3. Open `[ ]` checkboxes in `aidlc-docs/construction/plans/*.md`
4. Failing tests (run the test command if cheap, otherwise check CI logs)
5. Stale TODO/FIXME comments in source
6. Recent session logs — pick up an "later" deferred from previous sessions

Synthesize a candidate work item. Present:

```
## Autonomous Work Proposal

**Candidate**: <one-line summary>
**Source signal**: <where this came from>
**Why now**: <urgency / unblock value>
**Proposed delegation**:

| Role | Sub-task | Acceptance |
|------|----------|------------|
| <role> | <what they do> | <how we know it's done> |

**Risk / cost**: <small | medium | large>

Approve, adjust, or pick something else?
- **approve** — proceed to Step 4
- **adjust** — change the plan
- **alt** — show next-best candidate
- **task <description>** — give me a directed task instead
- **cancel** — stop
```

Wait for user.

### Step 3: Directed Mode — Decompose Task

Given the user's task:

1. Read `IDEA.md` / `docs/requirements.md` / `aidlc-docs/aidlc-state.md` for context
2. Decompose into specialist sub-tasks. Match each to a role using the agents' `description` frontmatter
3. Identify parallelizable sub-tasks (no shared file writes, no ordering dependency)
4. Present the decomposition table (same shape as Step 2) and wait for approval

### Step 4: Delegate

For each approved sub-task, call:

```
Agent(
  subagent_type="<role-name>",
  description="<3-5 word task summary>",
  prompt="""
  Sub-task: <full description>

  Read these artifacts before acting:
  - <path>
  - <path>

  Acceptance criteria:
  - <criterion>
  - <criterion>

  Out of scope (do not do):
  - <thing>

  Report back in the standard format.
  """
)
```

**Parallelism rule**: When multiple sub-tasks are independent, send them in a **single message with multiple Agent tool calls** so they run concurrently. Sequential delegations only when one's output feeds another.

### Step 5: Synthesize and Present

After agents return:

1. Aggregate their reports into one summary table (Role / Status / Files / Blockers)
2. Identify integration issues (overlapping files, conflicting decisions)
3. If a reviewer is in the team, delegate a final review pass (read-only) before presenting
4. Present to user:

```
## Team Run Complete

### Summary
| Role | Status | Files Changed | Notes |
|------|--------|---------------|-------|
| ... | ... | ... | ... |

### Decisions Made
- <decision> — by <role>

### Blockers
- <blocker> — needs <action>

### Recommended Next Step
<one suggestion>

Options:
- **approve** — accept and write session log
- **adjust** — describe what to change, I'll re-delegate
- **revert** — undo all changes
```

### Step 6: Session Log

On approval, write `docs/sessions/YYYY-MM-DD-team-<short-title>.md`:

- Mode (autonomous / directed)
- Original task / autonomous candidate
- Roster used (which agents, which sub-tasks)
- Files changed
- Decisions and rationale
- Open follow-ups

Also append to `aidlc-docs/audit.md` per AIDLC audit format if any AIDLC stage advanced.

---

## Guidelines

1. **Approval before delegation** — never spawn agents without showing the plan
2. **Parallel by default** — if sub-tasks are independent, dispatch them in one message
3. **Scope discipline** — if an agent reports work outside its sub-task, flag it; don't quietly merge
4. **Use the roster** — re-read `docs/AGENT-TEAM.md` if you forget who does what
5. **AIDLC compliance** — for any work that advances an AIDLC stage, the relevant per-stage plan file and `aidlc-state.md` must be updated
6. **Health check first** — in autonomous mode, run the dev-skill Step 0 health check (TECH-DEBT escalation, pending cross-checks) before proposing work
7. **Don't impersonate specialists** — if you find yourself implementing code or writing tests directly, stop and delegate

## Error Handling

| Situation | Response |
|-----------|----------|
| No agents in `.claude/agents/` | Suggest `/build-team` first, exit |
| `docs/AGENT-TEAM.md` missing | Re-derive from `.claude/agents/*.md` frontmatter |
| Specialist returns blocked | Surface blocker to user, propose unblock options |
| Agent goes out of scope | Reject the work, re-delegate with tighter scope |
| User rejects synthesis | Offer revert or targeted re-delegation |

## Example Invocations

```
/team-lead
/team-lead "rip out the legacy auth middleware and replace with the new one"
/team-lead status
/team-lead roster
/team-lead replan
```
```

---

## Guidelines

1. **Signals over templates** — every proposed role must trace to a project signal. If you can't name the signal, don't propose the role.
2. **Mandatory team-lead** — singleton, always present, model = `opus`.
3. **One agent file per role** — multi-instance is a runtime concern (parallel `Agent(...)` calls), not a file-layout one.
4. **Tool scoping** — narrow each agent to the tools it actually needs. Reviewer is read-only-ish; planner doesn't need Bash.
5. **AIDLC alignment** — every agent prompt names the artifacts it produces and consumes, with paths.
6. **Brownfield awareness** — if `IDEA.md` has a "Current State" section, all agents must respect "What Must Not Change" constraints.
7. **No auto-commit** — the team-lead never commits; it presents synthesis and waits for user approval.
8. **Korean UX hint** — the project owner often works in Korean. Generated agent prompts should accept Korean input naturally and may surface short Korean summary alongside English when role descriptions are presented to the user. Do not translate AIDLC artifact paths.

---

## Error Handling

| Situation | Response |
|-----------|----------|
| Both team and dev-skill missing | Suggest `/init-project` first |
| `aidlc-docs/` empty but IDEA.md exists | Generate a smaller team (team-lead + planner + 1 developer) and note that more roles will be suggested after `/init-project` |
| User wants a role not in catalog | Accept it; generate with custom name and a prompt scaffolded from the closest catalog entry |
| Existing team but stale (units in aidlc-state changed) | Offer `regenerate` or `add` to reconcile |

---

## Example Invocations

Full guided flow:
```
/build-team
```

Propose only, no files:
```
/build-team propose
```

Add a role to existing team:
```
/build-team add security-engineer
```

Show the current team:
```
/build-team roster
```

Replace the existing team:
```
/build-team regenerate
```
