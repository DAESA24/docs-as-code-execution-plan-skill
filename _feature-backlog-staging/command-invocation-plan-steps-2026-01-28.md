---
doc_type: feature-backlog-item
title: "Command Invocation Plan Steps"
created: 2026-01-28
status: captured
feature_type_tags:
  - enhancement
  - automation
  - integration
---

# Command Invocation Plan Steps

## Summary

Enable the docs-as-code-execution-plan skill to recognize command/skill invocations (e.g., `/feature-backlog-staging`, `/handoff-general`) as valid plan steps when converting input documents into execution plans.

## Problem Statement

When converting a checklist or requirements document into a docs-as-code execution plan, the skill currently only translates:
- Bash scripts
- Manual file operations
- Directory creation

It does **not** recognize command invocations like:
```
| Directory | Command | Notes |
|-----------|---------|-------|
| `feature-backlog-staging/` | `/feature-backlog-staging` | Sets up CLAUDE.md, template, and workflow |
| `_handoffs/` | `/handoff-general` | Sets up handoff utility directory |
```

These requirements get dropped during plan generation, resulting in incomplete execution plans.

## Discovery Context

- **Source:** Execution of `2026-01-28-project-init-execution-plan.md` for execution-logger-skill-dev
- **Input document:** `dev-project-setup-checklist.md` (clearly listed command invocations in Phase 2, Section 2.1)
- **Outcome:** Generated plan omitted `_feature-backlog-staging/` and `_handoffs/` directories
- **Root cause:** Skill has no logic to parse command invocation syntax from input documents

## Desired Behavior

When the skill generates an execution plan from an input document:

1. **Recognize command patterns** - Detect `/command-name` or `/skill-name` syntax in input
2. **Generate appropriate plan steps** - Create steps that invoke the command/skill
3. **Include validation** - Add checkboxes to verify command output (directories created, files generated)

### Example Input

```markdown
| Directory | Command |
|-----------|---------|
| `_feature-backlog-staging/` | `/feature-backlog-staging` |
```

### Example Output in Generated Plan

```markdown
### STEP 2.X: Set Up Feature Backlog Staging

**Autonomous:** YES

**Actions:**
- Invoke `/feature-backlog-staging check` to create the staging directory with templates

**Validation Checklist:**
- [ ] `_feature-backlog-staging/` directory exists
- [ ] `_feature-backlog-staging/CLAUDE.md` exists
- [ ] `_feature-backlog-staging/_frontmatter-template.md` exists

**Report:** "STEP 2.X COMPLETE: Feature backlog staging directory initialized"
```

## Implementation Notes

- The skill should accept command invocations as inputs from any input document type (backlog items, checklists, requirements docs)
- No need for project-type awareness - just faithfully translate what's in the input document
- Command steps may need different validation patterns than bash script steps

## Out of Scope

- Automatic detection of required commands based on project type (e.g., "this is a -dev project, so add feature-backlog-staging")
- The skill should remain generic; project-type requirements belong in the input document
