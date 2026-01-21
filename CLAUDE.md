# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Repository Purpose

Development repository for the `docs-as-code-execution-plan` Claude Code skill. This skill automates the docs-as-code LLM execution pattern for complex, multi-step tasks.

**Dual role:**

- **Source repository:** All skill development happens here
- **Public showcase:** README and sanitized files demonstrate the pattern

**Important:** The skill is installed globally at `~/.claude/skills/docs-as-code-execution-plan/`. Changes made in this repo must be manually synced to the global location to take effect.

## Project Structure

```
docs-as-code-execution-plan-skill-dev/
├── .claude/
│   └── commands/
│       └── skill-enhance.md          # Bootstrap command for self-enhancement
├── _handoffs/                        # Session handoff documents (gitignored)
├── docs/
│   └── archives/                     # Completed execution plans (gitignored)
├── feature-backlog-staging/          # Feature intake system
│   ├── CLAUDE.md                     # Workflow instructions for this directory
│   ├── _frontmatter-template.md      # Schema for feature items
│   └── *.md                          # Feature backlog items (x- prefix = completed)
├── user-context/
│   ├── @insight-decision-captures/   # Captured insights from sessions
│   └── *.md                          # Reference materials
├── CLAUDE.md                         # This file
└── README.md                         # Public documentation
```

## When Working in This Project

### Enhancing this skill

Use the `/skill-enhance` command. This command exists because using the skill to enhance itself creates a circular dependency - the skill cannot generate a plan demonstrating features it does not yet have.

```
/skill-enhance feature-backlog-staging/<feature-file>.md
```

If no argument is provided, the command lists available feature backlog items.

### Capturing a new feature idea

Create a file in `feature-backlog-staging/` following the frontmatter template. See `feature-backlog-staging/CLAUDE.md` for the full workflow.

### Testing changes to the skill

Test in a SEPARATE project, not this one. Testing here will not catch permission issues that occur when the skill runs in other contexts.

- Trigger phrase: "Create a docs-as-code execution plan for..."
- Slash command: `/execution-plan <topic words>`

### Syncing changes to the installed skill

After making changes:

1. Copy updated files to `~/.claude/skills/docs-as-code-execution-plan/`
2. Validate:
   ```powershell
   python ~/.claude/skills/skill-creator/scripts/quick_validate.py ~/.claude/skills/docs-as-code-execution-plan
   ```

## Available Commands

| Command           | Location                             | Purpose                                                      |
|-------------------|--------------------------------------|--------------------------------------------------------------|
| `/skill-enhance`  | `.claude/commands/skill-enhance.md`  | Bootstrap for self-enhancement (avoids circular dependency)  |
| `/execution-plan` | `~/.claude/commands/execution-plan.md` | Global command for general use in any project              |

## Installed Skill Structure

The globally installed skill at `~/.claude/skills/docs-as-code-execution-plan/`:

```text
docs-as-code-execution-plan/
├── SKILL.md                                    # Skill definition with YAML frontmatter
└── references/
    ├── docs-as-code-guide.md                   # Pattern documentation
    └── docs-as-code-execution-plan-template.md # Plan template
```

## File Naming Conventions

**Execution plans:** `YYYY-MM-DD-<topic-words>-execution-plan[-vN].md`

- kebab-case, 3-4 topic words max
- Version suffix only if previous version exists

**Skill enhancement plans:** `YYYY-MM-DD-<topic>-skill-enhancement-plan.md`

**Feature backlog items:** `<topic>-<date>.md` or `x-<topic>-<date>.md` (x- prefix = completed/archived)

## Execution Plan Structure (Quick Reference)

Plans created with this skill include these sections:

| Section                | Purpose                                                                         |
|------------------------|---------------------------------------------------------------------------------|
| YAML front matter      | Metadata (title, status, tags)                                                  |
| Execution Instructions | Rules for sequential execution, stop-on-failure                                 |
| Test Plan Summary      | Phase overview with pass/fail tracking                                          |
| Pre-Flight Validation  | Prerequisites check before execution                                            |
| Phases with Steps      | Each step has: Autonomous marker, Actions, Validation Checklist, Report marker  |
| Agent Execution Notes  | Preservation targets, safe modification targets                                 |
| Rollback Procedure     | How to undo if needed                                                           |
| Dev Agent Record       | Post-execution log (filled after completion)                                    |

**Key patterns:**

- Validation Checklists use `- [ ]` checkboxes marked `- [x]` during execution
- Report markers: `**Report:** "STEP X.Y COMPLETE: <summary>"`
- Phase validation step at end of each phase

## Privacy Notes

Files excluded from git (see `.gitignore`):

- `_handoffs/` - Session-specific documents
- `docs/archives/` - Contain personal Windows paths
- `user-context/manual-test-screenshots/` - Test artifacts
- `user-context/2025-12-02-ollama-startup-fix-v3-plan.md` - Personal infrastructure

When making changes to public files, replace personal paths (`/c/Users/drewa/...`) with generic paths (`/c/Users/username/...`).

## Windows-Specific

All bash scripts use Git Bash paths:

- Correct: `/c/Users/username/...`
- Wrong: `C:\Users\username\...`

Python scripts with emojis need: `PYTHONIOENCODING=utf-8`
