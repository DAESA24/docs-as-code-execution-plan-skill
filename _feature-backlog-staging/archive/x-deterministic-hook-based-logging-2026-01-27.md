---
doc_type: feature-backlog-item
title: "Deterministic Hook-Based Execution Logging"
created: 2026-01-27
status: rejected
rejected_date: 2026-01-28
rejected_reason: "Skill-bundled hooks only fire when the skill is invoked, not on every tool call. This architecture constraint makes self-contained always-on logging impossible. Pivoting to separate execution-logger skill with settings-based hooks instead."
feature_type_tags:
  - enhancement
  - automation
  - reliability
  - hooks
  - blocker-found
  - architecture-pivot
successor_project: "execution-logger-skill-dev"
successor_brief: "execution-logger-skill-dev/execution-logger-skill-brief.md"
poc_archive: "docs-as-code-execution-plan-skill-dev/docs/archive/x-2026-01-28-hook-poc-phase0/"
---

# Deterministic Hook-Based Execution Logging

## Problem Statement

The current execution logging feature (implemented 2026-01-27) relies on Claude following instructions to log events during plan execution. This approach is **non-deterministic** - Claude can forget to log, stop logging mid-execution, or inconsistently request permission for logging commands.

### Evidence from Production Runs

Three consecutive execution plan runs on 2026-01-27 demonstrated the inconsistency:

| Run | Archive | Logging Behavior | Summary Generated |
|-----|---------|------------------|-------------------|
| A | `x-a-2026-01-27-foundation-fixes` | Logged consistently (16 events) | NO |
| B | `x-b-2026-01-27-skills-dev-submodule-migration` | Perfect - logged all events (29 events) | YES |
| C | `x-c-2026-01-27-commands-pbc-submodule-migration` | Started logging, stopped after 1 event, required 2 manual reminders, then asked permission for every subsequent log entry | YES (after prompting) |

### Root Cause Analysis

The logging failures stem from instruction-based implementation:

1. **Context drift** - As Claude's context fills with plan execution details, logging instructions lose salience
2. **Priority competition** - Claude prioritizes "doing the work" over "documenting the work"
3. **Permission inconsistency** - Echo/append commands sometimes trigger permission prompts, sometimes don't
4. **No enforcement mechanism** - Nothing prevents Claude from skipping logs

### Current State

- Logging is instruction-based (SKILL.md tells Claude to log)
- Claude must remember to log after each operation
- Claude must construct and execute bash `echo >> log.jsonl` commands
- Permission prompts may or may not appear (unpredictable)
- No mechanism to detect or recover from missed log events

### Target State

- Logging is **hook-based** (automatic, system-level)
- Every tool call triggers a log event without Claude's involvement
- No permission prompts for logging operations
- Logging cannot be forgotten or skipped
- 100% deterministic: same inputs → same logging behavior

## Requirements

### Hook Configuration

| Requirement | Justification |
|-------------|---------------|
| **MUST** use skill-bundled hooks (in SKILL.md frontmatter) | Self-contained deployment - hooks travel with the skill, no separate user configuration required. This is the only hook mechanism that satisfies "no user setup" constraint. |
| **MUST** trigger on `PostToolUse` for all relevant tools (Bash, Edit, Write, Read) | PostToolUse fires after successful tool completion, capturing the actual operations performed. This is the hook event that provides tool call data needed for audit logging. |
| **MUST** trigger on `Stop` event to finalize log and generate summary | Stop fires when Claude finishes responding, which is the only reliable signal that execution is complete. Without this, the log would lack the execution_complete event and summary. |
| **MUST** use `once: true` for finalization hook | Without `once: true`, the Stop hook fires after every Claude response during execution, creating duplicate execution_complete events and overwriting the summary file. Log integrity requires exactly one finalization. |
| **MUST NOT** require user setup or configuration | The problem statement identifies "no enforcement mechanism" as a root cause. Requiring user setup introduces a new failure mode (misconfiguration) and shifts burden to users who expect the skill to "just work." |

### Log File Coordination

| Requirement | Justification |
|-------------|---------------|
| **MUST** detect active log file location from execution context | Hook scripts run in isolation and don't share Claude's context. They need a reliable way to find the log file without hardcoding paths, which would break across projects. |
| **MUST** use marker file for log path coordination | Marker file (`.claude-execution-log-path`) is explicit, debuggable, and persists across Claude responses within a session. Environment variables don't reliably persist, and convention-based discovery is fragile. |
| **MUST** handle case where no execution plan is active (skip logging) | The skill's hooks fire on ALL tool calls in the session, not just during plan execution. Without graceful skip behavior, hooks would error or create orphan logs when used outside execution context. |
| **MUST** append events atomically (no partial writes) | Concurrent hook invocations could interleave writes, corrupting JSONL format. Atomic append (single `echo >>` operation) ensures each line is complete and parseable. |

### Hook Script Requirements

| Requirement | Justification |
|-------------|---------------|
| **MUST** be self-contained (no external dependencies beyond bash/jq) | Users may not have Python, Node, or other runtimes installed. Bash and jq are universally available in Git Bash (Windows) and standard Unix environments where Claude Code runs. |
| **MUST** handle errors gracefully (logging failure shouldn't halt execution) | Logging is observability, not core functionality. A logging failure (disk full, permission issue) should not prevent the actual plan execution from completing. Use `exit 0` even on error. |
| **MUST** work on Windows (Git Bash) and Unix | Drew's environment is Windows with Git Bash. Many Claude Code users are on macOS/Linux. Scripts must use POSIX-compatible constructs that work in both environments. |
| **MUST** use Git Bash paths (`/c/Users/...` not `C:\Users\...`) | Windows backslash paths break in bash scripts. Git Bash uses Unix-style paths. All path handling must use forward slashes and `/c/` mount notation for Windows drives. |

### Backward Compatibility

| Requirement | Justification |
|-------------|---------------|
| **MUST** maintain existing log format (JSONL with same event schema) | Existing tooling (log analysis scripts, summary generators) expects the current schema. Changing format would break downstream consumers and require migration effort. |
| **MUST** generate same execution summary format | Users and processes that consume summaries expect consistent structure. The summary is a human-readable artifact; changing its format creates confusion and breaks workflows. |

**Note:** "Support plans created before hook implementation" was removed as a requirement. The marker file approach inherently provides backward compatibility: old plans don't create marker files, so hooks skip gracefully. No special code needed.

## Proposed Architecture

### Option 1: Skill-Bundled Hooks (Recommended)

Add hooks directly to SKILL.md frontmatter:

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

**Key insight:** Hooks receive rich context via stdin JSON, including:
- `session_id` - For correlating events
- `tool_name` - Which tool was called
- `tool_input` - Full tool parameters
- `cwd` - Working directory

### Log Path Coordination Strategy

The hook scripts need to know where to write logs. Options:

**Option A: Marker File**
When execution starts, create `.claude-execution-log-path` in working directory containing the log file path. Hook scripts read this file.

**Option B: Environment Variable**
Use `CLAUDE_EXECUTION_LOG` environment variable set during pre-flight. Hooks check for this variable.

**Option C: Convention-Based Discovery**
Hook scripts scan for `*-execution-log.jsonl` in `docs/*/` directories, use most recently modified.

**Recommendation:** Option A (Marker File) - Most explicit, no env var persistence issues, easy to verify.

### Hook Script: log-tool-event.sh

```bash
#!/usr/bin/env bash
# Receives tool call info via stdin, appends to execution log

set -euo pipefail

# Read marker file for log path
MARKER_FILE=".claude-execution-log-path"
if [[ ! -f "$MARKER_FILE" ]]; then
    # No active execution - skip silently
    exit 0
fi

LOG_FILE=$(cat "$MARKER_FILE")
if [[ ! -f "$LOG_FILE" ]]; then
    # Log file doesn't exist - skip silently
    exit 0
fi

# Read hook input from stdin
INPUT=$(cat)

# Extract fields using jq
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name // "unknown"')
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# Build log event based on tool type
case "$TOOL_NAME" in
    Bash)
        COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""' | head -c 200)
        EVENT=$(jq -n \
            --arg ts "$TIMESTAMP" \
            --arg tool "$TOOL_NAME" \
            --arg cmd "$COMMAND" \
            '{event:"tool_call",ts:$ts,tool:$tool,command:$cmd}')
        ;;
    Edit|Write)
        FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // ""')
        EVENT=$(jq -n \
            --arg ts "$TIMESTAMP" \
            --arg tool "$TOOL_NAME" \
            --arg file "$FILE_PATH" \
            '{event:"tool_call",ts:$ts,tool:$tool,file:$file}')
        ;;
    Read)
        FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // ""')
        EVENT=$(jq -n \
            --arg ts "$TIMESTAMP" \
            --arg tool "$TOOL_NAME" \
            --arg file "$FILE_PATH" \
            '{event:"tool_call",ts:$ts,tool:$tool,file:$file}')
        ;;
    *)
        # Unknown tool - log generic event
        EVENT=$(jq -n \
            --arg ts "$TIMESTAMP" \
            --arg tool "$TOOL_NAME" \
            '{event:"tool_call",ts:$ts,tool:$tool}')
        ;;
esac

# Append atomically
echo "$EVENT" >> "$LOG_FILE"
```

### Hook Script: finalize-execution-log.sh

```bash
#!/usr/bin/env bash
# Finalizes log and generates execution summary

set -euo pipefail

MARKER_FILE=".claude-execution-log-path"
if [[ ! -f "$MARKER_FILE" ]]; then
    exit 0
fi

LOG_FILE=$(cat "$MARKER_FILE")
if [[ ! -f "$LOG_FILE" ]]; then
    exit 0
fi

# Write execution_complete event
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
TOTAL_EVENTS=$(wc -l < "$LOG_FILE")

echo "{\"event\":\"execution_complete\",\"ts\":\"$TIMESTAMP\",\"status\":\"success\",\"total_events\":$TOTAL_EVENTS}" >> "$LOG_FILE"

# Generate summary (basic version - Claude can enhance)
SUMMARY_FILE="${LOG_FILE%.jsonl}-summary.md"
LOG_DIR=$(dirname "$LOG_FILE")
LOG_NAME=$(basename "$LOG_FILE")

cat > "$SUMMARY_FILE" << EOF
# Execution Summary

- **Log File:** \`$LOG_NAME\`
- **Completed:** $TIMESTAMP
- **Total Events:** $TOTAL_EVENTS

## Event Summary

\`\`\`
$(jq -r '.event' "$LOG_FILE" | sort | uniq -c | sort -rn)
\`\`\`

---

- **Generated by:** finalize-execution-log.sh (hook)
EOF

# Clean up marker file
rm -f "$MARKER_FILE"

# Output for Claude's context (optional)
echo "Execution log finalized: $LOG_FILE"
echo "Summary generated: $SUMMARY_FILE"
```

## Implementation Phases

### Phase 0: Proof of Concept - Minimal Hook

**Goal:** Validate that skill-bundled hooks work as expected and answer key technical unknowns.

**Scope:**

- Create minimal SKILL.md with PostToolUse and PostToolUseFailure hooks
- Hook script: Log tool name, stdin JSON, and environment variables to a temp file
- Test in isolated project (not this skill's development repo)
- Deliberately trigger a tool failure to test PostToolUseFailure

**Success Criteria:**

- [ ] Hook fires on every Bash/Edit/Write call
- [ ] No permission prompts for hook execution
- [ ] Hook receives expected stdin JSON
- [ ] Hook output appears (or doesn't appear) as expected

**Technical Validations Answered:**

| Question | Test | Result |
|----------|------|--------|
| TV1: Does `$SKILL_DIR` resolve in hook commands? | Echo `$SKILL_DIR` to log file | Pending |
| TV2: Does PostToolUseFailure hook fire on errors? | Run invalid bash command, check if hook fires | Pending |
| TV3: What is the exact stdin JSON schema? | Log full stdin to file, document structure | Pending |
| TV4: Are there Windows/Git Bash path issues? | Test on Drew's Windows environment | Pending |

**Learnings to Capture:**

- Actual stdin JSON schema (document for reference)
- Any environment variables available to hooks
- Performance impact (subjective observation)
- Any unexpected behaviors

### Phase 1: Proof of Concept - Log Coordination

**Goal:** Validate marker file approach for log path coordination.

**Scope:**

- Extend PoC to use marker file pattern
- Test: Start execution (creates marker), run tools (hooks log), stop (finalize)
- Test: Run tools without active execution (hooks skip gracefully)
- Test: Include Read tool calls to assess noise level

**Success Criteria:**

- [ ] Marker file created correctly during pre-flight
- [ ] Hook scripts read marker file reliably
- [ ] Hooks skip gracefully when no marker exists
- [ ] Marker file cleaned up on completion
- [ ] Stop hook with `once: true` fires exactly once

**Technical Validations Answered:**

| Question | Test | Result |
|----------|------|--------|
| TV5: Does `once: true` work correctly? | Run multiple tool calls, verify only one finalization | Pending |
| TV6: Can hooks detect phase/step context? | Examine stdin JSON for any contextual data | Pending |

**Design Decision Evidence Gathered:**

| Decision | Evidence to Collect |
|----------|---------------------|
| DD1: Log Read calls? | Count Read events vs. mutating events in test log, assess signal-to-noise |
| DD2: Hybrid approach for phase/step? | Confirm hooks cannot detect semantic boundaries from stdin |

**Learnings to Capture:**

- Race conditions observed?
- Path encoding issues?
- What happens if execution is interrupted?
- Read tool call noise level (count and assess)

### Phase 2: Full Implementation

**Prerequisites:**

- All Technical Validations (TV1-TV6) have documented results
- All Design Decisions (DD1-DD2) have Drew's explicit choice recorded
- PoC learnings incorporated into requirements/architecture

**Scope:**

- Update SKILL.md with full hook configuration
- Create production hook scripts in `scripts/` directory
- Update pre-flight validation to create marker file
- Update template with marker file initialization
- Add hook scripts to skill package

**Files to Modify:**

- `SKILL.md` - Add hooks frontmatter
- `references/docs-as-code-execution-plan-template.md` - Add marker file initialization
- NEW: `scripts/log-tool-event.sh`
- NEW: `scripts/finalize-execution-log.sh`

### Phase 3: Remove Instruction-Based Logging

**Goal:** Clean up now-redundant instruction-based logging.

**Scope:**

- Remove "Log Events to Emit" sections from template
- Remove manual logging instructions from SKILL.md
- Update guide to document hook-based approach
- Keep schema documentation (still relevant)

**Note:** Do this AFTER Phase 2 is validated to avoid losing logging entirely.

## Acceptance Criteria

### Determinism

- [ ] 10 consecutive execution plan runs produce consistent logging behavior
- [ ] No permission prompts for logging operations
- [ ] No manual intervention required for logging

### Completeness

- [ ] Every Bash call logged
- [ ] Every Edit call logged
- [ ] Every Write call logged
- [ ] execution_start event captured
- [ ] execution_complete event captured
- [ ] Execution summary generated automatically

### Robustness

- [ ] Hooks skip gracefully when no execution active
- [ ] Hook failures don't halt plan execution
- [ ] Interrupted executions leave partial but valid logs
- [ ] Works on Windows (Git Bash) and Unix

### Backward Compatibility

- [ ] Log format unchanged (same JSONL schema)
- [ ] Summary format unchanged (same Markdown structure)
- [ ] Plans created before this feature work correctly

## Technical Validations (Answered by PoC)

These are factual unknowns that PoC testing will resolve with pass/fail results.

| ID | Question | PoC Phase | Test Method | Result | Notes |
|----|----------|-----------|-------------|--------|-------|
| TV1 | Does `$SKILL_DIR` resolve in hook commands? | 0 | Echo `$SKILL_DIR` to log file | **N/A** | Not documented; not supported per official docs |
| TV2 | Does PostToolUseFailure hook fire on errors? | 0 | Run invalid bash command, check hook | **N/A** | Not supported in skill-bundled hooks per docs |
| TV3 | What is the exact stdin JSON schema? | 0 | Log full stdin, document structure | **N/A** | Could not test - hooks never fired |
| TV4 | Are there Windows/Git Bash path issues? | 0 | Test on Windows environment | **N/A** | Could not test - hooks never fired |
| TV5 | Does `once: true` work correctly? | 1 | Multiple tool calls, verify one finalization | **BLOCKED** | Cannot test - skill-bundled hooks don't fire automatically |
| TV6 | Can hooks detect phase/step context from stdin? | 1 | Examine stdin JSON for contextual data | **BLOCKED** | Cannot test - skill-bundled hooks don't fire automatically |

**CRITICAL FINDING (Phase 0):** Skill-bundled hooks only fire when the skill is invoked, NOT on every tool call. This invalidates the core assumption that skill-bundled hooks provide "always-on" logging. See Phase 0 PoC Results for details.

**Approval Gate:** ~~All Technical Validations must have documented results before proceeding to Phase 2.~~ **BLOCKED** - Architecture pivot required before Phase 1 can proceed.

## Design Decisions (Drew Decides After PoC Evidence)

These are choices between valid options. PoC gathers evidence; Drew makes the final call.

### DD1: Should Read tool calls be logged?

**Options:**

| Option | Pros | Cons |
|--------|------|------|
| A: Log all tools (including Read) | Complete audit trail, no missed context | Noisy logs, larger files |
| B: Log only mutating tools (Bash, Edit, Write) | Cleaner logs, focused on changes | May miss relevant context |
| C: Make configurable via plan metadata | Flexible per-project | Added complexity |

**Evidence to gather in PoC Phase 1:** Run test with Read logging enabled, count Read events vs. mutating events, assess signal-to-noise ratio.

**Drew's Decision:** Pending (awaiting PoC evidence)

### DD2: How to handle phase/step boundary events?

**Context:** Hooks capture tool calls automatically, but semantic events (phase_start, step_complete) require understanding execution context.

**Options:**

| Option | Pros | Cons |
|--------|------|------|
| A: Hybrid - hooks log tools, Claude logs phases/steps | Clear separation, plays to strengths | Still relies on Claude for some logging |
| B: Marker bash commands hooks recognize | Fully automated | Fragile pattern matching, clutters execution |
| C: Accept tool-call-only logs | Simplest implementation | Loses semantic structure |

**Evidence to gather in PoC Phase 1:** Confirm whether stdin JSON contains any phase/step context (TV6). If yes, Option A may not be needed.

**Drew's Decision:** Pending (awaiting TV6 result)

**Approval Gate:** All Design Decisions must have Drew's explicit choice recorded before proceeding to Phase 2.

## Research Notes

### Claude Code Hooks Documentation

From official documentation (retrieved 2026-01-27):

**Hook Events Available:**
- `PreToolUse` - Before tool call (can block)
- `PostToolUse` - After successful tool call
- `PostToolUseFailure` - After failed tool call
- `Stop` - When agent finishes responding
- `SessionStart` / `SessionEnd` - Session boundaries

**Skill-Bundled Hooks:**
Skills CAN include hooks in YAML frontmatter. Supported events: PreToolUse, PostToolUse, Stop.

**Hook Input (stdin JSON):**
```json
{
  "session_id": "abc123",
  "transcript_path": "path/to/transcript.jsonl",
  "cwd": "working directory",
  "hook_event_name": "PostToolUse",
  "tool_name": "Bash|Edit|Write|etc",
  "tool_input": { /* tool-specific data */ }
}
```

**Key Insight:** Hooks execute in user space, outside Claude's permission model. This is why they won't trigger permission prompts.

### Why Previous Logging Failed

Analysis of Run C behavior:
1. Claude logged `execution_start` event (instruction followed initially)
2. Context filled with submodule registration commands
3. Logging instructions lost salience
4. User reminded Claude to log
5. Claude started requesting permission (possibly different context state)
6. Each `echo >>` command treated as new operation requiring approval

The permission inconsistency suggests Claude's internal state affects whether bash commands are auto-approved. Hooks bypass this entirely.

## Related Items

- **Builds on:** `x-execution-logging-2026-01-27.md` (implemented)
- **Related to:** `x-autonomous-execution-permissions-2026-01-26.md` (blocked)

## Context and Evidence

### Origin

Discussion with Drew on 2026-01-27 after observing inconsistent logging across three consecutive execution plan runs. Drew requested exploration of hook-based solutions for deterministic logging.

### Design Principles

> "Enforcement over instruction" - System-level mechanisms are more reliable than behavioral instructions.

> "Fail-safe logging" - If Claude forgets, hooks remember.

> "No user setup required" - Skill-bundled hooks are self-contained.

### Why Hooks Are the Right Solution

| Approach | Deterministic | No Permission Prompts | Self-Contained | Always-On |
|----------|--------------|----------------------|----------------|-----------|
| Instructions to Claude | NO | NO | YES | N/A |
| Global settings.json hooks | YES | YES | NO (requires setup) | **YES** |
| Skill-bundled hooks | YES | YES | YES | **NO** |

~~Skill-bundled hooks provide all three properties.~~

**UPDATE (2026-01-28):** Phase 0 PoC revealed skill-bundled hooks only fire when the skill is invoked, NOT on every tool call. This means skill-bundled hooks **cannot** provide always-on logging. Settings-based hooks are required for that use case.

## Phase 0 PoC Results (2026-01-28)

### Critical Finding: Skill-Bundled Hooks Are Scoped to Skill Lifecycle

From official Claude Code documentation:

> "These hooks are scoped to the component's lifecycle and only run when that component is active."

**What this means:**
- Installing a skill with hooks at `~/.claude/skills/` does NOT make hooks run automatically
- Skills must be invoked (via `/skill-name` or automatic context-based invocation) for hooks to fire
- For always-on hooks, must use settings-based hooks in `.claude/settings.json`

### Additional Constraints Discovered

1. **$SKILL_DIR is not documented** - Cannot use relative paths from skill directory
2. **PostToolUseFailure not supported in skills** - Only PreToolUse, PostToolUse, Stop events
3. **Skills only load on invocation** - Full skill content (including hooks) only loads when skill is used

### Implications

The proposed architecture of "skill-bundled hooks for self-contained deployment" is **not viable** for always-on logging.

**Alternative approaches:**

1. **Settings-based hooks** - Deploy hooks via user/project settings.json
   - Pro: Works for always-on logging
   - Con: Not self-contained; requires user configuration

2. **Hybrid approach** - Skill provides scripts, user adds hooks to settings
   - Pro: Skill packages logic; settings enable it
   - Con: Adds setup friction

3. **Abandon hook-based approach** - Find different mechanism
   - Con: Back to the original problem of instruction-based unreliability

### PoC Artifacts

- **Execution Plan:** `docs/2026-01-28-hook-poc-phase0/2026-01-28-hook-poc-phase0-execution-plan.md`
- **Results Summary:** `docs/2026-01-28-hook-poc-phase0/poc-phase0-results.md`

---

- **Document Status:** Blocked - architecture pivot required
- **Last Updated:** 2026-01-28
- **Next Steps:**
  1. ~~Run Phase 0 PoC to answer TV1-TV4~~ **DONE** - Architecture blocker discovered
  2. ~~Run Phase 1 PoC to answer TV5-TV6 and gather DD1-DD2 evidence~~ **BLOCKED**
  3. **DECISION REQUIRED:** Choose alternative approach:
     - Option A: Pivot to settings-based hooks (requires user configuration)
     - Option B: Hybrid approach (skill provides scripts, user enables hooks)
     - Option C: Abandon hook-based approach (find different mechanism)
     - Option D: Defer feature (accept current instruction-based limitations)
  4. If proceeding: Update architecture based on chosen approach
  5. If proceeding: Run new PoC validating settings-based hooks work as expected
