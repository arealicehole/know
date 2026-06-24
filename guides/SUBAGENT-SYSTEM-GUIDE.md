# Comprehensive Guide to Claude Code Subagent Systems

**Version:** 1.0
**Date:** 2025-11-09
**Purpose:** Reference documentation for designing, implementing, and optimizing subagent architectures in Claude Code

---

## Table of Contents

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [Architecture Patterns](#architecture-patterns)
4. [Creating Subagents](#creating-subagents)
5. [Configuration Reference](#configuration-reference)
6. [Design Principles](#design-principles)
7. [Performance Optimization](#performance-optimization)
8. [Real-World Examples](#real-world-examples)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Source References](#source-references)

---

## Introduction

Subagent systems enable Claude Code to decompose complex tasks into specialized, parallel workflows. Each subagent operates in an isolated context with custom instructions and selective tool access, enabling:

- **Parallel Execution:** 3-5x performance improvements through concurrent task processing
- **Specialized Expertise:** Domain-specific agents with tailored system prompts
- **Context Isolation:** Prevents context pollution and extends session longevity
- **Tool Restrictions:** Enhanced security and focus through granular permissions

### When to Use Subagents

✅ **Use subagents for:**
- Multi-step workflows requiring different capabilities
- Tasks that can be parallelized (research, content generation, testing)
- Operations requiring specialized context or tool sets
- Complex orchestration across multiple domains

❌ **Avoid subagents for:**
- Simple, single-step tasks
- Operations requiring shared context state
- Rapid back-and-forth interactions
- Tasks where latency overhead outweighs benefits

---

## Core Concepts

### Subagent Anatomy

A subagent consists of:

1. **Identity** - Unique name and description for invocation
2. **System Prompt** - Specialized instructions defining behavior
3. **Tool Access** - Restricted or full permission set
4. **Model Selection** - Performance/cost optimization (Haiku/Sonnet/Opus)
5. **Context Window** - Isolated from parent conversation

### Execution Flow

```
User Query
    ↓
Main Agent (Orchestrator)
    ↓
Delegates to Subagents (Parallel/Sequential)
    ↓
Subagents Execute in Isolated Contexts
    ↓
Results Return to Orchestrator
    ↓
Orchestrator Synthesizes & Responds
```

### Context Isolation Benefits

- **Fresh Context:** Each subagent starts with clean context, preventing pollution
- **Parallel Safety:** Multiple agents can run simultaneously without interference
- **Focused Instructions:** Specialized prompts without irrelevant history
- **Extended Sessions:** Main conversation doesn't grow with subagent work

---

## Architecture Patterns

### Pattern 1: Sequential (v1.0 - Deep)

```
Orchestrator → Plan → Search1 → Search2 → Search3 → Synthesis
```

**Characteristics:**
- Simple linear workflow
- Each step waits for previous completion
- Good for dependent tasks
- 2-4x slower than parallel approaches

**Use Cases:**
- Small research tasks (1-3 queries)
- Debugging/validation workflows
- Processes requiring strict ordering

---

### Pattern 2: Parallel (Wide)

```
Orchestrator → Plan → [Search1, Search2, Search3, Search4] (parallel) → Synthesis
```

**Characteristics:**
- Spawns multiple agents in single Task call
- All execute simultaneously
- Results collected when complete
- 2-4x faster than sequential

**Use Cases:**
- Independent search queries
- Parallel content generation
- Concurrent testing/validation

---

### Pattern 3: Wide-then-Deep (v2.0 Hybrid - Recommended)

```
Orchestrator → Plan →
    Batch 1: [Search1, Search2, Search3, Search4] → Mini-Synthesis A
    Batch 2: [Search5, Search6, Search7, Search8] → Mini-Synthesis B
    Sequential: [Search9] → Mini-Synthesis C
    → Final Synthesis (combines A, B, C)
```

**Characteristics:**
- Parallelizes independent searches
- Hierarchical synthesis reduces bottleneck
- Batches of 4-6 agents per group
- 3-5x faster with maintained quality

**Use Cases:**
- Complex research (8+ queries)
- Large-scale content generation
- Production multi-agent workflows

**Performance Metrics:**
| Task Complexity | Sequential | Parallel | Wide-then-Deep | Speedup |
|-----------------|-----------|----------|----------------|---------|
| Simple (3 searches) | 2 min | 1.5 min | 1 min | 2x |
| Moderate (8 searches) | 6 min | 3 min | 2 min | 3x |
| Complex (15 searches) | 12 min | 5 min | 3.5 min | 3.4x |

---

### Pattern 4: Parallel Generation (Wizard Pattern)

```
Orchestrator → Discovery → Strategy → Design Analysis →
    [Agent1, Agent2, ..., Agent10] (parallel) → Collection & Integration
```

**Characteristics:**
- 10+ agents spawned simultaneously
- Each generates 5+ outputs independently
- Shared design system/guidelines
- ~10x faster than sequential generation

**Use Cases:**
- Landing page generation
- Test suite creation
- Documentation generation
- Content scaling workflows

**Example:** 10 agents × 5 pages = 50 pages generated concurrently

---

## Creating Subagents

### Method 1: CLI Wizard (Recommended for Beginners)

```bash
# Access agent management interface
/agents

# Select "Create New Agent"
# Choose scope: Project (.claude/agents/) or User (~/.claude/agents/)
# Describe purpose: "Execute research searches quickly with WebSearch"
# Select tools: WebSearch, WebFetch
# Select model: haiku
# Save & customize with 'e' if needed
```

---

### Method 2: Manual Creation (Recommended for Experienced Users)

**Project-level agent:**
```bash
# Create in .claude/agents/ directory
touch .claude/agents/my-agent.md
```

**User-level agent:**
```bash
# Create in ~/.claude/agents/ directory (works across all projects)
touch ~/.claude/agents/my-agent.md
```

**Precedence:** Project-level agents override user-level agents with same name.

---

### Method 3: Programmatic Creation (CLI Flag)

```bash
claude --agents '[{"name":"temp-searcher","tools":"WebSearch","model":"haiku"}]'
```

Use for:
- Automation scripts
- Session-specific agents
- Testing/experimentation

---

## Configuration Reference

### YAML Frontmatter Structure

```yaml
---
name: agent-name
description: Brief purpose statement shown in agent list. Used by Claude to auto-select.
model: sonnet
tools: "Tool1, Tool2, Tool3"
---

# System Prompt Content Below

You are a specialized agent for...

## Your Responsibilities

1. Task A
2. Task B

## Output Format

Return results as...

## Constraints

- Don't do X
- Always do Y
```

---

### Configuration Parameters

#### `name` (required)
- Lowercase, hyphenated identifier
- Unique within scope (project/user)
- Used for invocation: `Use the my-agent agent to...`

**Examples:**
- `searcher-v2`
- `code-reviewer`
- `design-agent`

---

#### `description` (required)
- Natural language purpose statement
- Shown in `/agents` list
- Used by Claude for automatic agent selection
- Should be concise but descriptive (1-2 sentences)

**Examples:**
- `"FAST targeted research executor. Optimized for parallel execution with tool restrictions."`
- `"Autonomous research orchestrator with parallel search execution and hierarchical synthesis."`
- `"Code quality reviewer focusing on security, performance, and best practices."`

---

#### `model` (optional)
- Specifies which Claude model to use
- Accepts: `haiku`, `sonnet`, `opus`, or `inherit`
- Default: Inherits from parent session

**Selection Guidelines:**
| Model | Speed | Cost | Use Case |
|-------|-------|------|----------|
| Haiku | Fastest | Cheapest | Quick searches, batch processing, simple tasks |
| Sonnet | Moderate | Moderate | Complex reasoning, synthesis, orchestration |
| Opus | Slowest | Expensive | Highest quality, critical analysis |

**Example:**
```yaml
model: haiku  # Fast research executor
```

---

#### `tools` (optional)
- Comma-separated list of allowed tools
- If omitted, agent inherits ALL tools from parent
- Empty string `""` = NO tools allowed

**Tool Restriction Benefits:**
- 15-20% focus improvement
- Better security
- Prevents scope creep
- Forces agent specialization

**Examples:**
```yaml
tools: "WebSearch, WebFetch"          # Research agent
tools: "Read, Write, Edit"            # Code editing agent
tools: ""                             # Pure reasoning agent (planner/synthesizer)
tools:                                # Inherit all tools (orchestrator)
```

---

### System Prompt Best Practices

#### 1. Clear Role Definition

```markdown
You are a FAST research executor optimized for parallel execution.

## Core Responsibility

Execute ONE specific search query quickly and return structured findings.
```

#### 2. Specific Instructions

```markdown
## Speed Optimizations

**Fast execution:**
- Focus on the SINGLE query provided
- Use WebSearch for broad info (fast)
- Use WebFetch only for critical deep-dives (selective)
- Aim for <30 seconds per search
```

#### 3. Output Format Specification

```markdown
## Output Format

```json
{
  "search_query": "what was searched",
  "findings": {
    "key_insights": [...],
    "summary": "...",
    "gaps_identified": [...]
  }
}
```
```

#### 4. Clear Constraints

```markdown
## Important Constraints

- ONE query per invocation
- Return structured JSON
- Always cite sources
- No synthesis or analysis
- Speed is critical (you run in parallel with other searchers)
```

---

## Design Principles

### Principle 1: Single Responsibility

Each agent should do ONE thing well.

**Good:**
```yaml
name: searcher-v2
description: Execute web searches and return structured findings
tools: "WebSearch, WebFetch"
```

**Bad:**
```yaml
name: research-agent
description: Search, synthesize, analyze, and report on research topics
tools: "WebSearch, WebFetch, Write, Edit, Read"
```

**Why:** Focused agents are easier to optimize, debug, and parallelize.

---

### Principle 2: Tool Minimalism

Only provide tools absolutely necessary for the agent's function.

**Benefits:**
- 15-20% focus improvement (measured in CAF-Research)
- Better security posture
- Prevents unintended behaviors
- Faster execution (less decision overhead)

**Example:** Research searcher needs `WebSearch` and `WebFetch`, not `Edit` or `Bash`.

---

### Principle 3: Context Isolation

Design agents to operate independently without shared state.

**Enables:**
- Parallel execution without race conditions
- Fresh context prevents pollution
- Easier debugging and testing
- Better scalability

**Pattern:**
```
Orchestrator prepares inputs → Spawns agents → Collects outputs → Synthesizes
```

---

### Principle 4: Structured Communication

Agents should return structured data (JSON) for easier processing.

**Good:**
```json
{
  "status": "success",
  "findings": [...],
  "metadata": {...}
}
```

**Bad:**
```
I found some interesting information about X. Here are the details...
[unstructured text]
```

**Why:** Structured outputs enable:
- Automated processing
- Error handling
- Multi-agent synthesis
- Metrics tracking

---

### Principle 5: Performance Optimization

Choose appropriate models and optimize for speed.

**Model Selection:**
- **Haiku:** Fast executors (searchers, generators, testers)
- **Sonnet:** Complex reasoning (orchestrators, synthesizers, planners)
- **Opus:** Critical quality needs only

**Speed Targets:**
- Searcher: <30 seconds
- Mini-synthesizer: <20 seconds
- Final synthesizer: 2-3 minutes

---

## Performance Optimization

### Parallel Execution (2-4x Speedup)

**Key:** Spawn multiple agents in SINGLE Task tool call.

**Correct:**
```
Single message with multiple Task calls:
- Task: searcher with query A
- Task: searcher with query B
- Task: searcher with query C
- Task: searcher with query D
```

**Incorrect:**
```
Message 1: Task with query A
[wait for result]
Message 2: Task with query B
[wait for result]
...
```

**Why:** Parallel execution eliminates sequential bottlenecks.

---

### Hierarchical Synthesis (40-60% Faster)

**Problem:** Single synthesizer processing 15 raw search results = slow bottleneck

**Solution:** Hierarchical synthesis with mini-synthesizers

```
Batch 1 searches (4 results) → Mini-synthesizer A (20s)
Batch 2 searches (4 results) → Mini-synthesizer B (20s)
Batch 3 searches (4 results) → Mini-synthesizer C (20s)
    ↓
Final synthesizer combines [A, B, C] (2 min)
```

**Benefits:**
- 60-80% reduction in final synthesis input size
- Faster final synthesis
- Better quality through focused batch analysis
- Parallelizable mini-synthesis (future optimization)

---

### Batch Size Optimization

**Default:** 4-6 searches per batch

**Trade-offs:**

| Batch Size | Mini-Synthesis Speed | Final Synthesis Load | Optimal For |
|-----------|---------------------|---------------------|-------------|
| 3-4 | Faster | Higher | Simple research |
| 4-6 | Balanced | Balanced | Most workflows |
| 6-8 | Slower | Lower | Complex synthesis needs |

**Recommendation:** 4-6 strikes best balance for 90% of use cases.

---

### Tool Access Restrictions

**Measured Impact:** 15-20% focus improvement (CAF-Research benchmarks)

**Pattern:**
```yaml
# Planner - pure reasoning
tools: ""

# Searcher - search only
tools: "WebSearch, WebFetch"

# Synthesizer - no tools (data provided)
tools: ""

# Orchestrator - full access
tools:  # inherits all
```

**Why it works:**
- Reduces decision space
- Forces specialization
- Prevents scope creep
- Better security

---

### Streaming Progress Updates

**User Experience Enhancement:**

```
"Planning research... (planner invoked)"
"Executing 6 searches in parallel (Batch 1/2)..."
"Batch 1 complete. Mini-synthesizing findings..."
"Executing 4 searches (Batch 2/2)..."
"Final synthesis in progress..."
"Research complete. Saving to /r/..."
```

**Benefits:**
- Reduced perceived latency
- Transparency into execution
- Debugging visibility
- User confidence

---

## Real-World Examples

### Example 1: CAF-Research v2.0 (Research Automation)

**Architecture:** Wide-then-Deep hybrid with 5 specialized agents

**Agents:**

1. **orchestrator-v2** (Sonnet, all tools)
   - Coordinates parallel workflow
   - Manages batching and synthesis orchestration
   - Saves final reports

2. **planner-v2** (Sonnet, no tools)
   - Creates parallel-optimized research plans
   - Identifies independent vs. dependent searches
   - Assigns batch groups
   - Returns structured JSON only

3. **searcher-v2** (Haiku, WebSearch + WebFetch)
   - Executes single search query
   - <30 second target
   - Returns structured findings with citations
   - No synthesis (focused execution)

4. **mini-synthesizer** (Haiku, no tools)
   - Processes 3-5 search results per batch
   - <20 second target
   - Extracts patterns and key findings
   - Identifies gaps for final synthesis

5. **synthesizer-v2** (Sonnet, no tools)
   - Combines all mini-syntheses
   - Deep analysis and coherent report
   - 2-3 minute execution
   - Comprehensive with citations

**Performance:**
- 3-5x faster than sequential v1.0
- Maintained quality (>80% completeness)
- Typical execution: 2-4 minutes for 8-15 searches

**Source:** [CAF-Research](https://github.com/arealicehole/CAF-Research)

---

### Example 2: SEO Content Generator (Parallel Generation)

**Architecture:** 10 parallel design agents generating 50 landing pages

**Workflow:**

1. **Discovery Agent** - Scans 20+ pages to extract:
   - Business model
   - Value propositions
   - Target audience
   - Design system (CSS, colors, typography)

2. **Strategy Agent** - Creates:
   - 10 pillar topics
   - 50 subpillar topics
   - Aligned to business offerings

3. **Orchestrator** - Spawns 10 design agents simultaneously:
   - Each receives 5 subpillar topics
   - Shared design system specifications
   - Database schema for CTA tracking
   - Isolated execution contexts

4. **Design Agents** (×10 parallel) - Each generates:
   - 5 SEO-optimized landing pages
   - 1000-2000 words per page
   - 2-3 CTAs per page
   - Schema.org markup
   - Internal linking structure

5. **Collection Agent** - Aggregates:
   - 50 generated pages
   - CTA database entries
   - Quality validation

**Performance:**
- ~10x faster than sequential generation
- Consistent design system across all pages
- Complete in ~15-20 minutes (vs 2-3 hours sequential)

**Source:** [claude-code-agents-wizard-v2](https://github.com/IncomeStreamSurfer/claude-code-agents-wizard-v2/tree/design-agent)

---

### Example 3: Code Review Workflow (Sequential Quality Gates)

**Architecture:** Specialized review agents with different focus areas

**Agents:**

1. **security-reviewer** - Scans for:
   - SQL injection vulnerabilities
   - XSS risks
   - Authentication flaws
   - OWASP Top 10 issues

2. **performance-reviewer** - Analyzes:
   - Algorithm complexity
   - Database query optimization
   - Memory leaks
   - Caching opportunities

3. **style-reviewer** - Checks:
   - Code formatting
   - Naming conventions
   - Documentation completeness
   - Best practice adherence

4. **synthesis-reviewer** - Combines findings:
   - Prioritizes issues
   - Eliminates duplicates
   - Provides actionable recommendations

**Execution:**
```
Main Agent → [security, performance, style] (parallel) → synthesis (sequential)
```

**Benefits:**
- Parallel review saves time
- Specialized focus improves quality
- Structured output enables automation

---

## Best Practices

### 1. Start with Claude Generation

Use `/agents` command and describe your needs:
```
"Create an agent that executes web searches quickly, returns JSON results with citations, and uses only WebSearch and WebFetch tools"
```

Claude will generate initial configuration. Press `e` to customize.

---

### 2. Design for Parallelization

**Identify independent operations:**
- Multiple search queries (research)
- Multiple content pieces (generation)
- Multiple test suites (validation)

**Create dependency-free agents:**
- Self-contained inputs
- No shared state
- Structured outputs

**Batch appropriately:**
- 4-6 agents per batch (optimal)
- Mini-synthesis between batches
- Final synthesis combining batches

---

### 3. Use Appropriate Models

**Haiku for:**
- Quick searches
- Batch processing
- Simple transformations
- Cost-sensitive workflows

**Sonnet for:**
- Complex reasoning
- Orchestration
- Deep synthesis
- Quality-critical tasks

**Opus for:**
- Highest quality needs
- Critical analysis
- When cost is secondary

---

### 4. Restrict Tools Aggressively

**Default:** Give agents MINIMUM tools needed

**Benefits:**
- 15-20% focus improvement
- Better security
- Prevents scope creep
- Faster execution

**Example:**
```yaml
# Don't give searcher editing tools
tools: "WebSearch, WebFetch"  # ✅

# Avoid this
tools: "WebSearch, WebFetch, Edit, Write, Bash"  # ❌
```

---

### 5. Structure All Outputs

**Use JSON for agent-to-agent communication:**

```json
{
  "status": "success",
  "data": {
    "findings": [...],
    "metadata": {...}
  },
  "next_steps": [...]
}
```

**Benefits:**
- Easier orchestration
- Error handling
- Metrics collection
- Automated processing

---

### 6. Version Your Agents

**Pattern:**
```
.claude/agents/
  ├── searcher.md          # Stable production
  ├── searcher-v2.md       # Optimized version
  ├── searcher-experimental.md  # Testing
```

**Benefits:**
- A/B testing
- Gradual migration
- Rollback capability
- Performance comparison

---

### 7. Write Clear System Prompts

**Include:**
- Role definition
- Specific responsibilities
- Output format requirements
- Performance targets
- Constraints and limitations

**Example structure:**
```markdown
You are a [role] optimized for [purpose].

## Core Responsibility
[One sentence describing primary function]

## Instructions
[Step-by-step process]

## Output Format
[Structured format specification]

## Constraints
[What NOT to do]

## Performance Targets
[Speed/quality goals]
```

---

### 8. Monitor and Optimize

**Track metrics:**
- Execution time per agent
- Parallelization effectiveness
- Quality/completeness scores
- Cost per operation

**Iterate:**
- A/B test prompt variations
- Experiment with batch sizes
- Try different model assignments
- Optimize tool restrictions

---

### 9. Version Control Agents

**Commit to git:**
```bash
git add .claude/agents/
git commit -m "Add research orchestration agents v2.0"
```

**Benefits:**
- Team collaboration
- Change tracking
- Rollback capability
- Documentation of evolution

---

### 10. Document Your System

**Create a README in `.claude/agents/`:**

```markdown
# Project Agents

## Agent Roster

- **orchestrator-v2**: Main research coordinator (use this by default)
- **searcher-v2**: Fast web search executor
- **synthesizer-v2**: Final report generator

## Usage

```
Use the orchestrator-v2 agent to research: [topic]
```

## Performance

- Simple research: ~1 minute
- Complex research: ~3-4 minutes
```

---

## Troubleshooting

### Problem: Agents Running Sequentially Instead of Parallel

**Symptoms:**
- Research taking 3-5x longer than expected
- Each search completes before next starts

**Causes:**
1. Multiple messages with single Task calls
2. Planner not marking searches as `parallelizable: true`
3. Using v1 agents instead of v2

**Solutions:**
```
✅ Single message with multiple Task calls:
- Task: searcher query A
- Task: searcher query B
- Task: searcher query C

❌ Multiple messages:
Message 1: Task query A
[wait]
Message 2: Task query B
```

---

### Problem: Lower Quality Results

**Symptoms:**
- Missing insights
- Incomplete analysis
- Lower completeness scores

**Causes:**
1. Mini-synthesizers missing nuances
2. Batch sizes too large
3. Using Haiku where Sonnet needed

**Solutions:**
- Reduce batch size to 3-4
- Use Sonnet for final synthesis
- Add quality validation step
- Check mini-synthesizer prompts

---

### Problem: Tools Not Working in Agent

**Symptoms:**
- "Tool not available" errors
- Agent can't execute expected operations

**Causes:**
1. Tools not specified in YAML
2. Typo in tool name
3. Tool not available in context

**Solutions:**
```yaml
# Check tools field
tools: "WebSearch, WebFetch"  # ✅ Correct

tools: "websearch, webfetch"  # ❌ Wrong case

tools: ""  # ❌ No tools allowed
```

---

### Problem: Agent Not Being Auto-Selected

**Symptoms:**
- Have to explicitly invoke: "Use the X agent..."
- Claude doesn't automatically choose agent

**Causes:**
1. Description not descriptive enough
2. Multiple agents with similar descriptions
3. User request doesn't match description keywords

**Solutions:**
- Make description more specific and keyword-rich
- Include use case examples in description
- Differentiate similar agents clearly

```yaml
# Weak description
description: "Helps with research tasks"

# Strong description
description: "FAST targeted research executor for parallel web searches. Returns structured JSON findings."
```

---

### Problem: High Costs

**Symptoms:**
- Unexpected API usage
- High token consumption

**Causes:**
1. Using Sonnet/Opus where Haiku sufficient
2. Too many parallel agents spawned
3. Large context windows
4. Inefficient prompts

**Solutions:**
- Use Haiku for simple executors
- Batch size optimization (4-6 optimal)
- Trim system prompts to essentials
- Monitor token usage per agent

---

### Problem: Slow Synthesis

**Symptoms:**
- Final synthesis taking 5-10 minutes
- Bottleneck at synthesis phase

**Causes:**
1. Synthesizer processing too many raw results
2. Not using hierarchical synthesis
3. Using Haiku where Sonnet needed

**Solutions:**
- Implement mini-synthesizers for batches
- Reduce final synthesizer input by 60-80%
- Use Sonnet for final synthesis
- Structure mini-synthesis outputs

---

## Advanced Patterns

### Pattern: Recursive Refinement

```
Orchestrator →
  Draft Generator (Haiku) →
  Quality Reviewer (Sonnet) →
  If quality < threshold: Refinement Generator (Sonnet) →
  Final Validator (Sonnet)
```

**Use Cases:**
- Content generation with quality gates
- Code generation with review cycles
- Iterative research refinement

---

### Pattern: Competitive Generation

```
Orchestrator →
  [Generator A, Generator B, Generator C] (parallel, different approaches) →
  Evaluator selects best →
  Refinement of winner
```

**Use Cases:**
- Multiple solution approaches
- A/B testing content
- Algorithm optimization

---

### Pattern: Dependency Chain

```
Orchestrator →
  Agent A (independent) →
  Agent B (depends on A) →
  Agent C (depends on B) →
  Synthesis
```

**Use Cases:**
- Multi-step data pipelines
- Sequential validation workflows
- Build systems

---

### Pattern: Fan-Out/Fan-In

```
Orchestrator →
  [Worker 1, Worker 2, ..., Worker N] (parallel) →
  Aggregator →
  Validator →
  Reporter
```

**Use Cases:**
- Distributed testing
- Parallel data processing
- Large-scale generation

---

## Source References

### Primary Sources

1. **CAF-Research Repository**
   - URL: https://github.com/arealicehole/CAF-Research
   - Focus: Multi-agent research automation with v1/v2 architecture comparison
   - Key Contributions: Wide-then-Deep pattern, hierarchical synthesis, performance benchmarks

2. **Claude Code Official Documentation**
   - URL: https://code.claude.com/docs/en/sub-agents
   - Focus: Official subagent system guide
   - Key Contributions: Configuration format, creation methods, best practices

3. **Claude Code Agents Wizard v2**
   - URL: https://github.com/IncomeStreamSurfer/claude-code-agents-wizard-v2/tree/design-agent
   - Focus: Parallel content generation orchestration
   - Key Contributions: Wizard pattern, large-scale parallelization (10+ agents)

4. **Subagent Systems Video Tutorial**
   - URL: https://www.youtube.com/watch?v=2RBKrIlsYN4
   - Focus: Visual demonstration and explanation
   - Note: Content not accessible for this document

### Implementation Examples

5. **Local CAF-Research Implementation**
   - Location: `C:\Users\figon\zeebot\.claude\agents\`
   - Files: `orchestrator-v2.md`, `planner-v2.md`, `searcher-v2.md`, `mini-synthesizer.md`, `synthesizer-v2.md`
   - Focus: Production research automation agents

---

## Appendix: Quick Reference

### Agent Creation Checklist

- [ ] Clear, single responsibility defined
- [ ] Descriptive name (lowercase, hyphenated)
- [ ] Comprehensive description (1-2 sentences)
- [ ] Appropriate model selected (Haiku/Sonnet/Opus)
- [ ] Minimal tool set specified
- [ ] System prompt includes:
  - [ ] Role definition
  - [ ] Specific instructions
  - [ ] Output format
  - [ ] Constraints
  - [ ] Performance targets
- [ ] Tested in isolation
- [ ] Tested in orchestration
- [ ] Documented in project README
- [ ] Version controlled

---

### Performance Optimization Checklist

- [ ] Parallel execution for independent tasks
- [ ] Batch size optimized (4-6)
- [ ] Hierarchical synthesis implemented
- [ ] Tool access restricted appropriately
- [ ] Haiku used for simple executors
- [ ] Sonnet used for complex reasoning
- [ ] Streaming progress updates added
- [ ] Structured outputs (JSON) enforced
- [ ] Execution time monitored
- [ ] Cost per operation tracked

---

### Common Agent Types

| Agent Type | Model | Tools | Purpose |
|-----------|-------|-------|---------|
| Orchestrator | Sonnet | All | Workflow coordination |
| Planner | Sonnet | None | Strategic planning |
| Searcher | Haiku | WebSearch, WebFetch | Web research |
| Synthesizer | Sonnet | None | Analysis & reporting |
| Mini-Synthesizer | Haiku | None | Batch processing |
| Code Reviewer | Sonnet | Read | Code analysis |
| Generator | Haiku | Write | Content creation |
| Validator | Sonnet | Read, Bash | Quality assurance |

---

## Conclusion

Subagent systems transform Claude Code from a sequential assistant into a parallel orchestration platform. Key takeaways:

1. **Architecture Matters:** Wide-then-Deep achieves 3-5x speedups
2. **Tool Restrictions Help:** 15-20% focus improvement measured
3. **Specialization Wins:** Single-responsibility agents outperform generalists
4. **Hierarchical Synthesis Scales:** 60-80% reduction in synthesis bottlenecks
5. **Parallelization is Critical:** Spawn agents in single Task call

**Getting Started:**
1. Use `/agents` to create your first agent
2. Start with simple sequential workflows
3. Identify parallelization opportunities
4. Implement hierarchical synthesis for scale
5. Monitor, measure, and optimize

**Next Steps:**
- Experiment with the patterns in this guide
- Adapt examples to your use cases
- Share improvements and learnings
- Build your agent library over time

---

**Document Version:** 1.0
**Last Updated:** 2025-11-09
**Maintained By:** Research & Development
**Feedback:** File issues or suggestions through your project's feedback channel

---

*This guide synthesizes research from CAF-Research, Claude Code official documentation, community implementations, and production deployments. It represents current best practices as of November 2025.*
