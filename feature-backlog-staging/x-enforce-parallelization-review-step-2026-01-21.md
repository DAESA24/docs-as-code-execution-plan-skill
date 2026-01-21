---
doc_type: feature-backlog-item
title: "Enforce Parallelization Review as Required Workflow Step"
created: 2026-01-21
status: implemented
approved_date: 2026-01-21
implemented_date: 2026-01-21
execution_plan: docs/archives/2026-01-21-enforce-parallelization-review-skill-enhancement-plan.md
feature_type_tags:
  - enhancement
  - documentation
  - ux
---

# Enforce Parallelization Review as Required Workflow Step

## Problem Statement

The "Post-Draft Parallelization Review" step in Mode 1 (Create Execution Plan) is currently marked as "optional," which causes Claude to skip the evaluation entirely rather than treating it as a required decision point with variable outcomes.

## Requirements

- Parallelization review MUST be evaluated for every plan (not skippable)
- Plans MUST document either parallelization annotations OR explicit "not viable" rationale
- Quality checklist MUST include parallelization assessment verification
- Template MUST include a placeholder for the parallelization assessment decision

### Observed Behavior

During the FFmpeg migration execution plan creation (2026-01-21), Claude:

1. Read the skill instructions including the parallelization review section
2. Skipped the evaluation entirely without documenting any decision
3. Generated a plan without parallelization annotations AND without any "not viable" justification
4. Only noticed the gap when Drew explicitly asked about sub-agent strategy

### Root Cause

The word "optional" in the heading signals to Claude that the entire step can be skipped, when the actual intent is:

- The **evaluation** is required
- The **outcome** varies (add annotations OR document "not viable")

### Impact

- Plans may miss parallelization opportunities that would improve execution efficiency
- Plans lack documentation of why parallelization was or wasn't used
- No audit trail for the decision, making it hard to revisit later
- Unused HTML comment blocks (parallelization placeholders) remain in plans, adding noise

## Proposed Changes

### Change 1: Remove "optional" from the heading

**Current (CLAUDE.md line ~45-50):**

```markdown
**4. Post-Draft Parallelization Review** (optional)
```

**Proposed:**

```markdown
**4. Post-Draft Parallelization Review**
```

### Change 2: Reframe as decision gate with required output

**Current instructions (paraphrased):**

```markdown
- Evaluate parallelization potential using decision criteria (see guide Pattern 7)
- If viable: [steps to add annotations]
- If not viable: remove all parallelization HTML comment blocks from the plan
```

**Proposed replacement:**

```markdown
**4. Post-Draft Parallelization Review**

Before finalizing the plan, evaluate parallelization potential:

1. Review Pattern 7 decision criteria in the guide
2. Assess: Are there 2+ independent tasks in any phase?
3. Document decision in the plan's Agent Execution Notes section:
   - If parallelizing: Add annotations per the guide
   - If not parallelizing: Add "**Parallelization Assessment:** Not viable - [brief reason]"
4. Remove unused HTML comment blocks from the plan

**Required output:** The plan must contain either parallelization annotations OR an explicit "not viable" note. Plans without this assessment are incomplete.
```

### Change 3: Add to Quality Checklist

**Current Quality Checklist (CLAUDE.md):**

```markdown
## Quality Checklist

Before executing any plan, verify:

- [ ] YAML front matter complete (title, created, status, agent, etc.)
- [ ] Execution Instructions section present
...
- [ ] (If parallel) Parallelizable steps have `**Dependencies:**` annotation
- [ ] (If parallel) Parallelizable steps have `**Output Artifacts:**` section
- [ ] (If parallel) Parallelizable steps have `**Orchestrator Verification:**` commands
- [ ] (If parallel) Phases with parallel steps have parallelization summary
```

**Proposed addition (insert before the "(If parallel)" items):**

```markdown
- [ ] Parallelization assessment documented (either annotations added OR "not viable" note in Agent Execution Notes)
```

### Change 4: Update template Agent Execution Notes section

**Current template (docs-as-code-execution-plan-template.md):**

The Agent Execution Notes section doesn't have a placeholder for parallelization assessment.

**Proposed addition to template:**

Add to the Agent Execution Notes section:

```markdown
### Parallelization Assessment

<!-- Claude fills this in during plan creation -->
**Decision:** [Viable - see annotations above | Not viable]
**Rationale:** [Brief explanation of why parallelization is or isn't appropriate]
```

## Implementation Phases

### Phase 1: Update Guide Documentation

- File: `references/docs-as-code-guide.md`
- If Pattern 7 references "optional", update framing to match new required decision gate approach

### Phase 2: Update Template

- File: `references/docs-as-code-execution-plan-template.md`
- Add Parallelization Assessment subsection to Agent Execution Notes

### Phase 3: Update Skill Definition

- File: `SKILL.md`
- Remove "(optional)" from Post-Draft Parallelization Review heading
- Rewrite step as decision gate with required output
- Add checkbox to Quality Checklist

## Files to Modify

1. **`SKILL.md`** (skill definition with workflow instructions)
   - Remove "(optional)" from Post-Draft Parallelization Review heading
   - Rewrite the step as a decision gate with required output
   - Add checkbox to Quality Checklist

2. **`references/docs-as-code-execution-plan-template.md`**
   - Add Parallelization Assessment subsection to Agent Execution Notes

3. **`references/docs-as-code-guide.md`** (if exists and references the workflow)
   - Update any references to the parallelization review step to match new framing

## Acceptance Criteria

- [ ] The word "optional" does not appear in the parallelization review step
- [ ] The step explicitly states that plans without assessment are incomplete
- [ ] Quality Checklist includes parallelization assessment checkbox
- [ ] Template includes Parallelization Assessment placeholder in Agent Execution Notes
- [ ] Test: Generate a new execution plan and verify Claude documents the parallelization decision

## Context and Evidence

### Session where this was discovered

- **Date:** 2026-01-21
- **Task:** FFmpeg migration to pbc-media-ingestion
- **Plan generated:** `pbc-media-ingestion-dev/docs/2025-01-21-ffmpeg-migration-execution-plan.md`

### Claude's self-assessment of why it skipped the step

> "Honestly, I didn't consciously skip it - I just didn't notice it as a required step. The skill instructions list it as '(optional)' which caused me to treat it as skippable rather than as a decision point that requires explicit evaluation."

### Parallelization assessment that should have been documented

For the FFmpeg migration plan, the correct assessment was:

> **Parallelization Assessment:** Not viable
>
> **Rationale:** Phases have tight sequential dependencies (create dirs → copy files → create docs → update PATH → test). Within phases, potential parallel work (copying 3 executables, creating 3 doc files) is I/O-bound with trivial benefit that doesn't justify orchestration overhead.

This assessment was only produced when Drew explicitly asked about it after the plan was generated.

---

- **Document Status:** Captured
- **Last Updated:** 2026-01-21
- **Origin:** FFmpeg migration session feedback
