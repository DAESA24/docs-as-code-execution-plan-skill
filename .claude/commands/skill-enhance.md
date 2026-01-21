# Skill Enhancement Execution Plan

Generates a docs-as-code execution plan for enhancing the docs-as-code-execution-plan skill itself, avoiding circular dependency by not invoking the skill.

## Arguments

$ARGUMENTS - Path to the approved feature backlog item (e.g., `user-context/feature-backlog-staging/sub-agent-parallelization-strategy-2026-01-21.md`)

## If No Arguments Provided

If `$ARGUMENTS` is empty or not provided:

1. **List available feature backlog items** - Scan `user-context/feature-backlog-staging/` for files with `status: approved` or `status: captured`
2. **Present options to Drew** - Show a numbered list of available items with their titles and status
3. **Ask Drew to select** - "Which feature backlog item should I create an enhancement plan for?"
4. **Proceed** with the selected item

## Purpose

When enhancing this skill, using the skill itself to generate the execution plan creates a circular dependency - the skill can't generate a plan demonstrating features it doesn't yet have. This command provides a manual workflow that borrows best practices from the skill without invoking it.

## Prerequisites

- Feature backlog item must exist and have `status: approved` (or be ready for approval)
- Feature backlog item should be sufficiently detailed (problem statement, requirements, proposed implementation)

## Instructions

### Step 1: Validate Feature Backlog Item

1. Read the feature backlog item at the provided path
2. Verify it has required frontmatter (`doc_type: feature-backlog-item`, `status: approved` or `captured`)
3. If status is `captured`, ask Drew if he wants to approve it first
4. Extract key information:
   - Title
   - Problem statement
   - Core requirements
   - Proposed implementation phases
   - Implementation order (if specified)

### Step 2: Read Current Skill Files

Read these files to understand what exists (for dependency checking):

```
~/.claude/skills/docs-as-code-execution-plan/
├── SKILL.md
└── references/
    ├── docs-as-code-guide.md
    └── docs-as-code-execution-plan-template.md
```

Also read the project CLAUDE.md for any structural conventions.

### Step 3: Perform Dependency Check

Compare proposed changes against existing content:

| Existing Content | Proposed Change | Conflict Status |
|------------------|-----------------|-----------------|
| (from skill files) | (from feature backlog) | Additive / Conflict / Terminology Change |

Flag any potential conflicts for Drew's review before proceeding.

### Step 4: Generate Execution Plan

Create execution plan following these structural requirements:

**YAML Front Matter:**
```yaml
---
doc_type: skill-enhancement-plan  # REQUIRED
title: <Feature Title> - Skill Enhancement
created: <today YYYY-MM-DD>
status: draft
agent: dev
execution_mode: docs-as-code
user_intervention: minimal
source_feature: <path to feature backlog item>
tags:
  - skill-enhancement
  - docs-as-code
  - <additional tags from feature>
---
```

**Required Sections:**

1. **Header** - Purpose, Audience (Claude Code), User Intervention note
2. **Execution Instructions** - Standard sequential execution rules with checkbox marking
3. **Test Plan Summary** - Phase overview table
4. **Problem Statement** - From feature backlog item
5. **Pre-Flight Validation** - Check skill files exist, paths correct
6. **Phase 1: Dependency Check** - Always first phase for skill enhancements
7. **Phase 2+: Implementation Phases** - Following skill update order:
   - `docs-as-code-guide.md` (pattern documentation)
   - `docs-as-code-execution-plan-template.md` (template implementation)
   - `SKILL.md` (skill definition updates)
   - Project `CLAUDE.md` (if structural conventions change)
8. **Phase N: Validation** - Final validation that all changes are consistent
9. **Rollback Procedure** - How to undo changes
10. **Agent Execution Notes** - Preservation targets, safe modification targets
11. **Dev Agent Record** - Empty, filled after execution

**Per-Step Requirements:**

- `**Autonomous:** YES/NO` marker
- Clear actions (bash scripts where applicable)
- `**Validation Checklist:**` with `- [ ]` checkboxes
- `**Report:** "STEP X.Y COMPLETE: <summary>"` marker

### Step 5: Save Execution Plan

1. Determine filename: `YYYY-MM-DD-<topic>-skill-enhancement-plan.md`
2. Check for existing files with same topic (add version if needed)
3. Save to `docs/` folder

### Step 6: Update Feature Backlog Item

1. Add `execution_plan: docs/<filename>.md` to frontmatter
2. If status was `captured`, update to `approved` and add `approved_date`
3. Do NOT change status to `implemented` - that happens after plan execution

### Step 7: Report Summary

Report to Drew:
- Execution plan location
- Number of phases
- Any dependency conflicts found
- Feature backlog item updated with execution_plan reference
- Next step: "Review the plan, then run 'execute the <plan name>'"

## Example

**Input:**
```
/skill-enhance user-context/feature-backlog-staging/sub-agent-parallelization-strategy-2026-01-21.md
```

**Output:**
- Creates: `docs/2026-01-21-sub-agent-parallelization-skill-enhancement-plan.md`
- Updates: Feature backlog item with `execution_plan` field
- Reports: Summary of what was created

## Key Differences from Main Skill

| Aspect | Main Skill | This Command |
|--------|-----------|--------------|
| Invokes skill | Yes | No (avoids circular dependency) |
| Input | User description | Feature backlog item |
| First phase | Varies | Always dependency check |
| Update order | Varies | Fixed: guide → template → SKILL.md → CLAUDE.md |
| Output naming | `-execution-plan.md` | `-skill-enhancement-plan.md` |
| Backlog update | N/A | Adds execution_plan field |

## Bootstrap Pattern

This command implements the "bootstrap" pattern for self-referential systems: when a system needs to improve itself, use a simplified/manual version of the desired end state to bridge the gap. The generated plan can demonstrate new features while implementing them.
