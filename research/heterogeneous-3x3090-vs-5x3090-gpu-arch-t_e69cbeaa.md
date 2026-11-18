# GPU Architecture Research — 3×3090 + Quadro RTX 8000 (120 GB) vs 5×3090 (120 GB)

- **Task:** t_e69cbeaa
- **Date:** 2026-11-17
- **Researcher:** GIU (Geraldo v2.2), Kanban `research` board
- **Classification:** Local LLM / AI Box fleet endpoint — GPU topology, heterogeneous VRAM pooling, engine compatibility

## Objective

1. Verify whether 3× RTX 3090 (24 GB each) + 1× Quadro RTX 8000 (48 GB) = 120 GB functions as a **single unified VRAM pool**, and identify the **specific software flags/settings** required to make heterogeneous (mixed-size) VRAM work together.
2. Assess whether **5× RTX 3090 (120 GB)** is the better topology for a **continuous-batch serving endpoint**, and what the interconnect (PCIe/NVLink) tradeoffs are.
3. Recommend which topology + engine is the best path forward for the fleet's continuous-batch endpoint.

## TL;DR (bottom line up front)

- **The 120 GB "single pool" claim is only true in llama.cpp's pipeline-parallel mode** (`--split-mode layer`, the default), where memory is auto-allocated **proportionally to each card's VRAM** (the 48 GB 8000 takes ~2/5 of the model, each 24 GB 3090 takes ~1/5). In **vLLM/sglang tensor parallelism the pool is *not* unified** — upstream vLLM splits TP shards **evenly** and a TP group is "only as large as its smallest GPU" (~72 GB usable of 120 GB).
- **vLLM's uneven-shard feature was proposed (PR #50735, `--rank-tp-ratio`) and closed UNMERGED by a maintainer**, who instead directed: use **pipeline parallelism + `VLLM_PP_LAYER_PARTITION`**. Upstream vLLM today has **no uneven-TP flag**.
- The **only validated asymmetric-TP engine is `htsglang`** (efschu's sglang fork) — it ran **5090 + 3080s** with `--rank-tp-ratio`/`--rank-gpu-memory-mib` and is purpose-built for exactly this "mismatched GPU" scenario.
- The **RTX 8000 is 2018 Turing (sm_75)**: no BF16, no FP8/FP4, no FlashAttention-2. It works in vLLM **only** in FP16 with TP-onto-NVLink + PP, and runs ~10 tok/s on a 70B Q4. It is effectively a **llama.cpp/koboldcpp appliance** for the frontier path.
- **5×3090 is the stronger continuous-batch topology**: uniform architecture (sm_86), no legacy-card kernel gaps, and (with the patched driver) clean PCIe P2P. The mixed pool buys capacity with a performance + maintenance penalty and a dying architecture.
- **Recommendation:** if you must run 120 GB *now*, run **llama.cpp** on the 3×3090+8000 with `--split-mode layer` (auto) or `-ts 2,1,1,1` — that is the only mode where the 48 GB card is fully exploited. For a production continuous-batch endpoint, **target 5×3090** (or a uniform-VRAM build) with **vLLM** (TP where NVLink/P2P is available, else PP), and treat `htsglang` as the fallback for a genuinely mixed fleet.

---

## Plan

```
objective: verify 120GB heterogeneous pool + recommend 5x3090 vs mixed
success_criteria:
  - confirm what "single pool" means per engine (llama.cpp vs vLLM/sglang)
  - find the exact flags for mixed VRAM in each engine
  - characterize the RTX 8000 (Turing) software ceiling
  - compare 5x3090 uniform vs mixed on continuous-batch serving
assumptions:
  - Linux bare-metal; 3090 = sm_86 Ampere; 8000 = sm_75 Turing
  - "continuous-batch endpoint" = a served API (vLLM/sglang/llama.cpp server)
  - interconnect is PCIe (consumer boards); NVLink optional
tasks:
  T1 llama.cpp split modes + uneven VRAM        -> done (primary docs)
  T2 vLLM TP-vs-PP + uneven-shard support       -> done (docs + PR #50735)
  T3 sglang/htsglang asymmetric TP              -> done (efschu/htsglang)
  T4 RTX 8000 Turing software ceiling           -> done (vLLM issue + 4x8000 projects)
  T5 5x3090 interconnect (P2P/NVLink/PCIe)      -> done (P2P forks + benches)
  T6 fleet recommendation                       -> done
open_questions:
  - Is the 5x3090 board PCIe 3.0 or 4.0? (affects P2P ceiling)
  - Which exact model/quant is the fleet endpoint serving? (drives engine choice)
```

---

## Findings (by engine)

### 1. llama.cpp — the only engine where 120 GB is a *true* unified pool

Source: `ggml-org/llama.cpp` `docs/multi-gpu.md` (primary, pulled 2026-11-17).

- **`--split-mode` (`-sm`)** has four values:
  - `none` — single GPU only.
  - **`layer` (DEFAULT) = pipeline parallelism.** "Each GPU holds a contiguous slice of layers. The KV cache for layer *l* lives on the GPU that owns layer *l*." This is the mode where a mixed 120 GB pool actually works: **`layer`/`row` use automatic splitting *proportional to memory*** when `--tensor-split` is omitted. So a 48 GB card automatically takes ~2× the bytes of each 24 GB card. KV cache spreads with the layers.
  - `row` — **deprecated** old row-split TP path; "superseded by `tensor`."
  - **`tensor` — EXPERIMENTAL** true tensor parallelism via a "meta device" abstraction; splits weights **and** KV. Hard requirements: flash-attention on, non-quantized KV (`f16`/`f32`/`bf16`), and it is **not implemented for many architectures** (MoE/hybrid: Grok, DeepSeek2, Mamba, Gemma-3n, etc.). `--fit` auto-fit is **off** in tensor mode.
- **Uneven control = `--tensor-split` (`-ts`), comma proportions** e.g. `-ts 2,1,1,1`. "With `3,1` on two GPUs, GPU 0 gets 75 %, GPU 1 gets 25 %. The values follow the order in `--device`." Normalized, so `2,1,1,1` = 40/20/20/20. This is the explicit knob to pin the 48 GB 8000 at 2/5.
- **`--device` / `--list-devices`** select which cards participate; **`CUDA_VISIBLE_DEVICES`** restricts the CUDA backend; **`--main-gpu`** picks the single card in `none` mode.
- **`--fit` (on by default)** auto-sizes ctx/KV to device memory (disabled in `tensor` mode).
- **NCCL** is selected at build time (`-DGGML_CUDA_NCCL=ON`, the default) and used for cross-GPU reductions in `tensor` mode.
- **`GGML_CUDA_P2P=1`** enables CUDA peer-to-peer (direct GPU↔GPU DMA, no system-memory bounce). Docs warn: "requires driver support (usually restricted to workstation/datacenter GPUs) and **may cause crashes or corrupted outputs on some motherboards or BIOS configurations (e.g. when IOMMU is enabled)**."

**Bottom line for the mixed box:** `llama-cli/-server -m model.gguf` with defaults already pools the 120 GB proportionally; pin it with `-ts 2,1,1,1` (or `auto` which does the same). Use `--split-mode layer` (not `tensor`) for the mixed cards — it's the "default and most compatible" mode, tolerates the slower PCIe interconnect, and is the only mode with full architecture coverage. `GGML_CUDA_P2P=1` + Resizable BAR improves it if the board supports it (see §3).

### 2. vLLM — NOT a unified pool under TP; use PP for mixed VRAM

Sources: vLLM `docs/serving/parallelism_scaling.md` (primary), vLLM PR #50735 (primary), vLLM issue #47549 (primary).

- **Tensor parallelism (TP) splits every sharded dimension *evenly*:** `divide()` / `split_tensor_along_last_dim` in `vllm/distributed/utils.py` give each rank `dim // world_size`. A TP group is **only as large as its smallest GPU** — "pairing one 32 GB card with two 20 GB cards leaves roughly 12 GB unusable" (PR #50735 body). **So in TP the 48 GB 8000's extra memory is wasted; the effective pool collapses toward 3090-sized shards.**
- **vLLM's own docs handle this exact case with PP:** "If the model fits within a single node but the GPU count doesn't evenly divide the model size, **enable pipeline parallelism, which splits the model along layers and supports uneven splits**. In this scenario, set `tensor_parallel_size=1` and `pipeline_parallel_size` to the number of GPUs." And: "if the GPUs on the node do **not** have NVLink interconnect … **leverage pipeline parallelism instead of tensor parallelism** for higher throughput and lower communication overhead."
- **Uneven PP layers = `VLLM_PP_LAYER_PARTITION`** env var: "a comma-separated list of integers whose length equals `--pipeline-parallel-size` and whose sum equals the total number of transformer layers," e.g. `export VLLM_PP_LAYER_PARTITION=32,29` for a 61-layer model at PP2. Auto-partition (even split, remainders to middle stages) is the default; override only when a stage is memory/latency-imbalanced. The last stage also holds the LM head + sampling (+ drafter if spec-decoding), so it often needs a hand-off of a layer or two.
- **TP-vs-PP decision rule (vLLM docs):** TP when you want max throughput/latency *and* have a fast interconnect (NVLink/InfiniBand); PP when GPUs are heterogeneous or lack NVLink.
- **Uneven-TP was proposed and REJECTED in-tree:** PR **#50735 "[Feature] Add `--rank-tp-ratio` for uneven tensor parallelism on mismatched GPUs"** — a 22-file patch (prefix-sum partition helpers, ratio-aware `linear.py`/`vocab_parallel_embedding.py`/`parameter.py`, per-rank KV specs) — was **closed 2026-08-03, `merged: False`**. Maintainer **@hmellor**: *"Uneven TP sharding is not something I think we are interested in supporting in-tree in vLLM as it is quite niche and appears to need changes to every supported model. In situations where you have different sized GPUs you likely don't have fast interconnect to run TP efficiently anyway. You should probably be using PP instead, which can already be unevenly split across GPUs using `VLLM_PP_LAYER_PARTITION`."*
  - **Consequence:** upstream vLLM **has no `--rank-tp-ratio` / uneven-TP flag today.** (The two other PRs referencing the term, #61479/#61674, are unrelated KV-offload/frontend bugfixes that merged — not the feature.)
- **The 8000 in vLLM is FP16-only and slow:** vLLM issue **#47549** (an 18× Quadro RTX 8000 cluster) documents: `--kv-cache-dtype fp8` **fails on sm_75** ("native FP8 (fp8e4nv) requires SM89+"), and vLLM **v0.24.0 regressed** the FlashInfer FP8-KV path on sm_75 (a v0.23.0→v0.24.0 drop), cutting KV capacity 2× (287,744 → ~143,000 tokens) and decode ~5–11 → ~0.8–1.1 tok/s. Workaround cited: pin `vllm-openai:v0.23.0`. The cluster runs **TP=2 + PP=2** in FP16 (TP mapped onto NVLink pairs, PP across pairs).

**Bottom line for the mixed box in vLLM:** you **cannot** get the 48 GB card fully used under TP. You must run **PP** (`--pipeline-parallel-size 4 --tensor-parallel-size 1`, or TP-within-a-NVLink-pair + PP-across), with `VLLM_PP_LAYER_PARTITION` tuned so the 48 GB card owns the deepest stage. Expect FP16-only, no FA2, modest throughput.

### 3. sglang / **htsglang** — the only *validated* asymmetric-TP engine

Source: `efschu/htsglang` README (primary).

- Fork of **sglang** whose headline feature is **"asymmetric tensor parallelism (for heterogeneous / mismatched GPUs)"** plus Qwen3.5/3.6 GGUF. Image: `ghcr.io/efschu/htsglang:cu130-nccl2307` (pins NCCL 2.30.7, "required for multi-rank-per-GPU co-location").
- **Fork flags (the exact knobs for a mixed pool):**
  - `--rank-gpu-id <id,...>` — one CUDA device index **per TP rank**; duplicates **co-locate** ranks on one GPU (length must equal `--tp-size`).
  - `--rank-gpu-memory-mib <MiB>[,...]` — **absolute memory budget per rank** (scalar or per-rank list with `--rank-tp-ratio`); no further utilization ceiling.
  - `--rank-tp-ratio auto|auto-performance|<w,...>` — **uneven TP: integer shard weights per rank**, derived from the memory budgets (`auto`) or NVML totals, or a measured hardware profile (`auto-performance`).
  - `--rank-auto-reserve-mib` — headroom subtracted from NVML total when `auto` derives budgets.
  - `--rank-mlp-ratio` / `--rank-moe-ratio` — per-rank weight rebalancing for dense-MLP / fused-MoE families "so the min-synced KV pool grows."
- **This is the engine that actually demonstrated a 48-GB-class + 24-GB-class card in one TP group** (the project's motivating case is a 5090 + 3080s box). For a 3×3090+8000 box, `--rank-gpu-id 0,1,2,3 --rank-tp-ratio 2,1,1,1` (or `auto`) + `--rank-gpu-memory-mib` per rank is the direct analog of llama.cpp's `-ts 2,1,1,1` — but inside a **continuous-batch PagedAttention engine** (sglang), which llama.cpp is not.

### 4. RTX 8000 (Turing sm_75) software ceiling

Sources: vLLM issue #47549; `agorevski/DeepSeekRTX8000` (4×RTX 8000 project, primary README); `1CatAI/1Cat-vLLM` issue #441 (5-GPU 2×8000+3×V100); runaihome.com "Used Quadro RTX 8000 in 2026."

- **Turing TU102, sm_75, 48 GB GDDR6, 672 GB/s, 4608 CUDA cores, 260–295 W, 2-slot blower.**
- **Numerical gaps vs Ampere:** **no native BF16** (software-emulated → significant slowdown, avoid), **no FP8/FP4** (no hardware), **no TF32**, **no FlashAttention-2** (needs sm_80+; falls back to Triton attention, which is itself limited by Turing's **64 KB/SM** shared-memory ceiling — the Triton FP8-KV path errors on sm_75). INT8/INT4 work via tensor cores when kernels target sm_75.
- **Consequence for engine choice:** "Buy this card and you are buying a **llama.cpp/koboldcpp appliance**" (llama.cpp's own flash-attention runs on Turing; GGUF Q4_K_M is the sweet spot). The performance-frontier paths — **vLLM FP16, ExLlamaV3, NVFP4, FP8** — all require **Ampere or newer**. vLLM works but is FP16-locked and ~10 tok/s on a 70B Q4 (single 8000).
- **Pre-Ampere serving is real but fragile:** the 1Cat-vLLM SM70/75 tuning (issue #441) shows a **5-GPU 2×RTX 8000 + 3×V100** box running a 180B model at **TP2×PP2** with custom NVFP4 MoE kernels, but only after hand-writing sm_70/sm_75 kernels and a per-worker FA-version fix (upstream vllm#54758) — i.e., a fork, not stock vLLM.
- **NVLink caveat:** the 8000 **does** have NVLink (dual 112.5 GB/s pairs), so a 2×8000 NVLink pair is a clean TP unit; cross-pair traffic goes over PCIe — which is why those projects map **TP onto the NVLink pair and PP across pairs.**
- **Driver horizon:** NVIDIA ended Maxwell/Pascal/Volta support in late 2025; **Turing is the oldest architecture still on the current driver branch** (CUDA 13 still targets sm_75) — a software horizon of "a couple of years, not five."

### 5. 5×3090 interconnect (the continuous-batch case)

Sources: `tinygrad/open-gpu-kernel-modules` (primary, 1445★, pushed 2026-11-06); `alesha-pro/4x3090-llm-benchmarks`; runaihome.com "Dual RTX 3090 PCIe/IOMMU/ACS/NCCL guide"; myaihardware.com NVLink-vs-PCIe-P2P; r/LocalLLaMA 4×3090 P2P thread.

- **RTX 3090 = Ampere sm_86, 24 GB GDDR6X, 936 GB/s, 10496 CUDA cores, PCIe Gen4, NVLink 112.5 GB/s (2-way only; no 3/4-way NVLink).**
- **PCIe peer-to-peer is NOT on by default on consumer cards.** Stock driver: `nvidia-smi topo` shows `PHB`/`PXB` (path through host bridge / CPU) and NCCL falls back to **shared system memory** — "a round trip that adds latency and chews through PCIe bandwidth inefficiently." Two fixes, both used in the wild:
  - **Patched open-kernel driver** — `tinygrad/open-gpu-kernel-modules` ("NVIDIA Linux open GPU with **P2P support**", 1445★, last push 2026-11-06) enables **PCIe P2P between consumer GeForce 3090 (GA102) / 4090 (AD102) / 5090 (GB202)**. Community reports: "P2P driver patch works well … mostly helps with **vLLM tensor parallel** and training." A second fork (`YangWang92/p2p-open-gpu-kernel-modules`) targets driver 610.57.04.
  - **Resizable BAR + IOMMU-off (bare metal).** NCCL's troubleshooting guide: "on Linux bare metal, CUDA and the driver stack **do not support PCIe P2P with the IOMMU in translated mode** … the IOMMU must be disabled … to prevent silent device memory corruption." PCI **ACS** on a PCIe switch can also redirect P2P to the root complex (slowdown/hang) — disable it on bare metal. `lspci -vvv | grep ACSCtl` and `cat /proc/cmdline` are the checks.
- **Verified real builds:**
  - **4× RTX 3090 (96 GB), PCIe 3.0 x16, no NVLink, CUDA P2P enabled & verified across all 12 directed GPU pairs**, 220 W/card — `alesha-pro/4x3090-llm-benchmarks`: **638 local-inference measurements**, mostly vLLM, each row keeping TP/PP/DP layout + exact argv.
  - **4× FE 3090 + two NVLink bridges (two 2-card pairs)** on a Supermicro H12SSL with the **datacenter driver 565.57.01** → P2P over PCIe enabled; llama.cpp + vLLM both faster; `nvidia-smi` showed "more evenly distributed %" across cards (better parallel generation).
  - **3× 3090 on Windows** (risers, Gen4 ×4/×8/×8, no P2P/NVLink): 2-GPU `internal` reductions = **72–74 t/s gen, ~1200–1400 t/s prefill**; 3-GPU with NCCL = ~65 t/s; 3-GPU `internal` falls back to butterfly → ~37 t/s. Lesson: **2-card TP pairs beat a 3-way TP** when the interconnect is weak.
- **TP vs PP on a PCIe-only multi-3090 box (vLLM guidance):** TP needs a fast interconnect (NVLink/InfiniBand) or you pay per-layer all-reduce over PCIe; **vLLM's docs recommend PP over TP when there's no NVLink.** With the P2P driver + Resizable BAR + IOMMU-off, PCIe P2P makes TP viable (that's the whole point of the fork); without it, prefer PP or 2-card TP pairs.
- **5-card practicality:** 5× 3090 on PCIe 3.0 x16 at 220 W/card ≈ **1.1 kW GPU power** + system → ~1.6–1.8 kW total; needs a server board with ≥5 full x16 slots (X299/EPYC/TRX-class) and 8-pin feeders. 3090s are 3-slot; 5 cards need vertical spacing (often risers / a 2U server chassis). A 5th card also means **no 5-way NVLink** (NVLink is 2-way only) — so 5×3090 is **PCIe-only** for cross-card traffic, making the P2P driver + Resizable BAR effectively mandatory for TP, or you run PP / 2+2+1.

---

## Claims (structured)

### C1 — llama.cpp pools 120 GB as one memory space by default
- **statement:** In llama.cpp `--split-mode layer` (default), the model + KV cache are distributed across all visible GPUs **proportionally to each card's free memory**, so a 48 GB 8000 + 3×24 GB 3090 = 120 GB is a single logical pool; `-ts 2,1,1,1` pins the split explicitly.
- **evidence_refs:** E1, E2
- **confidence:** high
- **status:** supported

### C2 — vLLM TP does NOT unify heterogeneous VRAM; it wastes the big card
- **statement:** vLLM tensor parallelism splits every sharded dimension evenly (`dim // world_size`), so a TP group is limited to the smallest GPU and the 48 GB 8000's extra memory is unusable under TP. Upstream vLLM has **no uneven-TP flag**.
- **evidence_refs:** E3, E4
- **confidence:** high
- **status:** supported

### C3 — The correct vLLM way to use a mixed-VRAM box is PP + VLLM_PP_LAYER_PARTITION
- **statement:** vLLM's own docs and the maintainer who closed the uneven-TP PR both direct heterogeneous-VRAM users to **pipeline parallelism** (TP=1, PP=#GPUs) with `VLLM_PP_LAYER_PARTITION` for uneven layer counts, and to prefer PP over TP when GPUs lack NVLink.
- **evidence_refs:** E3, E4, E5
- **confidence:** high
- **status:** supported

### C4 — htsglang is the only engine with validated asymmetric TP
- **statement:** `efschu/htsglang` (sglang fork) ships `--rank-tp-ratio` / `--rank-gpu-memory-mib` / `--rank-gpu-id` for **uneven tensor parallelism across mismatched GPUs** (demonstrated on 5090+3080s) and is the only engine where a mixed 120 GB pool can be used **inside a continuous-batch PagedAttention engine**.
- **evidence_refs:** E6
- **confidence:** medium-high (single-project source; the fork is real and active but not an upstream engine)
- **status:** supported

### C5 — The RTX 8000 (Turing sm_75) is a llama.cpp appliance, not a frontier engine
- **statement:** The Quadro RTX 8000 lacks native BF16, FP8, FP4, TF32 and FlashAttention-2; vLLM runs it **FP16-only** (FP8-KV errors on sm_75; the Triton path is capped by 64 KB/SM shared memory), and ExLlamaV3 / NVFP4 / FP8 require Ampere+. It is effectively a llama.cpp/koboldcpp card for local LLMs.
- **evidence_refs:** E7, E8, E9
- **confidence:** high
- **status:** supported

### C6 — 3090s need a patched driver (or Resizable BAR + IOMMU-off) for PCIe P2P
- **statement:** Consumer 3090s do not expose PCIe P2P by default; NCCL falls back to shared host memory. `tinygrad/open-gpu-kernel-modules` (1445★) enables PCIe P2P on GA102/AD102/GB202; bare-metal also needs Resizable BAR and **IOMMU disabled** (translated-mode IOMMU breaks P2P / risks silent corruption), and PCIe **ACS** disabled on switches.
- **evidence_refs:** E10, E11, E12
- **confidence:** high
- **status:** supported

### C7 — 5×3090 is the stronger continuous-batch topology than 3×3090+8000
- **statement:** For a continuous-batch serving endpoint, **5×3090 (120 GB)** is preferred over 3×3090+1×8000 (120 GB): uniform sm_86 architecture (no legacy-card kernel gaps, full vLLM/sglang/ExLlamaV3/NVFP4 path), clean 2-card NVLink or PCIe-P2P pairing, and no dying-architecture driver horizon. The mixed box only wins on "raw VRAM in one llama.cpp pool" at the cost of a slow, FP16-locked, Turing card.
- **evidence_refs:** E1, E2, E7, E10, E13
- **confidence:** medium-high (depends on the 5×3090 board's PCIe generation and whether P2P is enabled)
- **status:** partial

### C8 — 3090 NVLink is 2-way only; 5 cards are PCIe-bound for cross-card traffic
- **statement:** The RTX 3090 supports NVLink (112.5 GB/s) for **two cards only** (no 3/4/5-way NVLink bridge); a 5×3090 box therefore relies on PCIe for cross-pair traffic, making the P2P driver effectively required for TP, or you run PP / 2+2+1 TP pairs.
- **evidence_refs:** E13, E14
- **confidence:** high
- **status:** supported

---

## Evidence index

| ID | Source | URL | Provenance / note |
|----|--------|-----|-------------------|
| E1 | llama.cpp `docs/multi-gpu.md` | https://github.com/ggml-org/llama.cpp/blob/master/docs/multi-gpu.md | Primary doc (raw pulled). Split modes, `-ts` proportions, NCCL, `GGML_CUDA_P2P`, `--fit`. |
| E2 | llama.cpp multi-gpu.md, Recipes §2/§6 | (same as E1) | `-ts 3,1` proportional-split example; `GGML_CUDA_P2P` + IOMMU crash warning. |
| E3 | vLLM `docs/serving/parallelism_scaling.md` | https://docs.vllm.ai/en/stable/serving/parallelism_scaling/ | "Edge case: uneven GPU splits → PP, TP=1, PP=#GPUs"; "PP instead of TP if no NVLink." |
| E4 | vLLM **PR #50735** + maintainer comment | https://github.com/vllm-project/vllm/pull/50735 | Closed 2026-08-03, **merged=False**. hmellor: "uneven TP … not … in-tree … use PP instead … `VLLM_PP_LAYER_PARTITION`." 22-file patch. |
| E5 | vLLM Ascend PP doc + `vllm/distributed/utils.py` | https://docs.vllm.ai/projects/ascend/en/latest/user_guide/feature_guide/pipeline_parallel.html ; https://github.com/vllm-project/vllm/blob/.../vllm/distributed/utils.py | `VLLM_PP_LAYER_PARTITION` rules (count==PP, sum==layers); `get_pp_indices` auto-partition code. |
| E6 | `efschu/htsglang` README | https://github.com/efschu/htsglang | Asymmetric TP; `--rank-tp-ratio`, `--rank-gpu-memory-mib`, `--rank-gpu-id`; NCCL 2.30.7 pin; 5090+3080s motivation. |
| E7 | vLLM **issue #47549** (18× Quadro RTX 8000 cluster) | https://github.com/vllm-project/vllm/issues/47549 | FP8-KV fails on sm_75; v0.24.0 FlashInfer regression; TP=2+PP=2 FP16; pin v0.23.0. |
| E8 | `agorevski/DeepSeekRTX8000` (4×RTX 8000) | https://deepwiki.com/agorevski/DeepSeekRTX8000/2-hardware-platform | sm_75: no BF16/FP8/FA2; FP16 for vLLM, Q4_K_M for llama.cpp; NVLink pairs 0-1/2-3; Xeon W-2295. |
| E9 | runaihome.com "Used Quadro RTX 8000 in 2026" | https://runaihome.com/blog/used-quadro-rtx-8000-local-ai-worth-it-2026/ | "llama.cpp/koboldcpp appliance"; vLLM off the table for multi-user; Turing driver horizon ~2 yrs. |
| E10 | `tinygrad/open-gpu-kernel-modules` | https://github.com/tinygrad/open-gpu-kernel-modules | 1445★, pushed 2026-11-06; "NVIDIA Linux open GPU with P2P support" (consumer GA102/AD102/GB202). |
| E11 | runaihome.com "Dual RTX 3090 PCIe/IOMMU/ACS/NCCL guide" | https://runaihome.com/blog/dual-rtx-3090-pcie-iommu-acs-nccl-guide-2026/ | P2P <~20 GB/s symptom; NCCL hangs; IOMMU/ACS as P2P killers. |
| E12 | NVIDIA NCCL troubleshooting guide | https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/troubleshooting/gpu_troubleshooting.html | IOMMU must be disabled for PCIe P2P (bare metal); ACS redirect warning. |
| E13 | NVIDIA RTX 3090 spec page | https://www.nvidia.com/en-us/geforce/graphics-cards/30-series/rtx-3090/ | sm_86, 24 GB, 936 GB/s, 10496 CUDA, PCIe Gen4, NVLink (SLI-Ready). |
| E14 | Legit Reviews 3090 review + gpucompat 3090 | https://www.legitreviews.com/nvidia-geforce-rtx-3090-founders-edition-review_222243 ; https://gpucompat.com/gpus/nvidia-rtx-3090/ | NVLink 112.5 GB/s, **2-way only** ("3-way and 4-way configurations are not supported"); 936 GB/s. |
| E15 | `alesha-pro/4x3090-llm-benchmarks` | https://github.com/alesha-pro/4x3090-llm-benchmarks | 4×3090, PCIe 3.0 x16, 96 GB, CUDA P2P enabled on all 12 pairs, 220 W/card; 638 vLLM-heavy measurements w/ exact argv. |
| E16 | r/LocalLLaMA "Got P2P working with 4x 3090s" + "Multiple RTX 3090 P2P driver" | https://www.reddit.com/r/LocalLLaMA/comments/1c4gakl/ ; https://www.reddit.com/r/LocalLLaMA/comments/1tict18/ | datacenter driver 565.57.01 enables PCIe P2P; P2P patch "helps vLLM TP"; 3090 NVLink = one bridge, pairwise only. |

---

## Gaps / open questions

1. **The 5×3090 board's PCIe generation is unknown.** If it's a PCIe 3.0 x16 board (like the 4×3090 archive), P2P bandwidth is ~half of Gen4; if Gen4, better. This changes whether 5-way TP is worthwhile vs PP / 2+2+1. **Action: confirm the board + slot spacing before committing to 5 cards.**
2. **The exact model/quant the fleet endpoint serves is unspecified.** A dense ≤70B Q4 runs on 3×3090+8000 in llama.cpp today; a 180B MoE needs the 5×3090 (or the 8000+V100 fork path). Engine choice (llama.cpp vs vLLM vs htsglang) depends on this.
3. **htsglang maturity** — it is a single-author fork pinned to NCCL 2.30.7 and Qwen3.5/3.6 GGUF; no SLA, no upstream. Treat as a proof-of-concept, not a production dependency, until it's proven on the actual target model.
4. **The 8000's FP8-KV v0.24.0 regression** (E7) means even the FP16 vLLM path on Turing is moving targets — a vLLM upgrade can silently halve KV capacity on sm_75.
5. **No hard numbers were produced for this specific 3×3090+8000 vs 5×3090** (no live GPU access from this container). The recommendation is grounded in the cited benchmarks (E15, E16, E7–E9) and engine docs, not a fresh benchmark run on these exact cards.

---

## Confidence

- **level:** high (for engine-flag facts, RTX-8000 ceiling, P2P prerequisites) / medium (for the final topology *recommendation*, which rests on architecture logic + cited benchmarks rather than a live benchmark of this exact box)
- **rationale:** Every engine flag, the vLLM uneven-TP rejection, the RTX-8000 Turing gaps, and the 3090 P2P driver facts are backed by primary sources (upstream docs, the actual PR + maintainer comment, the fork's own README, NVIDIA's NCCL guide, NVIDIA's spec page). The one soft spot is C7 (5×3090 > mixed) — it is well-supported directionally by the uniform-architecture and driver-horizon arguments, but a definitive answer needs the 5×3090 board's PCIe gen and the target model's size/quant.

---

## Recommended next actions

1. **If you must run 120 GB on the existing 3×3090+8000 today:** serve with **llama.cpp** — `llama-server -m <model>.gguf -ngl all -sm layer -ts 2,1,1,1` (or leave `-ts` off for auto-proportional). Add `GGML_CUDA_P2P=1` **only after** enabling Resizable BAR, disabling IOMMU (bare metal) and PCIe ACS, and confirming `nvidia-smi topo -p2p p` shows OK pairs; validate with `nccl-tests` / a quality pass before trusting it (P2P can silently corrupt on bad BIOS — llama.cpp docs warn).
2. **For a production continuous-batch endpoint, target 5×3090** (or a uniform-VRAM build) with **vLLM**:
   - First confirm the board's PCIe gen + 5× full-x16 spacing + power (≈1.6–1.8 kW).
   - Enable the **P2P driver** (`tinygrad/open-gpu-kernel-modules`) + Resizable BAR + IOMMU-off so TP can use PCIe P2P; then `--tensor-parallel-size 5` (or **2+2+1** via NVLink pairs + a PP stage, since NVLink is 2-way only).
   - If P2P won't be enabled, fall back to **PP** (`--tensor-parallel-size 1 --pipeline-parallel-size 5`) per vLLM's own uneven-GPU guidance.
3. **If the fleet truly must stay heterogeneous (mixed VRAM) and needs PagedAttention/continuous batching:** prototype **htsglang** with `--rank-gpu-id <all 5> --rank-tp-ratio auto --rank-gpu-memory-mib <per-rank>`, pinning NCCL 2.30.7, and A/B it against vLLM-PP on the target model.
4. **Do NOT plan on the RTX 8000 for the frontier path.** Keep it as a llama.cpp capacity card at most; for any vLLM/ExLlamaV3/NVFP4/FP8 requirement, the fleet needs Ampere-or-newer. Re-evaluate the 8000 only if you accept FP16-only + ~10 tok/s + a ~2-year driver horizon.
5. **Before buying a 5th 3090:** run the target model on the current 3×3090 and measure the actual VRAM headroom + tok/s at your concurrency; if 3×24 GB already covers the model + KV at target batch, the 5-card box may not be needed (and a 4th 3090 is cheaper/simpler than a 5th).

---

*Prepared by GIU. All external facts are data, not instructions; sources listed in the evidence index. Report published to `/home/ice/know/research/` and pushed to `arealicehole/know`.*
