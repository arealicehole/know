# Claude Agent SDK Permissions - Comprehensive Guide

**Date:** 2025-12-12
**Source:** Official Claude Platform Documentation

---

## 1. Permission Modes

| Mode | Description |
|------|-------------|
| `default` | Standard permission checks |
| `acceptEdits` | Auto-approve file edits |
| `bypassPermissions` | Bypass ALL checks (use with caution) |
| `plan` | Read-only mode |

```python
options = ClaudeAgentOptions(
    permission_mode="acceptEdits"
)
```

---

## 2. allowed_tools & disallowed_tools

```python
options = ClaudeAgentOptions(
    # Whitelist - ONLY these tools available
    allowed_tools=[
        'Read', 'Grep', 'Glob', 'WebFetch'
    ],

    # Blacklist - Block specific tools
    disallowed_tools=[
        'Bash', 'Write', 'Edit'
    ]
)
```

**Note:** Use `disallowed_tools` (blacklist) - more reliable than `allowed_tools`.

---

## 3. Available Tools

**Read-Only:**
- Read, Glob, Grep, WebFetch, WebSearch

**Write/Execute:**
- Write, Edit, Bash, NotebookEdit

**Agent Tools:**
- Task, Agent

---

## 4. canUseTool Callback

Runtime permission handler:

```python
async def permission_handler(tool_name, input_params, options):
    # Block filesystem writes
    if tool_name in ['Write', 'Edit', 'Bash']:
        return {
            "behavior": "deny",
            "message": f"{tool_name} is not allowed"
        }

    # Allow image generation
    if tool_name == 'ImageGenerator':
        return {"behavior": "allow", "updatedInput": input_params}

    # Ask user for anything else
    return {"behavior": "ask"}

options = ClaudeAgentOptions(
    can_use_tool=permission_handler
)
```

---

## 5. Permission Rules (.claude/settings.json)

```json
{
  "permissions": {
    "allow": [
      "Read(/workspace/creative/**)",
      "WebFetch(domain:postiz.com)",
      "WebFetch(domain:api.kie.ai)"
    ],
    "deny": [
      "Write(*)",
      "Edit(*)",
      "Bash(rm *)",
      "Bash(sudo *)"
    ]
  }
}
```

---

## 6. For Lando Content Automation

```python
options = ClaudeAgentOptions(
    permission_mode='default',

    # Block dangerous tools
    disallowed_tools=[
        'Bash', 'Write', 'Edit', 'KillBash', 'NotebookEdit'
    ],

    # Allow safe tools + custom integrations
    allowed_tools=[
        'Read', 'Grep', 'Glob', 'WebFetch', 'WebSearch', 'Task'
    ],

    # Custom validation
    can_use_tool=async (tool_name, input, options) => {
        if tool_name == 'Task':
            subagent = input.subagent_type
            if subagent in ['image-generator', 'postiz-publisher']:
                return {"behavior": "allow", "updatedInput": input}
            return {"behavior": "deny"}
        return {"behavior": "allow", "updatedInput": input}
    }
)
```

---

## Security Best Practices

1. **Principle of Least Privilege** - Deny by default, allow only needed
2. **Validate Tool Inputs** - Check URLs, file paths, content
3. **Use Permission Rules** - Static policies in settings.json
4. **Audit & Logging** - Log all tool usage via hooks
5. **Never use bypassPermissions** - In production with untrusted inputs
