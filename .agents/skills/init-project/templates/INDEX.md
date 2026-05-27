# init-project Artifact Templates

These are the **output document templates** that `/init-project` writes to disk. They are kept here, outside `SKILL.md`, and loaded **just-in-time** at the step that needs them.

**Why separated:** keeping bulky markdown templates inline in `SKILL.md` inflates the skill prompt and pushes it toward "too low altitude" (hardcoded boilerplate). The skill should carry *intent and procedure*; the verbatim boilerplate lives here and is read only when that step runs.

| Template | Used by | Generates |
|----------|---------|-----------|
| `requirements-quick.md` | Quick Step 3 | `docs/requirements.md` (quick mode) |
| `aidlc-state-quick.md` | Quick Step 4 | `aidlc-docs/aidlc-state.md` (quick mode) |
| `requirements.md` | Step 5 | `docs/requirements.md` (full) |
| `refinement-log.md` | Step 6.1 | `docs/refinement-log.md` |
| `refinement-questions.md` | Step 6.2 | `docs/refinement-questions.md` |
| `vision.md` | Step 7 | `docs/vision.md` |
| `tech-env.md` | Step 7 | `docs/tech-env.md` |
| `aidlc-state.md` | Step 8 | `aidlc-docs/aidlc-state.md` (full) |
| `audit-init.md` | Step 8 | `aidlc-docs/audit.md` (initial entry) |
| `requirements-reference.md` | Step 9 | `aidlc-docs/inception/requirements/requirements.md` |
| `AGENTS.md` | Step 16 | project root `AGENTS.md` (full mode) |
| `AGENTS-quick.md` | Quick Step 6 | project root `AGENTS.md` (quick mode — shortened) |
| `DESIGN.md` | Step 16.1 | `docs/DESIGN.md` |
| `TECH-DEBT.md` | Step 16.2 | `docs/TECH-DEBT.md` |
| `README.md` | Step 16.3 | project root `README.md` |

**How to use:** read the named template, substitute `{placeholders}` with project-specific content drawn from prior-stage artifacts, write to the destination path. Do not echo the template back to the user — fill and write it.

**Canonical worked example:** see `examples/book-tracker/` at the project root for a full end-to-end example of what these artifacts look like once filled in (idea → requirements → state → final structure).
