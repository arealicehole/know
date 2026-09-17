# Local Open-Weight Image Generation Stack — Dual RTX 3090 (Sep 2026)

**Task slug:** `local-img-gen-dual-3090-kie-replacement`  
**Date (UTC):** 2026-09-17  
**Hardware target:** 2× RTX 3090 24GB (Ampere sm_86), 32GB system RAM, Linux, exclusive creative mode (no concurrent LLM)  
**Goal:** Replace paid Kie cloud API with local programmatic txt2img + img2img/edit + ControlNet-class control; ComfyUI preferred; privacy-centric / open-source preference.

---

## Objective

Evaluate and compare open-weight image models usable on dual 3090 for a day-1 and day-7 install path covering generation, edit, and structural control, with license clarity (BFL non-commercial vs Apache-only), VRAM/quant notes, ComfyUI support, multi-GPU pattern, throughput estimates, and Ampere pitfalls.

## Summary

**Apache-first day-1 stack (recommended for privacy + commercial-clean path):**

1. **ComfyUI** as the sole runtime + HTTP `/prompt` API façade (Kie replacement).
2. **FLUX.2 [klein] 4B** (Apache 2.0) — operator already has GGUF; unified txt2img + multi-ref edit; ~13GB VRAM; sub-second–few-second interactive. Primary edit/gen workhorse.
3. **Z-Image-Turbo 6B** (Apache 2.0, Tongyi-MAI) — fast photoreal + bilingual text; BF16 fits one 3090 with headroom; 8-step turbo.
4. **Two ComfyUI workers** (`CUDA_VISIBLE_DEVICES=0|1`, ports 8188/8189) rather than single-process multi-GPU model parallel — max throughput, isolation, simpler recovery.

**Day-7 quality/control expansion:**

5. **Qwen-Image-2512 + Qwen-Image-Edit (Apache 2.0)** — strongest open text rendering / instruction edit; FP8 or GGUF on 24GB; Lightning LoRA for 4-step.
6. **SDXL + ControlNet + IP-Adapter** — mature structural control (Canny/Depth/OpenPose/etc.) ecosystem; OpenRAIL++.
7. Optional quality/aesthetic: **Krea-2 Turbo** (open weights exist; **community license, not Apache** — commercial may need paid path).
8. Optional: **FLUX.1 [schnell]** (Apache) for mature FLUX.1 LoRA ecosystem without BFL commercial fee.
9. **Avoid production use of FLUX.1 [dev]/Fill/Kontext/Canny/Depth and FLUX.2 [dev]** without a paid BFL commercial license — weights are gated non-commercial.

**BFL license fork:** If commercial deliverables matter, treat FLUX.1-dev family and FLUX.2-dev as R&D-only unless licensed. Prefer Apache: FLUX.2-klein-4B, FLUX.1-schnell, Qwen-Image*, Z-Image-Turbo.

**Multi-GPU:** Prefer **two independent workers** for API throughput. Use ComfyUI-MultiGPU only when a single workflow needs TE/VAE offload onto the second card (large Qwen BF16 path). 32GB system RAM is the real bottleneck when both workers load heavy text encoders.

---

## A. Model shortlist

| Model | HF / source | Params (HF safetensors) | License commercial-ok? | Role | ComfyUI | 24GB fit |
|---|---|---|---|---|---|---|
| **FLUX.2-klein-4B** | `black-forest-labs/FLUX.2-klein-4B`, GGUF `unsloth/FLUX.2-klein-4B-GGUF` | ~3.88B | **Yes — Apache 2.0** | Day-1 gen+edit+multi-ref | Native | Yes (~13GB; GGUF lower) |
| **Z-Image-Turbo** | `Tongyi-MAI/Z-Image-Turbo`, Comfy `Comfy-Org/z_image_turbo` | ~6.15B | **Yes — Apache 2.0** | Day-1 fast t2i | Native | Yes BF16 ~14–16GB; FP8 ~8GB |
| **Qwen-Image-2512** | `Qwen/Qwen-Image-2512` | ~20.43B | **Yes — Apache 2.0** | Day-7 quality t2i + text | Native docs | FP8 ~16–21GB; BF16 needs offload/2×GPU; GGUF Q4 ~8–13GB |
| **Qwen-Image-Edit** (+2511 snapshots) | `Qwen/Qwen-Image-Edit` | ~20.43B | **Yes — Apache 2.0** | Day-7 instruction edit | Native templates | FP8 ~20.5GB; GGUF Q4_K_M ~13.2GB |
| **FLUX.1-schnell** | `black-forest-labs/FLUX.1-schnell` | ~11.89B | **Yes — Apache 2.0** | Fast FLUX.1 commercial | Native | FP8/GGUF comfortable; FP16 tight |
| **FLUX.1-dev** | `black-forest-labs/FLUX.1-dev` | ~12B class | **No free commercial** — NC license | Quality R&D | Native | FP8 ~12–17GB; FP16 ~24GB |
| **FLUX.1-Fill / Kontext / Canny / Depth** | BFL `FLUX.1-*-dev` | ~11.9B class | **No free commercial** — same NC family | Fill/edit/control R&D | Native | Similar to dev + control overhead |
| **FLUX.2-dev** | `black-forest-labs/FLUX.2-dev` | ~32.2B | **No free commercial** (gated other) | High-end R&D | Native | Wants 24GB+; huge download |
| **SDXL base** | `stabilityai/stable-diffusion-xl-base-1.0` | ~3.5B UNet class | **Yes — OpenRAIL++** (use restrictions, no $1M revenue cap) | Control ecosystem | Mature | ~6.5–10GB + CN |
| **SD 3.5 Medium/Large** | `stabilityai/stable-diffusion-3.5-*` | 2.47B / 8.15B | **Conditional** — Community License free commercial **under $1M revenue** | Alt t2i + official CNs | Native | Med easy; Large ~18GB FP16 |
| **Krea-2 Raw/Turbo** | `krea/Krea-2-Raw`, `krea/Krea-2-Turbo` | ~12.82B | **Open weights, community license** — commercial may need `opensource@krea.ai` | Aesthetic t2i | Official ComfyUI | Turbo 8-step; BF16 heavy |

### Per-model notes

#### A1. Qwen-Image / 2512 / Edit (Apache 2.0)
- **Repos:** `Qwen/Qwen-Image`, `Qwen/Qwen-Image-2512`, `Qwen/Qwen-Image-Edit` (and community Edit-2511).
- **Architecture:** 20B MMDiT; strong complex text (EN/ZH); Edit feeds image to Qwen2.5-VL + VAE for semantic + appearance edit.
- **Download size:** Full HF trees ~**57.7 GB** each for 2512 and Edit (transformer shards + large TE).
- **VRAM (community-measured, Qwen does not publish official VRAM):**
  - BF16 full ~**40.9 GB** (needs offload or multi-GPU / does not fit single 24GB fully resident).
  - FP8 ~**16–20.5 GB** → fits one 3090 with care.
  - GGUF Q4_K_M ~**13.1–13.2 GB**; Q4_0 ~**11.9 GB**.
  - Lightning LoRA (~850MB) can cut steps 40→4.
- **ComfyUI:** Official tutorials for Qwen-Image-2512 and Edit-2511 templates.
- **Control:** Instruction edit / mask / multi-image; not classic ControlNet-first (use SDXL CN or FLUX control for edges/depth).

#### A2. Z-Image-Turbo (Tongyi-MAI, Apache 2.0)
- **Repo:** `Tongyi-MAI/Z-Image-Turbo`; Comfy pack `Comfy-Org/z_image_turbo`.
- **6B** single-stream DiT; distilled **8 NFE**; guidance_scale=0 for turbo.
- **VRAM:** BF16 ~**14–16 GB**; FP8 ~**8 GB**; GGUF down to ~6 GB class.
- **Speed:** ~**2–3 s / 1024²** on RTX 4090 (scale ~0.7–0.9× on 3090 → ~3–5 s estimate).
- **Edit:** Z-Image-Edit listed as *to be released* on model card at research time — do not block day-1 on it.
- **Download:** Full tree ~32.8 GB; Comfy split bf16 diffusion alone ~12.3 GB + TE.

#### A3. FLUX.2 klein 4B (Apache 2.0) ★ day-1 primary
- **Repo:** `black-forest-labs/FLUX.2-klein-4B`; GGUF `unsloth/FLUX.2-klein-4B-GGUF`.
- **Unified** text-to-image + image-to-image **multi-reference editing**.
- **~13 GB VRAM**; 4-step distilled; consumer 3090/4070 class explicitly called out by BFL.
- **License:** Apache 2.0 — commercial OK.
- **Operator asset:** already has FLUX.2-klein GGUF on another host → copy first.
- **Note:** FLUX.2 klein **9B** is non-commercial; stick to **4B**.

#### A4. FLUX.2 dev
- **~32.2B** params; gated; license **other** (non-Apache); huge tree **~178 GB**.
- Community guides: wants **24GB+**; quant/GGUF paths exist but quality/ops heavier.
- **Commercial:** treat as needing BFL commercial terms unless counsel says otherwise.
- Lower priority vs klein-4B + Qwen for this operator.

#### A5. FLUX.1 family
| Variant | License | Use |
|---|---|---|
| **schnell** | Apache 2.0 | Commercial-clean FLUX.1 speed path |
| **dev** | FLUX.1 Non-Commercial | R&D / personal only without paid BFL |
| **Fill-dev** | NC (same family) | Inpaint/outpaint; operator already has weights |
| **Kontext-dev** | NC | Instruction/contextual edit |
| **Canny-dev / Depth-dev** | NC | Structural control (not classic SD ControlNet adapters; BFL control models) |

- **VRAM (FLUX.1 dev/schnell class):** FP16 ~**24 GB**; FP8 ~**12–17 GB**; GGUF Q4 ~**6–8 GB**. Control adapters add ~few GB.
- Fill already on-hand → OK for **non-commercial / internal experiments**; not for client production without BFL commercial license.

#### A6. SDXL ControlNet / IP-Adapter
- **Base:** `stabilityai/stable-diffusion-xl-base-1.0` — **OpenRAIL++**.
- **VRAM:** base ~**6.5–10 GB**; + ControlNet/IP-Adapter still easy on 24GB.
- **Ecosystem:** richest open ControlNet (Canny, Depth, OpenPose, Lineart, Tile…) + IP-Adapter face/style.
- **Role:** day-7 **control plane** when Qwen/FLUX.2-klein lack edge-conditioned layout fidelity.

#### A7. SD 3.5
- Medium ~2.5B / Large ~8.1B; official ControlNets (Canny/Depth/Blur for Large).
- **License:** Stability **Community License** — free commercial if org **<$1M annual revenue**; else Enterprise.
- VRAM: Medium consumer-friendly (~10 GB class excl. some TE accounting); Large ~**18 GB FP16**.
- Prefer only if revenue gate is OK and you want Stability CNs; else SDXL + Apache models cleaner.

#### A8. Krea-2
- **Local weights exist:** `krea/Krea-2-Raw`, `krea/Krea-2-Turbo` (~12.8B).
- GitHub `krea-ai/krea-2`; ComfyUI listed first-class.
- Turbo: 8 steps, up to ~2K; Raw for LoRA train → apply on Turbo.
- **License:** community license with **permissive use** language but **not Apache**; commercial → contact Krea. **Not pure Apache-only path.**
- Download Turbo tree ~**62 GB** with TE/transformer.

---

## B. Stack (runtime)

### Preferred architecture
```
[Client / Hermes / scripts]
        |  HTTP JSON
        v
  load balancer / sticky router (optional)
   /              \
v8188 worker0    v8189 worker1
 CUDA:0           CUDA:1
 ComfyUI          ComfyUI
 shared models on disk (read-only NFS/local)
```

### Components
| Layer | Choice | Why |
|---|---|---|
| UI/engine | **ComfyUI** (latest stable) | Native workflows for Qwen, Z-Image, FLUX.2, Krea-2; `/prompt` API |
| Quant | **GGUF** (ComfyUI-GGUF) + **FP8** safetensors | 24GB headroom; operator already GGUF-fluent |
| Fast path | Z-Image-Turbo BF16/FP8 + FLUX.2-klein | Interactive |
| Quality path | Qwen-Image-2512 FP8 + Lightning | Text + realism |
| Edit path | FLUX.2-klein multi-ref + Qwen-Image-Edit GGUF/FP8 | Apache-clean |
| Control path | SDXL ControlNet + IP-Adapter; optional FLUX.1 Canny/Depth **if BFL licensed** | Structural |
| API shim | Thin FastAPI/Flask wrapping Comfy `/prompt` + WS progress | Drop-in vs Kie |

### Dual-3090 pattern: **two workers > one process**
| Pattern | Pros | Cons | Verdict |
|---|---|---|---|
| **2× ComfyUI instances** | 2 concurrent jobs; crash isolation; simple CUDA_VISIBLE_DEVICES | 2× model RAM if both load same heavy TE; 32GB host RAM tight | **Default** |
| **1× Comfy + MultiGPU nodes** | TE on GPU1, DiT on GPU0 for oversized models | Sequential steps; complexity; less API concurrency | Use for Qwen BF16 only |
| **1× model parallel** | Rare true speedup | Poor Comfy support; NVLink not on 3090 consumer | Avoid |

Launch sketch:
```bash
# GPU0
CUDA_VISIBLE_DEVICES=0 python main.py --listen 0.0.0.0 --port 8188 --cuda-device 0
# GPU1
CUDA_VISIBLE_DEVICES=1 python main.py --listen 0.0.0.0 --port 8189 --cuda-device 0
```
Share `models/` via bind mount; pin workflows per specialty (worker0=turbo gen, worker1=edit) to avoid thrashing.

### Throughput estimates (order-of-magnitude, 1024², exclusive GPUs)
| Workload | Single 3090 | Dual workers |
|---|---|---|
| Z-Image-Turbo 8-step | ~3–5 s → **~12–20 img/min** | **~24–40 img/min** |
| FLUX.2-klein 4-step | ~1–4 s interactive | ~2× |
| FLUX.1 FP8 ~20–28 step | ~15–35 s | ~2× |
| Qwen-Image FP8 50-step | ~30–90 s (Lightning 4-step much faster) | ~2× if both fit |
| SDXL + CN 25-step | ~5–12 s | ~2× |

*Estimates synthesized from published 4090 timings and typical Ampere derate; verify on-box.*

### System RAM (32GB) constraint
- Avoid loading two full BF16 Qwen TEs simultaneously.
- Prefer FP8/GGUF; enable Comfy model management / partial unload.
- Do not run LLM stack concurrently (operator already exclusive creative mode).

---

## C. Control matrix

| Capability | Best Apache / clean path | Alt (license caveats) | Notes |
|---|---|---|---|
| txt2img quality | Qwen-Image-2512; Z-Image-Turbo | FLUX.1-dev NC; FLUX.2-dev; Krea-2 community | Qwen leads text-in-image |
| txt2img speed | Z-Image-Turbo; FLUX.2-klein | FLUX.1-schnell Apache | Day-1 interactive |
| img2img / instruction edit | **FLUX.2-klein** multi-ref; **Qwen-Image-Edit** | FLUX.1-Kontext NC | Apache pair covers edit |
| Inpaint/outpaint | Qwen-Edit + masks; SDXL inpaint | **FLUX.1-Fill** NC (already owned) | Use Fill only if license OK |
| Edge/pose/depth control | **SDXL ControlNet** (+IP-Adapter) | FLUX.1 Canny/Depth NC; SD3.5 CN ($1M) | SDXL = day-7 control default |
| Style/ID reference | IP-Adapter on SDXL; FLUX.2-klein multi-ref | — | |
| Bilingual text glyphs | Qwen-Image*; Z-Image-Turbo | — | Qwen strongest |
| Commercial-clean default | Qwen*, Z-Image*, FLUX.2-klein, FLUX.1-schnell, SDXL | — | |

---

## D. API integration (Kie replacement)

1. Run dual ComfyUI with CORS/`--listen`.
2. Export workflows as **API-format JSON** (Comfy “Save (API format)”).
3. Client flow:
   - `POST /prompt` with `{ "prompt": <workflow>, "client_id": ... }`
   - Poll `GET /history/{prompt_id}` or WebSocket `/ws`
   - Fetch `GET /view?filename=...`
4. Thin gateway maps Kie-like requests → workflow templates:
   - `txt2img` → Z-Image or klein or Qwen template
   - `img2img/edit` → klein multi-ref or Qwen-Edit
   - `control` → SDXL CN template (preprocess Canny/Depth in nodes)
5. Queue strategy: round-robin 8188/8189; sticky session for multi-step edits.
6. Optional: `comfy-cli`, Runflow-style wrappers, or simple Python `httpx` client.

Sources: ComfyUI server routes docs; community API guides (POST `/prompt`).

---

## E. Cost / license

| Path | Models | $ software | Commercial risk |
|---|---|---|---|
| **Apache-only (recommended)** | FLUX.2-klein-4B, Z-Image-Turbo, Qwen-Image*, FLUX.1-schnell, (SDXL OpenRAIL++) | $0 | Low; OpenRAIL++ has use-policy clauses (not revenue cap) |
| **BFL NC + paid commercial** | FLUX.1-dev/Fill/Kontext/Canny/Depth, FLUX.2-dev | BFL self-serve commercial | High if used unpaid for client work |
| **Stability Community** | SD 3.5 | $0 if **<$1M** revenue else Enterprise | Medium — track revenue gate |
| **Krea community** | Krea-2 | Possibly free community; commercial contact | Medium — read current license page |

**BFL FLUX.1 Non-Commercial** explicitly includes Fill, Depth, Canny, Redux, Kontext under NC; commercial requires license from BFL. Outputs generally usable, but **using the model** for revenue-generating / production end-user impact is restricted.

**Operator already holds FLUX.1-Fill + FLUX.2-klein GGUF:** keep klein on production path; quarantine Fill for non-commercial unless BFL license purchased.

Electricity/hardware: dual 3090 already owned → **$0/image** marginal vs Kie API fees.

---

## F. Pitfalls (sm_86 3090 / ops)

1. **Blackwell-only kernels:** Skip NVFP4 / RTX 50-only wheels and “CUDA 12.8 Blackwell ready” binaries that lack sm_86. Prefer stock PyTorch cu12.x builds with Ampere kernels; FlashAttention2 / SDPA OK; avoid FA3 if not built for Ampere.
2. **32GB host RAM:** Dual workers + two large TEs → swap death. Pin one “heavy” model type per GPU; use FP8/GGUF.
3. **License confusion:** Downloadable ≠ commercial-ok (FLUX.1-dev family, FLUX.2-dev, SD3.5 over $1M, Krea community).
4. **Qwen BF16 on 24GB:** Will not fully resident; use FP8/GGUF or MultiGPU offload.
5. **Z-Image-Edit not released** at card time — don’t plan control/edit solely on it.
6. **FLUX.2-dev size:** ~178GB tree; poor day-1 ROI vs klein-4B.
7. **Text encoder double-count:** Published “model VRAM” often excludes TE — budget extra 4–10GB.
8. **Comfy custom nodes drift:** Pin ComfyUI commit + node versions; native nodes preferred for Qwen/Z-Image/FLUX.2.
9. **PCIe dual-GPU without NVLink:** fine for 2-worker; bad for tensor parallel expectations.
10. **OpenRAIL++ / filters:** SDXL still has behavioral use restrictions — review LICENSE for prohibited uses.
11. **Gated HF repos:** BFL dev + Stability need HF token / accept terms before CI download.
12. **Ampere BF16:** 3090 supports BF16 tensor cores adequately; FP16 also fine — match checkpoint dtype.

---

## G. Checklist

### Day-1 (same day usable API)
- [ ] Fresh Linux user/venv or container; NVIDIA driver ≥ 535/550; CUDA toolkit matching torch
- [ ] Install **ComfyUI** + ComfyUI-Manager; enable API
- [ ] Copy **FLUX.2-klein GGUF** from other host → `models/unet` or GGUF folder; download matching VAE/TE if needed
- [ ] `hf download Tongyi-MAI/Z-Image-Turbo` or Comfy-Org split **bf16/fp8**
- [ ] Import official Z-Image-Turbo + FLUX.2-klein workflow templates; smoke-test UI
- [ ] Launch **two workers** 8188/8189 with `CUDA_VISIBLE_DEVICES`
- [ ] Write 20-line Python client: queue prompt, wait, save PNG
- [ ] Benchmark 20 images each model; record VRAM (`nvidia-smi`) and sec/img
- [ ] Document Apache-only policy; do **not** wire FLUX.1-Fill into production routes yet

### Day-7 (quality + control + edit parity with cloud)
- [ ] Qwen-Image-2512 **FP8** or GGUF + Lightning LoRA; native Comfy tutorial workflow
- [ ] Qwen-Image-Edit (2511 if standardized) GGUF/FP8 edit template
- [ ] SDXL base + refiner optional + **ControlNet Canny/Depth/OpenPose** + **IP-Adapter**
- [ ] Optional FLUX.1-schnell Apache for LoRA ecosystem
- [ ] Optional Krea-2 Turbo **after** license read
- [ ] Gateway: map endpoints txt2img/img2img/control; auth; job store; disk quota
- [ ] Model lifecycle: unload idle graphs; per-worker specialty to spare 32GB RAM
- [ ] Regression set: text posters, faces, hands, CN pose lock, multi-ref product edit
- [ ] Decide BFL commercial purchase **only if** Fill/Kontext/dev quality is mandatory
- [ ] Backup workflows JSON + pin hashes of weight files

### Explicit non-goals day-1
- FLUX.2-dev full download
- Concurrent LLM on same GPUs
- Blackwell-only quant packs

---

## Claims (verifier-oriented)

| ID | Statement | Status | Confidence | Evidence |
|---|---|---|---|---|
| C1 | Qwen-Image-2512 and Qwen-Image-Edit are Apache-2.0 on HF | supported | high | HF README frontmatter license: apache-2.0 |
| C2 | Z-Image-Turbo is Apache-2.0 6B turbo 8-step; BF16 ~14–16GB | supported | high | Tongyi README + localaimaster |
| C3 | FLUX.2-klein-4B is Apache-2.0, ~13GB VRAM, unified gen+edit | supported | high | BFL HF card |
| C4 | FLUX.1-dev/Fill/Kontext/Canny/Depth are Non-Commercial without paid BFL | supported | high | LICENSE-FLUX1-dev text |
| C5 | FLUX.1-schnell is Apache-2.0 | supported | high | HF API license apache-2.0 |
| C6 | Krea-2 open weights exist (Raw/Turbo) with community not Apache license | supported | high | krea-ai/krea-2 README |
| C7 | SD 3.5 Community License free commercial under $1M revenue | supported | high | stability.ai/license summaries |
| C8 | SDXL is OpenRAIL++ with mature ControlNet/IP-Adapter | supported | high | HF SDXL card + ecosystem knowledge cited via digitalapplied |
| C9 | Dual ComfyUI workers is preferred multi-GPU throughput pattern | partial | medium | Comfy discussions + dual-GPU guides |
| C10 | Qwen FP8 ~20.5GB / BF16 ~40.9GB community figures | partial | medium | localaimaster (not vendor-official VRAM) |
| C11 | FLUX.2-dev ~32B params, gated, very large download | supported | high | HF API params 32.2B; tree ~178GB |
| C12 | NVFP4 / Blackwell-only paths should be avoided on sm_86 | partial | medium | community guides note Blackwell NVFP4; Ampere needs standard kernels |

---

## Gaps
- Exact on-box 3090 ms/img not measured in this research run (no local GPUs in research container).
- FLUX.2-dev exact commercial license text not fully retrieved (gated 401 on raw README).
- Krea community license full legal text not mirrored — operator must read https://www.krea.ai/krea-2-licensing before commercial use.
- Z-Image-Edit release status may change after this snapshot.
- Qwen-Image-Edit-2511 vs base Edit naming — community standardized on 2511; confirm exact HF repo at install time.
- ComfyUI MultiGPU_WorkUnits “same GPU type” constraint — verify current docs version.
- Firecrawl extract 403 forced curl/HF API path — some blog secondary claims less triangulated.

## Confidence
- **level:** medium-high on licenses/repos/architecture; **medium** on absolute VRAM/throughput numbers (community benches + HF sizes).
- **rationale:** Primary HF cards + license files + HF API param/size trees verified; VRAM/speed from secondary guides cross-checked but not re-benchmed on 3090.

## Recommended next actions
1. Day-1 install checklist on the dual-3090 host; capture `nvidia-smi` logs.
2. Legal skim: Apache-only policy vs buy BFL commercial for Fill/Kontext.
3. Implement Comfy gateway replacing Kie endpoints; A/B image quality vs historical Kie outputs.
4. After day-7 Qwen+SDXL CN, freeze a “production model set” hash list in git.
5. Optional: file Huly issue for install execution under ops/creative project.

---

## Evidence index

| ID | Source | URL | Provenance |
|---|---|---|---|
| E1 | Qwen-Image-2512 card | https://huggingface.co/Qwen/Qwen-Image-2512 | curl raw README |
| E2 | Qwen-Image-Edit card | https://huggingface.co/Qwen/Qwen-Image-Edit | curl raw README |
| E3 | Qwen-Image base | https://huggingface.co/Qwen/Qwen-Image | curl raw README |
| E4 | Z-Image-Turbo card | https://huggingface.co/Tongyi-MAI/Z-Image-Turbo | curl raw README |
| E5 | FLUX.2-klein-4B card | https://huggingface.co/black-forest-labs/FLUX.2-klein-4B | curl raw README |
| E6 | FLUX.1 NC license | https://raw.githubusercontent.com/black-forest-labs/flux/main/model_licenses/LICENSE-FLUX1-dev | curl |
| E7 | Krea-2 README | https://github.com/krea-ai/krea-2 | curl raw README |
| E8 | SDXL card | https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0 | curl raw README |
| E9 | HF API model metadata | https://huggingface.co/api/models/* | curl JSON |
| E10 | HF tree sizes | https://huggingface.co/api/models/*/tree/main | curl JSON |
| E11 | Qwen-Edit local VRAM guide | https://localaimaster.com/blog/qwen-image-edit-local-guide | curl HTML |
| E12 | Z-Image Turbo Comfy guide | https://localaimaster.com/blog/z-image-turbo-comfyui | curl HTML |
| E13 | Local 2026 license map | https://www.digitalapplied.com/blog/local-image-generation-flux-stable-diffusion-comfyui-2026 | curl HTML |
| E14 | FLUX local 3090 guide | https://insiderllm.com/guides/flux-locally-complete-guide/ | curl HTML |
| E15 | Comfy Qwen-2512 docs | https://docs.comfy.org/tutorials/image/qwen/qwen-image-2512 | curl HTML |
| E16 | Comfy Z-Image docs | https://docs.comfy.org/tutorials/image/z-image/z-image-turbo | curl HTML |
| E17 | Stability license page | https://stability.ai/license | web_search snippet |
| E18 | BFL Kontext announcement | https://bfl.ai/announcements/flux-1-kontext-dev | web_search |
| E19 | Comfy multi-GPU patterns | GitHub ComfyUI discussions; dual-GPU guides | web_search |
| E20 | Comfy API /prompt | https://docs.comfy.org/development/comfyui-server/comms_routes | web_search |
| E21 | FLUX.2-klein GGUF | https://huggingface.co/unsloth/FLUX.2-klein-4B-GGUF | HF API |
| E22 | VentureBeat klein Apache | https://venturebeat.com/technology/black-forest-labs-launches-open-source-flux-2-klein-to-generate-ai-images-in | web_search |

---

*Research agent: Geraldo v2.2 (geraldov21). Single archive path below.*
