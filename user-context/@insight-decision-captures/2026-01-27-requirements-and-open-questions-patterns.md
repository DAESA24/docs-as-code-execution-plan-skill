---
doc_type: insight-decision-capture
title: "Feature Backlog Requirements and Open Questions Patterns"
created: 2026-01-27
source: Discussion during deterministic-hook-based-logging backlog item creation
tags:
  - requirements-engineering
  - feature-backlog
  - proof-of-concept
  - decision-making
---

# Feature Backlog Requirements and Open Questions Patterns

## Context

During creation of the `deterministic-hook-based-logging-2026-01-27.md` feature backlog item, Drew and Claude developed patterns for handling requirements and open questions that improve backlog item quality and approval workflow.

## Insight 1: Eliminate SHOULD Requirements

### The Problem

SHOULD requirements in backlog items are problematic because:

- Claude tends to deprioritize or ignore anything marked as optional
- They create ambiguity about what's actually required
- They defer decisions that should be made upfront

### The Solution

Every requirement must be either:

- **MUST** - Required for the feature to be considered complete
- **Removed** - If it's truly optional, it's not a requirement

### Decision Process for SHOULD → MUST or Remove

When evaluating a SHOULD requirement:

1. Ask: "What happens if this requirement is not met?"
2. If the answer involves failure, corruption, or broken functionality → **MUST**
3. If the answer is "nothing bad, just nice to have" → **Remove** (or document as a future enhancement)
4. If the answer is "it's already covered by another requirement" → **Remove** (note why)

### Example from This Session

| Original SHOULD | Analysis | Decision |
|-----------------|----------|----------|
| SHOULD use `once: true` for finalization hook | Without it, Stop hook fires multiple times, corrupting log | **MUST** |
| SHOULD use marker file for log path coordination | This is the mechanism to fulfill the MUST "detect log file location" | **MUST** (made specific) |
| SHOULD support plans created before hook implementation | Already covered by "handle case where no execution plan is active" | **Remove** |

## Insight 2: Requirements Need Justifications

### The Problem

Requirements without justifications:

- Are harder for humans to evaluate ("why is this here?")
- Are easier for Claude to deprioritize ("seems arbitrary")
- Don't capture the reasoning for future reference
- Can't be properly challenged or validated

### The Solution

Use a two-column table format for requirements:

| Requirement | Justification |
|-------------|---------------|
| **MUST** do X | Because Y happens if we don't, which violates Z |

### What Makes a Good Justification

A justification should answer one or more of:

- **What breaks if this isn't met?** (technical consequence)
- **What user problem does this solve?** (business/UX justification)
- **What constraint does this satisfy?** (design constraint)
- **What principle does this uphold?** (architectural consistency)

### Example Justifications

| Requirement | Weak Justification | Strong Justification |
|-------------|-------------------|----------------------|
| MUST use marker file | "It's the best approach" | "Marker file is explicit, debuggable, and persists across Claude responses. Environment variables don't reliably persist, and convention-based discovery is fragile." |
| MUST handle errors gracefully | "Errors are bad" | "Logging is observability, not core functionality. A logging failure (disk full, permission issue) should not prevent the actual plan execution from completing." |

## Insight 3: Categorize Open Questions

### The Problem

Open questions in backlog items are often a mixed bag of:

- Things we don't know if they'll work (technical unknowns)
- Things we need to choose between (design decisions)

Treating them the same creates confusion about what PoC should accomplish and what requires human judgment.

### The Solution

Split open questions into two categories:

**Technical Validations (TV)** - Factual unknowns with pass/fail outcomes

- Can be answered by testing
- Result is objective: "it works" or "it doesn't"
- Informs requirements (may need to adjust if something doesn't work)

**Design Decisions (DD)** - Choices between valid options

- Cannot be answered by testing alone
- PoC gathers evidence; human makes final call
- Result is a choice, not a fact

### How to Categorize

| Question Type | Indicator | Category |
|---------------|-----------|----------|
| "Does X work?" | Yes/no answer possible | Technical Validation |
| "What is X?" | Factual discovery | Technical Validation |
| "Should we do X or Y?" | Both are valid options | Design Decision |
| "How much X is too much?" | Judgment required | Design Decision |

### Example from This Session

| Original Question | Category | Rationale |
|-------------------|----------|-----------|
| Does `$SKILL_DIR` resolve in hook commands? | TV | Factual - either it does or doesn't |
| Does PostToolUseFailure hook fire on errors? | TV | Factual - testable |
| Should we log Read tool calls? | DD | Both logging and not logging are valid |
| How to handle phase/step boundaries? | DD | Multiple valid approaches exist |

## Insight 4: PoC Phases Should Answer Specific Questions

### The Problem

PoC phases often have vague goals like "validate the approach works" without specifying what questions they answer or what success looks like.

### The Solution

Each PoC phase should explicitly list:

1. **Technical Validations Answered** - Which TVs this phase resolves
2. **Design Decision Evidence Gathered** - What data this phase collects for DDs
3. **Success Criteria** - Observable outcomes that indicate pass/fail

### Template for PoC Phase

```markdown
### Phase N: [Name]

**Goal:** [One sentence]

**Scope:**
- [Specific activities]

**Success Criteria:**
- [ ] [Observable outcome 1]
- [ ] [Observable outcome 2]

**Technical Validations Answered:**

| Question | Test | Result |
|----------|------|--------|
| TVn: [Question] | [How to test] | Pending |

**Design Decision Evidence Gathered:**

| Decision | Evidence to Collect |
|----------|---------------------|
| DDn: [Decision] | [What data to gather] |
```

## Insight 5: Approval Gates Create Clear Workflow

### The Problem

Without explicit gates, it's unclear when a backlog item is ready for implementation. This leads to:

- Premature implementation (before questions are answered)
- Stalled items (unclear what's blocking approval)
- Scope creep (requirements change during implementation)

### The Solution

Add explicit approval gates:

1. **After Technical Validations:** "All TVs must have documented results before proceeding to Phase 2"
2. **After Design Decisions:** "All DDs must have Drew's explicit choice recorded before proceeding to Phase 2"
3. **For Full Approval:** "Mark status as 'approved' when all validations documented and decisions made"

### Workflow Visualization

```
[Captured]
    ↓
Run PoC Phase 0 → Answer TV1-TV4
    ↓
Run PoC Phase 1 → Answer TV5-TV6, Gather DD evidence
    ↓
Drew reviews evidence → Makes DD1-DD2 decisions
    ↓
Update requirements based on results
    ↓
[Approved] → Ready for /skill-enhance
```

## Applying These Patterns

### When Creating a New Backlog Item

1. **Write requirements as MUST only** - If it's not a MUST, don't list it as a requirement
2. **Add justification column** - Every MUST needs a "because"
3. **List open questions** - Don't pretend you know everything
4. **Categorize questions** - TV for facts, DD for choices
5. **Design PoC to answer questions** - Map each TV/DD to a PoC phase
6. **Add approval gates** - Make the workflow explicit

### When Reviewing an Existing Backlog Item

1. **Check for SHOULD requirements** - Convert to MUST or remove
2. **Check for missing justifications** - Add them
3. **Check for vague open questions** - Categorize as TV or DD
4. **Check for clear approval criteria** - Add gates if missing

## Why This Matters

### For Drew

- Clear requirements with justifications are easier to review and approve
- Knowing which questions are TVs vs DDs clarifies what needs human judgment
- Explicit approval gates prevent premature implementation

### For Claude

- MUST requirements are unambiguous - no guessing about priority
- Justifications provide context for implementation decisions
- PoC phases with specific questions prevent aimless exploration
- Approval gates provide clear stopping points

### For Future Reference

- Documented justifications explain "why" for future maintainers
- TV/DD split shows what was validated vs. what was chosen
- PoC results capture what was learned

---

- **Document Status:** Complete
- **Applies To:** All feature backlog items in this project
- **Related:** `_feature-backlog-staging/CLAUDE.md`, `deterministic-hook-based-logging-2026-01-27.md`
