---
doc_type: feature-backlog-item
title: "Add Validation Loop Execution Protocol"
created: 2026-01-26
status: implemented
implemented_date: 2026-01-26
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
execution_plan: docs/2026-01-26-validation-loop-protocol-skill-enhancement-plan.md
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

### Phase 1: Create execution-protocol-reference.md

- **File:** `references/execution-protocol-reference.md` (NEW)
- **Purpose:** Authoritative source for execution rules, separate from pattern documentation
- **Changes:**
  - Create new reference document focused on execution behavior
  - Include the core principles from Manus context engineering (why the protocol matters)
  - Document the non-negotiable execution steps (Critical Rules)
  - Add "What NOT to Do" anti-patterns table
  - Include recovery protocol (resume from last checked item)

### Phase 2: Update docs-as-code-execution-plan-template.md

- **File:** `references/docs-as-code-execution-plan-template.md`
- **Purpose:** Embed protocol inline so it travels with every generated plan
- **Changes:**
  - Add "Execution Protocol: Validation Loop (MANDATORY)" section after header
  - Include the full protocol inline (same approach as skill-enhance)
  - Add "Errors Encountered" section placeholder
  - Update Execution Instructions to reference the protocol section

### Phase 3: Update SKILL.md

- **File:** `SKILL.md`
- **Purpose:** Reference the protocol document, keep SKILL.md focused on modes
- **Changes:**
  - Update Mode 2 (Execute Existing Plan) to reference `execution-protocol-reference.md`
  - Keep Mode 2 focused (no inline rules or anti-patterns table)
  - Update Quality Checklist to include "Execution Protocol section present" item

## Acceptance Criteria

- [ ] `references/execution-protocol-reference.md` exists with Critical Rules and anti-patterns table
- [ ] Template includes inline execution protocol section (mandatory, not optional)
- [ ] Template includes "Errors Encountered" section placeholder
- [ ] SKILL.md Mode 2 references `execution-protocol-reference.md` (not inline rules)
- [ ] Quality Checklist includes "Execution Protocol section present" item
- [ ] Test: Generate a new execution plan and verify protocol section is included
- [ ] Test: Execute a plan and verify checkboxes are marked immediately (not batched)

## Reference Materials

### Primary Reference: Planning With Files Skill

The validation loop protocol patterns are established in the Planning With Files skill (`~/.claude/skills/planning-with-files/`). This skill demonstrates proven patterns for:

- **Critical Rules section** (SKILL.md lines 189-244) - Numbered, non-negotiable rules
- **Anti-Patterns table** (SKILL.md lines 350-360) - "Don't / Do Instead" format
- **Errors Encountered section** - Single section in the plan template (line 162-163), format: `- [Error]: [Resolution]`
- **The Loop pattern** - Explicit workflow showing when to read/write/edit around each phase

### Specific Patterns to Adapt

| Pattern | Planning With Files Location | Target Location |
| ------- | ---------------------------- | --------------- |
| Critical Rules | SKILL.md "Critical Rules" section | `execution-protocol-reference.md` |
| Read Before Decide | Rule #2: "Before any major decision, read the plan file" | `execution-protocol-reference.md` + template inline |
| Update After Act | Rule #3: "After completing any phase, immediately update" | `execution-protocol-reference.md` + template inline |
| Log All Errors | Rule #5: "Every error goes in the 'Errors Encountered' section" | `execution-protocol-reference.md` + template inline |
| Anti-patterns table | "Anti-Patterns to Avoid" table | `execution-protocol-reference.md` |
| Errors Encountered section | task_plan.md template (line 162-163) | Template inline |

### Errors Encountered Section Placement

The "Errors Encountered" section is a **single section** in the plan (not per-phase). Located in the plan body alongside "Decisions Made":

```markdown
## Errors Encountered
- [Error]: [Resolution]
```

This matches the Planning With Files task_plan.md template structure.

### Secondary References

- **skill-enhance skill:** `~/.claude/skills/skill-enhance/` - Shows inline protocol in generated plans
- **Manus context engineering:** `~/.claude/skills/planning-with-files/reference.md` - Documents the underlying principles

### Architectural Decision: Separate Reference Document

**Decision:** Create `execution-protocol-reference.md` rather than adding rules to SKILL.md or docs-as-code-guide.md.

**Rationale:**

- **Separation of concerns:** Guide explains the pattern (creation), protocol explains execution behavior
- **Focused and unambiguous:** A dedicated file clearly communicates "these are the execution rules"
- **SKILL.md stays lean:** Modes and quality checklist only, no inline rules or anti-patterns
- **Template is critical:** The inline protocol in the template is what Claude sees during execution; the reference doc is authoritative source

**Key insight:** During plan execution, Claude's attention is on the plan file itself. Reference documents loaded at skill invocation may fade after many tool calls. This is why the template must embed the protocol inline.

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
- **Related:** planning-with-files validation loop protocol, skill-enhance inline protocol
