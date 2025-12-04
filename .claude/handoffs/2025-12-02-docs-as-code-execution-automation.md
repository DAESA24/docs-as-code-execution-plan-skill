# Docs-as-Code Execution Automation: Skill + Slash Command

- **Created:** 2025-12-02 11:30
- **Purpose:** Create a skill and slash command to automate the docs-as-code LLM execution pattern
- **Status:** Pickup completed

## Primary Request and Intent

The user wants to automate the recurring docs-as-code LLM execution pattern used for infrastructure projects. This pattern has been used successfully across multiple projects:
- OpenMemory v1.2.1 upgrade
- Ollama startup fixes (v1, v2, v3)
- Duplicati upgrade

The goal is to create **both**:
1. A **skill** for the full create → execute → archive workflow
2. A **slash command** for quick template scaffolding

## Key Technical Concepts

- **Docs-as-Code:** Documentation that IS the implementation (pre-written bash scripts, not instructions)
- **Read-Once Architecture:** All context embedded upfront, no file exploration during execution
- **Explicit Success Criteria:** Programmatically verifiable conditions per step
- **Minimal Approval Points:** Only for destructive/irreversible operations (marked with 🚨)
- **Built-In Validation:** Check → Execute → Verify pattern
- **Status Prefixes:** ✅/❌/⚠️/⏭️/ℹ️ for LLM parsing
- **Dev Agent Record:** Lessons learned section for procedural knowledge extraction

## Files and Code Sections

### The Guide (Source of Truth)
`user-context/2025-11-17-docs-as-code-llm-execution-guide.md`

Key structure from guide:
```markdown
# [Operation Name] Execution Plan

- **Purpose:** [1-sentence goal]
- **Audience:** Claude Code (autonomous execution agent)
- **User Intervention:** Only for steps marked 🚨
- **Estimated Duration:** [X minutes]
- **Version:** 1.0

---

## Pre-Flight Validation
[bash script with error handling]

## Phase 1: [Phase Name]
**Autonomous:** YES/NO
[bash script]
**Success Criteria:** [verifiable condition]

## Rollback Plan
[bash script]
```

### Example Execution Plans (Templates)
- `user-context/2025-12-02-ollama-startup-fix-v3-plan.md` - Iterative fix (7 phases, reference example)

### Skill Creator Reference (MUST REVIEW FIRST)
`C:\Users\drewa\.claude\skills\skill-creator` - Best practices for creating skills

## Problem Solving

### Pattern Analysis Completed

Every execution plan follows this structure:
1. **Metadata Header** - Created, Status, Agent, Execution Mode, Duration, User Intervention
2. **Test Plan Summary** - Table with phases, test methods, success indicators, results
3. **Problem Statement / Overview**
4. **Risk Assessment** - Table with likelihood, impact, mitigation
5. **Pre-Flight Validation** - Bash script
6. **Numbered Phases** - Each with Autonomous flag, bash scripts, success criteria
7. **Rollback Procedure**
8. **Completion Checklist** - Checkboxes
9. **Acceptance Criteria** - Checkboxes
10. **Dev Agent Record** - Execution notes, issues encountered, procedural learnings
11. **Footer Metadata**

### Automation Approach Decided

**Hybrid approach agreed upon:**

| Component | Purpose | When to Use |
|-----------|---------|-------------|
| Skill | Full workflow (create → execute → archive) | "Help me create and execute an execution plan for X" |
| Slash Command | Quick template scaffolding | Just need the structure fast |

## Pending Tasks

- [x] Review the skill-creator skill at `C:\Users\drewa\.claude\skills\skill-creator`
- [x] Review docs-as-code guide and example execution plan
- [x] Create docs-as-code execution plan for building the skill
- [ ] Execute the plan to create `docs-as-code-execution-plan` skill
- [ ] Create the `/execution-plan` slash command
- [ ] Test both with a sample task
- [ ] Consider OpenMemory integration for procedural learnings

**Execution Plan:** `docs/2025-12-03-infrastructure-skill-creation-execution-plan.md`

## Current Work

Analysis phase complete. Ready to move to implementation. The user explicitly wants the skill-creator reviewed BEFORE writing the new skill to ensure best practices are followed.

## Next Steps

1. **Review skill-creator skill** at `C:\Users\drewa\.claude\skills\skill-creator` - understand the structure, conventions, and best practices
2. **Create the skill** (`infrastructure-execution` or similar name) following the patterns learned
3. **Create the slash command** for quick template generation
4. **Test** with a real or hypothetical infrastructure task

## Important Context

### User's Skill Location
Skills are stored at: `C:\Users\drewa\.claude\skills\`

### Key Files to Reference During Implementation
- Guide: `user-context/2025-11-17-docs-as-code-llm-execution-guide.md`
- Example plan: `user-context/2025-12-02-ollama-startup-fix-v3-plan.md`

### OpenMemory Integration Idea
The procedural learnings from execution plans could be stored via:
```bash
opm add "Learning text here" --tags "procedural,infrastructure"
```
This would allow recalling past learnings when creating future plans.

### Windows-Specific Considerations
All bash scripts use Git Bash paths (`/c/Users/drewa/...`) and Windows-specific commands (`taskkill`, `schtasks.exe`, `powershell.exe`).

---

- **Document Status:** Ready for pickup
- **Workspace:** `C:\Users\drewa\work\dev\cc-skills-dev\docs-as-code-execution-plan-skill`
