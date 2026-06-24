# Claude Agent SDK Hooks - Comprehensive Guide

**Date:** 2025-12-12
**Source:** Official Claude Platform Documentation

---

## 1. What Are Hooks?

Hooks are pre/post tool execution callbacks that allow you to:
- Log tool usage for auditing
- Validate inputs before execution
- Modify tool behavior
- Block dangerous operations
- Track metrics and analytics

---

## 2. Available Hook Types

```python
HookEvent = Literal[
    "PreToolUse",        # Before tool execution
    "PostToolUse",       # After tool execution
    "UserPromptSubmit",  # When user submits prompt
    "Stop",              # When stopping execution
    "SubagentStop",      # When subagent stops
    "PreCompact"         # Before message compaction
]
```

---

## 3. Hook Registration

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher

async def my_hook(input_data, tool_use_id, context):
    # Your logic here
    return {}  # Return empty to allow execution

options = ClaudeAgentOptions(
    hooks={
        'PreToolUse': [
            HookMatcher(
                matcher='Bash',  # Match specific tool
                hooks=[my_hook],
                timeout=120      # Timeout in seconds
            )
        ]
    }
)
```

---

## 4. Hook Decisions

### Block a Tool
```python
async def security_hook(input_data, tool_use_id, context):
    if input_data['tool_name'] == 'Bash':
        command = input_data['tool_input'].get('command', '')
        if 'rm -rf' in command:
            return {
                'hookSpecificOutput': {
                    'hookEventName': 'PreToolUse',
                    'permissionDecision': 'deny',
                    'permissionDecisionReason': 'Dangerous command blocked'
                }
            }
    return {}  # Allow
```

### Modify Input
```python
async def path_redirect_hook(input_data, tool_use_id, context):
    if input_data['tool_name'] == 'Write':
        return {
            'hookSpecificOutput': {
                'hookEventName': 'PreToolUse',
                'updatedInput': {
                    **input_data['tool_input'],
                    'file_path': f"./sandbox/{input_data['tool_input']['file_path']}"
                }
            }
        }
    return {}
```

---

## 5. Use Cases

### Audit Logging
```python
async def audit_hook(input_data, tool_use_id, context):
    log_entry = {
        "timestamp": datetime.now().isoformat(),
        "tool": input_data.get('tool_name'),
        "session": input_data.get('session_id')
    }
    with open("audit.jsonl", 'a') as f:
        f.write(json.dumps(log_entry) + '\n')
    return {}
```

### Rate Limiting
```python
api_call_times = defaultdict(list)

async def rate_limit_hook(input_data, tool_use_id, context):
    tool = input_data.get('tool_name')
    if tool not in ["WebFetch", "WebSearch"]:
        return {}

    now = datetime.now()
    key = f"{tool}:{input_data.get('session_id')}"

    # Remove old entries
    api_call_times[key] = [t for t in api_call_times[key] if (now - t).seconds < 60]

    if len(api_call_times[key]) >= 10:
        return {
            'hookSpecificOutput': {
                'hookEventName': 'PreToolUse',
                'permissionDecision': 'deny',
                'permissionDecisionReason': 'Rate limit exceeded'
            }
        }

    api_call_times[key].append(now)
    return {}
```

### Content Validation
```python
async def validate_content_hook(input_data, tool_use_id, context):
    if input_data['tool_name'] != 'Write':
        return {}

    content = input_data['tool_input'].get('content', '')

    if len(content) < 100:
        return {
            'hookSpecificOutput': {
                'hookEventName': 'PreToolUse',
                'permissionDecision': 'deny',
                'permissionDecisionReason': 'Content too short'
            }
        }
    return {}
```

---

## Key Notes

- All hooks are async
- Default timeout: 60 seconds
- Return `{}` to allow execution
- Return denial decision to block
- Hooks have full access to tool input and session context
