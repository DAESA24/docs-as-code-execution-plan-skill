# Docs-as-Code Execution Plan Skill

A Claude Code skill that automates the docs-as-code LLM execution pattern for complex, multi-step tasks.

## Overview

This skill implements a structured approach to creating, executing, and archiving automation plans. The docs-as-code pattern treats documentation as executable specifications - plans contain pre-written bash scripts that Claude executes autonomously, rather than improvising solutions.

### Core Principles

1. **Pre-Written Scripts** - Complete bash scripts, not instructions
2. **Read-Once Architecture** - All context embedded upfront
3. **Explicit Success Criteria** - Programmatically verifiable conditions
4. **Minimal Approval Points** - Only for destructive operations
5. **Built-In Validation** - Check, Execute, Verify pattern

## Installation

The skill is installed at:

```
~/.claude/skills/docs-as-code-execution-plan/
├── SKILL.md
└── references/
    ├── docs-as-code-guide.md
    └── docs-as-code-execution-plan-template.md
```

The slash command is installed at:

```
~/.claude/commands/execution-plan.md
```

### Verify Installation

```bash
# Check skill
ls ~/.claude/skills/docs-as-code-execution-plan/

# Check command
ls ~/.claude/commands/execution-plan.md
```

## Usage

### Trigger Phrases (Skill)

The skill triggers on phrases that include "docs-as-code":

- "Create a docs-as-code execution plan for..."
- "Help me plan this change using a docs-as-code execution plan"
- "Create a docs-as-code plan"
- "Execute this plan" (in context of a docs-as-code plan)
- "Archive this execution plan" (in context of a docs-as-code plan)

### Slash Command

Quick template scaffolding:

```
/execution-plan <topic words>
```

Examples:

```
/execution-plan ollama startup fix
/execution-plan database migration setup
/execution-plan openmemory upgrade
```

## Workflow Modes

### Mode 1: Create Execution Plan

Use when starting a new infrastructure task. The skill will:

1. Gather requirements (goal, current state, target state, risks)
2. Generate plan structure with YAML front matter
3. Create phases with bash scripts and success criteria
4. Include pre-flight validation and rollback procedure
5. Save to project's `docs/` folder

### Mode 2: Execute Existing Plan

Use to run a previously created plan. The skill will:

1. Read the entire plan before starting
2. Run pre-flight validation
3. Execute phases in order (autonomous or with approval)
4. Verify success criteria after each phase
5. Handle failures with rollback if needed

### Mode 3: Archive Completed Plan

Use after completing a task. The skill will:

1. Update YAML front matter status
2. Complete the Dev Agent Record section
3. Move to `docs/archives/` folder
4. Extract procedural learnings to OpenMemory (if available)

## File Naming Convention

All execution plans follow this pattern:

```
YYYY-MM-DD-<topic-words>-execution-plan[-vN].md
```

| Element | Rule |
|---------|------|
| Case | kebab-case (lowercase, hyphens) |
| Date prefix | Today's date |
| Topic words | 3-4 words max |
| Suffix | Always `-execution-plan` |
| Versioning | `-v2`, `-v3` only if previous exists |

Examples:

```
2025-12-03-ollama-startup-fix-execution-plan.md
2025-12-03-database-migration-execution-plan.md
2025-12-03-database-migration-execution-plan-v2.md
```

## Reference Files

The skill includes two reference files:

### docs-as-code-guide.md

Full documentation of the docs-as-code pattern, including:

- Why documentation is execution context
- Anti-patterns to avoid
- YAML front matter schema
- Status prefixes for LLM parsing
- Windows-specific patterns

### docs-as-code-execution-plan-template.md

Complete template with all sections:

- YAML front matter
- Test Plan Summary table
- Problem Statement
- Risk Assessment
- Pre-Flight Validation
- Numbered Phases
- Rollback Procedure
- Completion/Acceptance Checklists
- Dev Agent Record

## Related Files

- **Source Guide:** [user-context/2025-11-17-docs-as-code-llm-execution-guide.md](user-context/2025-11-17-docs-as-code-llm-execution-guide.md)

## Development

This dev project serves as the source repository for the skill. To make changes:

1. Edit files in this project
2. Copy updated files to `~/.claude/skills/docs-as-code-execution-plan/`
3. Run validation: `python ~/.claude/skills/skill-creator/scripts/quick_validate.py ~/.claude/skills/docs-as-code-execution-plan`

---

- **Created:** 2025-12-03
- **Last Updated:** 2025-12-04
- **Status:** In Progress (pending manual trigger test)
- **Author:** Drew Arnold (with Claude Code)
