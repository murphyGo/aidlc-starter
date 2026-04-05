# AI-DLC Starter

Transform ideas into fully-specified, development-ready projects using AI-DLC methodology and Claude-powered automation.

## What is This?

AI-DLC Starter is a template project that bridges:
- **AI-DLC** (AWS's AI-Driven Development Life Cycle methodology)
- **Claude Code Skills** (executable automation commands)

**Result**: Idea → Enhanced Requirements → AI-DLC Specs → Working Project

## Quick Start

### 1. Clone and Setup

```bash
git clone https://github.com/murphyGo/aidlc-starter.git my-project
cd my-project
```

### 2. Write Your Idea

Create `docs/inception.md` with your project idea:

```markdown
# My Project

## Vision
A REST API for managing personal book collections

## Core Features
- User authentication (JWT)
- CRUD operations for books
- Search by title/author
- Export to CSV

## Tech Preferences
- Language: Go
- Database: PostgreSQL
```

### 3. Run Init

```bash
claude
```

Then run:
```
/init-project
```

Claude will:
1. **Analyze** your idea and suggest improvements
2. **Refine** requirements through interactive dialogue
3. **Generate** AI-DLC specification documents
4. **Create** project-specific development skills

## Three-Stage Automation

### Stage 0: Interactive Requirements Refinement

Claude analyzes your inception.md and:
- Identifies gaps (error handling, edge cases, security)
- Suggests improvements with clear rationale
- Engages in dialogue until requirements are solid
- Generates enhanced `docs/requirements.md`

### Stage 1: AI-DLC Spec Generation

Creates AI-DLC input documents:
- `docs/vision.md` - Project vision and goals
- `docs/tech-env.md` - Technical environment spec

Then guides AI-DLC workflow to produce:
- `aidlc-docs/inception/` - Requirements, user stories, application design
- `aidlc-docs/construction/` - Functional design, NFRs, build plans

### Stage 2: Skill Generation

Creates project-specific automation:
- `/dev-{project}` - Main development driver
- `/code-review` - Code quality checks
- `/tech-debt` - Debt management
- `/cross-check` - Requirements compliance

## Available Skills

| Skill | Purpose |
|-------|---------|
| `/init-project` | Bootstrap new project from idea |
| `/code-review git` | Review changed files for issues |
| `/tech-debt` | View/manage technical debt |
| `/cross-check` | Verify implementation vs requirements |

## Project Structure

```
aidlc-starter/
├── README.md
├── CLAUDE.md                    # Claude Code context
├── docs/
│   ├── inception.md             # Your idea (input)
│   ├── DESIGN.md                # Architecture decisions
│   ├── requirements.md          # Enhanced requirements (generated)
│   ├── development-plan.md      # Development roadmap (generated)
│   └── TECH-DEBT.md             # Technical debt tracker
├── .claude/
│   └── skills/
│       ├── init-project/        # Bootstrap skill
│       ├── code-review/         # Code quality skill
│       ├── tech-debt/           # Debt management skill
│       └── cross-check/         # Compliance skill
└── aidlc-workflows/             # AWS AI-DLC rules
    └── aidlc-rules/
        ├── aws-aidlc-rules/     # Core workflow
        └── aws-aidlc-rule-details/  # Stage details
```

## Feedback Loop

The system supports continuous improvement:

1. **Session Logs** - Every dev session creates traceability
2. **TECH-DEBT Tracking** - Issues captured and prioritized
3. **Cross-Check Reports** - Phase completion triggers compliance verification
4. **Plan Evolution** - Development plan updates as work progresses

## Requirements

- [Claude Code CLI](https://github.com/anthropics/claude-code)
- Git

## How It Works

```
┌─────────────────┐
│  inception.md   │  Your rough idea
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  /init-project  │  Claude-powered refinement
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ requirements.md │  Enhanced, structured requirements
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    AI-DLC       │  Spec generation workflow
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  aidlc-docs/    │  Complete specifications
│  dev skills     │  Project automation
└─────────────────┘
```

## License

MIT
