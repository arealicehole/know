# Local Image Gen AI Box — Kie Replacement (Final Pick List + Install Plan)

**Task:** `t_40a6a892`  
**Date (UTC):** 2026-09-17  
**Agent:** Geraldo v2.2 (`geraldov21`)  
**Hardware:** AI Box dual RTX 3090 24GB (Ampere sm_86), 32GB system RAM, Linux, exclusive creative mode (no concurrent LLM on same GPUs)  
**Companion deep dive:** `/home/ice/know/research/local-img-gen-dual-3090-kie-replacement-20260917.md`  
**Scratch evidence:** `/srv/scratch/img-gen-research/` (HF API cards, license text, size trees)

---

## objective

Synthesize open-weight local image generation options into a **final pick list + day-1/day-7 install plan** so AI Box can replace paid Kie cloud spend with programmatic txt2img, img2img/edit, and ControlNet-class control via ComfyUI, under a commercial-clean (Apache-first) license path.

## summary

**Final pick (Apache-first production path):**

| Priority | Model / layer | Role | License | Why it wins |
|---|---|---|---|---|
| **P0 runtime** | **ComfyUI** dual workers `:8188` / `:8189` | Engine + Kie-replacement HTTP API | OSS | Native templates for all P0 models; `POST /prompt` |
| **P0 gen+edit** | **FLUX.2 [klein] 4B** (+ GGUF already owned) | Fast unified txt2img + multi-ref edit | **Apache 2.0** | ~13GB VRAM; BFL names RTX 3090; commercial OK |
| **P0 fast t2i** | **Z-Image-Turbo 6B** | Photoreal + bilingual text; 8-step | **Apache 2.0** | Fits ≤16GB BF16; Comfy native; Fun Union ControlNet available |
| **P0 day-1 control** | **Z-Image-Turbo Fun Union ControlNet** | Canny / HED / Depth / Pose / MLSD | Apache stack | Prefer over waiting for SDXL day-1; still keep SDXL as depth ecosystem |
| **P1 quality** | **Qwen-Image-2512** FP8 (+ Lightning 4-step LoRA) | Best open text-in-image / realism | **Apache 2.0** | ~20B; FP8 fits one 3090; official Comfy templates |
| **P1 edit** | **Qwen-Image-Edit** FP8/GGUF | Instruction + semantic/appearance edit | **Apache 2.0** | Complements klein multi-ref |
| **P1 control depth** | **SDXL + ControlNet + IP-Adapter** | Mature edge/pose/ID ecosystem | OpenRAIL++ | Fallback when Z-Image CN / instruction edit insufficient |
| **P2 optional** | **FLUX.1 [schnell]** | Mature FLUX.1 LoRA ecosystem | **Apache 2.0** | Commercial-clean FLUX.1 path |
| **P2 optional** | **Krea-2 Turbo** | Aesthetic t2i | Community (not Apache) | Only after license skim |
| **R&D only** | FLUX.1-dev / Fill / Kontext / Canny / Depth; FLUX.2-dev | Quality tools | **Non-commercial** without BFL paid license | Keep Fill offline from production routes until licensed |

**Multi-GPU pattern:** two independent ComfyUI processes (`CUDA_VISIBLE_DEVICES=0|1`), shared model disk — **not** single-process model parallel (no NVLink benefit; worse API concurrency).

**Ops alignment:** Matches Frank’s day-1 intuition (Z-Image-Turbo + Qwen-Image(+Edit) Apache, FLUX Tools license-aware, SDXL for ControlNet depth) with one upgrade: **Z-Image Fun Union ControlNet lands structural control on day-1**, so SDXL can slip to day-7 without blocking Kie parity.

**Exclusive creative:** AI Box creative lane stays separate from home `:8088` / LLM serving.

---

## final_pick_list

### Ship (production / commercial-clean)

1. **ComfyUI** (latest nightly or stable with native Qwen/Z-Image/FLUX.2 nodes)
2. **black-forest-labs/FLUX.2-klein-4B** or **unsloth/FLUX.2-klein-4B-GGUF** (copy from other host first)
3. **Tongyi-MAI/Z-Image-Turbo** via **Comfy-Org/z_image_turbo** split files (`z_image_turbo_bf16` or fp8 + `qwen_3_4b` TE + `ae` VAE)
4. **alibaba-pai/Z-Image-Turbo-Fun-Controlnet-Union** → `models/model_patches/`
5. **Qwen/Qwen-Image-2512** via **Comfy-Org/Qwen-Image_ComfyUI** FP8 diffusion + FP8 TE + VAE + optional Lightning LoRA
6. **Qwen/Qwen-Image-Edit** (FP8/GGUF path; confirm Edit-2511 template name at install)
7. **stabilityai/stable-diffusion-xl-base-1.0** + ControlNet Canny/Depth/OpenPose + IP-Adapter (day-7)
8. Thin **Kie-shim gateway** (FastAPI) → Comfy `/prompt` + `/history` + `/view`

### Do not ship to production routes (unless BFL commercial purchased)

- FLUX.1-dev, Fill-dev, Kontext-dev, Canny-dev, Depth-dev (NC license; Fill already owned → quarantine)
- FLUX.2-dev (~32B, ~178GB tree, non-Apache gated)
- FLUX.2 klein **9B** (NC; use **4B** only)

### Conditional

- **SD 3.5**: Stability Community License free commercial **under $1M revenue** — skip if Apache-only policy preferred
- **Krea-2**: open weights exist; community license — contact/read before client work

---

## install_plan

### Assumptions

- AI Box: exclusive creative GPUs; 32GB host RAM is the binding constraint
- Prefer stock PyTorch cu12.x with **sm_86** kernels; **no** Blackwell-only / NVFP4 packs
- Home Comfy at `/srv/apps/comfyui` is reference only — AI Box gets its own install or clean remote bind
- HF token available for gated downloads (schnell/dev if pulled); Apache models mostly ungated

### Day-1 (same day usable API — kill Kie for gen+edit+basic control)

```text
[ ] 0. Host prep
    - NVIDIA driver ≥535/550; nvidia-smi shows 2× 3090
    - mkdir -p ~/comfy-aibox && python3 -m venv .venv && source .venv/bin/activate
    - pip install torch torchvision --index-url https://download.pytorch.org/whl/cu124  # pin to known Ampere build
    - git clone https://github.com/comfyanonymous/ComfyUI.git
    - pip install -r requirements.txt; optional ComfyUI-Manager
    - Install city96 ComfyUI-GGUF if using klein GGUF

[ ] 1. Models (disk budget ~80–120GB day-1)
    # FLUX.2-klein — copy GGUF from other host first
    rsync -avP <other-host>:.../FLUX.2-klein*.gguf  models/unet/   # or diffusion_models per node docs
    # or hf download black-forest-labs/FLUX.2-klein-4B --local-dir ...

    # Z-Image-Turbo Comfy split
    hf download Comfy-Org/z_image_turbo \
      split_files/diffusion_models/z_image_turbo_bf16.safetensors \
      split_files/text_encoders/qwen_3_4b.safetensors \
      split_files/vae/ae.safetensors \
      --local-dir-use-symlinks False
    # place into models/{diffusion_models,text_encoders,vae}/

    # Z-Image Fun Union ControlNet
    hf download alibaba-pai/Z-Image-Turbo-Fun-Controlnet-Union \
      Z-Image-Turbo-Fun-Controlnet-Union.safetensors
    # → models/model_patches/

[ ] 2. Dual workers (two processes, shared models dir)
    # terminal A
    CUDA_VISIBLE_DEVICES=0 python main.py --listen 0.0.0.0 --port 8188
    # terminal B
    CUDA_VISIBLE_DEVICES=1 python main.py --listen 0.0.0.0 --port 8189
    # Optional specialty: 8188 = Z-Image turbo+CN; 8189 = klein edit

[ ] 3. Workflows (from Comfy templates / docs)
    - image_z_image_turbo.json
    - image_z_image_turbo_fun_union_controlnet.json
    - FLUX.2-klein t2i + multi-ref edit (Comfy native / BFL examples)
    - Save each as **API format** JSON for gateway

[ ] 4. Smoke + bench
    - 10× Z-Image 1024² 8-step; log sec/img + nvidia-smi peak
    - 10× klein 4-step t2i + 5× multi-ref edit
    - 5× Z-Image CN depth or canny lock
    - Target order-of-magnitude: Z-Image ~3–5s/img/GPU; dual workers ~2× throughput

[ ] 5. Kie shim (minimal)
    - POST /v1/images/generations → expand API-format workflow, POST Comfy /prompt
    - poll GET /history/{id} or WS /ws; GET /view?filename=
    - round-robin 8188/8189; sticky for multi-step edit sessions
    - Do NOT register FLUX.1-Fill routes

[ ] 6. Policy
    - Document Apache-only production allowlist
    - Quarantine any FLUX.1-dev family weights outside production model path
```

**Day-1 disk/layout sketch:**

```text
ComfyUI/
  models/
    diffusion_models/   z_image_turbo_bf16.safetensors  [+ klein if safetensors]
    unet/ or diffusion_models/   FLUX.2-klein*.gguf
    text_encoders/      qwen_3_4b.safetensors
    vae/                ae.safetensors
    model_patches/      Z-Image-Turbo-Fun-Controlnet-Union.safetensors
  user/default/workflows/  (API JSON exports)
```

### Day-7 (quality + edit parity + SDXL control depth)

```text
[ ] Qwen-Image-2512 FP8 pack (Comfy-Org/Qwen-Image_ComfyUI)
      diffusion: qwen_image_2512_fp8_e4m3fn.safetensors
      TE: qwen_2.5_vl_7b_fp8_scaled.safetensors
      VAE: qwen_image_vae.safetensors
      LoRA: Qwen-Image-Lightning-4steps-V1.0.safetensors
[ ] Qwen-Image-Edit FP8 or GGUF + official edit template
[ ] SDXL base + ControlNet (Canny, Depth, OpenPose) + IP-Adapter Plus
[ ] Optional: FLUX.1-schnell (Apache) for LoRA zoo
[ ] Optional: Krea-2 Turbo AFTER reading krea community license
[ ] Gateway endpoints: txt2img | img2img/edit | control | (optional) inpaint
[ ] Model lifecycle: /free unload; one heavy TE per worker max (32GB RAM)
[ ] Regression set: posters/text, faces/hands, pose lock, multi-ref product, bilingual glyphs
[ ] Pin Comfy commit + weight SHA256 manifest in git
[ ] Decide BFL commercial ONLY if Fill/Kontext quality is mandatory vs Qwen-Edit+klein
```

### Explicit non-goals

- Full FLUX.2-dev download (~178GB)
- Concurrent LLM on AI Box GPUs during creative exclusive mode
- Blackwell-only quant wheels
- Production use of FLUX.1-dev family without paid BFL license

### Home vs AI Box

| Host | Role |
|---|---|
| AI Box dual-3090 | **Creative exclusive** Comfy workers; Kie replacement |
| Home `:8088` / existing Comfy | Leave alone; do not steal VRAM for this stack |
| Other host with klein GGUF + Fill | Source of truth for weight copy; Fill stays non-prod |

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | FLUX.2-klein-4B is Apache 2.0, ~13GB VRAM, unified gen+multi-ref edit, named for RTX 3090/4070 | E1 E2 E14 | high | supported |
| C2 | Z-Image-Turbo is Apache 2.0, 6B, 8 NFE turbo, fits ~16GB consumer VRAM | E3 E4 E14 | high | supported |
| C3 | Qwen-Image-2512 and Qwen-Image-Edit are Apache 2.0 ~20B open weights with native Comfy paths | E5 E6 E14 | high | supported |
| C4 | FLUX.1-dev family including Fill/Kontext/Canny/Depth is Non-Commercial without BFL paid license | E7 E14 | high | supported |
| C5 | FLUX.1-schnell is Apache 2.0 | E14 | high | supported |
| C6 | FLUX.2-dev is gated non-Apache (~32.2B params, ~178GB weight tree) | E14 E15 | high | supported |
| C7 | Krea-2 open weights exist under krea-2-community-license (not Apache) | E14 | high | supported |
| C8 | ComfyUI exposes POST /prompt, history, view, ws suitable as Kie replacement façade | E8 | high | supported |
| C9 | Z-Image-Turbo Fun Union ControlNet (Canny/HED/Depth/Pose/MLSD) has official Comfy workflow | E4 | high | supported |
| C10 | Qwen BF16 full residency ~40GB-class; FP8 ~16–21GB community figures for 24GB cards | E9 | medium | partial |
| C11 | Dual ComfyUI workers preferred over single-process multi-GPU for API throughput on dual 3090 | E10 | medium | partial |
| C12 | Day-1 Apache stack (klein + Z-Image ± CN) is sufficient to start killing Kie spend before Qwen day-7 | E1–E4 synthesis | medium | partial |
| C13 | Z-Image-Edit still “to be released” on Tongyi card — do not block edit path on it | E3 | high | supported |
| C14 | Operator should quarantine owned FLUX.1-Fill from production until BFL commercial | E7 | high | supported |

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | FLUX.2-klein-4B HF card | https://huggingface.co/black-forest-labs/FLUX.2-klein-4B | web_extract + curl README | Apache 2.0; ~13GB VRAM; RTX 3090/4070; multi-ref edit |
| E2 | FLUX.2-klein Comfy/Diffusers mention | same + BFL blog | web_extract | Available in ComfyUI and Diffusers |
| E3 | Z-Image-Turbo HF card | https://huggingface.co/Tongyi-MAI/Z-Image-Turbo | web_extract + local README | Apache 2.0; 6B; 8 NFE; ≤16G VRAM; Z-Image-Edit TBD |
| E4 | Comfy Z-Image-Turbo docs | https://docs.comfy.org/tutorials/image/z-image/z-image-turbo | web_extract | Split files + Fun Union ControlNet workflow |
| E5 | Qwen-Image-2512 HF | https://huggingface.co/Qwen/Qwen-Image-2512 | web_extract | Dec 2025 update; 20B; realism/text improvements |
| E6 | Comfy Qwen-2512 docs | https://docs.comfy.org/tutorials/image/qwen/qwen-image-2512 | web_extract | FP8 recommended; Lightning 4-step LoRA paths |
| E7 | FLUX.1-dev NC license | https://raw.githubusercontent.com/black-forest-labs/flux/main/model_licenses/LICENSE-FLUX1-dev | curl | Lists Fill/Depth/Canny/Kontext under NC; commercial needs BFL |
| E8 | Comfy server routes | https://docs.comfy.org/development/comfyui-server/comms_routes | web_extract | POST /prompt, /history, /view, /ws |
| E9 | Qwen-Edit local VRAM guide | https://localaimaster.com/blog/qwen-image-edit-local-guide | prior research curl | Community FP8 ~20.5GB / BF16 ~40.9GB class figures |
| E10 | Dual-GPU Comfy patterns | community dual-worker guides | prior research web_search | Two instances via CUDA_VISIBLE_DEVICES common pattern |
| E11 | Qwen-Image-Edit HF | https://huggingface.co/Qwen/Qwen-Image-Edit | web_extract | Instruction edit; semantic + appearance dual path |
| E12 | BFL self-hosted commercial terms | https://bfl.ai/legal/self-hosted-commercial-license-terms | web_search | Paid path for licensed FLUX dev self-host |
| E13 | DigitalApplied 2026 license map | https://www.digitalapplied.com/blog/local-image-generation-flux-stable-diffusion-comfyui-2026 | web_search | SD3.5 $1M revenue gate; Apache/open defaults |
| E14 | HF API model cards batch | https://huggingface.co/api/models/* | curl JSON in scratch | Licenses + param counts for all shortlist models |
| E15 | HF tree sizes | https://huggingface.co/api/models/*/tree/main | curl JSON size_info.json | klein ~24GB; Z-Image ~33GB; Qwen ~58GB; FLUX.2-dev ~178GB |
| E16 | AI Box docs index | /home/ice/fed/docs/ai-box/README.md | local | ssh ai-box; creative hardware context |
| E17 | Prior Geraldo deep report | /home/ice/know/research/local-img-gen-dual-3090-kie-replacement-20260917.md | local | Full A–G comparison + claims |

## gaps

- On-box 3090 latency/VRAM not measured in this container (no GPUs here) — day-1 bench is mandatory.
- Kanban attachment landscape MD not readable inside Docker mount; synthesis used task body + prior deep research + live HF/Comfy verification.
- Qwen-Image-Edit-2511 vs base Edit repo naming — confirm template/HF id at install time.
- Fun Union ControlNet quality vs classic SDXL ControlNet not A/B’d — keep SDXL on day-7.
- Krea community license full PDF not legal-reviewed here.
- FLUX.2-dev full commercial contract text partially gated — treat as non-prod.
- Home Comfy path `/srv/apps/comfyui` not inspected from this container (not mounted).
- Julio deep_research task_id=7c5e924b not found in maestro status (comment noise / different system).

## confidence

- **level:** high on licenses, repos, Comfy support, and pick ordering; **medium** on absolute sec/img and exact FP8 VRAM until AI Box benches
- **rationale:** Primary HF cards + NC license text + Comfy official docs + HF API param/size trees verified this run; throughput extrapolated from vendor/community 4090-class numbers with Ampere derate

## recommended_next_actions

1. **Ops install ticket:** execute Day-1 checklist on `ai-box` (Tailscale); attach nvidia-smi benches.
2. **Copy weights:** rsync FLUX.2-klein GGUF from other host; leave Fill out of production model dir.
3. **Build Kie shim:** map historical Kie call shapes → three API workflows (t2i / edit / control).
4. **License policy note:** Apache-only allowlist in repo; escalate only if Fill/Kontext quality gap is real after Qwen-Edit+klein.
5. **Day-7:** Qwen-2512 FP8 + Edit + SDXL CN; freeze SHA256 manifest.
6. Optional Huly: file install issue under AI Box / creative project with this report path.

---

## control_matrix_quickref

| Need | Day-1 pick | Day-7 pick |
|---|---|---|
| Fast txt2img | Z-Image-Turbo; klein | + Qwen-2512 Lightning |
| Quality / text glyphs | Z-Image (good); klein OK | **Qwen-2512** |
| Multi-ref / quick edit | **klein** | + **Qwen-Edit** |
| Canny/Depth/Pose | **Z-Image Fun Union CN** | + **SDXL ControlNet** |
| Inpaint | Qwen-Edit masks / SDXL | Fill only if BFL licensed |
| Commercial-clean | Apache stack only | same |

---

## throughput_estimates (verify on box)

| Workload | 1× 3090 | 2 workers |
|---|---|---|
| Z-Image-Turbo 8-step 1024² | ~3–5 s → ~12–20/min | ~24–40/min |
| FLUX.2-klein 4-step | ~1–4 s | ~2× |
| Qwen FP8 50-step | ~30–90 s (Lightning 4-step much faster) | ~2× if RAM allows |
| SDXL + CN 25-step | ~5–12 s | ~2× |

---

*Single durable archive for task t_40a6a892. Do not double-publish to fed/docs or research-output.*
