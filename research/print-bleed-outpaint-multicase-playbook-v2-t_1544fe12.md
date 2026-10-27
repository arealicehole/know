# Print bleed outpaint multi-case playbook v2 — Q297 autopsy + fail-closed routing

- **task_id:** `t_1544fe12`
- **date:** 2026-10-27
- **agent:** Geraldo v2.2 (geraldov21)
- **skill:** `print-bleed-outpaint` (symlinked onto geraldov21 from print-junkie after prior crash)
- **related:** `research-print-outpaint-exec-t_b7a11aaa.md` (2026-09-30 thin-ring exec playbook)
- **lab host:** AI Box ComfyUI `http://100.99.186.66:8188` (v0.37.0, RTX 3090)
- **lab scratch:** `/srv/scratch/t_1544fe12/`

---

## objective

Autopsy the Q297 Autism Awareness handout bleed failure (cork/stone/matte outer frame; center OK after lock) and produce a **multi-case** fail-closed playbook that routes solid-color edges vs busy photographic edges, with mandatory QC gates, Procedure E (literal motif) placement, and engine order Qwen-Image-2.1 → FLUX.1-Fill → InstantX (still blocked).

## summary

Q297-class failure is primarily an **input-conditioning / pad-seed** problem, not a “Qwen cannot outpaint” problem.

1. **Transparent / cream / white pad rings** bias generative models toward frame/matte priors or void letterbox. Alpha-0 pads collapse to black under naive RGB flatten (`RGBA→RGB` drops alpha as black) or to white under composite-on-white. Both are hostile seeds for continuous scene bleed.
2. **Solid-edge art** (navy/black posters, neon CLS, flat brand fields) should **not** ask the model to invent a scene. Prefer **deterministic edge-extend** (edge-pixel replicate / solid scene-color pad) and only optional low-denoise join dissolve. Full denoise on a pre-seeded navy ring can invent brown “cork/stone” texture (reproduced in lab: `solid_qwen_edge` FAIL, edge RGB ≈ 109,101,71 vs navy 3,19,61).
3. **Busy photographic edges** need a **real edge seed** (edge-match or sampled scene-color pad) + scene prompt. Alpha-0/black pad on desert art produced **black letterbox** (`busy_qwen_alpha` FAIL, edge lum 6.6 vs source edge ~185). Edge-match + Fill both PASS on busy fixtures.
4. **FLUX.1-Fill** with `ImagePadForOutpaint` (24 px, feather 8, guidance 30, cfg 1, steps 20) was the **most color-faithful** solid-edge engine in this lab (dist 8.2 to navy) and strong on busy (dist 8.3). Keep as default fallback when Qwen invents frames.
5. **InstantX Qwen CN-Inpaint remains blocked** on AI Box: controlnet list has only SDXL canny/depth/union — no `Qwen-Image-InstantX-ControlNet-Inpainting.safetensors`.
6. **Original Q297 job files were not mounted** into this Geraldo container (`/srv/storage/onedrive/...` and print-junkie scratch not visible). Autopsy of the live 2400×3600 poster uses **operator brief + CLS description + controlled navy solid fixture**. Lab claims about Q297 geometry are **partial** until those files are attached; mechanism claims are **supported** by live Comfy runs.

### Lab pass table (locked keep-zone, ring QC vs source edge)

| Exp | Setup | Edge mean RGB | Dist→src edge | PASS | Notes |
|-----|-------|---------------|---------------|------|-------|
| solid_edge_match_only | deterministic edge replicate, no AI | 11,25,62 | **0.5** | **Y** | Primary solid path |
| solid_fill | FLUX.1-Fill pad24 feather8 | 6,20,61 | **8.2** | **Y** | Best generative solid |
| solid_qwen_scene | Qwen + solid navy pad denoise1 | 1,19,66 | 13.3 | **Y** | Good |
| solid_qwen_alpha | Qwen + black flatten of alpha0 | 0,13,56 | 18.4 | **Y*** | Navy-ish by accident; **hostile on busy** |
| solid_qwen_white | Qwen + white composite pad | 0,17,63 | 14.4 | **Y*** | Model filled navy this seed; **still banned seed** (frame prior) |
| solid_qwen_edge | Qwen + edge seed + denoise **0.35** | **109,101,71** | **123.5** | **N** | **Brown cork/stone invent** — matches Q297 symptom class |
| busy_fill | FLUX.1-Fill | 205,151,93 | **8.3** | **Y** | Best busy generative |
| busy_qwen_edge | Qwen + edge-match denoise1 | 232,178,108 | 36.3 | **Y** | Warm sky continued |
| busy_qwen_scene | Qwen + sampled solid pad | 188,124,61 | 48.4 | **Y** | Acceptable |
| busy_qwen_alpha | Qwen + black pad | **9,6,4** | **266** | **N** | **Black letterbox** |

\* “Pass” on navy-like mean does **not** authorize white/alpha0 seeds — they remain fail-closed banned inputs because they are the documented production failure mode (cream/white frame prior) and destroy busy edges.

---

## plan

```yaml
objective: Q297 autopsy + multi-case outpaint playbook v2 with fail-closed QC
success_criteria:
  - root-cause chain for cork/matte ring with evidence
  - case router: solid vs busy vs motif vs dual-margin vs contain
  - engine order + hyperparams per case
  - mandatory QC gates (programmatic)
  - InstantX status
  - durable report at know/research + kanban complete
assumptions:
  - Q297 source files not in container; use brief + synthetic navy CLS-like fixture
  - AI Box :8188 is the only generative host
  - Skill scripts may be re-materialized under /srv/scratch when profile FS is virtualized
tasks:
  - T1: inventory skill + prior playbook + Comfy weights
  - T2: build solid + busy fixtures; pad modes alpha0/white/edge/scene
  - T3: run Qwen-2.1 + FLUX.Fill matrix on AI Box
  - T4: lock keep-zone + ring QC metrics
  - T5: write playbook v2 + publish
open_questions:
  - exact Q297 v2 seed/prompt/pad code path (operator files not mounted)
  - whether production prep used white composite vs cream brand pad vs alpha0
```

---

## claims

### claim_id: C1
- **statement:** Transparent alpha-0 pad corners are RGBA(0,0,0,0); naive `Image.convert("RGB")` flattens them to black; white composite flattens them to white. Both are non-scene seeds.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E1]

### claim_id: C2
- **statement:** On solid navy poster fixtures, FLUX.1-Fill thin-ring outpaint (pad 24, feather 8, guidance 30, cfg 1, steps 20) continues navy with edge-mean distance ≈8 RGB to source edge and no white/tan frame flags.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E2]

### claim_id: C3
- **statement:** Qwen-Image-2.1 edit with edge-matched navy seed at denoise 0.35 invented a brown outer field (edge ≈109,101,71, dist 123 from navy) — same failure class as Q297 cork/stone frame.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E3]

### claim_id: C4
- **statement:** Busy desert edges with alpha0→black pad + Qwen produce black letterbox rings (edge lum ≈6.6, dist 266); edge-match or scene-color pad + Qwen/Fill restore continuous warm edges.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E4]

### claim_id: C5
- **statement:** Deterministic edge-pixel replicate (no AI) is the lowest-risk solid-edge bleed method (dist 0.5) and should be Case S0 before any generative call.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E5]

### claim_id: C6
- **statement:** InstantX Qwen ControlNet Inpainting weights are not installed on AI Box as of this run; only SDXL controlnets are present.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E6]

### claim_id: C7
- **statement:** Q297 production failure (cork/matte ring, good center after lock) is consistent with (a) non-scene pad seed + full generative rewrite of ring and/or (b) low-denoise/join pass that still invents texture on flat fields; lock re-paste correctly preserves center and cannot fix ring.
- **status:** partial
- **confidence:** medium
- **evidence_refs:** [E3, E7]
- **note:** Exact Q297 v2 tensors not in container; mechanism triangulated from brief + lab.

### claim_id: C8
- **statement:** Keep-zone re-paste yields bit-identical center (keep MAD 0.0 in all locked lab runs) and must remain mandatory after every generative pass.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E8]

### claim_id: C9
- **statement:** Default engine order remains Qwen-Image-2.1 edit → FLUX.1-Fill → InstantX, but **Case S (solid)** should prefer Fill or deterministic edge over Qwen denoise invent; **Case B (busy)** may use either Qwen or Fill once pad is edge-seeded.
- **status:** supported
- **confidence:** high
- **evidence_refs:** [E2, E3, E4, E5]

---

## evidence_index

### E1 — Alpha flatten demo (lab)
- **source_label:** lab PIL probe
- **source_url:** `file:///srv/scratch/t_1544fe12/runs/solid_alpha0/padded_rgba.png`
- **provenance:** terminal python on AI Box worker container 2026-10-27
- **excerpt:** alpha0 corner RGBA (0,0,0,0) → convert RGB (0,0,0); alpha_composite on white → (255,255,255); edge-match/scene-color corners (3,19,61).

### E2 — solid_fill PASS
- **source_label:** Comfy run solid_fill
- **source_url:** `file:///srv/scratch/t_1544fe12/runs/solid_fill/q297_solid_fill_00001_.png`
- **provenance:** FLUX.1-Fill API graph, prompt_id `cc0541ee-6af2-46de-a831-9f6f19e68e88`, seed 29704; QC results_final.json
- **excerpt:** edge mean [5.6, 20.4, 60.7], dist 8.2, flags [], passed true; ring_hist navy-like black%/navy% dominant, white% 0.

### E3 — solid_qwen_edge FAIL (cork-class)
- **source_label:** Comfy run solid_qwen_edge
- **source_url:** `file:///srv/scratch/t_1544fe12/runs/solid_qwen_edge/q297_solid_qwen_edge_00001_.png`
- **provenance:** Qwen-Image-2.1, seed 29702, steps 28, cfg 1.0, denoise 0.35, edge-matched pad; prompt_id `73cdfa60-1059-4e6a-ac27-ee18c2149389`
- **excerpt:** edge mean [109.0, 101.3, 70.6], dist 123.5, flag solid_edge_far / brown_cork class; mid_top (138,126,74).

### E4 — busy alpha FAIL vs edge/Fill PASS
- **source_label:** busy_qwen_alpha / busy_qwen_edge / busy_fill
- **source_url:** `file:///srv/scratch/t_1544fe12/qc/results_final.json`
- **provenance:** seeds 29707 / 29705 / 29706
- **excerpt:** busy_qwen_alpha passed false black_letterbox dist 266; busy_qwen_edge passed true dist 36.3; busy_fill passed true dist 8.3.

### E5 — deterministic edge-match only
- **source_label:** solid_edge_match padded_rgb
- **source_url:** `file:///srv/scratch/t_1544fe12/runs/solid_edge_match/padded_rgb.png`
- **provenance:** prep_outpaint_canvas.py --pad-mode edge-match
- **excerpt:** edge dist 0.5 to source, passed true, no AI.

### E6 — InstantX missing
- **source_label:** Comfy /models/controlnet
- **source_url:** `http://100.99.186.66:8188/models/controlnet`
- **provenance:** curl 2026-10-27
- **excerpt:** `["controlnet-canny-sdxl-1.0-fp16.safetensors", "controlnet-depth-sdxl-1.0-fp16.safetensors", "controlnet-union-sdxl-1.0-promax.safetensors"]` — no InstantX Qwen inpaint.

### E7 — Operator brief (Q297 symptoms)
- **source_label:** kanban task body / attachment metadata
- **source_url:** kanban://research/t_1544fe12
- **provenance:** print-junkie handoff 2026-10-27; attachment `q297-geraldo-outpaint-brief.md` (7615 B) listed but path not mounted in container FS
- **excerpt:** Q297 v1/v2 cork/stone/matte outer ring; center OK after lock; 2400×3600→2448×3648; navy RGB(3,19,61); reject white/cream/cork frames.

### E8 — keep MAD 0 after lock
- **source_label:** lock pass measurements
- **source_url:** `file:///srv/scratch/t_1544fe12/qc/results.json`
- **provenance:** lab lock script
- **excerpt:** keep_mad 0.0 on all locked runs when keep-zone re-pasted.

### E9 — Weights present for A/B
- **source_label:** Comfy diffusion_models
- **source_url:** `http://100.99.186.66:8188/models/diffusion_models`
- **provenance:** curl
- **excerpt:** includes `qwen_image_2.1_int8_convrot.safetensors`, `flux1-fill-dev.safetensors`; TE `qwen3vl_8b_int8_convrot`; VAE `qwen_image_2.1_vae_bf16` + `ae.safetensors`.

### E10 — Prior thin-ring playbook
- **source_label:** t_b7a11aaa research
- **source_url:** `file:///home/ice/know/research/research-print-outpaint-exec-t_b7a11aaa.md`
- **provenance:** Geraldo 2026-09-30
- **excerpt:** execution > model for 19–24 px rings; Fill guidance 30 / cfg 1 / denoise 1 / steps 20; always re-paste; InstantX download path documented but optional.

---

## Playbook v2 — fail-closed multi-case router

### Global hard rules (unchanged + strengthened)

1. **No PIL edge-smear / row-extrude** as final bleed.
2. **Never ship raw model pass** — lock canvas @ 300 DPI + **re-paste keep-zone**.
3. **Pad ring ≈ bleed only** (19 px @0.0625″/300; prefer **24** multiple of 8). No 400 px tutorial pads.
4. **Banned pad seeds:** alpha-0 alone, cream brand void, pure white matte, pure black void on non-black scenes.
5. **Required pad seeds:**  
   - solid field → edge-pixel replicate **or** solid `pad-rgb` = sampled outer mean  
   - busy photo → edge-pixel replicate **or** sampled edge mean solid as starting ring  
   - filigree/line-art → Procedure E literal motif strips
6. **Qwen sampler:** cfg **1.0**, steps 20–28 (40 quality). Never cfg 2+.
7. **Host:** AI Box `:8188` via `ai-box-queue` when worker up. Tag `AIBOX-QWEN21` / `AIBOX-FILL`.
8. **QC fail-closed** before deliver (gates below). No vibes-only ship.
9. **Outpaint is not art repair** — pre-QC source for truncated type/subjects.
10. **InstantX** only after A+B lose **and** weights exist (currently blocked).

---

### Case classifier (run before any Comfy job)

```text
measure source outer band (2–4 px):
  mean_rgb, lum, var, gradient_energy

if art has continuous line-art/filigree to edge requiring stroke continue:
  → CASE M (motif / Procedure E)
elif subjects flush L/R and must terminate inside frame:
  → CASE F (contain — Procedure F; do not re-paste edge-flush keep)
elif Frank named bleed+safe both:
  → CASE V (dual-margin explicit-pad 38 @0.0625+0.0625)
elif lum < 50 AND var < threshold AND gradient_energy low (flat field):
  → CASE S (solid edge)
else:
  → CASE B (busy / photographic / textured scene)
```

Practical solid detector: corner samples within ~25 RGB of each other and mean lum < 60 (navy/black/maroon fields) **or** user tags “flat background poster”.

---

### CASE S — Solid-edge (Q297 class: navy/black/neon-on-flat)

**Goal:** extend the **same flat color** past trim. Do **not** invent material (cork, stone, wood, paper, concrete).

**Order (fail-closed):**

| Step | Action | When ship |
|------|--------|-----------|
| **S0** | Deterministic edge-pixel replicate (or solid pad-rgb = exact field color) into bleed; **no model** | If vision/ring QC pass → **SHIP** as print-ready after lock/guides. Preferred for pure flat. |
| **S1** | FLUX.1-Fill: unpadded trim → `ImagePadForOutpaint` L/R/T/B=24, feather **6–12**, guidance **30**, steps **20**, cfg **1**, denoise **1**, euler; prompt = continuous flat color only | If S0 soft join or vendor wants generative grain |
| **S2** | Qwen-2.1 only if S1 weak: input must already be **scene-color or edge-matched RGB** (never white/alpha0). denoise **1.0** full ring OK on solid color pad; **avoid** “join dissolve” denoise 0.3–0.5 on flat navy (lab: invents cork). cfg 1, steps 20–28 | |
| **S3** | InstantX — blocked until weights | |

**Prompts (solid):**
- Positive: `continuous flat <color name> field matching the existing background, same RGB, no texture change, no border, no frame, no matte, no cork, no stone, no wood, no paper edge`
- Negative: `frame, matte, border, parchment, cork, wood, stone, concrete, white margin, cream margin, polaroid, collage, new text`

**Q297-specific numbers:**
- Field RGB ≈ `(3, 19, 61)` navy
- Geometry example: trim 2400×3600 → full 2448×3648 @ 19 px; lab used 800×1200 → 848×1248 @ 24 px

**Do NOT:**
- Cream/white pad “for the model to fill”
- Procedure E low-denoise invent on empty cream (that is motif-only)
- Mirror virtual-pixel whole plate

---

### CASE B — Busy photographic / textured scene

**Goal:** continue sky/sand/wall/asphalt/gradient with matching light and grain.

**Order:**
| Step | Action |
|------|--------|
| **B0** | Prep with `--pad-mode edge-match` or `solid` sampled from outer mean (lab busy sample ≈ 212,153,90). Save keep_zone. |
| **B1** | Qwen-2.1 default: cfg 1, steps 25, denoise 1.0, scene-descriptive positive, strong no-frame negative. 2–4 seeds. |
| **B2** | FLUX.1-Fill fallback (often **tighter color** — lab dist 8.3 vs Qwen edge 36). Same Fill hyperparams as S1. |
| **B3** | InstantX blocked |

**Prompts (busy):** scene nouns only (`desert sunset sky and sand continue…`). Empty positive if model starts framing.

**Do NOT:** alpha0/black pad (lab → black letterbox).

---

### CASE M — Motif / filigree / line-art (Procedure E)

**When:** existing strokes must continue; cream gutters left by bring-in.

1. Seed ring from **real edge pixels** of a full-bleed master (not cream).
2. Hard paste keep; optional thin feather 12–18 px.
3. Ship literal extend if QA passes.
4. Optional Qwen join dissolve: VAEEncode seed, denoise **0.28–0.45**, cfg 1 — **only** to dissolve join; forbid new pattern.
5. Always re-paste KEEP after.

**Not for solid navy** — solid uses Case S0/S1.

---

### CASE V — Dual-margin (0.0625 safe + 0.0625 bleed)

1. Raster art into **safe** box (e.g. 3×4 → 862×1162).
2. `explicit-pad` 38 px each side → full 938×1238.
3. Route S or B on the padded plate.
4. Lock with `--bleed-in 0.0625 --safe-in 0.0625`.

---

### CASE F — L/R subject contain (Procedure F)

- Scale-in keep so it **never** reaches canvas L/R; model completes cluster edges; **hard-clear** outer ~10% ; **do not** re-paste edge-flush keep.
- Separate from Q297 (Q297 center was fine).

---

### Engine hyperparams (cheat sheet)

| Engine | Key settings |
|--------|----------------|
| **Qwen-Image-2.1** | UNET `qwen_image_2.1_int8_convrot` + TE `qwen3vl_8b_int8_convrot` + VAE `qwen_image_2.1_vae_bf16`; TextEncodeQwenImage21 resolution=0; euler/simple; **cfg 1.0**; steps 20–28; denoise 1.0 (ring) or 0.28–0.45 (motif join only) |
| **FLUX.1-Fill** | UNET `flux1-fill-dev` + DualCLIP clip_l+t5xxl_fp8 + VAE ae; DifferentialDiffusion; FluxGuidance **30**; InpaintModelConditioning noise_mask true; ImagePadForOutpaint pad **24** feather **8**; euler; steps **20**; cfg **1**; denoise **1** |
| **InstantX** | **BLOCKED** — download `Qwen-Image-InstantX-ControlNet-Inpainting.safetensors` (Apache, ~4.23 GB, SHA256 `49c01aafe6545c1f6e7627a724c8fb13357fe74efee235622158f0a8f30e5458`) per t_b7a11aaa |

---

## Mandatory QC gates (fail-closed)

Implement before `kanban_complete` / customer file. Lab reference: `/srv/scratch/t_1544fe12/qc/bleed_qc.py` + `results_final.json`.

### Gate G0 — Input seed (pre-Comfy)
- [ ] `padded_rgb` corner is **not** (255,255,255) unless source edge is white
- [ ] `padded_rgb` corner is **not** (0,0,0) unless source edge is black/navy solid Case S
- [ ] For Case S: corner within **ΔE/RGB dist ≤ 25** of field color
- [ ] For Case B: corner within **dist ≤ 40** of source edge mean **or** edge-match mode
- [ ] meta.json has keep_origin, output_size, pad, pad_mode
- [ ] zero unsubstituted `PARAM_*` in workflow

### Gate G1 — Ring color (post-gen, pre-lock OK too)
- [ ] **white_ring:** ring mean lum > 230 and low var → **FAIL**
- [ ] **black_letterbox:** ring lum < 25 while source edge lum > 60 → **FAIL**
- [ ] **brown_cork_frame:** on solid dark sources, ring lum > 55 and (R−B)>40 and far from field → **FAIL**
- [ ] **tan_parchment_frame:** strong tan jump vs source on non-tan art → **FAIL**
- [ ] Case S: ring mean dist to field RGB ≤ **40** (target ≤ 20)
- [ ] Case B: ring mean dist to source edge ≤ **70** (target ≤ 40)

### Gate G2 — Keep integrity (post-lock)
- [ ] keep-zone re-pasted
- [ ] keep MAD ≤ 2 (lab achieved 0)
- [ ] output size exact full canvas; DPI 300

### Gate G3 — Geometry
- [ ] cut guide cyan inset = bleed_px from outer
- [ ] safe guide green inset = bleed+safe from outer
- [ ] no guides on vendor print file

### Gate G4 — Vision spot (TOP and BOTTOM rings)
- [ ] continuous scene OR continuous flat field
- [ ] no hard join line at old art edge
- [ ] no new matte/parchment/cork/wood
- [ ] type inside safe, bit-identical

**Any G0–G3 fail → do not ship; change case/engine/seed and rerun.**

---

## Q297 autopsy (mechanism)

```text
Approved flat (navy RGB ~3,19,61, neon center)
        │
        ▼
Pad bleed ring ── if alpha0/cream/white ──► hostile seed (black or white void)
        │                                   model prior: matte / frame / letterbox
        │         if edge-match + wrong denoise (0.35 invent)
        │                           ──► brown cork/stone texture in ring  [lab E3]
        ▼
Qwen/Fill generative ring
        │
        ▼
Lock + re-paste keep ──► center bit-perfect (good) ; ring unchanged (bad ring stays)
        │
        ▼
Human sees: cork/matte frame, OK poster center  == Q297 report
```

**Correct Q297 path:**
1. Classify **Case S**.
2. **S0** edge-replicate navy into 19–24 px ring → QC → often ship.
3. Else **S1 Fill** on trim with pad24 feather8, prompt flat navy only.
4. Lock + keep re-paste + CMYK TIF.
5. Never cream pad; never ask model to “design a border”.

---

## Skill patch recommendations (for print-junkie)

1. `prep_outpaint_canvas.py`: add `--pad-mode {alpha0,solid,edge-match}` (lab prototype under `/srv/scratch/t_1544fe12/skill/scripts/prep_outpaint_canvas.py`); default **edge-match** for production Qwen path; keep alpha0 only for mask generation.
2. Always write `padded_rgb.png` (opaque) for Qwen LoadImage — never rely on alpha.
3. Auto-sample `--pad-rgb` from outer 2 px when solid/edge unknown.
4. Add `scripts/bleed_qc.py` gates G0–G2; exit nonzero blocks deliverable naming `*_print_ready*`.
5. Case router section in SKILL.md pointing at this report.
6. Document **denoise 0.3–0.5 on flat solids = cork invent risk**; reserve low denoise for Procedure E motif joins only.
7. InstantX: still optional; install weights before enabling Procedure C.

---

## gaps

1. **Q297 original assets not readable** in Geraldo Docker (OneDrive + print-junkie scratch not mounted). Cannot pixel-diff v1/v2 production PNGs. Mechanism supported via fixture proxy + brief.
2. Attachment path `.../attachments/t_1544fe12/q297-geraldo-outpaint-brief.md` reported by API but absent on container FS — body recovered from `kanban_show` worker_context.
3. Vision API 429 during run — QC is **metric-first**; human vision still required on TOP/BOTTOM rings at delivery.
4. Single-seed matrix (not 4-seed bakeoff) — directionally strong; production should still multi-seed Qwen on Case B.
5. Feather/pad sweep for Fill (6 vs 12 vs 16) not fully grid-searched this run — defaults 24/8 match prior playbook and passed.
6. Dual-3090 second worker `:8189` not exercised.

## confidence

- **level:** **high** on pad-seed mechanism, Case S vs B router, Fill solid performance, InstantX blocked, QC gates  
- **level:** **medium** on exact Q297 v2 code path / seed  
- **rationale:** Live Comfy evidence for all major branches; production files missing for bit-level autopsy.

## recommended_next_actions

1. **print-junkie:** Re-run Q297 with Case S0 (edge-match navy) → if needed S1 Fill; ship locked `AIBOX-FILL` or deterministic; attach QC json.
2. **print-junkie:** Merge prep `--pad-mode edge-match|solid` + bleed_qc.py into skill; default off alpha0 for Qwen.
3. **ops:** Mount Customer Files or attach Q297 v1/v2 PNGs on follow-up card if bit-level autopsy still desired.
4. **ops (optional):** Install InstantX CN inpaint 4.23 GB to unlock Procedure C.
5. **Do not** treat Qwen as broken for bleed — treat **pad conditioning + case routing** as the product.

---

## appendix A — lab commands (repro)

```bash
WORK=/srv/scratch/t_1544fe12
SKILL=$WORK/skill
HOST=http://100.99.186.66:8188

# prep edge-match solid
python3 $SKILL/scripts/prep_outpaint_canvas.py \
  -i $WORK/fixtures/solid_navy_poster.png \
  -o $WORK/runs/solid_edge_match \
  --mode bleed-from-trim --bleed-in 0.08 --dpi 300 --pad-mode edge-match

# Fill solid
python3 $SKILL/scripts/run_comfy_outpaint.py \
  --host $HOST \
  --workflow $SKILL/templates/flux_fill_outpaint.json \
  --image $WORK/fixtures/solid_navy_poster.png \
  --prompt 'continuous deep navy flat field, no border no frame no cork' \
  --steps 20 --cfg 1 --denoise 1 --guidance 30 \
  --prefix q297_solid_fill --out-dir $WORK/runs/solid_fill
```

## appendix B — file map

| Path | Role |
|------|------|
| `/home/ice/know/research/print-bleed-outpaint-multicase-playbook-v2-t_1544fe12.md` | **This report (durable)** |
| `/srv/scratch/t_1544fe12/qc/results_final.json` | Pass/fail matrix |
| `/srv/scratch/t_1544fe12/qc/ring_hist.json` | Ring pixel class histograms |
| `/srv/scratch/t_1544fe12/runs/*/` | Per-exp gens + locked |
| `/srv/scratch/t_1544fe12/skill/` | Materialized prep/run/templates used |
| `/home/ice/know/research/research-print-outpaint-exec-t_b7a11aaa.md` | Prior thin-ring playbook |

## appendix C — prior playbook delta

| Topic | t_b7a11aaa (2026-09-30) | v2 this task |
|-------|-------------------------|--------------|
| Focus | Thin ring exec, Qwen default | **Case router** solid vs busy vs motif |
| Solid navy | Not primary | **S0 deterministic / S1 Fill first** |
| Pad default | cream/alpha common | **edge-match or scene-color required** |
| Low denoise | Motif join | **Forbidden on flat solid** (cork invent) |
| QC | Checklist prose | **Fail-closed metric gates** |
| InstantX | Download instructions | **Still blocked on box** |
