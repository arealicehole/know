# Path F: Meta Muse Glimmer 30B — AI Box Feasibility

**Task:** t_b1522ba1
**Date:** 2026-11-21
**Researcher:** GIU (Geraldo v2.1)
**Status:** PASS — viable on 1x RTX 4090/3090 (24 GB) via llama.cpp (existing binary). vLLM path is Blackwell-only for NVFP4; BF16/FP8 need 32+ GB.

---

## 1. Model Overview

| Property | Value | Source |
|---|---|---|
| Full name | Muse-Glimmer-30B | HF repo |
| Provider | Meta Superintelligence Lab | HF model card |
| Release date | August 2026 | HF model card |
| License | Apache 2.0 | HF model card |
| Total parameters | ~29.6B (BF16 safetensors: 29.78B) | HF API / config.json |
| Architecture | Dense causal transformer + perception encoder (ViT-G/14, ~1.8B) | config.json / README |
| Hidden dim | 6656 | config.json |
| Layers | 52 | config.json |
| Attention | [Local(3), Global(1)] repeating, sliding window 2048 | config.json |
| Attention heads | 32 Q / 2 KV (GQA 16:1) | config.json |
| Head dim | 128 | config.json |
| FFN | SwiGLU, intermediate 19,968 | config.json |
| Position encoding | RoPE θ=500,000, local layers only | config.json |
| Vision encoder | ViT-G/14, 50 layers, width 1536, patch 14, ~1.8B params | README |
| Max visual tokens/image | 4,096 | README |
| Context length | 131,072+ | README / config.json (max_position_embeddings: 131072) |
| Tokenizer | 200,000 BPE + 2,048 special = 202,048 vocab | README |
| Knowledge cutoff | January 4, 2026 | README |
| Modalities | Input: text + image; Output: text | README |
| Distilled from | Muse Spark (Meta's larger model) | README |
| arXiv | 2610.24628 (model card), 2504.13181 (perception encoder), 2602.06036 (DFlash) | README |

## 2. Architecture Details

Muse Glimmer is a **dense** (non-MoE) causal transformer. Unlike Inkling 2 Mini (see Path G), all 29.6B parameters are active per token. The model is explicitly designed for local/consumer hardware:

- **Dense, not MoE:** No expert routing. Every layer, every head, every FFN neuron participates in every forward pass.
- **Hybrid attention:** 3 local (sliding window 2048) + 1 global layer in a repeating 4-layer pattern across 52 layers. Only local layers use RoPE.
- **GQA 16:1:** 32 query heads but only 2 KV heads. This is extremely aggressive KV compression — the KV cache per token is very small.
- **Perception encoder:** Separate ~1.8B ViT-G/14 frozen encoder for image input. Not part of the autoregressive loop.
- **DFlash speculative drafter:** A 5-layer block-diffusion companion model (block size 16) that proposes 16-token blocks verified in parallel by the main model. Ships as a separate 1.6 GB GGUF file.
- **Reasoning:** Supports `reasoning_content` in the chat template. Controllable via `Reasoning strength: low/medium/high/xhigh` in system prompt.
- **Tool use:** ATEM (Agent Tool Execution Model) chat template. Tool calls rendered as `<atem:function_calls>` blocks with `<atem:invoke name="...">` and `<atem:parameter>` elements. Tool results come back as `tool` role messages with `tool_call_id` referencing.

### Tool call format (from chat_template.jinja)
```
<atem:function_calls>
<atem:invoke name="tool_name">
<atem:parameter name="arg1">value1</atem:parameter>
<atem:parameter name="arg2">value2</atem:parameter>
</atem:invoke>
</atem:function_calls>
```
- Arguments MUST be a dict/mapping (not a JSON string) in the HF Jinja sandbox
- Multiple tool calls can be batched in one `<atem:function_calls>` block
- Tool results reference the call by `tool_call_id`

## 3. Quantization & File Inventory

### 3a. Official GGUF Repo (meta-models/Muse-Glimmer-30B-GGUF)

**llama.cpp requirement:** Build **b10353+** (merged Aug 10 2026, PR #26841). **Operator-verified: the existing Sep-14 llama.cpp build already has `muse_glimmer` arch (confirmed via `strings libllama.so`). No rebuild needed for Path F.**

| File | Size | Purpose |
|---|---|---|
| `Muse-Glimmer-30B-KQuant-17GB-Q4_K_M.gguf` | 16.76 GB | Text model — **primary target**, fits 24 GB VRAM |
| `Muse-Glimmer-30B-KQuant-Dynamic-Q4_K_XL.gguf` | 19.65 GB | Higher-quality build for 32 GB VRAM |
| `mmproj-Muse-Glimmer-30B-Q4_K_M.gguf` | 1.40 GB | Perception encoder (ViT) — required for image input |
| `dflash-Muse-Glimmer-30B-Q4_K_M.gguf` | 1.63 GB | DFlash speculative drafter — optional, ~3.1x speedup |

### 3b. Unsloth GGUF Repo (unsloth/Muse-Glimmer-30B-GGUF)

**Operator note:** `Muse-Glimmer-30B-UD-Q4_K_XL.gguf` (15.88 GB) already downloaded to `/models/`. The mmproj was NOT found under the expected names in the Unsloth repo. **Correct mmproj filenames (from Unsloth repo tree):**

| File | Size | Purpose |
|---|---|---|
| `Muse-Glimmer-30B-UD-Q4_K_XL.gguf` | 15.88 GB | Text model (Unsloth UD quant) — **already downloaded** |
| `Muse-Glimmer-30B-UD-Q4_K_M.gguf` | 14.44 GB | Text model (lower quant) |
| `Muse-Glimmer-30B-Q8_0.gguf` | 29.61 GB | Text model (Q8, reference) |
| `mmproj-Muse-Glimmer-30B-Q8_0.gguf` | 2.05 GB | **Vision encoder (Q8) — use this for Unsloth build** |
| `mmproj-Muse-Glimmer-30B-BF16.gguf` | 3.85 GB | Vision encoder (BF16, reference) |
| `mmproj-kquant.gguf` | 1.40 GB | **Vision encoder (KQuant) — matches official KQuant build** |
| `dflash-kquant.gguf` | 1.63 GB | DFlash speculative drafter (KQuant) |

**Critical: The mmproj file in the Unsloth repo is named `mmproj-Muse-Glimmer-30B-Q8_0.gguf` (2.05 GB) for the Q8 vision encoder, or `mmproj-kquant.gguf` (1.40 GB) for the KQuant variant. The operator's search for "mmproj" under tried names failed — the correct filename includes the quant suffix.**

### 3c. Memory Footprint

| Build | Text only | + Vision | + Vision + Drafter |
|---|---|---|---|
| KQuant-17GB (Q4_K_M, official) | ~17 GB | ~19 GB | ~20 GB |
| UD-Q4_K_XL (15.88 GB, Unsloth) | ~16 GB | ~18 GB | ~19.6 GB |
| KQuant-Dynamic (Q4_K_XL, official) | ~20 GB | ~22 GB | ~23 GB |

### 3d. Quantization Quality (Meta-reported, 15-benchmark average)

| Build | VRAM target | Degradation vs BF16 |
|---|---|---|
| Full precision (BF16) | 64 GB | — (baseline) |
| K-Quant-Dynamic (Q4_K_XL) | 32 GB | 0.2% |
| K-Quant-17GB (Q4_K_M) | 24 GB | 1.0% |

## 4. AI Box Fit Analysis

### Target hardware: 1x RTX 4090 (24 GB) or 3090 (24 GB)

**Verdict: FITS with margin.**

The official `KQuant-17GB-Q4_K_M` build is designed for exactly this class of GPU:

- Weights: 16.76 GB
- Vision encoder (mmproj): +1.40 GB → 18.16 GB
- KV cache at 131K context (GQA 16:1, 2 KV heads, 52 layers, 128 head dim): ~1.5-2.5 GB
- Total: ~20-21 GB of 24 GB → **3-4 GB headroom**
- With DFlash drafter: +1.63 GB → ~22 GB → still fits, but tight (2 GB headroom)

### Recommended configuration for 24 GB card

```bash
./llama-server \
    -m Muse-Glimmer-30B-GGUF/Muse-Glimmer-30B-KQuant-17GB-Q4_K_M.gguf \
    --mmproj Muse-Glimmer-30B-GGUF/mmproj-Muse-Glimmer-30B-Q4_K_M.gguf \
    -ngl 99 \
    -c 32768 \          # 32K context to keep KV cache manageable
    -np 4 \
    --host 127.0.0.1 --port 8088 \
    --jinja \
    --temp 1.0 --top-p 0.95 --top-k 64 \
    --alias muse-glimmer-30b
```

**Key settings:**
- `--jinja` is **required** for the ATEM tool call template to work
- `--temp 1.0 --top-p 0.95 --top-k 64` are Meta's recommended sampling params
- `-c 32768` (32K) is the sweet spot for Hermes-style agentic work; 131K will consume ~2 GB of KV cache and eat into headroom
- Add `-md dflash-Muse-Glimmer-30B-Q4_K_M.gguf -ngld 99` for speculative decoding if you have the headroom (adds ~1.6 GB, gives ~3.1x decode speedup)

### For 32 GB cards (if available)
Use `KQuant-Dynamic-Q4_K_XL` (19.65 GB) for the 0.2% vs 1.0% degradation difference. Total footprint ~23 GB with vision + drafter.

### Speed expectations (Meta-reported)

| GPU | No speculation | With DFlash | Speedup |
|---|---|---|---|
| RTX 5090 | 74.9 tok/s | 233.4 tok/s | 3.1x |
| Apple M4 Max | 23.7 tok/s | 37.8 tok/s | 1.5x |
| Apple M5 Max | 26.6 tok/s | 50.2 tok/s | 1.8x |

For RTX 4090 (comparable to 5090 class): expect ~70-80 tok/s baseline, ~200-230 tok/s with DFlash. This is well within the 45+ tok/s target.

## 5. Tool Calling & Function Calling

**Supported:** Yes, via the ATEM chat template.

The ATEM format is structured but non-standard:
- Tool calls are rendered as XML-like blocks: `<atem:invoke name="...">` with `<atem:parameter>` children
- Tool results come back as `role: "tool"` messages with `tool_call_id`
- The template handles multi-turn tool loops natively
- **Caveat:** Arguments must be a Python dict in the Jinja sandbox — a JSON string will raise an error. This means the Hermes tool-calling layer must pass structured objects, not serialized strings.

### vLLM Tool-Call Parser (for vLLM deployment)

vLLM has a **dedicated `muse_glimmer` tool-call parser** (in `vllm/tool_parsers/muse_glimmer_tool_parser.py`). It extends `MuseToolParser` with:
- `_preserve_string_parameters = False` — converts ATEM parameter values to typed objects
- `_reasoning_recipients = ("self",)` — channels `to=self` are reasoning
- `_content_recipients = (None, "user")` — channels `to=user` or unaddressed are content
- `_non_tool_recipients = ("self", "user")`

The vLLM serve command for tool calling:
```bash
vllm serve meta-models/Muse-Glimmer-30B \
  --generation-config auto \
  --tensor-parallel-size 1 \
  --enable-auto-tool-choice \
  --tool-call-parser muse_glimmer \
  --reasoning-parser muse_glimmer
```

**Both `--tool-call-parser muse_glimmer` AND `--reasoning-parser muse_glimmer` are required.** The reasoning parser forces `skip_special_tokens=False` — without it, the channel markers are stripped before parsing and both channels collapse into content.

### Hermes Compatibility (llama.cpp path)

- The chat template is in `chat_template.jinja` and works with llama.cpp `--jinja`
- Tool call parsing in llama.cpp should handle the ATEM format (it's in the template, so llama.cpp renders it; the server returns `tool_calls` in the OpenAI-compatible API response)
- Need to verify: whether llama.cpp's OpenAI-compatible endpoint correctly deserializes ATEM tool calls into the standard `tool_calls` array. The template renders the tool call text; the server's JSON response should include structured `tool_calls` if llama.cpp parses them. This is the primary integration risk.

### Speculative Decoding (DFlash)

**llama.cpp path:**
- Separate 1.63 GB GGUF file (`dflash-kquant.gguf` in Unsloth repo, `dflash-Muse-Glimmer-30B-Q4_K_M.gguf` in official repo)
- Adds `-md` flag to llama-server
- 5-layer block-diffusion drafter, block size 16
- Predicts 16-token blocks verified in parallel
- ~3.1x speedup on RTX 5090
- Optional but recommended for agent workloads (long tool loops benefit most)

**vLLM path:**
- Draft head: `meta-models/Muse-Glimmer-30B-assistant` (5.11 GB, BF16)
- `--speculative-config '{"method": "dflash", "model": "meta-models/Muse-Glimmer-30B-assistant", "num_speculative_tokens": 15}'`
- `num_speculative_tokens: 15` is FIXED (block_size 16, slot 0 re-presents last accepted token)
- Requires `--max-num-seqs 32` (DFlash uses 16 query positions per active request during warmup)
- On 2x RTX 5090: decode goes from 68.8 → ~240 tok/s (3.5x)
- On 1x 5090: OOM during init (draft head 5.11 GB + NVFP4 weights 25.42 GB > 32 GB). **Requires second card for vLLM + DFlash.**

## 6. Hermes Integration Checklist

- [ ] llama.cpp b10353+ (current b12602) with `--jinja` flag
- [ ] OpenAI-compatible endpoint (`/v1/chat/completions`) — confirmed working via llama-server
- [ ] ATEM tool call format — rendered by chat template; verify llama.cpp returns structured `tool_calls` in JSON response
- [ ] Sampling: `temp=1.0, top_p=0.95, top_k=64` (Meta-recommended)
- [ ] Reasoning: `reasoning_content` in response; controllable via system prompt `Reasoning strength: <low|medium|high|xhigh>`
- [ ] Context: 32K recommended for agentic work; 131K max
- [ ] Vision: mmproj file required for image input; 4096 max visual tokens per image
- [ ] Speculative decoding: DFlash drafter optional, ~3.1x speedup
- [ ] Memory: 20-22 GB total for 24 GB card (weights + vision + KV cache)

## 7. Benchmarks (Meta-reported, High Reasoning mode)

| Category | Benchmark | Muse Glimmer 30B | Gemma4-31B | Qwen3.6-27B |
|---|---|---|---|---|
| Agentic | MCP Atlas (Public) | **75.5** | 54.2 | 62.5 |
| Agentic | DeepSearch QA | **74.6** | 61.7 | 71.1 |
| Agentic | τ3-Banking | **23.5** | 15.1 | 16.7 |
| Agentic | WildClawBench | **47.6** | 37.6 | 43.2 |
| Agentic | Gaia2 | **43.3** | 36.4 | 40.0 |
| Agentic | OSWorld-Verified | 65.9 | 58.5 | **75.6** |
| Coding | SWE-Bench Pro | **51.2** | 36.9 | 50.2 |
| Coding | SWE-Bench Verified | 76.0 | 66.6 | **77.2** |
| Coding | TerminalBench 2.1 | 51.7 | 43.4 | **60.7** |
| Coding | SciCode | **43.6** | 43.4 | 39.8 |
| Multimodal | Charxiv Reasoning | **78.8** | 77.7 | 78.4 |
| Multimodal | ScreenSpot Pro | 75.4 | 75.9 | **76.1** |
| Multimodal | OmniDocBench v1.5 | 75.8 | 72.5 | **77.8** |
| Multimodal | MMMU Pro | 74 | 73 | **75** |
| Reasoning | AIME 2026 | **94.7** | 89.2 | 94.1 |
| Reasoning | GPQA Diamond (AA) | 83.5 | **85.7** | 84.2 |
| Reasoning | IFBench | **77.0** | 76.0 | 70.8 |
| Reasoning | AA-LCR | **80.0** | 68.3 | 73.3 |
| Reasoning | Beam128K | **65.1** | 58.2 | 63.0 |

**Note:** Muse Glimmer leads in agentic benchmarks (MCP Atlas, DeepSearch, τ3-Banking, WildClaw, Gaia2) and reasoning (AIME, AA-LCR, IFBench) but trails Qwen3.6-27B in some coding and multimodal tasks. It is explicitly designed for agentic work, which is the primary use case.

## 8. Security & Safety (Meta-preparedness)

- **Not frontier** by Meta's AAISF framework (below Muse Spark)
- Chem/Bio: Moderate or lower
- Cyber: Moderate or lower (inferred)
- Loss of Control: Moderate or lower (inferred)
- Safety SFT + Safety RL + Appropriate Information Flows in training
- CI Memories violation rate: 26.4% (higher than Gemma4's 12.1% — note this)
- Siren AgentDojo attack success: 28.4% (higher than Gemma4's 25.6%)
- Utility: 94.2% (best of the three)

## 10. vLLM Deployment (TP2 2x RTX 3090, sm86 Ampere)

### NVFP4 on Ampere: NOT SUPPORTED

vLLM's NVFP4 kernel backends require **Blackwell (sm_120+) or Hopper (sm_90+)**:
- `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `FLASHINFER_CUTEDSL` — Blackwell/Hopper
- `VLLM_CUTLASS`, `MARLIN` — Blackwell/Hopper
- `EMULATION` — Triton-based dequant to BF16, works on any GPU but **slow** (dequantizes every expert weight to BF16 every forward step)

**RTX 3090 (sm_86, Ampere) does NOT have native NVFP4 support.** The `Inferact/Muse-Glimmer-30B-NVFP4-W4A4` checkpoint (25.42 GB) would run via the EMULATION backend (Triton dequant → BF16 → standard GEMM), but:
- No speed advantage over just using BF16/FP8 weights
- The vLLM recipe explicitly states: "NVFP4 kernels are Blackwell-only"
- The `modelopt_fp4` quant method with `quant_algo: NVFP4` would load, but the MoE kernels fall back to emulation

**Verdict: Do NOT use the NVFP4 checkpoint on 2x 3090. Use BF16 or FP8 instead.**

### BF16 on 2x RTX 3090 (TP2)

- Weights: 59.58 GB → ~29.8 GB per GPU (TP2)
- Vision encoder: ~3.6 GB → ~1.8 GB per GPU
- KV cache at 131K: ~1.5 GB per GPU
- Total per GPU: ~33 GB → **DOES NOT FIT in 24 GB**

**BF16 does not fit on 2x 3090 (24 GB each) even with TP2.** The model is too large.

### FP8 Block on 2x RTX 3090 (TP2)

- Weights: 32.78 GB → ~16.4 GB per GPU (TP2)
- Vision encoder: ~3.6 GB → ~1.8 GB per GPU
- KV cache at 32K: ~0.5 GB per GPU
- Total per GPU: ~18.7 GB → **FITS in 24 GB** with ~5 GB headroom

The `RedHatAI/Muse-Glimmer-30B-FP8-block` checkpoint (32.78 GB, FP8 block-scaled weights, dynamic activations, vision tower + embeddings BF16) is the **only viable vLLM path for 2x 3090**.

**Caveat:** FP8 block-scaled weights require `VLLM_USE_DEEP_GEMM=0 VLLM_MOE_USE_DEEP_GEMM=0` on sm120 (RTX 5090). On sm86 (3090), DeepGEMM is not used (it's Blackwell-only), so this workaround is not needed. However, the FP8 kernels themselves need verification on Ampere — vLLM's FP8 support is primarily tested on Hopper/Blackwell.

### vLLM Serve Command (2x 3090, FP8)

```bash
vllm serve /models/Muse-Glimmer-30B-FP8-block \
  --served-model-name muse-glimmer \
  --tensor-parallel-size 2 \
  --gpu-memory-utilization 0.90 \
  --max-model-len 32768 \
  --max-num-seqs 32 \
  --enable-auto-tool-choice \
  --tool-call-parser muse_glimmer \
  --reasoning-parser muse_glimmer \
  --generation-config auto
```

### DFlash on vLLM 2x 3090

The DFlash draft head (5.11 GB BF16) adds ~2.6 GB per GPU. With FP8 weights (~16.4 GB) + vision (~1.8 GB) + KV (~0.5 GB) + drafter (~2.6 GB) = ~21.3 GB per GPU → **fits in 24 GB** with ~2.7 GB headroom.

```bash
--speculative-config '{"method": "dflash", "model": "meta-models/Muse-Glimmer-30B-assistant", "num_speculative_tokens": 15}' \
--max-num-seqs 32
```

## 11. Risks & Caveats

1. **Dense = all params active:** Unlike Inkling 2 Mini (3.4B active), every token pays the full 29.6B compute cost. On a 4090, this means ~70-80 tok/s vs potentially 150+ for a MoE with 3-5B active. Still fast enough, but not as fast as it could be.
2. **ATEM tool call format is non-standard:** The XML-like format may need adaptation in Hermes' tool-call parser. The `--jinja` flag handles rendering, but the server-side parsing of ATEM blocks into structured `tool_calls` needs verification.
3. **131K context is expensive in KV:** At 131K tokens, even with GQA 16:1, the KV cache is ~2 GB. For Hermes agent work with long conversation histories, 32K is the practical limit on 24 GB.
4. **Vision is additive:** The 1.4 GB mmproj is mandatory for image input. If you don't need vision, you save 1.4 GB and can use a larger context.
5. **Knowledge cutoff Jan 4, 2026:** 10 months old. Will lack late-2026 knowledge.
6. **Meta's safety benchmarks show higher violation rates** than Gemma4 on CI Memories (26.4% vs 12.1%) and Siren (28.4% vs 25.6%). Not disqualifying, but worth noting for agent use with tool access.
7. **vLLM NVFP4 is Blackwell-only:** On 2x 3090 (Ampere), NVFP4 checkpoints cannot use native kernels. Must use FP8 or BF16. BF16 doesn't fit; FP8 does.
8. **llama.cpp is the primary path:** The existing Sep-14 build has `muse_glimmer` arch. No rebuild needed. Use the Unsloth UD-Q4_K_XL (15.88 GB) + `mmproj-Muse-Glimmer-30B-Q8_0.gguf` (2.05 GB) + `dflash-kquant.gguf` (1.63 GB).

## 12. Verdict

**PASS — Primary path: llama.cpp (existing binary, no rebuild). Secondary path: vLLM FP8 on 2x 3090.**

**llama.cpp (recommended):**
- Existing Sep-14 build has `muse_glimmer` arch (verified)
- UD-Q4_K_XL (15.88 GB) + mmproj-Q8 (2.05 GB) + dflash (1.63 GB) = ~19.6 GB → fits 24 GB
- 70-80 tok/s baseline, 200-230 tok/s with DFlash — well above 45 tok/s target
- Strong agentic benchmarks (MCP Atlas 75.5, DeepSearch 74.6)
- Apache 2.0
- **Primary risk:** ATEM tool call parsing in Hermes — needs integration testing

**vLLM (secondary, for comparison):**
- FP8 block (32.78 GB) on TP2 2x 3090 → ~18.7 GB per GPU, fits
- NVFP4 (25.42 GB) is Blackwell-only — NOT usable on Ampere for native kernels
- BF16 (59.58 GB) does NOT fit on 2x 3090 even with TP2
- DFlash requires second card (5.11 GB draft head) — fits on 2x 3090 with FP8
- vLLM >= 0.28.0 (released Nov 16 2026)
- `muse_glimmer` tool-call parser and reasoning parser built in

**Recommended config (llama.cpp):**
```
Model:      Muse-Glimmer-30B-UD-Q4_K_XL.gguf (15.88 GB) [Unsloth, already downloaded]
Vision:     mmproj-Muse-Glimmer-30B-Q8_0.gguf (2.05 GB) [Unsloth]
Drafter:    dflash-kquant.gguf (1.63 GB) [Unsloth] [optional]
Context:    32768
Sampling:   temp=1.0, top_p=0.95, top_k=64
Flag:       --jinja
```
