# Qwen-Image-2.1 i2i bird-drift root cause (ComfyUI 0.37.x)

- task_id: `t_4a3a2a9f`
- board: swarm-ops
- requested_by: print-junkie (Frank)
- researcher: Geraldo v2.2 (geraldov21)
- date_utc: 2026-10-15
- status: verified from Comfy-Org/ComfyUI master source + official edit template

---

## objective

Explain why print-junkie’s Qwen-Image-2.1 “outpaint / i2i” Stokes run (`denoise=0.72`, prompt mentioning desert sunset / golden hour) produced a flock of birds instead of preserving the reference photo, verify the hypothesized `keep_vision` / blank-latent mechanism in ComfyUI `TextEncodeQwenImage21`, and deliver a standardized graph + settings recommendation for true identity-preserving edits vs classic SD-style i2i.

## summary

**Root cause is confirmed, with one important refinement of the operator’s wording.**

In `TextEncodeQwenImage21` (ComfyUI `comfy_extras/nodes_qwen.py`):

1. The node’s **sampler latent output is always a blank zero tensor** sized to the first reference (64-ch, 16× spatial), **not** a VAE-encoded copy of the photo.
2. When a VAE is connected and references are present, the node builds `ref_latents = [vae.encode(ref), …]`, attaches them to conditioning as `reference_latents`, and sets  
   **`keep_vision = (len(ref_latents) == 0)`**  
   i.e. **`keep_vision=False` exactly when references + VAE are present**.
3. `keep_vision=False` does **not** mean “the model never sees the photo.” It means the **Qwen3-VL vision-tower tokens are stripped from the text embedding** after encode, and replaced by **DiT-side reference latents spliced at `image_slots`**. The photo still conditions the run — but through **VAE latents in the DiT sequence**, not through VL tokens kept in the prompt embedding.
4. Official Qwen-Image-2.1 **image edit** is therefore **not SD-style img2img**. The official template wires:
   - `TextEncodeQwenImage21` with **VAE + image_1…**
   - KSampler **`denoise=1.0`**, **`cfg=1`**, **`euler` / `simple`**, **~25 steps** (published pipeline often 40–50)
   - latent_image = the encoder’s **empty** latent (not VAEEncode of the photo)
   - identity / layout come from **`reference_latents` + edit instruction**, not from partial denoising of an encoded init image
5. print-junkie’s Stokes graph used **classic i2i semantics** (`denoise=0.72` on what is effectively an empty / weakly structured canvas) plus a **generative scenic prompt** (“desert sunset”, “golden hour”, “warm desert light”). At high denoise on a blank canvas, the **text prompt dominates**; birds are a plausible sampling failure mode of that prompt distribution, not evidence that vision was “off” in the sense of zero image conditioning — but they **are** evidence that the graph was not running the native edit path (or was fighting it with SD-i2i assumptions).

**Standard fix:** stop treating Qwen-Image-2.1 as SD img2img. Use the **native edit graph** (refs on `TextEncodeQwenImage21` + VAE, sample the node’s empty latent at **denoise 1.0**, CFG 1, instruction-style prompt with `<image1>`). Optionally keep a **separate** low-denoise path only if you deliberately VAE-encode the photo into `latent_image` (non-standard for 2.1 edit; not what Comfy ships).

---

## plan

| task_id | title | status |
|---|---|---|
| T1 | Pull `TextEncodeQwenImage21` source and verify blank latent + `keep_vision` | done |
| T2 | Pull `qwen_image21.py` TE path for vision drop / `image_slots` | done |
| T3 | Pull `QwenImage21` model + DiT `build_sequence` for ref latent splice | done |
| T4 | Inspect official `image_qwen_image_2_1_image_edit.json` sampler settings | done |
| T5 | Map Stokes symptoms (birds, denoise 0.72, scenic prompt) onto mechanism | done |
| T6 | Publish standardized recommendation | done |

Assumptions:
- AI-box live filesystem was not reachable from this research container (`ai-box` DNS missing; print-junkie scratch not mounted). Verification is against **Comfy-Org/ComfyUI master** source and **Comfy-Org/workflow_templates** official edit JSON, which match the 0.37.0-era Qwen-Image-2.1 nodes described in the task (node behavior is stable in current master; issue #16433 explicitly cites ComfyUI 0.37.0 + this node).
- Stokes API payload was not re-fetched live; behavior is inferred from the task’s stated settings (denoise 0.72, scenic prompt, Qwen-Image-2.1) which are sufficient given the source-level mechanism.

---

## claims

### claim_id: C1
- **statement:** `TextEncodeQwenImage21` always returns an all-zero latent of shape `[1, 64, H/16, W/16]` sized from the first reference (or `resolution`), never a VAE-encoded init image.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E1]

### claim_id: C2
- **statement:** With VAE connected and ≥1 reference image, the node sets `keep_vision = False` because `keep_vision = len(ref_latents) == 0` and `ref_latents` is populated by `vae.encode(s)`.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E1]

### claim_id: C3
- **statement:** When `keep_vision` is false, the text encoder **drops VL vision tokens** from the embedding and records `image_slots` so the DiT can splice **reference latents** into those positions. Comment in source: “vision tokens are replaced by reference latents in the DiT”; “with no latents coming (no vae) they stay, and the image conditions through the text encoder alone.”
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E2], [E3]

### claim_id: C4
- **statement:** Operator phrasing “vision tower is disabled when a reference image is present” is **directionally right about `keep_vision=False`**, but **incomplete**: vision features are not discarded from the pipeline; they are **replaced by VAE reference latents** in the DiT sequence. If VAE is omitted, vision tokens are **kept** (`keep_vision=True`) and the image conditions via the text encoder only (weaker / different path).
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E1], [E2], [E3], [E4]

### claim_id: C5
- **statement:** Official ComfyUI Qwen-Image-2.1 **Image Edit** template samples with **denoise=1.0**, **cfg=1**, **euler/simple**, **steps=25**, feeds KSampler `latent_image` from `TextEncodeQwenImage21.latent` (empty) when `custom_size` is off, and requires VAE on the encode node with `image_1` as edit target.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E5], [E6], [E7]

### claim_id: C6
- **statement:** Using **denoise ≈ 0.72** on the encoder’s **empty** latent is **not** “72% edit / 28% preserve reference pixels.” With a zero init, partial denoise still starts from noise-dominated empty structure; reference identity must come from conditioning (`reference_latents`), while a scenic full-frame prompt competes as a fresh generation. That mismatch explains bird-swarm / prompt-dominated failures.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E1], [E5], [E8], [E9]

### claim_id: C7
- **statement:** The correct standardized path for identity-preserving Qwen-Image-2.1 edits is the **native edit graph** (instruction prompt + `<image1>`…, VAE on encode, empty latent, denoise 1.0, cfg 1), not SD-style “lower denoise + same checkpoint.”
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E5], [E6], [E7]

### claim_id: C8
- **statement:** Lowering denoise to 0.3–0.4 *without* switching to native edit semantics is a **weak / non-standard palliative** if `latent_image` remains the empty tensor; it may reduce prompt freedom slightly but does not implement true pixel init img2img and is not what Comfy documents for 2.1 edit.
- **status:** supported
- **confidence:** medium-high
- **evidence_refs:** [E5], [E8], [E9]

### claim_id: C9
- **statement:** Birds in the Stokes desert-sunset run are best explained as **text-prompt-dominated sampling** (scenic / sky / wildlife priors in the prompt) under a blank-canvas + high-denoise regime, not as a random VAE bug unique to Stokes. (MPS VAE corruption is a separate known failure mode that also yields “text wins, refs ignored,” but that is Apple Silicon-specific; AI-box is CUDA.)
- **status:** supported
- **confidence:** medium-high
- **evidence_refs:** [E9], [E10], task body

---

## evidence_index

### E1 — TextEncodeQwenImage21 source (blank latent + keep_vision)
- **source_label:** Comfy-Org/ComfyUI `comfy_extras/nodes_qwen.py` (master)
- **source_url:** https://raw.githubusercontent.com/Comfy-Org/ComfyUI/master/comfy_extras/nodes_qwen.py
- **provenance:** web_extract 2026-10-15; path stamp = raw GitHub master
- **excerpt (critical lines):**
  ```python
  if vae is not None:
      ref_latents.append(vae.encode(s))

  keep_vision = len(ref_latents) == 0
  positive = clip.encode_from_tokens_scheduled(
      clip.tokenize(prompt, images=images_vl, keep_vision=keep_vision, prevent_empty_text=True))
  negative = clip.encode_from_tokens_scheduled(
      clip.tokenize(negative_prompt, images=images_vl, keep_vision=keep_vision, prevent_empty_text=True))
  if len(ref_latents) > 0:
      positive = node_helpers.conditioning_set_values(
          positive, {"reference_latents": ref_latents}, append=True)
      negative = node_helpers.conditioning_set_values(
          negative, {"reference_latents": ref_latents}, append=True)
  latent = torch.zeros([1, 64, latent_h // 16, latent_w // 16],
                       device=comfy.model_management.intermediate_device())
  return io.NodeOutput(positive, negative, {"samples": latent})
  ```
- **note:** Docs explicitly call the third output an **“Empty latent”** (E4).

### E2 — TE drops vision tokens unless keep_vision
- **source_label:** `comfy/text_encoders/qwen_image21.py`
- **source_url:** https://raw.githubusercontent.com/Comfy-Org/ComfyUI/master/comfy/text_encoders/qwen_image21.py
- **provenance:** web_extract 2026-10-15
- **excerpt:**
  ```python
  # vision tokens are replaced by reference latents in the DiT: drop them and
  # record where each image goes
  # with no latents coming (no vae) they stay, and the image conditions through
  # the text encoder alone
  slots = []
  if not token_weight_pairs.get("keep_vision", False):
      for start, size in image_spans:
          keep[start:start + size] = False
          slots.append(int(keep[:start].sum()))
  ...
  if len(slots) > 0:
      extra["image_slots"] = slots
  ```

### E3 — DiT splices reference latents at image_slots
- **source_label:** `comfy/ldm/qwen_image21/model.py` `build_sequence`
- **source_url:** https://raw.githubusercontent.com/Comfy-Org/ComfyUI/master/comfy/ldm/qwen_image21/model.py
- **provenance:** curl raw 2026-10-15
- **excerpt:**
  ```python
  def build_sequence(self, x, context, ref_latents, image_slots):
      # text with each reference image spliced in at its slot, target image last
      ...
      slots = (image_slots + [txt.shape[1]] * len(ref_latents))[:len(ref_latents)]
      ...
      for (start, end), img in zip(zip(bounds[:-1], bounds[1:]), ref_latents + [x]):
          ...
  ```
- **related:** `QwenImage21.extra_conds` passes `image_slots` through (`comfy/model_base.py`); parent `QwenImage.extra_conds` maps `reference_latents` → `ref_latents` CONDList.

### E4 — Official node docs: empty latent + optional VAE dual path
- **source_label:** ComfyUI docs TextEncodeQwenImage21
- **source_url:** https://docs.comfy.org/built-in-nodes/TextEncodeQwenImage21
- **provenance:** web_extract 2026-10-15
- **excerpt:** “When a VAE is connected, the same reference latents are attached to both the positive and the negative conditioning… **Empty latent** on the first reference image’s size… Sample at the latent this node returns: any other size shifts the edit.” “When omitted, the reference images condition the result through the text encoder only.”

### E5 — Official image-edit template sampler
- **source_label:** `image_qwen_image_2_1_image_edit.json` subgraph “Image Edit (Qwen Image 2.1)”
- **source_url:** https://raw.githubusercontent.com/Comfy-Org/workflow_templates/main/templates/image_qwen_image_2_1_image_edit.json
- **provenance:** downloaded and parsed 2026-10-15
- **findings:**
  - Node `474` `TextEncodeQwenImage21` ← VAE + images
  - Node `458` `KSampler` widgets: `steps=25`, `cfg=1`, `sampler_name=euler`, `scheduler=simple`, **`denoise=1`**
  - Switch `468` `custom_size=false` → KSampler `latent_image` ← encode node latent (link 724), **not** EmptyLatentImage
  - Models: `qwen_image_2.1_int8_convrot`, `qwen3vl_8b_int8_convrot`, `qwen_image_2.1_vae_bf16`
  - Optional PE: `qwen3.5_9b_qwen_image_2.1_pe_i2i` (off by default)

### E6 — Official tutorial settings
- **source_label:** docs.comfy.org Qwen-Image-2.1 tutorial
- **source_url:** https://docs.comfy.org/tutorials/image/qwen/qwen-image-2-1
- **provenance:** web_extract 2026-10-15
- **excerpt:** All three workflows: steps 25, cfg 1, euler, simple. “At cfg 1 ComfyUI skips the negative conditioning pass.” Edit prompting uses `<image1>`, `<image2>`, …; `image_1` is the image being edited. Published pipeline ~40–50 steps; simple local edits can work at 4–8 steps.

### E7 — Template note text (embedded)
- **source_label:** same edit JSON MarkdownNote
- **source_url:** (E5)
- **excerpt:** “image_1 is the edit target. The rest are references.” “cfg: keep 1 for the Qwen Image 2.1 official path.” “custom_size off: canvas comes from the encode latent (image_1).”

### E8 — KSampler denoise semantics (Comfy core)
- **source_label:** `nodes.py` KSampler tooltip + `common_ksampler`
- **source_url:** https://raw.githubusercontent.com/Comfy-Org/ComfyUI/master/nodes.py
- **provenance:** curl 2026-10-15
- **excerpt:** denoise tooltip: “lower values will maintain the structure of the **initial image** allowing for image to image sampling.” That assumes a structured init latent. Empty zeros do not provide photographic structure.

### E9 — Symptom ↔ mechanism mapping (Stokes)
- **source_label:** task body t_4a3a2a9f + operator note
- **source_url:** kanban://swarm-ops/t_4a3a2a9f
- **provenance:** kanban_show + OOB operator note 2026-10-15
- **support note:** Reported settings: Qwen-Image-2.1, denoise 0.72, prompt with desert sunset / golden hour / warm desert light → output flock of birds, reference not preserved. Matches blank-canvas + prompt-dominated regime.

### E10 — Separate failure mode: MPS VAE ref encode corruption
- **source_label:** Comfy-Org/ComfyUI#16433
- **source_url:** https://github.com/Comfy-Org/ComfyUI/issues/16433
- **provenance:** web_search snippet 2026-10-15
- **support note:** On Apple MPS, corrupted `vae.encode` inside `TextEncodeQwenImage21` yields coherent images that **ignore references and follow text** — same *symptom class* as Stokes, different root (device bug). AI-box CUDA path should not hit this; included to prevent mis-attribution.

### E11 — Community dual-encoding explanation
- **source_label:** comfy.icu Text Encode Qwen Image 2.1 Smart Model Loader writeup
- **source_url:** https://comfy.icu/node/Text-Encode-Qwen-Image-2-1-Smart-Model-Loader
- **provenance:** web_search 2026-10-15
- **excerpt:** Each reference is handled twice: RGB → vision tower; full image → VAE `reference_latents`. Output latent is 64-ch @ 16×; “feed the sampler this latent, because any other size shifts the edit.”

---

## findings (narrative)

### 1. What the code actually does

```
LoadImage (photo)
    │
    ▼
TextEncodeQwenImage21
    │  images_vl[]  → tokenize (VL sees RGB over white)
    │  if vae:
    │       ref_latents[] = vae.encode(RGBA)
    │       keep_vision = False   ← strips VL tokens after encode
    │       conditioning += reference_latents
    │       extra.image_slots = positions where vision tokens were
    │  else:
    │       keep_vision = True    ← VL tokens remain (text-only conditioning path)
    │
    ├─ positive / negative ──► KSampler
    └─ latent = ZEROS(1,64,H/16,W/16) ──► KSampler.latent_image   ★ not photo encode
         │
         ▼
QwenImage21 DiT.build_sequence:
    splice each ref_latent into the text sequence at image_slots
    append the (noisy) target latent x last
    text+refs use t=0 modulation (prefix); target uses sampled t
```

So the operator’s one-liner:

> keep_vision = len(ref_latents)==0  and blank zero latent

is **source-accurate**. The refinement is: **vision is swapped for ref latents, not deleted from the model.**

### 2. Why birds

Under native edit, a prompt should read like an **instruction against `<image1>`**, e.g.:

```text
Keep the person, pose, face, clothing, and composition of <image1> unchanged.
Only extend/outpaint the canvas with matching desert dusk light and horizon.
Do not add animals, birds, or new subjects.
```

A prompt that **re-describes a full scene** (“cinematic desert sunset, golden hour, warm desert light, …”) at **denoise 0.72** on an **empty** latent asks the model to **generate a landscape**. Birds are a common desert-sky generative cliché — consistent with prompt dominance, not with “VAE randomly inserted birds.”

### 3. SD-i2i vs Qwen-2.1-edit (cheat sheet)

| Dimension | Classic SD img2img | Qwen-Image-2.1 native edit |
|---|---|---|
| Init latent | VAEEncode(photo) | **zeros** from TextEncodeQwenImage21 |
| Identity lock | low denoise (0.3–0.5) | **reference_latents + instruction**, denoise **1.0** |
| CFG | often 5–7 | **1.0** (neg prompt inert at 1) |
| Prompt style | tags / scene tags | **sentences + `<imageN>`** |
| Vision path | N/A (CLIP) | VL encode then **drop tokens iff VAE refs exist** |
| Multi-ref | uncommon | first-class (up to 10 official / 16 node) |

### 4. Outpainting specifically

Native 2.1 edit is **instruction edit**, not automatic canvas expand. For outpaint-like behavior:

1. Build a larger canvas (pad photo, or set resolution / custom size carefully — mismatched size **shifts** the edit per docs).
2. Put the padded (or original) image on `image_1`.
3. Prompt: preserve `<image1>` subject; fill only the new border region; forbid new subjects.
4. Still sample **denoise=1.0** on the encoder empty latent sized to the canvas.
5. Optional: paint a visible region mark on `image_1` (official docs: Mask Editor paint ends up in RGB and is named by color in the prompt) to localize the fill.

If the Stokes “outpaint” graph was literally SD outpaint (encoded init + denoise 0.72 + noise mask) on Qwen-2.1 weights, that is an **architecture mismatch**.

---

## standardized recommendation (for print-junkie / AI-box)

### A. Canonical identity-preserving edit (DEFAULT)

**Graph**
1. `UNETLoader` → `qwen_image_2.1_*` (int8_convrot OK on 3090-class)
2. `CLIPLoader` type `qwen_image` → `qwen3vl_8b_*`
3. `VAELoader` → `qwen_image_2.1_vae_bf16`
4. Optional `QwenImage21Cache` (device=auto, dtype=default)
5. `LoadImage` → `TextEncodeQwenImage21.images.image_1` (+ more slots as needed)
6. Same VAE → `TextEncodeQwenImage21.vae` (**required** for proper edit)
7. `TextEncodeQwenImage21.positive/negative` → `KSampler`
8. `TextEncodeQwenImage21.latent` → `KSampler.latent_image` (**do not** substitute a hand-built EmptyLatent at a different aspect)
9. `KSampler` → `VAEDecode` → save

**Sampler (match official)**
- denoise: **1.0**
- cfg: **1.0**
- sampler: **euler**
- scheduler: **simple**
- steps: **25** baseline (try 8–12 for tiny local edits; 30–40 if hands/fine detail fail)

**Prompt contract**
- Instruction, not scene essay.
- Name refs: `<image1>`, `<image2>`, …
- Explicit preserve list (identity, pose, wardrobe, background regions to keep).
- Explicit forbid list when doing environment work (`no birds, no animals, no extra people, no new objects in the sky`).
- Avoid re-captioning the whole photo as if t2i.

**Resolution**
- Prefer template default: `resolution=0` (keep ref size, round to 32) for 1:1 edit.
- For outpaint canvas growth: resize/pad **before** encode so `image_1` already is the target canvas; keep sampler latent from the encode node.

### B. What NOT to do (Stokes anti-patterns)

1. ~~denoise 0.72 on empty Qwen latent expecting SD-style preserve~~  
2. ~~Scenic t2i prompt without `<image1>` preserve language~~  
3. ~~Hand EmptyLatentImage at wrong aspect while refs are another size~~  
4. ~~cfg 5–7 “because SD”~~ (docs: cfg 5 degrades badly; cfg 1 is the published path)  
5. ~~Omit VAE on TextEncodeQwenImage21 and expect full appearance lock~~ (falls back to VL-only conditioning)  
6. ~~Assume `keep_vision=False` means “no image conditioning”~~ — it means “use ref latents instead of VL tokens”

### C. Experimental / non-default: true pixel-init refine

Only if product needs SD-like “tweak this exact latent”:

- `VAEEncode(photo)` → `KSampler.latent_image`
- Still run `TextEncodeQwenImage21` with same photo on `image_1` + VAE (so `reference_latents` exist)
- Start denoise experiments **low** (0.25–0.45) **in addition to** ref conditioning
- Treat as **unsupported-by-template**; A/B against native denoise=1 edit
- Do **not** ship this as the default Stokes path without eval

### D. Prompt patch for the failed Stokes job (drop-in)

Replace desert-sunset scene prompt with:

```text
Edit <image1> only. Preserve the exact person, face, hair, body, clothing, pose, camera framing, and all existing background structure from <image1>.
If extending the frame, continue the existing environment and lighting coherently.
Do not add birds, animals, people, vehicles, or new foreground subjects.
Do not replace the sky with a different scene.
Subtle warm late-day light is allowed only as a grade, not as a full relight or new landscape.
```

Neg (only matters if cfg>1): `birds, flock, animals, text, watermark, extra limbs, distorted face`

### E. Minimal acceptance tests (for implementer)

| # | setup | expect |
|---|---|---|
| T1 | Official edit graph, denoise 1, prompt “keep `<image1>` unchanged” | near-identity / tiny drift |
| T2 | Same + “change shirt color to red, keep face/pose” | local edit, face stable |
| T3 | Stokes old graph denoise 0.72 + scenic prompt | reproduce birds / prompt dominance (control) |
| T4 | Native graph + Stokes photo + instruction outpaint prompt | no bird swarm; subject preserved |
| T5 | Native graph but **disconnect VAE** | weaker lock / VL-only path (document delta) |
| T6 | Native graph, wrong EmptyLatent aspect | shifted / broken edit (docs warning) |

---

## gaps

1. **Live AI-box Comfy checkout hash** not read this run (no DNS/`ai-box` from research container). Behavior matched to Comfy master + 0.37.0-cited issue; if AI-box pins an older pre-2.1 node fork, re-diff `nodes_qwen.py` on box.
2. **Exact Stokes workflow JSON / API graph** for the bird run was not on mounted volumes; conclusion uses task-stated denoise/prompt and architecture-level proof. Attach the failing workflow JSON on retry if anything still diverges after switching to native edit.
3. **No pixel A/B** executed here (research-only task). Print-junkie should run acceptance tests T1–T4 on box.
4. **Quantization / GGUF vs int8_convrot** quality delta not evaluated.
5. Whether Stokes currently feeds **VAEEncode init** vs **empty latent** into KSampler is still an open runtime fact — either way, denoise 0.72 + scenic prompt is wrong for native edit; if they already use empty latent, C6 applies directly; if they VAEEncode, they are on path C-experimental without ref-latents discipline.

## confidence

- **level:** high
- **rationale:** Core mechanism is literal source (`keep_vision = len(ref_latents)==0`, `torch.zeros` latent, vision-token drop, DiT `build_sequence` splice) plus official template denoise=1. Symptom mapping to birds is mechanistic inference from prompt+settings (medium-high), not a reproduced pixel trial.

## recommended_next_actions

1. **print-junkie:** Retarget Stokes Qwen-Image-2.1 “i2i/outpaint” to **native edit graph** (section A); default denoise **1.0**, cfg **1**, instruction prompts with `<image1>`.
2. **print-junkie:** Re-run Stokes photo with prompt patch (section D); compare to bird baseline.
3. **print-junkie:** Add guardrails in API: if model family is `qwen_image_2.1` and mode is edit, **reject or rewrite** SD-style denoise∈(0.4,0.9) + scene-caption prompts; force encode-node latent.
4. **optional:** Store `qwen21_outpaint_edit.json` as the known-good template (task already points at `~/.hermes/profiles/print-junkie/skills/qwen-image/scripts/qwen21_outpaint_edit.json` — verify it matches section A, not SD-i2i).
5. **do not** prioritize “lower denoise to 0.3 on empty latent” as the primary fix.
6. If after native edit the ref is still ignored on CUDA, then debug VAE/ref_latent plumbing (wrong VAE, refs not connected, resolution mismatch) — not denoise folklore.

---

## appendix for Julio / Huly (optional drain)

- Project context: print-junkie / local image gen / Qwen-Image-2.1
- Outcome: root cause verified in ComfyUI source; standardized edit path specified
- Artifact: `/home/ice/know/research/qwen-image-21-i2i-birds-root-cause-t_4a3a2a9f.md`

---

## appendix

| Item | Value |
|---|---|
| Root cause | Blank sampler latent + native ref-latent edit path misunderstood as SD-i2i; scenic prompt @ denoise 0.72 → text dominance (birds) |
| `keep_vision` | `False` when VAE ref latents exist — strips VL tokens so DiT uses `reference_latents` |
| Official denoise | **1.0** (not 0.72) |
| Official cfg | **1.0** |
| Primary fix | Native `TextEncodeQwenImage21` edit graph + instruction prompt |
| Secondary | Prompt hygiene; API guardrails |
| Non-fix | “Just lower denoise on empty latent” as default |
