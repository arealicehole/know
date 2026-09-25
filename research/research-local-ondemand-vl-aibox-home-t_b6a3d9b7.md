# Local/On-Demand VL Models: Home 5060 Ti + AI Box Side-by-Side Reality

**Task:** t_b6a3d9b7
**Date:** 2026-09-25
**Author:** Geraldo v2.1 (GIU)
**Status:** Complete

---

## Objective

Identify the best open-weight vision-language models for Hermes Discord/Telegram screenshot describe + light UI/OCR as: (1) on-demand spin-up on home RTX 5060 Ti 16GB (preferred), (2) side-by-side on AI Box dual-3090 when A2 Qwen is not live, (3) optional serverless comparison. Deliver a ranked shortlist with VRAM fit matrix, on-demand architecture modeled on Parakeet, and Hermes wiring guidance.

**Hard constraint:** Qwen3.8-27B fleet primary on AI Box (`:8088`, Path A2 vLLM AutoRound W4A16) keeps vision OFF. Do NOT recommend enabling mm on the live A2 primary.

---

## Plan

| Task ID | Title | Status |
|---------|-------|--------|
| T1 | Survey top open-weight VL models (2025-2026) | done |
| T2 | Identify smallest/densest useful VL candidates | done |
| T3 | Build home 5060 Ti 16GB VRAM fit matrix | done |
| T4 | AI Box side-by-side reality check | done |
| T5 | Design on-demand architecture (Parakeet twin) | done |
| T6 | Hermes wiring: aux.vision to local endpoint | done |

---

## Findings

### 1. Best Overall Open-VL Models (2025-2026) for Agent Screenshot/OCR/UI

Ranked for the specific use case of Hermes Discord/Telegram screenshot describe + light UI/OCR (not MMMU bragging):

| Rank | Model | Params | License | Strengths | Weaknesses |
|------|-------|--------|---------|-----------|------------|
| 1 | **Qwen3-VL-8B-Instruct** | 8B (9B on HF) | Apache 2.0 | Best-in-class OCR, native 256K context, strong GUI/agent capabilities (recognizes elements, invokes tools, completes tasks), 32 languages, video support, excellent multi-image. 19M downloads/mo on HF. | 8B dense — needs ~6-8 GB at Q4, tight on 16GB with context |
| 2 | **MiniCPM-V 4.5** | 8B (8.7B total) | Apache 2.0 | OpenCompass avg 77.0 (beats GPT-4o-latest, Qwen2.5-VL-72B), 3D-Resampler 96x video compression, LLaVA-UHD 4x less visual tokens, fast/deep thinking modes, runs on llama.cpp/ollama/vLLM/SGLang | 8B — similar VRAM footprint to Qwen3-VL-8B |
| 3 | **Qwen3-VL-4B-Instruct** | 4B | Apache 2.0 | Same architecture as 8B at half the size, strong OCR, 256K context, GUI agent capabilities, 2.9M+ downloads/mo | 4B — slightly weaker reasoning than 8B, still needs ~3-4 GB at Q4 |
| 4 | **Gemma 4 12B** | 12B | Apache 2.0 | Native multimodal (image, video, audio on 12B), 256K context, 140+ languages, strong reasoning, configurable thinking modes, ~8 GB at Q4 | 12B — needs ~12 GB with context on 16GB, tight |
| 5 | **Gemma 4 26B-A4B (MoE)** | 26B (4B active) | Apache 2.0 | MoE = 4B active compute speed, 16 GB weights at Q4, native multimodal, 256K context, audio | 16 GB — won't fit 16GB card with context; needs 24GB |
| 6 | **Qwen3-VL-2B-Instruct** | 2B | Apache 2.0 | Smallest in Qwen3-VL family, 256K context, GUI agent, 2.9M downloads/mo, ~2 GB at Q4 | 2B — weakest OCR/reasoning, acceptable for simple describe |
| 7 | **LLaVA-OneVision-1.5-8B-Instruct** | 8B | Apache 2.0 | Fully open (weights+data+code), strong multi-image/video, SigLIP encoder | No native agent/GUI mode, no thinking modes |
| 8 | **Molmo-7B-D-0924** | 7B | Apache 2.0 (fully open: weights+data+code) | Allen AI fully-open, strong grounding (bounding boxes), between GPT-4V and GPT-4o | No agent/GUI mode, older generation (Sep 2024) |

**Key insight for agent screenshot/OCR/UI:** Qwen3-VL family has the strongest native agent/GUI capabilities — "operates PC/mobile GUIs, recognizes elements, understands functions, invokes tools, completes tasks." MiniCPM-V 4.5 is the best-performing sub-30B model on OpenCompass. Gemma 4 12B is the best "all-rounder" at 12B with native audio. For pure OCR, Qwen3-VL-8B is the best open-weight scorer.

### 2. Best Smallest/Densest VL Still Useful for Discord Screenshots + UI Text

| Model | Params | Q4 VRAM | Verdict |
|-------|--------|---------|---------|
| **Qwen3-VL-4B-Instruct** | 4B | ~3 GB weights, ~5-6 GB total | **Best balance** — strong OCR, 256K context, agent/GUI, fits easily in 16GB with room for Parakeet |
| **Qwen3-VL-2B-Instruct** | 2B | ~2 GB weights, ~3-4 GB total | Smallest useful Qwen3-VL — fine for simple describe, weaker on complex OCR |
| **MiniCPM-V 4.5 (8B)** | 8B | ~6 GB weights, ~8-9 GB total | Best sub-30B on OpenCompass, 96x video compression — tight on 16GB with Parakeet |
| **Gemma 4 E4B** | 4.5B | ~4 GB | Native multimodal + audio, 8GB GPU floor, very efficient |
| **Gemma 4 12B** | 12B | ~8 GB weights, ~12 GB total | Best 12B all-rounder, tight on 16GB with other models |

**Recommendation:** Qwen3-VL-4B-Instruct is the sweet spot for Discord screenshot describe + light UI/OCR. It has the same architecture and capabilities as the 8B (just at 4B scale), 256K context, native GUI agent mode, and fits comfortably in ~5-6 GB of VRAM, leaving ~10 GB headroom on the 5060 Ti for Parakeet ASR and other services.

### 3. Home 5060 Ti 16GB Fit Matrix

Current state: ~3.6 GB used (Parakeet ASR) / ~12.2 GB free.

| Model | Params | Quant | Est VRAM (weights) | Est VRAM (with context) | Engine | Cold Start Class | Fit on 5060 Ti 16GB |
|-------|--------|-------|-------------------|------------------------|--------|-----------------|---------------------|
| Qwen3-VL-2B-Instruct | 2B | Q4_K_M | ~2 GB | ~3-4 GB | llama.cpp+mmproj / ollama | ~10-20s | **YES** — leaves ~9 GB for Parakeet |
| Qwen3-VL-4B-Instruct | 4B | Q4_K_M | ~3 GB | ~5-6 GB | llama.cpp+mmproj / ollama / vLLM | ~15-30s | **YES** — leaves ~7 GB for Parakeet |
| Qwen3-VL-8B-Instruct | 8B | Q4_K_M | ~6 GB | ~8-9 GB | llama.cpp+mmproj / vLLM / SGLang | ~30-60s | **TIGHT** — ~3-4 GB left for Parakeet (Parakeet needs ~3.6 GB) |
| MiniCPM-V 4.5 | 8B | Q4_K_M | ~6 GB | ~8-9 GB | llama.cpp / ollama / vLLM / SGLang | ~30-60s | **TIGHT** — same as Qwen3-VL-8B |
| Gemma 4 E4B | 4.5B | Q4_K_M | ~4 GB | ~6-7 GB | ollama / vLLM | ~15-30s | **YES** — leaves ~6 GB for Parakeet |
| Gemma 4 12B | 12B | Q4_K_M | ~8 GB | ~11-12 GB | ollama / vLLM | ~30-60s | **TIGHT** — ~1-2 GB left, barely with Parakeet |
| Gemma 4 26B-A4B | 26B | Q4_K_M | ~16 GB | ~20+ GB | vLLM / SGLang | ~60-90s | **NO** — won't fit on 16GB |
| Gemma 4 31B | 31B | Q4_K_M | ~20 GB | ~25+ GB | vLLM / SGLang | ~90s+ | **NO** — needs 24GB+ |

**VRAM budget on 5060 Ti 16GB:**
- Parakeet ASR: ~3.6 GB (persistent)
- Available for VL: ~12.2 GB
- Qwen3-VL-4B Q4 + 8K context: ~6 GB → leaves ~6 GB for OS/display/other → **comfortable**
- Qwen3-VL-8B Q4 + 8K context: ~9 GB → leaves ~3 GB → **tight but workable** (Parakeet can park)
- Gemma 4 12B Q4 + 8K context: ~12 GB → leaves ~0-1 GB → **too tight with Parakeet active**

**Recommendation for 5060 Ti:** Qwen3-VL-4B-Instruct at Q4_K_M (or Q5_K_M for better quality if VRAM allows) as the primary VL. Qwen3-VL-8B as a "stretch" option when Parakeet is parked.

### 4. AI Box Side-by-Side Reality Check

**Current state:** A2 Qwen3.8-27B owns BOTH 3090s (~23.2 + 23.1 GB used, ~0.8-1.1 GB free per card). **Side-by-side VL while A2 is live: NO.** There is no meaningful VRAM headroom.

**When side-by-side WOULD work:**

| Scenario | VRAM Free | VL Feasible? | Notes |
|----------|----------|--------------|-------|
| A2 Qwen live (current) | ~0.8-1.1 GB/card | **NO** | Not enough for any VL model |
| Path D (llama.cpp AD-Q4) on one 3090 | ~14-16 GB free on one card | **YES** — 8B VL at Q4 fits | llama.cpp AD-Q4 of 27B ≈ ~14 GB → one 3090 has ~10 GB free (tight), other 3090 fully free for VL |
| Qwen stopped entirely | ~24 GB/card × 2 | **YES** — any VL up to 32B | Full 48 GB available |
| Qwen on one 3090 only (single-GPU mode) | ~24 GB on second card | **YES** — 8B-32B VL on free card | Qwen3.8-27B Q4 ≈ ~14 GB on one 3090, other 3090 free |

**Key constraint:** No CPU weight offload allowed. One process owns :8088. Consent needed before stop/swap.

**Realistic AI Box VL options (when A2 not live):**
- **Qwen3-VL-8B at Q4_K_M (~6 GB)** on one 3090 while Qwen runs on the other → **best side-by-side candidate**
- **MiniCPM-V 4.5 at Q4_K_M (~6 GB)** → same
- **Qwen3-VL-32B at Q4_K_M (~20 GB)** on a single free 3090 → **only if Qwen is stopped or on the other card**
- **Gemma 4 26B-A4B at Q4_K_M (~16 GB)** on one free 3090 → tight but possible

**Verdict:** Side-by-side on AI Box while A2 Qwen is live is a **fantasy** (~1 GB free). The only realistic AI Box VL path is: (a) stop A2 Qwen temporarily, or (b) move Qwen to one 3090 and run VL on the other. Both require ops consent and interrupt fleet service. **Home 5060 Ti is the correct on-demand VL location.**

### 5. On-Demand Architecture (Parakeet Twin)

Model on the Parakeet `:8772` pattern: systemd user unit, drop-in HTTP, cold start ~30-90s, VRAM-aware park/hold locks, start/stop without touching AI Box.

**Proposed: `vl-api` systemd user unit on home Fedora**

```
[Unit]
Description=On-demand VL API (Qwen3-VL-4B) for Hermes aux.vision
After=network.target parakeet-api.service
Wants=parakeet-api.service

[Service]
Type=simple
User=ice
WorkingDirectory=/home/ice/vl-api
ExecStart=/usr/bin/python3 /home/ice/vl-api/server.py \
  --model /home/ice/models/Qwen3-VL-4B-Instruct-Q4_K_M \
  --port 8773 \
  --ctx 8192 \
  --gpu 0
ExecStartPre=/home/ice/vl-api/check-vram.sh
Restart=on-failure
RestartSec=30
TimeoutStartSec=120
TimeoutStopSec=60

[Install]
WantedBy=default.target
```

**Port assignment:** `:8773` (next to Parakeet `:8772`, not 8088)

**Cold start estimate:** ~20-40s for Qwen3-VL-4B Q4 on 5060 Ti (model load ~15-25s + warmup ~5-15s). Similar to Parakeet's 30-90s range.

**VRAM park pattern (coexistence with Parakeet):**
- When VL is idle >5 min: park VL (release VRAM, keep model in RAM)
- When Parakeet needs VRAM: park VL first
- When VL is requested: unpark VL (load from RAM to VRAM ~5-10s), park Parakeet if needed
- Lock file: `/home/ice/vl-api/.park` (same pattern as Parakeet's `.park`)
- VRAM budget: 16 GB total - 1 GB system = 15 GB usable
  - Parakeet ASR: 3.6 GB (can park to RAM)
  - VL 4B Q4: 5-6 GB
  - Context/KV: 2-3 GB
  - OS/display: 1-2 GB
  - **Total: ~12-14 GB → fits with both active, park one if needed**

**OpenAI-compatible endpoint:**
- `POST http://localhost:8773/v1/chat/completions` — accepts image_url + text
- `POST http://localhost:8773/v1/images/describe` — simple describe (if using a wrapper)
- Health: `GET http://localhost:8773/health` → `{"status":"ok","vram_free_gb":X}`
- Park: `POST http://localhost:8773/park` → release VRAM
- Unpark: `POST http://localhost:8773/unpark` → load to VRAM

**Timer wake:** systemd timer `vl-api-wake.timer` — wake on demand (Hermes aux.vision request triggers unpark). Idle stop after 5 min via `vl-api-idle.timer`.

### 6. Hermes Wiring: aux.vision to Local Endpoint

**Current state:** Hermes uses auxiliary vision via `nous` / `deepseek/deepseek-v4.1-flash` describer path. Qwen text-only on AI Box.

**Proposed wiring:**

```yaml
# Hermes config (config.yaml)
vision:
  mode: aux  # keep as auxiliary, NOT native main
  aux:
    provider: openai-compatible
    base_url: http://localhost:8773/v1
    model: qwen3-vl-4b
    # Fallback to cloud if local is down:
    fallback:
      provider: nous
      model: deepseek/deepseek-v4.1-flash
```

**Flow:**
1. Hermes receives image (Discord/Telegram screenshot)
2. aux.vision checks `http://localhost:8773/health`
3. If up → send to local VL endpoint (Qwen3-VL-4B)
4. If down/timeout → fall back to cloud aux (deepseek-v4.1-flash)
5. Qwen text-only on AI Box stays untouched

**Key rule:** Qwen3.8-27B on AI Box `:8088` remains **text-only, vision OFF**. The local VL on home 5060 Ti is a **separate** path. They never share VRAM or GPU.

**Do NOT:**
- Enable mm on live A2 Qwen primary
- Run VL on AI Box while A2 is live
- Use 8088 or 8772 ports for VL

### 7. Serverless Comparison (Optional — Frank favors local)

| Provider | Model | Cost (per 1M tokens, approx) | Latency | Notes |
|----------|-------|------------------------------|---------|-------|
| Modal | Qwen2.5-VL-7B | ~$0.02-0.05 | ~2-5s (warm) | GPU serverless, spin up in ~30-60s cold |
| Modal | MiniCPM-V 4.5 | ~$0.02-0.05 | ~2-5s (warm) | Same |
| Featherless | Qwen3-VL-8B | ~$0.01-0.03 | ~1-3s (warm) | Inference provider on HF, pay-per-use |
| Novita | Qwen3-VL-8B | ~$0.01-0.03 | ~1-3s (warm) | Same |

**Verdict:** Local Qwen3-VL-4B on 5060 Ti is **free** (electricity only) vs $0.02-0.05 per 1M tokens on serverless. At moderate usage (50-100 images/day), local is cheaper after ~2-3 days of serverless use. Local also has zero data egress, full privacy, and no cold-start dependency on external services. **Local wins for this use case.**

---

## Claims with Evidence

### C1: Qwen3-VL family (2B/4B/8B/30B-A3B/32B) is Apache 2.0 licensed, 256K native context, with native GUI/agent capabilities

- **Evidence:** HF model cards for Qwen3-VL-2B, 4B, 8B — all show `License: apache-2.0`, "Native 256K context, expandable to 1M", "Operates PC/mobile GUIs—recognizes elements, understands functions, invokes tools, completes tasks"
- **Source URLs:**
  - https://huggingface.co/Qwen/Qwen3-VL-2B-Instruct
  - https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct
  - https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct
- **Provenance:** Direct HF model card extraction (browser)
- **Confidence:** High (primary source, 3 independent model cards confirm)

### C2: Qwen3-VL-8B-Instruct is 9B params (HF) / 8B (nominal), BF16, 19.1M downloads/month

- **Evidence:** HF model card: "Model size: 9B params, Tensor type: BF16, Downloads last month: 19,149,628"
- **Source URL:** https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct
- **Provenance:** Direct HF model card extraction
- **Confidence:** High

### C3: Qwen3-VL VRAM at Q4_K_M: 2B≈2 GB, 4B≈3 GB, 8B≈6 GB, 30B≈18 GB, 32B≈20 GB (weights only)

- **Evidence:** local-llm.net Qwen3-VL page: "The 2B needs about 2 GB of weights at Q4_K_M and the 235B about 130 GB" with table showing 2B=2GB, 4B=3GB, 8B=6GB, 30B=18GB, 32B=20GB
- **Source URL:** https://www.local-llm.net/models/qwen3-vl/
- **Provenance:** Browser extraction of local-llm.net
- **Confidence:** High (cites ollama + community benchmarks)

### C4: MiniCPM-V 4.5 is 8B (8.7B total) built on Qwen3-8B + SigLIP2-400M, OpenCompass avg 77.0, Apache 2.0, supports llama.cpp/ollama/vLLM/SGLang

- **Evidence:** HF model card: "built on Qwen3-8B and SigLIP2-400M with a total of 8B parameters", "OpenCompass avg score 77.0", "License: apache-2.0", "llama.cpp and ollama support for efficient CPU inference", "SGLang and vLLM support"
- **Source URL:** https://huggingface.co/openbmb/MiniCPM-V-4_5
- **Provenance:** Direct HF model card extraction
- **Confidence:** High

### C5: Gemma 4 family (E2B/E4B/12B/26B-A4B/31B) is Apache 2.0, natively multimodal (image+video+audio), 256K context, released Apr 2026

- **Evidence:** Google AI Developers model card: "Gemma 4 released with text, audio and image input and long up to 256K context window", "Apache 2.0", "five distinct sizes: E2B, E4B, 12B, 26B A4B, and 31B", "Processes Text, Image with variable aspect ratio and resolution support (all models), Video, and Audio"
- **Source URL:** https://ai.google.dev/gemma/docs/core/model_card_4
- **Provenance:** Direct Google AI Developers doc extraction
- **Confidence:** High (primary source)

### C6: Gemma 4 VRAM at Q4_K_M: E2B≈2.3 GB, E4B≈4 GB, 12B≈8 GB, 26B-A4B≈16 GB, 31B≈20 GB

- **Evidence:** modelfit.io GPU requirements page: "E2B (2.3 GB) runs on anything including phones. E4B (4 GB) is the 8GB-GPU pick. 12B (8 GB) is the sweet spot for 12-16GB GPUs. 26B-A4B (16 GB, MoE) wants 24GB. 31B (20 GB, dense) is the quality flagship for 24-32GB"
- **Source URL:** https://modelfit.io/blog/gemma-4-gpu-requirements/
- **Provenance:** Browser extraction
- **Confidence:** Medium-High (engine estimates, not measured)

### C7: Qwen3-VL is supported by llama.cpp (GGUF) as of Oct 30, 2025, vLLM, and SGLang

- **Evidence:** Unsloth docs: "Qwen3-VL is now supported for GGUFs by llama.cpp as of 30th October 2025"; HF Qwen3-VL-32B-Thinking-GGUF discussion: "They both supports Qwen3-VL series. llama.cpp: Download newer releases. vLLM: docs.vllm.ai"; SGLang docs: "Qwen/Qwen3-VL-30B-A3B-Instruct" in multimodal language models list
- **Source URLs:**
  - https://unsloth.ai/docs/models/tutorials/qwen3-how-to-run-and-fine-tune/qwen3-vl-how-to-run-and-fine-tune
  - https://huggingface.co/Qwen/Qwen3-VL-32B-Thinking-GGUF/discussions/2
  - https://docs.sglang.io/docs/supported-models/multimodal_language_models
- **Provenance:** Browser + search extraction
- **Confidence:** High (multiple independent sources)

### C8: Qwen3-VL-30B-A3B (MoE) needs ~22.8 GB at Q4_K_M — won't fit on 16GB card

- **Evidence:** willitrunai.com: "Qwen3-VL 30B A3B Instruct (30B parameters) requires approximately 22.8 GB of VRAM with Q4_K_M quantization"
- **Source URL:** https://willitrunai.com/models/qwen-3-vl-30b-a3b
- **Provenance:** Search result
- **Confidence:** Medium-High

### C9: Qwen3.8-27B (fleet primary) on AI Box uses both 3090s with ~0.8-1.1 GB free per card — no room for side-by-side VL

- **Evidence:** Task body (ops inventory stamped 2026-09-25): "qwen-path-a2 owns BOTH cards — ~23.2 + 23.1 GB used, ~0.8–1.1 GB free — NO headroom for side-by-side VL while A2 is up"
- **Source URL:** Kanban task t_b6a3d9b7 body
- **Provenance:** Ops inventory (primary source, same session)
- **Confidence:** High (ops-confirmed)

### C10: MiniCPM-V 4.5 can process high-res images up to 1.8M pixels using 4x less visual tokens than most MLLMs (LLaVA-UHD architecture)

- **Evidence:** HF model card: "Based on LLaVA-UHD architecture, MiniCPM-V 4.5 can process high-resolution images with any aspect ratio and up to 1.8 million pixels (e.g., 1344x1344), using 4x less visual tokens than most MLLMs"
- **Source URL:** https://huggingface.co/openbmb/MiniCPM-V-4_5
- **Provenance:** Direct HF model card extraction
- **Confidence:** High

### C11: LLaVA-OneVision-1.5-8B-Instruct is Apache 2.0, fully open framework, strong multi-image/video

- **Evidence:** HF model card: "LLaVA-OneVision-1.5: Fully Open Framework for Democratized Multimodal Training", arxiv 2509.23661
- **Source URL:** https://huggingface.co/lmms-lab/LLaVA-OneVision-1.5-8B-Instruct
- **Provenance:** Search + HF extraction
- **Confidence:** Medium-High

### C12: Qwen3-VL-2B has 2.9M downloads/month, 2B params, BF16

- **Evidence:** HF model card: "Model size: 2B params, Tensor type: BF16, Downloads last month: 2,960,646"
- **Source URL:** https://huggingface.co/Qwen/Qwen3-VL-2B-Instruct
- **Provenance:** Direct HF model card extraction
- **Confidence:** High

---

## Evidence Index

| ID | Source Label | Source URL | Provenance |
|----|-------------|------------|------------|
| E1 | Qwen3-VL-2B HF card | https://huggingface.co/Qwen/Qwen3-VL-2B-Instruct | Browser extraction |
| E2 | Qwen3-VL-4B HF card | https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct | Search (HF) |
| E3 | Qwen3-VL-8B HF card | https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct | Browser extraction |
| E4 | MiniCPM-V 4.5 HF card | https://huggingface.co/openbmb/MiniCPM-V-4_5 | Browser extraction |
| E5 | Gemma 4 model card | https://ai.google.dev/gemma/docs/core/model_card_4 | Browser extraction |
| E6 | local-llm.net Qwen3-VL | https://www.local-llm.net/models/qwen3-vl/ | Browser extraction |
| E7 | modelfit.io Gemma 4 VRAM | https://modelfit.io/blog/gemma-4-gpu-requirements/ | Browser extraction |
| E8 | Presenc AI VLM 2026 | https://presenc.ai/research/best-open-weight-vision-language-models-2026 | Browser extraction |
| E9 | Unsloth Qwen3-VL guide | https://unsloth.ai/docs/models/tutorials/qwen3-how-to-run-and-fine-tune/qwen3-vl-how-to-run-and-fine-tune | Search |
| E10 | SGLang multimodal docs | https://docs.sglang.io/docs/supported-models/multimodal_language_models | Search |
| E11 | willitrunai Qwen3-VL-30B-A3B | https://willitrunai.com/models/qwen-3-vl-30b-a3b | Search |
| E12 | MiniCPM-V 4.5 arXiv | https://arxiv.org/abs/2509.18154 | Search |
| E13 | LLaVA-OneVision-1.5 arXiv | https://arxiv.org/abs/2509.23661 | Search |
| E14 | Qwen3.8 AI Box ops inventory | (task t_b6a3d9b7 body) | Ops inventory |

---

## Gaps

1. **No measured VRAM on 5060 Ti specifically** — all VRAM figures are from model cards / community reports (mostly 3090/4090/Mac). The 5060 Ti is Blackwell (sm_120) with 16 GB GDDR7. Cold-start times are estimates (20-60s range based on model size), not measured on this specific card.
2. **Qwen3-VL-8B + Parakeet coexistence on 16GB** is theoretically tight (~9 GB + 3.6 GB = 12.6 GB) but not measured. May need Parakeet park/hold to be more aggressive.
3. **No direct benchmark of Qwen3-VL-4B vs MiniCPM-V 4.5 for Discord screenshot describe** — both are 8B-class (4B vs 8B). Qwen3-VL-4B has the edge in OCR + agent; MiniCPM-V 4.5 has the edge in OpenCompass overall. A head-to-head on real Discord screenshots would settle this.
4. **Gemma 4 E4B audio support** — confirmed on model card but not tested for Hermes audio describe use case.
5. **Modal/serverless latency for VL** — cited as ~2-5s warm but not measured on this network.

---

## Confidence

- **Level:** medium-high
- **Rationale:** Model specs, licenses, parameter counts, and VRAM estimates are from primary sources (HF model cards, Google AI docs, arXiv). Engine support (llama.cpp, vLLM, SGLang) is confirmed by multiple independent sources. The AI Box inventory is ops-confirmed. The main uncertainty is the exact VRAM fit on the 5060 Ti (Blackwell, 16 GB GDDR7) — estimates are extrapolated from 3090/4090 data. Cold-start times are estimates, not measured.

---

## Recommended Next Actions

1. **Download Qwen3-VL-4B-Instruct Q4_K_M GGUF** (from HF/ollama) to `/home/ice/models/` on home Fedora
2. **Write `vl-api` systemd user unit** (template in §5) with `:8773` port, OpenAI-compatible endpoint, park/hold locks
3. **Test cold start** on 5060 Ti — measure actual seconds, validate VRAM usage with `nvidia-smi`
4. **Wire Hermes aux.vision** to `http://localhost:8773/v1` with cloud fallback
5. **Head-to-head: Qwen3-VL-4B vs MiniCPM-V 4.5** on 5-10 real Discord/Telegram screenshots — pick the winner
6. **If 4B is too weak**, bump to Qwen3-VL-8B with Parakeet park/hold
7. **Do NOT** touch AI Box A2 Qwen — leave it as-is
8. **Optional:** Test Gemma 4 E4B as a lightweight alternative if Qwen3-VL-4B feels heavy

---

## Summary for Ops

**Top 3 ranked:**
1. **Qwen3-VL-4B-Instruct** (Apache 2.0, ~5-6 GB at Q4) — best balance of OCR + agent/GUI + 256K context, fits easily on 5060 Ti alongside Parakeet
2. **MiniCPM-V 4.5** (Apache 2.0, ~8-9 GB at Q4) — best sub-30B on OpenCompass, 96x video compression, tight but workable on 16GB with Parakeet parked
3. **Qwen3-VL-8B-Instruct** (Apache 2.0, ~8-9 GB at Q4) — best OCR in open weights, strongest agent capabilities, tight on 16GB with Parakeet

**Home vs AI Box:** Home 5060 Ti is the correct location for on-demand VL. AI Box side-by-side while A2 Qwen is live is impossible (~1 GB free). Only run VL on AI Box if Frank accepts stopping A2 or moving Qwen to a single 3090.

**Port:** `:8773` (next to Parakeet `:8772`)

**Do NOT enable mm on live A2 Qwen primary.**
