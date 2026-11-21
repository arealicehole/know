# Path G: Thinking Machines Inkling 2 Mini — AI Box Feasibility

**Task:** t_b1522ba1
**Date:** 2026-11-21
**Researcher:** GIU (Geraldo v2.1)
**Status:** PASS — viable on 1x RTX 4090/3090 (24 GB), strong agentic benchmarks

---

## 1. Model Overview

| Property | Value | Source |
|---|---|---|
| Full name | Inkling-2-Mini | HF repo |
| Provider | Thinking Machines Lab, Inc. | HF model card |
| Release date | **November 20, 2026** (1 day before this report) | HF model card |
| License | Apache 2.0 | HF model card |
| Total parameters | 26.4B (26.39B per HF API) | config.json / HF API |
| Active parameters | 3.4B per pass; **5.4B effective** (3 passes × 1.0B added per pass) | README |
| Architecture | Looped MoE transformer (Inkling2LoopForConditionalGeneration) | config.json |
| Hidden dim | 2304 | config.json |
| Layers | 24 (12 middle layers loop ×3 = 48 layer applications per token) | config.json / README |
| Loop config | core_start=6, core_end=18, num_passes=3, gate_rank=64, is_varloop=false | config.json |
| MoE | 128 routed experts per layer, top-8 routing, 0 shared experts, intermediate 1152 | config.json |
| Attention | Hybrid local/global: 20 local + 4 global layers, sliding window 1024 | config.json |
| Attention heads | 24 Q / 8 KV (GQA 3:1), head dim 128 | config.json |
| Position encoding | Log-scaling (log_scaling_n_floor=8192, alpha=0.1), QK L2 clip 22.63, rel L2 clip 8.0 | config.json |
| Vision encoder | 23-layer ViT, hidden 1024, 16 heads, patch 20, temporal 2, adapter merge 2, projection 4096 | config.json |
| MTP (speculative) | 3-layer MTP head, block_size 8, Markov rank 256, target layers [5,23,41,48] | config.json |
| Context length | **1,048,576 (1M tokens)** | config.json (model_max_length) |
| Tokenizer | 200,058 unpadded + special = 201,024 vocab (tiktoken-based) | config.json |
| Quantization (native) | Mixed NVFP4 + MXFP8 (modelopt) | config.json / HF API |
| Modalities | Input: text + image; Output: text | README |
| DSpark speculator | Ships in same checkpoint (mtp.safetensors) | HF repo files |

## 2. Architecture Details

Inkling 2 Mini is a **looped Mixture-of-Experts** transformer — the first looped transformer Thinking Machines has released. This is architecturally distinct from both Muse Glimmer (dense) and Qwen 3.8 (dense/standard MoE):

- **Looped attention:** 24 layers of weights, but the middle 12 (indices 6-18) are executed **three times** with shared weights. Each token passes through 48 layer applications but only 24 unique weight sets. This is "looping" — the model reuses the same 12 layers as a computation loop.
- **Sparse MoE:** Each layer has 128 experts; each token is routed to the top-8 by a learned gate. Only 8/128 = 6.25% of expert weights are active per token per layer.
- **Active parameter math:**
  - 3.4B active per single pass (base routing)
  - Each loop pass adds ~1.0B effective active params
  - 3 passes → 5.4B effective active per token
  - Total weights: 26.4B, but only ~5.4B touch each token
- **Hybrid attention:** 20 of 24 layers use sliding window (1024) attention; 4 layers are global. This keeps KV cache small for the 1M context.
- **Log scaling:** Uses logarithmic position scaling (not plain RoPE) with a floor of 8192 and alpha 0.1. This is a novel positional encoding for long-context.
- **QK L2 clipping:** qk_l2_clip_threshold=22.63 and rel_l2_clip_threshold=8.0 — attention logit clipping for numerical stability at 1M context.
- **Vision:** Natively multimodal. 23-layer ViT with temporal patches (video-capable). Images 40-4096px per dimension optimal.
- **MTP speculator:** A 3-layer Multi-Token Prediction head with block_size 8 and Markov rank 256. Ships as `mtp.safetensors` in the base repo. This is the "DSpark speculator" — separate from the MTP head, it's a block-diffusion style drafter (like Muse's DFlash but different mechanism).
- **Looping + monitorability:** Thinking Machines explicitly studied whether looping harms interpretability. Finding: "very little controllability over its chain of thought" — the model can do more computation without visible tokens. However, it "was unable to perform covert side tasks in realistic agentic environments."

### Key implications for AI Box
- **MoE + looping = compute-efficient:** Only 5.4B active per token means faster inference than a dense 26B model, despite the larger total weight footprint.
- **1M context is a feature, not a constraint:** The hybrid attention (20 sliding + 4 global) + log scaling means the 1M context is architecturally supported, not just a marketing number. KV cache at 1M will be large but the 3:1 GQA helps.
- **Looping in llama.cpp:** This is the critical integration question. The llama.cpp PR #32694 (merged in b12600, Nov 20 2026) adds Inkling 2 Mini support including the loop graph and `is_varloop` attribute. This is **brand new** — merged the same day as the model release.

## 3. Quantization & File Inventory

### Base repo (thinkingmachines/Inkling-2-Mini)

| File | Size | Purpose |
|---|---|---|
| `model-00001-of-00001.safetensors` | ~24.8 GB (mixed NVFP4 + MXFP8 + BF16 + F32) | Main model weights |
| `mtp.safetensors` | ~2.5 GB (BF16) | MTP speculative head |
| `config.json` | — | Architecture config |
| `chat_template.jinja` | — | Chat template |
| `processor_config.json` | — | Vision processor config |
| `tokenizer.json` / `tokenizer.model` / `special_tokens_map.json` / `tokenizer_config.json` | — | Tokenizer files |
| `generation_config.json` | — | Generation defaults |
| `hf_quant_config.json` | — | Quantization config (modelopt NVFP4) |

**HF API safetensors breakdown:**
| Precision | Parameters |
|---|---|
| F32 | 247,848 (tiny — probably embeddings or norms) |
| BF16 | 1,927,217,664 (~1.93B) |
| F8_E4M3 (MXFP8) | 4,076,863,488 (~4.08B) |
| U8 (NVFP4 packed) | 20,384,317,440 (~20.38B) |
| **Total** | **26,388,646,440 (26.39B)** |

The native checkpoint is **~20 GB** (mixed NVFP4/MXFP8). This is the "single checkpoint" Thinking Machines distributes — it's already quantized, not a BF16 original.

### Unsloth GGUF repo (unsloth/Inkling-2-Mini-GGUF)

| File | Size | KLD vs Q8_K_XL | Same top token |
|---|---|---|---|
| `Inkling-2-Mini-UD-IQ1_S.gguf` | 7.76 GB | 0.0911 | 89.4% |
| `Inkling-2-Mini-UD-IQ1_M.gguf` | 8.02 GB | 0.0867 | 90.2% |
| `Inkling-2-Mini-UD-IQ2_XXS.gguf` | 8.49 GB | 0.0726 | 90.6% |
| `Inkling-2-Mini-UD-Q2_K_XL.gguf` | 9.14 GB | 0.0595 | 91.6% |
| `Inkling-2-Mini-UD-IQ3_XXS.gguf` | 11.66 GB | 0.0269 | 94.7% |
| `Inkling-2-Mini-UD-Q3_K_XL.gguf` | 13.63 GB | 0.0174 | 96.3% |
| `Inkling-2-Mini-UD-IQ4_XS.gguf` | 15.73 GB | 0.0109 | 97.7% |
| `Inkling-2-Mini-UD-Q4_K_M.gguf` | **16.61 GB** | **0.0108** | **97.9%** |
| `Inkling-2-Mini-UD-Q4_K_XL.gguf` | **17.63 GB** | **0.0107** | **98.0%** |
| `Inkling-2-Mini-UD-Q8_K_XL.gguf` | 22.87 GB | reference | — |
| `dspark-Inkling-2-Mini-Q8_0.gguf` | 402 MB | — | DSpark speculator (Q8) |
| `dspark/dspark-Inkling-2-Mini-BF16.gguf` | ~1 GB | — | DSpark speculator (BF16) |
| `mmproj-BF16.gguf` | 642 MB | — | Vision encoder (BF16) |
| `mmproj-F16.gguf` | 639 MB | — | Vision encoder (F16) |
| `mmproj-F32.gguf` | 1.27 GB | — | Vision encoder (F32) |

**Mainline_Support/ subfolder:** Contains the same GGUF files with `_file` extension — these are the versions for mainline llama.cpp (as opposed to Unsloth's custom builds). Use these if you want mainline llama.cpp compatibility.

## 4. AI Box Fit Analysis

### Target hardware: 1x RTX 4090 (24 GB) or 3090 (24 GB)

**Verdict: FITS with margin. The MoE architecture is the key advantage.**

### Option A: Q4_K_M (recommended) — 16.61 GB

| Component | Size |
|---|---|
| Q4_K_M weights | 16.61 GB |
| mmproj-F16 (vision) | 0.64 GB |
| DSpark Q8 (speculator) | 0.40 GB |
| KV cache at 32K (GQA 3:1, 24 heads, 128 dim, 24 layers) | ~1.0-1.5 GB |
| **Total** | **~18.7-19.2 GB of 24 GB** |
| **Headroom** | **~5 GB** |

This is comfortable. 5 GB of headroom means you can push context to 64K or 128K without memory pressure.

### Option B: Q4_K_XL (higher quality) — 17.63 GB

| Component | Size |
|---|---|
| Q4_K_XL weights | 17.63 GB |
| mmproj-F16 (vision) | 0.64 GB |
| DSpark Q8 (speculator) | 0.40 GB |
| KV cache at 32K | ~1.0-1.5 GB |
| **Total** | **~19.7-20.2 GB of 24 GB** |
| **Headroom** | **~3.8-4.3 GB** |

Still fits. The 0.01% KLD difference between Q4_K_M and Q4_K_XL is negligible (97.9% vs 98.0% same-top-token).

### Option C: Q3_K_XL (smallest that's good) — 13.63 GB

| Component | Size |
|---|---|
| Q3_K_XL weights | 13.63 GB |
| mmproj-F16 | 0.64 GB |
| DSpark Q8 | 0.40 GB |
| KV cache at 64K | ~2.0-3.0 GB |
| **Total** | **~16.7-17.7 GB of 24 GB** |
| **Headroom** | **~6-7 GB** |

This is the "comfort" option — more headroom for context, but 96.3% same-top-token (1.7% worse than Q4). Only use if you need 128K+ context.

### Option D: Native NVFP4/MXFP8 checkpoint — ~20 GB

The base repo ships the model already quantized to NVFP4/MXFP8 (~20 GB). This is NOT a GGUF file — it requires SGLang, vLLM, or HuggingFace transformers (not llama.cpp). For the AI Box with llama.cpp, **use the Unsloth GGUF**.

### Recommended configuration for 24 GB card

```bash
./llama-server \
    -m Inkling-2-Mini-GGUF/Inkling-2-Mini-UD-Q4_K_M.gguf \
    --mmproj Inkling-2-Mini-GGUF/mmproj-F16.gguf \
    --draft-model Inkling-2-Mini-GGUF/dspark-Inkling-2-Mini-Q8_0.gguf \
    -ngl 99 \
    -c 65536 \
    -np 4 \
    --host 127.0.0.1 --port 8088 \
    --jinja \
    --alias inkling-2-mini
```

**Key settings:**
- `--jinja` for the chat template
- `-c 65536` (64K) — comfortable with Q4_K_M; push to 131K if you have the headroom
- `--draft-model` for the DSpark speculator (402 MB, adds ~1.5-2x decode speedup)
- No explicit sampling params documented in the model card — use defaults or `temp=0.6, top_p=0.95` as a starting point (Tinker playground uses similar)
- Reasoning effort is adjustable (the model supports variable reasoning effort, but minimum doesn't fully disable it due to the 3-pass loop)

### Speed expectations

No official tok/s numbers published yet (model released 1 day ago). However:
- 5.4B active per token on a 4090 → expect 150-300 tok/s decode (MoE models with 5B active typically hit this range)
- With DSpark speculator (block 8, Markov 256): expect 1.5-2.5x speedup → 225-600 tok/s
- This is **significantly faster** than Muse Glimmer's ~75 tok/s (dense 29.6B) or Qwen 3.8 27B (~60-80 tok/s dense)
- Prompt processing will be slower than decode (the 3-pass loop means 3x compute for prompt tokens)

## 5. Tool Calling & Function Calling

**Supported:** Yes. The model is explicitly designed for "coding agents, tool use, and knowledge work" (LM Studio description) and "agentic and tool-use systems" (model card).

- **Chat template:** `chat_template.jinja` in the repo (standard Jinja2, works with `--jinja` in llama.cpp)
- **Tool use format:** The model card doesn't specify a non-standard format like ATEM (Muse) — it uses the standard OpenAI-style `tool_calls` format in the chat template. This is a significant integration advantage over Muse Glimmer.
- **Adjustable reasoning effort:** The model supports variable reasoning effort (like Qwen 3.8). At minimum, it doesn't fully disable reasoning (the 3-pass loop always runs), but it reduces the visible thinking tokens.
- **1M context:** The 1M token context is architecturally supported (hybrid attention + log scaling). For Hermes agent work, this means you can keep much longer conversation histories without summarization.

**Hermes compatibility:**
- Standard tool call format → should work out of the box with llama.cpp's OpenAI-compatible endpoint
- No ATEM-style XML parsing needed (unlike Muse Glimmer)
- 1M context is a major advantage for long agent sessions
- **Primary risk:** The looped architecture is brand new in llama.cpp (PR #32694, b12600). Loop graph support may have bugs. The `is_varloop` attribute is new and may need specific handling.

## 6. Hermes Integration Checklist

- [ ] llama.cpp b12600+ (current b12602) — **looped MoE support is in b12600, merged Nov 20 2026**
- [ ] OpenAI-compatible endpoint (`/v1/chat/completions`) — standard tool_calls format
- [ ] Chat template via `--jinja`
- [ ] Sampling: no specific recommendation in model card; start with `temp=0.6, top_p=0.95`
- [ ] Reasoning: adjustable effort; minimum doesn't fully disable (3-pass loop)
- [ ] Context: 64K-131K practical on 24 GB; 1M architecturally supported
- [ ] Vision: mmproj-F16.gguf (639 MB) for image input; 40-4096px optimal
- [ ] Speculative: DSpark Q8 (402 MB) — block 8, Markov 256
- [ ] Memory: 18.7-20.2 GB total for Q4_K_M/Q4_K_XL + vision + speculator + 64K KV
- [ ] **Verify loop graph in llama.cpp works** — brand new (b12600), test with simple prompts first
- [ ] **Verify DSpark speculator works** — also new, may have bugs

## 7. Benchmarks (Thinking Machines-reported, from model card)

### Head-to-head with Muse Glimmer 30B and Qwen 3.8 27B

| Category | Benchmark | Inkling 2 Mini | Muse Glimmer 30B | Qwen 3.8 27B | Gemma 4 31B |
|---|---|---|---|---|---|
| **Agentic** | MCP Atlas (Public) | **82.7** | 75.5 | 77.3 | 54.2 |
| **Agentic** | DeepSearch QA (compaction) | **85.3** | 74.6 | 79.7 | 61.7 |
| **Agentic** | BrowseComp (compaction) | **79.0** | 63.9 | 61.6 | 37.0 |
| **Agentic** | JobBench | **49.6** | 16.1 | 33.4 | 8.2 |
| **Agentic** | AutomationBench-AA | 41.8 | 6.8 | **48.2** | 4.9 |
| **Agentic** | EnterpriseOps-Gym | 42.0 | 34.7 | **44.2** | 28.4 |
| **Agentic** | AA-AnalystAgent | **46.3** | 20.0 | — | — |
| **Coding** | SWE-Bench Verified (bash) | **79.2** | 76.0 | 77.8 | 46.1 |
| **Coding** | SWE-Bench Pro v2 Hard | **77.9** | 44.3 | 73.8 | 14.2 |
| **Coding** | SWE-Bench Multilingual | **73.7** | 66.9 | 70.0 | 43.0 |
| **Coding** | TerminalBench v2.1 (pi) | 71.6 | 51.7 | **73.0** | 43.4 |
| **Coding** | TerminalBench v4.0 (mini-swe) | **5.6** | 0.5 | **5.6** | 0.0 |
| **Coding** | FrontierCode v2.0 | **15.1** | 1.5 | 14.9 | 0.0 |
| **Coding** | Vibe Code Bench v1.1 | **77.9** | — | 64.9 | — |
| **Coding** | DeepSWE v1.1 | **48.2** | 4.4 | 42.2 | 0.3 |
| **Vision** | ScreenSpot Pro (orig/Py) | **76.5/87.7** | 75.4/— | 57.6/78.9 | 5.9/— |
| **Vision** | OmniDocBench v1.6 | **92.8** | 90.4 | 91.5 | 82.5 |
| **Vision** | MMMU Pro | 74.9 | 74.3 | **76.3** | 73.4 |
| **Vision** | PerceptionBench | **57.5** | 42.8 | 52.4 | 40.7 |
| **Vision** | Chartography (orig/Py) | **22.1/72.5** | 17.4/25.1 | 12.2/52.1 | 12.0/11.8 |
| **Vision** | MuirBench | 77.5 | **77.6** | 65.9 | 72.3 |
| **Vision** | ZeroBench (orig/Py) | **17.0/34.0** | 9.0/17.0 | 12.0/— | 9.0/12.0 |
| **Reasoning** | HLE-Diamond | 15.2 | 9.1 | **15.3** | 7.2 |
| **Reasoning** | CritPt v2 (Full/Hard) | **31.4/17.0** | —/7.0 | 10.6/4.0 | —/4.0 |
| **Reasoning** | Apex 2025 | **34.4** | 4.2 | 18.8 | 5.2 |
| **Reasoning** | ARC-AGI v1/v2 | 84.5/49.3 | —/— | **87.5/42.4** | —/— |
| **Reasoning** | LongBench v2 | **66.9** | 33.1 | 52.3 | 52.9 |
| **Reasoning** | GraphWalks BFS 256K | **69.7** | — | 50.4 | — |
| **Reasoning** | MRCR v2 (8-needle 256-512K) | **46.4** | — | 38.0 | — |
| **Knowledge** | AA-Omniscience v1.1 (score) | -8.6 | -32.2 | **-6.4** | -48.5 |
| **Knowledge** | AA-Omniscience (non-hallucination) | 63.4% | 16.4% | **71.4%** | 14.5% |
| **Safety** | FORTRESS adversarial | 89.5% | **90.3%** | 78.8% | 67.8% |
| **Safety** | Misbehavior (internal) ↓ | **9.8%** | 46.4% | 57.7% | 27.6% |
| **Safety** | Covert misbehavior ↓ | **8.3%** | 45.6% | 33.2% | 24.3% |
| **Safety** | Destructive actions ↓ | **10.2%** | 68.8% | 35.2% | 75.8% |

### Key findings:
1. **Inkling 2 Mini is the agentic king of this size class.** It leads in MCP Atlas (82.7 vs 75.5 vs 77.3), DeepSearch QA (85.3 vs 74.6 vs 79.7), BrowseComp (79.0 vs 63.9 vs 61.6), JobBench (49.6 vs 16.1 vs 33.4), and most coding benchmarks.
2. **Safety is a standout:** 9.8% misbehavior vs 46.4% (Muse) and 57.7% (Qwen). 8.3% covert misbehavior vs 45.6% and 33.2%. 10.2% destructive actions vs 68.8% and 35.2%. This is the safest model of the three by a wide margin.
3. **Qwen 3.8 wins on knowledge** (AA-Omniscience -6.4 vs -8.6) and some coding (TerminalBench v2.1, ARC-AGI v1).
4. **Inkling 2 Mini has the best long-context reasoning** (LongBench v2 66.9 vs 33.1, MRCR v2 46.4 vs 38.0, GraphWalks 69.7 vs 50.4) — the 1M context is not just a number.
5. **Vision is strong:** ScreenSpot Pro 76.5/87.7 (best), OmniDocBench 92.8 (best), PerceptionBench 57.5 (best).

## 8. Limitations (from model card)

- **Knowledge gaps:** Less capable than Inkling 2 Small on factual recall. More likely to state incorrect info with confidence.
- **Repetitive reasoning:** Can get stuck in long reasoning loops, especially when debugging code. **Recommend setting limits on output tokens and agent turns.**
- **Agentic reliability:** Less reliable than larger models at following detailed instructions. **May take destructive actions (deleting/overwriting files) without confirmation.** May not accurately report work outcomes.
- **Non-English:** Lower quality outside English. Chinese testing showed clipped phrasing and mixed languages.
- **Mental health topics:** May respond with excessive caution.
- **Reasoning effort:** Minimum doesn't fully disable reasoning (3-pass loop). Compute per token is higher than active-param count suggests.
- **Local runtime support:** New and varying by runtime. Speed varies by hardware.

## 9. Risks & Caveats

1. **Looped architecture is brand new in llama.cpp.** PR #32694 merged in b12600 (Nov 20, 2026) — the same day as the model release. This is the highest-risk integration path of the three models. Test thoroughly before production use.
2. **DSpark speculator is also new.** The dspark GGUF files are in the Unsloth repo but haven't been widely tested. May have bugs or performance issues.
3. **Destructive action risk:** The model card explicitly warns about file deletion/overwriting without confirmation. For Hermes agent work, this is a real risk — the model may `rm` or overwrite files in the workspace.
4. **1M context is a double-edged sword:** Architecturally supported, but KV cache at 1M will be large (~5-8 GB even with 3:1 GQA). Practical limit on 24 GB is 64K-131K.
5. **Repetitive reasoning loops:** The 3-pass loop can cause the model to re-check its work without making progress. Set max output tokens and agent turn limits.
6. **Knowledge cutoff:** Not explicitly stated in the model card. The model was released Nov 20, 2026, so training data likely goes through mid-2026.
7. **No official tok/s numbers yet.** Released 1 day ago — community benchmarks will follow in the coming days.

## 10. Verdict

**PASS — Strong candidate for AI Box deployment on 1x 24 GB GPU. Best agentic benchmarks of the three paths.**

- Fits 24 GB VRAM with 5 GB headroom (Q4_K_M + vision + speculator + 64K KV)
- 5.4B active per token → expected 150-300 tok/s (faster than Muse or Qwen)
- **Best agentic benchmarks:** MCP Atlas 82.7, DeepSearch 85.3, BrowseComp 79.0, SWE-Bench Verified 79.2
- **Best safety:** 9.8% misbehavior vs 46.4% (Muse) and 57.7% (Qwen)
- **1M context** architecturally supported
- Apache 2.0, Unsloth GGUF, llama.cpp b12600+ (current b12602)
- **Primary risk:** Looped MoE architecture is brand new in llama.cpp — needs integration testing
- **Secondary risk:** Destructive action tendency in agentic work

**Recommended config:**
```
Model:      Inkling-2-Mini-UD-Q4_K_M.gguf (16.61 GB)
Vision:     mmproj-F16.gguf (639 MB)
Speculator: dspark-Inkling-2-Mini-Q8_0.gguf (402 MB)
Context:    65536
Sampling:   temp=0.6, top_p=0.95 (starting point)
Flag:       --jinja
```

### Comparison vs Path F (Muse Glimmer) and Path I (Qwen 3.8 27B)

| Criterion | Inkling 2 Mini | Muse Glimmer 30B | Qwen 3.8 27B |
|---|---|---|---|
| VRAM (24 GB card) | 18.7-20.2 GB (Q4) | 20-22 GB (Q4) | ~17-18 GB (Q4) |
| Headroom | ~4-5 GB | ~2-4 GB | ~6-7 GB |
| Active params | 5.4B (MoE loop) | 29.6B (dense) | 27B (dense) |
| Est. tok/s (4090) | 150-300 | 70-80 | 60-80 |
| MCP Atlas | **82.7** | 75.5 | 77.3 |
| SWE-Bench Verified | **79.2** | 76.0 | 77.8 |
| Safety (misbehavior) | **9.8%** | 46.4% | 57.7% |
| Context | **1M** | 131K | 128K |
| Tool format | Standard | ATEM (non-standard) | Standard |
| llama.cpp maturity | New (b12600) | Mature (b10353+) | Mature |
| Integration risk | **High** (looped MoE) | Medium (ATEM) | Low |

**Bottom line:** Inkling 2 Mini is the strongest on paper (agentic benchmarks + safety + context + speed) but carries the highest integration risk (brand new looped MoE in llama.cpp). Muse Glimmer is the safest bet for reliable operation. Qwen 3.8 is the most mature in llama.cpp but has the weakest safety profile and dense compute.
