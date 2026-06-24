# Claude Agent SDK Custom Tools - Comprehensive Guide

**Date:** 2025-12-12
**Source:** Official Claude Platform Documentation

---

## 1. @tool Decorator Syntax

```python
from claude_agent_sdk import tool
from typing import Any

@tool(
    name: str,              # Tool identifier
    description: str,       # What it does
    input_schema: dict      # Parameter definitions
)
async def my_tool(args: dict[str, Any]) -> dict[str, Any]:
    return {
        "content": [{
            "type": "text",
            "text": "Result here"
        }]
    }
```

---

## 2. Input Schema Examples

### Simple Type Mapping
```python
@tool("fetch_url", "Fetch URL content", {
    "url": str,
    "max_length": int,
    "language": str
})
async def fetch_url(args: dict[str, Any]) -> dict[str, Any]:
    url = args["url"]
    language = args.get("language", "english")  # Optional with default
```

### JSON Schema (Complex)
```python
@tool("generate_image", "Generate image", {
    "type": "object",
    "properties": {
        "prompt": {"type": "string"},
        "size": {"type": "string", "enum": ["small", "medium", "large"]},
        "count": {"type": "integer", "minimum": 1, "maximum": 4}
    },
    "required": ["prompt", "size"]
})
```

---

## 3. Return Values

### Success
```python
return {
    "content": [{
        "type": "text",
        "text": "Your response here"
    }]
}
```

### Error
```python
return {
    "content": [{
        "type": "text",
        "text": "Error message"
    }],
    "is_error": True
}
```

---

## 4. create_sdk_mcp_server

Bundle tools into an MCP server:

```python
from claude_agent_sdk import tool, create_sdk_mcp_server

@tool("fetch_url", "Fetch URL", {"url": str})
async def fetch_url(args): ...

@tool("generate_image", "Generate image", {"prompt": str})
async def generate_image(args): ...

# Bundle into server
content_server = create_sdk_mcp_server(
    name="content-automation",
    version="1.0.0",
    tools=[fetch_url, generate_image]
)

# Use in client
options = ClaudeAgentOptions(
    mcp_servers={"content": content_server},
    allowed_tools=[
        "mcp__content__fetch_url",
        "mcp__content__generate_image"
    ]
)
```

---

## 5. Example: Postiz Publishing Tool

```python
@tool(
    "publish_to_postiz",
    "Publish content to Postiz",
    {"content": str, "platforms": list, "schedule_time": str}
)
async def publish_to_postiz(args: dict[str, Any]) -> dict[str, Any]:
    content = args["content"]
    platforms = args["platforms"]

    # Validation
    if not content or len(content) < 10:
        return {
            "content": [{"type": "text", "text": "Content too short"}],
            "is_error": True
        }

    # API call
    async with aiohttp.ClientSession() as session:
        async with session.post(
            "https://api.postiz.com/v1/posts",
            json={"content": content, "platforms": platforms},
            headers={"Authorization": f"Bearer {os.environ['POSTIZ_API_KEY']}"}
        ) as response:
            if response.status != 200:
                return {
                    "content": [{"type": "text", "text": f"API error: {response.status}"}],
                    "is_error": True
                }

            result = await response.json()
            return {
                "content": [{
                    "type": "text",
                    "text": f"Published to {', '.join(platforms)}"
                }]
            }
```

---

## Best Practices

1. **Naming**: Use `verb_noun` format (e.g., `fetch_url`, `generate_image`)
2. **Descriptions**: Be comprehensive about what the tool does
3. **Validation**: Always validate inputs before processing
4. **Error handling**: Return `is_error: True` on failures
5. **Async**: All tools should be async for non-blocking operations
6. **Environment variables**: Never hardcode API keys
