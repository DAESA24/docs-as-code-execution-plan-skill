# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Development repository for the `docs-as-code-execution-plan` Claude Code skill. This skill automates the docs-as-code LLM execution pattern for complex, multi-step tasks with pre-written bash scripts.

The skill is installed globally at `~/.claude/skills/docs-as-code-execution-plan/`. This repo serves as the source repository for development and as a public showcase.

## Skill Architecture

### Installed Skill Structure

```
~/.claude/skills/docs-as-code-execution-plan/
├── SKILL.md                  # Skill definition with YAML frontmatter
└── references/
    ├── docs-as-code-guide.md                      # Pattern documentation
    └── docs-as-code-execution-plan-template.md    # Plan template with YAML
```

### Slash Command

```
~/.claude/commands/execution-plan.md    # Quick template scaffolding
```

## Development Workflow

### Making Changes to the Skill

1. Edit files in this project's `user-context/` for reference material
2. Copy updated files to `~/.claude/skills/docs-as-code-execution-plan/`
3. Validate:
   ```powershell
   python ~/.claude/skills/skill-creator/scripts/quick_validate.py ~/.claude/skills/docs-as-code-execution-plan
   ```

### Testing

Test skill in a separate project (not this one) to catch permission issues:
- Trigger phrase: "Create a docs-as-code execution plan for..."
- Slash command: `/execution-plan <topic words>`

## File Naming Convention

Execution plans follow: `YYYY-MM-DD-<topic-words>-execution-plan[-vN].md`

- kebab-case, 3-4 topic words max
- Version suffix only if previous exists

## Key Reference Files

| File | Purpose |
|------|---------|
| [user-context/2025-11-17-docs-as-code-llm-execution-guide.md](user-context/2025-11-17-docs-as-code-llm-execution-guide.md) | Source guide (sanitized for public) |
| [docs/archives/](docs/archives/) | Archived execution plans (gitignored - contains personal paths) |

## Privacy Notes

Files excluded from git (see `.gitignore`):
- `.claude/handoffs/` - Session-specific documents
- `docs/archives/` - Contain personal Windows paths
- `user-context/manual-test-screenshots/` - Test artifacts
- `user-context/2025-12-02-ollama-startup-fix-v3-plan.md` - Personal infrastructure

When making changes to public files, replace personal paths (`/c/Users/drewa/...`) with generic paths (`/c/Users/username/...`).

## Windows-Specific

All bash scripts use Git Bash paths:
- Correct: `/c/Users/username/...`
- Wrong: `C:\Users\username\...`

Python scripts with emojis need: `PYTHONIOENCODING=utf-8`
