# PoC Phase 0 Results Summary

- **Execution Date:** 2026-01-28
- **Status:** Complete with Blocker
- **Executed By:** Claude Opus 4.5

---

## Executive Summary

**Skill-bundled hooks cannot provide always-on logging.** They only fire when the skill is explicitly invoked, not on every tool call. This is a fundamental architecture constraint documented in official Claude Code documentation.

---

## Key Finding

### Skill-Bundled Hooks Are Scoped to Skill Lifecycle

From official documentation:

> "These hooks are scoped to the component's lifecycle and only run when that component is active."

**What this means:**
- Installing a skill with hooks at `~/.claude/skills/` does NOT make hooks run automatically
- Skills must be invoked (via `/skill-name` or automatic context-based invocation) for hooks to fire
- For always-on hooks, must use settings-based hooks in `.claude/settings.json`

---

## Technical Validations Summary

| ID | Question | Result |
|----|----------|--------|
| TV1 | Does `$SKILL_DIR` resolve? | NOT SUPPORTED - Not documented |
| TV2 | Does PostToolUseFailure work in skills? | NOT SUPPORTED - Only PreToolUse, PostToolUse, Stop |
| TV3 | stdin JSON schema? | Could not test - hooks never fired |
| TV4 | Windows path issues? | Could not test - hooks never fired |

---

## Implications for Hook-Based Logging Feature

### Current Approach: BLOCKED

The proposed approach of packaging execution logging as a skill with embedded hooks will not work.

### Alternative Approaches

1. **Settings-based hooks** - Deploy hooks via `.claude/settings.json` instead of SKILL.md
   - Pro: Works for always-on logging
   - Con: Requires modifying user/project settings, not a self-contained skill

2. **Hybrid approach** - Skill provides scripts, user configures hooks in settings
   - Pro: Skill still packages the logic
   - Con: More complex setup for users

3. **Abandon hook-based logging** - Use different mechanism
   - Pro: Avoids hook complexity
   - Con: Need to find alternative approach

---

## Recommendation

**Pivot to settings-based hooks** if always-on logging is required. The skill can still package:
- Hook scripts (in `scripts/` directory)
- Documentation for setup
- Templates for settings.json configuration

But the hooks themselves must be registered in settings files, not SKILL.md frontmatter.

---

## Documentation References

- [Hooks Reference](https://code.claude.com/docs/en/hooks.md)
- [Hooks in Skills](https://code.claude.com/docs/en/hooks.md#hooks-in-skills-and-agents)
- [Skills Documentation](https://code.claude.com/docs/en/skills.md)

---

- **Related Plan:** `2026-01-28-hook-poc-phase0-execution-plan.md`
- **Feature Backlog:** `_feature-backlog-staging/deterministic-hook-based-logging-2026-01-27.md`
