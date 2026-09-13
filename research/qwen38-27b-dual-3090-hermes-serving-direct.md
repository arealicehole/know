# Qwen3.8-27B on Dual RTX 3090 — Multi-Hermes Serving Plan

**Task:** direct research (no kanban dispatch env)  
**Date (UTC):** 2026-09-13  
**Agent:** Geraldo v2.2 (GIU)  
**Hardware target:** 2× RTX 3090 sm_86 Ampere, ~48 GB total / ~42–44 GB usable, 32 GB host RAM (operator), driver ~610, CUDA 13-class, PL 275 W, Omarchy/Arch ai-box, no NVLink assumed, no Blackwell sm_120 binaries.

---

## 1. Executive recommendation (≤15 lines)

1. **WINNER for multi-Hermes + one OpenAI endpoint:** **SGLang TP=2** with **AWQ-INT4** weights (`cyankiwi/Qwen3.8-27B-AWQ-INT4`) + **FP8 KV** (`--kv-cache-dtype fp8_e4m3`) + continuous batching.
2. **Why not stock 0xSero defaults alone:** validated 0xSero TP2 recipe is excellent for throughput (~113 t/s decode, ~1.5k prefill) but **max context ~64k** with BF16 KV pool ~119k tokens — **does not meet 256k goal**.
3. **256k path on 2×3090:** community dual-3090 WSL2 writeup launches **context-length 245760** with AWQ-INT4 + fp8 KV + TP2 (near-native). Treat **256k as achievable with fp8 KV**, not with BF16 KV on 2×24 GB.
4. **Ampere rule:** prefer **W4A16 AWQ/INT4 Marlin** over weight-FP8 for decode speed (no FP8 tensor cores on sm_86). FP8 is still useful for **KV storage**.
5. **Fallback A (full 262144 + best single-stream):** **llama.cpp** `Q4_K_M`/`AD-Q4_K_M` + `--cache-type-k/v q4_0` + optional MTP; multi-client via `-np` is weaker than continuous batch.
6. **Fallback B (continuous batch, alternate engine):** **vLLM TP2** AWQ/INT4 or requant recipes (syv-ai single-3090 proves long ctx; dual TP2 for headroom). FP8 weights OK for VRAM but Marlin W8A16 path on Ampere.
7. **Do not day-1 EXL3+DFlash2 for multi-tenant:** batch-1 / queue; great experimental 256k single-stream, fails continuous-batch goal.
8. **Hermes day-1:** server `--tool-call-parser qwen3_coder --reasoning-parser qwen3`; clients set `chat_template_kwargs.enable_thinking: false` for tool loops unless you need thinking; budget `max_tokens` high when thinking on.
9. **Primary deploy:** pin 0xSero container stack *or* bare SGLang main with dual-3090 long-ctx flags; expose `:8088`; systemd; PL 275 W; NCCL P2P optional.
10. **Concurrency policy:** size for **shared token pool**, not 20×256k; typical Hermes turns 4k–32k; cap `max-running-requests` ~4–16; client timeouts 600–1800s for long prefills.
11. **Quality primary:** AWQ-INT4 (agents) on SGLang; **AtomicChat AD-Q5_K_M / AD-Q4_K_M** on llama.cpp fallback; AD-Q6/Q8 only if dropping context or adding more VRAM headroom.
12. **Operator prior-art “~1.2M token capacity”:** treat as **pool-scale claim on larger TP / fp8-KV configs**, not as 2×3090 BF16 validated; 0xSero TP2 measured **~119k** BF16 pool / **64k** max ctx.

---

## 2. VRAM BOM — primary config (SGLang TP2, ~256k target)

Assumptions: 2×24 GB = 48 GB; leave ~2–3 GB/GPU driver+fragmentation → **~42–44 GB usable**. Hybrid GDN: **16 full-attention layers** dominate KV; Mamba/GDN state is a **separate concurrency tax**.

| Component | Estimate | Notes |
|---|---:|---|
| AWQ-INT4 target weights | ~15–19 GB total | 0xSero: ~19 GB AWQ on disk/load class; split by TP≈2 → ~9.5 GB/GPU |
| Optional DSpark draft | ~2.6 GB | Enable only if KV headroom remains; 0xSero enables on TP2 |
| Activations / graphs / workspace | ~2–4 GB/GPU | TP2: disable prefill CUDA graph; limit decode graph BS |
| Mamba/GDN state pool | 1–several GB | sizes **concurrency**, not only context |
| **KV pool (fp8_e4m3)** | remainder ~12–18 GB total | needed for ~200–256k **shared** tokens |
| **KV pool (bf16)** on TP2 | ~119k tokens measured | **cannot** hold one 256k sequence (0xSero) |
| Host RAM (32 GB) | critical | model download + Docker shm + page cache; avoid desktop compositor on GPUs |

**Token-pool mental model:** engines reserve a **global KV token budget**. One 256k job can consume the whole pool; ten 8k Hermes turns share it. Design for **sum(seq_lens) ≤ pool**, not `num_clients × 256k`.

**Rough concurrent capacity (primary SGLang fp8 KV, order-of-magnitude):**

| Typical prompt+history | Concurrent sequences (order) |
|---|---|
| 256k (rare full window) | 1 (maybe 1 + tiny) |
| 64k | ~3–4 |
| 32k | ~6–8 |
| 8k–16k Hermes turns | ~10–20 if pool ≥200k and GDN state allows |

0xSero TP2 BF16 measured: pool **118,693**, max concurrent **48** (GDN/mamba bound differs on TP4). With fp8 KV, pool tokens rise ~2× vs bf16 for same bytes — necessary for 256k.

---

## 3. Exact candidate commands (copy-paste)

### 3A. PRIMARY — SGLang TP2 AWQ + fp8 KV ~245k–256k (community dual-3090)

```bash
# Bare-metal / WSL-class launch (from dual-3090 community recipe; pin your paths)
python -m sglang.launch_server \
  --model-path /models/cyankiwi-Qwen3.8-27B-AWQ-INT4 \
  --served-model-name Qwen3.8-27B \
  --host 0.0.0.0 --port 8088 \
  --tp-size 2 \
  --quantization compressed-tensors \
  --mem-fraction-static 0.80 \
  --kv-cache-dtype fp8_e4m3 \
  --chunked-prefill-size 8192 \
  --context-length 245760 \
  --dtype bfloat16 \
  --mamba-ssm-dtype bfloat16 \
  --disable-custom-all-reduce \
  --enable-tf32-matmul \
  --schedule-policy lpm \
  --trust-remote-code \
  --tool-call-parser qwen3_coder \
  --reasoning-parser qwen3 \
  --max-running-requests 4 \
  --allow-auto-truncate \
  --cuda-graph-backend-prefill disabled \
  --stream-interval 1
# Optional speculative (VRAM permitting):
#   --speculative-algorithm DSPARK \
#   --speculative-draft-model-path /models/Qwen3.8-27B-DSpark \
#   --speculative-dspark-block-size 7 \
#   --speculative-draft-model-quantization unquant
```

**0xSero one-command container (throughput-first, 64k class — good Phase-1 soak):**

```bash
IMAGE='ghcr.io/0xsero/qwen38-3090-sglang@sha256:cc2bfc369c709c53d239ef2af41d4ae2530904975816bb2ca484188008586ac9'
docker run --gpus all --shm-size 32g -p 8088:8000 \
  -v /models/qwen38:/models -e TP=2 -e PORT=8000 \
  "$IMAGE"
```

### 3B. FALLBACK — llama.cpp full 262k + MTP (single-stream / low concurrency)

```bash
llama-server \
  -m /models/Qwen3.8-27B-AD-Q4_K_M.gguf \
  -c 262144 -ngl 999 -fa 1 \
  --split-mode tensor --tensor-split 1,1 \
  --cache-type-k q4_0 --cache-type-v q4_0 \
  --spec-type draft-mtp --spec-draft-n-max 2 \
  --parallel 1 \
  --host 0.0.0.0 --port 8088 \
  --jinja
# Multi-client without MTP advantage: raise --parallel 4..8, drop or keep MTP knowing gain collapses
```

### 3C. ALTERNATE continuous batch — vLLM TP2

```bash
# Ampere: AWQ/INT4 preferred for speed; FP8 weights save VRAM via Marlin W8A16 (not native FP8 math)
vllm serve cyankiwi/Qwen3.8-27B-AWQ-INT4 \
  --host 0.0.0.0 --port 8088 \
  --tensor-parallel-size 2 \
  --max-model-len 131072 \
  --gpu-memory-utilization 0.90 \
  --kv-cache-dtype fp8 \
  --enable-chunked-prefill \
  --enable-prefix-caching \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder \
  --reasoning-parser qwen3
# Stretch 200k–256k only after measuring free KV in logs; start 131k then raise.
# No-NVLink boxes often need: NCCL_P2P_DISABLE=1
```

### 3D. EXPERIMENTAL only — EXL3 + DFlash2/MTP (batch-1)

```bash
# MiaAI kit; on 3090 MUST use CACHE_QUANT=4 (Hadamard), NOT nvfp4/fp8
GPU_MEM_GB=22 CONTEXT_SIZE=262144 CACHE_QUANT=4 DRAFT=mtp ./start.sh
# Concurrent Hermes will QUEUE — not continuous batch
```

---

## 4. HF / GitHub IDs

| Role | ID |
|---|---|
| Upstream model | `Qwen/Qwen3.8-27B` |
| AWQ INT4 (SGLang primary) | `cyankiwi/Qwen3.8-27B-AWQ-INT4` |
| DSpark draft | `RadixArk/Qwen3.8-27B-DSpark` |
| AutoRound W4A16 (alt) | `MIRALABS/Qwen3.8-27B-W4A16-AutoRound` |
| FP8 weights (vLLM/SGLang, Ampere = Marlin path) | `Qwen/Qwen3.8-27B-FP8` |
| AtomicChat GGUF ladder | `AtomicChat/Qwen3.8-27B-GGUF` |
| Unsloth GGUF + MTP tensors | `unsloth/Qwen3.8-27B-GGUF` |
| EXL3 3.5bpw target | `Mia-AiLab/Qwen3.8-27B-EXL3-3.5bpw` |
| DFlash2 EXL3 draft | `Mia-AiLab/Qwen3.8-27B-DFlash2-EXL3-5.0bpw` |
| SGLang dual-3090 recipe | `https://github.com/0xSero/qwen38-3090-sglang` |
| llama.cpp MTP cookbook | `https://github.com/sudoingX/qwen38-mtp` |
| vLLM single-3090 long-ctx kit | `https://github.com/syv-ai/qwen38-27b-rtx3090` |
| vLLM 4×3090 FP8 notes | `https://github.com/TIANWENtianw/qwen3.8-27b-vllm-deployment` |
| EXL3 kit | `https://github.com/MiaAI-Lab/...` (DFlash2-EXL3 README / exllamav3 fork) |
| SGLang cookbook | `https://docs.sglang.io/cookbook/autoregressive/Qwen/Qwen3.8-27B` |
| Dual-3090 WSL2 pitfalls (245760) | `https://dev.to/digitalmarket-world/i-got-qwen38-27b-running-on-dual-rtx-3090s-no-nvlink-under-wsl2-every-pitfall-i-hit-3oao` |

**AtomicChat sizes (from HF tree dump):**

| File | Bytes | ~GiB |
|---|---:|---:|
| AD-Q4_K_M | 17,120,781,792 | **17.1** |
| AD-Q5_K_M | 20,232,512,992 | **20.2** |
| AD-Q6_K | 25,005,438,432 | **25.0** |
| Q8_0 | 28,887,832,032 | **28.9** |

---

## 5. Multi-Hermes concurrency policy

**Goal match:** continuous batching so many profiles hit **one** base URL without single-slot freeze.

| Policy | Value |
|---|---|
| Endpoint | `http://ai-box:8088/v1` shared by all Hermes profiles |
| Engine | SGLang (primary) or vLLM (alt) — **not** EXL3 for multi-tenant |
| `max-running-requests` | start **4**, raise to **8–16** if GDN state + KV allow |
| Per-profile max_model_len client hint | default **32k–64k**; allow one “long” profile at 200k+ |
| Admission | reject/queue when KV utilization > ~85% (SGLang metrics) |
| Client timeout | **600s** normal; **1800s** for 100k+ prefill |
| Streaming | on; tools: prefer non-thinking or tested thinking+tools path |
| Sticky sessions | not required if prefix/radix cache on; still helps TTFT |
| llama.cpp path | `-np 4..8` without relying on MTP for multi; expect serialization pain vs SGLang |
| Fairness | LPM / FCFS schedule; avoid one 256k prefill starving agents — use chunked prefill |

**5–20 agent clients:** assume **not** all active decode simultaneously. Design for **4–8 in-flight** generations and the rest waiting on tools/CPU. Continuous batch shines when several short/medium prompts overlap.

---

## 6. Risks and unknowns (Ampere vs Blackwell cookbook mismatch)

1. **Blackwell/DGX Spark cookbooks ≠ 3090:** NVFP4 weight matmul, native FP8 tensor cores, sm_120 wheels — **do not** copy blindly.
2. **0xSero vs community 256k:** official 0xSero TP2 validates **64k / ~119k BF16 pool**; 256k needs **fp8 KV + lower mem-fraction experiments** — verify on **this** box.
3. **FP8 KV quality:** generally good; deep-needle / agent tool JSON at 200k+ needs **your** NIAH and tool-call soak (club-3090 notes warn 262144 allocated ≠ filled quality).
4. **GDN hybrid state:** concurrency limited by **mamba/GDN state**, not only KV tokens (0xSero TP4: 275k KV but max concurrent 9).
5. **Thinking + tools:** SGLang issue #36537 (Flash-Next + qwen3_coder loops); vLLM historical non-standard tool XML when thinking on; Hermes docs: empty `content` if reasoning-parser mishandled.
6. **32 GB host RAM:** loading 17–29 GB weights + Docker + two GPU bar mappings can **thrash**; headless, no compositor on GPUs (sudoingX rule 7).
7. **PCIe no NVLink:** TP all-reduce over PCIe; optional aikitoria P2P kernel + ReBAR; else `NCCL_P2P_DISABLE=1` / `--disable-custom-all-reduce`.
8. **PL 275 W:** expect lower tok/s than 350 W benches; thermal throttle under 8-wide batch.
9. **CUDA 13 / driver 610:** 0xSero pins torch cu130 + specific sglang commit — version skew breaks GDN kernels (triton `_grid_2` etc.).
10. **Speculative decode under load:** DSpark/MTP help C1; multi-tenant often better **without** draft model (more KV).

---

## 7. Verification checklist

1. `nvidia-smi -L` → two 3090s; set `POWER_LIMIT=275` both.
2. Boot server; `nvidia-smi` → **both** GPUs non-zero VRAM; no heavy host RAM weight residency (`ps` RSS sane).
3. `curl -s localhost:8088/v1/models | jq` → model id visible.
4. Short chat `/v1/chat/completions` with `/no_think` or `enable_thinking:false`.
5. Parallel soak: 5–10 concurrent 2k-prompt completions; watch continuous batch (no hard single-slot freeze).
6. When free: one **200–256k** prefill (synthetic) — confirm no OOM, measure TTFT.
7. Tool-call smoke: OpenAI `tools` array → `tool_calls` returned; Hermes profile one tool round-trip.
8. Confirm **no CPU weight offload** (all layers GPU; llama.cpp `-ngl 999` and no partial offload logs).
9. Metrics: KV usage, running requests, tok/s decode aggregate.
10. Fail test: kill one client mid-stream — others continue.

---

## 8. Phased deploy plan

| Phase | Action | Success |
|---|---|---|
| **P0** | Inventory drivers, CUDA, P2P, disk for models, headless GPU | `nvidia-smi`, topology |
| **P1** | Deploy **0xSero TP2 container** on :8088 (64k class) | benches ~100+ t/s decode C1 |
| **P2** | Point **one** Hermes profile; tool-call + thinking flags soak | stable agent loop |
| **P3** | Switch/add **fp8 KV + context-length 200k–245760** bare or patched entrypoint | one 200k completion |
| **P4** | Multi-profile 5–10 concurrent policy + timeouts | no meltdown; queue OK |
| **P5** | Optional: llama.cpp 262k sidecar on :8089 for long-ctx jobs | 262k fits, MTP on |
| **P6** | systemd units, healthchecks, PL 275, log rotation | reboot-safe |
| **P7** | Reject EXL3 as primary; keep as research lane | — |

**systemd sketch:**

```ini
[Unit]
Description=Qwen3.8-27B SGLang dual-3090
After=network.target nvidia-persistenced.service

[Service]
Type=simple
Environment=CUDA_VISIBLE_DEVICES=0,1
Environment=NCCL_P2P_DISABLE=1
ExecStart=/usr/bin/docker run --rm --gpus all --shm-size 32g -p 8088:8000 -v /models/qwen38:/models -e TP=2 IMAGE
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Health: `curl -fsS localhost:8088/v1/models` in systemd timer or docker HEALTHCHECK.

---

## 9. Research answers A–G (condensed)

### A) SGLang/vLLM TP2 @ ~256k today?
- **SGLang:** Yes **with conditions**. BF16 KV TP2 validated **64k** (0xSero). **~245k** demonstrated on dual-3090 with **AWQ-INT4 + fp8_e4m3 KV + TP2** (DEV community). Weight format: **AWQ-INT4 / compressed-tensors**, not BF16. Flags: tp 2, mem-fraction ~0.8–0.85, chunked prefill, tool-call-parser `qwen3_coder`, reasoning-parser `qwen3`, disable prefill cuda graph on tight VRAM, mamba-radix `extra_buffer` when using hybrid defaults. **sm_86:** Marlin INT4 real; FP8 weight compute is dequant path; FP8 KV storage used successfully in community.
- **vLLM:** Long context proven on **1×3090** (syv-ai 150k–245k with patches). TP2 dual-3090 for headroom is reasonable; 4×3090 FP8@262k documented (TIANWENtianw). Start max-model-len 131k then climb.

### B) VRAM budget
See BOM table. **256k concurrent count ≈ 1**. Hermes 8–32k → many. Token pool shared.

### C) Quality path ≤42 GB @ 256k
| Rank | Format | Fit @256k dual | Agent/tool notes |
|---|---|---|---|
| Primary serve | AWQ-INT4 | yes w/ fp8 KV | best Ampere decode |
| llama.cpp primary | AD-Q4_K_M / AD-Q5_K_M | yes w/ q4 KV | strong KL; MTP free |
| Higher quality GGUF | AD-Q6_K / Q8_0 | 256k tight/no on dual without heavy KV quant | prefer shorter ctx |
| FP8 weights | Qwen FP8 | VRAM OK | slower math on Ampere |
| EXL3 3.5bpw | yes w/ Hadamard-4 KV | quality OK; **no multi-batch** |

### D) Multi-tenant
SGLang/vLLM continuous batch **>>** llama.cpp `-np`. Timeouts 10–30 min for long prefill. One endpoint, many profiles.

### E) Hermes pitfalls
- Empty content: reasoning-parser + thinking; set `enable_thinking: false` for tools or raise max_tokens.
- Parsers: `qwen3_coder` + `qwen3` on server.
- Streaming+tools: test; some bugs with thinking+tool loops (SGLang #36537 family).
- `/no_think` in user content works on many Qwen templates.

### F) Day-2 ops
- Prefer **CUDA 12.8–13** stack matching wheel; 0xSero pins cu130.
- NCCL over PCIe; optional P2P kernel; else disable P2P.
- PL 275 both cards; persistenced on.
- 32 GB RAM: don’t parallel-download giant quants; monitor `free -h` at load.
- systemd + `/v1/models` health; logs to journald.

### G) Tradeoff table + WINNER

| Option | 256k | Multi-Hermes batch | Tooling | Ampere maturity | Verdict |
|---|---|---|---|---|---|
| (1) llama.cpp 262k q4KV+MTP | **Best native 262k** | Weak (parallel slots) | Good w/ jinja | Excellent | **Fallback / long-ctx sidecar** |
| (2) SGLang TP2 cont. batch | **Yes w/ fp8 KV (~245k shown)** | **Best** | qwen3_coder | Excellent (0xSero) | **WINNER** |
| (3) EXL3 256k exp | Yes (Hadamard-4) | **Fails (batch-1)** | Works | Experimental bugs | Research only |
| (4) vLLM TP2 | Likely 128k–256k w/ KV quant | Strong | qwen3_coder | Strong (syv patches) | **Strong alternate** |

**WINNER: (2) SGLang TP2 continuous batch (AWQ-INT4 + fp8 KV for 256k class; 0xSero container for Phase-1).**

---

## 10. Citations (major claims)

- 0xSero dual-3090 SGLang benches, AWQ>FP8 on Ampere, TP2 64k / 118k KV, DSpark, flags: https://github.com/0xSero/qwen38-3090-sglang
- sudoingX MTP + q4 KV 262k single 24GB, multi-GPU tensor split, parallel kills MTP gain: https://github.com/sudoingX/qwen38-mtp
- Dual-3090 WSL2 SGLang **context-length 245760** + fp8 KV + TP2 + parsers: https://dev.to/digitalmarket-world/i-got-qwen38-27b-running-on-dual-rtx-3090s-no-nvlink-under-wsl2-every-pitfall-i-hit-3oao
- syv-ai vLLM single-3090 150k–245k continuous batch / DFlash2: https://github.com/syv-ai/qwen38-27b-rtx3090
- TIANWENtianw 4×3090 vLLM FP8 max 262144, NCCL_P2P_DISABLE: https://github.com/TIANWENtianw/qwen3.8-27b-vllm-deployment
- MiaAI EXL3 batch-1, Ampere CACHE_QUANT=4, tool calling, always-on reasoning: MiaAI-Lab DFlash2-EXL3 README
- AtomicChat GGUF sizes: Hugging Face `AtomicChat/Qwen3.8-27B-GGUF` tree (AD-Q4_K_M 17.12 GB … Q8_0 28.89 GB)
- SGLang cookbook Qwen3.8-27B: https://docs.sglang.io/cookbook/autoregressive/Qwen/Qwen3.8-27B
- Hermes providers empty content / enable_thinking: https://hermes-agent.nousresearch.com/docs/integrations/providers
- SGLang thinking+tool parser issue: https://github.com/sgl-project/sglang/issues/36537
- MIRALABS AutoRound dual-3090 262144 note: https://huggingface.co/MIRALABS/Qwen3.8-27B-W4A16-AutoRound
- vLLM recipe FP8 TP4 262144: https://recipes.vllm.ai/Qwen/Qwen3.8-27B

---

## Structured claims index

| claim_id | statement | status | confidence |
|---|---|---|---|
| C1 | 0xSero TP2 SGLang AWQ on 2×3090 ≈113 t/s decode, ~64k max ctx, ~119k BF16 KV pool | supported | high |
| C2 | Ampere should prefer AWQ-INT4 Marlin over FP8 weight compute for speed | supported | high |
| C3 | llama.cpp q4_0 KV enables ~262k with Q4 weights on 24GB-class | supported | high |
| C4 | MTP speculative gain collapses as parallel streams increase | supported | high |
| C5 | Dual-3090 SGLang can set context-length ~245760 with fp8 KV + AWQ TP2 | supported (community) | medium |
| C6 | EXL3 path is effectively batch-1 / queue for concurrent requests | supported | high |
| C7 | Hermes needs qwen3_coder parser + careful enable_thinking | supported | high |
| C8 | True multi-Hermes continuous batch winner is SGLang (or vLLM), not llama.cpp/EXL3 | supported | high |
| C9 | Operator “1.2M capacity” is not the measured 2×3090 BF16 pool | supported (correction) | high |
| C10 | Full 256k **with** high concurrency **and** BF16 KV on 2×24GB is not realistic | supported | high |

## Gaps

- No on-box measurement on operator’s Omarchy dual-3090 in this session (research-only).
- Exact fp8 KV KB/token for Qwen3.8 GDN on SGLang sm_86 not re-derived from first principles here — use startup log “token capacity”.
- geldeki/AutoRound dual-3090 262k claim not fully byte-verified beyond HF card blurb.
- Firecrawl blocked; some HTML docs only partially grepped.
- Host RAM stated 32 GB vs possible higher — confirm before large parallel loads.

## Confidence

- **level:** medium-high for stack choice; medium for exact 256k multi-request headroom on this chassis until P3 soak.
- **rationale:** Multiple independent prior-art repos with concrete flags/benches; 256k multi-tenant remains capacity-planning sensitive.

## Recommended next actions

1. P1: boot 0xSero TP2 on ai-box :8088; record nvidia-smi + `/v1/models`.
2. P3: fp8 KV long-context launch; log KV token capacity from SGLang.
3. Wire one Hermes profile; tool-call smoke with thinking off.
4. Optionally stand llama.cpp 262k sidecar for heavy long-ctx jobs.
5. File Huly GIU issue + doc (this report).
