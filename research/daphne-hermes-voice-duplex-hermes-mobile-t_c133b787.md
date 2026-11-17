# [GIU] Daphne (Daphne / hermes-mobile plugin / voice duplex) — Research Investigation

**Task:** t_c133b787 · **Board:** research · **Agent:** Geraldov21 · **Date:** 2026-11-17 (MST)

---

## Objective

Recover project context for "Daphne" (voice duplex / hermes-mobile plugin) and deliver a structured research report on Hermes full-duplex voice architecture: current voice mode state, the hermes-mobile plugin ecosystem, and the full-duplex options that fit a Hermes iPhone app (local full-duplex model vs chained STT→LLM→TTS with barge-in).

## Summary

No Huly issue, doc, card, or project named "Daphne" exists in the Huly index (verified via Julio `search_huly` with 3 query variants — zero hits). The term does not appear in any prior Geraldo report in `/home/ice/know/research/`. The name is therefore either an unrecorded internal alias Frank uses, or a confusion with an existing parked project. The closest real artifacts are: (1) the **parked "Hermes Full-Duplex Voice AI Pipeline" architecture blueprint** in Huly (swarm-crew-docs doc 6aa4b8d25386a86803c5bb5d, captured 2026-09-12 from Frank's Discord #gen "Full duplex VA" thread — Pipecat/LiveKit + Silero VAD + faster-whisper + Kokoro/XTTSv2, dual RTX 3090 split, Hermes as cognition backend only); (2) the community **`dylan-buck/Hermes-iOS`** repo (122★, active) — a self-hosted iPhone app + Python relay + host connector whose "Talk mode" runs **OpenAI Realtime (`gpt-realtime-1.5`/`gpt-realtime`) over WebRTC** with a `hermes_delegate` tool that defers real work to Hermes; (3) the **`geniok1980/hermes-mobile-adapter`** plugin (5★, pushed 2026-11-16 — yesterday) — a Hermes gateway platform plugin exposing `ws://host:8787/ws` for any mobile app (text, voice notes WAV/OGG, media, approval cards, avatar states) with TS/Python SDKs. Hermes itself has shipped substantial first-party voice: chained voice mode with VAD barge-in on every surface, streaming STT, sentence-by-sentence streaming TTS, a true full-duplex desktop mode (`gpt-live-1` via WebRTC that delegates to Hermes), a "Hey Hermes" wake word, and a fast-model `voice_chat` auxiliary slot. Local full-duplex model options: **Moshi (Kyutai)** is the reference — Apache-2.0, 160ms theoretical / ~200ms practical latency, 24GB GPU (PyTorch), 1.1 kbps Mimi codec; **Qwen2.5-Omni-7B** (~7.1GB Q4, fits 12GB card) is a strong local omni model but is turn-based chat, not continuous full-duplex.

## Claims

### C1 — No "Daphne" exists in Huly or prior research
- statement: Huly search (3 query variants: "Daphne voice duplex hermes-mobile iOS", "Daphne project", "voice iPhone app hermes mobile") returns no Daphne issue/doc/card. No prior know/ report references it.
- evidence: E1, E2
- confidence: high
- status: supported

### C2 — Parked full-duplex blueprint matches the "Daphne" description (voice duplex + Hermes)
- statement: Huly doc 6aa4b8d25386a86803c5bb5d ("Hermes Full-Duplex Voice AI Pipeline — Architecture Blueprint (parked project)") is the durable record of Frank's full-duplex voice project: WebRTC + Silero VAD barge-in fast loop, Hermes as slow-loop cognition, Pipecat or LiveKit orchestration, faster-whisper STT, Kokoro/XTTSv2 TTS, dual 3090 split (cognitive node GPU0, reflex node GPU1). Status: parked, no work started. Blueprint claims a "Kortexa LiveKit plugin for Hermes" that must be verified before use.
- evidence: E3
- confidence: high
- status: supported

### C3 — Hermes-iOS (community) is the existing Hermes iPhone app, and its voice is NOT local full-duplex
- statement: dylan-buck/Hermes-iOS (122★, created 2026-03-31, last push 2026-10-02) = Swift app + Python relay (control plane: pairing, jobs, SSE, APNs) + host-side connector (WebSocket jobs/RPC, MCP `hermes_mobile` server, SQLite sensor store, background service). Talk mode: app does WebRTC SDP exchange with `https://api.openai.com/v1/realtime/calls?model=...` using ephemeral keys bootstrapped by the relay; default models `gpt-realtime-1.5`/`gpt-realtime`, voice `ballad`; the voice model's only tool is `hermes_delegate` which runs real work through the connector→Hermes. iOS background audio is wired (WebRTC stays alive on app switch, Live Activities, CarPlay optional).
- evidence: E4, E5, E6, E7
- confidence: high
- status: supported

### C4 — hermes-mobile-adapter is a real, fresh Hermes plugin for custom mobile clients
- statement: geniok1980/hermes-mobile-adapter (5★, created 2026-10-11, last push 2026-11-16) installs as `~/.hermes/plugins/hermes-mobile`, registers a `mobile` gateway platform listening on port 8787 `/ws` (token auth, pairing approval via `hermes pairing approve mobile <id>`). Protocol v1 frames: text, audio (WAV/OGG up, agent voice down, 8MB cap), media both directions, approval/clarify cards (`prompt`), message streaming edits, avatar states (thinking→speaking→idle). Ships TypeScript (`hermes-mobile-client` npm) and Python SDKs, web + RN examples, 130 tests. Docs in Russian; protocol in English. Voice here is voice-note (record→transcribe via Hermes STT→reply TTS), i.e. chained half-duplex — not continuous full-duplex.
- evidence: E8, E9
- confidence: high
- status: supported

### C5 — Hermes first-party voice state (as of Nov 2026 docs)
- statement: Hermes Agent (NousResearch, ~264k★, pushed 2026-11-17) voice features: (a) CLI/TUI/Desktop push-to-talk chained voice with two-stage VAD silence detection; (b) **barge-in on all surfaces** — mic stays live during playback, interrupting speech cuts playback and submits, noise floor calibrated against quiet room, `voice.barge_in_threshold_multiplier` default 3.0; (c) streaming STT (OpenAI `gpt-live-transcribe`, xAI, ElevenLabs Scribe v2) with fallback to file path; (d) sentence-by-sentence streaming TTS on all providers; (e) **GPT-Live full-duplex desktop mode** (`voice.voice_chat_mode: gpt-live`, OpenAI `gpt-live-1`) — one full-duplex voice model handling interruptions/backchannels, delegates real work to Hermes via `session.delegation.created` → normal chat turn; billed $0.05/min by OpenAI; (f) "Hey Hermes" wake word (local hotword, hands-free session start); (g) `auxiliary.voice_chat` fast-model slot with reasoning off by default for voice turns; (h) local STT (faster-whisper, zero API keys) and local/free TTS (Edge, NeuTTS) options.
- evidence: E10, E11, E12
- confidence: high
- status: supported

### C6 — Local full-duplex model options
- statement: **Moshi (Kyutai)** — Apache-2.0, 11.5k★, the reference real-time full-duplex spoken LM: RQ-Transformer over Mimi streaming codec (1.1 kbps, 80ms frame), theoretical 160ms / ~200ms practical latency (on L4), backchannels/interruption native. PyTorch interactive mode needs ~24GB VRAM GPU (per README); MLX (Apple Silicon) and Rust builds exist. **Qwen2.5-Omni-7B** — 11B params, ~7.1GB at Q4_K_M, fits 12GB cards (e.g. RTX 3060/4060); omni (speech+image in, speech out) but operates as turn-based chat, not a continuous full-duplex duplex engine — good as the LLM brain of a Pipecat-style pipeline, not as the duplex transport itself.
- evidence: E13, E14, E15
- confidence: medium (VRAM figures are README/secondary; Qwen omni duplex-capability caveat is inference from architecture docs)
- status: supported (with noted caveats)

### C7 — Hardware fit for the parked blueprint
- statement: The blueprint's dual-loop design maps onto the existing dual RTX 3090 (24GB each) AI Box: GPU0 = Hermes cognition (llama.cpp :8088 already serving), GPU1 = reflex node (faster-whisper, streaming TTS, Pipecat/WebRTC). Existing assets named: Parakeet STT :8772, faster-whisper containers, dual-3090 box. Not yet present: Pipecat/LiveKit, continuous WebRTC loop with Silero VAD, Kokoro/XTTSv2 streaming TTS, Hermes↔Pipecat `[interrupted]` bridge.
- evidence: E3
- confidence: high
- status: supported

### C8 — Recommended path for a Hermes iPhone app with full-duplex voice
- statement: Three viable tiers:
  1. **Ship fast (weeks):** reuse dylan-buck/Hermes-iOS as-is (self-hosted relay + connector, OpenAI Realtime talk) — real-time voice on iPhone today, but audio pipeline is OpenAI-hosted (not 100% sovereign) and billed per session.
  2. **Custom client, chained voice (weeks):** hermes-mobile-adapter (8787 WS) + own app; voice = voice-note loop (record→STT→agent→TTS) with Hermes barge-in semantics on the server; fully self-hosted but half-duplex on phone.
  3. **True full-duplex, sovereign (the parked blueprint, months):** Pipecat/LiveKit reflex node on GPU1 + Silero VAD barge-in + Hermes slow loop; client connects over WebRTC from the iPhone (the Hermes-iOS app already demonstrates the WebRTC client pattern; the blueprint's transport is host-side). Moshi is an alternative single-model duplex engine if a 24GB GPU is dedicated to it — but it lacks Hermes tool-calling integration (it is the conversation itself), so it competes with rather than complements the blueprint.
- evidence: E3, E4, E5, E6, E8, E10, E13
- confidence: medium (engineering recommendation, grounded in the cited artifacts)
- status: supported (recommendation, not fact)

## Evidence Index

- **E1** — Julio `search_huly` "Daphne voice duplex hermes-mobile iOS": 4 results, none named Daphne (Huly workbench). Provenance: MCP query 2026-11-17, raw JSON captured in-run.
- **E2** — `ls /home/ice/know/research/ | grep -iE daphne|iphone|duplex|hermes-ios|voice`: only `business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md` (unrelated, Oct 2025). Provenance: container terminal, know/ mount.
- **E3** — Huly doc 6aa4b8d25386a86803c5bb5d, swarm-crew-docs, "Hermes Full-Duplex Voice AI Pipeline — Architecture Blueprint (parked project)", modified 2026-09-12. Provenance: `mcp__julio__read_document` full text retrieved 2026-11-17. Durable copies: `/srv/app-data/huly-artifacts/ops/20260911-Hermes_FullDuplex_Architecture.pdf` + `20260911-hermes-fullduplex-voice-blueprint.md`.
- **E4** — dylan-buck/Hermes-iOS `README.md` (master, 8694 bytes). Provenance: raw.githubusercontent fetch, HTTP 200, 2026-11-17.
- **E5** — dylan-buck/Hermes-iOS `relay/README.md`, `connector/README.md`, `docs/CONFIGURATION.md`, `relay/.env.example`. Provenance: raw.githubusercontent fetches, HTTP 200.
- **E6** — dylan-buck/Hermes-iOS `HermesMobile/Services/Live/LiveVoiceSessionService.swift`: WebRTC `RTCPeerConnection`, SDP upload to `https://api.openai.com/v1/realtime/calls?model=...`, ephemeral client secret from relay bootstrap. Provenance: raw.githubusercontent fetch, HTTP 200 (48659 bytes).
- **E7** — dylan-buck/Hermes-iOS `connector/src/hermes_mobile_connector/talk_support.py`: `DEFAULT_REALTIME_MODELS = ["gpt-realtime-1.5", "gpt-realtime"]`, `DEFAULT_REALTIME_VOICE = "ballad"`, `hermes_delegate` delegation rules in voice system prompt, sensor-freshness + memory context injection. Provenance: raw.githubusercontent fetch, HTTP 200.
- **E8** — geniok1980/hermes-mobile-adapter `README.md` (main, 11551 bytes). Provenance: raw.githubusercontent fetch, HTTP 200.
- **E9** — geniok1980/hermes-mobile-adapter `site-docs/protocol/index.md` (frame spec, MAX_AUDIO_BYTES 8MiB, capabilities JSON). Provenance: raw.githubusercontent fetch, HTTP 200.
- **E10** — NousResearch/hermes-agent `website/docs/user-guide/features/voice-mode.md` (660 lines): chained voice, barge-in section, GPT-Live mode, voice_chat aux slot, STT/TTS provider matrix. Provenance: raw.githubusercontent fetch, HTTP 200.
- **E11** — NousResearch/hermes-agent `website/docs/guides/use-voice-mode-with-hermes.md`. Provenance: raw.githubusercontent fetch, HTTP 200.
- **E12** — NousResearch/hermes-agent `website/docs/user-guide/features/wake-word.md`. Provenance: raw.githubusercontent fetch, HTTP 200.
- **E13** — kyutai-labs/moshi `README.md` + Hugging Face model card: 160ms theoretical / 200ms practical latency on L4, Mimi 1.1 kbps streaming codec, "GPU with a significant amount of memory (24GB)" for PyTorch interactive. License Apache-2.0, 11542★, pushed 2026-09-09. Provenance: raw.githubusercontent fetch (200) + GitHub API + web_search snippets (huggingface.co/kyutai/moshika-pytorch-bf16).
- **E14** — Qwen2.5-Omni-7B VRAM: ~7.08–7.1GB at Q4_K_M (llmrun.dev model page, modeldrops.com, via web_search). Provenance: web_search snippets 2026-11-17 (secondary sources; official Qwen card not fetched — gap).
- **E15** — GitHub API repo metadata (stars/pushed/created): dylan-buck/Hermes-iOS (122, 2026-10-02), geniok1980/hermes-mobile-adapter (5, 2026-11-16), kyutai-labs/moshi (11542, 2026-09-09), NousResearch/hermes-agent (263987, 2026-11-17). Provenance: api.github.com, 2026-11-17.

## Gaps

1. **Identity of "Daphne" unresolved.** No Huly artifact or prior report carries the name. Possible interpretations: (a) Frank's working name for the parked full-duplex blueprint (E3); (b) a proposed name for the iPhone-app track (E4/E8); (c) an entirely new idea not yet in Huly. **Needs Frank to confirm which, or a Huly issue reference.**
2. **Kortexa LiveKit/Hermes plugin** (named in blueprint E3) — existence unverified; blueprint itself flags "VERIFY EXISTENCE before relying on it."
3. **Qwen2.5-Omni-7B official spec** — VRAM figures from secondary sites; official model card not fetched.
4. **Huly doc read URLs are localhost workbench links** — external sharing should use `https://huly.poop.quest/...` equivalents; exact public URLs not constructed here.
5. **Moshi 24GB figure** is the README's guidance for PyTorch interactive mode; quantized/MLX footprints could be lower but were not benchmarked.
6. No benchmark/latency measurements were run (no local GPU access from this research container); all latency figures are vendor-reported.

## Confidence

- level: **high** for C1–C5 (direct document/source retrieval, all HTTP 200 or MCP reads, cross-checked against multiple files per repo); **medium** for C6–C8 (hardware figures from secondary sources; architecture recommendation is reasoned synthesis).
- rationale: Every factual claim maps to a retrieved primary artifact (repo source files, official docs, Huly doc, GitHub API). The only non-trivial gap is the Daphne identity itself, which no source can resolve without Frank.

## Recommended Next Actions

1. **Ask Frank** (or file a Huly clarification via `[HULY]` swarm-ops task): "Daphne" = which artifact? If it's the parked blueprint → un-park it (entry point: Phase 1, Pipecat env + WebRTC transport, per E3). If it's a new iPhone-app project → create a Huly issue linking E3, E4, E8.
2. **Verify the Kortexa LiveKit plugin** claim before any build (blueprint's own open item).
3. **If pursuing the iPhone app:** pilot dylan-buck/Hermes-iOS against the existing Hermes install (relay on a VPS/Fly + connector next to the Hermes host) as the fastest path to a working phone voice experience (C8 tier 1) — it demonstrates the WebRTC client + background audio the blueprint will need anyway.
4. **If pursuing true full-duplex:** stand up Phase 1 on the AI Box per E3 (Pipecat env on GPU1, WebRTC transport), reusing llama.cpp :8088 as the cognitive node; benchmark Moshi on a spare 24GB card as a comparison datapoint.
5. **File a Huly research record** of this report (via `[HULY]` kanban task for Julio drain) so the Daphne investigation is durable.

---
*Extraction paths: web_search (Brave) for discovery; raw.githubusercontent.com curl fetches (HTTP 200) for all repo source files (markdown-first, no browser fallback needed — Firecrawl was 403ing); api.github.com for repo metadata; Julio MCP (`search_huly`, `read_document`) for Huly state; container terminal for local know/ scan. No browser tooling was required.*
