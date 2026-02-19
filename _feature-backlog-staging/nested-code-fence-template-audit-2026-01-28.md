# Nested Code Fence Rendering Bug - Template Audit

- **Created:** 2026-01-28
- **Status:** Staged
- **Priority:** Medium
- **Origin:** Discovered while creating execution plan for execution-logger-skill-dev project initialization

---

## Problem Statement

The docs-as-code-execution-plan template produces markdown that doesn't render correctly in VS Code Markdown Preview or Obsidian when execution plans contain heredocs with embedded markdown.

### Root Cause

When a bash code block contains a heredoc that itself contains markdown with triple backticks, the markdown parser sees the inner backticks and prematurely closes the outer code fence:

```markdown
```bash
cat > CLAUDE.md <<'EOF'
# Some Header

```                          ← Parser thinks code block ends here
project/
├── file.md
```                          ← This is now orphaned

More content...
EOF
```                          ← Actual intended end
```

### Symptoms

- Bold text (`**text**`) stops rendering after the broken fence
- Headers don't render as headers
- All markdown syntax after the break appears as plain text
- Problem cascades through the rest of the document

### Evidence

Execution plan `2026-01-28-project-init-execution-plan.md` for execution-logger-skill-dev:
- Steps 2.2 and 2.3 contain heredocs that create CLAUDE.md and README.md
- Both heredocs contain markdown with code blocks (directory trees, bash examples)
- Everything after Step 2.2 fails to render correctly

---

## Requested Action

### 1. Audit the Template

Review `references/docs-as-code-execution-plan-template.md` for:

- Any examples showing heredocs with embedded markdown
- Guidance on creating files via heredoc
- Code block fence patterns used

### 2. Audit the Guide

Review `references/docs-as-code-guide.md` for:

- Heredoc examples (especially Pattern 1: File Migration, Pattern 2: Git Workflow)
- Token optimization section on heredocs
- Any embedded markdown in code examples

### 3. Identify All Problem Patterns

Document all locations where:

- Triple backticks appear inside code blocks
- Heredocs are shown containing markdown content
- Nested code structures exist

### 4. Fix with Consistent Pattern

Apply one of these solutions consistently:

**Option A: 4-backtick outer fence**
````bash
cat > file.md <<'EOF'
```
nested code
```
EOF
````

**Option B: Tildes inside heredocs**
```bash
cat > file.md <<'EOF'
~~~
nested code
~~~
EOF
```

**Recommendation:** Option A (4-backtick outer fence) is cleaner because:
- The heredoc content remains valid markdown if extracted
- No need to remember different fence styles for different contexts
- More explicit about the nesting level

### 5. Add Guidance to Template

Add a note in the template about this pattern:

```markdown
**Note on Heredocs with Markdown:** When a bash script creates a file containing
markdown with code blocks, use 4 backticks for the outer fence to prevent
rendering issues. See Pattern X in the guide.
```

---

## Acceptance Criteria

- [ ] Template audited for nested fence issues
- [ ] Guide audited for nested fence issues
- [ ] All problematic patterns identified and documented
- [ ] Fixes applied using consistent pattern (4-backtick recommended)
- [ ] Guidance added to template/guide about heredocs with embedded markdown
- [ ] Test: Create a sample execution plan with heredoc-created markdown files
- [ ] Test: Verify renders correctly in VS Code Markdown Preview
- [ ] Test: Verify renders correctly in Obsidian

---

## Related Files

| File | Issue |
| ---- | ----- |
| `references/docs-as-code-execution-plan-template.md` | May contain problematic patterns |
| `references/docs-as-code-guide.md` | Heredoc examples may have nested fences |
| `SKILL.md` | May reference patterns that cause issues |

---

- **Document Status:** Staged
- **Last Updated:** 2026-01-28
