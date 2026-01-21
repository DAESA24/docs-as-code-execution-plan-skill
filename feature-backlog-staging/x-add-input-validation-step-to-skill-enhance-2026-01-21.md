---
doc_type: feature-backlog-item
title: "Add Input Validation Step to Skill-Enhance Command"
created: 2026-01-21
status: implemented
approved_date: 2026-01-21
implemented_date: 2026-01-21
execution_plan: docs/archives/2026-01-21-input-validation-skill-enhancement-plan.md
feature_type_tags:
  - enhancement
  - ux
  - documentation
---

# Add Input Validation Step to Skill-Enhance Command

## Problem Statement

The skill-enhance command accepts feature backlog items as input and transforms them into execution plans. However, the command proceeds directly to plan generation without first validating that the backlog item contains the sections it expects to extract. This creates a mismatch between implicit expectations and actual inputs, causing suboptimal execution plans.

### Observed Behavior

During the "Enforce Parallelization Review" feature backlog session (2026-01-21), Drew manually asked Claude to review the backlog item against skill-enhance requirements before running the command. This revealed four structural gaps:

1. **No explicit Requirements section** - Command expects "Core requirements" but item had only "Proposed Changes" and "Acceptance Criteria"
2. **No Implementation Phases section** - Command expects "Proposed implementation phases" but item listed changes without phase organization
3. **Incorrect file references** - Item referenced `CLAUDE.md` as skill instructions, but actual file is `SKILL.md`
4. **No implementation order** - Command expects ordering guidance for the fixed update sequence (guide → template → SKILL.md → CLAUDE.md)

### Root Cause

The skill-enhance command's Step 1 ("Validate Feature Backlog Item") only checks frontmatter validity (`doc_type`, `status`), not structural completeness. The "Extract key information" substep lists expected sections but doesn't validate their presence or suggest adding them.

### Impact

- Execution plans generated from incomplete inputs require more inference/guessing
- File reference errors propagate into plan steps, causing confusion during execution
- Missing phase organization means Claude must infer ordering rather than following explicit guidance
- No feedback loop to improve backlog item quality before plan generation

## Requirements

- Skill-enhance MUST validate backlog item structure before proceeding to plan generation
- Validation MUST check for: Requirements section, Implementation Phases section, correct file references
- If gaps are found, Claude MUST present them and offer to improve the backlog item
- User MUST be able to skip validation and proceed with current item if desired
- Validation step MUST be lightweight (reading two files, comparing structure)

## Proposed Changes

### Change 1: Add "Step 0: Input Quality Review" before current Step 1

Insert a new step at the beginning of the Instructions section:

```markdown
### Step 0: Input Quality Review (Before Validation)

Before validating frontmatter, review the backlog item's structural completeness:

1. **Read the backlog item** at the provided path
2. **Check for expected sections:**
   - [ ] `## Requirements` - Explicit list of MUST/SHOULD requirements
   - [ ] `## Implementation Phases` - Phases with file targets and ordering
   - [ ] File references point to correct skill files (`SKILL.md` not `CLAUDE.md`)
3. **If gaps found:**
   - Present a summary: "This backlog item is missing: [list]"
   - Offer: "Should I suggest additions to improve it as an input, or proceed as-is?"
   - If user wants improvements: suggest specific additions, wait for approval, apply edits
4. **If complete:** Proceed to Step 1
```

### Change 2: Update Step 1 to reference the new validation

Current Step 1 title:
```markdown
### Step 1: Validate Feature Backlog Item
```

Update to clarify scope:
```markdown
### Step 1: Validate Feature Backlog Item (Frontmatter)
```

Add note at start:
```markdown
**Note:** Structural completeness was verified in Step 0. This step validates frontmatter only.
```

### Change 3: Add expected sections checklist to Prerequisites

Current Prerequisites section:
```markdown
## Prerequisites

- Feature backlog item must exist and have `status: approved` (or be ready for approval)
- Feature backlog item should be sufficiently detailed (problem statement, requirements, proposed implementation)
```

Add explicit checklist:
```markdown
## Prerequisites

- Feature backlog item must exist and have `status: approved` (or be ready for approval)
- Feature backlog item should be sufficiently detailed:
  - `## Problem Statement` - What issue this solves
  - `## Requirements` - Explicit MUST/SHOULD list (not just prose)
  - `## Implementation Phases` - Ordered phases with file targets
  - `## Acceptance Criteria` - Verifiable completion conditions
  - Correct file references (`SKILL.md` for skill definition, not `CLAUDE.md`)
```

## Implementation Phases

### Phase 1: Update Command Documentation

- File: `.claude/commands/skill-enhance.md`
- Add Step 0: Input Quality Review
- Update Step 1 title and add note
- Expand Prerequisites section

### Phase 2: Update Feature Backlog CLAUDE.md (if exists)

- File: `feature-backlog-staging/CLAUDE.md`
- Add guidance for creating skill-enhance-ready backlog items
- Reference the expected sections checklist

## Files to Modify

1. **`.claude/commands/skill-enhance.md`**
   - Add Step 0: Input Quality Review
   - Update Step 1 scope clarification
   - Expand Prerequisites with expected sections checklist

2. **`feature-backlog-staging/CLAUDE.md`** (if applicable)
   - Add "Creating Skill-Enhance-Ready Items" guidance section

## Acceptance Criteria

- [ ] Step 0 exists in skill-enhance command with structural validation checklist
- [ ] Prerequisites section lists expected backlog item sections
- [ ] Running skill-enhance on an incomplete item triggers review and improvement offer
- [ ] User can skip review and proceed with current item
- [ ] Test: Run skill-enhance on a minimal backlog item, verify it offers to improve structure

## Context and Evidence

### Source Insight Capture

- **Document:** `user-context/@insight-decision-captures/feature-backlog-review-step-capture-2026-01-21.md`
- **Key Pattern:** P.1 - Input Validation Before Complex Processing
- **Key Decision:** D.1 - Add Pre-Execution Review Step to Skill-Enhance Workflow

### Session Evidence

The manual review in the source session identified these specific gaps that would be caught by this validation:

| Expected Section | What Was Found | Gap Type |
|------------------|----------------|----------|
| `## Requirements` | Acceptance Criteria only | Missing section |
| `## Implementation Phases` | Proposed Changes (unordered) | Missing section |
| File: `SKILL.md` | Referenced `CLAUDE.md` | Incorrect reference |
| Implementation order | Not specified | Missing guidance |

### Rationale from Capture

> "A 2-minute review that improves input quality prevents compounding issues throughout execution plan generation. The review caught 4 gaps in this session that would have required correction later."

---

- **Document Status:** Captured
- **Last Updated:** 2026-01-21
- **Origin:** `user-context/@insight-decision-captures/feature-backlog-review-step-capture-2026-01-21.md`
