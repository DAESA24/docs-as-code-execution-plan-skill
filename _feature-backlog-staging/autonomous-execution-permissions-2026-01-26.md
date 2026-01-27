---
doc_type: feature-backlog-item
title: "Add Autonomous Execution Permissions"
created: 2026-01-26
status: captured
execution_order: 3
execution_sequence:
  - position: 1
    item: validation-loop-execution-protocol-2026-01-26.md
    status: implemented
  - position: 2
    item: pre-execution-git-safety-check-2026-01-26.md
    status: implemented
  - position: 3
    item: autonomous-execution-permissions-2026-01-26.md
    status: THIS ITEM
  - position: 4
    item: execution-logging-2026-01-27.md
    status: pending (depends on this item)
depends_on: pre-execution-git-safety-check-2026-01-26.md
feature_type_tags:
  - enhancement
  - execution
  - permissions
---

# Add Autonomous Execution Permissions

## Problem Statement

When executing a docs-as-code execution plan, Claude Code prompts for permission on every tool operation (Edit, Write, Bash). This creates significant friction despite the fact that all scripts in the plan are pre-vetted during plan creation. The user approves each prompt without reviewing because the vetting already happened - making the prompts "security theater" rather than actual security.

### Prerequisites

**This feature depends on:** `pre-execution-git-safety-check-2026-01-26.md`

The safety net (git rollback point) must be in place before granting broad permissions. Pattern: "Safety first, then freedom."

### Current State

- Skill has no `allowed-tools` field in SKILL.md frontmatter
- Every Edit, Write, and Bash operation prompts for approval
- User approves without reviewing (plan is already vetted)
- Friction without corresponding security benefit

### Target State

- Skill declares `allowed-tools` in SKILL.md frontmatter
- Autonomous steps execute without permission prompts
- Steps marked `Autonomous: NO` still pause for user confirmation (skill logic, not tool permissions)
- Trust model is explicit: vetting happens at plan creation, not execution

## Specification Status

**Field:** `allowed-tools` is a valid YAML frontmatter field per the [Agent Skills Specification](https://agentskills.io/specification).

**Status:** **Experimental** - The spec explicitly states support may vary between agent implementations.

**Implication:** Claude Code may or may not honor this field. Implementation must include verification testing.

### Syntax Options

```yaml
# Simple (blanket permissions)
allowed-tools: Read Edit Write Bash

# Scoped (more secure - recommended)
allowed-tools: Read Edit Write Bash(git:*) Bash(npm:*)
```

The scoped syntax (`Bash(git:*)`) restricts which commands are pre-approved. Consider using scoped permissions for better security posture.

## Requirements

The following requirements enable frictionless autonomous execution:

- **MUST** verify Claude Code honors `allowed-tools` before full implementation
- **MUST** add `allowed-tools` field to SKILL.md frontmatter
- **MUST** include these tools: `Read`, `Edit`, `Write`, `Bash`, `Glob`
- **MUST** document the trust model (vetting at plan creation, not execution)
- **MUST** document the experimental status of `allowed-tools`
- **MUST** preserve `Autonomous: NO` pause behavior for destructive operations
- **SHOULD** consider scoped Bash permissions (e.g., `Bash(git:*)`) vs blanket `Bash`
- **SHOULD** clarify the distinction between tool permissions and `Autonomous: YES/NO` markers
- **SHOULD** note that user's deny rules in settings still take precedence
- **SHOULD** include fallback documentation if feature isn't fully supported

## Implementation Phases

Implementation follows the skill file update order defined in `skill-dev-config.yaml`:

### Phase 0: Verify Claude Code Support

- **Goal:** Confirm Claude Code honors `allowed-tools` before proceeding
- **Steps:**
  1. Create a minimal test skill with `allowed-tools: Read Edit`
  2. Invoke the skill in a test project
  3. Attempt Read and Edit operations
  4. Observe whether permission prompts appear
- **Gate:** If prompts still appear, this feature cannot be implemented as designed. Document findings and close as "blocked on Claude Code support."

### Phase 1: Update docs-as-code-guide.md

- **File:** `references/docs-as-code-guide.md`
- **Changes:**
  - Add section: "Trust Model: Plan-Time Vetting"
  - Document the relationship between `allowed-tools` and `Autonomous: YES/NO`
  - Explain that tool permissions enable execution, markers control flow

### Phase 2: Update SKILL.md

- **File:** `SKILL.md`
- **Changes:**
  - Add `allowed-tools: Read, Edit, Write, Bash` to YAML frontmatter
  - Add "Permissions and Trust Model" section documenting:
    - Why broad permissions are granted
    - How vetting happens at plan creation
    - How `Autonomous: NO` still creates pause points
    - How user deny rules take precedence

## Acceptance Criteria

### Phase 0 Gate (Must Pass Before Proceeding)

- [ ] Test skill created with `allowed-tools` field
- [ ] Claude Code verified to honor `allowed-tools` (no prompts for listed tools)
- [ ] If not honored: document findings, close feature as blocked

### Implementation Criteria

- [ ] SKILL.md frontmatter includes `allowed-tools: Read Edit Write Bash Glob`
- [ ] Guide documents the trust model (plan-time vetting)
- [ ] Guide explains `allowed-tools` vs `Autonomous: YES/NO` distinction
- [ ] Guide notes experimental status of `allowed-tools`
- [ ] SKILL.md has "Permissions and Trust Model" documentation section

### Validation Criteria

- [ ] Test: Execute autonomous step, verify no permission prompt
- [ ] Test: Execute `Autonomous: NO` step, verify pause for confirmation
- [ ] Test: Verify user deny rules in settings.json still block operations

## Technical Details

### SKILL.md Frontmatter Change

```yaml
---
name: docs-as-code-execution-plan
description: ...existing description...
allowed-tools: Read Edit Write Bash Glob
---
```

**Note:** The spec uses space-delimited format, not comma-delimited.

### Trust Model Documentation

The trust model shifts vetting from execution-time to plan-creation-time:

| Stage | What Happens | Who Reviews |
|-------|--------------|-------------|
| Plan Creation | Scripts written, documented | User reviews plan |
| Plan Approval | User says "execute this plan" | Implicit approval of all scripts |
| Plan Execution | Scripts run autonomously | No review (already vetted) |

### Two-Layer Permission Model

| Layer | Controls | Example |
|-------|----------|---------|
| `allowed-tools` (SKILL.md) | Can Claude use this tool without prompting? | `Bash` → yes, run bash |
| `Autonomous: YES/NO` (plan) | Should Claude pause and ask user? | `NO` → pause before destructive operation |

These are independent:
- `allowed-tools` enables the tool at the permission level
- `Autonomous: NO` is a skill-level instruction to pause and confirm

## Related Items

- **Depends on:** `pre-execution-git-safety-check-2026-01-26.md`
  - The safety check must be implemented first
  - Pattern: "Safety first, then freedom"

- **Depended on by:** `execution-logging-2026-01-27.md`
  - Logging is most valuable during autonomous execution
  - Pattern: "Autonomy requires accountability"

## Context and Evidence

### Origin

Discussion with Drew on 2026-01-26. Key insight: Drew already approves every permission prompt without reviewing because the plan is pre-vetted. The prompts are friction without security benefit.

### Design Principle

> "Safety first, then freedom" - the git safety check provides actual rollback capability, enabling us to grant broad permissions with confidence.

---

- **Document Status:** Captured (Updated with spec verification)
- **Last Updated:** 2026-01-27
- **Depends On:** pre-execution-git-safety-check-2026-01-26.md
