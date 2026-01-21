---
doc_type: feature-backlog-item
title: "Sub-Agent Parallelization Strategy for Execution Plans"
created: 2026-01-21
status: captured
feature_type_tags:
  - enhancement
  - performance
  - automation
---

# Sub-Agent Parallelization Strategy for Execution Plans

- **Created:** 2026-01-21
- **Source:** Session discussion about context window optimization
- **Audience:** Claude Code (executing agent)
- **Status:** Captured

---

## Problem Statement

Complex, multi-phase execution plans consume significant context window in the orchestrating Claude Code session. When phases contain multiple independent tasks, executing them sequentially wastes context that could be preserved by delegating to sub-agents running in parallel.

**Current state:**
- All execution plan steps run in the main session
- Independent tasks within a phase execute sequentially
- Context accumulates linearly regardless of task independence

**Target state:**
- Skill evaluates whether a plan benefits from sub-agent parallelization
- Independent tasks within phases can run concurrently via sub-agents
- Orchestrating session validates sub-agent output before proceeding
- Context window preserved for complex multi-phase plans

---

## Core Requirements

### 1. Orchestrator Must Validate Sub-Agent Output

This is the critical constraint. The main session must be able to verify:

1. **Work was actually completed** - Not just reported as done
2. **Work matches the assigned task** - Sub-agent didn't drift or misunderstand
3. **Results are verifiable in the main conversation** - Drew can see proof

**Implication:** Validation criteria must be programmatically verifiable, not dependent on "reading the sub-agent's mind."

### 2. Template Must Expose Dependencies

The plan structure must clearly indicate which tasks can parallelize:

- Tasks with no dependencies on other tasks in the same phase
- Tasks whose outputs don't conflict (no race conditions on files)
- Tasks with independent validation criteria

### 3. Sub-Agent Output Artifacts Must Be Specified

Each parallelizable task needs:

- What files/state changes the sub-agent produces
- Verification commands the orchestrator runs post-completion
- Clear success criteria that work without sub-agent context

---

## Proposed Implementation

### Phase 1: Template Structure Changes (Foundation)

Add structural patterns to the execution plan template that enable sub-agent evaluation.

#### 1.1 Task Dependency Annotation

Add optional dependency marker to steps:

```markdown
### STEP 2.1: Create database migration script

**Autonomous:** YES
**Dependencies:** None  # or "Depends on: 2.0" if sequential required
**Parallelizable:** YES
```

#### 1.2 Output Artifact Specification

Add to parallelizable steps:

```markdown
**Output Artifacts:**
- File: `migrations/001_add_users_table.sql`
- File: `migrations/001_add_users_table_test.sql`

**Orchestrator Verification:**
```bash
# Commands the main session runs to validate sub-agent work
[ -f "migrations/001_add_users_table.sql" ] && echo "✅ Migration file exists"
grep -q "CREATE TABLE users" migrations/001_add_users_table.sql && echo "✅ Table creation present"
```
```

#### 1.3 Phase Parallelization Summary

Add to phase headers when applicable:

```markdown
## Phase 2: Database Setup

**Parallelization:** Steps 2.1, 2.2, 2.3 can run concurrently (no dependencies)
**Sequential requirement:** Step 2.4 must wait for 2.1-2.3 completion
```

### Phase 2: Skill Evaluation Logic (Uses Structure)

Add evaluation logic to Mode 2 (Execute Existing Plan) in SKILL.md.

#### 2.1 Decision Criteria

When reading a plan, evaluate:

| Criterion | Check | Threshold |
|-----------|-------|-----------|
| Independent tasks exist | Steps marked `Dependencies: None` in same phase | 2+ tasks |
| Outputs are verifiable | Steps have `Orchestrator Verification` commands | All parallel tasks |
| No file conflicts | Output artifacts don't overlap | Zero conflicts |
| Complexity justifies overhead | Total parallel tasks × avg task complexity | Worth sub-agent spawn cost |

#### 2.2 Workflow Integration

**Mode 1 (Create Execution Plan) - Post-Draft Review Step:**

Add as final step of Mode 1, after draft is complete:

```markdown
7. **Post-Draft Parallelization Review**
   - Plan saved with `status: draft`
   - Evaluate parallelization potential (see decision criteria above)
   - If viable:
     a. Identify which tasks can parallelize
     b. Add dependency/artifact/verification annotations to those steps
     c. Add phase parallelization summaries
     d. Propose strategy to user with rationale
   - If not viable: note in plan why sequential execution is preferred
   - Update status to `ready` after user approval
```

**Rationale for post-draft placement:**
- Parallelization requires seeing the complete dependency graph
- Separates correctness (plan content) from optimization (execution strategy)
- Clean decision point: user reviews draft for correctness, then approves execution strategy

**Mode 2 (Execute Existing Plan) - Parallelization Execution:**

```
1. Read entire plan
2. Check for parallelization annotations
3. If parallel execution approved:
   a. Spawn sub-agents for independent tasks
   b. After sub-agent completion, run orchestrator verification
   c. Report consolidated results before proceeding
4. If sequential: execute as normal
```

#### 2.3 Sub-Agent Task Prompt Template

When spawning sub-agents, use structured prompt:

```markdown
## Task Assignment

**Step:** [Step number and title]
**From Plan:** [Plan file path]

**Actions:**
[Copied from plan step]

**Output Artifacts Required:**
[Copied from plan step]

**Success Criteria:**
[Copied from plan step]

**Important:**
- Complete ONLY this specific step
- Do NOT proceed to other steps
- Report completion with artifact verification
```

### Phase 3: Guide Documentation

Add pattern documentation to `docs-as-code-guide.md`:

- "Sub-Agent Parallelization Pattern" in Critical Patterns section
- When to use vs. when sequential is better
- Anti-patterns (sub-agents for simple tasks, missing verification)

---

## Implementation Order

Following the skill's dependency chain:

| Order | File | Changes |
|-------|------|---------|
| 1 | `docs-as-code-guide.md` | Add Sub-Agent Parallelization pattern |
| 2 | `docs-as-code-execution-plan-template.md` | Add dependency/artifact/verification fields |
| 3 | `SKILL.md` | Add evaluation logic to Mode 2, update Quality Checklist |

---

## Decision Criteria: When NOT to Use Sub-Agents

Sub-agent parallelization adds overhead. Skip when:

- Phase has fewer than 2 independent tasks
- Tasks are simple (< 2 bash commands each)
- Validation requires context sub-agent has but orchestrator lacks
- File outputs overlap (race condition risk)
- User prefers visibility over speed (watching each step)

---

## Open Questions

1. **Sub-agent failure handling:** If one sub-agent fails, should others continue or abort all?
2. **Progress visibility:** How does orchestrator report parallel task progress to user?
3. **Context sharing:** Should sub-agents receive any shared context beyond their specific task?
4. **Retry logic:** If orchestrator verification fails, retry sub-agent or escalate to user?

---

## Success Metrics

- Context window usage reduction for parallelizable plans (target: 30-50% reduction)
- No increase in execution failures due to sub-agent drift
- User can validate all sub-agent work through orchestrator verification commands

---

- **Document Status:** Captured
- **Last Updated:** 2026-01-21
- **Related Docs:** [template-enhancements-backlog-implemented.md](template-enhancements-backlog-implemented.md)
