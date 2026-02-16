# Claude Code Skills Collection

Custom skills for [Claude Code](https://claude.com/claude-code) — Anthropic's CLI for Claude. Each skill adds specialized commands for building projects, learning concepts, and leveling up from beginner to professional.

## Skills Included

| # | Skill | Command | Description |
|---|-------|---------|-------------|
| 1 | **FastAPI** | `/fastapi` | Build FastAPI projects, learn concepts, create endpoints, add JWT auth, database integration, testing, and deployment. |
| 2 | **SQLModel** | `/sqlmodel` | Build database-driven apps with SQLModel. Models, CRUD, relationships, FastAPI integration, Alembic migrations. |
| 3 | **API** | `/api` | Learn about APIs, build API projects, consume third-party APIs, design REST/GraphQL APIs. |
| 4 | **CV Maker** | `/cv-maker` | Create, update, and optimize professional CVs/resumes. PDF, HTML, Markdown. ATS optimization. |

## Installation

Copy any skill folder into your Claude Code skills directory:

```bash
# Copy a single skill
cp -r fastapi/ ~/.claude/skills/fastapi/

# Or copy all skills at once
cp -r */ ~/.claude/skills/
```

Skills are auto-discovered by Claude Code — no configuration needed. Just restart Claude Code and type the command (e.g., `/fastapi roadmap`).

## Skill Structure

Each skill follows the same format:

```
skill-name/
├── SKILL.md              # Main skill file (YAML frontmatter + instructions)
└── references/           # Optional detailed reference docs
    ├── topic1.md
    └── topic2.md
```

The `SKILL.md` file contains:
- **YAML frontmatter** — name, description, allowed tools
- **Command definitions** — available actions and how to invoke them
- **Learning roadmap** — progressive levels from beginner to professional
- **Code patterns** — best practices and reusable templates
- **Debugging guide** — common issues and fixes

## Quick Start Examples

```bash
# Scaffold a new FastAPI project
/fastapi new blog api

# Learn SQLModel relationships
/sqlmodel learn relationships

# Design a REST API
/api design user management api

# Create a professional CV
/cv-maker

# Generate CRUD for a resource
/sqlmodel crud products
/fastapi crud users
```

## Requirements

- [Claude Code](https://claude.com/claude-code) CLI installed
- Skills placed in `~/.claude/skills/` directory
