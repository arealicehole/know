# Programmatic print outpaint — Fill vs Qwen execution + alternatives

**Task ID:** `t_b7a11aaa`  
**Agent:** Geraldo v2.2 (`geraldov21`)  
**Date (UTC):** 2026-09-30  
**Consumer:** Print Junkie / WVRS-class flyer bleed on AI Box dual-3090 Comfy (`:8188`/`:8189`)  
**Scope lock:** self-host open weights only; home 5060 reserved for Parakeet (not Fill primary); print geometry bleed 0.0625", safe 0.125", 300 DPI; art re-paste keep-zone after gen.

---

## objective

Diagnose whether Print Junkie’s weak bleed rings (parchment/matte frames, incomplete scene continue, mask/pad ambiguity, OOM on wrong host) are primarily **model limits** vs **pad/mask/graph/API execution**, then give **programmatic recipes** for owned models (FLUX.1-Fill, Qwen-Image-Edit 2511 / Qwen 2.1 edit, Qwen InstantX outpaint blueprint, SDXL CN) plus a ranked A→B→C playbook ops can paste into `print-bleed-outpaint`.

## summary

**Primary diagnosis (high confidence):** for *print bleed* rings of ~19–38 px at 300 DPI, **execution dominates model**. Official FLUX Fill outpaint is `ImagePadForOutpaint` → `DifferentialDiffusion` → `FluxGuidance(~30)` → `InpaintModelConditioning` → `KSampler(denoise=1, cfg=1, steps≈20, euler)`. Frame/parchment artifacts usually mean (1) prompt bias (“border/frame/matte”), (2) oversized pad relative to content, (3) missing or wrong **re-paste** of locked type/art keep-zone, (4) inverted or non-feathered mask, (5) resolution not multiple of 8/16 after pad, or (6) running Fill on the wrong GPU/host (home 5060 OOM history). Fill is still the correct *mask-faithful* tool for thin rings when graph is correct.

**Model levers (owned inventory):**
| Path | On disk | Role for bleed | License note |
|---|---|---|---|
| **A. FLUX.1-Fill-dev** | `flux1-fill-dev.safetensors` (quarantine-nc OK) | **Primary** thin-ring outpaint | FLUX.1 [dev] Non-Commercial Materials — quarantine local OK; not cloud product |
| **B. Qwen-Image-Edit-2511** | `qwen_image_edit_2511_fp8mixed.safetensors` | Pad + NL instruction expand **without InstantX**; better semantic “continue desert sunset” when Fill frames | Apache 2.0 stack (Qwen) |
| **C. Qwen InstantX CN-Inpaint** | **NOT on disk** | Unblocks blueprint `Image Outpainting (Qwen-Image).json` | Apache-2.0 InstantX; Comfy repack **4.23 GB** exact name below |
| SDXL + CN canny/depth/union | On disk | Competitive continuous sky/desert *only* if traditional pad+inpaint CN workflow tuned; heavier ops | OpenRAIL++ |
| FLUX.2-klein-4B / Z-Image-Turbo | On disk | **Do not** use as pure t2i edge-extend for locked type flyers | Apache — wrong tool class |

**Ranked production playbook (WVRS desert sunset + locked type):**  
**A** → tune Fill thin-ring + re-paste (no download).  
**B** → if A still frames/seams, Qwen-Edit pad+instruction on AI Box.  
**C** → download InstantX Inpaint CN (4.23 GB) + retarget blueprint to owned Qwen checkpoint names.  
Do **not** promote klein/z-image pure t2i outpaint. Keep dual workers on AI Box only.

---

## plan

| task_id | title | status |
|---|---|---|
| T1 | Official Fill outpaint graph + hyperparams | done |
| T2 | InstantX weight exact name/size/license/HF | done |
| T3 | Qwen-Edit 2511 pad/NL path without InstantX | done |
| T4 | Comfy `/prompt` client pattern + PARAM bugs | done |
| T5 | SDXL / anti-patterns / download shortlist | done |
| T6 | WVRS ranked playbook + durable publish | done |

**Assumptions:** inventory from task body (2026-09-30 AI Box creative mode) is ground truth; skill path referenced but not mounted in this container — recipes are source-backed for skill paste-in.  
**Success criteria:** diagnosis checklist + per-model recipes + Python API pattern + download shortlist + A→B→C playbook with citations; no invented weights.

---

## findings

### 1. Model vs execution diagnosis checklist

Use this **before** swapping models. Symptom → likely cause → fix.

| Symptom | Likely cause | Check / fix |
|---|---|---|
| Matte / parchment / picture-frame border in ring | Prompt contains *frame, border, matte, canvas, poster edge, white margin*; or model “completing” a framed object | Positive: describe **scene content only** (“continuous desert sunset sky, warm sand dunes continuing past edge, no border”). Negative / avoid: frame, matte, border, vignette, white margin, polaroid. Prefer empty or scene-only prompt on Fill (community reports empty prompt works well). |
| Hard seam / halo at original edge | `feathering=0` or too large; mask not soft | Official pad uses feathering **24** on large pads; for 19–38 px rings use feathering **~4–12** (≤ half pad width). Soften mask further with blur if needed. |
| Original art “eaten” or type ruined | Mask covers keep-zone; no **re-paste**; wrong mask polarity | Comfy convention: pad mask marks **areas to generate** (extension white/high). After decode, **composite original pixels** back into keep-zone (ImageCompositeMasked / PIL paste). Print path: re-paste locked type + safe zone after gen. |
| Only one side extends / wrong geometry | Pad L/T/R/B not matching bleed math | Compute px = inches × DPI. Bleed 0.0625" @ 300 DPI = **18.75 → round to 19 or 24 (prefer multiple of 8)**. Safe 0.125" = **37.5 → 38 or 40**. Uniform ring: same L=T=R=B. |
| Soft mush / incomplete scene continue | Pad too large in one shot; denoise/guidance off; no multi-pass | For >~64–128 px per side, multi-pass smaller rings. Fill: denoise **1.0**, FluxGuidance **~30**, KSampler cfg **1**, steps **20** (official template). |
| Color shift in unmasked area | Known Fill limitation (BFL card) | Re-paste original RGB into unmasked region after VAE decode (always for print). |
| Edge texture lines | BFL known limitation on complex textures | Slightly larger feather; second pass with lower guidance; or Qwen-Edit semantic continue. |
| OOM / second-pass fail | Wrong host (home 5060); concurrent LLM; full BF16 Fill + large canvas | **AI Box only**; dual workers one job each; unload other models; keep final canvas modest; GGUF/FP8 Fill only if needed. |
| 500 on `/prompt` | UI JSON not API format; unsubstituted `PARAM_*`; missing node class; wrong ckpt filename | Export **API format**; string-replace all PARAM tokens; validate node class_types against server object_info. |
| Ring looks correct in preview but print crops wrong | Geometry applied after gen not matching prepress | Lock: final trim size + bleed; outpaint canvas = trim + 2×bleed; re-paste art at exact offsets. |

**Mask polarity (Comfy Fill path):**
- `ImagePadForOutpaint` emits IMAGE (padded, typically gray fill in new area) + MASK for extension region.
- Feed both into `InpaintModelConditioning` with positive (FluxGuidance) and negative (`ConditioningZeroOut` of same text encode in official template).
- White/high mask = paint; protect original with low mask + mandatory pixel re-paste for type lock.

**ImagePadForOutpaint vs manual RGBA pad:**
- Prefer **core `ImagePadForOutpaint`** (or KJNodes `ImagePadForOutpaintMasked`) so mask geometry matches pad. Manual RGBA pad is fine if you build an equivalent mask and do not flip polarity.
- Pad amounts step-friendly: **multiples of 8** (Flux latent / VAE friendly); Qwen packs often want dims compatible with VAE×patch (docs note latent packing — keep even multiples of 16 when unsure).

**DifferentialDiffusion:** present in official Fill outpaint template; improves boundary matching of input vs fill. Keep enabled (strength 1 in template widgets).

**Print geometry quick math (300 DPI):**

```text
bleed_px = round(0.0625 * 300)  # 19
safe_px  = round(0.125  * 300)  # 38
# Prefer ceil to multiple of 8:
bleed8 = ((bleed_px + 7) // 8) * 8   # 24
safe8  = ((safe_px  + 7) // 8) * 8   # 40
```

For *bleed-only* production rings, start **24 px** all sides (or true 19 if skill already locks non-8 and latent path tolerates). Avoid 400 px demo pads from tutorials on flyer art — those teach framing and OOM.

---

### 2. Best programmatic recipes by owned model class

#### 2.1 FLUX.1-Fill (current primary)

**Sources:** Comfy official Fill tutorial + `flux_fill_outpaint_example.json` template.

**Graph (API class_types, official order):**

```text
UNETLoader(flux1-fill-dev.safetensors)
  → DifferentialDiffusion
DualCLIPLoader(clip_l + t5xxl_fp16, type flux)
VAELoader(ae.safetensors)
LoadImage → ImagePadForOutpaint(left,top,right,bottom,feathering)
CLIPTextEncode(positive scene text)
  → FluxGuidance(guidance≈30)
  → InpaintModelConditioning(positive, negative, vae, pixels=padded, mask=pad_mask)
CLIPTextEncode → ConditioningZeroOut → negative of InpaintModelConditioning
KSampler(model=DD, steps=20, cfg=1, sampler=euler, scheduler=normal, denoise=1)
VAEDecode → SaveImage
[+ post: ImageCompositeMasked / external PIL re-paste original keep-zone]
```

**Official template widget defaults (large scenic demo):** pad L/R/B = **400**, T = **0**, feathering **24**; FluxGuidance **30**; KSampler steps **20**, cfg **1**, denoise **1**, euler/normal.

**Print-bleed hyperparams (recommended):**

| Param | Thin bleed ring | Notes |
|---|---|---|
| left/top/right/bottom | **19–24** (or **38–40** if expanding into safe) | Not 400. Match prepress. |
| feathering | **6–12** (thin); **16–24** if pad ≥64 | ≤ ~half of pad width |
| FluxGuidance | **25–30** | Official 30; lower if over-saturated edges |
| steps | **20–28** | Official 20; +steps rarely fixes frames |
| cfg (KSampler) | **1** | Fill is guidance-distilled |
| denoise | **1.0** | Fill advantage vs base Flux outpaint |
| prompt | Scene continue only; try **empty** if still framing | Avoid “poster/frame/border” |
| multi-pass | If need >64 px/side: 2–3 passes of 24–32 px | Re-paste each pass |
| VRAM | Full Fill ~24GB class | AI Box 3090; not home 5060 primary |

**Multi-pass strategy:**  
Pass1: pad 24 all sides → gen → re-paste keep → optional slight crop alignment.  
Pass2: only sides still short of target.  
Never double full-canvas denoise without re-paste.

**License:** FLUX.1 Fill [dev] = Non-Commercial Materials (same family as FLUX.1 [dev] NC). Local quarantine OK per ops; **do not** expose as commercial cloud product weight without BFL paid license. Generated *images* may have broader use under BFL’s stated terms — ops already quarantine weights.

---

#### 2.2 Qwen-Image-Edit 2511 / Qwen 2.1 edit — pad + NL without InstantX

**On disk:** `qwen_image_edit_2511_fp8mixed.safetensors` (+ likely shared `qwen_image_vae`, `qwen_2.5_vl_7b_*` TE). Official Comfy docs list `qwen_image_edit_2511_bf16` / Lightning 4-step LoRA; fp8mixed is the owned production quant.

**When it beats Fill:**
- Semantic “continue this desert sunset background past the edges” with less frame bias.
- Instruction edits that understand layout (keep subject, extend environment).
- Apache-clean commercial path when NC Fill is policy-sensitive for a client route.

**When Fill still wins:**
- Exact mask-limited thin ring with maximum pixel lock on interior (Fill + re-paste).
- Already-tuned Fill graph and no instruction drift.

**Programmatic pattern (no InstantX):**

```text
1. Client-side or graph pad: expand canvas by bleed_px (gray or edge-replicate).
2. Optional: build mask = ring only (for any latent noise mask path).
3. Qwen-Image-Edit workflow (API export of native edit template):
   - Load edit UNet (2511 fp8mixed)
   - Load Qwen2.5-VL TE + qwen_image_vae
   - Scale Image to Total Pixels ~1MP if huge inputs (official tip) — for print bleed
     you may bypass if already print-res and VRAM OK
   - Instruction prompt examples:
     "Extend the image canvas on all sides. Continue the desert sunset sky and sand
      seamlessly beyond the original borders. Do not add a frame, matte, border,
      white margin, or vignette. Keep the central artwork and any text unchanged."
4. Post: **hard composite** original (or original+type layer) into center keep-zone.
```

**Note:** Edit models can still drift global appearance (2511 improves consistency vs older). For locked type flyers, **never** trust the model alone — re-paste vector/raster type and logo after edit.

**Community:** Civitai “Qwen 2511 Outpaint everything” style workflows confirm pad+edit outpaint is a real pattern; treat as secondary evidence, prefer Comfy native edit template + your pad.

---

#### 2.3 Qwen official outpaint / InstantX blueprint (blocked → unblock)

**Exact Comfy weight filename (required by official template):**

```text
Qwen-Image-InstantX-ControlNet-Inpainting.safetensors
```

**HF sources:**
| Item | URL |
|---|---|
| Comfy repack (preferred) | `https://huggingface.co/Comfy-Org/Qwen-Image-InstantX-ControlNets` |
| File path | `split_files/controlnet/Qwen-Image-InstantX-ControlNet-Inpainting.safetensors` |
| Size | **4,234,599,432 bytes (~4.23 GB)**; SHA256 `49c01aafe6545c1f6e7627a724c8fb13357fe74efee235622158f0a8f30e5458` |
| Upstream InstantX | `https://huggingface.co/InstantX/Qwen-Image-ControlNet-Inpainting` |
| License (InstantX card) | **apache-2.0** |
| Params (card) | ~2B ControlNet (6 double blocks), trained 1328², supports outpainting |
| Comfy version | native support; template needs ComfyUI **≥ 0.3.59** |
| Official workflow | `image_qwen_image_instantx_inpainting_controlnet.json` |

**Place on AI Box:**

```text
ComfyUI/models/controlnet/Qwen-Image-InstantX-ControlNet-Inpainting.safetensors
```

**Retarget steps for blocked blueprint `Image Outpainting (Qwen-Image).json`:**
1. Download Inpaint CN file above (Union CN optional — not required for outpaint-inpaint).
2. Fix diffusion ckpt string: template still names **`qwen_image_fp8_e4m3fn.safetensors`** — retarget to owned **`qwen_image_2512_fp8_e4m3fn.safetensors`** (or whichever base Qwen-Image UNet you run with InstantX; InstantX card is trained vs Qwen-Image base — prefer base 2512 path over Edit UNet unless docs confirm Edit+CN).
3. TE: `qwen_2.5_vl_7b_fp8_scaled.safetensors`; VAE: `qwen_image_vae.safetensors`.
4. Graph uses `ControlNetLoader` → `ControlNetInpaintingAliMamaApply` + `SetLatentNoiseMask` + optional `ImageCompositeMasked` (template notes paste-back).
5. For outpaint: pad image + mask ring (same as Fill), then CN inpaint apply; prompt should **describe full scene** (InstantX limitation: sensitive to prompts; prefer descriptive full-image prompt, not bare instructions).
6. Export API JSON; wire PARAM for image name, pads, prompt, seed.
7. Smoke on `:8188` only first; watch VRAM (Qwen base FP8 + 4.2GB CN + TE).

**Optional Union CN:** `Qwen-Image-InstantX-ControlNet-Union.safetensors` (~3.54 GB) — not needed for bleed outpaint.

---

#### 2.4 SDXL ControlNet / traditional outpaint

**Competitive when:** continuous sky/sand gradients and you already have SDXL inpaint or CN union workflows; team knows SDXL denoise/mask folklore.

**Recipe sketch:** Checkpoint SDXL → `ImagePadForOutpaint` → VAE encode → SetLatentNoiseMask or inpaint conditioning → CN canny/depth optional on padded image → KSampler denoise 0.85–1.0 with dedicated inpaint ckpt if available.

**Vs Fill for WVRS:** SDXL often weaker prompt-scene fidelity and text safety; use as **D** fallback if A–C fail, not primary.

---

#### 2.5 What not to use for edge extend

| Model | Why not |
|---|---|
| **FLUX.2-klein-4B pure t2i** | Gen/edit model — no fill mask contract; will redraw whole frame; ruins locked type |
| **Z-Image-Turbo pure t2i** | Same — turbo t2i/CN, not bleed ring fill |
| Base FLUX.1-dev outpaint without Fill | Possible but denoise must be <1 for consistency; Fill exists specifically so denoise=1 stays consistent |
| Home 5060 Fill | OOM history; reserved ASR; routing policy forbids |
| Cloud BFL Fill API | Ops want self-host; NC weight already local |

---

### 3. Code-level Comfy execution

**Minimal Python client (stdlib + optional websocket-client)** — pattern from Comfy docs Method 2:

```python
import json, uuid, urllib.request, urllib.parse, time
from pathlib import Path

SERVERS = ["http://AI_BOX:8188", "http://AI_BOX:8189"]  # dual workers

def upload_image(server, path, image_type="input", overwrite=True):
    # multipart POST /upload/image  (use requests or manual multipart)
    ...

def load_api_workflow(path):
    return json.loads(Path(path).read_text())

def substitute(workflow, mapping):
    """Replace PARAM_* string tokens AND set known node inputs."""
    raw = json.dumps(workflow)
    for k, v in mapping.items():
        raw = raw.replace(k, str(v))
    wf = json.loads(raw)
    # Prefer explicit node edits over tokens when possible:
    # wf["17"]["inputs"]["image"] = uploaded_name
    # wf["44"]["inputs"]["left"] = 24
    # wf["23"]["inputs"]["text"] = prompt
    # wf["3"]["inputs"]["seed"] = seed
    if "PARAM_" in raw:
        raise ValueError("Unsubstituted PARAM_ tokens remain — abort before /prompt")
    return wf

def queue_prompt(server, workflow, client_id=None):
    client_id = client_id or str(uuid.uuid4())
    payload = {"prompt": workflow, "client_id": client_id}
    req = urllib.request.Request(
        f"{server}/prompt",
        data=json.dumps(payload).encode(),
        headers={"Content-Type": "application/json"},
    )
    with urllib.request.urlopen(req) as r:
        return json.loads(r.read()), client_id

def wait_history(server, prompt_id, timeout=600, poll=1.0):
    t0 = time.time()
    while time.time() - t0 < timeout:
        with urllib.request.urlopen(f"{server}/history/{prompt_id}") as r:
            h = json.loads(r.read())
        if prompt_id in h:
            return h[prompt_id]
        time.sleep(poll)
    raise TimeoutError(prompt_id)

def view(server, filename, subfolder="", folder_type="output"):
    q = urllib.parse.urlencode(
        {"filename": filename, "subfolder": subfolder, "type": folder_type}
    )
    with urllib.request.urlopen(f"{server}/view?{q}") as r:
        return r.read()

def pick_server(servers):
    # GET /queue — prefer fewer running+pending
    best, score = servers[0], 1e9
    for s in servers:
        try:
            with urllib.request.urlopen(f"{s}/queue", timeout=2) as r:
                q = json.loads(r.read())
            n = len(q.get("queue_running", [])) + len(q.get("queue_pending", []))
            if n < score:
                best, score = s, n
        except Exception:
            continue
    return best
```

**Flow:** upload art → substitute pad/prompt/seed/ckpt → `POST /prompt` → poll `/history/{id}` (or WS `executing` node=null) → `GET /view` → local re-paste keep-zone → write print-ready PNG.

**Common 500 / failure modes:**
1. Submitting **UI-format** workflow (nodes as list with links array) instead of **API format** (`{node_id: {class_type, inputs}}`).
2. Leftover `"PARAM_IMAGE"`, `"{{pad}}"`, `%SEED%` strings → validation error or nonsense paths.
3. Filename in LoadImage not equal to **upload response `name`** (Comfy may rename collisions).
4. Missing custom node class on worker (blueprint needs node not installed).
5. Checkpoint path differs per worker disk — keep shared models mount.
6. Hitting home Comfy URL by mistake → OOM / wrong models.

**Dual-GPU queueing `:8188`/`:8189`:**
- Two processes, `CUDA_VISIBLE_DEVICES=0|1`, **shared** `models/` disk.
- Client round-robin / least-queue; sticky session if multi-pass needs same cached UNet.
- Do not run Fill + Qwen-Edit heavy TE on same GPU concurrently without `/free`.
- Creative exclusive lane: no LLM on those GPUs during bleed batch.

**Post-gen re-paste (print-critical, any model):**

```python
from PIL import Image
out = Image.open("comfy_out.png").convert("RGBA")
orig = Image.open("art_keep.png").convert("RGBA")
# out size = orig + pad; paste orig at (left, top)
left = top = 24  # must match pad used
out.paste(orig, (left, top))
out.convert("RGB").save("bleed_locked.png", dpi=(300, 300))
```

---

### 4. Download shortlist (only if better than tuning Fill)

| Priority | Artifact | Size | License | Why |
|---|---|---|---|---|
| **P0** | `Qwen-Image-InstantX-ControlNet-Inpainting.safetensors` from Comfy-Org repack | **4.23 GB** | Apache-2.0 (InstantX) | Unblocks official Qwen outpaint/inpaint CN blueprint; FOSS commercial-clean lever |
| P1 optional | InstantX Union CN | ~3.54 GB | Apache-2.0 | General control — not required for bleed |
| P1 optional | Qwen-Image-Edit-2511 Lightning 4-step LoRA | small | check lightx2v card | Faster edit passes |
| Skip | More FLUX.1-dev family (Kontext etc.) | large | NC | Policy/quarantine already |
| Skip | Random SDXL “outpaint UNet packs” | varies | varies | Only if A–C fail and SDXL already warm |

**FOSS helper nodes (usually already in core):** `ImagePadForOutpaint`, `InpaintModelConditioning`, `DifferentialDiffusion`, `FluxGuidance`, `ImageCompositeMasked`, `GrowMask`, `SetLatentNoiseMask`. KJNodes `ImagePadForOutpaintMasked` if optional mask merge needed. No mandatory exotic custom node for Fill path.

**Install one-liner (AI Box):**

```bash
hf download Comfy-Org/Qwen-Image-InstantX-ControlNets \
  split_files/controlnet/Qwen-Image-InstantX-ControlNet-Inpainting.safetensors \
  --local-dir /path/to/ComfyUI/models/controlnet-tmp
# move to models/controlnet/ with exact filename
```

---

### 5. Concrete recommendation — Print Junkie WVRS-class flyers

**Scenario:** desert sunset continuous background + locked type/logo; need bleed ring only; programmatic Comfy on AI Box.

#### Ranked playbook (paste into skill)

```text
### WVRS / Print Junkie bleed outpaint — try A → B → C

PRECHECK (every job)
- [ ] Route host = AI Box Comfy :8188 or :8189 only (never home 5060)
- [ ] Input = final art WITHOUT extra decorative frame in pixels
- [ ] Compute pad_px from 0.0625" bleed @ 300 DPI → 19–24 (prefer multiple of 8)
- [ ] Workflow = API-format JSON; zero unsubstituted PARAM_* tokens
- [ ] After gen: always re-paste original keep-zone (type + safe 0.125")

A) FLUX.1-Fill thin-ring (DEFAULT — no download)
- Graph: ImagePadForOutpaint → DifferentialDiffusion → FluxGuidance(30)
         → InpaintModelConditioning → KSampler(20 steps, cfg=1, denoise=1, euler)
- Pad: L=T=R=B = 24 (or 19 if skill locks exact); feathering = 8
- Prompt: "continuous desert sunset sky and sand dunes extending seamlessly,
  same lighting and color grade, no border, no frame, no matte, no white edge"
  — or empty prompt if A still frames
- Negative path: ConditioningZeroOut (official) / avoid frame words in positive
- Seed: randomize 2–4 candidates; pick least seam
- If soft seam: feathering 12; if frame: strip prompt nouns; verify mask preview
- If OOM: reduce longest side before pad; free other models; single GPU job
- PASS criteria: no matte ring, continuous sky/sand, type pixel-identical after re-paste

B) Qwen-Image-Edit-2511 pad + NL (if A fails frame/seam after 3 seeds)
- Pad same geometry client-side or in-graph
- Model: qwen_image_edit_2511_fp8mixed (+ TE + qwen VAE)
- Instruction: extend canvas; continue desert sunset; do not change text/logo;
  forbid frame/matte/border
- Lightning 4-step LoRA optional for speed
- Mandatory re-paste keep-zone (Edit can drift)
- PASS criteria: same as A; prefer if semantic continue clearly better

C) InstantX Qwen CN-Inpaint (if A+B fail or need mask-faithful Apache path)
- Download Qwen-Image-InstantX-ControlNet-Inpainting.safetensors (4.23 GB)
- Retarget blueprint ckpt names to owned qwen_image_2512_fp8_e4m3fn (not old fp8 name)
- Pad+mask → ControlNetInpaintingAliMamaApply path; descriptive full-scene prompt
- Composite original; Comfy ≥ 0.3.59
- PASS criteria: same; promotes FOSS commercial-clean stack

DO NOT
- klein / z-image pure t2i for edge extend
- 400px tutorial pads on flyer bleed
- Skip re-paste
- Run Fill on home 5060
- Ship NC Fill weights into public product API without license review
```

**Ops opinion alignment:** Frank’s take is correct — **execution first**. InstantX is the main *other model* install; Qwen-Edit pad is the main *no-download* alternative.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | Official FLUX Fill outpaint uses ImagePadForOutpaint + DifferentialDiffusion + FluxGuidance + InpaintModelConditioning + KSampler | E1 E2 E3 | high | supported |
| C2 | Official template defaults include FluxGuidance 30, steps 20, cfg 1, denoise 1, euler, pad demo 400/0/400/400 feather 24 | E2 E3 | high | supported |
| C3 | Fill is specialized so denoise can be 1.0 while staying consistent vs base Flux outpaint | E4 | high | supported |
| C4 | Print bleed 0.0625" @ 300 DPI ≈ 19 px; safe 0.125" ≈ 38 px — far smaller than tutorial 400 px pads | E0 geometry | high | supported |
| C5 | Frame/matte failures are commonly prompt + pad-size + missing re-paste execution issues | E1 E4 E5 synthesis | medium | partial |
| C6 | BFL documents color shift outside fill and edge lines on complex textures as Fill limitations | E6 | high | supported |
| C7 | FLUX.1 Fill [dev] is Non-Commercial Materials (FLUX.1 [dev] NC family) | E6 E7 E14 | high | supported |
| C8 | InstantX Qwen-Image ControlNet Inpainting supports outpainting; Apache-2.0; ~2B; Comfy native ≥0.3.59 | E8 E9 | high | supported |
| C9 | Exact Comfy filename is `Qwen-Image-InstantX-ControlNet-Inpainting.safetensors` at 4.23 GB (4234599432 bytes) | E9 E10 E11 | high | supported |
| C10 | Official InstantX Comfy template still references `qwen_image_fp8_e4m3fn.safetensors` — must retarget to owned 2512/edit names | E12 | high | supported |
| C11 | Qwen-Image-Edit-2511 is instruction edit with improved consistency; native Comfy templates; fp8mixed weight exists in Comfy-Org packs | E13 E15 | high | supported |
| C12 | Pad + NL Qwen outpaint without InstantX is a viable community/production pattern but needs hard composite for locked type | E13 E16 | medium | partial |
| C13 | Comfy programmatic path is upload → API workflow → POST /prompt → history/WS → GET /view | E17 | high | supported |
| C14 | klein/z-image should not be primary pure-t2i bleed extend tools | E14 inventory role | high | supported |
| C15 | Dual independent Comfy workers on 8188/8189 is the AI Box concurrency pattern | E14 | high | supported |
| C16 | ImagePadForOutpaint builds extension mask; feathering controls edge blend | E1 E18 | high | supported |

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E0 | Task inventory + print geometry | kanban `t_b7a11aaa` body | dispatcher task | Owned weights list; bleed 0.0625" / safe 0.125" / 300 DPI; dual :8188/:8189 |
| E1 | ComfyUI Flux.1 Fill dev tutorial | https://docs.comfy.org/tutorials/flux/flux-1-fill-dev | web_extract | Inpaint + outpaint same models; flux1-fill-dev + dual CLIP + ae; outpaint workflow |
| E2 | Official flux_fill_outpaint_example.json | https://raw.githubusercontent.com/Comfy-Org/workflow_templates/main/templates/flux_fill_outpaint_example.json | web_extract | Node graph + widgets: guidance 30; pad 400,0,400,400,f24; DD; IMC; KSampler 20/1/euler/denoise1 |
| E3 | Keris studio Comfy outpaint writeup | https://www.keris-studio.fr/blog/?p=14614 | web_search snippet | Documents pad, feathering, DifferentialDiffusion, FluxGuidance 30, IMC, KSampler cfg 1 |
| E4 | Stable Diffusion Art — Flux Fill outpaint | https://stable-diffusion-art.com/flux-fill-outpaint/ | web_extract | Fill vs Flux: denoise 1 consistency; pad directions; NC license note; ~24GB VRAM |
| E5 | DCAI Flux Tools Comfy guide | https://www.digitalcreativeai.net/en/post/how-use-powerful-flux1-tools-modify-images-comfyui | web_extract | Outpaint pad + feathering; DifferentialDiffusion quality note |
| E6 | HF FLUX.1-Fill-dev model card | https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev | web_extract | NC license agree wall; guidance_scale 30 example; limitations color shift / edge lines |
| E7 | FLUX.1 [dev] NC license | https://huggingface.co/black-forest-labs/FLUX.1-dev/blob/main/LICENSE.md | web_search | Non-commercial model license family |
| E8 | InstantX Qwen-Image-ControlNet-Inpainting | https://huggingface.co/InstantX/Qwen-Image-ControlNet-Inpainting | web_extract | apache-2.0; outpainting supported; Comfy ≥0.3.59; prompt sensitivity |
| E9 | Comfy-Org InstantX ControlNets README | https://huggingface.co/Comfy-Org/Qwen-Image-InstantX-ControlNets | web_extract | Exact filenames under models/controlnet/ |
| E10 | HF blob size page | https://huggingface.co/Comfy-Org/Qwen-Image-InstantX-ControlNets/blob/main/split_files/controlnet/Qwen-Image-InstantX-ControlNet-Inpainting.safetensors | web_extract | 4.23 GB |
| E11 | HF API tree listing | huggingface.co/api/models/.../tree/main/split_files/controlnet | terminal urllib | size 4234599432; sha256 49c01aaf… |
| E12 | Official InstantX inpaint workflow JSON | https://raw.githubusercontent.com/Comfy-Org/workflow_templates/refs/heads/main/templates/image_qwen_image_instantx_inpainting_controlnet.json | web_extract | ControlNetLoader filename; qwen_image_fp8_e4m3fn; AliMama apply; composite notes |
| E13 | Comfy Qwen-Image-Edit-2511 docs | https://docs.comfy.org/tutorials/image/qwen/qwen-image-edit-2511 | web_extract | Native edit workflow; model file names; consistency improvements |
| E14 | Prior GIU local img-gen pick list | /home/ice/know/research/local-image-gen-aibox-kie-replacement-t_40a6a892.md | local read | Dual workers; Fill NC quarantine; Apache Qwen/klein/z-image roles; /prompt façade |
| E15 | Comfy blog Qwen Edit 2511 | https://blog.comfy.org/p/qwen-image-edit-2511-and-qwen-image | web_extract | fp8mixed weight link; instruction editing |
| E16 | Civitai Qwen 2511 outpaint workflows | https://civitai.com/models/2287120/qwen2511-outpaint-everything | web_search | Community pad/outpaint with 2511 (secondary) |
| E17 | Comfy API examples | https://docs.comfy.org/development/comfyui-server/api-examples | web_extract | /prompt, WS, /history, /view patterns |
| E18 | Comfy basic outpaint tutorial | https://docs.comfy.org/tutorials/basic/outpaint | web_extract | Pad Image for outpainting params left/top/right/bottom/feathering; mask build |

---

## gaps

1. **Live skill JSON** (`print-bleed-outpaint` / `flux_fill_outpaint.json`) not readable from this container mount — recipes are official-template-aligned; skill owner should diff node IDs/PARAM names when pasting.
2. **No on-box smoke run** this session (no AI Box GPU from research container) — hyperparams are source-backed defaults, not A/B measured on WVRS-0327 art.
3. **InstantX + Qwen-Image-2512 vs Edit-2511** pairing: official CN card targets **Qwen-Image** base; treat Edit+CN as experimental until verified on AI Box.
4. **Exact home OOM stack trace** not re-fetched — treat as routing constraint only.
5. **BFL commercial license purchase** decision is legal/ops, not resolved here.

---

## confidence

- **level:** `high` on official Fill graph, InstantX filename/size/license, Comfy API pattern, dual-worker routing, anti-patterns for klein/z-image.  
- **level:** `medium` on “execution > model” for *this* WVRS symptom set (strong mechanistic fit; no live pixel proof this run).  
- **rationale:** Multiple primary sources (Comfy templates, HF cards, API docs) triangulate the graph and downloads; flyer-specific quality ranking is reasoned from geometry + known failure modes pending on-box bakeoff.

```json
{"level": "high", "rationale": "Official Comfy/BFL/InstantX/HF primary sources lock graph, weights, and API; WVRS ranking is execution-first pending on-box smoke."}
```

---

## recommended_next_actions

1. **Skill patch (Print Junkie):** set default pad to bleed-derived 19–24 px, feathering 8, FluxGuidance 30, denoise 1, cfg 1; add mandatory re-paste; strip frame words from default prompt; assert no `PARAM_` left before `/prompt`.
2. **On-box bakeoff:** same WVRS flyer, 4 seeds × {Fill A, Qwen-Edit B} on `:8188`; score frame/seam/type lock; only then download InstantX if A+B lose.
3. **If installing C:** pull 4.23 GB InstantX Inpaint CN; retarget blueprint ckpt strings; pin Comfy ≥0.3.59; document SHA256 `49c01aafe6545c1f6e7627a724c8fb13357fe74efee235622158f0a8f30e5458`.
4. **Gateway:** least-queue picker across 8188/8189; refuse Fill jobs whose target host is home.
5. **Optional Huly:** file ops note linking this report path for Julio drain if Frank wants durable issue trail.

---

## appendix

- Task: `t_b7a11aaa`
- Report path: `/home/ice/know/research/research-print-outpaint-exec-t_b7a11aaa.md`
- Playbook rank: **A Fill-tune → B Qwen-Edit pad → C InstantX 4.23GB**
- Do not use: klein/z-image pure t2i; home 5060 Fill; 400px demo pads for bleed
