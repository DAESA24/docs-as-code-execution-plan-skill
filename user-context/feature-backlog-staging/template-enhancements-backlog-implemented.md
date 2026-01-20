# Docs-as-Code Execution Plan Skill - Template Enhancements Backlog

- **Created:** 2026-01-20
- **Source:** Analysis of Claude Memory Deletion Plan vs. current template
- **Audience:** Claude Code (executing agent)
- **Status:** Complete
- **Implemented:** 2026-01-20
- **Reference:** [2026-01-20-skill-template-enhancements-execution-plan.md](../../docs/archives/2026-01-20-skill-template-enhancements-execution-plan.md)

---

## Document Architecture

### Skill File Inventory

| File | Path | Purpose |
|------|------|---------|
| `SKILL.md` | `docs-as-code-execution-plan/SKILL.md` | Entry point - when/how to use, workflow modes, quality checklist |
| `template` | `docs-as-code-execution-plan/references/docs-as-code-execution-plan-template.md` | Copy-paste template for new plans |
| `guide` | `docs-as-code-execution-plan/references/docs-as-code-guide.md` | Authoritative source - principles, patterns, anti-patterns |

### Document Dependencies

```
guide (authoritative source)
  ↓
template (implements patterns from guide)
  ↓
SKILL.md (summarizes and references both)
```

### Required Update Order

1. **First:** `docs-as-code-guide.md` - Add patterns/principles
2. **Second:** `docs-as-code-execution-plan-template.md` - Add structural sections
3. **Third:** `SKILL.md` - Update Quality Checklist and workflow references

**Rationale:** Guide is source of truth. Template implements it. SKILL.md summarizes. Updating in this order prevents inconsistencies.

---

## Feature Specifications

### Feature 1: Per-Step Report Markers

**Problem:** No standardized format for step completion reporting.

**Solution:** Add `**Report:**` field to each step.

**Format:**
```markdown
**Report:** "STEP X.Y COMPLETE: <summary with key metrics>"
```

**Example:**
```markdown
**Report:** "STEP 0.1 COMPLETE: Baseline snapshots created (5 snapshots)"
```

**Files to Update:**

| File | Action |
|------|--------|
| guide | Add to "Critical Patterns" section as new pattern |
| template | Add `**Report:**` field after `**Success Criteria:**` in step template |
| SKILL.md | Add to Quality Checklist: "Each step has Report marker" |

---

### Feature 2: Execution Instructions Block

**Problem:** Template assumes agent knows how to execute. No explicit rules.

**Solution:** Add "Execution Instructions" section in plan header.

**Content to Add:**
```markdown
## Execution Instructions

- Execute steps SEQUENTIALLY in exact order listed
- Complete ALL validation substeps before proceeding to next step
- If ANY validation fails, STOP immediately and report failure
- Do NOT skip validation steps
- Do NOT batch multiple steps together
- Report results after EACH step completion
```

**Files to Update:**

| File | Action |
|------|--------|
| guide | Add as new principle (6th principle): "Explicit Execution Rules" |
| template | Add `## Execution Instructions` section after front matter, before Test Plan Summary |
| SKILL.md | Add to "Mode 2: Execute Existing Plan" - reference execution instructions |

---

### Feature 3: Agent Execution Notes Section

**Problem:** No dedicated section for agent-specific context (what to preserve, what's safe to modify, trigger phrases).

**Solution:** Add "Agent Execution Notes" section at end of plan.

**Structure:**
```markdown
## Agent Execution Notes

### Critical Execution Requirements
[Numbered list of rules]

### Preservation Targets
[Files/directories that must NOT change]

### Safe Modification Targets
[Files/directories the plan may alter]

### Execution Trigger
[Trigger phrase and file path for invoking plan]
```

**Files to Update:**

| File | Action |
|------|--------|
| guide | Add as new pattern in "Critical Patterns" section |
| template | Add `## Agent Execution Notes` section before `## Dev Agent Record` |
| SKILL.md | Add to Quality Checklist: "Agent Execution Notes section present" |

---

### Feature 4: Detailed Validation Substeps with In-Place Checkbox Marking

**Problem:** Success criteria are typically one line. Complex steps need multiple verification points. Additionally, when execution spans multiple sessions, a new agent instance has no persistent record of what was actually validated vs. what was skipped.

**Solution:** Expand Success Criteria to support multiple checkpoints using checkboxes, AND require the agent to edit the plan file to mark checkboxes as complete during execution.

**Why In-Place Marking Matters:**

- Execution often spans multiple chat sessions
- A new agent instance needs visual indicators of completed work
- Prevents re-validation of already-verified items
- Creates an audit trail in the plan document itself
- Distinguishes between "reported as done" vs. "actually marked done in file"

**Enhanced Format:**
```markdown
**Validation Checklist:**

- [ ] Primary condition verified
- [ ] Secondary condition verified
- [ ] Edge case handled
- [ ] No side effects on preserved targets

**Instruction:** Mark each checkbox in this file as validation is confirmed.
```

**After Execution (edited in-place):**
```markdown
**Validation Checklist:**

- [x] Primary condition verified
- [x] Secondary condition verified
- [x] Edge case handled
- [x] No side effects on preserved targets

**Instruction:** Mark each checkbox in this file as validation is confirmed.
```

**Alternative (for simpler steps where persistent tracking is not needed):**
```markdown
**Validation:**

- Verify condition A
- Verify condition B
- Display output to confirm
```

**Key Distinction:**

- Use `- [ ]` checkboxes when the step needs persistent tracking (mark in file)
- Use `-` bullet points when verification is transient (report in response only)

**Execution Instructions to Add:**
The Execution Instructions section (Feature 2) should include:
```markdown
- When steps contain `- [ ]` checkboxes, EDIT THIS FILE to mark them `- [x]` as each item is verified
- This creates a persistent record for multi-session execution
```

**Files to Update:**

| File | Action |
|------|--------|
| guide | Expand "Explicit Success Criteria" section to show multi-point validation AND explain in-place marking requirement |
| template | Change `**Success Criteria:**` to `**Validation Checklist:**` with checkboxes and marking instruction |
| SKILL.md | Update Quality Checklist: "Validation checklists use checkboxes for persistent tracking" |
| SKILL.md | Add to "Mode 2: Execute Existing Plan": "Edit plan file to mark checkboxes as items are validated" |

---

### Feature 5: Expected Results Block

**Problem:** Success criteria describe what should happen, but not specific expected values.

**Solution:** Add optional `**Expected Results:**` block for steps with measurable outcomes.

**Format:**
```markdown
**Expected Results:**

- File count: ~178 (may vary)
- Directory exists: YES
- Exit code: 0
```

**Guidance:** Use when:
- Operations produce countable outputs
- Specific values can be predicted
- Partial success needs detection

**Files to Update:**

| File | Action |
|------|--------|
| guide | Add to "Built-In Validation" pattern as optional enhancement |
| template | Add `**Expected Results:**` (optional) after Actions, before Success Criteria |
| SKILL.md | No change needed |

---

### Feature 6: Phase Validation Steps

**Problem:** No consolidated validation at phase boundaries. Issues may be caught only at final validation.

**Solution:** Add dedicated validation step at end of each phase.

**Pattern:**
```markdown
### STEP X.N: Validate Phase X Completion

**Autonomous:** YES

**Actions:**
[Compare current state to baseline/expected state]

**Expected Results:**

- [List what should be true after phase completes]

**Report:** "STEP X.N COMPLETE: Phase X validation passed"
```

**Files to Update:**

| File | Action |
|------|--------|
| guide | Add "Phase Boundary Validation" to patterns section |
| template | Add comment in template: `<!-- Add Phase Validation step at end of each phase -->` |
| SKILL.md | Add to "Mode 1: Create Execution Plan" - "Include phase validation step at end of each phase" |

---

### Feature 7: Session Context / Future Sessions

**Problem:** Plans may span sessions. New agent instance needs to know how to pick up the plan.

**Solution:** Include in Agent Execution Notes or as separate brief section.

**Content:**
```markdown
### Execution Trigger

**Trigger phrase:** "execute the [plan name]"
**Plan location:** `path/to/plan.md`
**Resume guidance:** If interrupted, re-read this file and continue from last incomplete step.
```

**Files to Update:**

| File | Action |
|------|--------|
| guide | Brief mention in "LLM-Optimized Structure" section |
| template | Include in Agent Execution Notes section (Feature 3) |
| SKILL.md | No change needed (covered by template) |

---

## Implementation Summary

### Changes by File

**docs-as-code-guide.md:**

1. Add 6th principle: "Explicit Execution Rules"
2. Add pattern: "Per-Step Report Markers"
3. Add pattern: "Agent Execution Notes"
4. Add pattern: "Phase Boundary Validation"
5. Expand: "Explicit Success Criteria" to show multi-point validation
6. Expand: "Built-In Validation" to include Expected Results
7. Brief addition to "LLM-Optimized Structure" for session context

**docs-as-code-execution-plan-template.md:**

1. Add section: `## Execution Instructions` (after front matter)
2. Add field: `**Expected Results:**` (optional, per step)
3. Add field: `**Report:**` (per step)
4. Expand: `**Success Criteria:**` to checkbox list format
5. Add section: `## Agent Execution Notes` (before Dev Agent Record)
6. Add comment: Phase validation step guidance

**SKILL.md:**

1. Update: Quality Checklist with new items
2. Update: "Mode 1: Create Execution Plan" workflow
3. Update: "Mode 2: Execute Existing Plan" to reference execution instructions

### Priority Order

| Priority | Feature | Rationale |
|----------|---------|-----------|
| High | 2. Execution Instructions | Prevents common execution errors |
| High | 3. Agent Execution Notes | Critical for safe execution |
| High | 1. Per-Step Report Markers | Enables progress tracking |
| Medium | 4. Detailed Validation | Catches edge cases |
| Medium | 6. Phase Validation Steps | Catches phase-level issues |
| Medium | 5. Expected Results | Makes validation objective |
| Low | 7. Session Context | Nice-to-have for multi-session |

---

## Execution Guidance

When implementing these features:

1. Read this entire document first
2. Update files in dependency order: guide -> template -> SKILL.md
3. After each file update, verify no broken references
4. Test by creating a sample plan using updated template
5. Verify sample plan contains all new sections

---

- **Document Status:** Complete
- **Last Updated:** 2026-01-20
- **Implementation Notes:** All 7 features implemented across guide, template, and SKILL.md. Skill validation passed.
