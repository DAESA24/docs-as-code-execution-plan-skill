# Docs-as-Code Execution Plan Skill Development

- **Project Type:** Claude Code Skill Development
- **Skill Name:** docs-as-code-execution-plan
- **Target Location:** `~/.claude/skills/docs-as-code-execution-plan/`

## Project Purpose

This workspace is for developing and maintaining the `docs-as-code-execution-plan` skill, which automates the creation, execution, and archiving of LLM execution plans following the docs-as-code pattern.

## Directory Structure

```
docs-as-code-execution-plan-skill/
├── .claude/
│   └── handoffs/           # Session handoffs for continuity
├── docs/
│   └── *.md                # Active execution plans for this skill's development
├── user-context/
│   ├── docs-as-code guide  # The source pattern documentation
│   └── example plans       # Reference execution plans
└── CLAUDE.md               # This file
```

## Development Workflow

### Starting a Session

1. Run `/pickup` to resume from the latest handoff
2. Review the execution plan in `docs/`
3. Continue from where the previous session left off

### Creating the Skill

The skill will be created at `~/.claude/skills/docs-as-code-execution-plan/` with:
- `SKILL.md` - Main skill definition
- `references/` - Template and guide files

### Testing the Skill

After installation, test with trigger phrases:
- "Create an execution plan for X"
- "Help me plan this change"
- "Execute this plan"
- "Archive this execution plan"

## Key Files

| File | Purpose |
|------|---------|
| `docs/2025-12-03-infrastructure-skill-creation-execution-plan.md` | The execution plan for building this skill |
| `user-context/2025-11-17-docs-as-code-llm-execution-guide.md` | The docs-as-code pattern guide |
| `user-context/2025-12-02-ollama-startup-fix-v3-plan.md` | Example execution plan for reference |

## Skill Requirements

The skill must support:

1. **Directory Management**
   - Auto-create `docs/` and `docs/archives/` if missing
   - New plans → `docs/`
   - Completed plans → `docs/archives/`

2. **Changelog Integration**
   - Auto-create `CHANGELOG.md` if missing
   - Update changelog when plans complete successfully

3. **File Naming Convention**
   - Format: `YYYY-MM-DD-<topic-words>-execution-plan[-vN].md`
   - Auto-detect and increment versions

4. **Three Workflow Modes**
   - Create: Generate new execution plans
   - Execute: Run existing plans autonomously
   - Archive: Move completed plans and update changelog

## References

- **Skill Creator:** `~/.claude/skills/skill-creator/` - Best practices for skill creation
- **Pattern Guide:** `user-context/2025-11-17-docs-as-code-llm-execution-guide.md`

---

- **Document Status:** Active
- **Created:** 2025-12-03
- **Related:** [cc-skills-dev](../) parent directory for all skill development
