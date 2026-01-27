---
doc_type: feature-backlog-item
title: "Add Validation Loop Execution Protocol"
created: 2026-01-26
status: captured
execution_order: 1
execution_sequence:
  - position: 1
    item: validation-loop-execution-protocol-2026-01-26.md
    status: THIS ITEM
  - position: 2
    item: pre-execution-git-safety-check-2026-01-26.md
    depends_on: position 1
  - position: 3
    item: autonomous-execution-permissions-2026-01-26.md
    depends_on: position 2
feature_type_tags:
  - enhancement
  - execution
  - documentation
---

# Add Validation Loop Execution Protocol

## Problem Statement

The docs-as-code execution plan skill lacks explicit enforcement of the validation loop protocol during plan execution. While the current execution instructions mention marking checkboxes and reporting after each step, Claude routinely batches these updates to the end of execution rather than performing them immediately after each step.

### Observed Behavior

During execution of the `2026-01-26-url-to-markdown-json-meta-execution-plan.md` in `pbc-web-crawling-dev`:

1. Claude executed all phases sequentially
2. Checkboxes were marked in bulk at the very end
3. No re-reading of the plan file between phases
4. Plan file was not treated as an operational hub during execution

### Root Cause

The current SKILL.md Mode 2 (Execute Existing Plan) has weak, buried guidance:

- "Edit plan file to mark Validation Checklist checkboxes as items are verified" (line 145)
- "Report after each step using the Report marker format" (line 146)
- "Verify Validation Checklist after each step" (line 147)

These are bullet points in a list, not framed as a non-negotiable protocol. Compare to the skill-enhance skill which:

1. Includes the full validation loop protocol **inline in every generated plan**
2. Frames it as "Non-Negotiable Steps" with explicit anti-patterns
3. Has an explicit "What NOT to Do" table showing correct vs incorrect behavior

### Impact

- Plan files are not updated during execution, making recovery from interruptions difficult
- No audit trail of step-by-step progress
- Goals drift out of attention window during long executions
- Errors may be silently retried instead of logged

## Requirements

The following requirements ensure the validation loop protocol is enforced during execution:

- **MUST** include the validation loop protocol inline in the execution plan template
- **MUST** frame the protocol as non-negotiable with explicit anti-patterns
- **MUST** require reading the plan before each phase (attention refresh)
- **MUST** require marking checkboxes immediately after each step (not batched)
- **MUST** require logging errors to the plan file before retrying
- **SHOULD** align with the protocol already defined in skill-enhance's `reference.md`
- **SHOULD** reference the Manus context engineering principles as rationale

## Implementation Phases

Implementation follows the skill file update order defined in `skill-dev-config.yaml`:

### Phase 1: Update docs-as-code-guide.md

- **File:** `references/docs-as-code-guide.md`
- **Changes:**
  - Add new pattern: "Pattern N: Validation Loop Protocol"
  - Include the core principles from Manus context engineering
  - Document the non-negotiable execution steps
  - Add anti-patterns table

### Phase 2: Update docs-as-code-execution-plan-template.md

- **File:** `references/docs-as-code-execution-plan-template.md`
- **Changes:**
  - Add "Execution Protocol: Validation Loop (MANDATORY)" section after header
  - Include the full protocol inline (same approach as skill-enhance)
  - Add "Errors Encountered" section placeholder
  - Update Execution Instructions to reference the protocol section

### Phase 3: Update SKILL.md

- **File:** `SKILL.md`
- **Changes:**
  - Update Mode 2 (Execute Existing Plan) to reference the validation loop protocol
  - Add explicit "Non-Negotiable Execution Rules" subsection
  - Add "What NOT to Do" anti-patterns table
  - Update Quality Checklist to include protocol presence check

## Acceptance Criteria

- [ ] `docs-as-code-guide.md` contains validation loop pattern documentation
- [ ] Template includes inline execution protocol section (mandatory, not optional)
- [ ] Template includes "Errors Encountered" section placeholder
- [ ] SKILL.md Mode 2 explicitly references validation loop protocol
- [ ] SKILL.md includes anti-patterns table for execution behavior
- [ ] Quality Checklist includes "Execution Protocol section present" item
- [ ] Test: Generate a new execution plan and verify protocol section is included
- [ ] Test: Execute a plan and verify checkboxes are marked immediately (not batched)

## Reference Materials

The validation loop protocol is well-documented in the skill-enhance skill:

- **Protocol Definition:** `~/.claude/skills/skill-enhance/reference.md`
- **Inline Template:** `~/.claude/skills/skill-enhance/SKILL.md` (lines 157-254)
- **Origin:** Manus context engineering principles (planning-with-files skill reference)

Key sections to align with:

1. Core Principles (Filesystem as External Memory, Attention Manipulation, Keep Failure Traces)
2. The Loop - Non-Negotiable Steps (BEFORE/DURING/AFTER each phase)
3. What NOT to Do (Anti-Patterns table)
4. Recovery Protocol (resume from last checked item)
5. Completion Protocol (update feature backlog, archive)

## Context and Evidence

### Comparison: Current vs Expected Behavior

| Aspect | Current Behavior | Expected Behavior |
|--------|------------------|-------------------|
| Checkbox marking | Batched at end | Immediate after each step |
| Plan re-reading | Once at start | Before each phase |
| Error handling | Silent retry | Log to file, then retry |
| Progress visibility | Verbal reports only | File updates + verbal |
| Recovery | Restart from beginning | Resume from last [x] |

### Session Evidence

- **Date:** 2026-01-26
- **Project:** pbc-web-crawling-dev
- **Plan executed:** `2026-01-26-url-to-markdown-json-meta-execution-plan.md`
- **Observation:** Screenshot shows all checkboxes marked at once after execution completed

---

- **Document Status:** Captured
- **Last Updated:** 2026-01-26
- **Related:** skill-enhance validation loop protocol
