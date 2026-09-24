# Hotel Lobby COLORS — Local Production Guide (sumi404 shape + AI Box H3/LTX)

**Task:** `t_a9db8a90`  
**Agent:** Geraldo v2.2 (`geraldov21`)  
**Date:** 2026-09-23 (session) / report stamped 2026-09-24 UTC  
**Hardware ground truth (ops, this session):** AI Box Omarchy — 2×RTX 3090, ~31 GiB RAM; video Comfy `:8288` (GPU0) + `:8289` (GPU1) UP under `~/video-prep`; home Fedora 5060 Ti image lane separate  
**Urgency:** High — tonight-runnable recipe, no cloud product path

---

## objective

Map a **local open/self-host pipeline** that recreates the viral **Hotel Lobby COLORS** AI duo format (orange cyclorama booth, hanging condenser mic, two full-body subjects, independent performance motion, vertical 9:16, ~10–15s) using **only weights/nodes already on the AI Box inventory** (MiniMax H3 FL2VA/Ref2VA + turbo LoRAs; LTX-2.5 distilled int8 native AV), with a still-composition handoff from home image Comfy if needed. Reverse public DIY shape from Starrd/LightX/ImageAt/sumi404-class posts; produce copy-paste prompts, Comfy graph outlines, res/duration ladder, ffmpeg audio mux, go/no-go on dual-identity lock, and failure modes for 31 GiB host RAM.

## summary

**Recommended primary path: still-first → H3 Ref2VA turbo-4 (Path A R&D), with LTX-2.5 I2V as Path B ship/fallback.**

Cloud templates (Starrd, LightX, ImageAt/Genjutsu, CapCut effects) all share one shape: **(1) two separate identity photos → one staged dual-subject orange-booth still → (2) multi-subject-capable I2V / motion-transfer → (3) mux real track, export 9:16.** Pure single-subject motion-control fails on duos (one freezes). Pure T2V melts celebrity/likeness identity. sumi404’s public X sample was not recoverable this run (x.com abuse block); reconstruction confidence is **high** from Starrd DIY + LightX + ImageAt + COLORS format ground truth, **medium** on sumi404’s exact private stack.

**Tonight recipe (AI Box):**
1. Compose dual-booth still on home 5060 / image Comfy (FLUX.2-klein multi-ref or Z-Image + IP-Adapter/SDXL) **or** H3-assisted still if already in video tree — 9:16, full-body L/R, gap + hanging mic, matte orange cyclorama.
2. On video worker `:8288` or `:8289`, load **Ref2VA pruned int8** + TE + both VAEs + **ref2v turbo 4-step** LoRA.
3. Graph: dual identity refs as `<Picture 1>` / `<Picture 2>` **plus** booth still as composition lock (or FL2VA first_frame = booth still if identity already baked into still).
4. Generate **5.17s (124f)** draft @ ~768×1344 (9:16 short-edge 768) turbo-4; expect ~3–6 min/3090 class per ops smoke notes, then ladder to 10–15s if identity holds.
5. **Strip H3 native audio** (or run silent-ish prompt) and **ffmpeg-mux owned/licensed audio** — do not ship synthesized “Hotel Lobby” as the real track.
6. For **US client-facing ship**, prefer **LTX-2.5 I2V** from the same still (native AV optional, better license posture) and still mux real audio if lip-sync to the record matters more than generative audio.

**Go/no-go dual celebrity identity with current weights:**  
- **GO for personal R&D / original characters** with strong front+full-body refs and still-first staging.  
- **CONDITIONAL GO for dual likeness** — Ref2VA multi-ref is the right tool (≤9 images); expect identity melt, twinning, and orange bleed without iteration; GitHub #15454 documents multi-speaker voice leakage even when visuals stay distinct.  
- **NO-GO pure T2V** for recognizable faces.  
- **NO-GO commercial client use of real celebrity likenesses** without consent/license — local weights skip cloud blocklists but not law.  
- **H3 Community license** remains a **US territory caveat** for client ship (prefer LTX labeling); personal R&D posture per ops inventory.

---

## plan

```yaml
objective: Local Hotel Lobby COLORS production guide scored to live H3/LTX stack
success_criteria:
  - reverse public DIY method with confidence labels
  - Path A H3 Ref2VA + Path B LTX recipes with node names, prompts, ladder, mux
  - still-first primary decision with fallbacks
  - tonight-runnable day-1 on :8288/:8289 without new cloud deps
  - durable report at know path + kanban attach + findings comment
  - >=8 evidence-backed claims
assumptions:
  - inventory in task body is ground truth (weights present; Wan/Kokoro/SeedVR2 NOT assumed)
  - video workers already UP; do not require restart for research
  - sumi404 exact stack may be thin/paywalled → reconstruct from public DIY
tasks:
  - T1: reverse Starrd/LightX/ImageAt/COLORS format
  - T2: H3 native R2V/I2V/FL2VA docs + turbo LoRAs + multi-subject pitfalls
  - T3: LTX-2.5 I2V/T2V capabilities vs duo format
  - T4: merge prior dual-3090 H3/LTX stack research + image still stack
  - T5: write recipe + claims + publish
open_questions:
  - exact sumi404 private node graph (blocked X fetch)
  - whether local-video-gen-ops skill file is installed on this profile (not found on disk)
  - measured dual-celebrity identity scores on this box (not run this session — recipe only)
```

---

## findings

### 1. What the trend is (format lock)

| Element | Spec |
|---|---|
| Set | Seamless **matte orange cyclorama** (wall+floor continuous), no furniture |
| Subjects | **Two**, full-body preferred, side-by-side with clear gap |
| Prop | Single **black/silver condenser mic** hanging on thin cable between them |
| Camera | **Static / locked-off**, eye-level, **no cuts**, no whip pans |
| Aspect | **Vertical 9:16** (cloud samples often 15s; Starrd cites 15s 9:16) |
| Motion | **Independent** performances (lean-into-mic vs bob/wait) — not mirrored clone |
| Audio (cloud) | Real track **“Hotel Lobby” — Unc & Phew** (Quavo+Takeoff), May 20 2022; COLORS performance June 2022, repeatedly reposted 2025–2026 |
| Why “without restrictions” | Cloud celebrity face filters / safety blocks; local open weights do not apply those blocklists |

Sources: Starrd blog DIY (2026-09-18), LightX guide (2026-09-23), ImageAt Genjutsu character-sheet path, task ground truth, BallerAlert/news context (listed; not re-fetched body).

### 2. Reverse of public methods (sumi404-class)

**sumi404 X post** (`https://x.com/sumi404_ai/status/2102771746650673526`): hook claims step-by-step celebrity hotel lobby **without restrictions**; media vertical amplify ~1920×2160. **Not recoverable this run** (x.com anonymous abuse block via jina/r.jina). Treat method as **reconstructed**, not verbatim reverse-engineered.

**Starrd DIY (highest-signal public recipe)** — three steps:

1. **Still composition (multi-ref image model):** two separate photos → one vertical still, both full-body upright, gap, orange cyclorama, hanging mic. Explicit anti-blend / anti-sit / anti-crop prompt (full text in prompt bank below).
2. **Video:** feed still into a model that accepts **image + preferably video reference** and handles **multiple characters natively** (they name **Seedance 2.0**). Dedicated single-subject motion-control tools lock onto the larger figure and freeze the other — **not a prompt bug**.
3. **Audio:** render **silent**, lay **real track** under with mux. Do not ask the video model to invent the record.

**LightX:** same product surface (2 photos → orange booth duo, 9:16); publishes long timed motion prompts with fake lyric stubs for generative lip-sync tools — useful motion language, weak for local mux-of-real-track path.

**ImageAt / Genjutsu path:** different technical shape — **master performance video + two Character Sheets** → replace left/right performers while keeping choreography/camera/timing/soundtrack. First upload = left, second = right. Tips: front + full-body front + full-body back per sheet. This is **video-driven identity swap**, closer to H3 Fun ControlNet / SAM3 character-replacement community graphs than pure I2V.

**Implication for local stack:**  
- Primary local mirror of Starrd = **still-first + multi-subject I2V/R2V**.  
- Optional advanced = **pose/control video of COLORS-like bounce + Ref2VA/ControlNet** (harder; needs clean control extract; Fun ControlNet Union is on H3 docs but not claimed installed as custom beyond native).  
- Do **not** start with pure T2V celebrity names.

**Confidence:** high on Starrd/LightX/ImageAt shape; medium on sumi404 exact tools (likely cloud multi-ref + I2V family, not pure T2V).

### 3. Path A — MiniMax H3 (best quality R&D on this box)

#### Inventory match (from task + prior research t_8bc4e581 + Comfy-Org docs)

| Role | File (on disk per ops) | Comfy folder |
|---|---|---|
| DiT T2V/I2V (FL2VA) | `minimax_h3_fl2va_pruned_int8_convrot.safetensors` | `models/diffusion_models/` |
| DiT R2V (Ref2VA) | `minimax_h3_ref2va_pruned_int8_convrot.safetensors` | `models/diffusion_models/` |
| TE | `qwen3vl_32b_minimax_h3_int8_convrot` and/or `..._nvfp4_awq` | `models/text_encoders/` |
| Video VAE | fp16 and/or int8_convrot | `models/vae/` |
| Audio VAE | fp32 | `models/vae/` |
| LoRA | fl2v turbo 4-step 768p, fl2v turbo 8-step, **ref2v turbo 4-step** | `models/loras/` |

Official Comfy names (docs):  
- Nodes: `MiniMaxH3ImageToVideo` (FL2VA), `MiniMaxH3ReferenceToVideo` (Ref2VA), `MiniMaxH3AddGuide`, VAEDecode + `VAEDecodeAudio` → `SaveVideo`  
- CLIP/TE type: **`minimax`**  
- Duration grid: **17k+5 frames @ 24 fps** (124≈5.17s, 192≈8s, 243≈10.12s, 294≈12.25s, 362≈15.08s)  
- Native canvas: **768 short edge**, multiple of 32; 9:16 ≈ **768×1344** (or Resolution Selector megapixels ~0.4–0.98; avoid busting area cap)  
- Ref limits: **≤9 images, ≤3 videos, ≤3 audio**  
- `ref_image_size`: `match` (fast) vs `max` (stronger ID, up to 2048 short edge)  
- Turbo R2V: Lightning checkbox / turbo_mode → **`minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16`** style LoRA at ~4 steps  
- Turbo FL2VA: 8-step LoRA (`minimax_h3_fl2v_turbo_8step_...`) or community 4-step 768p variants on disk  
- Sampler baseline community: `res_multistep` + simple/beta/normal; turbo short schedules trade motion/audio quality  
- Launch flags critical on 31 GiB RAM: **`--disable-pinned-memory --fp16-intermediates`** (pinned default OOM-kills); optional `--use-sage-attention`  
- **Not open offline:** Context-IR, Regenerate-2K (no true 2K H3 local)

#### Dual-subject identity lock strategy (H3)

**Best quality R&D recipe (hybrid still + refs):**

1. Build **booth plate still** with both subjects already staged (home image stack).  
2. Load **Ref2VA** checkpoint (not FL2VA) for multi-identity jobs.  
3. Wire references in order:
   - `<Picture 1>` = left subject identity sheet (face + full body if possible)  
   - `<Picture 2>` = right subject identity sheet  
   - `<Picture 3>` = composed booth still (composition / wardrobe / set lock)  
   - Optional: extra face crops as 4–5 if melt  
4. Prompt must **tag and assign jobs** explicitly (official R2V rule).  
5. Motion language: independent A/B performance, static camera, no cuts, full body feet on floor.  
6. Audio: either (a) prompt ambient booth room tone only and mux later, or (b) generate disposable AV then ffmpeg replace audio stream. For real track lip-sync, **mux wins** over generative music.  
7. If identity is already perfect in the still and you only need motion: **FL2VA I2V** with `first_frame` = booth still can work with fewer refs — but Ref2VA is safer when faces drift.

**FL2VA-only first_frame path (faster, weaker ID):**  
`MiniMaxH3ImageToVideo` + FL2VA DiT + first_frame=booth still + dual-subject motion prompt. Good for OC / non-likeness; weak for dual celebrity.

**ControlNet / replacement advanced path:** H3 Fun ControlNet Union docs exist (Canny/Depth/HED/MLSD/Pose control video + inpaint). ImageAt’s Genjutsu shape maps here if you extract pose from a COLORS-like master. **Only if nodes/weights present** — task does not list Fun ControlNet weights as on-disk; treat as optional day-7, not tonight blocker.

#### Comfy graph outline — Path A primary (Ref2VA turbo)

```text
[LoadImage x N] identity L, identity R, booth_still
        |
[MiniMaxH3ReferenceToVideo]  (or template "MiniMax H3 R2V")
  - positive: R2V prompt with <Picture 1> <Picture 2> <Picture 3>
  - width/height: 768 x 1344 (9:16)  OR Resolution Selector 9:16 ~0.4–0.98 MP, multiple 32
  - length/frames: 124 (5s draft) → 243/362 later
  - ref_image_size: max for ID drafts; match for speed
  - turbo / Lightning: ON, 4 steps, strength ~1.0 (then tune 0.8–1.0)
        |
[UNETLoader] diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors
[LoraLoaderModelOnly] loras/*ref2v*turbo*4step*.safetensors  (if not baked via turbo_mode checkbox)
[CLIPLoader] text_encoders/qwen3vl_32b_minimax_h3_*  type=minimax
[VAELoader] video VAE + audio VAE
        |
[KSampler / native H3 sampler path per template]
[VAEDecode] video
[VAEDecodeAudio] audio   # keep for preview; strip later for real track
[SaveVideo] mp4
```

UI path: Template Library → Video → **MiniMax H3 R2V**; enable Lightning/turbo.  
API path: export API-format JSON from working graph; `POST http://ai-box:8288/prompt` (GPU0) or `:8289` (GPU1).

**Exclusive gate:** video creative workers already own 82xx. If fat LLM still on `:8088`, ops gate (`creative-start.sh` / stop) may require LLM stop before heavy jobs — **document, do not invent restart** if workers already UP. Do not co-tenant LLM + H3 on same 31 GiB host RAM.

#### Res / duration ladder (H3)

| Tier | Frames (17k+5) | Time @24fps | Res (9:16) | Steps | Purpose | Wall time ballpark 1×3090 |
|---|---|---|---|---|---|---|
| Smoke | 124 | ~5.2s | 576×1024 or 768×1344 | turbo-4 | ID + set check | ~3–6 min (ops smoke class 5s@864×480 turbo-4 ~3–4 min) |
| Draft | 192–243 | ~8–10s | 768×1344 | turbo-4 or 8 | motion independence | ~1.5–3× smoke (superlinear in frames) |
| Hero R&D | 362 | ~15.1s | 768×1344 | 8–20 non-turbo if needed | final local | can run tens of minutes; sampling dominates |
| Avoid | — | — | 2K / 1920×2160 native H3 | — | **not open** offline | N/A — upscale later if ever |

Ops note: open base ceiling ~1344×768 landscape equivalent area; **not** open 2K H3. Vertical 768×1344 is the correct “full quality local” target, not tweet’s 1920×2160.

### 4. Path B — LTX-2.5 distilled int8 (US/client-ship safer)

#### Inventory match

| Role | File |
|---|---|
| Transformer | `ltx-2.5-22b-distilled-transformer-comfy-int8-convrot.safetensors` |
| TE | `gemma4-12b-with-proj-ltx-2.5-comfy-int8-convrot.safetensors` |
| Optional enhancer TE | gemma4_e2b scraps on disk (not primary) |
| VAEs | ltx video + audio bf16 |
| Upscaler | latent spatial x2 bf16 |

Native workflows (Comfy docs): **T2V, I2V, FLF2V** with synchronized audio; Gemma 4 12B TE holds multi-subject prompt structure better than older LTX; optional prompt enhancer (+1–2 min); auto duration; distilled = faster.

#### Can LTX do Hotel Lobby?

| Criterion | Assessment |
|---|---|
| Orange booth + hanging mic look | **Yes** via strong still + I2V prompt (“animate from start image…”) |
| Dual independent motion | **Partial–good** if still already separates bodies; weaker native multi-ref ID lock than H3 Ref2VA (no 9-image Ref2VA equivalent in base LTX templates) |
| Face lock | **I2V keyframe-first** helps composition/ID from still; dual celebrity still harder than H3 R2V multi-ref |
| Lip / body motion | Decent body bounce; lip-sync to **external real track** still wants post mux + accept imperfect mouths OR generative audio (not the record) |
| Native audio | **Yes** — advantage vs silent pipelines |
| License / ship | **Preferred for US client display** per prior stack research (H3 Community excludes US territory without separate grant) |
| Speed | Prior research: ~2.5 min/5s on 3090 int8 class (community) — attractive for drafts |
| MSR (Multiple Subject Reference) | Community/YouTube LTX 2.5 MSR tutorials exist (2026-08); **not verified installed** on this box — do not block tonight on MSR |

**Path B graph outline:**

```text
[LoadImage] booth_still_9x16
[LTX-2.5 I2V template]
  UNETLoader → ltx-2.5-22b-distilled-...int8-convrot
  CLIP/TE → gemma4-12b-with-proj-ltx-2.5-...
  VAE video + audio bf16
  optional latent spatial upscaler x2 after base
  prompt: motion-only continuation; static camera; independent L/R performance
[SaveVideo]
```

**Pros vs H3:** cleaner ship license story; fast distilled; native AV; simpler I2V.  
**Cons vs H3:** weaker dedicated multi-ref identity toolbox; duo celebrity lock more dependent on still quality; multi-speaker control less documented than H3 R2V tags.

### 5. Still-first vs video-first (decision)

| Option | Verdict | When |
|---|---|---|
| **(a) Compose dual booth still → I2V/R2V** | **PRIMARY** | Always for recognizable faces / dual subjects |
| **(b) Pure T2V** | **Fallback only** | OC / non-likeness stress tests; expect identity melt on celebs |
| **(c) Two single-subject clips + composite** | **Last resort** | If multi-subject models twin or freeze one body; hard to match lighting/mic/parallax; use only if (a) fails after 3+ seeds |

**Still production (home Fedora 5060 Ti / image Comfy `~/comfy-prep`):**  
From image-gen research t_40a6a892: **FLUX.2-klein-4B multi-ref edit** (Apache) or **Z-Image-Turbo + IP-Adapter/SDXL** for dual identity composite; hand PNG to AI Box via shared path/rsync. Vertical 9:16 canvas (e.g. 768×1344 or 1080×1920 still — video model will resample to H3 grid).

Do **not** generate the still on AI Box video GPUs while H3 weights resident if RAM is tight — prefer home image lane.

### 6. Audio strategy

| Approach | Use |
|---|---|
| **ffmpeg mux of owned/licensed track** | **Default for authentic Hotel Lobby sound** (matches Starrd DIY advice) |
| H3/LTX native AV | Booth SFX, original ad-libs, temp preview only — **not** a substitute for the record |
| Kokoro / Wan TTS | **Not on disk / not product-default** per inventory — out of scope tonight |
| Generative “cover” audio | Legal/quality mess; avoid for this format |

**ffmpeg one-liners:**

```bash
# Replace generative audio with licensed/owned WAV/MP3 (video copy, audio aac)
ffmpeg -y -i h3_out.mp4 -i hotel_lobby_owned.wav \
  -map 0:v:0 -map 1:a:0 -c:v copy -c:a aac -b:a 192k \
  -shortest -movflags +faststart hotel_lobby_mux.mp4

# If H3/LTX video is longer than track, trim video to audio
ffmpeg -y -i h3_out.mp4 -i track.wav \
  -filter_complex "[0:v]trim=duration=15,setpts=PTS-STARTPTS[v]" \
  -map "[v]" -map 1:a -c:v libx264 -crf 18 -c:a aac -b:a 192k \
  -shortest hotel_lobby_15s.mp4

# Strip audio only (silent plate for later)
ffmpeg -y -i h3_out.mp4 -c:v copy -an h3_silent.mp4

# Loudness / sanity check
ffmpeg -i hotel_lobby_mux.mp4 -af volumedetect -f null - 2>&1 | grep -E 'mean_volume|max_volume'
ffprobe -hide_banner hotel_lobby_mux.mp4
```

**Legal note:** “Hotel Lobby” master is copyrighted. Local guide assumes operator supplies **owned, licensed, or original** audio. Cloud templates that ship the real track are a commercial product license path — not something this report copies.

### 7. Celebrity / “without restrictions” honesty

1. **Cloud blocks ≠ local blocks.** Open weights generally do not implement celebrity denylists the way consumer apps do. That is the technical meaning of “without restrictions.”  
2. **Identity quality ≠ permission.** Skipping a filter does not grant rights to a person’s likeness, trademark, or the musical work.  
3. **Local quality still needs references.** Names in a T2V prompt are not enough. Use **Ref2VA / multi-ref still / IP-Adapter-class** sheets.  
4. **Consent & commercial risk:** personal R&D and **original characters** are the safe technical guide path. Commercial ads with real celebrity likenesses need clearance; keep client ship on **OC / licensed talent / employee likeness with release**.  
5. **Label AI** on social platforms when required.  
6. **H3 license:** Community license territory exclusions (US/EU/UK/KR) documented in prior research — **R&D personal vs US client display** split remains: H3 for R&D quality, **LTX for ship labeling** unless MiniMax grant obtained.

### 8. Concrete day-1 recipe (tonight)

#### Pre-flight

```bash
# On AI Box (ops)
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8288/system_stats
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8289/system_stats
nvidia-smi --query-gpu=index,memory.used,memory.total,utilization.gpu --format=csv
free -h
# If LLM :8088 is fat-resident and RAM <~8 GiB free, stop LLM via creative gate before H3
# ~/video-prep/bin/creative-start.sh   # already UP per inventory — only if needed
```

Confirm files exist under `~/video-prep/ComfyUI/models/{diffusion_models,text_encoders,vae,loras}/` matching inventory names.

#### Step A — Still (home 5060 or any multi-ref image)

Use **Still prompt bank** below with two clean front-facing photos (separate files, full body if possible). Export PNG `booth_L_R_9x16.png`.

#### Step B — Motion (AI Box video Comfy)

1. Open UI `http://ai-box:8288` (or 8289).  
2. Template **MiniMax H3 R2V** (or I2V FL2VA if still is perfect).  
3. Load refs + prompt from **R2V prompt bank**.  
4. Set 768×1344, frames **124**, turbo-4 ON.  
5. Queue 1 seed; if twinning, raise `ref_image_size=max`, add face crops, or switch FL2VA first_frame-only.  
6. Save MP4 → mux audio.

#### Step C — Path B ship check (optional same night)

Reload LTX-2.5 I2V template on the other GPU worker; same still; compare identity hold and motion independence at 5s.

#### Expected wall time (order of magnitude)

| Job | Expect |
|---|---|
| Still multi-ref (5060) | seconds–2 min |
| H3 Ref2VA turbo-4 5s 768-class | ~3–6 min / 3090 |
| H3 15s non-turbo | often 15–30+ min class (prior 15s ~23 min verified community on 3090 offload) |
| LTX int8 5s | ~2–4 min class |
| ffmpeg mux | <2 s |

#### Failure modes & fixes

| Failure | Fix |
|---|---|
| **Identity melt / face morph** | Stronger refs; `ref_image_size=max`; still-first; more face crops; avoid pure T2V; lower motion amplitude in prompt |
| **Both subjects same motion** | Explicit A/B timed beats; “NOT mirrored”; independent verbs; try Ref2VA not single-subject ControlNet; reject single-subject motion tools |
| **One subject frozen** | Classic single-subject motion model failure — switch Ref2VA/LTX I2V multi; ensure both silhouettes large in still |
| **Orange bleed / color shift** | Lock set in still; prompt “matte seamless orange cyclorama edge-to-edge”; reduce style refs |
| **Subjects blend into one body** | Rebuild still with gap; “do not blend”; separate silhouettes; never use one group photo as sole ID |
| **Mic disappears / becomes third arm** | Still must paint mic clearly; prompt “single black condenser on thin cable, not held” |
| **Cropped legs / sitting** | Full-body still + “feet flat on floor, fully upright, NOT sitting” |
| **OOM / worker death on 31 GiB RAM** | `--disable-pinned-memory`; `--fp16-intermediates`; one GPU job only; shorter frames; NVFP4 TE saves RAM but no Ampere speed win; do not enable high-ram/smart-memory flags that worsen pin behavior; stop LLM |
| **Black video** | TE on wrong GPU / multi-GPU pin issues; verify with luma/hash; both VAEs loaded |
| **No audio in file** | Missing audio VAE or VAEDecodeAudio — or intentional silent path |
| **Voice accent leak across subjects** | Known H3 issue (#15454); visuals may be fine; prefer mux real track and ignore generative dialogue |
| **Wrong aspect** | Force 9:16 selector; multiple of 32 |
| **Trying 2K H3** | Impossible offline — upscale externally later if ever (SeedVR2 **not** on disk) |

### 9. Missing weights/nodes (blocking only)

| Item | Status | Blocker? |
|---|---|---|
| H3 FL2VA/Ref2VA/TE/VAEs/turbo LoRAs | On disk per inventory | No |
| LTX-2.5 distilled stack | On disk | No |
| Wan 2.2 full pack | NOT on disk | No (not required) |
| Kokoro product default | NOT wired | No (mux file audio) |
| SeedVR2 / FlashVSR | NOT on disk | No (skip 2K) |
| Fun ControlNet Union weights | Not listed | No for primary; optional |
| Official COLORS master for control video | Not assumed | No |
| MiniMax paid API / Hailuo / Starrd | Explicitly out of scope | — |

**No blocking downloads required for tonight Path A/B if inventory holds.**

---

## prompt bank (copy-paste)

### Still — dual subject booth plate (Starrd-derived, local-adapted)

```text
A photorealistic VERTICAL 9:16 studio photograph of the TWO subjects from the reference photos performing together in a music session booth.

LEFT subject: the first subject, standing TALL and FULLY UPRIGHT — body vertical, balanced, NOT sitting, NOT crouching. If the subject is an animal it stands on BOTH hind legs in a confident human-like posture. Arms or front paws raised and clearly separated from the body at about chest height, as if gesturing while rapping.

RIGHT subject: the second subject, standing TALL and FULLY UPRIGHT on both feet, arms down and clearly separated from the torso, relaxed and confident.

PRESERVE the exact identity of each subject — face, hair, skin tone, breed, fur, markings and clothing — exactly as in their reference photo. Only the pose and the setting change. Do NOT blend the two subjects together.

They stand side by side with a clear gap of empty floor between them, neither overlapping nor touching, each forming ONE clean unobstructed silhouette. BOTH are shown FULL BODY from head to feet, feet flat on the floor, with headroom above and floor visible below. The pair is centered and fills MOST OF THE FRAME HEIGHT.

SETTING: a seamless matte ORANGE studio cyclorama filling the entire background edge to edge — one flat continuous orange wall and floor, no corners, no furniture, no props. A single black condenser microphone hangs on a thin cable from the top of the frame in the gap between the two subjects. Soft even frontal studio lighting with warm orange bounce.

Photorealistic, sharp focus, natural colors. Vertical 9:16. No text, no logos, no watermarks, no other people or animals.
```

### H3 R2V — motion + dual ID (primary)

```text
Overall: Vertical 9:16 COLORS-style music booth performance, one continuous shot, locked-off static camera, no cuts, no zoom, no whip pan.

References:
- <Picture 1> is LEFT subject identity. Preserve face, hair, skin, body, clothing exactly. <Picture 1> drives LEFT identity only.
- <Picture 2> is RIGHT subject identity. Preserve face, hair, skin, body, clothing exactly. <Picture 2> drives RIGHT identity only.
- <Picture 3> is the staged booth composition plate. Keep set, mic placement, wardrobe blocking, full-body framing, and orange cyclorama from <Picture 3>.

Setting: seamless matte orange cyclorama wall and floor edge to edge; single black condenser microphone hanging on a thin cable in the gap between subjects; soft even frontal light with warm orange bounce; no furniture; no logos; no extra people.

Cast and blocking: LEFT = <Picture 1>, RIGHT = <Picture 2>, full body, feet on floor, clear gap, neither overlapping.

Motion (independent, NOT mirrored):
0.0–2.5s — both subjects bounce lightly on the beat, knees loose, shoulders rolling; LEFT leans slightly toward the hanging mic; RIGHT nods and waits.
2.5–6.0s — LEFT performs into the mic space with clear mouth motion and a hand chop on downbeats; RIGHT ad-libs with small shoulder bounce and head nods only.
6.0–10.0s — they swap energy: RIGHT leans into the mic space and performs; LEFT steps half a step back, still bouncing, light ad-libs.
10.0–end — both lean in together on the hook energy, dual head-bop, then settle; mic stays centered; nobody grabs the mic.

Camera: static medium-full shot, eye level, locked tripod, shallow depth, photoreal, 24fps feel.

Audio (generative, disposable): quiet booth room tone and soft body movement only; NO famous song recreation; NO full musical arrangement; optional light ad-lib breaths. Final music will be replaced in post.
```

*(Trim timed sections to match actual frame count — for 5s smoke keep only first two beats.)*

### H3 FL2VA I2V — first_frame only (if still already locks ID)

```text
Use the provided start image as the first frame. Animate a vertical 9:16 COLORS booth duo performance. Static locked camera, no cuts. Seamless matte orange cyclorama, hanging black condenser mic between two full-body subjects. Independent motion: LEFT leans into mic and raps with hand gestures; RIGHT bobs and waits, then they swap energy; never identical mirrored choreography. Feet stay on the floor. Preserve identities, wardrobe, mic, and orange set from the start frame. Soft frontal light. Photoreal. Disposable ambient audio only.
```

### LTX-2.5 I2V

```text
Use the provided start image as the first frame. Continue the scene naturally. Static locked-off camera, no cuts, no zoom. The two subjects in the orange COLORS-style booth perform independently: the left subject leans toward the hanging microphone and raps with clear mouth and hand motion while the right subject bobs and waits, then they trade energy. Keep full bodies, feet on the floor, mic centered, seamless orange cyclorama unchanged. Photoreal, vertical 9:16. Soft studio light. Audio: light booth ambience and performance breaths only.
```

### Negative / avoid list (where negatives exist)

```text
mirrored identical choreography, clone twin motion, face morph, identity blend, fused bodies, sitting, crouching, cropped legs, extra limbs, third person, handheld mic, moving camera, whip pan, hard cuts, jump cuts, green screen edges, furniture, logos, watermark, text overlay, warped orange wall, melted faces, single subject only
```

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | Public cloud Hotel Lobby recipes are still-first dual-photo → multi-subject animate → mux real track | E1 Starrd DIY; E2 LightX; E3 ImageAt | high | supported |
| C2 | Format lock = orange cyclorama, hanging mic, dual full-body, static camera, independent motion, 9:16 ~10–15s | E1; E2; task GT | high | supported |
| C3 | Single-subject motion-control tools fail on duos (one freezes); need multi-character native video models | E1 Starrd “wrong model” tip | high | supported |
| C4 | H3 open local path = FL2VA (T2V/I2V) + Ref2VA (R2V ≤9 img); native stereo AV; 768 short-edge; 17k+5 duration grid; no offline 2K | E4 Comfy H3 overview; E5 H3 native workflows; E6 prior t_8bc4e581 | high | supported |
| C5 | Ref2VA + tagged multi-refs + turbo 4-step LoRA is the correct H3 tool for dual identity Hotel Lobby R&D on this inventory | E5; E4; task inventory | high | supported |
| C6 | Still-first beats pure T2V for likeness; pure T2V melts celebrity identity | E1; E6; ops take | high | supported |
| C7 | LTX-2.5 distilled int8 I2V can do the format from a strong still with native AV; weaker multi-ref ID toolbox than H3 Ref2VA; better US ship license posture | E7 LTX docs; E6 license notes | medium-high | supported |
| C8 | Real track should be ffmpeg-muxed; generative audio is preview-only for this trend | E1 Starrd step 4; audio section | high | supported |
| C9 | Local open weights skip cloud celebrity blocklists but still need strong refs; commercial likeness/music use remains a legal risk | E1 FAQ AI label; legal analysis | high | supported (not legal advice) |
| C10 | 31 GiB host RAM requires `--disable-pinned-memory` (+ fp16 intermediates); dual 3090 does not TP-speed H3 in Comfy | E6 tonyd2wild / prior research | high | supported |
| C11 | Tonight path needs no Wan/Kokoro/SeedVR2; inventory H3+LTX sufficient if files present | task inventory; missing table | high | supported |
| C12 | H3 multi-speaker voice conditioning can leak across subjects even when visuals stay distinct | E8 GitHub #15454 | high | supported |
| C13 | sumi404 exact private stack not recovered (X abuse block); method reconstructed from public DIY | E9 fetch failure | medium | partial |
| C14 | Ops smoke class supports ~3–4 min/5s turbo-4 at 864×480 on 3090; 15s can reach ~20+ min | task inventory; E6 | medium | partial (tiered; not re-bench this run) |
| C15 | Image still lane: FLUX.2-klein multi-ref / Z-Image on home 5060 is the right compositor | E10 t_40a6a892 | medium-high | supported |

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | Starrd Hotel Lobby DIY blog | https://www.getstarrd.app/blog/how-to-make-hotel-lobby-colors-ai-video | curl HTML→text 2026-09-24 | Two photos; still multi-ref prompt; Seedance multi-char; silent then real track; independent motion |
| E2 | LightX Hotel Lobby guide | https://www.lightxeditor.com/blog/how-to-make-hotel-lobby-ai-video/ | jina reader md 2026-09-24 | 2 photos→orange booth; timed motion prompts; Unc & Phew context |
| E3 | ImageAt Genjutsu trend page | https://imageat.com/trends/hotel-lobby-swap-ai-video | curl HTML→text | Master video + 2 character sheets; L/R order; preserves choreography/audio |
| E4 | ComfyUI MiniMax H3 overview | https://docs.comfy.org/tutorials/video/minimax/minimax-h3 | curl HTML→text | Native stereo; R2V/I2V/T2V; MiniMaxH3ImageToVideo / ReferenceToVideo; 768 canvas; Sage Attention |
| E5 | ComfyUI H3 native workflows | https://docs.comfy.org/tutorials/video/minimax/minimax-h3-native | curl HTML→text | File names FL2VA/Ref2VA; turbo 8-step FL2V / 4-step Ref2V LoRAs; ref tags; AddGuide |
| E6 | Prior GIU stack research | `/home/ice/know/research/minimax-h3-community-video-stack-dual-3090-t_8bc4e581.md` | local durable 2026-09-17 | License US exclusion; RAM flags; LTX primary ship; no Comfy TP; timings |
| E7 | ComfyUI LTX-2.5 workflows | https://docs.comfy.org/tutorials/video/ltx/ltx-2-5 | curl HTML→text | T2V/I2V/FLF2V; distilled int8 filenames matching inventory; native AV; multi-subject TE claims |
| E8 | ComfyUI issue #15454 | https://github.com/Comfy-Org/ComfyUI/issues/15454 | GitHub API | Multi-speaker voice leak FL2VA+Ref2VA; visuals OK |
| E9 | sumi404 X status fetch | https://x.com/sumi404_ai/status/2102771746650673526 | jina 403 AbuseAlleviation | Thread body not recovered |
| E10 | Local image gen pick list | `/home/ice/know/research/local-image-gen-aibox-kie-replacement-t_40a6a892.md` | local durable | FLUX.2-klein multi-ref, Z-Image, dual image ports |
| E11 | ComfyUI Wiki H3 guide | https://comfyui-wiki.com/en/tutorial/advanced/video/minimax/minimax-h3 | curl HTML→text | Install table; troubleshooting no audio / OOM / 256p fail |
| E12 | Task body inventory | kanban `t_a9db8a90` | dispatcher | Ports 8288/8289; exact weight filenames; constraints |

## gaps

| gap_id | gap | impact |
|---|---|---|
| G1 | sumi404 full tutorial text/media not fetched | Cannot cite their exact model names; reconstruction used |
| G2 | `local-video-gen-ops` skill file not found on geraldov21 profile disk this run | Relied on task inventory + prior research instead of live skill |
| G3 | No live generation bake-off executed this session (research-only) | Wall times and dual-celeb ID scores are extrapolated |
| G4 | Fun ControlNet / LTX MSR install state unknown | Advanced pose-transfer path not tonight-certified |
| G5 | Exact on-disk LoRA filenames may differ slightly from Comfy-Org doc strings | Operator should `ls models/loras/*ref2v* *fl2v*` |
| G6 | Musical work + likeness legal clearance is operator-side | Guide stays technical |

## confidence

```yaml
level: medium-high
rationale: |
  Format reverse and H3/LTX node/file recipes are well-sourced (Starrd+Comfy docs+prior GIU research).
  Medium deductions: sumi404 thread unrecovered; no on-box generation verify this run;
  dual-celebrity quality is inherently seed- and ref-dependent; license posture is analysis not counsel.
```

## recommended_next_actions

1. **Tonight:** Run smoke **H3 Ref2VA turbo-4 @ 124f 768×1344** on `:8288` with OC or self refs; mux owned audio; log VRAM + wall time.  
2. **Same night:** Parallel **LTX I2V** on `:8289` same still — pick winner on identity independence.  
3. **Still pipeline:** Confirm home 5060 multi-ref graph (klein or Z-Image) exports clean 9:16 booth plates.  
4. **If client ship:** Prefer LTX-labeled outputs; keep H3 in R&D bucket pending MiniMax grant decision.  
5. **Day-7 optional:** Pose extract from a non-copyright-problematic bounce plate + H3 Fun ControlNet if weights added; still not required.  
6. **Do not:** pull Wan/SeedVR2/Kokoro just for this trend; do not call MiniMax paid API; do not chase native 2K H3.

---

## appendix A — quick decision card

```text
PRIMARY: still-first → H3 Ref2VA + ref2v turbo-4 → ffmpeg mux owned audio
SHIP FALLBACK: same still → LTX-2.5 I2V int8 → mux or native AV
AVOID: pure T2V celebs | single-subject motion control | cloud Starrd as product path
PORTS: video :8288 / :8289 only | still on home image Comfy
RAM: --disable-pinned-memory --fp16-intermediates | one heavy job
```

## appendix B — related durable research

- `/home/ice/know/research/minimax-h3-community-video-stack-dual-3090-t_8bc4e581.md`  
- `/home/ice/know/research/local-image-gen-aibox-kie-replacement-t_40a6a892.md`  
- This file: `/home/ice/know/research/research-hotel-lobby-colors-local-t_a9db8a90.md`

---

*End of report — Geraldo v2.2 / GIU / t_a9db8a90*
