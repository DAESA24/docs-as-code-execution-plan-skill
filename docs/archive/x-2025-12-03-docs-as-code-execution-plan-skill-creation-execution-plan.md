---
title: Docs-as-Code Execution Plan Skill Creation
created: 2025-12-03
status: complete
agent: dev
execution_mode: docs-as-code
user_intervention: minimal
tags:
  - skill-creation
  - docs-as-code
  - automation
  - meta
---

# Docs-as-Code Execution Plan Skill Creation Plan

- **Purpose:** Create a skill and slash command to automate docs-as-code LLM execution plan workflows
- **Audience:** Claude Code (autonomous execution agent)
- **User Intervention:** Only for steps marked 🚨
- **Estimated Duration:** 30 minutes

---

## Test Plan Summary

| Phase | Test Method | Success Indicator | Result |
|-------|-------------|-------------------|--------|
| Pre-Flight | Run bash script | All ✅ messages, no ❌ errors | ✅ |
| Phase 1 | Check directory structure | Skill directory created with SKILL.md | ✅ |
| Phase 2 | Read SKILL.md | Contains complete workflow instructions | ✅ |
| Phase 3 | Check references/ | Guide and template files exist | ✅ |
| Phase 4 | Check commands/ | Slash command file exists | ✅ |
| Phase 5 | Run validation script | quick_validate.py passes | ✅ |
| Phase 6 | Trigger skill | Skill responds to trigger phrases | ✅ |
| Phase 7 | Check README | README.md exists in dev directory with usage docs | ✅ |

---

## Problem Statement

The docs-as-code LLM execution pattern has been used successfully across multiple infrastructure projects:

- OpenMemory v1.2.1 upgrade (10 phases)
- Ollama startup fixes v1, v2, v3 (iterative refinement)
- Duplicati upgrade

### Current State

Creating execution plans requires manually:

1. Recalling the pattern structure from memory
2. Copying from previous plans
3. Adapting the template each time

No automated tooling exists to scaffold or guide the process.

### Target State

A Claude Code skill and slash command that:

- Automates plan creation with proper structure
- Guides execution with the docs-as-code pattern
- Supports archiving with lessons learned extraction

### Solution

Create a **skill** that:
1. Guides the full workflow (problem analysis → plan creation → execution → archive)
2. Embeds the pattern knowledge for consistent, high-quality plans
3. Includes YAML front matter for machine-parseable metadata

Create a **slash command** that:
1. Quickly scaffolds a new execution plan template
2. Pre-fills YAML front matter with sensible defaults
3. Enables fast iteration without full skill invocation

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Skill not triggering | Low | Medium | Clear description with trigger phrases |
| Template too rigid | Medium | Low | Make template sections optional/flexible |
| Guide too large for context | Low | Medium | Use references/ for detailed guide |
| Slash command conflicts | Very Low | Low | Use unique command name |

---

## Pre-Flight Validation

**Autonomous:** YES

```bash
echo "=== PRE-FLIGHT CHECKS ==="
echo ""

# Define paths
SKILL_CREATOR="/c/Users/drewa/.claude/skills/skill-creator"
SKILLS_DIR="/c/Users/drewa/.claude/skills"
COMMANDS_DIR="/c/Users/drewa/.claude/commands"
GUIDE_SOURCE="/c/Users/drewa/work/dev/cc-skills-dev/docs-as-code-execution-plan-skill/user-context/2025-11-17-docs-as-code-llm-execution-guide.md"
EXAMPLE_PLAN="/c/Users/drewa/work/dev/cc-skills-dev/docs-as-code-execution-plan-skill/user-context/2025-12-02-ollama-startup-fix-v3-plan.md"

# 1. Check skill-creator exists
if [ -d "$SKILL_CREATOR" ]; then
    echo "✅ skill-creator found: $SKILL_CREATOR"
else
    echo "❌ ERROR: skill-creator not found"
    exit 1
fi

# 2. Check init_skill.py exists
if [ -f "$SKILL_CREATOR/scripts/init_skill.py" ]; then
    echo "✅ init_skill.py exists"
else
    echo "❌ ERROR: init_skill.py not found"
    exit 1
fi

# 3. Check skills directory is writable
if [ -d "$SKILLS_DIR" ] && [ -w "$SKILLS_DIR" ]; then
    echo "✅ Skills directory writable: $SKILLS_DIR"
else
    echo "❌ ERROR: Skills directory not accessible"
    exit 1
fi

# 4. Check commands directory exists
if [ -d "$COMMANDS_DIR" ]; then
    echo "✅ Commands directory exists: $COMMANDS_DIR"
else
    echo "ℹ️ Commands directory not found, will create"
    mkdir -p "$COMMANDS_DIR"
    echo "✅ Created commands directory"
fi

# 5. Check source guide exists
if [ -f "$GUIDE_SOURCE" ]; then
    echo "✅ Docs-as-code guide found"
else
    echo "❌ ERROR: Guide not found at $GUIDE_SOURCE"
    exit 1
fi

# 6. Check example plan exists
if [ -f "$EXAMPLE_PLAN" ]; then
    echo "✅ Example execution plan found"
else
    echo "⚠️ Example plan not found (optional)"
fi

# 7. Check if skill already exists
if [ -d "$SKILLS_DIR/docs-as-code-execution-plan" ]; then
    echo "⚠️ Skill already exists - will be overwritten"
else
    echo "✅ No existing skill (fresh install)"
fi

echo ""
echo "✅ Pre-flight checks passed"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- skill-creator directory and init_skill.py exist
- Skills and commands directories accessible
- Source guide file exists

---

## Phase 1: Initialize Skill Directory

**Autonomous:** YES

```bash
echo ""
echo "=== PHASE 1: INITIALIZE SKILL DIRECTORY ==="
echo ""

SKILL_CREATOR="/c/Users/drewa/.claude/skills/skill-creator"
SKILLS_DIR="/c/Users/drewa/.claude/skills"
SKILL_NAME="docs-as-code-execution-plan"

# Run init_skill.py
cd "$SKILL_CREATOR" || exit 1

python3 scripts/init_skill.py "$SKILL_NAME" --path "$SKILLS_DIR"

# Verify creation
if [ -d "$SKILLS_DIR/$SKILL_NAME" ]; then
    echo ""
    echo "✅ Skill directory created"
    echo ""
    echo "Contents:"
    ls -la "$SKILLS_DIR/$SKILL_NAME/"
else
    echo "❌ ERROR: Skill directory not created"
    exit 1
fi

echo ""
echo "✅ Phase 1 Complete - Skill Directory Initialized"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- Directory `~/.claude/skills/docs-as-code-execution-plan/` exists
- Contains SKILL.md template
- Contains example resource directories

---

## Phase 2: Write SKILL.md Content

**Autonomous:** YES

This phase replaces the generated SKILL.md with the actual skill content.

### SKILL.md Content

```markdown
---
name: docs-as-code-execution-plan
description: This skill automates the docs-as-code LLM execution pattern for complex, multi-step tasks. Use this skill when creating execution plans for development projects (upgrades, migrations, feature implementations, automation setup), when executing existing execution plans, or when archiving completed plans with lessons learned. Trigger phrases include "create an execution plan", "help me plan this change", "create a docs-as-code plan", "execute this plan", or "archive this execution plan".
---

# Docs-as-Code Execution Plan Skill

This skill implements the docs-as-code pattern for creating and executing automation plans for complex, multi-step tasks.

## When to Use This Skill

- Creating execution plans for complex changes (upgrades, migrations, feature implementations)
- Executing existing docs-as-code plans autonomously
- Archiving completed plans with lessons learned
- Converting ad-hoc development work into repeatable patterns

## Core Principles

The docs-as-code pattern follows five critical principles:

1. **Pre-Written Scripts** - Write complete bash scripts, not instructions. The LLM executes, doesn't improvise.
2. **Read-Once Architecture** - Include all context upfront. No file exploration during execution.
3. **Explicit Success Criteria** - Every step has programmatically verifiable success conditions.
4. **Minimal Approval Points** - Require approval only for destructive/irreversible operations (marked 🚨).
5. **Built-In Validation** - Check → Execute → Verify pattern for every critical operation.

## Workflow Modes

### Mode 1: Create Execution Plan

To create a new execution plan:

1. **Gather Requirements**
   - Understand the infrastructure change goal
   - Identify current state and target state
   - List all systems/services affected
   - Identify potential risks and rollback needs

2. **Generate Plan Structure**
   - Use the template in `references/execution-plan-template.md`
   - Fill YAML front matter with project metadata
   - Define phases (typically 5-10 for complex changes)
   - Write pre-flight validation script
   - Write bash scripts for each phase
   - Define success criteria per phase
   - Include rollback procedure

3. **Save Plan**
   - Save to project's `docs/` folder
   - Follow the file naming convention (see below)

## File Naming Convention

All execution plans follow this naming pattern:

```
YYYY-MM-DD-<topic-words>-execution-plan[-vN].md
```

### Rules

| Element | Rule |
|---------|------|
| **Case** | kebab-case (lowercase, hyphens) |
| **Date prefix** | `YYYY-MM-DD` (today's date) |
| **Topic words** | 3-4 words max describing the goal |
| **Suffix** | Always ends with `-execution-plan` |
| **Versioning** | `-v2`, `-v3`, etc. only if previous version exists |

### Examples

```
2025-12-03-ollama-startup-fix-execution-plan.md
2025-12-03-ollama-startup-fix-execution-plan-v2.md
2025-12-03-database-migration-setup-execution-plan.md
2025-12-03-openmemory-mcp-integration-execution-plan.md
```

### Version Detection Logic

When creating a new plan, check for existing files with the same topic:

```bash
# Generate base filename
base_name="YYYY-MM-DD-topic-words-execution-plan"
docs_dir="docs"

# Check for existing files
existing=$(ls "$docs_dir"/${base_name}*.md 2>/dev/null)

if [ -z "$existing" ]; then
    # No existing file → use base name
    filename="${base_name}.md"
else
    # Find highest version
    highest_v=$(ls "$docs_dir"/${base_name}*.md | grep -oE 'v[0-9]+' | sort -V | tail -1)
    if [ -z "$highest_v" ]; then
        # Base exists without version → next is v2
        filename="${base_name}-v2.md"
    else
        # Increment version
        next_v=$((${highest_v#v} + 1))
        filename="${base_name}-v${next_v}.md"
    fi
fi
```

### Mode 2: Execute Existing Plan

To execute an existing execution plan:

1. **Read the entire plan** before starting
2. **Run Pre-Flight Validation** - Stop if any check fails
3. **Execute phases in order**
   - Mark autonomous phases (`**Autonomous:** YES`) - execute without confirmation
   - Pause at approval points (🚨) - wait for user confirmation
4. **Verify success criteria** after each phase
5. **Update plan status** in YAML front matter as phases complete
6. **Handle failures** - Run rollback if needed, document in Dev Agent Record

### Mode 3: Archive Completed Plan

To archive a completed execution plan:

1. **Update YAML front matter** - Set `status: complete` or `status: failed`
2. **Complete Dev Agent Record section**
   - Execution date and duration
   - Issues encountered and resolutions
   - Procedural learnings for future reference
3. **Move to archives** - `docs/archives/` folder
4. **Extract procedural learnings** to OpenMemory (if available):
   ```bash
   opm add "Learning text" --tags "procedural,infrastructure,<project>"
   ```

## Status Prefixes

Use these prefixes in bash script output for LLM parsing:

- `✅` - Success
- `❌` - Error (stop execution)
- `⚠️` - Warning (continue with caution)
- `ℹ️` - Information
- `⏭️` - Skipped (step not needed)
- `🚨` - User approval required

## Windows-Specific Patterns

All scripts use Git Bash paths:
- ✅ `/c/Users/drewa/...` (correct)
- ❌ `C:\Users\drewa\...` (wrong)

For Windows-specific commands, use:
- `schtasks.exe` for Task Scheduler
- `powershell.exe -Command "..."` for PowerShell
- `taskkill /F /IM process.exe` for process termination

## Reference Materials

For detailed guidance, refer to:
- `references/execution-plan-template.md` - Complete template with all sections
- `references/docs-as-code-guide.md` - Full pattern documentation

## Quality Checklist

Before executing any plan, verify:

- [ ] YAML front matter complete (title, created, status, agent, etc.)
- [ ] Pre-flight validation script covers all prerequisites
- [ ] Each phase has `**Autonomous:** YES/NO` marker
- [ ] Each phase has explicit success criteria
- [ ] Approval points (🚨) only for destructive operations
- [ ] Rollback procedure included
- [ ] Dev Agent Record section present (empty, to be filled)
```

**Success Criteria:**
- SKILL.md contains complete workflow instructions
- Description includes trigger phrases
- All three workflow modes documented
- References to bundled resources included

---

## Phase 3: Create Reference Files

**Autonomous:** YES

### Step 3.1: Create Execution Plan Template

Create `references/execution-plan-template.md`:

```markdown
---
title: <Operation Name>
created: <YYYY-MM-DD>
status: draft  # draft | in-progress | complete | failed | rolled-back
agent: dev
execution_mode: docs-as-code
estimated_duration: <Xmin>
user_intervention: minimal  # none | minimal | moderate | heavy
tags:
  - <tag1>
  - <tag2>
---

# <Operation Name> Execution Plan

- **Purpose:** <1-sentence goal>
- **Audience:** Claude Code (autonomous execution agent)
- **User Intervention:** Only for steps marked 🚨
- **Estimated Duration:** <X minutes>

---

## Test Plan Summary

| Phase | Test Method | Success Indicator | Result |
|-------|-------------|-------------------|--------|
| Pre-Flight | Run bash script | All ✅ messages | ⏳ |
| Phase 1 | <method> | <indicator> | ⏳ |
| Phase 2 | <method> | <indicator> | ⏳ |
| ... | ... | ... | ⏳ |

---

## Problem Statement

<Describe the problem being solved>

### Current State
<What exists now>

### Target State
<What should exist after execution>

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| <risk1> | Low/Medium/High | Low/Medium/High | <mitigation> |
| <risk2> | Low/Medium/High | Low/Medium/High | <mitigation> |

---

## Pre-Flight Validation

**Autonomous:** YES

```bash
echo "=== PRE-FLIGHT CHECKS ==="

# Define paths
# <define all paths used in the plan>

# Check 1: <prerequisite>
if [ <condition> ]; then
    echo "✅ <check passed>"
else
    echo "❌ ERROR: <check failed>"
    exit 1
fi

# Check 2: <prerequisite>
# ...

echo ""
echo "✅ Pre-flight checks passed"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- <list verifiable conditions>

---

## Phase 1: <Phase Name>

**Autonomous:** YES

```bash
echo ""
echo "=== PHASE 1: <PHASE NAME> ==="
echo ""

# <implementation>

echo ""
echo "✅ Phase 1 Complete - <summary>"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- <verifiable condition>

---

## Phase 2: <Phase Name>

**Autonomous:** YES/NO

<!-- If NO, include: -->
### 🚨 USER APPROVAL REQUIRED

**What will happen:**
- <destructive action description>

**Safety measures:**
- ✅ <backup verified>
- ✅ <rollback available>

**Reversibility:** <how to undo>

Proceed? (yes/no)

```bash
# <implementation>
```

**Success Criteria:**
- <verifiable condition>

---

## Rollback Procedure

```bash
echo "=== ROLLBACK PROCEDURE ==="

# Undo operations in reverse order
# <step 1>
# <step 2>

echo "✅ Rollback complete"
```

---

## Completion Checklist

- [ ] All phases executed successfully
- [ ] All success criteria verified
- [ ] No rollback required
- [ ] Documentation updated (if applicable)

---

## Acceptance Criteria

- [ ] <primary goal achieved>
- [ ] <secondary goal achieved>
- [ ] <no regressions introduced>

---

## Dev Agent Record

- **Executed By:** <agent name>
- **Execution Date:** <YYYY-MM-DD HH:MM>
- **Duration:** <actual duration>

### Execution Notes

<Summary of execution>

### Issues Encountered

| Issue | Resolution |
|-------|------------|
| <issue> | <resolution> |

### Procedural Learnings

1. <learning for future reference>
2. <learning for future reference>

---

- **Document Status:** <status>
- **Last Updated:** <YYYY-MM-DD>
- **Related Docs:** <links>
```

### Step 3.2: Copy Docs-as-Code Guide

```bash
echo ""
echo "=== PHASE 3: CREATE REFERENCE FILES ==="
echo ""

SKILL_DIR="/c/Users/drewa/.claude/skills/docs-as-code-execution-plan"
GUIDE_SOURCE="/c/Users/drewa/work/dev/cc-skills-dev/docs-as-code-execution-plan-skill/user-context/2025-11-17-docs-as-code-llm-execution-guide.md"

# Create references directory if not exists
mkdir -p "$SKILL_DIR/references"

# Copy the guide
cp "$GUIDE_SOURCE" "$SKILL_DIR/references/docs-as-code-guide.md"

if [ -f "$SKILL_DIR/references/docs-as-code-guide.md" ]; then
    echo "✅ Copied docs-as-code guide to references/"
else
    echo "❌ ERROR: Failed to copy guide"
    exit 1
fi

# The template will be created via Write tool (already specified above)
echo "ℹ️ Template will be written via Edit tool"

echo ""
echo "✅ Phase 3 Complete - Reference Files Created"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- `references/docs-as-code-guide.md` exists
- `references/execution-plan-template.md` exists

---

## Phase 4: Create Slash Command

**Autonomous:** YES

Create a slash command for quick template scaffolding.

### Command File: `~/.claude/commands/execution-plan.md`

```markdown
# Execution Plan Template

Scaffolds a new docs-as-code execution plan with YAML front matter and standard structure.

## Arguments

$ARGUMENTS - Operation name (3-4 words, e.g., "ollama startup fix" or "database migration setup")

## Instructions

Generate a new execution plan file following these steps:

### Step 1: Determine Filename

Follow the naming convention:
- Format: `YYYY-MM-DD-<topic-words>-execution-plan[-vN].md`
- Use today's date as prefix
- Convert arguments to kebab-case (lowercase, hyphens)
- Always end with `-execution-plan`
- Check for existing files with same topic - if found, increment version (v2, v3, etc.)

### Step 2: Check for Existing Plans

Before creating the file, check if a plan with the same topic already exists:

```bash
# Example: If topic is "ollama startup fix"
base_name="$(date +%Y-%m-%d)-ollama-startup-fix-execution-plan"
existing=$(ls docs/${base_name}*.md 2>/dev/null)
# If exists, use -v2, -v3, etc.
```

### Step 3: Generate the Plan

Create the file with:
1. **YAML Front Matter** - title, created (today), status: draft, agent: dev, etc.
2. **All standard sections** from the docs-as-code pattern
3. **Placeholder content** for user to fill in

### Step 4: Save Location

Save to the current project's `docs/` folder.

## Examples

| Input | Output Filename |
|-------|-----------------|
| `ollama startup fix` | `2025-12-03-ollama-startup-fix-execution-plan.md` |
| `database migration` | `2025-12-03-database-migration-execution-plan.md` |
| (same topic, v2) | `2025-12-03-database-migration-execution-plan-v2.md` |

If no arguments provided, ask the user:
1. What is this execution plan for? (3-4 words describing the goal)
2. Brief description of the problem being solved?

Then generate the plan using the template from the docs-as-code-execution-plan skill.
```

```bash
echo ""
echo "=== PHASE 4: CREATE SLASH COMMAND ==="
echo ""

COMMANDS_DIR="/c/Users/drewa/.claude/commands"

# Create commands directory if not exists
mkdir -p "$COMMANDS_DIR"

# The command file will be written via Write tool
echo "ℹ️ Slash command will be written via Write tool"

# Verify directory
if [ -d "$COMMANDS_DIR" ]; then
    echo "✅ Commands directory ready: $COMMANDS_DIR"
else
    echo "❌ ERROR: Commands directory not accessible"
    exit 1
fi

echo ""
echo "✅ Phase 4 Complete - Slash Command Location Ready"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- `~/.claude/commands/execution-plan.md` exists
- Command includes argument handling
- Command references the skill template

---

## Phase 5: Validate Skill

**Autonomous:** YES

```bash
echo ""
echo "=== PHASE 5: VALIDATE SKILL ==="
echo ""

SKILL_CREATOR="/c/Users/drewa/.claude/skills/skill-creator"
SKILL_DIR="/c/Users/drewa/.claude/skills/docs-as-code-execution-plan"

cd "$SKILL_CREATOR" || exit 1

# Run validation (part of package_skill.py)
python3 scripts/quick_validate.py "$SKILL_DIR"

if [ $? -eq 0 ]; then
    echo ""
    echo "✅ Skill validation passed"
else
    echo ""
    echo "❌ Skill validation failed - check errors above"
    exit 1
fi

# List final structure
echo ""
echo "Final skill structure:"
echo "────────────────────────────────────────"
find "$SKILL_DIR" -type f | head -20
echo "────────────────────────────────────────"

echo ""
echo "✅ Phase 5 Complete - Skill Validated"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- `quick_validate.py` exits with code 0
- No validation errors reported

---

## Phase 6: Test Skill Trigger

**Autonomous:** PARTIAL

### Manual Test

After skill is installed, test with these trigger phrases:

1. "Help me create an execution plan for upgrading Node.js"
2. "I need to plan a database migration"
3. "Create a docs-as-code plan for this infrastructure change"

Expected behavior:
- Skill should trigger and offer to help create an execution plan
- Should reference the template and guide
- Should ask clarifying questions about the operation

```bash
echo ""
echo "=== PHASE 6: TEST READINESS ==="
echo ""

SKILL_DIR="/c/Users/drewa/.claude/skills/docs-as-code-execution-plan"
COMMANDS_DIR="/c/Users/drewa/.claude/commands"

echo "Skill Installation Check:"
echo ""

# Check skill exists
if [ -f "$SKILL_DIR/SKILL.md" ]; then
    echo "   ✅ SKILL.md exists"
else
    echo "   ❌ SKILL.md missing"
fi

# Check references
if [ -f "$SKILL_DIR/references/docs-as-code-guide.md" ]; then
    echo "   ✅ docs-as-code-guide.md exists"
else
    echo "   ❌ docs-as-code-guide.md missing"
fi

if [ -f "$SKILL_DIR/references/execution-plan-template.md" ]; then
    echo "   ✅ execution-plan-template.md exists"
else
    echo "   ❌ execution-plan-template.md missing"
fi

# Check slash command
if [ -f "$COMMANDS_DIR/execution-plan.md" ]; then
    echo "   ✅ /execution-plan command exists"
else
    echo "   ❌ /execution-plan command missing"
fi

echo ""
echo "════════════════════════════════════════"
echo "Ready for manual testing:"
echo "   1. Start new Claude Code session"
echo "   2. Say: 'Help me create an execution plan for X'"
echo "   3. Verify skill triggers"
echo "   4. Test /execution-plan slash command"
echo "════════════════════════════════════════"
```

**Success Criteria:**
- All files present
- Skill triggers on test phrases
- Slash command scaffolds template correctly

---

## Phase 7: Create User Documentation

**Autonomous:** YES

Create a README.md in the dev project directory with user documentation for the skill.

### README.md Content

The README should include:

1. **Overview** - What the skill does and why it exists
2. **Installation** - Where the skill is installed and how to verify
3. **Usage** - How to trigger the skill (trigger phrases, slash command)
4. **Workflow Modes** - Create, Execute, Archive with examples
5. **File Naming Convention** - How execution plans are named
6. **Reference Files** - What's included in the skill's references/
7. **Related Files** - Links to the source guide and example plans

```bash
echo ""
echo "=== PHASE 7: CREATE USER DOCUMENTATION ==="
echo ""

DEV_DIR="/c/Users/drewa/work/dev/cc-skills-dev/docs-as-code-execution-plan-skill"

# The README will be written via Write tool
echo "ℹ️ README.md will be written via Write tool"

# Verify dev directory
if [ -d "$DEV_DIR" ]; then
    echo "✅ Dev directory accessible: $DEV_DIR"
else
    echo "❌ ERROR: Dev directory not accessible"
    exit 1
fi

echo ""
echo "✅ Phase 7 Complete - User Documentation Created"
echo "════════════════════════════════════════"
```

**Success Criteria:**

- `README.md` exists in dev project root
- Contains installation verification steps
- Documents all three workflow modes
- Includes trigger phrase examples

---

## Rollback Procedure

```bash
echo "=== ROLLBACK PROCEDURE ==="
echo ""

SKILL_DIR="/c/Users/drewa/.claude/skills/docs-as-code-execution-plan"
COMMANDS_DIR="/c/Users/drewa/.claude/commands"

# Remove skill
if [ -d "$SKILL_DIR" ]; then
    rm -rf "$SKILL_DIR"
    echo "✅ Removed skill directory"
else
    echo "⏭️ Skill directory already absent"
fi

# Remove slash command
if [ -f "$COMMANDS_DIR/execution-plan.md" ]; then
    rm "$COMMANDS_DIR/execution-plan.md"
    echo "✅ Removed slash command"
else
    echo "⏭️ Slash command already absent"
fi

echo ""
echo "✅ Rollback complete"
```

---

## Completion Checklist

- [x] Skill directory created at `~/.claude/skills/docs-as-code-execution-plan/`
- [x] SKILL.md contains complete workflow instructions
- [x] `references/docs-as-code-guide.md` copied from source
- [x] `references/docs-as-code-execution-plan-template.md` created with YAML front matter
- [x] Slash command created at `~/.claude/commands/execution-plan.md`
- [x] Skill passes validation
- [x] Manual trigger test successful
- [x] README.md created in dev project directory with user documentation

---

## Acceptance Criteria

- [x] Skill triggers on phrases like "create a docs-as-code execution plan"
- [x] Generated plans include YAML front matter
- [x] Generated plans follow docs-as-code structure
- [x] Slash command scaffolds template correctly
- [x] All three workflow modes (create, execute, archive) documented

---

## Dev Agent Record

- **Executed By:** Claude Code (Opus 4.5)
- **Execution Date:** 2025-12-03 ~15:30 (initial), 2025-12-04 ~12:00-13:00 (refinements + manual test)
- **Duration:** ~15 minutes initial execution + ~20 minutes refinements + ~30 minutes manual testing

### Execution Notes

All phases completed successfully. Skill is fully functional.

**Phase-by-phase summary:**

1. **Pre-Flight:** All checks passed (skill-creator, directories, source files)
2. **Phase 1:** `init_skill.py` had Unicode encoding issue on Windows (cp1252 codec). Resolved by setting `PYTHONIOENCODING=utf-8`. Directory created but SKILL.md was empty.
3. **Phase 2:** Wrote complete SKILL.md content via Write tool (overwriting empty file from Phase 1)
4. **Phase 3:** Copied docs-as-code-guide.md and created execution-plan-template.md in references/
5. **Phase 4:** Created /execution-plan slash command
6. **Phase 5:** `quick_validate.py` passed - skill is valid
7. **Phase 6:** Manual trigger test successful (see details below)
8. **Phase 7:** Created README.md in dev project directory

**Post-execution refinements (2025-12-04 morning):**

1. Renamed execution plan file: `infrastructure-skill-creation` → `docs-as-code-execution-plan-skill-creation`
2. Updated SKILL.md trigger phrases to require "docs-as-code" in request (avoid triggering on generic "create an execution plan")
3. Renamed template file: `execution-plan-template.md` → `docs-as-code-execution-plan-template.md`
4. Updated all references to template in SKILL.md and README.md
5. Removed `estimated_duration` from template (LLM execution times are unpredictable)
6. Re-ran Phase 5 and 6 to validate changes - both passed

**Manual Trigger Test (2025-12-04 ~12:30):**

Tested in separate project: `openmemory-implementation-project`

Test scenario: Create execution plan for OpenMemory Dashboard bugfix PR contribution

**Test Results:**

| Test | Result | Notes |
|------|--------|-------|
| Trigger phrase detection | ✅ Pass | "create a docs-as-code execution plan for this project" correctly triggered skill |
| Skill invocation prompt | ✅ Expected | User prompted "Do you want to proceed with Skill?" - acceptable UX |
| Template reading | ✅ Pass | Skill read `references/docs-as-code-execution-plan-template.md` |
| Context integration | ✅ Pass | Incorporated conversation context (bug details, file paths, repo info) |
| Plan structure | ✅ Pass | Generated 733-line plan with all required sections |
| YAML frontmatter | ✅ Pass | Included title, created, status, agent, execution_mode, tags |
| Pre-written bash scripts | ✅ Pass | Real executable scripts, not instructions |
| Approval markers | ✅ Pass | Only Phase 1 (fork) and Phase 8 (PR) marked as requiring approval |
| Success criteria | ✅ Pass | Each phase has specific, verifiable criteria |
| File naming | ✅ Pass | `2025-12-04-openmemory-dashboard-bugfix-pr-execution-plan.md` |
| Write location | ✅ Pass | Saved to project's `docs/` folder |
| Post-creation summary | ✅ Pass | Displayed plan overview table with phase descriptions |

**Generated Plan Quality:**

The skill produced a high-quality execution plan that:

- Correctly structured 8 phases + pre-flight + rollback
- Included comprehensive pre-flight validation (10 checks)
- Wrote real bash scripts for autonomous phases
- Used status prefixes correctly (✅/❌/⚠️/ℹ️/⏭️)
- Added Risk Assessment table with realistic mitigations
- Included proper Dev Agent Record section (empty, to be filled)
- Referenced actual file paths from the conversation context

**Issue Found During Test:**

| Issue | Impact | Recommendation |
|-------|--------|----------------|
| Glob search approval prompt | Minor UX friction | Add `~/.claude/skills/**` to global settings with Glob + Read permissions |

The skill tried to read its own template via Glob, which triggered an approval prompt because the skills directory is outside the project. This is a global settings issue, not a skill defect.

### Issues Encountered

| Issue | Resolution |
|-------|------------|
| `init_skill.py` Unicode error (rocket emoji) | Set `PYTHONIOENCODING=utf-8` environment variable |
| SKILL.md created empty due to arrow character encoding | Overwrote with complete content via Write tool in Phase 2 |
| Original trigger phrases too generic | Updated description to require "docs-as-code" in trigger phrases |
| Template filename not specific enough | Renamed to `docs-as-code-execution-plan-template.md` |
| Glob search approval prompt in test | Recommendation: Add `~/.claude/skills/**` to global Claude Code settings |

### Procedural Learnings

1. **Windows Unicode handling:** Python scripts using emojis/special characters need `PYTHONIOENCODING=utf-8` on Windows with Git Bash
2. **init_skill.py workaround:** If template generation fails, the directory is still created - proceed to write SKILL.md content manually
3. **Execution plan review:** Review plan sections thoroughly before execution to catch placeholder content that may look incomplete when rendered
4. **Skill validation:** Use `quick_validate.py` rather than `package_skill.py` for validation-only checks
5. **Trigger phrase specificity:** When a skill could conflict with other workflows (e.g., multiple types of execution plans), make trigger phrases specific enough to avoid false triggers
6. **Time estimates:** Avoid including `estimated_duration` in execution plans - LLM execution speed varies unpredictably and estimates designed for human work don't apply
7. **Naming consistency:** Use the full pattern name (e.g., "docs-as-code-execution-plan") in filenames to avoid ambiguity with other execution plan types
8. **Global skill permissions:** Skills that read their own reference files need `~/.claude/skills/**` added to global Claude Code settings (Glob + Read) to avoid approval prompts when used in other projects
9. **Manual test in separate project:** Always test skills in a different project than where they were developed to catch cross-project permission issues
10. **Context integration quality:** A well-designed skill can incorporate rich context from the conversation (file paths, technical details, repo info) into generated artifacts

---

- **Document Status:** ✅ Complete
- **Last Updated:** 2025-12-04
- **Related Docs:**
  - [docs-as-code-guide](../user-context/2025-11-17-docs-as-code-llm-execution-guide.md)
  - [handoff](../.claude/handoffs/2025-12-02-docs-as-code-execution-automation-handoff.md)
  - [README](../README.md)
  - [Generated test plan](C:\Users\drewa\work\dev\local-infrastrructure-projects\openmemory-implementation-project\docs\2025-12-04-openmemory-dashboard-bugfix-pr-execution-plan.md)
