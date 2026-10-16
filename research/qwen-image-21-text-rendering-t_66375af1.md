# Qwen-Image-2.1: Text-Rendering Quality — Research Report

- **Task:** t_66375af1 (board: research, [GIU])
- **Requesting agent:** SCOR / Scorcese
- **Date:** 2026-10-15
- **Companion prior work:** `/home/ice/know/research/qwen-image-21-i2i-birds-root-cause-t_4a3a2a9f.md` (i2i strength tuning for the same model)

## Objective

Find actionable techniques for improving text-rendering quality (legibility, spelling, layout) with Qwen-Image-2.1 on ComfyUI, specifically to improve "Stokes Towing" banner rendering. Scope: prompt techniques, ComfyUI workflow settings (steps, resolution/aspect, seeds, LoRA stacks), text-rendering LoRAs (Civitai/HuggingFace), and model-version differences (2.1 vs 2.1 Turbo vs 2.0/2512).

## Success Criteria

- Concrete, copy-ready prompt patterns that reliably render short English strings (e.g. "STOKES TOWING").
- Workflow parameters (sampler, steps, cfg, resolution, aspect) with sources.
- Named, versioned text-rendering LoRAs with base-model compatibility and download links.
- Version comparison: which Qwen-Image variant is best for banner text, and what the local box should load.
- A concrete plan for the Stokes Towing banner case.

## Assumptions

- Local AI Box (host :8188, ComfyUI) was not reachable from this container during research (all probes to 172.17.0.1 / 100.99.186.66 / 192.168.178.5 / host.docker.internal on port 8188 failed) — the box IP list came from the prior birds report; treat it as "unverified this run."
- The box currently has some Qwen-Image-2.1 GGUF (Q8_0 or similar) loaded; exact file unknown.
- "Banner" = wide image, short uppercase words, high contrast, ideally 2048-class width.

## Plan (executed)

- T1 Model/version landscape — done (Qwen blog, ComfyUI docs, Civitai).
- T2 Prompt techniques for exact-text rendering — done (Civitai article, PixelDojo guide, dripxiforge/cliptics).
- T3 Workflow settings (steps/cfg/resolution/aspect) — done (docs.comfy.org Qwen-Image-2.1 page, molbal GGUF card).
- T4 LoRA landscape + base-model compatibility — done (Civitai search).
- T5 AI Box live probe — BLOCKED (no network path from container to host ComfyUI this run).
- T6 Synthesis + banner plan — done below.

## Findings

### F1 — Version landscape (what "Qwen-Image-2.1" actually is)

Qwen-Image-2.1 (open-sourced 2026-09-20) is a **7B single-stream DiT (32 layers) unified T2I + edit model** with native 2K (2048) output, 4-channel (RGBA) VAE, up to 10 reference images, and day-0 ComfyUI support. [E1, E2]

- **Base model**: 25 steps, cfg 1.0, euler, simple scheduler — the official ComfyUI template default. The published pipeline runs 40–50 steps; the template ships 25. [E2, E3]
- **Qwen-Image-2.1 Turbo**: distilled 8-step checkpoint (fixed ManualSigmas schedule). Fast drafts; "finer small print" is the full model's job. [E2, E4]
- **Predecessors**: Qwen-Image-2512 (20B, Dec 2025) and Qwen-Image 2.0 Pro (API-only) are larger and, per multiple guides, stronger at dense/small text; 2.1 is the compact local-friendly option (7.16 GB Q8_0 GGUF fits on the box). [E5, E6]

**Implication for the banner:** for a short two-word banner the 7B 2.1 base model is the right local choice. If text still misbehaves, the upgrade path is the 2512-20B GGUF on a bigger GPU, not more 2.1 LoRAs.

### F2 — Prompt techniques that reliably fix text (highest-leverage lever)

Consensus across all sources (Civitai 2512 guide, PixelDojo 2.1 guide, cliptics, segmind, dreampixelforge):

1. **Put every word that must appear inside double quotes.** This is the single most-cited trick. Model treats quoted strings as exact text to set.
2. **Name the type style and placement for each quoted string**: "bold condensed letters", "classic serif", "large uppercase letters across the top", "small text at the bottom reading '...'".
3. **Keep lines short.** One or two short lines per image. Dense small print is where the model degrades.
4. **High-contrast, clean background behind the text.** A busy scene behind small text wrecks legibility.
5. **Describe layout first, then the image**: name the piece (e.g. "towing company banner"), then the type, then the scene.
6. **Generate multiple seeds and keep the cleanest.** Text rendering is stochastic; 3–5 seeds at the same prompt is the standard fix for one garbled letter.
7. **Negative prompt only matters if cfg > 1** (see F3).

Copy-ready template for the banner:

> A professional towing company banner, large bold uppercase white letters "STOKES TOWING" centered across the top, smaller white text below reading "24/7 TOW SERVICE • (555) 014-2048", clean dark blue gradient background, high contrast, sharp crisp typography, professional signage design

### F3 — ComfyUI workflow settings that matter for text

From the official ComfyUI docs for Qwen-Image-2.1 (authoritative, current):

- **Sampler:** `euler` + `simple`, **cfg 1.0** (template default). [E2]
- **cfg is the text-adherence lever:** at cfg 1 the negative prompt is skipped entirely and text adherence is "as published." **Raising cfg to ~2 makes the model follow dense prompts more closely, including small text and numbers**, at the cost of over-sharpened edges; cfg 5 degrades badly. So: **if "Stokes Towing" renders with a wrong letter, try cfg 2.0 first** — it is the documented fix for small-text adherence and costs one A/B test. [E2]
- **Steps:** base template 25 (40–50 for max detail; more steps = better small-text stability). [E2, E3]
- **Resolution:** the Resolution Selector sets aspect + megapixel target; **1.0 MP ≈ 1024×1024, set ~4.0 MP for native 2048-class output**. Text rendered at 1 MP is physically too small to be legible at banner size — **generate at 2K (or higher MP target) so the letters are big in pixels.** [E2, E3]
- **Aspect ratio:** template offers 1:1, 16:9, 21:9, 3:2, 4:3, etc. For a banner use the widest ratio available (21:9) at the 4.0 MP target. **Caution: "targets well above the model's 2K trained size (e.g. 4K) lose prompt adherence"** — so do not overscale; 2K-class is the sweet spot, and if you need more width, outpaint (F5) rather than jump to 4K+. [E2]
- **Prompt enhancement (refine_prompt):** OFF by default; the 9B PE rewriter can rewrite your prompt — **for exact-text banners keep it OFF** so the model sees your quoted strings verbatim. [E2]
- **KV-cache node / Turbo ManualSigmas:** irrelevant to text quality; leave defaults. [E2]

### F4 — Text-rendering LoRAs (the "LoRA that improves text rendering" question)

Honest finding: **there is no dominant, well-known "text-rendering LoRA" for Qwen-Image-2.1.** Text rendering is the base model's headline strength, so the ecosystem's LoRAs target other things (character consistency, outpainting, turbo distillation, style). What exists:

| LoRA | Base model | What it does | Link |
|---|---|---|---|
| **Qwen-Image Multilingual Alignment LoRA** (Jellai, Nov 2025, 450 MB, Apache 2.0) | Qwen-Image (20B 2512) / Qwen-Image-Edit-2509 — **NOT 2.1** | Aligns English/Chinese/Japanese/Korean/etc. prompt→image; trained on 1.5M multilingual pairs. Community reports it reduces the old "prompt-in-Chinese" hack. | civitai.com/models/2122341 |
| **Kalaido-qwen-image-lora** (FractalAIResearch) | original Qwen-Image — **NOT 2.1**, and the HF page 404s as of this research (removed/moved) | RL fine-tune: "sharper and more readable text rendering", better aesthetics. Unusable as-is right now. | huggingface.co/FractalAIResearch/Kalaido-qwen-image-lora (404) |
| **Qwen-Image-2.1 Consistency LoRA** (2026-09-25) | Qwen-Image-2.1 | Edit consistency (nothing moves when editing) — not text, but relevant if the banner is an *edit* of an existing image. | civitai.com/models/2969143 |
| **Qwen-Image-2.1 Outpaint V2 LoRA** (ausboss) | Qwen-Image-2.1 | Seam-aware outpainting — useful for widening the banner (F5), not for text. | civitai.com/models/2964810, runcomfy workflow |
| **Qwen 2.1 Turbo LoRAs** (PrunAI, DMD) | Qwen-Image-2.1 | 5/8-step distillation. Speed only. | civitai.com/models/2958738 |
| **Qwen-Image-2512 Turbo-LoRA** | 2512-20B | Speed. | civitai.com/models/2268063 |

**Recommendation:** do not stack a text LoRA onto 2.1 — none is credible for that job on this base. If the banner text is still wrong after F2+F3, the fix is (a) more seeds, (b) cfg 2.0, (c) 2K resolution, or (d) switch the render to the 2512-20B model (with the Multilingual Alignment LoRA if needed) — not a 2.1 text LoRA. The "Multilingual Alignment" LoRA is the closest thing to a "text LoRA" but its base is 2512/2509, so it would only apply if the banner moves to 2512.

### F5 — Banner-specific geometry: extreme aspect ratios

A real banner is wider than any single 2K generation is comfortable with. Options, in order:

1. **Generate at 21:9 (or widest template ratio) at 4.0 MP** — one pass, text centered. Best when the banner fits ~2.3:1.
2. **Generate at 16:9 or 21:9, then outpaint the sides** with the Qwen-Image-2.1 outpaint workflow (gray-canvas padding + Outpaint V2 LoRA, runcomfy/Civitai templates). Outpainting keeps the rendered text intact and only paints new background — **this is the safe way to go wider without re-rolling the text.** [E7, E8]
3. **Composite**: render the text on a clean background at 2K, then place it over a generated photo banner in Post/Photoshop/Remotion. Most reliable for production signage; the Qwen-Image RGBA/alpha output (native 4-channel) makes text extraction trivial. [E1]
4. Avoid 4K+ single-shot: documented loss of prompt adherence. [E2]

### F6 — Stokes Towing concrete plan

1. Load/confirm `qwen_image_2.1` GGUF (Q8_0) + `qwen_image_2.1_vae_bf16` + `qwen3vl_8b` text encoder on the box (Comfy-Org HF layout).
2. T2I workflow (official `image_qwen_image_2_1_t2i` template), **refine_prompt OFF**, euler/simple, **25 steps**, start **cfg 1.0** → if letters garble, **cfg 2.0**, **aspect 21:9, MP target ~4.0** (2K-class wide).
3. Prompt per F2 template (quoted "STOKES TOWING", type style + placement + contrast).
4. Roll 4–6 seeds; keep the one with clean text.
5. If still wrong: outpaint for extra width (F5.2) or move the text layer to 2512-20B and re-composite (F5.3).
6. Final delivery: upscale the kept frame only if needed (Real-ESRGAN) — do not let the model re-render text at upscale.

## Claims (with evidence)

| id | Claim | Evidence | Status | Confidence |
|---|---|---|---|---|
| C1 | Quoting words in double quotes is the primary documented technique for exact text rendering in Qwen-Image (incl. 2.1) | [E5][E6][E4] | supported | high |
| C2 | Raising cfg from 1.0 to ~2.0 improves small-text/number adherence on Qwen-Image-2.1 at the cost of over-sharpened edges | [E2] | supported | high |
| C3 | cfg 1.0 (template default) skips the negative-prompt pass, so negative prompts are inert at default settings | [E2] | supported | high |
| C4 | Native 2K (4.0 MP target) is the sweet spot; targets well above 2K (4K+) lose prompt adherence | [E2][E3] | supported | high |
| C5 | Base 2.1 samples 25 steps / euler / simple; Turbo is a fixed 8-step ManualSigmas schedule; published pipeline runs 40–50 steps | [E2][E3] | supported | high |
| C6 | No dominant text-rendering LoRA exists for the Qwen-Image-2.1 base; 2.1 LoRAs target consistency, outpaint, turbo | [E9][E10][E11] | supported (absence-of-evidence, search-verified) | medium |
| C7 | Qwen-Image Multilingual Alignment LoRA (Jellai) is for the 2512/2509 base, not 2.1; 450 MB, Apache 2.0 | [E9] | supported | high |
| C8 | Outpainting (Outpaint V2 LoRA + gray-canvas pad) is the safe route to widen a banner without re-rendering text | [E7][E8] | supported | medium |
| C9 | The AI Box ComfyUI (:8188) was unreachable from this container at research time | direct probe (this run) | supported | high (for this run only) |

## Evidence Index

- [E1] Qwen official blog, "Qwen-Image-2.1: Compact, Efficient, and Unified Image Creation" (2026-09-20) — https://qwen.ai/blog?id=qwen-image-2.1 (7B DiT, native transparency, typography improvements, up to 10 refs)
- [E2] ComfyUI official docs, "Qwen-Image-2.1 ComfyUI Native Workflow Example" — https://docs.comfy.org/tutorials/image/qwen/qwen-image-2-1 (sampler/cfg/steps/resolution/KV-cache/prompt-enhancement/aspect; "targets well above 2K lose prompt adherence"; cfg-2 small-text note; model file layout)
- [E3] Civitai "Qwen Image 2.1 GGUF" (molbal) — https://civitai.com/models/2952547/qwen-image-21-gguf (Q8_0 7.16 GB, steps 25/cfg 1/euler/simple, 1.0 MP ≈ 1024², 4.0 MP ≈ 2048², edit resolution semantics)
- [E4] PixelDojo "Qwen Image 2.1 Prompting Guide" — https://pixeldojo.ai/guides/qwen-image-2-1-prompting-guide (quote-every-word, type style + placement, short lines, full model for fine print, Turbo 8 steps, 3-LoRA stack, negative prompt only full model)
- [E5] Civitai article "Qwen-Image-2512 — Prompt Guide & Best Practices" — https://civitai.com/articles/30826/qwen-image-2512-prompt-guide-and-best-practices (text rendering strengths, English+Chinese)
- [E6] dripxiforge "Qwen Image Prompts: A Working Guide" — https://www.dreampixelforge.com/blog/qwen-image-prompts (layout-first formula, exact-text rendering, negative prompts); corroborating: cliptics.com multi-language overlay guide; segmind prompt/parameter guide
- [E7] RunComfy "Qwen Image 2.1 Outpainting in ComfyUI" — https://www.runcomfy.com/comfyui-workflows/qwen-image-2-1-outpainting-in-comfyui (gray-canvas pad, Outpaint V2 LoRA by ausboss, seam-aware extension)
- [E8] Civitai "Qwen Image 2.1 Outpaint" workflow — https://civitai.com/models/2964810/qwen-image-21-outpaint (ratio-driven padding)
- [E9] Civitai "Qwen-Image Multilingual Alignment LoRA" (Jellai) — https://civitai.com/models/2122341/qwen-image-multilingual-alignment-lora (10-language alignment, 450 MB, base Qwen-Image / Edit-2509, Apache 2.0, strength 1)
- [E10] Civitai "Qwen Image 2.1 Consistency LoRA" — https://civitai.com/models/2969143/qwen-image-21-consistency-lora (edit consistency, 2.1 base)
- [E11] Civitai "Qwen 2.1 Turbo LoRAs" (PrunAI) — https://civitai.com/models/2958738/qwen-21-turbo-loras (5/8-step DMD adapters, 2.1 base); GitHub foprc/qwen-image-2.1-comfyui-workflow (Q8_0, 3-slot LoRA stack, CFG-1 sanity note)
- [E12] HuggingFace FractalAIResearch/Kalaido-qwen-image-lora — https://huggingface.co/FractalAIResearch/Kalaido-qwen-image-lora (text-readable RL fine-tune of original Qwen-Image; **page 404s as of 2026-10-15**)

## Gaps / Open Questions

1. **AI Box state unknown this run** — ComfyUI :8188 unreachable from the container (all candidate IPs failed). Cannot confirm which Qwen-Image-2.1 GGUF is loaded, available VRAM, or test a live prompt. Need a box-reachable probe (or the operator to confirm the box IP/port) to validate F2–F6 live.
2. **No live benchmark** — claims about cfg 2.0 / 2K / 21:9 improving "Stokes Towing" specifically are based on documented behavior + community consensus, not a measured render. A 3-seed A/B at cfg 1.0 vs 2.0 would close this.
3. **Banner aspect unknown** — exact Stokes Towing banner ratio (2:1? 3:1?) not provided; 21:9 is the widest template ratio, wider would need outpaint or composite.
4. **2512-20B on the box?** Unknown if the box can hold 20B GGUF; if the banner must be production-grade and 2.1 keeps failing, that's the fallback, but it changes VRAM math.
5. Kalaido LoRA 404 — unclear if moved; if it reappears it's the only "text-sharpening" LoRA in the family, but on the 20B base.

## Confidence

- **level: medium**
- **rationale:** Model facts, workflow parameters, and the prompt-technique consensus are well-sourced (official Qwen blog + official ComfyUI docs + multiple independent guides) — those parts are high confidence. The two weak points are (a) no live AI Box validation this run, and (b) the banner-specific outcome (does "STOKES TOWING" render clean at 21:9/2K/cfg-2) is inferred, not measured. LoRA-absence claim is medium: searches were thorough across Civitai/HF but the ecosystem moves fast and new 2.1 text LoRAs could appear.

## Recommended Next Actions

1. **Live A/B on the box** (operator, ~15 min): at 21:9 / 4.0 MP, render the F2 banner prompt at cfg 1.0 vs cfg 2.0, 3 seeds each; keep the cleanest. This validates the whole plan empirically.
2. **Confirm box model files**: verify `qwen_image_2.1_int8_convrot.safetensors` (or GGUF Q8_0) + VAE + qwen3vl_8b text encoder are present; the official ComfyUI template `image_qwen_image_2_1_t2i` loads them by name.
3. **If text still garbles**: (a) 4–6 seeds, (b) drop text to one line only, (c) raise MP target (bigger letters), (d) outpaint for width, (e) fall back to 2512-20B + Multilingual Alignment LoRA + composite in Post.
4. **Do not** install a text LoRA onto 2.1 in expectation of fixing letters — none credible exists for that base (C6).
5. **Document the winning settings** on the box (steps/cfg/MP/aspect/prompt) in the SCOR project notes so the banner pipeline is reproducible.

*Extraction path: web_search + web_extract (markdown-first) for all evidence; no browser fallback needed. AI Box probe: direct curl from container, all candidates failed.*
