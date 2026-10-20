# Local Omni model recommendations 2026-10-20

**Task:** t_322d6031 — research local omni model stack
**Hardware (authoritative):** A = home `fedora` RTX 5060 Ti 16GB, 62GB RAM (no CPU offload for GPU inference). B = `ai-box` 2× RTX 3090 24GB, 32GB RAM, Comfy :8288/:8289, exclusive modes `llm(:8088)` / `image` / `video` / `video-split`, vLLM TP2 often resident on both GPUs.
**Pipeline:** stickman / flat-cartoon / line-explainer → picture clips + VO + BGM + ffmpeg stitch. 16:9 primary (9:16 later). H3 open-Base already proven on ai-box (1024×576 / 1344×768, turbo-4/8, ~768p ceiling).
**License policy:** Apache/MIT/OpenRAIL-friendly preferred for future US client work; territorial/community licenses must be explicitly labeled.

**Disk inventory status (per ops steer 2026-10-20):** ai-box model dirs could NOT be re-inventoried live from this container (no SSH key mounted; Tailscale 100.99.186.66 rejects this key; `ai-box` hostname unresolvable from the Docker sandbox). Every "already on disk" below is per on-disk fleet docs (h3-loras HANDOFF/README, hotel-lobby keeper lock, VG_HARNESS_SPEC) and must be re-confirmed on-box by ops/Su-v2 (`ls ~/video-prep/ComfyUI/models/{checkpoints,diffusion_models,loras,vae,text_encoders}`). **All new model rows are marked NOT ON DISK until verified.** The X-bookmark archive (`/home/ice/fed/x-bookmarks/`) and `hardware-inventory.md` symlink target (`/srv/app-data/huly-artifacts/ops/20260831-dual-3090-hardware-inventory.md`) are **not visible from this container** (dangling on the mounted volume) — treat as unavailable here, not as evidence.

---

## Executive picks

| # | Role | Pick | License | Where | VRAM | Disk (approx) | On disk? | Install complexity |
|---|---|---|---|---|---|---|---|---|
| 1 | Video I2V (primary) | **Wan 2.2 TI2V-5B** (ComfyUI native) | Apache-2.0 | ai-box (or 5060 Ti) | ~10–12GB (BF16/FP8) | ~14–20GB (weights + VAE) | **NOT on disk — download** | Low (native ComfyUI template) |
| 2 | Video I2V / FL2V (backup / lock) | **MiniMax H3 open-Base + FL2V turbo-4/8 + Ref2V turbo** | **Community — territorial, R&D quarantine for US client ship** | ai-box | 24GB class | ~40–60GB | **On disk per h3-loras docs — RE-VERIFY on-box** | Zero (proven) |
| 3 | Video I2V (draft on 5060 Ti) | **Wan 2.2 TI2V-5B FP8** | Apache-2.0 | home 5060 Ti | ~10–12GB | ~15GB | **NOT on disk — download** | Low |
| 4 | TTS (narrator lock) | **Kokoro-82M** (ONNX, Apache-2.0) | Apache-2.0 | 5060 Ti (CPU also fine) | ~1GB / CPU | ~500MB | **NOT on disk — download** | Very low (`pip`/ONNX) |
| 5 | TTS (expressive / clone backup) | **Chatterbox-Turbo 350M** (Resemble AI) | MIT | 5060 Ti | ~3–4GB | ~1.5–2GB | **NOT on disk — download** | Low (`pip install chatterbox-tts`) |
| 6 | BGM | **HeartMuLa-oss-3B** (+ HeartCodec-oss-20260123) | open (commercial OK per source; verify per-file) | 5060 Ti | ~5.2GB @ Q8_0 | ~8–10GB | **UNVERIFIED — fleet has skill, weights unconfirmed** | Low–Med |
| 7 | BGM (backup, simpler) | **MusicGen (Meta AudioCraft) large** | non-commercial (Meta) → label | 5060 Ti / idle 3090 | ~6–8GB | ~5–7GB | **NOT on disk — download** | Low |
| 8 | SFX v1 | **Static licensed pack** (no gen model) | per-pack | — | — | — | n/a | None |
| 9 | SFX gen (optional later) | **Stable Audio Open** / AudioCraft AudioGen | NC-community (label) | idle 3090 | ~4–6GB | ~2–4GB | **NOT on disk — download** | Med |

**One-line stack:** H3 (proven lock, R&D) + **Wan 2.2 TI2V-5B** (Apache I2V hero, **must download**) on ai-box; **Kokoro-82M** VO on 5060 Ti (local, **download**); **HeartMuLa-oss-3B** BGM on 5060 Ti (verify weights); static SFX pack; ffmpeg stitch. **Edge TTS is excluded — cloud, not local.**

---

## Video — AI Box 2×3090

### V1 — Wan 2.2 TI2V-5B (PICK, Apache-2.0)
- **Role:** I2V (character sheet → 5–10s clip), also T2V. 720P @ 24fps, hybrid TI2V.
- **License:** Apache-2.0 (full commercial, US client-safe) — verified via HF `Wan-AI/Wan2.2-TI2V-5B` + wavespeed summary.
- **VRAM/disk:** runs on consumer 4090 per HF; ~10–12GB on a 3090 for 720p/5–10s; weights + VAE ≈ 14–20GB.
- **Install:** ComfyUI **native** support — official templates for T2V, I2V, and **first-last-frame** (FL2V-style lock) exist. Drop in `Comfy-Org` repackaged `wan2.2_ti2v_5B_fp16`/`14B fp8` + VAE; use `video_wan2_2_5B_ti2v.json` template.
- **Why pick:** cleanest commercial license of any current open I2V; native ComfyUI; first-last-frame template matches the "still + lock" workflow; no territorial clause.
- **Dual-3090:** run single-GPU per worker (:8288 GPU0 / :8289 GPU1). 5B fits one 3090 → **sequential two workers** for throughput; no USP/FSDP needed at 5B.

### V2 — MiniMax H3 open-Base + turbo LoRAs (BACKUP / proven lock, TERRITORIAL — label)
- **Role:** FL2V (first-last frame) + Ref2VA character continuity — already the proven pipeline on this box.
- **License:** MiniMax **community license — excludes US/EU/UK/KR self-hosting** (verified via multiple 2026-08 sources). **R&D / non-US-client quarantine only.** Do not ship US-client deliverables on H3 without legal clearance.
- **Already staged (do not re-download):** `minimax_h3_fl2v_turbo_4step`, `fl2v_turbo_8step`, `ref2v_turbo_4step`, `character_swap_pro4500`, `larryvrh turbo-v4` under ai-box `~/video-prep/ComfyUI/models/loras/` + CARD docs + MANIFEST (see `/home/ice/know/video-gen/h3-loras/`).
- **Res ladder (from proven recipe):** 1024×576 smoke / **1344×768 hero** 16:9, 124 frames (~5.2s) turbo-4 ~7–8 min per gen; 243/362 frames for 10s/15s. Open-Base ceiling ~768p short edge.
- **Why keep:** best proven character/FL2V lock for stickman continuity; but the territorial license is the reason Wan 2.2 is the *hero* and H3 the *R&D/quarantine* path.
- **Avoid:** 2K / "H3 2K" is **not** open-offline; upscale via SeedVR2/FlashVSR separately if ever needed.

### V3 — Wan 2.2 A14B (optional quality bump)
- MoE 14B T2V/I2V; Apache-2.0; **needs 24GB+** for 480p–720p quality. FP8 GGUF fits one 3090; 720p/longer is slow. Use only if 5B quality is insufficient; prefer 5B for 5–10s stickman clips (faster, lower VRAM, same license).

### T2V without stills
- Wan 2.2 (5B or 14B) T2V is "good enough" for B-roll/establishing shots but **without a character sheet it cannot hold the stickman identity** — do not use T2V as the primary for character continuity. Use I2V/FL2V from a locked still.

### Dual-3090 strategy (respect exclusive modes)
- **Recommended: sequential single-GPU** — one ComfyUI worker per GPU, queue jobs on the idle card. 5B/TI2V fits a single 24GB, so USP/FSDP multi-GPU model-parallel is **not needed** and adds fragility.
- Do NOT load a video model while `llm (:8088)` vLLM TP2 is resident — that is the exclusive-mode conflict (see Conflicts section).

### Explicit video rejects
- **LTX-2 / LTX-2.5 (19B/22B):** current flagship weights ~66GiB, 40+ steps, 22B — far beyond a 2×24GB box at usable speed for 5–10s clips; the 12GB low-VRAM GGUF path is community-patched and slow. The old LTX-Video 2B (2024) is superseded and not worth new install for this pipeline. **Avoid** for this hardware.
- **H3 2K / Context-IR / 2K-regenerate:** closed/hosted, not open-offline.
- **VRAM bombs:** any 22B–32B video DiT (LTX-2.5, FLUX.2-dev 32B used as video, etc.) — no free commercial or no fit.

---

## Video — optional 5060 Ti (16GB, no offload)

- **Draft/stills only:** **Wan 2.2 TI2V-5B FP8** fits 16GB with headroom (same model as ai-box, smaller resolution: 1024×576 / 768p, 5s, ~24fps). Good for cheap previews and still→clip drafts before committing ai-box GPU time.
- **No offload-first architectures as default.** If a model only runs via aggressive CPU offload, reject it here (hard constraint).
- **Do not** run H3 here — it's the ai-box R&D asset and the 5060 Ti is for Hermes/ASR/stitch/TTS/BGM.
- Image stills for the character sheet can be produced on the 5060 Ti via the existing FLUX.2-klein/Z-Image Apache stack (see `research/local-img-gen-dual-3090-kie-replacement.md`); keep that separation.

---

## TTS — 5060 Ti (narrator lock across 6 clips)

### T1 — Kokoro-82M (PICK, Apache-2.0)
- **Quality:** UTMOS MOS ~4.44–4.46, top of the small-TTS class (verified: heyneo CPU benchmark, claytonharbour audiobook writeup). Runs CPU-only; on the 5060 Ti it's ~1.5–1.6× real-time even on CPU.
- **Narrator lock:** ~24 English preset voices with **measured** F0/brightness (briantung.me) — pick one voice (e.g. `af_heart` default narrator) and it stays identical across all 6 clips. Deterministic, no reference audio needed → perfect "narrator lock."
- **VRAM/disk:** ~1GB (or pure CPU); ~500MB ONNX (`onnx-community/Kokoro-82M-v1.0-ONNX`). Apache-2.0, commercial-safe.
- **Install:** `pip` (kokoro / onnx). Lowest install complexity of any option.
- **Tradeoff:** uniform prosody, clean-cut pauses; a touch "flat" vs Gemini-class. For narration/VO this is a strength (clarity). Pipeline hygiene (strip markdown, 10ms crossfade, whisper-verify) is what closes the gap (claytonharbour).

### T2 — Chatterbox-Turbo 350M (BACKUP / expressive + zero-shot clone, MIT)
- **Quality:** Resemble AI SoTA open TTS; Turbo 350M = low compute/VRAM, **native paralinguistic tags** (`[laugh]`, `[cough]`, `[chuckle]`), single-step decoder. Beats Edge TTS on expressiveness; **zero-shot voice cloning from a reference clip** if you ever want a "person" voice instead of a preset.
- **License:** MIT (repo) / Apache-2.0 (model card) — commercial-safe. **Note: built-in PerTh watermark on every clip** (imperceptible, for traceability) — disclose if US-client compliance matters.
- **VRAM/disk:** ~3–4GB, ~1.5–2GB. `pip install chatterbox-tts`.
- **When to use:** if Kokoro's flat prosody is a problem, or you want cloning/emotion tags. Slightly heavier than Kokoro.
- **Chatterbox-Nano 110M:** CPU 3× real-time, tags, for tightest budgets — fine backup.

### T3 — Edge TTS (existing — CLOUD, NOT a local pick)
- Already on the fleet (`en-US-AriaNeural`) but it is a **cloud** baseline, not local. Per ops steer: it is **excluded from the recommendation set** — not privacy-first, not self-hosted. Keep only as an emergency fallback if a local TTS misbehaves mid-run; it is not part of the recommended stack.

**Decision:** **Kokoro-82M is the default narrator** (local, Apache, deterministic, trivial install; CPU-capable so it never competes for 5060 Ti VRAM against BGM/stitch). Chatterbox-Turbo as the expressive/clone upgrade when Kokoro's flatness hurts. Edge TTS is out of the stack (cloud).

---

## BGM (30–60s instrumental, prompt/tag control)

### B1 — HeartMuLa-oss-3B (PICK)
- **What:** open music foundation model family (HeartMuLa / heartlib), 3B/3.9B; "currently the best open-sourced model for lyrics controllability and music quality"; recommended pair is **HeartMuLa-oss-3B + HeartCodec-oss-20260123**. Explicitly marketed for **short, engaging music suitable as background music for short videos** — exactly this use case.
- **VRAM/disk:** ≈**5.2GB @ Q8_0** (selfhost.directory), so fits the 5060 Ti 16GB with room for a small TTS resident. Disk ~8–10GB.
- **Control:** textual style description + optional lyrics + reference audio. For a 30–60s instrumental bed, feed a style prompt (e.g. "lofi, mellow, no vocals, 90bpm") — check the oss-3B repo for the instrumental/vocal toggle.
- **License:** open, "commercial use OK" per source — **verify per-file license in the HF repo before US-client ship** (flagged, not confirmed Apache).
- **Install:** `heartlib` repo + weights; Med complexity. Run on 5060 Ti (fits), or on an idle 3090 in `image`/`video` mode if the 5060 Ti is busy stitching.
- **Status check:** the fleet already has a **HeartMuLa skill**; verify whether weights are installed — if not, this is the download.

### B2 — MusicGen (Meta AudioCraft, BACKUP, label non-commercial)
- MusicGen large ≈ 3B; ~6–8GB VRAM; strong text→music; **Meta license is non-commercial** → label, R&D/personal only, not US-client. Easier install than HeartMuLa (`audiocraft` pip). Good if HeartMuLa proves fiddly.

### B3 — static/original track (always valid fallback)
- Per the proven hotel-lobby recipe, an **owned/licensed/original** track ffmpeg-muxed is always acceptable and the lowest-risk path. Generative BGM is an enhancement, not a requirement.

**Decision:** **HeartMuLa-oss-3B** on the 5060 Ti (Q8_0, ~5.2GB) for the bed; MusicGen as labeled non-commercial backup; original track as the floor.

---

## SFX (honest: pack wins for v1)

- **v1 = static licensed SFX pack (PICK).** For a 6-clip stickman explainer, a curated pack (foley hits, whooshes, transitions, UI clicks) is faster, deterministic, license-clean, and higher quality-per-effort than a gen model. No install. **This is the honest recommendation.**
- **Optional gen model later (label non-commercial):**
  - **Stable Audio Open** (Meta) — text→sound/field recordings, ~4–6GB VRAM, NC-community license → label. Good for one-off custom SFX on an idle 3090.
  - **AudioCraft AudioGen** — text→SFX, NC, easy.
  - **Woosh (Sony AI, 2026)** — SFX foundation model, T2A + V2A; **code MIT, open weights CC-BY-NC** → weights are **non-commercial**, label. Strong if you later want V2A (generate SFX from the rendered clip). Not v1.
- **Recommendation:** build the pack first; only pull a gen model if a specific SFX can't be sourced.

---

## Install order + disk budget

**Disk budget (total ≈ 60–80GB new):**
- Wan 2.2 TI2V-5B + VAE: ~15–20GB (ai-box or 5060 Ti)
- HeartMuLa-oss-3B + HeartCodec: ~8–10GB (5060 Ti)
- Chatterbox-Turbo: ~2GB (5060 Ti)
- Kokoro-82M ONNX: ~0.5GB (5060 Ti, or CPU)
- MusicGen (optional): ~5–7GB (idle 3090)
- Stable Audio Open (optional SFX): ~3–4GB (idle 3090)
- (H3 + LoRAs: **on disk per h3-loras docs — re-verify on-box before trusting; NOT confirmed from this container**)

**Order (dependency-lightest first, verify each before the next):**
1. **Kokoro-82M** (5060 Ti) — `pip install kokoro` / pull ONNX; synthesize one test VO line. Lowest risk, unblocks VO immediately.
2. **Chatterbox-Turbo** (5060 Ti) — `pip install chatterbox-tts`; one test line (clone + tags). Backup VO.
3. **HeartMuLa-oss-3B + HeartCodec** (5060 Ti) — install `heartlib`, pull weights Q8_0; generate one 30s instrumental, verify no-vocal + duration.
4. **Wan 2.2 TI2V-5B** (ai-box first, then 5060 Ti draft copy) — add `Comfy-Org` repackaged 5B fp16 + VAE via ComfyUI; run the native `video_wan2_2_5B_ti2v` template on a locked still → one 5s 720p clip.
5. **(Optional) MusicGen / Stable Audio Open** on an idle 3090 only if the above gaps appear.
6. **Static SFX pack** — source + drop into the ffmpeg stitch; no GPU.

**ffmpeg stitch last** — reuse the existing long-form assembly skill; mux VO (Kokoro) + BGM (HeartMuLa) + SFX (pack) over the video clips.

---

## Conflicts with ai-box-mode llm/image/video

- **vLLM TP2 on both 3090s is the `llm (:8088)` baseline** and holds both GPUs. **Any Wan/H3 video gen MUST switch the box to `video` (or `image`) mode first** — this kills :8088 and is consent-gated (per H3 handoff: "First smoke only after Frank/ops consent for `ai-box-mode video`"). Never assume the box is free.
- **Exclusive modes mean: no simultaneous video + LLM.** The TTS (Kokoro/Chatterbox) and BGM (HeartMuLa) runs happen on the **5060 Ti**, not the ai-box, precisely so they don't touch the llm/image/video exclusivity. This is the key architectural split: **video on ai-box, all audio + stitch on 5060 Ti.**
- **Sequential single-GPU on ai-box:** run Wan 5B on GPU0 (:8288) or GPU1 (:8289) — one at a time; do not try USP/FSDP across both for a 5B model.
- **H3 R&D gens** also require `video` mode + consent and must not clobber the hotel_lobby USERLOCK (separate lock family).
- **5060 Ti hard constraint:** no CPU offload. Kokoro is CPU-capable anyway; HeartMuLa Q8_0 (~5.2GB) fits 16GB; Wan 5B FP8 (~10–12GB) fits 16GB. None need offload.

---

## Explicit rejects
1. **LTX-2 / LTX-2.5 (19B/22B, ~66GiB):** doesn't fit 2×24GB at usable 5–10s speed; community 12GB patches slow/unstable. Supersedes the old 2B which is not worth new install.
2. **Wan 2.2 A14B for 5–10s stickman clips:** works but overkill vs 5B; use only if 5B quality fails. (Apache, fine — just not the default.)
3. **H3 2K / Context-IR / 2K-regen:** closed/hosted, not open-offline.
4. **FLUX.1-dev / FLUX.2-dev as video:** gated non-commercial + not the right task; R&D only.
5. **Any CPU-offload inference path on 5060 Ti:** hard constraint, no exceptions — reject any model that only runs via offload.
6. **Edge TTS as a recommended TTS:** cloud, not local — excluded from the stack (ops steer); emergency fallback only.
7. **Woosh / Stable Audio Open / AudioGen for v1 SFX:** non-commercial weights + more effort than a pack earns at this stage (keep as later options, labeled).
8. **MusicGen as US-client BGM:** non-commercial — label; use HeartMuLa (open) for anything that ships.

---

## Sources
- Wan 2.2 TI2V-5B (Apache-2.0, 720P/24fps, consumer 4090): https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B
- Wan 2.2 family Apache-2.0 + native ComfyUI T2V/I2V/FL2V templates: https://huggingface.co/spaces/wavespeed/wan2.2 , https://docs.comfy.org/tutorials/video/wan/wan2_2 , https://comfyui-wiki.com/en/tutorial/advanced/video/wan2.2/wan2-2 , https://www.thundercompute.com/blog/wan-2-2-comfyui-ai-video-model
- LTX-2 / LTX-2.5 (22B, ~66GiB, 40-step, fp8/offload, ComfyUI): https://github.com/lightricks/ltx-2/blob/main/README.md , https://huggingface.co/Lightricks/LTX-2.5 , https://github.com/Lightricks/LTX-2/blob/main/LICENSE
- MiniMax H3 open-weights **territorial license (excludes US/EU/UK/KR)**: https://www.techtimes.com/articles/322904/20260804/minimax-h3-open-weights-exclude-us-eu-uk-korea-local-deployment.htm , https://explainx.ai/blog/minimax-h3-open-video-model-hailuo-july-2026 , https://llm-stats.com/blog/research/minimax-h3-launch
- Kokoro-82M (Apache-2.0, MOS ~4.45, 24 measured English voices): https://huggingface.co/hexgrad/Kokoro-82M , https://heyneo.com/blog/kokoro-supertonic-inflect-nano-pocket-tts-cpu-benchmark , https://briantung.me/blog/kokoro-voices-measured/ , https://www.claytonharbour.com/blog/kokoro-82m-tts-audiobook-quality
- Chatterbox (MIT/Apache, 350M Turbo, tags, clone, PerTh watermark): https://github.com/resemble-ai/chatterbox/ , https://huggingface.co/ResembleAI/chatterbox , https://www.resemble.ai/learn/models/chatterbox-multilingual
- HeartMuLa-oss-3B + HeartCodec (≈5.2GB @ Q8_0, short-video BGM): https://github.com/HeartMuLa/heartlib , https://heartmula.github.io/ , https://selfhost.directory/models/heartmula/heartmula-oss-3b , https://arxiv.org/html/2601.10547v1
- Stable Audio Open / AudioCraft (AudioGen, NC): https://arxiv.org/html/2407.14358v2 , https://ai.meta.com/resources/models-and-libraries/audiocraft/
- Woosh Sony AI (SFX foundation, T2A/V2A, weights CC-BY-NC): https://github.com/SonyResearch/Woosh , https://ai.sony/publications/woosh-a-sound-effects-foundation-model

**On-disk / fleet references (verified local where noted):**
- `/home/ice/know/video-gen/LOCAL_OMNI_ROUTE.md` (parent architecture)
- `/home/ice/know/video-gen/LOCAL_OMNI_MODEL_RESEARCH_BRIEF.md` (this brief)
- `/home/ice/know/video-gen/h3-loras/` (H3 LoRA cards + MANIFEST — claims ai-box staging; **re-verify on-box**, not confirmed from this container)
- `/home/ice/know/video-gen/HOTEL_LOBBY_H3_KEEPER_LOCK.md` (proven H3 lock + res ladder 1024×576 / 1344×768)
- `/home/ice/know/research/local-img-gen-dual-3090-kie-replacement.md` (Apache image stack for stills)
- `/home/ice/know/video-gen/VG_HARNESS_SPEC.md` (freeze/lock/one-knob harness contract)
- `~/.hermes/skills/**/local-video-gen-ops` (Hermes skill: video ops rules)
- `~/.hermes/skills/**/heartmula-*` (Hermes skill: music gen; **weights install unverified — see on-disk column**)

**Unavailable from this container (dangling / not mounted — do NOT treat as evidence):**
- `/home/ice/fed/x-bookmarks/` (X bookmark archive; `bookmarks.jsonl` + `query.py` not visible here)
- `/home/ice/fed/docs/ai-box/hardware-inventory.md` → symlink target `/srv/app-data/huly-artifacts/ops/20260831-dual-3090-hardware-inventory.md` (dangling on the mounted volume)
- `/home/ice/fed/docs/ai-box/remote-baseline.md` → symlink target (dangling)
- Live `ssh ai-box` (Tailscale 100.99.186.66 rejects this container's key; `ai-box` hostname unresolvable from the Docker sandbox)

---

## Provenance / method
- Extracted via `web_search` (exa/ddgs) + on-disk fleet docs; no browser navigation required (no paywalled/JS-only sources hit). Provenance per claim: web-search result text (labeled URL above) or local file path.
- **Disk inventory (ops steer #3):** attempted live `ssh ai-box` from the Docker sandbox — **failed** (no SSH key mounted; Tailscale 100.99.186.66 rejects; hostname unresolvable). All "on disk" claims are therefore from on-disk fleet docs only and are flagged **RE-VERIFY on-box**. Every new model is marked **NOT ON DISK** in the table until an on-box `ls` confirms.
- **X-bookmark / fed stacks (ops steer #4):** `/home/ice/fed/x-bookmarks/` and the `hardware-inventory.md` / `remote-baseline.md` symlink targets are **not visible from this container** (dangling on the mounted volume). Treated as unavailable, not as evidence. The qwen38 ai-box research prompt at `/home/ice/fed/docs/ai-box/20260913-qwen38-aibox-256k-research-prompt.txt` **is** readable here and confirms the 2×3090 / 32GB / no-offload / :8088 baseline.
- **No GPU inference was executed** for this task (research-only). No fabricated benchmark numbers: all VRAM/MOS figures are quoted from the cited sources, not measured here.
