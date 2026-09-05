# Diarization Pipelines: Local Open-Source Solutions for Multi-Speaker Transcription

**Task ID:** t_5516c0d0
**Date:** 2026-09-05 (MST)
**Researcher:** Geraldo v2.2 (GIU)

---

## 1. Objective

Map the landscape of **local, open-source speaker diarization** solutions that can run without cloud APIs — evaluating architecture, accuracy (DER), licensing, resource requirements, and integration into multi-speaker transcription pipelines. The goal is to give a decision-ready comparison for self-hosted deployment.

## 2. Summary

Seven diarization libraries/pipelines dominate the open-source space as of mid-2026:

| Solution | Architecture | License | Best DER (open) | Gated? | GPU Req |
|---|---|---|---|---|---|
| **DiariZen** (BUT-FIT) | WavLM-Large + Conformer + VBx | MIT / CC-BY-NC-4.0 | ~9.1% VoxConverse | No (NC) | Yes |
| **pyannote.audio** | ResNet segmentation + embedding | MIT (code) / gated (weights) | ~9% VoxConverse | Yes (HF) | Recommended |
| **NVIDIA NeMo Sortformer** | 18-layer Transformer (e2e) | Apache 2.0 | Competitive | No | NVIDIA only |
| **NVIDIA NeMo Cascaded** | MarbleNet + TitaNet + spectral | Apache 2.0 | Moderate | No | NVIDIA only |
| **SpeechBrain** | ECAPA-TDNN embeddings | Apache 2.0 | Moderate | No | Optional |
| **WhisperX** | faster-whisper + wav2vec2 + pyannote | Custom (MIT-like) | Inherits pyannote | Yes (pyannote) | <8GB VRAM |
| **whisper-diarization** | Whisper + NeMo VAD/emb + Demucs | BSD-2 | Moderate | No | Recommended |

**Key takeaway:** For pure diarization, **DiariZen** currently reports the best open-source DER on VoxConverse (9.1%) and is ungated. **NeMo Sortformer** is the strongest fully permissive (Apache 2.0) option and offers streaming. For full ASR+diarization, **WhisperX** is the fastest integrated pipeline; **whisper-diarization** offers a more modular NeMo-based alternative with a permissive license.

---

## 3. Plan

### Tasks

| ID | Title | Goal | Mode | Status |
|---|---|---|---|---|
| T1 | Survey landscape | Identify open-source diarization tools | parallel_scouts | done |
| T2 | Deep-dive architectures | Extract pipeline design, accuracy, licensing for each tool | parallel_scouts | done |
| T3 | Cross-verify benchmarks | Triangulate DER claims from official READMEs, third-party papers | verification | done |
| T4 | Compare deployment | Resource, license, integration comparison | direct | done |
| T5 | Write report | Structured findings with provenance | direct | active |

### Assumptions
- "Local" = runs without cloud API calls (some require HF token download but run locally after)
- "Open-source" = source-available with OSI-approved or research-friendly license
- DER evaluated on standard benchmarks (VoxConverse, AMI, DIHARD III) without collar unless noted

### Open Questions
- DiariZen's CC-BY-NC-4.0 weights create legal friction for commercial deployment
- WhisperX v4 timeline unclear (development underway per GitHub, no release date)
- NeMo Sortformer's quadratic attention scaling limits long-form audio

---

## 4. Findings

### 4.1 DiariZen (BUT Speech@FIT, 2025–2026)

**Architecture:** Hybrid pipeline combining EEND-style neural segmentation with classical clustering:
1. Structurally pruned WavLM-Large encoder with learned layer weighting
2. Conformer backend with powerset classification (pyannote 3.1 fork)
3. Segmentation aggregation via overlap-add
4. Speaker embedding extraction with overlap exclusion
5. VBx clustering with PLDA scoring

**Models on HF:**
- `BUT-FIT/diarizen-wavlm-base-s80-md`
- `BUT-FIT/diarizen-wavlm-large-s80-md`
- `BUT-FIT/diarizen-wavlm-large-s80-md-v2`

**Benchmark DER (no collar, official README):**

| Dataset | Pyannote 3.1 | DiariZen-Large-v2 |
|---|---|---|
| AMI-SDM | 22.4 | 13.9 |
| AISHELL-4 | 12.2 | 10.1 |
| AliMeeting far | 24.4 | 10.8 |
| NOTSOFAR-1 | – | 16.7 |
| MSDWild | 25.3 | 15.8 |
| DIHARD3 full | 21.7 | 14.5 |
| RAMC | 22.2 | 11.0 |
| VoxConverse | 11.3 | 9.1 |

**Third-party benchmark (arXiv:2509.26177, Sep 2025):** DiariZen 13.3% overall DER across multilingual datasets (196.6h, 5 languages). PyannoteAI commercial: 11.2%. DiariZen was top open-source.

**License:** Code MIT; weights CC-BY-NC-4.0 (non-commercial only — required by DIHARD-3, RAMC, MSDWild training data).

**Install:** Requires Python 3.10, CUDA 12.1 build of PyTorch, pyannote-audio fork, dscore submodule.

### 4.2 pyannote.audio (pyannoteAI / Hervé Bredin)

**Architecture:** Modular — ResNet-based segmentation → speaker embedding → clustering. v4.0 (2025) introduces `community-1` pipeline using Powerset multi-class cross entropy.

**Open-source models:**
- `pyannote/speaker-diarization-3.1` (gated, HF)
- `pyannote/speaker-diarization-community-1` (gated, HF, CC-BY-4.0)
- `pyannote/segmentation-3.0` (gated)

**Benchmarks (from VexaScribe/pyannote guides):**
- VoxConverse: ~9–11% DER
- AMI: ~12–14% DER
- DIHARD III: ~17–19% DER
- Commercial precision-2: 11.2% overall multilingual (per arXiv:2509.26177)

**License:** MIT for code. Models gated on HF (must accept user agreement + use token). `community-1` CC-BY-4.0.

**Used by:** WhisperX, DiariZen (fork), countless production systems.

### 4.3 NVIDIA NeMo Speech — Sortformer

**Architecture:** End-to-end Transformer encoder (18 layers). Sort Loss + Arrival Time Sorting resolves permutation problem without PIL. Outputs speaker labels in arrival-time order directly from audio.

**Models:**
- Offline: `nvidia/diar_sortformer_4spk-v1`
- Streaming: `nvidia/diar_streaming_sortformer_4spk-v2` (Arrival-Order Speaker Cache)

**Cascaded alternative:** MarbleNet VAD → TitaNet speaker embeddings → spectral clustering.

**Performance:** Competitive with pyannote 3.x. Best on NVIDIA GPUs. Degrades >4 speakers. Quadratic memory in attention limits long-form audio.

**License:** Apache 2.0 — fully ungated, commercially usable.

**Reference:** Park et al., INTERSPEECH 2022; streaming paper arXiv:2507.18446 (Aug 2025).

### 4.4 NVIDIA NeMo Speech — Cascaded Pipeline

**Stages:**
1. VAD: MarbleNet (from NeMo speech classification)
2. Embedding: TitaNet (`nvidia/speakerverification_en_titanet_large`)
3. Clustering: Spectral clustering

**Use case:** Lower complexity, modular replacement. Often used in whisper-diarization and research.

### 4.5 SpeechBrain

**Architecture:** ECAPA-TDNN speaker embeddings (Voxceleb-trained). Modular: VAD → embedding → clustering.

**DER:** Competitive on VoxCeleb speaker verification; diarization benchmarks trail pyannote/NeMo. Reported as "moderate" in third-party comparisons.

**License:** Apache 2.0. Python 3.7+ on Linux/MacOS.

**Strength:** Most customizable/research-friendly; full conversational AI toolkit beyond diarization.

### 4.6 WhisperX (m-bain)

**Architecture (integrated pipeline):**
1. VAD pre-pass (Silero or similar)
2. faster-whisper batched transcription (CTranslate2 backend)
3. wav2vec2 forced phoneme alignment → word-level timestamps
4. pyannote.audio diarization → speaker labels

**Performance:** 70x realtime with large-v2, <8GB VRAM. Batch size adjustable for memory constraints.

**Diarization quality:** Inherits pyannote.audio 3.1 quality. Overlapping speech not well handled.

**License:** Repository uses a permissive custom license (MIT-like). Diarization depends on gated pyannote models.

**Status:** v3.8.6 released May 2026. v4 development underway with "significantly improved diarization" per README.

**Output:** SRT/VTT/JSON with word-level timestamps + speaker IDs.

### 4.7 whisper-diarization (MahmoudAshraf97)

**Architecture:**
1. Demucs vocal separation (optional pre-processing)
2. Whisper transcription (faster-whisper backend)
3. WhisperX for word timestamps
4. MarbleNet VAD (NeMo) for silence exclusion
5. TitaNet speaker embeddings (NeMo) for segment identification
6. ctc-forced-aligner + punctuation realignment

**Advantage over WhisperX:** Uses NeMo (Apache 2.0) for VAD/embedding instead of gated pyannote — no HF token required for diarization components. More configurable language handling.

**License:** BSD-2-Clause (permissive).

**Limitation:** Overlapping speakers not addressed. ~5.6k GitHub stars.

---

## 5. Comparative Decision Matrix

### Pure Diarization (who spoke when)

| Criteria | DiariZen | pyannote 3.x | NeMo Sortformer | SpeechBrain |
|---|---|---|---|---|
| **Best DER** | ✅ Yes (~9%) | ~9-11% | Competitive | Moderate |
| **Commercial use** | ❌ CC-NC-4.0 | ✅ MIT (gated) | ✅ Apache 2.0 | ✅ Apache 2.0 |
| **HF token needed** | No | Yes | No | No |
| **GPU required** | Yes | Recommended | Yes (NVIDIA) | Optional |
| **Streaming** | No | No | ✅ Sortformer-stream | No |
| **Multi-language** | ✅ 5+ langs | English-focused | English-focused | Multi |
| **Ease of install** | Complex | pip | pip | pip |

### Full ASR + Diarization Pipeline

| Criteria | WhisperX | whisper-diarization | Notes |
|---|---|---|---|
| **ASR quality** | faster-whisper (large-v2) | faster-whisper (large-v2) | Same engine |
| **Word timestamps** | wav2vec2 forced align | ctc-forced-aligner | Both language-specific |
| **Diarization engine** | pyannote.audio 3.1 | NeMo MarbleNet+TitaNet | Different tradeoffs |
| **License friction** | Gated pyannote | Fully permissive | whisper-diarization wins |
| **Speed** | 70x realtime | Slower (more stages) | WhisperX optimized |
| **VRAM** | <8GB (batch_size adjust) | Higher (Demucs optional) | |
| **v4 roadmap** | Improved diarization TBD | Community-driven | |

### Recommendation by Use Case

- **Best open-source DER, research only:** DiariZen Large-v2
- **Best open-source DER, commercial:** NeMo Sortformer or pyannote.audio 3.1
- **Streaming real-time:** NeMo Streaming Sortformer v2
- **Full ASR+diarization, fastest:** WhisperX
- **Full ASR+diarization, fully permissive:** whisper-diarization
- **Custom research pipeline:** SpeechBrain
- **Avoid HF token requirement:** NeMo, whisper-diarization

---

## 6. Claims & Evidence

### C1: DiariZen achieves ~9.1% DER on VoxConverse, best open-source as of mid-2026
- **Evidence:** Official README benchmark table (v2 model). Third-party arXiv:2509.26177 reports 13.3% overall multilingual DER, best among open-source.
- **Confidence:** high
- **Status:** supported

### C2: NeMo Sortformer offers Apache 2.0 diarization with streaming capability
- **Evidence:** NVIDIA NeMo docs (nvidia.com/nemo/speech), HF models `nvidia/diar_sortformer_4spk-v1` and `nvidia/diar_streaming_sortformer_4spk-v2`.
- **Confidence:** high
- **Status:** supported

### C3: WhisperX inherits pyannote.audio diarization quality
- **Evidence:** WhisperX README explicitly states "Multispeaker ASR using speaker diarization from pyannote-audio." DiarizationPipeline class wraps pyannote.
- **Confidence:** high
- **Status:** supported

### C4: whisper-diarization avoids HF token requirement for diarization
- **Evidence:** Architecture uses NeMo MarbleNet (VAD) and TitaNet (embeddings) — both Apache 2.0, ungated on HF. License file: BSD-2-Clause.
- **Confidence:** high
- **Status:** supported

### C5: pyannote.audio 3.1 is gated and requires HF token
- **Evidence:** HF model card: "visit hf.co/pyannote/speaker-diarization and accept user conditions." VexaScribe guide confirms gating.
- **Confidence:** high
- **Status:** supported

### C6: SpeechBrain trails pyannote and NeMo on diarization benchmarks
- **Evidence:** Multiple comparison articles (brasstranscripts.com, assemblyai.com) rate SpeechBrain as "good for researchers" but "Pyannote more polished for production" and "slightly behind pyannote 3.x on benchmarks."
- **Confidence:** medium
- **Status:** supported

### C7: DiariZen weights are CC-BY-NC-4.0 (non-commercial)
- **Evidence:** MODEL_LICENSE file in repo: "MIT license for code... CC BY-NC 4.0 for model weights." README compliance note explicitly states non-commercial only.
- **Confidence:** high
- **Status:** supported

---

## 7. Evidence Index

| ID | Source Label | URL | Provenance | Key Excerpt |
|---|---|---|---|---|
| E1 | DiariZen README | https://github.com/BUTSpeechFIT/DiariZen | Official repo | Benchmark table: 9.1% VoxConverse for v2 |
| E2 | DiariZen arXiv tutorial | https://arxiv.org/html/2604.21507 | Academic paper | "Leading open-source state of the art at the time of writing" |
| E3 | Benchmarking Diarization Models | https://arxiv.org/html/2509.26177 | Third-party (ETH Zurich) | PyannoteAI 11.2%, DiariZen 13.3% overall |
| E4 | pyannote.ai benchmark | https://www.pyannote.ai/benchmark | Vendor site | DER comparison tables |
| E5 | pyannote.audio README | https://github.com/pyannote/pyannote-audio | Official repo | MIT license, gated models, community-1 |
| E6 | NVIDIA NeMo Sortformer docs | https://docs.nvidia.com/nemo/speech/nightly/ | Official docs | Apache 2.0, streaming + offline |
| E7 | NeMo HF models | https://huggingface.co/nvidia/diar_sortformer_4spk-v1 | HF model card | Sortformer architecture description |
| E8 | WhisperX README | https://github.com/m-bain/whisperX | Official repo | 70x realtime, <8GB VRAM, pyannote diarization |
| E9 | whisper-diarization README | https://github.com/MahmoudAshraf97/whisper-diarization | Official repo | BSD-2, NeMo VAD/embeddings |
| E10 | SpeechBrain ECAPA-TDNN | https://speechbrain.readthedocs.io/en/latest/ | Official docs | Apache 2.0, speaker verification |
| E11 | VexaScribe pyannote guide | https://vexascribe.com/pyannote-audio | Third-party guide | DER figures, licensing notes |
| E12 | BrassTranscripts comparison | https://brasstranscripts.com/blog/speaker-diarization-models-comparison | Third-party comparison | "Pyannote 3.1 wins on balance for open-source" |

---

## 8. Gaps / Unresolved

1. **WhisperX v4:** No public release date. "Significantly improved diarization" claimed but unverified.
2. **NeMo Sortformer long-form:** Quadratic attention scaling — practical limit on audio length unquantified in public docs.
3. **DiariZen streaming:** No online/streaming variant available yet.
4. **Non-English DER:** Most benchmarks focus on English; multilingual DER comparison limited.
5. **Real-time factor for DiariZen:** No published RTF or latency numbers.
6. **RAMC/DIHARD legal:** DiariZen's CC-BY-NC-4.0 is self-imposed; legal risk of commercial use unclear.

---

## 9. Confidence

**Level:** medium

**Rationale:**
- DER figures come from official READMEs and one third-party benchmark. Independent replication of all figures not verified.
- Architecture descriptions sourced from official docs and papers — high confidence.
- Licensing confirmed from LICENSE files in repos.
- Performance/speed claims (WhisperX 70x) from README — not independently benchmarked in this research.
- Streaming Sortformer paper is from Aug 2025; real-world deployment experience may be limited.

---

## 10. Recommended Next Actions

1. **For immediate deployment (commercial):** Benchmark NeMo Sortformer vs. pyannote.audio 3.1 on your own audio — both are ungated/commercially usable.
2. **For research best-accuracy:** Use DiariZen Large-v2 — but respect NC license or contact authors.
3. **For ASR+diarization:** Start with WhisperX for speed; switch to whisper-diarization if pyannote gating is a blocker.
4. **Watch:** WhisperX v4 release for improved diarization; DiariZen streaming variant if released.
5. **Legal review:** If deploying DiariZen commercially, consult legal counsel on CC-BY-NC-4.0 implications.

---

*Report generated by Geraldo v2.2 (GIU). Sources verified against official repos, HF model cards, and third-party benchmarks as cited. No cloud APIs were called for this research.*
