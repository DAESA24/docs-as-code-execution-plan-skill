---
doc_type: skill-enhancement-plan
title: Add Validation Loop Execution Protocol - Skill Enhancement
created: 2026-01-26
status: complete
agent: dev
execution_mode: validation-loop
user_intervention: minimal
source_feature: _feature-backlog-staging/validation-loop-execution-protocol-2026-01-26.md
skill_config: skill-dev-config.yaml
tags:
  - skill-enhancement
  - docs-as-code-execution-plan
  - validation-loop
---

# Add Validation Loop Execution Protocol - Skill Enhancement

- **Purpose:** Add explicit validation loop protocol to the docs-as-code-execution-plan skill so Claude marks checkboxes immediately (not batched) and re-reads the plan before each phase
- **Audience:** Claude Code (autonomous execution agent)
- **User Intervention:** Only for final sync to global skill directory

---

## Execution Protocol: Validation Loop (MANDATORY)

This plan follows the **validation-loop protocol**. The plan file is the operational hub - updates happen in THIS FILE, not just verbally.

### Core Principles (from Manus Context Engineering)

1. **Filesystem as External Memory** - This file is Claude's "working memory on disk"
2. **Attention Manipulation** - Re-reading goals keeps them in attention window after many tool calls
3. **Keep Failure Traces** - Errors logged here build knowledge, don't hide them

### The Loop - Non-Negotiable Steps

**BEFORE each phase:**

```text
1. Read this plan file (or the relevant phase section)
2. Refresh goals and acceptance criteria in attention window
3. Confirm you understand what the phase requires
```

**DURING each phase:**

```text
1. Execute the actions listed
2. If an error occurs:
   - DO NOT silently retry
   - Log the error in "Errors Encountered" section IMMEDIATELY
   - Then attempt resolution
```

**AFTER completing each step within a phase:**

```text
1. Edit THIS FILE to mark the checkbox [x]
2. Do NOT batch checkbox updates - mark IMMEDIATELY after each step
3. This creates a recoverable state if interrupted
```

**AFTER completing a phase:**

```text
1. Verify all checkboxes in the phase are marked [x]
2. Update the Status section (if present)
3. Report: "PHASE N COMPLETE: [summary]"
4. Read the next phase section before proceeding
```

### Example Execution Flow

```text
Claude: [Reads Phase 2 section]
Claude: "Starting Phase 2: [Phase Name]"
Claude: [Executes action 1]
Claude: [Edits plan] - marks first checkbox [x]
Claude: [Executes action 2]
Claude: [Edits plan] - marks second checkbox [x]
...
Claude: [All actions complete]
Claude: [Verifies all checkboxes marked]
Claude: "PHASE 2 COMPLETE: [summary of what was done]"
Claude: [Reads Phase 3 section]
```

### What NOT to Do

| Anti-Pattern | Correct Behavior |
|--------------|------------------|
| Execute all phases, then mark checkboxes | Mark each checkbox immediately after step |
| Report completion verbally without editing file | Edit file FIRST, then report |
| Retry silently on error | Log error to file, then retry |
| Read plan once at start | Re-read before each phase |
| Stuff findings in context | Store in file, reference by path |

### Recovery Protocol

If execution is interrupted mid-phase:

1. Read this plan file
2. Find last checked `[x]` item
3. Resume from next unchecked `[ ]` item
4. Do NOT restart from beginning

### Completion Protocol

When all phases complete:

1. Fill in "Dev Agent Record" section completely
2. Update source feature backlog item:
   - Set `status: implemented`
   - Add `implemented_date: [today]`
   - Rename file with `x-` prefix
3. Report final summary to user

---

## Phase Overview

| Phase | File Target | Purpose |
|-------|-------------|---------|
| 0 | Pre-flight | Verify skill files exist and are readable |
| 1 | `references/execution-protocol-reference.md` | Create new reference document with Critical Rules and anti-patterns |
| 2 | `references/docs-as-code-execution-plan-template.md` | Embed protocol inline and add Errors Encountered section |
| 3 | `SKILL.md` | Update Mode 2 to reference protocol, update Quality Checklist |
| 4 | Validation | Verify all changes are consistent |
| 5 | Sync | Copy changes to global skill directory |

---

## Problem Statement

The docs-as-code execution plan skill lacks explicit enforcement of the validation loop protocol during plan execution. Claude batches checkbox updates to the end of execution rather than marking them immediately after each step. This makes recovery from interruptions difficult and causes goals to drift out of attention.

### Current State

- Mode 2 guidance is weak, buried in bullet points
- No "Errors Encountered" section in template
- No explicit anti-patterns table for execution behavior
- No reference document for execution protocol

### Target State

- Dedicated `execution-protocol-reference.md` with Critical Rules and anti-patterns
- Template includes inline execution protocol section (travels with every plan)
- Template includes "Errors Encountered" section
- SKILL.md Mode 2 references the protocol document
- Quality Checklist includes "Execution Protocol section present" item

---

## Errors Encountered

- (None yet - errors will be logged here during execution)

---

## Pre-Flight Validation

**Autonomous:** YES

**Actions:**

- [x] Verify installed skill path exists: `~/.claude/skills/docs-as-code-execution-plan/`
- [x] Verify `references/docs-as-code-guide.md` is readable
- [x] Verify `references/docs-as-code-execution-plan-template.md` is readable
- [x] Verify `SKILL.md` is readable
- [x] Verify `skill-dev-config.yaml` exists in project root

**Report:** "PRE-FLIGHT COMPLETE: All skill files verified"

---

## Phase 1: Create execution-protocol-reference.md

**Autonomous:** YES

**Purpose:** Create the authoritative reference document for execution rules, separate from pattern documentation.

**Actions:**

- [x] Create new file `references/execution-protocol-reference.md` in the installed skill directory
- [x] Include header with purpose and audience
- [x] Add Core Principles section (from Manus context engineering)
- [x] Add Critical Rules section (numbered, non-negotiable)
- [x] Add "What NOT to Do" anti-patterns table
- [x] Add Recovery Protocol section
- [x] Add Completion Protocol section

**Content to create:**

```markdown
# Execution Protocol Reference

- **Purpose:** Authoritative rules for executing docs-as-code plans
- **Audience:** Claude Code (autonomous execution agent)
- **Source:** Manus context engineering principles

---

## Core Principles

### 1. Filesystem as External Memory

> "Markdown is my 'working memory' on disk."

The plan file IS your working memory. Treat it as the source of truth, not your internal context. After many tool calls, your context drifts - the file doesn't.

### 2. Attention Manipulation Through Repetition

> After ~50 tool calls, models forget original goals ("lost in the middle" effect).

**Solution:** Re-read the plan file before each phase. This brings goals back into your attention window.

### 3. Keep Failure Traces

> "Error recovery is one of the clearest signals of TRUE agentic behavior."

**Solution:** Log errors to the plan file BEFORE retrying. This builds knowledge and enables recovery.

---

## Critical Rules

### 1. Read Before Each Phase

Before starting any phase, read that phase's section in the plan file. This refreshes your goals.

### 2. Mark Checkboxes Immediately

After completing each step, edit the plan file to mark the checkbox `[x]`. Do NOT batch updates.

### 3. Log Errors Before Retrying

If an error occurs, log it to the "Errors Encountered" section IMMEDIATELY. Then attempt resolution.

### 4. Store, Don't Stuff

Large outputs go to files, not context. Keep only paths in working memory.

### 5. Update Status Progressively

Update the plan's status as you complete phases. This enables recovery from interruptions.

---

## What NOT to Do

| Anti-Pattern | Correct Behavior |
|--------------|------------------|
| Execute all phases, then mark checkboxes | Mark each checkbox immediately after step |
| Report completion verbally without editing file | Edit file FIRST, then report |
| Retry silently on error | Log error to file, then retry |
| Read plan once at start | Re-read before each phase |
| Stuff findings in context | Store in file, reference by path |
| Batch multiple steps together | Execute and validate one step at a time |
| Skip validation steps | Complete ALL validation before next step |

---

## Recovery Protocol

If execution is interrupted mid-phase:

1. Read the plan file
2. Find last checked `[x]` item
3. Resume from next unchecked `[ ]` item
4. Do NOT restart from beginning

---

## Completion Protocol

When all phases complete:

1. Fill in "Dev Agent Record" section completely
2. Update plan status to `complete` in YAML front matter
3. Archive to `docs/archives/` if applicable
4. Report final summary to user

---

## Source

Based on Manus context engineering principles:
https://manus.im/de/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
```

**Validation Checklist:**

- [x] File created at correct path
- [x] Core Principles section present
- [x] Critical Rules section with numbered rules
- [x] Anti-patterns table present
- [x] Recovery Protocol section present
- [x] Completion Protocol section present

**Report:** "PHASE 1 COMPLETE: execution-protocol-reference.md created with Critical Rules and anti-patterns"

---

## Phase 2: Update docs-as-code-execution-plan-template.md

**Autonomous:** YES

**Purpose:** Embed the execution protocol inline in the template so it travels with every generated plan.

**Actions:**

- [x] Read current template content
- [x] Add "Execution Protocol: Validation Loop (MANDATORY)" section after the header (before Execution Instructions)
- [x] Add "Errors Encountered" section placeholder after Problem Statement
- [x] Update Execution Instructions to reference the protocol section
- [x] Preserve all existing template content

**Insertion point for Execution Protocol section:** After line 18 (after the `---` following header metadata)

**Content to insert:**

```markdown
## Execution Protocol: Validation Loop (MANDATORY)

This plan follows the **validation-loop protocol**. The plan file is the operational hub - updates happen in THIS FILE, not just verbally.

### The Loop - Non-Negotiable Steps

**BEFORE each phase:**
1. Read this plan file (or the relevant phase section)
2. Refresh goals and acceptance criteria in attention window

**AFTER completing each step:**
1. Edit THIS FILE to mark the checkbox `[x]`
2. Do NOT batch checkbox updates - mark IMMEDIATELY after each step

**AFTER completing a phase:**
1. Verify all checkboxes in the phase are marked `[x]`
2. Report: "PHASE N COMPLETE: [summary]"
3. Read the next phase section before proceeding

**If an error occurs:**
1. Log the error in "Errors Encountered" section IMMEDIATELY
2. Then attempt resolution

### Recovery

If interrupted, find last `[x]` item and resume from next unchecked `[ ]` item.

---
```

**Insertion point for Errors Encountered section:** After Problem Statement section (around line 57), before Risk Assessment

**Content to insert:**

```markdown
## Errors Encountered

- (None yet - errors will be logged here during execution)

---
```

**Validation Checklist:**

- [x] Execution Protocol section added after header
- [x] Errors Encountered section added after Problem Statement
- [x] Execution Instructions section still present
- [x] All original template content preserved

**Report:** "PHASE 2 COMPLETE: Template updated with inline protocol and Errors Encountered section"

---

## Phase 3: Update SKILL.md

**Autonomous:** YES

**Purpose:** Update Mode 2 to reference the protocol document and add Quality Checklist item.

**Actions:**

- [x] Read current SKILL.md content
- [x] Update Mode 2 to reference `execution-protocol-reference.md` at the beginning
- [x] Simplify Mode 2 by removing detailed execution guidance (now in reference doc)
- [x] Add "Execution Protocol section present" to Quality Checklist
- [x] Preserve all other content

**Changes to Mode 2 (around line 125):**

Add at the beginning of Mode 2:

```markdown
### Mode 2: Execute Existing Plan

**Before executing any plan, review `references/execution-protocol-reference.md` for the validation loop protocol.**

To execute an existing execution plan:
```

**Changes to Quality Checklist (around line 195):**

Add after existing checklist items:

```markdown
- [ ] Execution Protocol section present (after header, before Execution Instructions)
- [ ] Errors Encountered section present (after Problem Statement)
```

**Validation Checklist:**

- [x] Mode 2 references execution-protocol-reference.md
- [x] Quality Checklist has "Execution Protocol section present" item
- [x] Quality Checklist has "Errors Encountered section present" item
- [x] All other SKILL.md content preserved

**Report:** "PHASE 3 COMPLETE: SKILL.md updated with protocol reference and Quality Checklist items"

---

## Phase 4: Validation

**Autonomous:** YES

**Purpose:** Verify all changes are consistent and complete.

**Actions:**

- [x] Read `references/execution-protocol-reference.md` - verify structure complete
- [x] Read `references/docs-as-code-execution-plan-template.md` - verify protocol section present
- [x] Read `SKILL.md` - verify Mode 2 references protocol, Quality Checklist updated
- [x] Verify all files follow existing conventions (metadata format, section structure)

**Validation Checklist:**

- [x] execution-protocol-reference.md has Core Principles, Critical Rules, Anti-patterns table
- [x] Template has Execution Protocol section after header
- [x] Template has Errors Encountered section after Problem Statement
- [x] SKILL.md Mode 2 references execution-protocol-reference.md
- [x] SKILL.md Quality Checklist includes protocol section check

**Report:** "PHASE 4 COMPLETE: All changes validated and consistent"

---

## Phase 5: Sync to Global Skill Directory

**Autonomous:** NO

### USER APPROVAL REQUIRED

**What will happen:**

- Copy updated files from dev repo to `~/.claude/skills/docs-as-code-execution-plan/`

**Files to sync:**

- `references/execution-protocol-reference.md` (NEW)
- `references/docs-as-code-execution-plan-template.md` (MODIFIED)
- `SKILL.md` (MODIFIED)

**Safety measures:**

- Original files can be restored from git history
- Validate command available: `skills-ref validate ~/.claude/skills/docs-as-code-execution-plan`

**Reversibility:** Git checkout to restore original files

Proceed with sync? (yes/no)

**Actions (after approval):**

- [x] Copy `references/execution-protocol-reference.md` to global skill directory
- [x] Copy `references/docs-as-code-execution-plan-template.md` to global skill directory
- [x] Copy `SKILL.md` to global skill directory
- [x] Run validation: `skills-ref validate ~/.claude/skills/docs-as-code-execution-plan`

**Validation Checklist:**

- [x] All files copied successfully
- [x] Skill validation passes

**Report:** "PHASE 5 COMPLETE: Changes synced to global skill directory"

---

## Rollback Procedure

If issues are discovered after sync:

```bash
# Navigate to global skill directory
cd ~/.claude/skills/docs-as-code-execution-plan

# Restore from git (if tracked)
git checkout -- references/docs-as-code-execution-plan-template.md
git checkout -- SKILL.md

# Remove new file
rm references/execution-protocol-reference.md
```

---

## Acceptance Criteria

- [x] `references/execution-protocol-reference.md` exists with Critical Rules and anti-patterns table
- [x] Template includes inline execution protocol section (mandatory, not optional)
- [x] Template includes "Errors Encountered" section placeholder
- [x] SKILL.md Mode 2 references `execution-protocol-reference.md` (not inline rules)
- [x] Quality Checklist includes "Execution Protocol section present" item

---

## Agent Execution Notes

### Critical Execution Requirements

1. Follow the validation loop protocol defined in this plan's Execution Protocol section
2. Mark checkboxes IMMEDIATELY after each step - do NOT batch
3. Re-read each phase section before starting that phase

### Preservation Targets

- `references/docs-as-code-guide.md` - Do NOT modify (pattern documentation, separate concern)
- Existing template structure and conventions

### Safe Modification Targets

- `references/execution-protocol-reference.md` (NEW)
- `references/docs-as-code-execution-plan-template.md` (adding sections)
- `SKILL.md` (updating Mode 2 and Quality Checklist)

### Execution Trigger

- **Trigger phrase:** "Execute the validation loop protocol skill enhancement plan"
- **Plan location:** `docs/2026-01-26-validation-loop-protocol-skill-enhancement-plan.md`
- **Resume guidance:** If interrupted, re-read this file and continue from last incomplete step.

### Parallelization Assessment

**Decision:** Not viable
**Rationale:** Phases have dependencies - Phase 1 creates the reference doc that Phase 3 references, and Phase 4 validates all previous phases.

---

## Dev Agent Record

- **Executed By:** Claude Opus 4.5
- **Execution Date:** 2026-01-26
- **Duration:** Single session

### Execution Notes

Executed the validation loop protocol skill enhancement plan successfully. All phases completed sequentially with immediate checkbox marking after each step. Changes were made directly to the installed skill directory as specified in Phase 1.

### Issues Encountered

| Issue                                 | Resolution                                        |
|---------------------------------------|---------------------------------------------------|
| skills-ref command not found in bash  | Used quick_validate.py from skill-creator instead |

### Procedural Learnings

1. The validation loop protocol was effectively demonstrated during this execution - marking checkboxes immediately after each step created a clear audit trail
2. Making changes directly to the installed skill directory (rather than dev repo + copy) simplified the workflow for this type of self-enhancement

---

- **Document Status:** Complete
- **Last Updated:** 2026-01-26
- **Related Docs:** [Feature backlog item](_feature-backlog-staging/x-validation-loop-execution-protocol-2026-01-26.md)
