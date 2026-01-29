---
handoff_id: 2026-01-27-hook-based-logging-poc-general-handoff
type: general
created: 2026-01-27 21:30
status: ready_for_pickup
workspace: c:\Users\drewa\work\dev\skills-dev\docs-as-code-execution-plan-skill-dev
pickup_history: []
---

# Hook-Based Logging PoC Plan Creation

- **Created:** 2026-01-27 21:30
- **Purpose:** Create and execute PoC execution plans for deterministic hook-based logging feature
- **Status:** Handoff - Ready for pickup
- **Pickup Summary:** TBD

## Primary Request and Intent

Drew observed inconsistent logging behavior across three consecutive execution plan runs. The current instruction-based logging is non-deterministic - Claude can forget to log, stop mid-execution, or inconsistently request permissions. Drew wants to implement hook-based logging that is 100% deterministic.

The immediate next step is to create a PoC execution plan that Claude can follow to validate the hook-based approach before full implementation.

## Key Technical Concepts

- **Claude Code Hooks** - System-level event handlers that fire automatically on tool use
- **Skill-bundled hooks** - Hooks defined in SKILL.md YAML frontmatter (travel with the skill)
- **PostToolUse / PostToolUseFailure** - Hook events for successful/failed tool calls
- **Stop hook with `once: true`** - Fires once when Claude finishes responding
- **Marker file pattern** - `.claude-execution-log-path` file coordinates log location between hooks

## Files and Code Sections

**Feature Backlog Item (fully documented):**
`_feature-backlog-staging/deterministic-hook-based-logging-2026-01-27.md`

**Insight Capture (requirements patterns):**
`user-context/@insight-decision-captures/2026-01-27-requirements-and-open-questions-patterns.md`

**Proposed hook configuration (from backlog item):**

```yaml
---
name: docs-as-code-execution-plan
description: ...
hooks:
  PostToolUse:
    - matcher: "Bash|Edit|Write|Read"
      hooks:
        - type: command
          command: "$SKILL_DIR/scripts/log-tool-event.sh"
  Stop:
    - hooks:
        - type: command
          command: "$SKILL_DIR/scripts/finalize-execution-log.sh"
          once: true
---
```

**Draft hook script (log-tool-event.sh) from backlog:**

```bash
#!/usr/bin/env bash
set -euo pipefail

MARKER_FILE=".claude-execution-log-path"
if [[ ! -f "$MARKER_FILE" ]]; then
    exit 0  # No active execution - skip silently
fi

LOG_FILE=$(cat "$MARKER_FILE")
if [[ ! -f "$LOG_FILE" ]]; then
    exit 0
fi

INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name // "unknown"')
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# Build and append event based on tool type
# ... (full script in backlog item)
```

## Problem Solving

**This session accomplished:**

1. Researched Claude Code hooks documentation
2. Created comprehensive feature backlog item with:
   - Problem statement with evidence from 3 production runs
   - Requirements tables with justification columns
   - Proposed architecture with hook scripts
   - Implementation phases (0-3)
3. Refined requirements approach:
   - Converted all SHOULD to MUST or removed
   - Added justification column to all requirements
4. Restructured open questions into:
   - Technical Validations (TV1-TV6) - factual, answered by PoC
   - Design Decisions (DD1-DD2) - choices, Drew decides after evidence
5. Created insight capture documenting these patterns

## Pending Tasks

- [ ] Create PoC Phase 0 execution plan
- [ ] Execute PoC Phase 0 (in isolated test project)
- [ ] Document TV1-TV4 results in backlog item
- [ ] Create PoC Phase 1 execution plan (after Phase 0 validated)
- [ ] Execute PoC Phase 1
- [ ] Document TV5-TV6 results and DD1-DD2 evidence
- [ ] Drew makes DD1-DD2 decisions
- [ ] Update backlog item to "approved" status

## Current Work

Session ended after creating the insight capture document. Next step is creating the PoC execution plan.

## Next Steps

1. **Create PoC Phase 0 Execution Plan**
   - Save to: `docs/2026-01-28-hook-poc-phase0/2026-01-28-hook-poc-phase0-execution-plan.md`
   - Use the docs-as-code execution plan template
   - Plan should be executable by Claude (Claude is the implementer)
   - Must answer TV1-TV4 from the backlog item
   - Test location: Create isolated test project (NOT this skill's dev repo)

2. **PoC Phase 0 Scope (from backlog):**
   - Create minimal SKILL.md with PostToolUse and PostToolUseFailure hooks
   - Hook script: Log tool name, stdin JSON, and environment variables to temp file
   - Test in isolated project
   - Deliberately trigger tool failure to test PostToolUseFailure

3. **Technical Validations to Answer:**
   | ID | Question | Test Method |
   |----|----------|-------------|
   | TV1 | Does `$SKILL_DIR` resolve? | Echo `$SKILL_DIR` to log file |
   | TV2 | Does PostToolUseFailure fire? | Run invalid bash command |
   | TV3 | What is stdin JSON schema? | Log full stdin to file |
   | TV4 | Windows/Git Bash path issues? | Test on Windows |

4. **After PoC Phase 0 completes:**
   - Update backlog item with TV1-TV4 results
   - Proceed to PoC Phase 1 if all pass
   - Adjust approach if any fail

## Important Context

**Why hooks instead of instructions:**
- Hooks fire automatically at system level - Claude can't forget
- No permission prompts (hooks bypass Claude's permission model)
- Self-contained in skill (no user setup required)

**Key insight from session:**
The permission inconsistency in Run C suggests Claude's internal state affects whether bash commands are auto-approved. Hooks bypass this entirely by operating at system level.

**Requirements patterns established:**
- No SHOULD requirements (MUST or remove)
- All requirements need justification column
- Open questions split into Technical Validations vs Design Decisions
- PoC phases explicitly map to questions they answer

**Backlog item structure:**
The backlog item at `_feature-backlog-staging/deterministic-hook-based-logging-2026-01-27.md` contains:
- Full requirements with justifications
- Proposed hook scripts (draft)
- TV1-TV6 technical validation table
- DD1-DD2 design decision options
- Approval gates between phases

---

- **Document Status:** Ready for pickup
- **Key Files:**
  - Backlog: `_feature-backlog-staging/deterministic-hook-based-logging-2026-01-27.md`
  - Insights: `user-context/@insight-decision-captures/2026-01-27-requirements-and-open-questions-patterns.md`
