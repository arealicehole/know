# Claude Agent SDK Sessions - Comprehensive Guide

**Date:** 2025-12-12
**Source:** Official Claude Platform Documentation

---

## 1. What Are Sessions?

Sessions are persistent conversation contexts that:
- Maintain state across multiple interactions
- Handle message history automatically
- Track files read, analysis performed, code executed
- Support resumption without restarting

---

## 2. Creating Sessions

Sessions are created automatically. Capture the ID from init message:

```python
async for message in query(prompt="Generate content"):
    if hasattr(message, 'subtype') and message.subtype == 'init':
        session_id = message.data.get('session_id')
        print(f"Session: {session_id}")
```

---

## 3. Multi-Turn Conversations

```python
async with ClaudeSDKClient(options=options) as client:
    # Turn 1: Generate
    await client.query("Create a LinkedIn post about AI")
    async for msg in client.receive_response():
        print(msg)

    # Turn 2: Revise (Claude remembers)
    await client.query("Make the hook more attention-grabbing")
    async for msg in client.receive_response():
        print(msg)

    # Turn 3: Approve
    await client.query("Confirm it's ready for publishing")
    async for msg in client.receive_response():
        print(msg)
```

---

## 4. Resuming Sessions

```python
# Resume existing session
async for message in query(
    prompt="Continue from where we left off",
    options=ClaudeAgentOptions(resume=session_id)
):
    print(message)
```

---

## 5. Session Storage Pattern

```python
class ContentSessionManager:
    def __init__(self, sessions_file="sessions.json"):
        self.sessions_file = Path(sessions_file)
        self.sessions = self._load_sessions()

    def save_session(self, content_id: str, session_id: str, metadata: dict):
        self.sessions[content_id] = {
            "session_id": session_id,
            "created_at": metadata.get("created_at"),
            "status": metadata.get("status", "draft")
        }
        self._save_sessions()

    def get_session(self, content_id: str) -> str | None:
        return self.sessions.get(content_id, {}).get("session_id")
```

---

## 6. Forking Sessions

Create a branch from existing session:

```python
async for message in query(
    prompt="Try a different approach",
    options=ClaudeAgentOptions(
        resume=session_id,
        fork_session=True  # Creates new branch
    )
):
    print(message)
```

---

## Use Case: Content Creation Workflow

1. **Generate** initial content → capture session_id
2. **Store** session_id with content metadata
3. **Resume** for revisions → Claude remembers original
4. **Final approval** → all context retained

This eliminates manual context management and prevents context loss between steps.
