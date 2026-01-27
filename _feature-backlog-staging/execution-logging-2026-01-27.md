---
doc_type: feature-backlog-item
title: "Add Execution Logging with Project Subdirectories"
created: 2026-01-27
status: captured
execution_order: 4
execution_sequence:
  - position: 1
    item: validation-loop-execution-protocol-2026-01-26.md
    status: implemented
  - position: 2
    item: pre-execution-git-safety-check-2026-01-26.md
    status: implemented
  - position: 3
    item: autonomous-execution-permissions-2026-01-26.md
    status: pending
  - position: 4
    item: execution-logging-2026-01-27.md
    status: THIS ITEM
depends_on: autonomous-execution-permissions-2026-01-26.md
feature_type_tags:
  - enhancement
  - execution
  - logging
  - documentation
---

# Add Execution Logging with Project Subdirectories

## Problem Statement

When Claude executes a docs-as-code execution plan autonomously, there is no persistent record of what operations were performed. If something goes wrong, troubleshooting requires reconstructing events from memory or git diffs. Additionally, execution artifacts (plan, logs) are scattered as flat files in `docs/`, making it difficult to associate related files.

### Prerequisites

**This feature depends on:** `autonomous-execution-permissions-2026-01-26.md`

Logging is most valuable when execution is autonomous. With permission prompts, the user sees each operation. With autonomous execution, the log becomes the primary audit trail.

### Current State

- Execution plans saved as flat files in `docs/`
- No execution log generated during plan execution
- No structured record of tool calls, outcomes, or errors
- Related files (plan, potential future logs) not grouped
- Post-execution analysis requires manual reconstruction

### Target State

- Execution artifacts grouped in project subdirectories under `docs/`
- JSONL execution log captures every significant event
- Markdown summary generated for human review
- Claude can analyze logs to identify patterns and improvements
- Clear audit trail for troubleshooting and rollback decisions

## Requirements

### Project Subdirectory Convention

- **MUST** create project subdirectory when generating execution plan
- **MUST** use naming pattern: `YYYY-MM-DD-<project-slug>/`
- **MUST** place all related artifacts in project subdirectory
- **SHOULD** limit project slug to 2-4 kebab-case words

### Execution Logging

- **MUST** generate JSONL log file during plan execution
- **MUST** capture: tool calls, outcomes, durations, errors
- **MUST** generate Markdown summary at execution completion
- **MUST** use naming pattern: `YYYY-MM-DD-<project-slug>-<doc-type>.<ext>`
- **SHOULD** truncate large outputs (first/last 500 chars)
- **SHOULD** include timestamps for all events
- **SHOULD** design log format for Claude-as-analyst (structured, filterable)

### Documentation

- **MUST** document project subdirectory convention in guide
- **MUST** document log event types and schema
- **MUST** update template with log initialization
- **SHOULD** provide examples of log analysis queries

### Asset Template

- **MUST** create `assets/templates/execution-log-template.jsonl` with sample events
- **MUST** create `assets/templates/execution-summary-template.md` with structure
- **SHOULD** include comments/documentation within templates where format allows

## Implementation Phases

Implementation follows the skill file update order defined in `skill-dev-config.yaml`:

### Phase 0: Create Asset Templates

- **Directory:** `assets/templates/`
- **Files to create:**
  - `execution-log-template.jsonl` - Sample log with all event types
  - `execution-summary-template.md` - Markdown summary structure
- **Purpose:** Concrete examples for reference during implementation and testing

### Phase 1: Update docs-as-code-guide.md

- **File:** `references/docs-as-code-guide.md`
- **Changes:**
  - Add Pattern 9: "Project Subdirectory Convention"
    - Directory naming: `YYYY-MM-DD-<project-slug>/`
    - File naming within directory
    - Rationale for grouping related artifacts
  - Add Pattern 10: "Execution Logging"
    - Log format (JSONL) and rationale
    - Event types and schema
    - Truncation policy for large outputs
    - Log analysis guidance for Claude

### Phase 2: Update docs-as-code-execution-plan-template.md

- **File:** `references/docs-as-code-execution-plan-template.md`
- **Changes:**
  - Update Pre-Flight Validation to initialize log file
  - Add log event emission patterns to phase templates
  - Add log finalization to Completion Checklist
  - Update example paths to use project subdirectory convention

### Phase 3: Update SKILL.md

- **File:** `SKILL.md`
- **Changes:**
  - Update "Mode 1: Create Execution Plan" workflow:
    - Create project subdirectory before saving plan
    - Use new naming convention
  - Update "Mode 2: Execute Existing Plan" workflow:
    - Initialize log at execution start
    - Emit events during execution
    - Generate summary at completion
  - Add "Execution Logging" section documenting:
    - Log file location and naming
    - Event types captured
    - How to analyze logs
  - Update file naming convention documentation

## Acceptance Criteria

### Asset Templates

- [ ] `assets/templates/execution-log-template.jsonl` created with all event types
- [ ] `assets/templates/execution-summary-template.md` created with full structure
- [ ] Templates are valid (JSONL parses, Markdown renders)

### Project Subdirectory Convention

- [ ] Guide documents Pattern 9 (Project Subdirectory Convention)
- [ ] Template updated with project subdirectory paths
- [ ] SKILL.md Mode 1 creates project subdirectory
- [ ] Test: Create plan, verify subdirectory structure created

### Execution Logging

- [ ] Guide documents Pattern 10 (Execution Logging)
- [ ] Template includes log initialization in Pre-Flight
- [ ] SKILL.md Mode 2 emits log events during execution
- [ ] Log file created with correct naming convention
- [ ] Summary file generated at execution completion
- [ ] Test: Execute plan, verify log contains expected events
- [ ] Test: Verify Claude can parse and analyze log file

## Technical Details

### Directory and File Naming Convention

**Directory pattern:**
```
docs/YYYY-MM-DD-<project-slug>/
```

**File patterns within directory:**
```
YYYY-MM-DD-<project-slug>-execution-plan.md
YYYY-MM-DD-<project-slug>-execution-log.jsonl
YYYY-MM-DD-<project-slug>-execution-summary.md
```

**Project slug rules:**
- 2-4 words, kebab-case
- Describes the goal/topic
- Examples: `skill-upgrade`, `database-migration`, `auth-refactor`

**Example structure:**
```
docs/
├── 2026-01-27-skill-upgrade/
│   ├── 2026-01-27-skill-upgrade-execution-plan.md
│   ├── 2026-01-27-skill-upgrade-execution-log.jsonl
│   └── 2026-01-27-skill-upgrade-execution-summary.md
└── archives/
    └── 2026-01-20-auth-migration/
        └── ...
```

### Log Event Schema (JSONL)

Each line is a self-contained JSON object:

```jsonl
{"event":"execution_start","ts":"2026-01-27T14:30:00Z","plan_path":"...","plan_title":"...","pre_exec_commit":"a1b2c3d"}
{"event":"phase_start","ts":"...","phase":1,"name":"Setup","autonomous":true}
{"event":"tool_call","ts":"...","tool":"Bash","command":"mkdir -p ...","exit_code":0,"duration_ms":45}
{"event":"tool_call","ts":"...","tool":"Edit","file":"...","lines_changed":5,"outcome":"success"}
{"event":"validation","ts":"...","step":"1.1","item":"Directory exists","result":"pass"}
{"event":"error","ts":"...","step":"1.3","type":"command_failed","message":"...","stderr_excerpt":"..."}
{"event":"user_approval","ts":"...","step":"2.1","action":"Delete files","response":"approved"}
{"event":"phase_complete","ts":"...","phase":1,"report":"PHASE 1 COMPLETE: ..."}
{"event":"execution_complete","ts":"...","status":"success","phases_completed":3,"errors":0,"duration_ms":600000}
```

### Event Types

| Event | Fields | When Emitted |
|-------|--------|--------------|
| `execution_start` | plan_path, plan_title, pre_exec_commit(s) | Before pre-flight |
| `phase_start` | phase, name, autonomous | Before each phase |
| `phase_complete` | phase, report | After each phase |
| `step_start` | step, name | Before each step |
| `step_complete` | step, report | After each step |
| `tool_call` | tool, params, outcome, duration_ms, stdout/stderr excerpt | After each tool use |
| `validation` | step, item, result | After each checkbox verification |
| `error` | step, type, message, recovery_action | When error occurs |
| `user_approval` | step, action, response | When user approval requested |
| `state_change` | type, details | File created/modified, git commit, etc. |
| `execution_complete` | status, phases_completed, errors, duration_ms | At end |

### Truncation Policy

For large outputs (stdout, stderr, file content):
- Keep first 500 characters
- Keep last 500 characters
- Mark truncation: `"...[truncated: 15000 chars]..."`

### Execution Summary (Markdown)

Generated at execution completion for human review:

```markdown
# Execution Summary: <Plan Title>

- **Executed:** 2026-01-27 14:30 - 14:45 (15 min)
- **Status:** Success
- **Phases:** 3/3 completed
- **Errors:** 0

## Timeline

| Time | Event | Details |
|------|-------|---------|
| 14:30:00 | Start | Pre-flight validation |
| 14:30:05 | Phase 1 | Setup - 2 steps |
| ... | ... | ... |

## Errors Encountered

(None)

## Files Modified

- `src/config.ts` - 15 lines changed
- `README.md` - created

## Log File

Full log: `2026-01-27-skill-upgrade-execution-log.jsonl`
```

## Related Items

- **Depends on:** `autonomous-execution-permissions-2026-01-26.md`
  - Logging is most valuable during autonomous execution
  - Pattern: "Autonomy requires accountability"

## Context and Evidence

### Origin

Discussion with Drew on 2026-01-27. Key insight: With autonomous execution permissions, Drew needs visibility into what Claude did. The log serves as an audit trail and enables pattern analysis for skill improvement.

### Design Principles

> "Claude as primary analyst" - The JSONL format is optimized for Claude to parse and analyze, enabling pattern detection across multiple executions.

> "Autonomy requires accountability" - Broad permissions are safe when there's a clear record of what happened.

### Why JSONL Over Markdown for Logs

| Criterion | JSONL | Markdown |
|-----------|-------|----------|
| Claude can parse | Native JSON | Regex/heuristics |
| Filter by event type | Trivial | Difficult |
| Aggregate across runs | Easy | Manual |
| Human readable | Needs viewer | Direct |

Solution: JSONL for data, auto-generate Markdown summary for humans.

### Asset Template: execution-log-template.jsonl

Complete example log demonstrating all event types:

```jsonl
{"event":"execution_start","ts":"YYYY-MM-DDTHH:MM:SSZ","plan_path":"docs/YYYY-MM-DD-project-slug/YYYY-MM-DD-project-slug-execution-plan.md","plan_title":"<Plan Title>","pre_exec_commit":"<commit-sha>","pre_exec_skill_commit":"<skill-commit-sha or null>"}
{"event":"phase_start","ts":"YYYY-MM-DDTHH:MM:SSZ","phase":0,"name":"Pre-Flight Validation","autonomous":true}
{"event":"tool_call","ts":"YYYY-MM-DDTHH:MM:SSZ","tool":"Bash","command":"git rev-parse --git-dir","exit_code":0,"stdout_excerpt":".git","duration_ms":45}
{"event":"validation","ts":"YYYY-MM-DDTHH:MM:SSZ","step":"0.1","item":"Git repository detected","result":"pass"}
{"event":"phase_complete","ts":"YYYY-MM-DDTHH:MM:SSZ","phase":0,"report":"PRE-FLIGHT COMPLETE: All checks passed"}
{"event":"phase_start","ts":"YYYY-MM-DDTHH:MM:SSZ","phase":1,"name":"<Phase 1 Name>","autonomous":true}
{"event":"step_start","ts":"YYYY-MM-DDTHH:MM:SSZ","step":"1.1","name":"<Step Name>"}
{"event":"tool_call","ts":"YYYY-MM-DDTHH:MM:SSZ","tool":"Edit","file":"/path/to/file.md","lines_changed":5,"outcome":"success","duration_ms":120}
{"event":"tool_call","ts":"YYYY-MM-DDTHH:MM:SSZ","tool":"Write","file":"/path/to/new-file.md","bytes_written":1024,"outcome":"success","duration_ms":80}
{"event":"tool_call","ts":"YYYY-MM-DDTHH:MM:SSZ","tool":"Read","file":"/path/to/file.md","lines_read":50,"duration_ms":30}
{"event":"validation","ts":"YYYY-MM-DDTHH:MM:SSZ","step":"1.1","item":"File updated correctly","result":"pass"}
{"event":"step_complete","ts":"YYYY-MM-DDTHH:MM:SSZ","step":"1.1","report":"STEP 1.1 COMPLETE: <summary>"}
{"event":"error","ts":"YYYY-MM-DDTHH:MM:SSZ","step":"1.2","type":"command_failed","command":"npm install","exit_code":1,"stderr_excerpt":"ENOENT: package.json not found...[truncated: 2500 chars]...","recovery_action":"Created package.json and retried"}
{"event":"state_change","ts":"YYYY-MM-DDTHH:MM:SSZ","type":"file_created","path":"/path/to/package.json"}
{"event":"phase_complete","ts":"YYYY-MM-DDTHH:MM:SSZ","phase":1,"report":"PHASE 1 COMPLETE: <summary>"}
{"event":"phase_start","ts":"YYYY-MM-DDTHH:MM:SSZ","phase":2,"name":"<Phase 2 Name>","autonomous":false}
{"event":"user_approval","ts":"YYYY-MM-DDTHH:MM:SSZ","step":"2.1","action":"Delete original files (350 files)","response":"approved"}
{"event":"tool_call","ts":"YYYY-MM-DDTHH:MM:SSZ","tool":"Bash","command":"rm -rf /path/to/original","exit_code":0,"stdout_excerpt":"","duration_ms":500}
{"event":"state_change","ts":"YYYY-MM-DDTHH:MM:SSZ","type":"directory_deleted","path":"/path/to/original","files_affected":350}
{"event":"state_change","ts":"YYYY-MM-DDTHH:MM:SSZ","type":"git_commit","sha":"abc123","message":"feat: implement feature X"}
{"event":"phase_complete","ts":"YYYY-MM-DDTHH:MM:SSZ","phase":2,"report":"PHASE 2 COMPLETE: <summary>"}
{"event":"execution_complete","ts":"YYYY-MM-DDTHH:MM:SSZ","status":"success","phases_completed":2,"total_phases":2,"errors_encountered":1,"errors_recovered":1,"duration_ms":300000}
```

### Asset Template: execution-summary-template.md

```markdown
# Execution Summary: <Plan Title>

- **Plan:** `<plan-path>`
- **Executed:** YYYY-MM-DD HH:MM - HH:MM (<duration>)
- **Status:** Success | Failed | Rolled Back
- **Phases:** N/N completed
- **Errors:** N encountered, N recovered

## Pre-Execution State

- **Git Commit:** `<pre_exec_commit>`
- **Skills Commit:** `<pre_exec_skill_commit>` (if applicable)
- **Rollback Command:** `git reset --hard <pre_exec_commit>`

## Timeline

| Time | Phase/Step | Event | Details |
|------|------------|-------|---------|
| HH:MM:SS | Pre-Flight | Start | Validation checks |
| HH:MM:SS | Pre-Flight | Complete | All checks passed |
| HH:MM:SS | Phase 1 | Start | <Phase Name> |
| HH:MM:SS | Step 1.1 | Complete | <Report summary> |
| ... | ... | ... | ... |

## Errors Encountered

| Step | Error | Recovery |
|------|-------|----------|
| 1.2 | command_failed: npm install | Created package.json and retried |

(Or: "None")

## Files Modified

| File | Action | Details |
|------|--------|---------|
| `src/config.ts` | edited | 15 lines changed |
| `README.md` | created | 1024 bytes |
| `/path/to/original/` | deleted | 350 files |

## Git Operations

| Time | Operation | Details |
|------|-----------|---------|
| HH:MM:SS | commit | `abc123` - "feat: implement feature X" |

(Or: "None")

## Log File

Full execution log: `<log-filename>.jsonl`

---

- **Generated:** YYYY-MM-DD HH:MM:SS
- **Log Events:** N total
```

---

- **Document Status:** Captured (Updated with asset templates)
- **Last Updated:** 2026-01-27
- **Depends On:** autonomous-execution-permissions-2026-01-26.md
