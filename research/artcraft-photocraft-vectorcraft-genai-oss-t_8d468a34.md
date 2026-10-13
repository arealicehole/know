# ArtCraft / Storytold PhotoCraft + VectorCraft — OSS Identity & Gen-AI Capability Report

**Task:** `t_8d468a34`  
**Board:** research  
**Agent:** Geraldo v2.2 (GIU)  
**Date:** 2026-10-13  
**Primary sources:** storytold GitHub READMEs, `docs/gaps.md`, `docs/roadmap.md`, `docs/mcp.md`, `docs/target-app-parity.md`, getartcraft.com product pages, GitHub API metadata  

---

## objective

Map the open-source identity of **ArtCraft / Storytold** apps **PhotoCraft** and **VectorCraft**, and assess their **generative-AI / agent automation** capabilities versus classical editor features — with provenance for every material claim.

## summary

**PhotoCraft** (`storytold/photocraft`) and **VectorCraft** (`storytold/vectorcraft`) are **clean-room, pure-Rust** reimplementations of **Adobe Photoshop** and **Adobe Illustrator** workflows, published by the **ArtCraft Team** under the **storytold** GitHub org. Both are **MIT OR Apache-2.0**, early-alpha / active-development, with large community traction (PhotoCraft ~57.7k★, VectorCraft ~8.4k★ as of 2026-10-13 API).

They are **not** generative-image products. Gen-AI pixel/vector synthesis (Firefly-class Generative Fill, Text-to-Vector, Neural Filters, etc.) is **explicitly missing** on both roadmaps. What *is* strong and shipping is:

1. **Classical “smart” algorithms** (PhotoCraft: Content-Aware Fill/Scale, healing/remove via patch completion, classical Select Subject/Object; VectorCraft: Image Trace with presets).
2. **Agent automation** first-class: shared **command registry** + **headless CLI** + **JSON control channel** + **MCP server** (stdio; optional bridge to live desktop), so AI agents drive the *editor* rather than an embedded diffusion stack.
3. **Sibling product** **ArtCraft** (`storytold/artcraft`, getartcraft.com) is the **AI image/video studio** (Seedance, compositing, BG removal, etc.) — separate from the Crafting Apps’ local editors.

**Bottom line for integrators:** treat PhotoCraft/VectorCraft as **open Photoshop/Illustrator engines with MCP**, not as local Stable-Diffusion / Firefly replacements. Pair with ArtCraft studio or external models if generative pixels/vectors are required.

## plan

| task_id | title | status |
|---------|--------|--------|
| T1 | Identify canonical repos, org, licenses, maturity | done |
| T2 | Extract official feature claims (PC + VC) | done |
| T3 | Gen-AI / ML gaps vs classical smart tools | done |
| T4 | Agent surfaces: CLI, MCP, control protocol | done |
| T5 | Ecosystem context (Crafting Apps + ArtCraft studio) | done |
| T6 | Structured report + know archive publish | done |

**Assumptions:** Prefer primary GitHub docs over third-party mirrors (photocraft.one, artcraft.wiki, etc.). Stars/forks are point-in-time API reads.

---

## claims

### claim_id: C1
- **statement:** PhotoCraft is an open-source, clean-room reimplementation of Adobe Photoshop in pure Rust by the ArtCraft Team under `storytold/photocraft`, dual-licensed MIT OR Apache-2.0, status early alpha.
- **evidence_refs:** [E1], [E3], [E7]
- **confidence:** high
- **status:** supported

### claim_id: C2
- **statement:** VectorCraft is an open-source, clean-room reimplementation of Adobe Illustrator in pure Rust under `storytold/vectorcraft`, same license family, status “in active development” / alpha (self-assessed ~65% ready for mainstream illustrator work as of 2026-10-10 README).
- **evidence_refs:** [E2], [E4], [E8]
- **confidence:** high
- **status:** supported

### claim_id: C3
- **statement:** As of GitHub API 2026-10-13, PhotoCraft has ~57,757 stars / ~9,741 forks / ~1,504 open issues; VectorCraft ~8,452 stars / ~3,968 forks / ~294 open issues. Both repos created 2026-09-30; very recent public launch with rapid release cadence (PC through v0.8.2; VC through v0.10.0 by 2026-10-12).
- **evidence_refs:** [E9], [E10]
- **confidence:** high
- **status:** supported

### claim_id: C4
- **statement:** PhotoCraft ships a broad classical Photoshop feature surface: layers/masks/adjustment layers (16), layer styles, type, vectors/paths, brushes, GPU compositor (wgpu), PSD/PSB round-trip emphasis, 70+ filters, retouch tools including Content-Aware Move/Fill/Scale, classical Select Subject/Object/Quick Selection, and multi-bit/multi-mode color (RGB/CMYK/Lab/Grayscale).
- **evidence_refs:** [E1], [E3], [E5], [E11]
- **confidence:** high
- **status:** supported

### claim_id: C5
- **statement:** VectorCraft ships a broad classical Illustrator feature surface: Pen/Direct Selection, exact Pathfinder booleans, live blends/envelope/mesh/repeats, Appearance, type, multi-artboard, Image Trace (12 presets per product page), SVG/PDF/.ai (PDF-compatible) interchange, native `.vectorcraft` JSON, high-performance SIMD render claims (~20k shapes ~27 ms).
- **evidence_refs:** [E2], [E4], [E12]
- **confidence:** high
- **status:** supported

### claim_id: C6
- **statement:** PhotoCraft does **not** implement Adobe-class generative AI: Generative Fill/Expand, Generative Upscale, Neural Filters, Sky Replacement, Super Zoom, or ML Select Subject/Sky. There is **no ML runtime** (scorecard AUTO-219-7). Gap G15 estimates 300–600 agent-hours **after** owner decision on model licenses (issue #41). Classical substitutes only (GrabCut/max-flow selection; Remove via non-local patch completion). Feature-breadth weight for AI/generative is estimated ~15%; AI readiness dimension ~10%.
- **evidence_refs:** [E1], [E13], [E14], [E15]
- **confidence:** high
- **status:** supported

### claim_id: C7
- **statement:** VectorCraft does **not** implement Adobe-class generative vector AI: Text to Vector Graphic, Generative Recolor, Generative Shape Fill, Generative Expand, Concept to Vector, Prompt to Edit, Rewrite Story. Gap G18 estimates 30–60 h after owner decision on pluggable providers. README lists “generative AI features” under What’s missing.
- **evidence_refs:** [E2], [E16], [E17]
- **confidence:** high
- **status:** supported

### claim_id: C8
- **statement:** Both apps are **agent-native**: every UI action is a command; CLI + JSON control channel + MCP server share the same command surface. PhotoCraft: `photocraft-cli run|batch|mcp`, bridge via `photocraft --control`, MCP tools include `command_list`, `command_run`, `command_batch`, `doc_inspect`, `render_preview` (+ UI tools when bridged). VectorCraft: `vectorcraft-cli mcp` (headless or `--connect`), prompts like `poster`/`icon-set`/`trace-and-style`, resources `vectorcraft://document`, etc. ArtCraft product principle: “Agent-ready — CLI, JSON control channel or MCP server.”
- **evidence_refs:** [E1], [E2], [E5], [E18], [E19], [E20]
- **confidence:** high
- **status:** supported

### claim_id: C9
- **statement:** PhotoCraft self-rates ~76% Photoshop feature breadth and ~45% “ready for real work” (alpha; alpha gate passes). Biggest non-AI gaps: PSD fidelity to Photoshop, interactive performance, bug backlog, brush/transform feel, missing tools, type depth, plugins (.8BF/UXP). VectorCraft self-rates higher on illustration readiness (~65% mainstream) but still alpha; beta blocked mainly by `.ai` exchange fidelity and interaction polish — not by gen-AI.
- **evidence_refs:** [E14], [E15], [E2], [E17]
- **confidence:** medium-high
- **status:** supported

### claim_id: C10
- **statement:** The broader ArtCraft “Crafting Apps” suite includes 13 pure-Rust editors (PhotoCraft, VectorCraft, FilmCraft, LightCraft, PdfCraft, EffectCraft, DesignCraft, FigCraft, WordCraft, SoundCraft, DeckCraft, CADCraft, GridCraft). Separate **ArtCraft** studio (`storytold/artcraft`, ~25.5k★) is the controllable AI image/video product (Seedance, compositing, mesh, BG removal) — not the same binary as PhotoCraft/VectorCraft.
- **evidence_refs:** [E5], [E6], [E21], [E22]
- **confidence:** high
- **status:** supported

### claim_id: C11
- **statement:** Platforms: native macOS / Windows / Linux / FreeBSD + WebAssembly web builds for both apps; launcher `storytold/craft-launcher`. Releases ship desktop installers and separate `*-cli` archives (esp. macOS). Offline/local-file emphasis for Crafting Apps.
- **evidence_refs:** [E3], [E4], [E1], [E2]
- **confidence:** high
- **status:** supported

### claim_id: C12
- **statement:** “On-device selection” and smart retouch in PhotoCraft marketing refer to **classical CV / patch algorithms running locally**, not cloud generative models — and not an ONNX/ort ML backend yet (roadmap M10 lists `ml`/ort as planned, status “ML backend not started” under smart features).
- **evidence_refs:** [E3], [E13], [E15]
- **confidence:** high
- **status:** supported

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|-------------|--------------|------------|------------|------------------------|
| E1 | PhotoCraft README | https://raw.githubusercontent.com/storytold/photocraft/main/README.md | web_extract/raw 2026-10-13 | Clean-room Photoshop; pure Rust; MIT/Apache; agent-ready MCP/CLI; status early alpha; biggest gaps include AI/generative features |
| E2 | VectorCraft README | https://raw.githubusercontent.com/storytold/vectorcraft/main/README.md | web_extract/raw 2026-10-13 | Clean-room Illustrator; MCP; Made-in-VC via command API; What’s missing includes generative AI features |
| E3 | getartcraft PhotoCraft page | https://getartcraft.com/apps/photocraft | web_extract 2026-10-13 | Early alpha; on-device selection; GPU canvas; PSD; downloads v0.7.0 listed on page at extract time |
| E4 | getartcraft VectorCraft page | https://getartcraft.com/apps/vectorcraft | web_extract 2026-10-13 | In development; Image Trace 12 presets; CLI assets; v0.9.0 on page at extract time |
| E5 | Crafting Apps index | https://getartcraft.com/apps | web_extract 2026-10-13 | Thirteen apps; Agent-ready principle; WASM list |
| E6 | ArtCraft home | https://getartcraft.com/ | web_extract 2026-10-13 | Controllable AI studio messaging; Seedance; open source studio |
| E7 | PhotoCraft GitHub HTML | https://github.com/storytold/photocraft | web_extract 2026-10-13 | About, license badges, stars display |
| E8 | VectorCraft GitHub HTML | https://github.com/storytold/vectorcraft | web_extract 2026-10-13 | About, MCP badge, stars display |
| E9 | GitHub API photocraft | https://api.github.com/repos/storytold/photocraft | curl 2026-10-13 | stargazers_count 57757; forks 9741; open_issues 1504; created 2026-09-30 |
| E10 | GitHub API vectorcraft | https://api.github.com/repos/storytold/vectorcraft | curl 2026-10-13 | stargazers 8452; forks 3968; open_issues 294; created 2026-09-30 |
| E11 | PhotoCraft scorecard | https://raw.githubusercontent.com/storytold/photocraft/main/docs/scorecard.md | web_extract | Content-Aware Fill/Spot Healing budgets; automation 631/631 menus; tools checklist |
| E12 | VectorCraft product highlights | E4 + E2 | combined | Image Trace; exact booleans; live effects |
| E13 | PhotoCraft gaps G15 | https://raw.githubusercontent.com/storytold/photocraft/main/docs/gaps.md | web_extract/python | G15 AI missing list; no ML runtime; 300–600 h + human #41 |
| E14 | PhotoCraft target-app-parity | https://raw.githubusercontent.com/storytold/photocraft/main/docs/target-app-parity.md | web_extract | ~76% breadth; ~45% ready; AI component ~15% / ~10% |
| E15 | PhotoCraft roadmap | https://raw.githubusercontent.com/storytold/photocraft/main/docs/roadmap.md | web_extract | M10 smart features classical done, ML not started; M11 MCP DoD; Later AI backend #41 |
| E16 | VectorCraft gaps G18 | https://raw.githubusercontent.com/storytold/vectorcraft/main/docs/gaps.md | raw+python | G18 Generative AI missing Adobe-list; 30–60 h after provider decision |
| E17 | VectorCraft README Status | E2 | | Alpha numbers; generative AI in missing list |
| E18 | PhotoCraft docs/mcp.md | https://raw.githubusercontent.com/storytold/photocraft/main/docs/mcp.md | web_extract | MCP tools, roots, security notes |
| E19 | PhotoCraft book automation/mcp | https://raw.githubusercontent.com/storytold/photocraft/main/book/src/automation/mcp.md | web_extract | Headless vs bridge backends |
| E20 | VectorCraft docs/mcp.md | https://raw.githubusercontent.com/storytold/vectorcraft/main/docs/mcp.md | web_extract | Headless/remote MCP; prompts; roots |
| E21 | GitHub API artcraft | https://api.github.com/repos/storytold/artcraft | curl | 25552 stars; “intentional crafting engine…” |
| E22 | GitHub org storytold | https://api.github.com/orgs/storytold | curl | “Open Source Tools and Models for Makers…”; 61 public repos |
| E23 | Releases API | api.github.com/.../releases | curl | PC v0.8.2 (2026-10-12); VC v0.10.0 (2026-10-12) |

---

## findings

### 1. Identity map

```
storytold (GitHub org) / ArtCraft Team
├── artcraft          → AI image/video studio (controllable gen + compose)
├── photocraft        → Photoshop-class raster editor (Crafting App)
├── vectorcraft       → Illustrator-class vector editor (Crafting App)
├── filmcraft, lightcraft, pdfcraft, effectcraft, designcraft,
│   figcraft, wordcraft, soundcraft, deckcraft, cadcraft, gridcraft
└── craft-launcher, craft-fonts, shared craft-* crates
```

**Trademark note (README):** ArtCraft name/logos are trademarks; forks must remove brand assets. Adobe trademarks used only for workflow comparison; no affiliation.

### 2. What “gen-AI capability” means here (critical distinction)

| Capability class | PhotoCraft | VectorCraft | Notes |
|------------------|------------|-------------|-------|
| Diffusion / Firefly-class generative fill | ❌ Missing (G15) | ❌ Missing (G18) | Needs owner model decision |
| Neural Filters / ML upscale / sky ML | ❌ | n/a | No ML runtime |
| Classical content-aware / heal / remove | ✅ Present | n/a (raster effects partial) | Patch / graph-cut style |
| Classical select subject/object | ✅ Classical only | n/a | Not ML Select Subject |
| Image Trace (raster→vector) | n/a | ✅ | Presets; not prompt-to-vector |
| Agent drives editor via MCP/CLI | ✅ Strong | ✅ Strong | Primary “AI” story |
| External ArtCraft studio models | Separate product | Separate product | Compose then edit in Craft apps |

### 3. Agent / automation surface (actionable)

**PhotoCraft**
```sh
photocraft-cli run input.psd --cmd <id> --params '...' --out out.png
photocraft-cli batch --actions grade.json --in ./raw --out ./graded
photocraft-cli mcp --automation-read-root /work --automation-write-root /work
# bridge: photocraft --control <port> + photocraft-cli mcp --bridge ...
```
Core MCP tools: `command_list`, `command_run`, `command_batch`, `doc_inspect`, `render_preview`.

**VectorCraft**
```sh
vectorcraft-cli mcp --headless
vectorcraft-cli mcp --connect 127.0.0.1:7979
vectorcraft-cli run --in file.vectorcraft --export out.pdf
claude mcp add vectorcraft -- vectorcraft-cli mcp --automation-read-root ... --automation-write-root ...
```
MCP prompts: `poster`, `icon-set`, `recolor`, `trace-and-style`, `export-set`.

**Security posture (both):** stdio MCP; explicit read/write roots; loopback control + token for bridge; docs warn against untrusted brokers.

### 4. Maturity / risk for production use

- **Very young public age** (~2 weeks from 2026-09-30 create to this report) despite enormous star counts — expect churn, incomplete PSD/.ai exchange, performance shortfalls (PC scorecard: many budgets unmet).
- PhotoCraft honesty: “not yet a Photoshop replacement for daily professional work.”
- Alpha gates pass for core workflows on both; beta needs fidelity/perf/bugs more than gen-AI.
- Third-party sites (photocraft.one, artcraft.wiki, getartcraft.app, etc.) may lag or rebrand; prefer `github.com/storytold/*` and `getartcraft.com`.

### 5. Practical integration patterns

1. **Agent retouch/compositing pipeline:** MCP PhotoCraft for layers/masks/export; keep generative steps in ArtCraft studio or local SD/Comfy.
2. **Agent illustration pipeline:** MCP VectorCraft command API (examples already built entirely via commands); Image Trace for raster logos → paths; no prompt-to-SVG native.
3. **Do not expect** Photoshop Generative Fill parity or Illustrator Text-to-Vector in-repo without new pluggable backend work + license decisions.

---

## gaps

| gap | detail |
|-----|--------|
| Live binary verification | Did not install/run `photocraft-cli`/`vectorcraft-cli` in this container; capabilities inferred from docs/tests claims |
| Full MCP tool catalog dump | PhotoCraft docs list core five + more; VectorCraft mcp.md is large — not every tool enumerated here |
| Issue #41 contents | Owner AI backend decision not fully extracted beyond gaps/roadmap references |
| ArtCraft studio model list | Studio page names Seedance / Nano Banana etc.; not exhaustively inventoried for this PC/VC-focused task |
| Star inflation / authenticity | Extraordinary star velocity not independently audited |
| Release page vs product page version skew | Product pages showed 0.7.0/0.9.0 while Releases API showed 0.8.2/0.10.0 — marketing pages may lag |

---

## confidence

- **level:** high  
- **rationale:** Core identity, license, missing gen-AI, and MCP/agent design are multi-source confirmed (README + gaps + roadmap + parity + product site + API). Self-scored parity percentages are first-party estimates, not third-party benchmarks — treated as supported *as self-assessment*, not as absolute ground truth. No local binary exercise slightly reduces confidence on runtime behavior.

---

## recommended_next_actions

1. **If the goal is local gen-AI pixels:** evaluate `storytold/artcraft` + external local models; do not wait on PhotoCraft G15.
2. **If the goal is agent-editable PSD/SVG pipelines:** pilot `photocraft-cli mcp` / `vectorcraft-cli mcp` with scoped roots; script 5–10 golden tasks (open → adjust → export).
3. **Track** PhotoCraft #41 / G15 and VectorCraft G18 for pluggable model providers.
4. **Pin versions** carefully (weekly releases); prefer GitHub Releases SHA256SUMS over third-party mirrors.
5. **Optional follow-up task:** deep-dive ArtCraft studio model catalog + licensing for a full “Storytold AI stack” map.

---

## appendix appendix (selected)

### PhotoCraft G15 (gaps.md)
> Missing: Generative Fill and Expand, Generative Upscale, Neural Filters, Sky Replacement, Super Zoom, ML Select Subject/Sky. There is no ML runtime (AUTO-219-7). Classical substitutes exist (GrabCut/max-flow selection, Remove tool by non-local patch completion). Estimate: 300–600 h. Human: owner decision #41.

### VectorCraft G18 (gaps.md)
> Missing: Text to Vector Graphic, Generative Recolor, Generative Shape Fill (#766), Generative Expand, Concept to Vector, Prompt to Edit, Rewrite Story. Adobe's run on its cloud models; ours would be a pluggable provider. Estimate: 30–60 h after that decision.

### PhotoCraft README status
> Biggest gaps are AI/generative features, 15 missing tools, depth in typography and pro workflows, and plug-in compatibility.

### VectorCraft README missing
> generative AI features.

---

*Report generated for kanban task t_8d468a34. Single durable archive path below.*
