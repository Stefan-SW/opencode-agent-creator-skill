# Agent Validation Checklist

Use this checklist to validate an agent file before deployment. The agent can read the target file and verify each item.

---

## How to Use

When validating an agent, read the agent file and check each item below. Mark issues as:

- **Error** - Must fix before use
- **Warning** - Should fix for quality
- **Info** - Optional improvement

---

## 1. File Structure

### Frontmatter Presence

- [ ] File starts with `---`
- [ ] Has closing `---` after YAML block
- [ ] YAML parses without errors
- [ ] Content exists after frontmatter

**Error if:** No frontmatter or invalid YAML syntax

---

## 2. Required Fields

### description (Required)

- [ ] Field exists and is a string
- [ ] Length is 20-1024 characters
- [ ] Contains trigger keywords ("Use when...", "Use for...", "Invoke when...")
- [ ] Contains at least one `<example>` block

**Validation:**

```yaml
# Good
description: >-
  Reviews code for security vulnerabilities.

  Use when asked to audit code, check for security issues,
  or review authentication implementations.

  <example>
  User: "Check this login code for vulnerabilities"
  Assistant: "I'll use the security-auditor agent."
  </example>

# Bad - too short, no triggers, no examples
description: Security helper
```

---

## 3. Optional Fields

### mode

- [ ] If present, value is one of: `primary`, `subagent`, `all`
- [ ] Default is `all` if not specified

**Validation:**

```yaml
# Valid
mode: subagent

# Invalid
mode: helper  # Error: not a valid mode
```

### tools (deprecated)

> **Warning:** The `tools` field is deprecated. Use `permission` instead.

- [ ] If present, flag as deprecated and suggest migrating to `permission`

### permission

- [ ] If present, is a dictionary/object
- [ ] Keys are valid permission targets: `edit`, `bash`, `webfetch`, `skill`, `task`
- [ ] Simple values are one of: `allow`, `ask`, `deny`
- [ ] Pattern dictionaries have string keys (glob patterns) and valid permission level values
- [ ] `write` is **not** a valid key — file writes are controlled by `edit`
- [ ] For `bash`, `skill`, and `task`: `"*"` wildcard comes first, specific rules after (last match wins)

**Validation:**

```yaml
# Good - simple permissions
permission:
  edit: ask
  bash: deny

# Good - pattern-based bash
permission:
  bash:
    "*": ask
    "git *": allow
    "rm -rf *": deny

# Bad
permission:
  write: ask       # Error: not a valid key; use edit:
  bash: maybe      # Error: invalid level
  edit:
    "*": sometimes # Error: invalid level
```

### model

- [ ] If present, is a string
- [ ] Format is `provider/model-id` (e.g., `github-copilot/claude-sonnet-4.6`)

### temperature

- [ ] If present, is a number
- [ ] Value is between 0.0 and 1.0

### steps

- [ ] If present, is an integer
- [ ] Value is at least 1
- [ ] `maxSteps` is deprecated — flag and suggest renaming to `steps`

### hidden

- [ ] If present, is a boolean
- [ ] Only meaningful when `mode: subagent`

---

## 4. Deprecated Fields

These fields should NOT be present in new agents:

| Field         | Status     | Replacement                           |
| ------------- | ---------- | ------------------------------------- |
| `tools`       | Deprecated | Use `permission` instead              |
| `maxSteps`    | Deprecated | Use `steps` instead                   |
| `name`        | Deprecated | Name comes from filename              |
| `skills`      | Deprecated | Load skills at runtime via skill tool |
| `permissions` | Renamed    | Use `permission` (singular)           |

**Error if:** `name`, `skills`, or `permissions` present  
**Warning if:** `tools` or `maxSteps` present — migrate to current equivalents

---

## 5. Permission Safety Patterns

### Check for Anti-Patterns

- [ ] Not all permissions set to `allow` (permission promiscuity)
- [ ] If `bash` is not `deny`, has pattern-based controls
- [ ] `edit: allow` is intentional and documented
- [ ] Dangerous bash commands (`rm`, `dd`, `mkfs`) are explicitly `deny`
- [ ] `"*"` wildcard is first in pattern maps, specific rules follow

**Warning if:**

- `bash: allow` without patterns
- `edit: allow` on an agent that shouldn't write files
- No `deny` rules for destructive commands when bash is enabled

---

## 6. Body Content

### Structure

- [ ] Has content after frontmatter
- [ ] Uses `##` headings for organization
- [ ] Has at least 3 sections

### Recommended Sections

- [ ] Role definition (first paragraph)
- [ ] Core Responsibilities
- [ ] Workflow or Process
- [ ] Limitations (what agent CANNOT do)

### Safety (if bash enabled)

- [ ] Contains safety keywords: ALWAYS, NEVER, verify, confirm, backup, check
- [ ] Has at least 3 safety-related instructions

---

## 7. Quick Validation Summary

```markdown
## Validation Report for [agent-name]

### Errors (Must Fix)

- [ ] ...

### Warnings (Should Fix)

- [ ] ...

### Info (Optional)

- [ ] ...

### Result

[ ] PASS - No errors
[ ] FAIL - Has errors that must be fixed
```

---

## Example Validation

When asked to validate an agent, output a report like:

```markdown
## Validation Report: code-reviewer.md

### Errors

None

### Warnings

1. Description could include more trigger keywords

### Info

1. Consider adding error handling section

### Result: PASS

Agent is valid and ready for use.
```
