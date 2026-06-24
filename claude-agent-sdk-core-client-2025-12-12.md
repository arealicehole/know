# Claude Agent SDK Core Client - Comprehensive Guide

**Date:** 2025-12-12
**Source:** Official Claude Platform Documentation

---

## 1. Installation & Setup

### Prerequisites
- Python 3.10+
- Anthropic account
- Claude Code CLI installed

### Installation Steps

```bash
# Install Claude Code CLI
curl -fsSL https://claude.ai/install.sh | bash

# Install Python SDK
pip install claude-agent-sdk

# Set API Key
echo "ANTHROPIC_API_KEY=your-api-key" > .env
```

---

## 2. Two Approaches: query() vs ClaudeSDKClient

### query() - One-off Tasks
- Single, independent interactions
- No conversation history needed
- Fresh start each time

### ClaudeSDKClient - Continuous Conversations
- Multi-turn conversations with context
- Following up on previous responses
- Using hooks and custom tools
- Session management

---

## 3. ClaudeAgentOptions - Complete Reference

```python
@dataclass
class ClaudeAgentOptions:
    allowed_tools: list[str]           # Whitelist tools
    disallowed_tools: list[str]        # Blacklist tools
    system_prompt: str | None          # Custom instructions
    model: str | None                  # claude-opus-4-5, etc.
    permission_mode: str | None        # default, acceptEdits, bypassPermissions
    continue_conversation: bool        # Multi-turn
    resume: str | None                 # Resume session by ID
    fork_session: bool                 # Branch session
    max_turns: int | None              # Limit turns
    mcp_servers: dict                  # External MCP servers
    cwd: str | Path | None             # Working directory
    hooks: dict | None                 # Pre/post hooks
    output_format: dict | None         # Structured outputs
```

---

## 4. Basic Usage Patterns

### Pattern 1: Simple Query
```python
async for message in query(
    prompt="Find all TODO comments",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    print(message)
```

### Pattern 2: ClaudeSDKClient
```python
async with ClaudeSDKClient(options=options) as client:
    await client.query("First question")
    async for msg in client.receive_response():
        print(msg)

    await client.query("Follow-up")  # Claude remembers context
    async for msg in client.receive_response():
        print(msg)
```

### Pattern 3: Resume Session
```python
async for message in query(
    prompt="Continue from before",
    options=ClaudeAgentOptions(resume=session_id)
):
    print(message)
```

---

## 5. Error Handling

```python
from claude_agent_sdk import (
    CLINotFoundError,      # Claude Code not installed
    CLIConnectionError,    # Can't connect
    ProcessError,          # Process failed
    CLIJSONDecodeError     # JSON parse failed
)

try:
    async for message in query(prompt="Hello"):
        print(message)
except CLINotFoundError:
    print("Install Claude Code first")
except ProcessError as e:
    print(f"Process failed: {e.exit_code}")
```

---

## 6. Context Management

- Sessions automatically manage message history
- Files read are cached in session context
- Tool results are remembered
- Use `max_turns` to limit context growth
- Use `fork_session=True` to branch

---

## Key Takeaways

| Concept | Details |
|---------|---------|
| Installation | Claude Code CLI + `pip install claude-agent-sdk` |
| One-off tasks | Use `query()` function |
| Multi-turn | Use `ClaudeSDKClient` class |
| Sessions | Capture from SystemMessage with `subtype == 'init'` |
| Resuming | Use `resume=session_id` option |
