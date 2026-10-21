# Agentic Apps Paradigm — Deep Research
**Task ID:** discord-20261021  
**Date:** 2026-10-21 UTC  
**Video:** [Agentic Apps: How Big Companies Are Actually Deploying AI Agents](https://youtu.be/cWjS56edaUA) (Cole Medin, ~12:29, ~23K views, ~12d ago at capture)  
**Sponsor context:** Oracle partnership / Oracle AI Agent Studio demo (disclosed in video)

---

## objective

Map the **agentic app** paradigm from Cole Medin’s video (Oracle-branded definition), document how **Oracle Agent Builder / AI Agent Studio / Private Agent Factory** actually work, triangulate with independent architecture sources, and produce a practical **home-build blueprint** (OSS stacks, patterns, and a concrete starter architecture) for outcome-focused multi-agent applications rather than “one chat agent does everything.”

## summary

**Your read is correct and industry-aligned.** The emerging meta is not “replace the product with a general agent,” it is **outcome-first software** where:

1. You design a **single-view app for a role/outcome** (manager coaching, leave admin, requisition intake, media ops desk, etc.).
2. You decompose work into **specialized agents / workflows** that run *inside* that app.
3. Every LLM call sits in a **deterministic workflow** (context load → code/conditionals → scoped tools → model → format → human gate).
4. **Critical rules are codified + tested** (policy functions), not left to pure RAG “usually right.”
5. The UI is a **home base**: widgets, next-action suggestions, draft communications, artifacts (PDF/slides), not walls of chat.

Oracle’s commercial packaging of this is **Fusion AI Agent Studio + Agentic Application Builder** (and a parallel **Private Agent Factory** on Oracle AI Database 26ai). The same design philosophy appears in Google’s agent design-pattern guide, IBM’s agentic-systems framing, and OSS stacks (LangGraph, n8n, Dify, Archon, Temporal-style durable execution).

**Home build translation:** treat Hermes/kanban multi-agent setups, n8n/Dify canvases, or LangGraph+FastAPI UIs as the “Agent Studio”; ship a thin outcome dashboard that *invokes* workflows with typed intents (`InitDisplay`, `Query`, `InvokeAction` equivalents), not a naked chat box as the product.

---

## plan (executed)

| ID | Task | Status |
|----|------|--------|
| T1 | Pull / reconstruct video transcript & chapter structure | done (secondary full transcript + Learning Atlas deep lesson; YT captions unavailable in-browser) |
| T2 | Oracle AI Agent Studio / Agentic App Builder / Agent Factory mechanics | done (official docs snippets + product PDFs + blog announcements) |
| T3 | Paradigm triangulation (Google/IBM/academic + industry) | done |
| T4 | Home-build stack comparison + concrete blueprint | done |
| T5 | Structured claims + provenance report → know archive | done |

---

## findings

### 1. Video core thesis (Cole Medin / Oracle framing)

**Source artifact:** full spoken transcript recovered via search index of the YouTube page + Learning Atlas deep lesson + Zovi chapter timestamps.

**Chapters (Zovi):**
- 0:00 Why one chat agent isn’t enough  
- 0:45 What is an agentic app?  
- 2:47 Building in Oracle AI Agent Studio  
- 3:27 Inside the agentic app  
- 4:36 Agents that prioritize next actions  
- 5:14 Every LLM call runs inside a workflow  
- 7:17 Security with policies  
- 8:20 Policy nodes vs RAG  
- 8:57 Building the leave-policy workflow  
- 9:30 Workflow permissions and triggers  
- 10:22 Visual outputs, not walls of text  
- 11:40 Recap  

**Key points from the talk:**

- Personal agents (second brain, coding assistant) = **one chat surface**. Fine for individuals; often wrong for company adoption.
- Temptation of the AI craze: make the **agent the entire app**. What orgs need is an **agentic app**: business outcome first, agents as tools inside a larger deterministic shell.
- Demo outcome: **manager coaching / team development engagement** for a director with ~12 reports — one view of engagement, objectives, follow-ups.
- Under the hood: each UI component invokes a **workflow**, not a free-form agent loop.
- Chat remains available, but is not the only interface: dashboard components, draft email with edit-before-send, top-right **background agents suggesting next actions**.
- Industry convergence claim: make agentic workflows **as deterministic as possible** for guarantees (context always loaded a certain way, right model via conditionals, formatting fixed, mission-critical paths can’t “wing it”).
- **Policies:** upload handbook PDF → Studio generates **functions with source, tests, strict I/O** → attach as least-privilege tools in workflow. Contrast with RAG chunk search (probabilistic; “usually” fails for benefits/vacation rules).
- Leave-policy example: load policy tools → **classification LLM** (parental leave vs vacation) → scoped tools only → JSON workflow output later bound to UI widgets.
- Security layers: who may **run** the workflow; **identity** the workflow runs as; granular agent capabilities.
- Triggers beyond click: email inbound etc.; app is home base for status, manual triggers, Q&A.
- Prefer **visual artifacts** (PDF learning-risk summary, decks) over text dumps.
- Business users can build without being technical; pro-dev path (VS Code + coding agents) teased for follow-ups.
- Cole maps this to his OSS **Archon** harness builder for coding workflows (YAML deterministic harness above coding agents).

**Oracle definition (as stated in video):** build the app around the **business outcome**, not around “what can this agent automate?”

### 2. What Oracle actually ships (two related product lines)

Do not conflate them:

| Product | Where it lives | Who it’s for | Core idea |
|---------|----------------|--------------|-----------|
| **Oracle AI Agent Studio for Fusion Applications** + **Agentic Application Builder** | Fusion SaaS / enterprise apps | LOB + IT inside Oracle Cloud Apps | Compose **agentic apps** from specialized **agent teams/workflows**, widgets, actions, communications, Ask Oracle; same security/business objects as Fusion |
| **Private Agent Factory / Agent Builder** | Oracle AI Database 26ai | Data/platform teams wanting private agentic patterns on DB | No-code canvas: MCP servers, docs, DB, chat inputs, private/cloud LLMs, **Select AI** in-DB execution |

**Announcements (2026):**
- Mar 23–24, 2026: Private Agent Factory intro; Fusion AI Agent Studio expands with **agentic applications builder**, orchestration, content intelligence, contextual memory.
- May 15, 2026: Fusion Insider — Agentic Application Builder composes apps from reusable Oracle/partner/custom agents inside Fusion security model.
- Jul 14, 2026: AI-native builder experience; TechTarget — no-code / low-code / pro-code under one roof.

### 3. Anatomy of an Oracle agentic app (Fusion)

From Oracle Fusion CoE blogs + official docs (26B/26C):

**Composition:** one or more **Agent Teams** from AI Agent Studio compose the app through four pillars (Anatomy blog):

1. **Information Displays** — cards, lists, dashboards, structured widgets  
2. **Actionable Insights** — suggested next actions the user can invoke  
3. **Communications** — draft emails/messages with parameter fill + send flows  
4. **Ask Oracle Advisors** — chat/query path still present, but scoped  

**Agents are the only reasoning layer** (Core Concepts blog): everything else renders, routes, or delivers what agents produce.

**Agent team types (Studio overview PDF / docs):**
- **Supervisor** — manages other agents/artifacts in an agentic flow  
- **Workflow** — predetermined ordered nodes (deterministic orchestration)

**Workflow node families (Workflow Agents technical guide):**
- **AI:** LLM, agent-invoke, subflow, RAG document  
- **Logic:** code (JS transforms), set variables  
- **Data:** business objects, external REST, document processor, vector DB  
- **Control:** if, switch, loop, parallel, **human approval**  
- **Communication:** send email  

**Example (procurement quotation → requisition):** set vars → BO get preferences → IF can create → document processor → currencies/UOM → LLM extract structured JSON → match supplier → LLM build payload → BO create PR → IF success → LLM confirm / else email failure. LLM only where unstructured→structured judgment is needed.

**Agentic app ↔ workflow contract (`OraMessageHint`):**  
Workflows enabled with **Expose to agentic apps** receive app-aware intents such as:

| Hint | Meaning |
|------|---------|
| `Summary` / `InitSubtitle` | short text for agent contribution |
| `InitDisplay` | startup widgets / info displays |
| `InitActions` / `InitCommunications` | startup suggestions |
| `Query` | user asked something |
| `InvokeAction` | UI command / widget action callback |
| `AdditionalContent` | side panel load |
| `FillParameters` / `SendCommunication` | comms lifecycle |

This is the critical abstraction: **the app is an intent router over workflows**, not a chat log owning state.

**Widgets:** LLM/agent nodes can emit `patternId` + `config` (e.g. `cardWidget`) with commands like `ora.Invoke("viewMaintenanceSummary")`.

**Tools/topics/credentials:** REST, business objects, **MCP** tools, reusable Topics (expertise slices), custom LLM credentials, human-in-the-loop approval nodes.

**Deploy:** test in Studio → publish → embed chat, webhooks, HTML/React, or consume from agentic app shell; user access controlled via Fusion roles.

### 4. Private Agent Factory / Agent Builder (DB line)

From Oracle DB blogs + agent-factory docs:

- Visual **Agent Builder / Workflow Builder** canvas  
- Nodes: MCP servers, document/DB/user inputs, prompts, LLM choice (private or cloud), **Select AI** for in-database NL→SQL / AI execution  
- Connect agents to Select AI, PL/SQL, REST, OpenAPI, MCP  
- Templates, live testing, governance  

This is closer to “platform agent factory on your data plane” than Fusion’s role-based business app shell—but the pattern (canvas + tools + workflows + least privilege) is the same family.

### 5. Paradigm outside Oracle (triangulation)

**Google Cloud Architecture (2026-05-28) — agent design patterns for agentic applications:**
- Agents for open-ended, multi-step, tool-using goals; non-agentic genAI often enough for pure summarize/translate/classify.
- Patterns: single-agent; multi-agent sequential / parallel / loop / review-critique / iterative refinement / coordinator / hierarchical decomposition / swarm; ReAct; human-in-the-loop; custom logic.
- Decision axes: deterministic vs dynamic orchestration, latency, cost (multi-call), human stakes.
- Explicit split: **workflows that are deterministic** vs **dynamic orchestration** vs **iteration** vs special requirements.

**IBM agentic AI systems:**
- Orchestration component coordinates agents; may use LLM for dynamic workflow generation **or** static BPM-style workflows.
- Catalog/governance emphasis (watsonx Orchestrate multi-agent).

**Academic / industry taxonomy (2025–2026):**
- Distinction **AI Agents** (task-bounded tool users) vs **Agentic AI** (multi-agent collab, decomposition, persistent memory, coordinated autonomy).
- Surveys note fragmentation and “conceptual retrofitting”; practical builders should ignore brand wars and keep: **outcome shell + scoped agents + durable workflow + eval/HITL**.

**Convergence claim (supported):** enterprises and serious builders are moving from “chatbot product” → “**application with agentic stages**.”

### 6. How to build this at home

#### 6.1 Design method (platform-agnostic)

1. **Name the role + outcome** in one sentence.  
   *Example:* “Ops lead gets one screen to triage overnight research findings into kanban + brief.”
2. **Sketch the single view** (3–7 components max): status widgets, next actions, draft comms, detail panels, optional chat.
3. **Inventory steps of the manual process**; mark each as:
   - deterministic code/API  
   - needs judgment (LLM)  
   - needs human approval  
   - needs retrieval (RAG OK) vs **must be exact** (codify policy)
4. **Define workflows first**, UI second. Each button = workflow invoke with typed intent.
5. **Least privilege tools** per workflow (not one mega-toolbag agent).
6. **Structured outputs** (JSON schema / widgets) always; free text only inside bounded fields.
7. **Eval harness:** golden tests for policy paths; trajectory logs for agent paths.
8. **Home base UX:** edit-before-send, show sources/policy version, never silent side effects on high-stakes actions.

#### 6.2 Stack choices (2026 OSS / self-host landscape)

| Layer | Strong home options | When to pick |
|-------|---------------------|--------------|
| Visual workflow + integrations | **n8n** (400+ nodes, AI/LangChain/MCP-ish glue, easy Docker) | Fastest path to “Studio-like” canvas + real APIs |
| LLM app platform (chat/RAG/agent/workflow UI) | **Dify**, **Coze Studio** | Want hosted-feeling app shell + model mgmt |
| Code-first durable agent graphs | **LangGraph** (+ checkpointing), **Temporal** + workers | Complex state, retries, long-running jobs |
| Multi-agent role play | CrewAI / AG2 / custom supervisor | Research/creative teams; watch nondeterminism |
| Coding-agent harness (YAML workflows above agents) | **Archon** (coleam00) | Dev workflows that mirror Medin’s thesis |
| RAG engine | RAGFlow, lightweight vector+rerank | Knowledge Q&A where “usually” is OK |
| App shell | Next.js / FastAPI+HTMX / Discord/Telegram bot as *view* | Your outcome UI + auth |
| Local models | llama.cpp / vLLM / your AI Box | Cost/privacy; pin models per node like Oracle conditionals |
| Protocol | **MCP** tool servers | Match Oracle’s external tool story |

**Jimmy Song 2026 comparison snapshot:** Coze/Dify/n8n best “full platform + self-host”; LangGraph best code graph; n8n great glue but not deepest multi-agent state; RAGFlow for deep RAG; Fabric for CLI skill packs.

#### 6.3 Reference home architecture (recommended default)

```text
┌─────────────────────────────────────────────┐
│ Outcome App Shell (Next.js / Discord / TUI) │
│  widgets | next-actions | drafts | chat     │
└──────────────────┬──────────────────────────┘
                   │ typed intents
                   │ (InitDisplay|Query|InvokeAction|...)
                   ▼
┌─────────────────────────────────────────────┐
│ Orchestrator (n8n OR LangGraph OR Hermes    │
│ kanban dispatcher)                          │
│  - load context                             │
│  - branch / parallel                        │
│  - policy functions (tested code)           │
│  - scoped tool calls (MCP)                  │
│  - LLM nodes (model pinned per step)        │
│  - HITL gates                               │
│  - format → widget JSON                     │
└───────┬───────────────────────────┬─────────┘
        │                           │
        ▼                           ▼
  Tool plane (MCP)            Data plane
  gmail, gh, cal,             Postgres, files,
  browser, kanban…            vectors, object store
        │
        ▼
  Eval + logs + policy versioning
```

**Map Oracle → home:**

| Oracle | Home equivalent |
|--------|-----------------|
| AI Agent Studio workflow | n8n workflow / LangGraph graph / Archon YAML |
| Agent team (specialized) | one workflow or one agent profile with narrow tools |
| Agentic app builder shell | your dashboard / bot “home channel” |
| `OraMessageHint` | intent enum on API `/invoke` |
| Policies as generated functions | Python modules + pytest; or OPA/Cedar for authz |
| Business object nodes | typed repository functions, not raw SQL in prompts |
| Human approval node | kanban `needs_input` / Discord confirm button / DB approval row |
| Widget `patternId` | shared UI component registry + JSON schema |
| Background “what next” agents | scheduled graph that writes `NextAction` rows |
| Fusion security | your auth (SSO) + per-workflow ACL + run-as service account |

#### 6.4 Minimal build path (weekend → month)

**Weekend MVP**
1. Pick one painful outcome you already live (e.g. GIU intel triage, media shot list, sales follow-ups).
2. Stand up **n8n** + one LLM + Postgres.
3. Three workflows: `init_display`, `query`, `invoke_action:draft_email`.
4. Ugly but real UI: single HTML page or Discord embeds showing cards + buttons that POST intents.
5. One **policy module** (e.g. “never post externally without approval”) as code, not prompt.

**Two-week hardening**
- JSON schemas for all LLM outputs; repair loop max 1.
- Separate models: small/fast for classify; larger for write.
- MCP servers for tools; revoke catch-all browsing from write paths.
- Golden tests for policy; trajectory sampling for agent steps.
- Artifact generation (Markdown→PDF) for “visual not walls of text.”

**Month productization**
- LangGraph or Temporal for long jobs; n8n remains glue for SaaS webhooks.
- Multi-agent only where parallel specialties help (researcher / editor / publisher), with **supervisor workflow** owning order.
- Memory: short-term thread state + episodic lessons in skills + durable facts elsewhere (avoid mega-context).
- Observability: OpenTelemetry traces per node; cost budgets per outcome.

#### 6.5 Anti-patterns (from video + enterprise patterns)

- Agent-as-entire-product with unbounded tools  
- RAG for compliance-critical rules  
- One god-agent with full prod credentials  
- Chat-only UX for multi-entity operational work  
- No human gate on irreversible actions  
- Dynamic multi-agent swarm where a 8-node deterministic graph would do  
- Skipping structured outputs / widget contracts  

#### 6.6 Fit to *this* environment (Hermes / swarm)

You already run pieces of an agentic *platform* (profiles, kanban, skills, MCP, local LLMs). The paradigm shift for **apps** is:

- Stop stopping at “Geraldo can research anything.”  
- Start shipping **outcome apps**: e.g. `Intel Triage App` that always: fetch → claim-extract → verify → write know path → kanban card → optional Huly `[HULY]` — with Geraldo as *one node*, not the product.  
- SCOR media pipeline as agentic app: ingest → VAD/transcribe → shot detect → edit plan → human approve → render.  
- Each Discord “home” channel can be an agentic-app shell with buttons/intents.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | Agentic apps prioritize business outcome + role UX; agents are embedded tools, not the whole product | E1, E2, E3 | high | supported |
| C2 | Reliable production pattern is LLM-inside-deterministic-workflow (context, conditionals, code, format, HITL) | E1, E4, E5, E6 | high | supported |
| C3 | Oracle Fusion AI Agent Studio builds agent teams (supervisor + workflow) with modular nodes and exposes them to agentic apps via hints like `OraMessageHint` | E4, E5, E7 | high | supported |
| C4 | Oracle distinguishes policy-as-tested-functions from probabilistic RAG for critical guidelines | E1, E2 | high | supported |
| C5 | Fusion agentic apps compose specialized agents into displays, actions, communications, and advisor chat | E3, E8 | medium-high | supported |
| C6 | Oracle also ships Private Agent Factory/Agent Builder on AI Database with MCP + Select AI canvas | E9, E10 | high | supported |
| C7 | Industry architecture guides independently recommend choosing deterministic vs dynamic multi-agent patterns by task structure, cost, latency, HITL | E6, E11 | high | supported |
| C8 | Practical home stacks that approximate Studio: n8n/Dify/Coze (canvas), LangGraph/Temporal (durable graphs), Archon (coding harness), MCP tools | E12, E13, E14 | high | supported |
| C9 | Cole Medin’s Archon is intentionally analogous to Oracle workflow determinism for coding agents | E1, E14 | medium-high | supported |
| C10 | “Agentic apps are the new meta” as a *deployment pattern* is well-supported; as a *guaranteed market winner brand* it is still vendor-contested | E1–E11 | medium | partial |

## evidence_index

| evidence_id | source_label | source_url | provenance | support note |
|-------------|--------------|------------|------------|--------------|
| E1 | Cole Medin video full transcript (indexed) | https://www.youtube.com/watch?v=cWjS56edaUA | web_search/exa transcript mirror of YT description/captions body; browser confirmed title/channel/length 12:29 | Primary talk content; Oracle-sponsored |
| E2 | Learning Atlas deep lesson on same video | https://learningatlas.us/learning/video/cWjS56edaUA | web_search snippet | Outcome-before-agent; constrain every call; codify critical rules |
| E3 | Zovi chapter index + summary | https://zoviai.com/agentic-apps-how-big-companies-are-actually-deploying-ai-agents/ | web_search | Timestamps + multi-agent/HITL framing |
| E4 | Oracle docs: create/enable workflows for agentic apps + OraMessageHint | https://docs.oracle.com/en/cloud/saas/fusion-ai/26c/aiaas/create-and-enable-workflows-for-using-in-agentic-apps.html | web_search content extract | Official contract between app shell and workflows |
| E5 | Oracle “How do I use AI Agent Studio?” PDF content (tools, topics, supervisor vs workflow, requisition example) | https://docs.oracle.com/en/cloud/saas/fusion-ai/26b/aiaas/how-do-i-use-ai-agent-studio.pdf | web_search extract | Node-level workflow design |
| E6 | Google Cloud: Choose design pattern for agentic AI system | https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system | browser snapshot 2026-05-28 | Independent pattern taxonomy |
| E7 | Enable workflow agent teams to generate widgets | https://docs.oracle.com/en/cloud/saas/fusion-ai/26b/aiaas/enable-agent-teams-to-produce-widgets.html | web_search extract | Widget patternId/config + ora.Invoke |
| E8 | Anatomy / Core Concepts / Learning Path (Fusion CoE) | https://blogs.oracle.com/fusioncoe/the-anatomy-of-an-agentic-app etc. | web_search descriptions (blog HTML was down in-session) | Four pillars; agents as intelligence layer |
| E9 | Introducing Private Agent Factory | https://blogs.oracle.com/database/introducing-private-agent-factory-unlocking-the-agentic-ai-potential-in-enterprises-with-oracle-ai-database-26ai | web_search | Agent Builder canvas, MCP, Select AI |
| E10 | Oracle Agent Factory product + MCP multi-agent blog | https://www.oracle.com/database/agent-factory/ ; https://blogs.oracle.com/ai-and-datascience/oracle-private-agent-factory-mcp-multi-agent | web_search | MCP node on canvas |
| E11 | IBM guide to agentic AI systems | https://www.ibm.com/think/architectures/patterns/agentic-ai | web_search | Orchestration + static or dynamic workflows |
| E12 | Jimmy Song OSS agent/workflow comparison 2026 | https://jimmysong.io/blog/open-source-ai-agent-workflow-comparison/ | web_search long extract | n8n/Dify/Coze/LangGraph/etc. |
| E13 | Oracle news: Agentic Applications Builder (2026-03-24) | https://www.oracle.com/news/announcement/oracle-expands-ai-agent-studio-for-fusion-applications-with-agentic-applications-builder-2026-03-24/ | web_search | Product expansion |
| E14 | Archon — coleam00 harness builder | https://github.com/coleam00/Archon ; https://archon.diy/ | web_search | YAML workflows above coding agents |
| E15 | Workflow agents technical guide PDF (node taxonomy) | https://www.oracle.com/apac/a/ocom/docs/applications/workflow-agents-ai-agent-studio-guide.pdf | web_search extract | AI/logic/data/control/comms nodes |
| E16 | TechTarget: Oracle AI agent builder no/low/pro code (2026-07-14) | https://www.techtarget.com/searchcustomerexperience/news/366645665/Oracle-AI-agent-builder-brings-no-code-low-code-and-pro-code-together | web_search | Positioning |

## gaps

1. **Native YouTube caption file** not downloaded in-container (baoyu script consent/timeout; YT UI showed “captions unavailable”). Transcript reconstructed from high-quality secondary indexes — treat quotes as faithful but not frame-accurate.
2. **Oracle Fusion CoE blog HTML** returned technical difficulty during browser fetch; anatomy/core-concepts details taken from search abstracts + docs PDFs, not full blog body.
3. **Firecrawl keyless 403** blocked bulk doc scrape; relied on search-indexed doc extracts + browser where possible.
4. No hands-on live trial of Fusion Agent Studio (requires Oracle Fusion tenancy) or Private Agent Factory in this session.
5. Academic full-texts (ACM/Springer surveys) not paywall-fetched; taxonomy claims are abstract-level only.
6. Home-build section is architecture guidance, not a checked-in starter repo from this run.

## confidence

- **level:** **high** on paradigm + Oracle mechanics + home-build mapping  
- **rationale:** Primary video content recovered in full from multiple mirrors; Oracle official docs provide concrete workflow/app contracts and node types; independent Google/IBM/OSS sources corroborate the deterministic-workflow-around-outcome pattern. Medium residual uncertainty only on blog-only marketing claims and untested live Studio UX details.

## recommended_next_actions

1. **Pick one outcome app** to build this week (suggested: Intel Triage or Media Desk) with explicit `InitDisplay` / `Query` / `InvokeAction` API — reuse existing agents as nodes only.
2. **Stand up n8n (or LangGraph)** beside Hermes; implement one policy-as-code path with tests (no RAG for that rule).
3. **Define a widget JSON schema registry** (card, list, next-action, draft-comms) shared by Discord/web UI.
4. **Optional deep dive:** when Fusion CoE blogs are up, extract full “anatomy” post; or spin Private Agent Factory trial if DB 26ai is in scope.
5. **Watch Medin follow-ups** on pro-code VS Code agentic app building; compare to Archon YAML.
6. If desired, spawn a build task (Su/coding agent) to scaffold `agentic-app-shell` template from this report.

---

## appendix A — video takeaways (one-pager)

1. Outcome before agent  
2. App shell + many specialized under-hood agents  
3. Every LLM call inside a workflow  
4. Codify critical rules (tests) > RAG for compliance  
5. Least-privilege policies + run identity  
6. HITL on side effects  
7. Visual artifacts over chat walls  
8. Home base for status, triggers, Q&A  

## appendix B — starter intent enum (copy-paste)

```python
from enum import Enum

class AppIntent(str, Enum):
    SUMMARY = "Summary"
    INIT_SUBTITLE = "InitSubtitle"
    INIT_DISPLAY = "InitDisplay"
    INIT_ACTIONS = "InitActions"
    INIT_COMMS = "InitCommunications"
    QUERY = "Query"
    INVOKE_ACTION = "InvokeAction"
    ADDITIONAL_CONTENT = "AdditionalContent"
    FILL_PARAMETERS = "FillParameters"
    SEND_COMMUNICATION = "SendCommunication"
```

## appendix C — publish metadata

- Archive path: `/home/ice/know/research/agentic-apps-paradigm-oracle-home-build-discord-20261021.md`
- Request origin: Discord thread “Agentic apps”  
- Researcher: Geraldo v2.2 (geraldov21)
