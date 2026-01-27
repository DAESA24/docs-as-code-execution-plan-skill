---
doc_type: feature-backlog-item
title: "Add Pre-Execution Git Safety Check"
created: 2026-01-26
status: captured
execution_order: 2
execution_sequence:
  - position: 1
    item: validation-loop-execution-protocol-2026-01-26.md
    status: implement first
  - position: 2
    item: pre-execution-git-safety-check-2026-01-26.md
    status: THIS ITEM
  - position: 3
    item: autonomous-execution-permissions-2026-01-26.md
    depends_on: this item
feature_type_tags:
  - enhancement
  - safety
  - execution
---

# Add Pre-Execution Git Safety Check

## Problem Statement

When executing a docs-as-code execution plan, there is no guarantee that a rollback point exists if something goes wrong. If the target directory has uncommitted changes or no recent commit, a failed execution could leave the codebase in a broken state with no easy recovery path.

This feature establishes a safety net that must be in place before granting broader autonomous execution permissions (see related: `autonomous-execution-permissions-2026-01-26.md`).

### Current State

- Plans execute without checking git status
- If execution fails mid-way, manual recovery is required
- No standardized rollback procedure tied to git
- User must remember to commit before execution (error-prone)

### Target State

- Pre-flight validation checks git working directory status
- If uncommitted changes exist, user is offered clear options (commit/stash/abort)
- Every execution has a known rollback point
- Rollback procedure can reference the pre-execution commit

## Requirements

The following requirements ensure a reliable rollback point exists before execution:

- **MUST** check if target directory is a git repository during pre-flight
- **MUST** check for uncommitted changes (staged and unstaged) in target paths
- **MUST** offer clear options if uncommitted changes are found:
  1. Create pre-execution commit (with standardized message)
  2. Stash changes (preserve separately)
  3. Abort execution (user handles manually)
- **MUST** document the rollback procedure referencing the pre-execution state
- **SHOULD** fail gracefully for non-git directories (warn but allow override)
- **SHOULD** record the pre-execution commit SHA in the plan's Dev Agent Record
- **SHOULD** scope the check to paths affected by the plan (not entire repo)

## Implementation Phases

Implementation follows the skill file update order defined in `skill-dev-config.yaml`:

### Phase 1: Update docs-as-code-guide.md

- **File:** `references/docs-as-code-guide.md`
- **Changes:**
  - Add new pattern: "Pattern N: Pre-Execution Safety Check"
  - Document the git status check logic
  - Document the commit/stash/abort decision flow
  - Explain the rollback guarantee this provides

### Phase 2: Update docs-as-code-execution-plan-template.md

- **File:** `references/docs-as-code-execution-plan-template.md`
- **Changes:**
  - Update Pre-Flight Validation section with git status check
  - Add bash script for checking uncommitted changes
  - Add decision flow for commit/stash/abort options
  - Update Rollback Procedure section to reference pre-execution commit
  - Add `pre_execution_commit` field placeholder to Dev Agent Record

### Phase 3: Update SKILL.md

- **File:** `SKILL.md`
- **Changes:**
  - Update Mode 2 (Execute Existing Plan) to include git safety check
  - Add documentation for the commit/stash/abort options
  - Update Quality Checklist to include "Pre-execution rollback point verified"
  - Document behavior for non-git directories

## Acceptance Criteria

- [ ] `docs-as-code-guide.md` contains pre-execution safety check pattern documentation
- [ ] Template pre-flight section includes git status check bash script
- [ ] Template includes commit/stash/abort decision flow
- [ ] Template rollback procedure references pre-execution commit
- [ ] Template Dev Agent Record includes `pre_execution_commit` field
- [ ] SKILL.md Mode 2 documents the git safety check requirement
- [ ] Quality Checklist includes rollback point verification
- [ ] Test: Execute plan in dirty working directory, verify commit option works
- [ ] Test: Execute plan in clean working directory, verify no prompts
- [ ] Test: Execute plan in non-git directory, verify graceful handling

## Technical Details

### Git Status Check Script

```bash
# Check if target directory is a git repo
if ! git rev-parse --git-dir > /dev/null 2>&1; then
    echo "WARNING: Not a git repository. No rollback point available."
    echo "Continue anyway? (yes/no)"
    # Wait for user response
    exit 0  # or exit 1 based on response
fi

# Check for uncommitted changes
if ! git diff --quiet HEAD 2>/dev/null || ! git diff --cached --quiet 2>/dev/null; then
    echo "WARNING: Uncommitted changes detected."
    echo ""
    echo "Options:"
    echo "  1. Create pre-execution commit"
    echo "  2. Stash changes"
    echo "  3. Abort execution"
    echo ""
    # Wait for user choice
fi

# Record current HEAD for rollback reference
PRE_EXEC_COMMIT=$(git rev-parse HEAD)
echo "Pre-execution commit: $PRE_EXEC_COMMIT"
```

### Standardized Commit Message

```
chore: pre-execution snapshot for [plan-name]

Automatic commit created before executing:
  [path/to/execution-plan.md]

To rollback: git reset --hard [this-commit-sha]
```

### Decision Flow

```
┌─────────────────────────────┐
│ Pre-Flight: Git Status Check │
└──────────────┬──────────────┘
               │
        Is git repo?
        /          \
      NO            YES
      │              │
  Warn user     Has uncommitted
  (override?)    changes?
      │          /      \
      │        NO        YES
      │         │         │
      │    Continue   Offer options:
      │         │     1. Commit
      │         │     2. Stash
      │         │     3. Abort
      │         │         │
      └─────────┴─────────┘
                │
         Record commit SHA
                │
         Continue execution
```

## Related Items

- **Enables:** `autonomous-execution-permissions-2026-01-26.md` (to be created)
  - This safety check is a prerequisite for granting broad `allowed-tools` permissions
  - Pattern: "Safety first, then freedom"

## Context and Evidence

### Origin

Discussion with Drew on 2026-01-26 about reducing execution friction while maintaining safety. Key insight: the current approval prompts are "security theater" since Drew approves without reviewing (the plan is already vetted). A git-based rollback point provides actual safety with less friction.

### Design Principle

> "Safety first, then freedom" - establish the rollback safety net before granting autonomous execution permissions.

---

- **Document Status:** Captured
- **Last Updated:** 2026-01-26
- **Related:** autonomous-execution-permissions (to be created after this is implemented)
