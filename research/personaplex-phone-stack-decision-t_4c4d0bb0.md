# PersonaPlex for the phone stack — GO / NO-GO decision

**Task:** `t_4c4d0bb0` · **Board:** research · **Profile:** geraldov21 (Geraldo)  
**Date (UTC):** 2026-10-28  
**Slug:** `personaplex-phone-stack-decision-t_4c4d0bb0`  
**Parent context:** `/home/ice/know/research/business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md` (Print Junkie / Google Voice spam + quote agent)

---

## objective

Decide whether NVIDIA **PersonaPlex-7B** (full-duplex speech-to-speech, Moshi-based) should be used for Frank's Print Junkie business-phone stack (spam gate + quote agent on Telnyx / LiveKit / Pipecat-class telephony), including: what it is, tool-calling reality, VRAM on ~16 GB consumer GPUs, license, telephony fit, and an explicit GO / NO-GO vs the prior cascade recommendation (Patter or LiveKit Agents + STT/LLM/TTS + PJ pricing tool).

## summary

**Verdict: NO-GO as the primary quote-agent brain for Phase 2. CONDITIONAL GO only as a later duplex R&D / naturalness spike behind the already-recommended Telnyx challenge gate — after cascade quote tools work.**

PersonaPlex is a real, open, commercially usable full-duplex S2S model (7B, Moshi + Helium backbone) with hybrid **text role prompts + audio voice prompts**, shipped server/UI (`python -m moshi.server`, port 8998, Opus over WebSocket), paper-reported conversational latencies on the order of **~0.17–0.24 s** on Full-Duplex-Bench categories, and strong customer-service *role adherence* in NVIDIA's Service-Duplex-Bench. Code is **MIT**; weights are **NVIDIA Open Model License** (commercially usable; gated HF accept required). Base bf16 needs ~**14 GB** VRAM; community **4-bit NF4** paths report ~**8–9.6 GB** and target **12–16 GB** cards (e.g. RTX 4070/4080). So **16 GB is feasible** for bare conversational inference with community quant — **not** the blocker.

The blockers for *this* product are product-shaped, not GPU-shaped:

1. **No native tool / function calling.** NVIDIA paper conclusion: *"Future work will explore … integration with external tools."* Official server is speech ↔ speech (+ inner text monologue tokens), not OpenAI-style tools.
2. **Community tool-calling exists but is a hybrid** (e.g. `abhinavpgagi/personaplex-tool-calling`): external LLM decides tools from the running transcript; results are **teacher-forced into the monologue one token per 80 ms frame**. That preserves duplex voice, but it is **not** self-contained S2S, needs an LLM API key, SSRF allowlists, and reports **~19 GB on a 24 GB A10G** with little headroom — worse than bare 4-bit PersonaPlex and hostile to multi-worker GPU sharing ("one conversation per process").
3. **No first-party Telnyx / LiveKit / Pipecat / SIP path.** Official I/O is browser/WebSocket Opus at Moshi rates (**24 kHz**, **12.5 Hz** frames). Telephony remains the prior plan: Telnyx (+ optional LiveKit SIP). PersonaPlex would need a **custom media bridge** (resample, packetize, session lifecycle, DTMF/AMD out-of-band).
4. **Quote correctness is a tool problem.** Prior research already required a hard PJ pricing tool and forbade vibe prices. Cascade stacks (Patter / LiveKit Agents / Pipecat) are purpose-built for that. PersonaPlex's strength is *natural duplex persona speech*, which does not unlock quote math.

**Relative to parent recommendation:** Keep **Phase 0–1** (GV hygiene + Telnyx DID + free screening + asymmetric challenge) unchanged. Keep **Phase 2** as **Patter or LiveKit Agents + Telnyx + PJ tool**. Treat PersonaPlex as **optional Phase 2.5 lab** on a dedicated GPU once quotes and spam metrics are green — not the critical path.

---

## plan

| task_id | title | status |
|---|---|---|
| T1 | Primary sources (HF/GitHub/paper/NVIDIA page) | done |
| T2 | Tool-calling / agent hooks | done |
| T3 | Hardware + 16 GB feasibility | done |
| T4 | Telephony / phone-stack fit vs parent architecture | done |
| T5 | License / commercial | done |
| T6 | Decision matrix + publish | done |

**Assumptions:** Same as parent — US AZ small biz, ~10 spam calls/hour on GV, need keep-number path, agent must tool-call real PJ pricing, prefer self-host / existing Telnyx skills, ~16 GB class GPU available for experiments.

---

## findings

### 1. What PersonaPlex is

| Item | Fact | Source |
|---|---|---|
| Product | Real-time **full-duplex speech-to-speech** conversational model with **persona control** | NVIDIA ADLR page; GitHub README; arXiv:2602.06053 |
| Size / lineage | **7B** params; finetune of **Kyutai Moshi**; LM backbone **Helium** | README; ADLR; paper |
| Conditioning | **Hybrid system prompt**: text role tokens on agent-text channel + short **voice sample** on agent-audio channel (zero-shot voice cloning) | Paper §3.1; ADLR architecture |
| Streams | Three Moshi-style streams: **user audio**, **agent text**, **agent audio** | Paper §3.1 |
| Codec / rates | **Mimi** neural codec; loaders default **SAMPLE_RATE = 24000**, **FRAME_RATE = 12.5** Hz | `moshi/moshi/models/loaders.py` |
| Frame text | Emits audio tokens + **inner-monologue text token** on a fine time grid (community tool docs: **one text token / 80 ms**) | Community tool-calling README; Moshi architecture |
| Training emphasis | Synthetic **customer-service** + fixed **assistant** roles; released ckpt also **Fisher English** (~7,303 conv / 1,217 h) for natural backchannels | Paper appendix A; README prompting guide |
| Official runtime | `pip install moshi/.` then `python -m moshi.server --ssl $SSL_DIR` → Web UI **:8998**; offline `python -m moshi.offline` | GitHub README; Dockerfile `EXPOSE 8998` |
| Voices | Packaged NAT/VAR female/male embeddings (NATF0–3, NATM0–3, VARF0–4, VARM0–4) | README |
| Benchmarks (released) | Full-Duplex-Bench table row includes smooth-turn latency **0.170 s**, interruption latency **0.240 s**, GPT-4o judge **4.290** (paper Table 6 excerpt) | arXiv HTML |
| ADLR marketing latency chart | PersonaPlex average latency bars competitive vs Moshi / Freeze-Omni / Gemini Live / Qwen2.5-Omni (chart on project page; not a telephony E2E number) | research.nvidia.com/labs/adlr/personaplex/ |
| Repo heat | GitHub `NVIDIA/personaplex` ~**11.7k** stars (API snapshot 2026-10-28) | GitHub API |
| Paper date | arXiv **2602.06053** v1 **2026-01-14**; ADLR post **2026-01-15** | arXiv API; ADLR |

**Not what it is:** Not a drop-in SIP PBX, not an ASR-only model, not a TTS-only model, not an official agent SDK with tools, not a multi-tenant inference server by default.

### 2. Tool calling / agent hooks — the decisive gap

| Layer | Native PersonaPlex | Implication for PJ quotes |
|---|---|---|
| Official Moshi server | Bidirectional **Opus audio** + server→client **text token stream**; text/voice **prompts** at session start | Can play "customer service persona" with facts stuffed into the prompt text — **static context**, not live pricing API |
| Paper roadmap | Explicit **future** "integration with external tools" | Confirms tools are **out of scope** for v1 research release |
| Prompt stuffing | Service examples embed menus, prices, schedules in the role string | OK for demos; **unsafe** for live print quotes (stale, inventable, no audit) |
| Community: `abhinavpgagi/personaplex-tool-calling` | External **LLM** watches transcript + monologue; HTTP tools via allowlisted hosts; **force text tokens** into monologue so model *speaks* result in-voice | Closest path to "tool-using duplex"; **extra moving parts + VRAM + LLM $** |
| Community caveats (same card) | Not self-contained S2S; **one LMGen / process lock**; multi-worker on one GPU "collapsed"; **~19 GB on 24 GB A10G**; OOM under load; needs HF gate + `LLM_API_KEY` | Poor fit as sole production brain on a shared 16 GB box |

**Conclusion on tools:** For Print Junkie, **required** capability is structured tool call → pricing engine → spoken confirmed numbers. That is **first-class** in LiveKit Agents / Patter / Pipecat + LLM. It is **second-class / research** on PersonaPlex. Do not bet the quote agent on monologue injection until a measured spike proves reliability on *your* tool schemas.

### 3. Hardware — will it run on ~16 GB?

| Configuration | Reported VRAM | Real-time? | Notes | Confidence |
|---|---|---|---|---|
| Official bf16 | ~**14 GB** weights floor; practical headroom higher under load | Yes on datacenter / 24 GB+ | NVIDIA README offers `--cpu-offload` (needs `accelerate`) when GPU short | High (matzelle + bnb card + cookertron agree ~14 GB bf16) |
| bitsandbytes **NF4** selective quant (brianmatzelle) | **~9.6 GB** load | Yes (compile + CUDA graphs kept) | Quantize **main temporal transformer only**; Mimi + depformer + embeds stay bf16 for audio quality | High for that recipe |
| cookertron 4-bit fork | Claims **~8 GB** 4-bit / **~12 GB** 8-bit | Intended yes on 4080/4090-class | Community fork; validate yourself | Medium (README claims; not NVIDIA) |
| ComfyUI plugin claims | 4-bit ~5–7 GB / 8-bit ~8–10 GB | Varies | Third-party; treat as unverified marketing | Low |
| Official on 4-bit | NVIDIA org (HF discussion): **no plans** for official 4-bit for 8 GB; FP8 weights-only for **16 GB** mentioned as WIP (Feb 2026 discussion snippet) | N/A | Prefer community recipes until NVIDIA ships | Medium |
| Tool-calling hybrid stack | **~19 GB** on A10G 24 GB | Fragile | Conflicts with "16 GB only" if tools required in-process | Medium–high (author self-report) |
| GGUF / moshi.cpp Q4_K | Fits smaller VRAM; quality tradeoffs if codec quantized | Maybe | Different runtime than official Python server | Medium |

**16 GB decision:** **Bare PersonaPlex conversational server: YES** with community 4-bit (plan ~10 GB + OS/CUDA slack). **PersonaPlex + in-process tool hybrid as documented: NO** on 16 GB — needs ~24 GB class or split processes (S2S on one GPU, reasoner elsewhere). **Cascade STT/LLM/TTS:** still the easier multi-model packing story on one 16 GB card (smaller specialized models).

### 4. Telephony / stack fit

Parent architecture (unchanged recommendation):

```
[Caller] → [GV DID] → forward → [Telnyx DID]
                         → free Inbound Call Screening
                         → challenge gate (say "Print Junkie" / random digits)
                         → pass: Frank ring OR quote agent
```

PersonaPlex insertion options:

| Pattern | Description | Effort | Risk |
|---|---|---|---|
| A. Replace agent with PersonaPlex only | Telnyx media → custom bridge → Moshi WS Opus | High | **No reliable live quotes** without tool layer |
| B. PersonaPlex + community tool fork | Same + external LLM tool loop | Very high | VRAM, SSRF, single-session, unproven on PJ math |
| C. Cascade primary (Patter/LiveKit); PersonaPlex lab | Quotes/tools on cascade; duplex model on spare GPU for naturalness experiments | Medium | Correct product sequencing |
| D. PersonaPlex as pre-agent "human sound" greeter only | After challenge, short duplex greeting then transfer to cascade/human | Medium | Two media stacks; limited ROI vs TeXML TTS greeting |

**Official integration surface:**

- WebSocket binary protocol (handshake / Opus audio / text) — client in repo; not SIP.
- Docker image based on CUDA 12.4 runtime, Opus dev libs, `moshi.server --ssl`.
- No matches found for first-party **PersonaPlex + Telnyx/Twilio/LiveKit** recipes in search (Telnyx↔LiveKit docs exist independently and remain the right telephony spine).

**Audio engineering tax if you still try:** PSTN/Telnyx often 8 kHz / 16 kHz PCM or provider frames; Moshi wants **24 kHz** Opus path in the reference server. Bridge must resample, absorb jitter, map hangup/transfer, and keep **full-duplex** (no half-duplex barge assumptions from simple TTS bots). Challenge-gate DTMF should stay on **Telnyx Call Control/TeXML**, not inside PersonaPlex.

### 5. License / commercial

| Component | License | Practical read |
|---|---|---|
| `NVIDIA/personaplex` code | **MIT** (`LICENSE-MIT`) | Permissive for server forks/bridges |
| Weights `nvidia/personaplex-7b-v1` | **NVIDIA Open Model License** | NVIDIA states models are **commercially usable**; derivative models allowed; NVIDIA **does not claim ownership of outputs**; redistribution requires license copy + Notice attribution; litigation/guardrail circumvention termination clauses; **indemnify NVIDIA** for third-party claims from your use; export/sanctions compliance |
| Base Moshi (Kyutai) | **CC-BY-4.0** (per ADLR availability blurb) | Attribution obligations on that lineage |
| HF access | **Gated** — must accept license on model card; `HF_TOKEN` required | Ops step before any deploy |
| Legal hygiene | Not a substitute for counsel | For a small print shop internal agent: license posture looks **usable**; still accept HF terms, keep Notice files, don't strip guardrails, don't redistribute weights without license text |

**Not a license NO-GO** for internal PJ phone agent use, assuming HF accept + commercial Open Model terms remain as retrieved 2026-10-28.

### 6. Decision matrix (this task)

| Criterion | Weight for PJ | PersonaPlex | Cascade (Patter / LiveKit + LLM tools) | Winner |
|---|---|---|---|---|
| Live structured **quote tools** | Critical | Weak / community-only | Strong | Cascade |
| Spam **challenge gate** | Critical | Wrong layer | Wrong layer (use Telnyx) | Telnyx (prior) |
| Natural full-duplex speech | Nice | **Best-in-class open** | Good enough with streaming TTS | PersonaPlex |
| Time-to-production on Telnyx | High | Custom bridge | Documented | Cascade |
| 16 GB single-GPU | High | OK bare 4-bit; poor with tool hybrid | Manageable | Tie / cascade easier multi-model |
| License | Medium | OK commercial Open Model | Apache/MIT typical | Tie |
| Multi-call concurrency | Medium | One heavy session / GPU typical | Scale with workers + smaller models | Cascade |
| Prompt-only "menu brain" demos | Low | Excellent | Excellent | Tie |

**Roll-up:** PersonaPlex wins the *voice naturalness* column and loses the *business-critical tool + telephony time-to-value* columns that define this project.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | PersonaPlex-7B is an open full-duplex S2S model (Moshi-based) with text role + voice prompts | E1 E2 E3 E4 | high | supported |
| C2 | Official release does **not** implement native external tool/function calling; paper lists tools as future work | E3 E5 | high | supported |
| C3 | Community tool-calling via monologue token forcing + external LLM exists and preserves duplex speech | E6 | high | supported |
| C4 | Community tool stack self-reports ~19 GB on 24 GB GPU and single-conversation process locking | E6 | medium | supported |
| C5 | bf16 footprint ~14 GB; selective 4-bit NF4 ~9.6 GB real-time on 12 GB-class cards is documented by independent implementers | E7 E8 E9 | high | supported |
| C6 | ~16 GB VRAM is sufficient for **bare** quantized PersonaPlex conversational use | E7 E8 E9 | high | supported |
| C7 | ~16 GB is **insufficient / unsafe** as sole host for the documented in-process tool-calling hybrid | E6 | medium | supported |
| C8 | Official I/O is Moshi WebSocket/Opus server (:8998), not SIP/Telnyx/LiveKit-native | E1 E10 E11 | high | supported |
| C9 | Code MIT; weights NVIDIA Open Model License with commercial use affirmed on NVIDIA license page | E1 E2 E12 | high | supported |
| C10 | HF weights are gated; token + license accept required | E1 E13 | high | supported |
| C11 | Released checkpoint Full-Duplex-Bench latencies include ~0.17 s and ~0.24 s category figures (lab benchmark, not PSTN E2E) | E3 | high | supported |
| C12 | For PJ quote agent + spam phone plan, cascade (Patter/LiveKit + Telnyx + tools) remains the correct Phase-2 primary | E14 + C2–C8 | high | supported |
| C13 | PersonaPlex should be NO-GO as primary agent; CONDITIONAL later lab behind Telnyx gate | synthesis C1–C12 | high | supported |
| C14 | No first-party PersonaPlex↔Telnyx integration guide found in this pass | E15 | medium | partial |

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | NVIDIA/personaplex README | https://github.com/NVIDIA/personaplex | curl raw 200; 2026-10-28 | Install `moshi/.`; `moshi.server --ssl`; HF gate; MIT code + NVIDIA Open Model weights; voice/prompt guide; `--cpu-offload` |
| E2 | NVIDIA ADLR project page | https://research.nvidia.com/labs/adlr/personaplex/ | curl HTML 200 | Full duplex; hybrid voice+text prompts; 7B Moshi/Helium/Mimi; availability MIT + Open Model; benchmark charts |
| E3 | arXiv 2602.06053 | https://arxiv.org/abs/2602.06053 · HTML | arXiv API + HTML extract | Abstract role+voice duplex; §3.1 three streams + hybrid prompt; conclusion future external tools; Table 6 released latencies 0.170 / 0.240 |
| E4 | loaders.py constants | https://github.com/NVIDIA/personaplex/blob/main/moshi/moshi/models/loaders.py | curl raw | `SAMPLE_RATE = 24000`, `FRAME_RATE = 12.5`, `DEFAULT_REPO = 'nvidia/personaplex-7b-v1'` |
| E5 | Paper conclusion tools | same as E3 | HTML extract | "Future work will explore post-training alignment and integration with external tools." |
| E6 | personaplex-tool-calling model card | https://huggingface.co/abhinavpgagi/personaplex-tool-calling | curl raw README 200 | Monologue force-feed; external LLM; ~19 GB A10G; one process lock; SSRF allowlist; not self-contained S2S |
| E7 | brianmatzelle bnb-4bit card | https://huggingface.co/brianmatzelle/personaplex-7b-v1-bnb-4bit | curl raw README 200 | bf16 ~14 GiB → NF4 ~9.6 GiB; RTX 4070+; selective quant |
| E8 | Matzelle blog | https://www.matzelle.co/blog/2026-02-15-personaplex-4bit-quantization | curl 200 | Implementation bugs/fixes; 9.6 GiB real-time; vs GGUF tradeoffs |
| E9 | cookertron quantized fork | https://github.com/cookertron/personaplex-7b-quantized | curl raw README 200 | 4-bit ~8 GB / 8-bit ~12 GB claims for 16 GB consumer GPUs |
| E10 | server.py | https://github.com/NVIDIA/personaplex/blob/main/moshi/moshi/server.py | curl raw | Opus loops; default port 8998; cuda device; cpu_offload loader flag |
| E11 | Dockerfile | https://github.com/NVIDIA/personaplex/blob/main/Dockerfile | curl raw | CUDA 12.4.1 runtime; libopus; EXPOSE 8998; `moshi.server --ssl` |
| E12 | NVIDIA Open Model License | https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-license/ | curl HTML 200 | "Models are commercially usable"; derivatives OK; no claim on outputs; Notice on redistribute; indemnity; export |
| E13 | HF gated raw | https://huggingface.co/nvidia/personaplex-7b-v1 | curl without auth | "Access to model … is restricted" |
| E14 | Parent GIU report | `/home/ice/know/research/business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md` | local know archive | Telnyx + challenge + Patter/LiveKit cascade + PJ tool; phased GO plan |
| E15 | Web search telephony×PersonaPlex | web_search 2026-10-28 | search | No first-party PersonaPlex SIP/Telnyx hits; abundant Telnyx↔LiveKit/Pipecat docs independent of PersonaPlex |
| E16 | GitHub API repo metadata | https://api.github.com/repos/NVIDIA/personaplex | curl JSON 200 | MIT license key; stars ~11719; created 2026-01-05 |

---

## gaps

1. **No on-host bench in this task** — did not download gated weights or measure RTX 16 GB VRAM/latency locally (HF auth not in scope).
2. **PSTN E2E latency unknown** — paper/ADLR numbers are duplex-bench lab figures, not Telnyx↔bridge↔PersonaPlex round-trip.
3. **NVIDIA FP8 16 GB path** — mentioned in HF discussion as in-progress; status not verified as released.
4. **cookertron / Comfy quant quality** — claims not independently verified; prefer matzelle-style selective NF4 narrative until smoke-tested.
5. **Multi-session concurrency design** — official server concurrency model not fully audited beyond community "one LMGen lock" report.
6. **Counsel review** of NVIDIA Open Model + indemnify clause for production customer-facing audio — recommended before public marketing claims.
7. **English-centric** training story (Fisher + synthetic service English) — non-English caller handling not evaluated.
8. **Firecrawl extract 403** this run — mitigated via curl/raw GitHub, arXiv HTML, and HF raw where ungated.

---

## confidence

| level | rationale |
|---|---|
| **high** (decision) | Primary architecture, license split, lack of native tools, and telephony non-integration are anchored in first-party docs/paper. Quantized 16 GB feasibility is multi-source. Parent cascade plan already matches the missing capabilities. |
| medium (perf numbers) | Exact VRAM under *your* drivers/CUDA and PSTN latency need a spike to harden. |
| medium (community tool path) | README is detailed and credible but single-project; not production-hardened for PJ. |

**Report-level confidence:** `{ "level": "high", "rationale": "GO/NO-GO rests on first-party capability gaps (tools, SIP) that quantization cannot fix; 16GB feasibility is secondary and supported." }`

---

## recommended_next_actions

### Decision (locked for orchestrator)

| Decision | Verdict | Why |
|---|---|---|
| PersonaPlex as **primary** PJ quote agent | **NO-GO** | No native tools; telephony bridge tax; quote correctness needs cascade tool loop |
| PersonaPlex on **16 GB** for duplex chat demo | **GO (lab)** | 4-bit community path ~8–10 GB class |
| PersonaPlex + community tool fork as production | **NO-GO now** | ~19 GB class reports; complexity; unproven on PJ pricing |
| Replace Telnyx challenge gate with PersonaPlex | **NO-GO** | Wrong layer; keep Telnyx screening + asymmetric challenge |
| Change parent Phase 0–1 spam plan | **NO-GO / no change** | Orthogonal and still correct |
| Phase 2 primary stack | **Keep Patter or LiveKit Agents + Telnyx + PJ tool** | Matches requirements |
| Later duplex naturalness R&D | **CONDITIONAL GO** | Only after quotes+spam metrics; dedicated GPU; success criteria below |

### If CONDITIONAL lab spike is funded (optional child card)

**Success criteria (fail closed back to cascade-only):**

1. HF license accepted; bare `moshi.server` 4-bit stays under **12 GB** steady on target GPU with 5 min duplex dialogue.
2. Bridge prototype: Telnyx media (or LiveKit room PCM) ↔ Opus/WS with hangup clean; measure p50/p95 reply latency on PSTN.
3. Tool path: either (a) reject tools and transfer-to-human/cascade for any price, or (b) community injector correctly returns **fixture** PJ quote JSON spoken without hallucinated dollars on 20 scripted calls.
4. Concurrency: document max simultaneous calls per GPU (expect 1).
5. Do **not** point public GV marketing at PersonaPlex until (1–3) green.

**Illustrative lab architecture (not Phase-2 production):**

```
Caller → Telnyx DID → screening → challenge
              │ pass
              ▼
     [media bridge: resample ↔ 24 kHz Opus WS]
              │
              ▼
     PersonaPlex moshi.server (4-bit, 1 session/GPU)
              │ monologue text (optional)
              ▼
     external reasoner LLM ──tool──► PJ pricing API
              │ force tokens / OR blind transfer
              ▼
     spoken answer in-persona   / cascade agent takes over
```

### Swarm follow-ups (create only if Frank wants the lab)

1. **Builder:** Telnyx↔Moshi Opus bridge spike (assignee: coding profile), parent `t_4c4d0bb0`.
2. **Builder:** PJ pricing tool remains on cascade path (already implied by `t_c4f16b85`) — do not block on PersonaPlex.
3. **Ops:** HF gate accept + GPU free schedule if lab approved.

### GO / NO-GO one-liner

**Ship the phone product on Telnyx + challenge + LiveKit/Patter cascade with a real quote tool. Do not wait on PersonaPlex. Revisit PersonaPlex only as a duplex UX upgrade after the money path works.**

---

## appendix — source checklist (acceptance)

| Required source | Retrieved? |
|---|---|
| https://github.com/NVIDIA/personaplex | yes (README, Dockerfile, server.py, loaders.py, API) |
| https://huggingface.co/nvidia/personaplex-7b-v1 | gated without token; card metadata via search + license links; accept-wall confirmed |
| https://research.nvidia.com/labs/adlr/personaplex/ | yes |
| https://arxiv.org/abs/2602.06053 | yes (API + HTML) |
| Community quant / tools | cookertron, brianmatzelle, abhinavpgagi tool-calling |
| Parent stack report | `business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md` |

## appendix — comparison snapshot

| | PersonaPlex primary | Cascade primary (recommended) |
|---|---|---|
| Spam kill | Needs Telnyx in front either way | Same |
| Quotes | Prompt fantasy or heavy custom tools | Native tool calling |
| Sound | Excellent duplex | Good streaming cascade |
| 16 GB | Bare yes / tools no | Yes with model choices |
| Weeks to useful | Bridge + research | Days with existing docs/skills |
| Verdict | Lab only | **Ship** |
