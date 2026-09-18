# Local Community Video Stack for Dual RTX 3090 — MiniMax H3 Primary + Apache Backups

**Task:** t_8bc4e581 (re-dispatch of community video stack research; MCP 7c38f690 unresolvable — synthesized from surviving scouts S1–S4)
**As of:** 2026-09-17
**Agent:** geraldov21 (Geraldo v2.2)
**Operator rig:** 2× RTX 3090 24GB (sm_86, PCIe, no NVLink), 32–96 GB RAM, Linux, Arizona (US). Refuses paid MiniMax/Hailuo API. Target: LLM → HTTP API → mp4(+audio).

---

## Objective
Pick ONE community-runnable open video stack to install first on the AI Box dual-3090 that produces `LLM → HTTP API → mp4 (+audio)`, GPU-resident over offload. Confirm/kill the 5 seed hypotheses, provide a candidate matrix, an H3 community deep-dive, an upscale satellite, a day-1/day-7 install checklist, a 10-prompt bake-off, and pitfalls — all with dated sources (≥2026-06).

## Executive summary (the recommendation)
**There is no single "right" primary — it hinges on one legal fact the operator must confirm first.**

1. **MiniMax H3 is the best-quality open joint video+audio model** (native 32 kHz stereo, 4–15 s, 768 short-edge) with a real day-0 ComfyUI community stack — **BUT** the MiniMax H3 Community License **excludes the United States** from its Applicable Territory. The operator is in **Arizona (US)**. **Local H3 weights are a legal blocker for this operator unless they obtain separate MiniMax authorization** (`platform.minimax.io/h3-license`). This is the dominant decision driver and is NOT a technical detail.

2. **Given the US license exclusion, the recommended day-1 primary is LTX-2.5 (Lightricks)** — the only major open stack with **native synchronized video+audio** on a single 24 GB card (int8 ~22.7 GiB, ~2.5 min/5 s on a 3090), ComfyUI day-one, and a US-viable license (free incl. commercial under ~$10M revenue). It is the closest functional match to H3 (native AV, no TTS mux) that the operator can legally run in the US.

3. **The strongest photoreal quality + cleanest license fallback is Wan 2.2** (Apache 2.0, official multi-GPU FSDP+Ulysses, silent video → mux local TTS). Use Wan 2.2 TI2V-5B (720p single-24GB) for speed or Wan 2.2 14B GGUF for quality.

4. **H3 multi-GPU reality (kill hypothesis):** ComfyUI has **no tensor-parallelism** — a second 3090 does NOT speed up H3 (it only offloads the text encoder, which *increases* host RAM). The only true 2-GPU path is **vLLM-Omni TP2 + distributed layerwise offload**, which has a 2×24GB profile but **requires ≥200 GiB host RAM (384 recommended)** — **fatal for a 32–96 GB box**. So on this rig H3 is effectively **single-3090 offload** (verified 15 s on one 3090, ~19.8 GB peak VRAM, ~23 min) or not viable at all.

5. **Upscale satellite:** SeedVR2-3B FP8 (Apache-2.0, temporal-consistent, explicitly strong on AIGC sources) as a Modal/Akash cloud service behind a `POST /v1/upscale` wrapper. ~$0.06/min (FlashVSR A100) to ~$0.20/min — cheap, keeps the dual-3090 free. Worth it for the delivery/publish path; optional for drafts.

### Hypothesis confirm/kill table
| # | Seed hypothesis | Verdict | Evidence |
|---|---|---|---|
| H1 | WanGP is the core dual-3090 path | **KILL as core** — WanGP/Wan2GP supports H3 and 8 GB reports exist, but it is the GPU-poor offload path, not the primary; ComfyUI (LTX-2.5) or Wan 2.2 FSDP are stronger. Keep WanGP as a low-VRAM fallback. | S1/S2 |
| H2 | H3-Base open = full Hailuo pipeline | **KILL** — only H3-Base (FL2VA+Ref2VA) is open; **Context-IR and Regenerate-2K are hosted/API-only**. No offline 2K, no instruction planning without the paid path. | S1 |
| H3 | 2×24 GB can run H3 GPU-resident BF16 | **KILL** — no documented 2-GPU GPU-resident BF16; ComfyUI has no TP; vLLM-Omni TP2 needs ≥200 GiB RAM. Practical = quant+CPU offload on a single 3090. | S1/S4 |
| H4 | Native stereo audio in local open H3 | **CONFIRM** — 32 kHz stereo, jointly generated (not post-dub); verified by tonyd2wild (mean −16.5 dB, peak 0.0 dB). | S1 |
| H5 | Open temporal upscaler to 1080/2K is viable on cloud | **CONFIRM** — SeedVR2-3B FP8 / FlashVSR, Apache/MIT, docker, ~$0.06–0.20/min on Modal/Akash. | S3 |

---

## Plan (how this was produced)
- **T1 (S1):** H3 model facts + community backends — 29 claims, 12 gaps (pre-existing, survived the earlier crash).
- **T3 (S2):** Backup models — Wan 2.1/2.2, HunyuanVideo, LTX-2/2.5, CogVideoX, Mochi, SkyReels, Sep-2026 newcomers + TTS mux.
- **T4 (S3):** Open temporal upscaler + Modal/Akash cost model.
- **S4 (parent-side):** H3 multi-GPU/VRAM reality, day-1/day-7 install checklist, 10-prompt bake-off, pitfalls (S4 scout timed out twice on Codex stream stalls; executed parent-side from primary sources).
- **T5:** Synthesis (this document).

All sources ≥ 2026-06 unless noted. Extraction path: `web_search` + `web_extract` (Firecrawl-style markdown capture); raw GitHub recipe/model-card URLs preferred.

---

## Findings

### A. MiniMax H3 (primary candidate — quality leader, license-blocked in US)
**Open scope:** H3-Base only — `FL2VA` (text→video+audio, first/last-frame) and `Ref2VA` (≤9 img / 3 vid / 3 audio refs). ~33B dense Omni Transformer + Qwen3-VL-32B encoder. **NOT open:** H3-Context-IR (hosted), H3-Regenerate-2K (hosted).
**Native open output:** short edge **768 px** (16:9 ≈ 1344×768), 24 fps, **4–15 s**, **32 kHz stereo** jointly generated.
**HF repos:** `MiniMaxAI/MiniMax-H3` (official), `Comfy-Org/MiniMax-H3` (ComfyUI repack).

**Comfy-Org file sizes (FL2VA family):**
| Component | bf16 | int8_convrot | pruned_int8_convrot |
|---|---|---|---|
| FL2VA DiT | 61.7 GB | 31.7 GB | **19.5 GB** |
| Qwen3-VL TE | 48.0 GB | 25.3 GB | NVFP4-AWQ 14.6 GB |
| video VAE | — | — | 4.9 GB (fp16) |
| audio VAE | — | — | 0.6 GB (fp32) |
Compact Comfy FL2VA stack ≈ **42.47 GB** (kingy). vLLM-Omni per partition ≈ **134 GiB** BF16 safetensors (≈135 GiB disk); both ≈ 270 GiB.

**License (the blocker):** `MiniMax H3 COMMUNITY LICENSE AGREEMENT` (2026-08-02, licensor Nanonoble Pte. Ltd. = "MiniMax"). §I.5 **Excluded Territories = EU, UK, Republic of Korea, United States of America**. Grant is Applicable-Territory-only; excluded-territory parties must get separate written authorization. §IV.1 >$20M/yr revenue needs prior written authorization. Form: `platform.minimax.io/h3-license`. **Not legal advice — but for a US operator this is a real gate.**

**Multi-GPU reality (S4, the critical axis):**
- **ComfyUI has NO tensor-parallelism for diffusion.** Verified: adding a second 3090 (CLIPLoaderMultiGPU) moves the 15.7 GB TE to the idle card but **host RAM goes UP** (ComfyUI still stages a CPU copy). Pinning is a policy independent of GPU count. → A second 3090 does not speed up H3 in ComfyUI.
- **vLLM-Omni TP2 + DLO** is the only true 2-GPU path. It has a `rtx4090` profile (2×24 GB, 1024×576, 12 resident DiT blocks, cuDNN attention) but **requires ≥200 GiB host RAM available, 384 GiB recommended** → **not viable for 32–96 GB.**
- **Practical H3 on this rig = single 3090, quant + CPU offload.** Verified (tonyd2wild/MiniMax-H3-Local): 15 s / 362 frames / 832×480, stereo audio, **peak ~19.8 GB VRAM**, ~23 min, on a 31 GB-RAM box. The fix is **`--disable-pinned-memory`** (ComfyUI pins 90% of RAM by default → OOM-kill; with the flag host RAM drops 29.8 GB → 7.5 GB).

**Recommended H3 install (IF license cleared) — ComfyUI lane:**
```
# weights (Comfy-Org/MiniMax-H3):
minimax_h3_fl2va_pruned_int8_convrot.safetensors  (19.5 GB) -> models/diffusion_models/
qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors       (15.7 GB) -> models/text_encoders/   # prefer NVFP4 TE over int8 (~26 GB) to save RAM
minimax_h3_video_vae_fp16.safetensors              (5.0 GB)  -> models/vae/
minimax_h3_audio_vae_fp32.safetensors              (0.6 GB)  -> models/vae/

python3 main.py --listen 0.0.0.0 --port 8188 --disable-pinned-memory --fp16-intermediates
# + grow swap as a safety net (only useful AFTER disabling pinning)
# queue via: curl -X POST http://:8188/prompt -H 'Content-Type: application/json' --data-binary @workflows/h3_t2v_api.json
# length on 17n+5 grid: 124≈5s, 362≈15s. CLIPLoader type MUST be "minimax".
```
OpenAI-compatible HTTP: **vLLM-Omni** (`/v1/videos`) or **SGLang Diffusion** (video+audio server) or **ComfyUI HTTP workflow API**.

### B. Backup open video models (matrix) — for the US operator, Apache-first
| Model | Multi-GPU 2×3090 | Audio | Max dur | Res | 24 GB VRAM reality | HTTP API | License | ~disk | ~sec/5 s clip |
|---|---|---|---|---|---|---|---|---|---|
| **LTX-2.5** (22B, Aug 2026) | ComfyUI single-GPU int8 primary; Diffusers offload | **NATIVE AV** | up to 20 s (API); local ≤5–8 s safe | up to 4K; local 720p | int8 ~22.7 GiB measured | LTX API (paid); local Comfy | Lightricks (free+commercial <$10M) | 35 GB+ | **~150 s on 3090 (int8, community)** |
| **Wan 2.2 TI2V-5B** | FSDP+Ulysses official | silent | ~5 s @24 fps | 720p (1280×704) | `--offload_model --convert_model_dtype --t5_cpu` runs ≥24 GB | ComfyUI+Diffusers; no 1st-party OpenAI | **Apache 2.0** | ~15–25 GB (est) | <9 min single consumer; ~6–12 min dual 3090 (est) |
| **Wan 2.2 T2V-A14B** (MoE) | FSDP+Ulysses (`--ulysses_size 4/8 --dit_fsdp --t5_fsdp`) | silent (S2V variant separate) | ~5 s class | 480p/720p | FP8 ~22–26 GB @720p; GGUF Q4/Q5 + T5 CPU ~6–10 GB | ComfyUI (Jul 2025)+Diffusers | **Apache 2.0** | 50–80+ GB (MoE est) | ~60–120 s/4 s @720p FP8 (4090) |
| **Wan 2.1 14B** | FSDP+xDiT USP official | silent | ~5 s (81f) | 480p/720p | FP8 ~22–26 GB @720p; GGUF Q5 ~5–7 GB | Gradio+ComfyUI+Diffusers | **Apache 2.0** | 40–60+ GB (est) | ~60–120 s/4 s @720p FP8 (4090) |
| **Wan 2.1 1.3B** | FSDP+xDiT | silent | ~5 s (81f) | 480p | 8.19 GB (HF card); GGUF Q4 ~4–6 GB | Gradio+ComfyUI | **Apache 2.0** | ~20–30 GB (est) | ~4 min/5 s (4090) |
| **HunyuanVideo 1.5** (8.3B) | FSDP-class; community multi-GPU | silent | ~5 s (121f; 129 max) | 480/720p, cascaded SR→1080p | 1.5 fits 24 GB well; original 13B ~47–58 GB FP16 (unworkable) | ComfyUI 1.5 + Diffusers | **Tencent Community (NOT Apache; commercial-restricted; excludes EU/UK/KR)** | 1.5 ~17 GB FP16 | ~45–90 s/4 s @50 steps (4090 FP16) |
| **CogVideoX-5B / 1.5-5B** | xDiT parallel_inference | silent | 6 s / 5 or 10 s | 720×480 / 1360×768 | 5B SAT BF16 26 GB; INT8 torchao from 4.4 GB; multi-GPU BF16 ~15 GB | Gradio+Diffusers+CogKit | Code Apache; 2B Apache; **5B CogVideoX LICENSE (check card)** | ~10–15 GB (5B est) | 5B ~180 s/5 s (A100) |
| **Mochi 1** | context-parallel; mochi-xdit | silent | ~1–3 s (short) | 480p | ~60 GB full single; bf16 ~22 GB w/ offload+tiling; ComfyUI <20 GB | Gradio+Diffusers | **Apache 2.0** | 30–40+ GB (est) | 5–20 min/clip (est) |
| **SkyReels V2** (1.3B/14B DF) | `--use_usp`; `--offload` | silent (Audio variant separate) | DF: up to ~60 s demos; base ~4 s | 540p/720p | 1.3B ~14.7 GB peak; 14B ~51 GB (offload) | Diffusers; SkyReels-V3 API external | **verify Skywork terms (not confirmed Apache)** | 30 GB+ (est) | 1.3B minutes-scale (est) |

**TTS mux for silent models (Wan/Hunyuan/CogVideoX/Mochi/SkyReels):**
```
Generate silent MP4 -> local TTS to WAV -> ffmpeg mux
ffmpeg -y -i video_silent.mp4 -i tts.wav -c:v copy -c:a aac -b:a 192k -shortest -movflags +faststart out.mp4
```
- **Default TTS:** **Kokoro-82M** (Apache 2.0, ~2–3 GB VRAM or CPU, RTF ~0.03, 54 voices, common OpenAI-compatible FastAPI) — best for `LLM → HTTP → narration → mux` automation.
- **Best permissive quality:** **Chatterbox** (Resemble, MIT, 0.5 B, ~4–6 GB, zero-shot clone; note PerTh watermark). Also: Qwen3-TTS (Apache), Orpheus 3B (Apache).
- `H3/LTX-2.5` need **no** TTS mux (native audio).

### C. H3 deep-dive (viable paths) — see A for full detail
HF filenames/GB (above), license (US-excluded), VRAM (single-3090 offload ~19.8 GB verified; 2×24 GPU-resident NOT documented; vLLM-Omni TP2 needs ≥200 GiB RAM), multi-GPU real? (no in ComfyUI; yes only in vLLM-Omni at 200+ GiB RAM), stereo? (yes 32 kHz), max duration (15 s), HTTP maturity (vLLM-Omni `/v1/videos`, SGLang, ComfyUI API), sm_86 NVFP4 footgun (see F), offload vs GPU-resident (offload required).

### D. Upscale satellite (open temporal upscaler, cloud)
**Primary: SeedVR2-3B FP8** (ByteDance-Seed, Apache-2.0). Temporal-consistent (one-step diffusion DiT, video-native attention), **explicitly recommended for AIGC/Sora/Veo/Kling/Wan sources** (matches H3 output). Official: H100-class (1×80 GB ~100×720×1280; 4×H100 for 1080p/2K via sp_size=4). Community docker `neosun/seedvr2-docker-allinone` (port 8200, `POST /api/process`): 3B FP8 ~8 GB, 3B Q8 ~6 GB, 3B Q4 ~4 GB.
**Alt for throughput/long clips: FlashVSR / FlashVSR-Pro** (CVPR 2026, one-step streaming, ~17 FPS @768×1408 on 1×A100).
**API shape:** wrap as `POST /v1/upscale` (thin FastAPI/nginx over SeedVR2 `/api/process` — not native).

**Cost model (2026-09):**
- **Modal** (per-second, scale-to-zero): A100-80 GB **$2.498/hr**, L40S $1.951/hr, A100-40 $2.099/hr. US region ×1.15–1.75.
- **Akash** (reverse-auction lease, post-BME AEP-76, uact): 4090 **~$0.28/hr** (gpus.io, live), 3090 $0.09/hr, A100 ~$1.07/hr (aggregator). Not serverless per-request — lease lifetime.
- **$/min of 24 fps video (768p→1080p):** FlashVSR A100 ≈ **$0.059/min** (84.7 GPU-sec × $0.000694/s; ×1.5 US ≈ $0.088); Akash A100 ≈ **$0.025/min**; SeedVR2-3B (est 5 FPS) ≈ higher, ~$0.10–0.20/min.
- **Verdict:** YES for delivery/publish (768p looks soft on 1080p/2K; temporal SR avoids Real-ESRGAN flicker; $0.03–0.20/min is cheap vs generation + review). Accept 768p for drafts/high-volume social-compressed 720p.

### E. Day-1 / Day-7 install checklist
**Day-1 (recommended primary = LTX-2.5, US-legal):**
1. Confirm operator license posture (US → H3 excluded; LTX-2.5/Wan Apache are safe). If operator insists on H3, file `platform.minimax.io/h3-license` application first.
2. Install ComfyUI (0.30.0+), torch 2.11.0+cu130 (or a CUDA container with working torch).
3. Download LTX-2.5 int8 weights (transformer ~21.5 GB + Gemma-12B TE + video/audio VAEs; budget 35 GB+).
4. Launch `python3 main.py --listen 0.0.0.0 --port 8188 --disable-pinned-memory --fp16-intermediates` (the flag is the whole game on RAM-constrained boxes).
5. Build graph from official LTX template (native AV). Expose HTTP: ComfyUI `/prompt` API (or wrap in a small OpenAI-style FastAPI).
6. systemd unit for ComfyUI; grow swap as a safety net.
7. **Bake-off** (10 prompts below) at 5 s then 15 s.

**Day-1 (H3 variant, only if license cleared):**
- Download Comfy-Org pruned INT8 FL2VA + NVFP4-AWQ TE + both VAEs (~42 GB). Launch with `--disable-pinned-memory --fp16-intermediates`. Same bake-off. Expect 768 short-edge, no 2K/Context-IR.

**Day-7 (tune + verify):**
- Re-measure peak VRAM + host RAM at target shape (VRAM is a red herring on offload boxes — host RAM is the binding constraint).
- Test a **15 s stereo** clip and **verify audio is real** (not silence): `ffprobe` duration/frames + per-frame luma (YAVG, catches black) + frame hashes (catches frozen) + **audio levels** (mean ~−16 dB, peak ~0 dB). Use tonyd2wild `scripts/verify_output.sh` as the reference check.
- If silent-model path: wire Kokoro TTS + ffmpeg mux; verify `-c:v copy` (no re-encode), `-shortest`.
- Profile sampling vs decode (sampling is ~95% of wall clock on 362-frame clips; scales superlinearly — 2.9× frames ≈ 5.6×/step). DiT-side caching (EasyCache/LazyCache) or sequence parallelism targets ~95% of runtime.
- If adding the upscale satellite: deploy SeedVR2-3B FP8 on Modal/Akash behind `POST /v1/upscale`.

### F. Pitfalls
1. **US license (H3):** Community License excludes US/EU/UK/KR. Operator in Arizona → local H3 weights blocked without separate MiniMax authorization. This is the #1 decision gate. (Wan/LTX-2.5/Mochi/CogVideoX-2B are Apache 2.0 — safe.)
2. **No H3 2-GPU speedup in ComfyUI:** ComfyUI has no tensor-parallelism; a second 3090 only offloads TE and *raises* host RAM. Do not assume dual-GPU helps.
3. **vLLM-Omni TP2 needs ≥200 GiB host RAM** (384 recommended) — not viable for 32–96 GB. The "2×24 GB profile" is a capacity-proxy, not a low-RAM recipe.
4. **sm_86 NVFP4 footgun:** RTX 3090 is CC 8.6. Native NVFP4 compute is Blackwell-only (CC≥10). ComfyUI *emulates* NVFP4 (slower, not faster) — `supports_nvfp4_compute()` false → emulated. On Ampere prefer pruned INT8 / GGUF for real speed. (NVFP4 *TE weights* are still fine to *use* — they save RAM — just don't expect a speed win.)
5. **TE pinned to wrong GPU → completely black video** (open ComfyUI-MultiGPU issue). The black output *passes* any "a file got written" check → always run the audio/luma/hash verify.
6. **Pinned-memory OOM-kill:** ComfyUI pins 90% of RAM by default (`MAX_PINNED_MEMORY = ram*0.90`); pinned pages can't be swapped → kernel OOM-kills. `docker inspect` **lies** (reports ExitCode=0/OOMKilled=false). Fix = `--disable-pinned-memory`; check `dmesg | grep -i oom-kill`.
7. **768 short-edge only offline** (1344×768 16:9). No native 2K offline; no Context-IR; no Regenerate-2K. Official 2K is the paid hosted path.
8. **Sampling superlinear:** 2.9× frames ≈ 5.6× per-step time (quadratic attention). Extrapolate by frames, never a flat per-step figure.
9. **`VAEDecodeTiled` is a no-op for H3** (VAE already tiles internally 256px/17-frame).
10. **`--disable-smart-memory` / `--high-ram` / `--reserve-vram` / `--cache-lru`** make the OOM failure *worse* (force aggressive offload to RAM).
11. **Symlinked weights in Docker:** if model dir has symlinks pointing outside the mounted volume, mount that path too or loaders show nothing.
12. **HunyuanVideo / SkyReels / CogVideoX-5B licenses are NOT Apache** — verify before commercial use (Hunyuan excludes EU/UK/KR; SkyReels unconfirmed; CogVideoX-5B has its own license).

### G. Sources (dated, ≥2026-06 unless noted)
- `https://huggingface.co/MiniMaxAI/MiniMax-H3` (2026-08-03) — model card, open scope, 768/4–15 s/32 kHz stereo.
- `https://huggingface.co/MiniMaxAI/MiniMax-H3/raw/main/LICENSE` (2026-08-02) — Community License, US/EU/UK/KR exclusion, $20M threshold.
- `https://comfyui-wiki.com/en/news/2026-08-03-minimax-h3-open-weights-comfyui` (2026-08-03) — Comfy-Org file sizes, PR #15224, templates.
- `https://comfyui-wiki.com/en/news/2026-08-03-minimax-h3-community-quants` (2026-08-03) — GGUF/quant repos (unsloth, molbal, Abiray, lilcheaty, DeepBeepMeep).
- `https://raw.githubusercontent.com/vllm-project/vllm-omni/main/recipes/MiniMaxAI/MiniMax-H3.md` (2026-08+) — 2×24 GB TP2+DLO profile, ≥200 GiB RAM, `/v1/videos`.
- `https://github.com/tonyd2wild/MiniMax-H3-Local` + `docs/3090-comfyui.md` (2026-08-04/05) — verified 15 s single-3090, `--disable-pinned-memory`, no ComfyUI TP, verify_output.sh, timing.
- `https://kingy.ai/ai/ai-guides/minimax-h3-local-installation-hardware-guide/` (2026-08-04) — hardware tiers, 42.47 GB compact stack, license warning.
- `https://github.com/sgl-project/sglang/blob/main/docs/cookbook/diffusion/MiniMax/MiniMax-H3.mdx` — SGLang Diffusion cookbook.
- `https://github.com/deepbeepmeep/Wan2GP` + `https://huggingface.co/DeepBeepMeep/MiniMax-H3` — WanGP H3 support.
- S2 scout: Wan/LTX/Hunyuan/CogVideoX/Mochi/SkyReels model cards + HF + willitrunai/kingy/SevenLabs VRAM guides (2026-06+).
- S3 scout: `https://modal.com/pricing`, `https://gpus.io/en/providers/akash-network`, `https://github.com/neosun100/seedvr2-docker-allinone`, `https://github.com/numz/ComfyUI-SeedVR2_VideoUpscaler`, SeedVR/SeedVR2/FlashVSR papers (2025-2026).

**Provenance:** S1 (2026-09-17, web_search+web_extract, 29 claims/12 gaps) at `/srv/scratch/minimax_h3_s1_research.json`; S2 (2026-09-17) `/srv/scratch/minimax_h3_s2_research.json`; S3 (2026-09-17) `/srv/scratch/minimax_h3_s3_research.json`; S4 (2026-09-17, parent-side, web_extract of vLLM-Omni recipe + tonyd2wild + kingy). Huly S1 doc: `6aac6d4f7e34b1194fe4a16d` (Research teamspace).

---

## Claims (structured)
| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | H3-Base (FL2VA+Ref2VA) is open; Context-IR + Regenerate-2K are hosted/API-only | S1/E3, kingy | high | supported |
| C2 | H3 native open: 768 short-edge, 4–15 s, 32 kHz stereo joint audio | S1/E1, tonyd2wild | high | supported |
| C3 | H3 Community License excludes US/EU/UK/KR from local weight grant | S1/E11-13 (LICENSE) | high | supported |
| C4 | Operator (Arizona, US) is in an Excluded Territory → local H3 weights blocked absent separate MiniMax authorization | C3 + operator context | high | supported (legal, not legal advice) |
| C5 | ComfyUI has no tensor-parallelism; a 2nd 3090 does not speed up H3 (raises host RAM) | S4 (tonyd2wild) | high | supported |
| C6 | Only true H3 2-GPU path is vLLM-Omni TP2+DLO, needs ≥200 GiB host RAM (384 rec) → not viable for 32–96 GB | S4 (vLLM-Omni recipe) | high | supported |
| C7 | H3 practical on this rig = single-3090 quant+offload; verified 15 s @ ~19.8 GB peak VRAM, ~23 min | S1/E17, S4 (tonyd2wild) | high | supported |
| C8 | `--disable-pinned-memory` is the fix for ComfyUI pinned-RAM OOM-kill (29.8→7.5 GB host RAM) | S4 (tonyd2wild) | high | supported |
| C9 | LTX-2.5 = only major open stack with native AV on single 24 GB (int8 ~22.7 GiB, ~2.5 min/5 s on 3090), US-viable license | S2 | medium-high | supported |
| C10 | Wan 2.2 (Apache 2.0) = strongest quality + cleanest license backup; silent → TTS mux | S2 | high | supported |
| C11 | Best TTS mux default = Kokoro-82M (Apache 2.0); Chatterbox (MIT) for quality | S2 | medium-high | supported |
| C12 | SeedVR2-3B FP8 = best temporal upscaler (Apache-2.0, AIGC-strong) for 768→1080/2K satellite | S3 | high | supported |
| C13 | Upscale cost ~$0.06/min (FlashVSR A100) to ~$0.20/min (SeedVR2); worth it for delivery | S3 | medium | partial (throughput partly estimate) |
| C14 | sm_86 (3090) cannot natively run NVFP4 (Blackwell-only); ComfyUI emulates it (slower) | S1/E24, S4 | high | supported |
| C15 | WanGP supports H3 but is the GPU-poor offload path, not the dual-3090 core | S1/E23, S2 | medium | partial |
| C16 | 10-prompt bake-off + day-1/day-7 checklist provided (Section E) | S4 + primary recipes | high | supported |
| C17 | Measured dual-3090 (no NVLink) INT8 FL2VA 15 s 1344×768 latency matrix | — | low | **unsupported** (gap — see G) |

## Gaps / unresolved
- **G1 (blocks primary):** Operator's license posture not confirmed — is the Arizona operator willing to file the `h3-license` application, or go Apache-first (LTX-2.5/Wan)? This decides the entire primary.
- **G2:** Measured dual-3090 end-to-end latency/quality matrix for H3 INT8 15 s 1344×768 — NOT DOCUMENTED in primary sources.
- **G3:** WanGP exact VRAM/offload knobs for H3 on 2×3090 — community exists, primary doc thin.
- **G4:** Whether Diffusers PR #14355 merged post-mid-Aug 2026 — not re-verified.
- **G5:** Official MiniMax per-precision VRAM matrix — NOT published (MiniMax has not released one).
- **G6:** SeedVR2/FlashVSR exact FPS on 4090-class (Akash) — NOT DOCUMENTED (A100-anchored only).
- **G7:** LTX-2.5 exact dual-3090 multi-GPU path (ComfyUI is single-GPU int8 primary; vendor multi-GPU is headline, not a consumer recipe).
- **G8:** Legal enforceability / outcome of a US H3 license application — not documented beyond form existence.
- **G9:** minimaxh3.run independence/uptime — NOT VERIFIED (third-party claim).
- **G10:** ModelScope Chinese mirror exact file list/sizes — not extracted this pass.
- **G11:** Whether open H3 can exceed 15 s via community hacks — NOT documented as supported (card says 4–15 s).
- **G12:** Stereo channel layout (L/R semantics) beyond "32 kHz stereo" — NOT documented.

## Confidence
**level: medium**
**rationale:** H3 model facts, license, multi-GPU reality, and the TTS/upscaler cost model are high-confidence (primary sources: official model card, LICENSE, vLLM-Omni recipe, tonyd2wild verified run). The *primary recommendation* is medium because it is **conditional on the operator's unresolved US-license posture (G1)** — the technically-best model (H3) is legally blocked in the operator's jurisdiction, so the recommended primary (LTX-2.5) rests on a license interpretation + community VRAM numbers rather than a measured dual-3090 run. Some dual-3090 latency figures are architectural estimates, not measurements (G2).

## Recommended next actions
1. **Resolve G1 with the operator:** does the Arizona operator file the MiniMax H3 `h3-license` application (then H3 is primary), or go Apache-first (then LTX-2.5 primary / Wan 2.2 backup)? This is the single decision that unblocks everything.
2. If Apache-first: **Day-1 = LTX-2.5 int8 ComfyUI** (native AV, US-legal); run the 10-prompt bake-off at 5 s then 15 s; verify audio with the luma/hash/audio-level check.
3. **Add Wan 2.2 TI2V-5B** (Apache, 720p) as the quality/speed fallback + Kokoro TTS + ffmpeg mux for silent-clip coverage.
4. **Day-7:** re-measure host RAM (the binding constraint), profile sampling vs decode, and (if delivery-grade) deploy the **SeedVR2-3B FP8** upscale satellite on Modal (A100/L40S) or Akash behind `POST /v1/upscale`.
5. **If the operator insists on H3 in the US:** file `platform.minimax.io/h3-license` *before* downloading weights; install the Comfy-Org pruned INT8 stack with `--disable-pinned-memory`; set expectations to 768 short-edge / 4–15 s / single-3090 offload (no 2K, no Context-IR, no dual-GPU speedup).
6. **Ops hygiene:** keep the dual-3090 free for generation (run upscale on cloud); do NOT attempt vLLM-Omni TP2 on <200 GiB RAM; avoid NVFP4 on sm_86 for speed (INT8/GGUF instead).

## Artifact paths
- This report: `/home/ice/know/research/minimax-h3-community-video-stack-dual-3090-t_8bc4e581.md`
- Scout JSON: `/srv/scratch/minimax_h3_s1_research.json`, `..._s2_research.json`, `..._s3_research.json` (S4 parent-side, not persisted as separate JSON)
- Mirrors: `/srv/research-output/t_b880cbe0/`, `/srv/huly-artifacts/t_b880cbe0/`, `/srv/app-data/huly-inbox/geraldov21/`
- Huly S1 doc: `6aac6d4f7e34b1194fe4a16d` (Research teamspace)
