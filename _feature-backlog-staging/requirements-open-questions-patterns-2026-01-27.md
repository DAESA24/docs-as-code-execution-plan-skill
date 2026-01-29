---
doc_type: feature-backlog-item
title: "Apply Requirements and Open Questions Patterns to Backlog Workflow"
created: 2026-01-27
status: captured
feature_type_tags:
  - process
  - documentation
  - requirements-engineering
placeholder_for: user-context/@insight-decision-captures/2026-01-27-requirements-and-open-questions-patterns.md
---

# Apply Requirements and Open Questions Patterns to Backlog Workflow

## Summary

This is a **placeholder** pointing to an insight capture document that defines patterns for improving feature backlog items.

**Full document:** [2026-01-27-requirements-and-open-questions-patterns.md](../user-context/@insight-decision-captures/2026-01-27-requirements-and-open-questions-patterns.md)

## Key Patterns Documented

1. **Eliminate SHOULD Requirements** - Every requirement must be MUST or removed; SHOULD creates ambiguity and gets ignored

2. **Requirements Need Justifications** - Two-column table format where each MUST has an explanation of why it's required

3. **Categorize Open Questions** - Split into:
   - **Technical Validations (TV)** - Factual unknowns with pass/fail outcomes (answered by PoC)
   - **Design Decisions (DD)** - Choices between valid options (human decides after evidence)

4. **PoC Phases Answer Specific Questions** - Each phase explicitly lists which TVs it answers and what DD evidence it gathers

5. **Approval Gates Create Clear Workflow** - Explicit gates between PoC completion, decisions, and approval

## When Ready to Implement

When Drew is ready to formalize these patterns into the backlog workflow:

1. Read the full insight capture document
2. Decide which patterns to adopt
3. Update `_feature-backlog-staging/CLAUDE.md` with new guidance
4. Update `_frontmatter-template.md` if schema changes needed
5. Mark this placeholder as implemented

## Origin

Developed during creation of `deterministic-hook-based-logging-2026-01-27.md` backlog item. The discussion about SHOULD requirements and how to handle open questions led to these reusable patterns.

---

- **Document Status:** Placeholder - awaiting decision to formalize
- **Last Updated:** 2026-01-27
- **Related:** `deterministic-hook-based-logging-2026-01-27.md` (where patterns were first applied)
