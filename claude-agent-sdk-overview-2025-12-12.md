# Claude Agent SDK Research Summary

**Date:** 2025-12-12
**Purpose:** Evaluate Claude Agent SDK for Lando content automation system

---

## 1. What is the Claude Agent SDK?

The Claude Agent SDK is **Anthropic's framework for building production AI agents** that autonomously execute tools and take actions. It's essentially Claude Code (the CLI) available as a library in Python and TypeScript, giving you the same underlying agent capabilities but programmable.

**Key distinction:** Unlike the Claude API (which requires you to implement tool execution loops manually), the Agent SDK handles tool execution, context management, and the agentic loop automatically.

**Available in:**
- Python: `pip install claude-agent-sdk`
- TypeScript/Node.js: `npm install @anthropic-ai/claude-agent-sdk`

---

## 2. Key Features & Capabilities

### Built-in Tools (No Implementation Required)

| Tool Category | Tools |
|--------------|-------|
| **File Ops** | Read, Write, Edit, Glob, Grep, NotebookEdit |
| **Execution** | Bash, Task (spawn subagents) |
| **Web** | WebFetch, WebSearch |
| **Advanced** | MCP tools, Custom tools via @tool decorator |

### Core Agent Features

| Feature | What It Does | Lando Use Case Fit |
|---------|--------------|-------------------|
| **Tool Use** | Claude decides which tools to use based on task | Excellent - LLM decides tool flow |
| **Sessions** | Maintain context across multiple exchanges | Good - content radar outputs can have multi-turn context |
| **Hooks** | Run custom code before/after tool execution | Great - audit logging, validation |
| **MCP Integration** | Connect to external systems/APIs | Critical - Postiz API, image gen, Discord |
| **Permissions** | Control which tools Claude can use | Good - safety guardrails |
| **Subagents** | Spawn specialized agents for subtasks | Useful - content creation pipeline delegation |
| **Streaming** | Real-time message streaming | Good - show progress |
| **Structured Outputs** | JSON schema validation | Useful - ensure consistent output format |

---

## 3. Comparison: Claude Agent SDK vs. Custom discord.py + OpenRouter

### Architecture Differences

| Aspect | Claude Agent SDK | discord.py + OpenRouter Custom |
|--------|------------------|--------------------------------|
| **Tool Execution** | Built-in, automatic | Manual implementation (you write loops) |
| **Context Management** | Automatic with sessions | Manual tracking |
| **Error Handling** | Built-in retry/recovery | You implement |
| **MCP Support** | Native (stdio, HTTP, SSE, SDK) | Would need manual implementation |
| **Deployment** | Production-ready patterns documented | Fully custom, untested patterns |
| **Learning Curve** | Moderate (learn SDK API) | High (learn SDK + implement tool loops) |
| **Time to Market** | Fast (weeks) | Slower (months of integration testing) |
| **Maintenance Burden** | Low (Anthropic maintains SDK) | High (you own all custom code) |

### Feature Comparison Table

| Feature | Agent SDK | Custom Solution |
|---------|-----------|-----------------|
| Listen for Discord triggers | ❌ Must wrap in Discord bot | ✅ Native |
| Process LLM pipeline | ✅ Built-in agentic loop | ✅ Custom implementation |
| Generate images | ✅ Via MCP tools | ✅ Custom MCP or API calls |
| Create Discord threads | ❌ Must add custom code | ✅ Native |
| Postiz API integration | ✅ Via custom MCP tool | ✅ Direct API calls |
| Publish content | ✅ Via custom MCP tool | ✅ Direct API calls |
| Human review workflows | ⚠️ Needs custom layer | ✅ Native Discord integration |

---

## 4. Is Claude Agent SDK Suitable for Lando?

**Requirements:**
1. Listen for triggers (scheduled or manual)
2. Process content ideas through LLM pipeline
3. Generate images
4. Create Discord threads for review
5. Integrate Postiz API for publishing

### Assessment: PARTIAL FIT

**What Works Well:**
- ✅ LLM pipeline + content processing (core strength)
- ✅ Image generation via custom MCP tools
- ✅ Integration with external APIs (Postiz) via MCP
- ✅ Session management for multi-turn workflows
- ✅ Structured outputs for consistent formatting

**What Doesn't Work Well:**
- ❌ Discord bot framework (Agent SDK isn't a Discord bot)
- ❌ Real-time Discord message listening/thread creation
- ⚠️ Scheduled triggers (not built-in, needs external scheduler)
- ⚠️ Human review workflows in Discord (possible but awkward)

**Verdict:** The SDK excels at the AI/processing core, but you'd need to wrap it in Discord infrastructure anyway.

---

## 5. Recommended Hybrid Architecture for Lando

```
┌─────────────────────────────────────────────────────────┐
│                   Discord Bot (discord.py)              │
│  • Listens to triggers (messages, slash commands)       │
│  • Creates/manages threads for review                   │
│  • Handles human feedback                               │
└──────────────────┬──────────────────────────────────────┘
                   │
        (calls Agent SDK processes)
                   │
┌──────────────────▼──────────────────────────────────────┐
│         Claude Agent SDK (Python)                       │
│  • Core LLM agentic loop                                │
│  • Content idea processing                              │
│  • Custom MCP tools:                                    │
│    - Image generation                                   │
│    - Postiz API integration                             │
│    - Content formatting                                 │
│  • Sessions for multi-turn workflows                    │
└──────────────────┬──────────────────────────────────────┘
                   │
        (returns processed content)
                   │
        (Discord bot publishes/reviews)
```

**Why This Works:**
- Discord bot handles real-time, user-facing interactions
- Agent SDK handles heavy lifting (LLM processing, API calls)
- Clean separation of concerns
- Leverage each tool's strengths

---

## 6. Pros & Cons

### Claude Agent SDK Pros
- Powerful agentic loop (tool use, context management, hooks)
- No need to implement tool execution loops
- Built-in security (permissions, sandboxing)
- MCP ecosystem for integrations
- Production-ready deployment patterns
- Session persistence across calls
- Anthropic maintains and updates it
- Type-safe custom tools (@tool decorator)

### Claude Agent SDK Cons
- Not a Discord bot framework
- Scheduled triggers require external orchestration
- Discord thread management not built-in
- Learning curve for SDK-specific patterns
- Locked into Claude models only
- Startup cost ~5¢/hour per container (for hosted deployments)

### Custom discord.py + OpenRouter Pros
- Full control over Discord integration
- Can use any LLM provider (Claude, GPT, local)
- No startup costs (scales to zero on your own infra)
- Familiar Discord framework patterns
- Can optimize for your specific workflow

### Custom discord.py + OpenRouter Cons
- You implement the entire agentic loop (tool use, context management)
- Error handling, retries, recovery all manual
- MCP integration is manual (not built-in)
- Higher maintenance burden
- Harder to test complex multi-step workflows
- No structured output validation
- Session management is your responsibility
- Harder to deploy securely

---

## 7. Code Examples

### Example 1: Agent SDK with Custom Tools

```python
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    tool,
    create_sdk_mcp_server
)
import asyncio

# Define custom tools for your workflow
@tool("generate_image", "Generate image from prompt", {"prompt": str})
async def generate_image(args: dict) -> dict:
    # Call your image generation API
    response = await generate_via_api(args["prompt"])
    return {
        "content": [{
            "type": "text",
            "text": f"Generated image: {response['url']}"
        }]
    }

@tool("publish_to_postiz", "Publish content via Postiz", {
    "content": str,
    "platform": str,
    "scheduled_at": str
})
async def publish_content(args: dict) -> dict:
    # Call Postiz API
    result = await postiz_client.create_post(
        content=args["content"],
        platform=args["platform"],
        scheduled_at=args["scheduled_at"]
    )
    return {
        "content": [{
            "type": "text",
            "text": f"Published to {args['platform']}: {result['post_id']}"
        }]
    }

# Create MCP server with tools
custom_server = create_sdk_mcp_server(
    name="content-tools",
    version="1.0.0",
    tools=[generate_image, publish_content]
)

# Use in your processing pipeline
async def process_content_idea(idea: str):
    options = ClaudeAgentOptions(
        mcp_servers={"content-tools": custom_server},
        allowed_tools=[
            "mcp__content-tools__generate_image",
            "mcp__content-tools__publish_to_postiz"
        ],
        permission_mode="acceptEdits"
    )

    async with ClaudeSDKClient(options=options) as client:
        prompt = f"""
        Take this content idea and:
        1. Refine it into a compelling hook
        2. Generate an accompanying image
        3. Prepare it for social media

        Idea: {idea}
        """

        await client.query(prompt)

        # Collect all messages
        async for message in client.receive_response():
            print(message)
```

### Example 2: Discord Bot Wrapper

```python
import discord
from discord.ext import commands, tasks

bot = commands.Bot(command_prefix="!")

@tasks.loop(hours=1)  # Scheduled trigger
async def generate_daily_content():
    # Get content ideas from your radar system
    ideas = await content_radar.get_today_ideas()

    for idea in ideas:
        # Trigger Agent SDK processing
        processed = await process_content_idea(idea)

        # Find your review channel
        channel = bot.get_channel(REVIEW_CHANNEL_ID)

        # Create thread for this content
        thread = await channel.create_thread(
            name=idea[:100],
            message=discord.Message.create(
                content=f"**Content Idea:** {idea}\n\n**Processed:** {processed['content']}"
            )
        )

        # Add reaction buttons for approval
        await thread.message.add_reaction("✅")  # Approve
        await thread.message.add_reaction("❌")  # Reject
        await thread.message.add_reaction("🔄")  # Revise

@bot.event
async def on_reaction_add(reaction, user):
    if user == bot.user:
        return

    if reaction.emoji == "✅":
        # Approve and publish via Postiz
        thread = reaction.message.channel
        content_id = thread.name  # Or store in DB

        # Call Postiz API to publish
        await postiz_client.publish(content_id)
        await thread.send("Published to social media!")

@generate_daily_content.before_loop
async def before_generate():
    await bot.wait_until_ready()

generate_daily_content.start()
bot.run(DISCORD_TOKEN)
```

---

## 8. Decision Framework

**Choose Agent SDK If:**
- LLM reasoning is complex (multi-step pipelines, tool chains)
- You need structured outputs and type safety
- You want Anthropic maintaining your tool infrastructure
- You're comfortable wrapping it in a Discord bot layer
- You want production-grade error handling/retries
- You plan to deploy to cloud sandboxes (Modal, E2B, etc.)

**Choose Custom discord.py + OpenRouter If:**
- You have simple LLM tasks (single API calls, no chaining)
- You need tight Discord integration as the core
- You want to use multiple LLM providers
- You're deploying to existing Python infrastructure
- You have a small team with deep Discord bot experience
- Cost efficiency is critical (scale to zero)

**Choose Both (Recommended for Lando) If:**
- You need Discord's UI/UX for human review
- You have complex LLM processing pipelines
- You want clean separation of concerns
- You value maintenance/support from Anthropic
- You're building content creation workflows
- You need scheduled + manual triggers

---

## 9. Production Deployment Patterns

| Pattern | Use Case | Cost | Startup |
|---------|----------|------|---------|
| **Ephemeral Sessions** | One-off tasks per trigger | Lowest (~$0.05/task) | Fast |
| **Long-Running Sessions** | Always-on agent | Highest (~$5/hour) | N/A |
| **Hybrid Sessions** | Resume with context | Medium | Medium |
| **Single Container** | Multiple SDK processes | Medium | Medium |

**For Lando:** Hybrid sessions (ephemeral containers hydrated with content radar state, resume when scheduled trigger fires).

**Recommended hosts:**
- Modal Labs (easiest, fully managed)
- Fly.io (Docker, good for long-running)
- E2B (specialized AI sandbox)
- Akash (decentralized, your choice)

---

## 10. Key Resources

- [Agent SDK Overview](https://platform.claude.com/docs/en/agent-sdk/overview.md)
- [Python SDK Reference](https://platform.claude.com/docs/en/agent-sdk/python.md)
- [Custom Tools Guide](https://platform.claude.com/docs/en/agent-sdk/custom-tools.md)
- [Hosting Guide](https://platform.claude.com/docs/en/agent-sdk/hosting.md)
- [MCP Integration](https://platform.claude.com/docs/en/agent-sdk/mcp.md)

---

## 11. Components Needed for Lando

Based on the PRD, these SDK components will be required:

1. **Core SDK Client** - ClaudeSDKClient initialization
2. **Custom MCP Tools** - @tool decorator for:
   - `fetch_and_summarize` (URL scraping)
   - `generate_image` (Kie.ai integration)
   - `generate_voice` (ElevenLabs integration)
   - `search_stock_footage` (Pexels API)
   - `assemble_video` (MoviePy)
   - `publish_to_postiz` (Postiz API)
3. **Sessions** - For multi-turn content refinement
4. **Structured Outputs** - For consistent content formatting
5. **Permissions** - To control tool access
6. **Hooks** - For logging and validation
7. **Deployment** - Akash hosting configuration

*Each component needs detailed research - see separate research docs.*
