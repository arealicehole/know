# Business phone spam defense + voice-agent migration (Google Voice / Print Junkie)

**Task:** `t_c4f16b85` · **Board:** research · **Profile:** geraldov21 (Geraldo)  
**Date (UTC):** 2026-10-27  
**Slug:** `business-phone-spam-voice-agent-gv-pj-t_c4f16b85`

---

## objective

Research keep-the-number spam defense for Frank's high-spam Google Voice (Print Junkie) business line (~10 scam calls/hour), compare carriers and self-host voice-agent stacks that can deliver real print quotes (tool-call into PJ pricing math), and recommend a phased GO/NO-GO plan with dollar estimates — without cold-abandoning the existing DID.

## summary

**Bottom line:** At ~10 scam calls/hour, Google Voice's built-in AI spam filter alone is insufficient (community + product-expert evidence: Workspace spam still rings ring groups; auto-attendant DTMF is the strongest *native* GV lever). The **fastest keep-number path** is: (1) tighten GV spam + auto-attendant if Workspace, (2) buy a cheap Telnyx DID, (3) **link-forward GV → Telnyx** with landline "verify by phone" + auto **press-1** accept, (4) put **Telnyx free Inbound Call Screening** (STIR/SHAKEN + reputation blocklist, free, default ON) in front of a **challenge gate** (say business name / press random digit / short speech CAPTCHA), then only human-passed calls reach Frank or a quote agent.

**Human scam farms defeat pure "press 1".** DTMF-only gates kill robocalls well but live call-center agents press keys. Prefer **asymmetric challenges** (spoken business name, random multi-digit, acoustic/speech CAPTCHA) + hang-up on failure + optional blocklist of failed CLIDs. Expect residual spam; measure pass-rate before declaring victory.

**Carrier pick for this stack:** **Telnyx first** (free inbound screening, owned network, ~$0.0032–0.0052/min inbound depending on SIP vs Voice API path, $1/mo local DID, first-class LiveKit + TeXML docs, existing Hermes Telnyx skills). Twilio is fine for speed-to-first-call and docs but ~2× list voice and no free screening product matching Telnyx's. SignalWire is strong for FreeSWITCH-native control and cheap SIP (~$0.003/min SIP class rates in third-party Oct 2026 roundups) but less "drop-in screening product." Plivo is cheapest CPaaS sticker for plain voice; weaker AI-agent ecosystem fit here.

**Port GO/NO-GO:** Port is **feasible** (personal GV: unlock ~$3 then port-out LOA; Workspace: admin PIN/code then receiving carrier drives port). **Do not port in week 1.** Port only after forward-path proves spam kill-rate and quote-agent quality for 2–4 weeks. Porting is the slow/fragile step (days–weeks; LOA name/address must match exactly).

**Voice agent for PJ quotes:** Prefer **self-host orchestration + Telnyx trunk**:
1. **Patter (MIT)** — fastest path to "agent on a Telnyx number" with Python/TS SDK, tool calls, BYO LLM/STT/TTS.
2. **LiveKit Agents + Telnyx SIP** — production-grade real-time media, official Telnyx guides, Apache-2.0 self-host option.
3. **Pipecat + Asterisk/FreeSWITCH (AudioSocket)** — max PBX control; more ops; ~$0.03–0.08/min all-in self-host estimate (vendor engineering blogs).
4. Hosted foil only if time-critical: **Vapi ~$0.05/min platform + models** (~$0.10–0.12 all-in with Twilio) or Retell/Bland similar — higher $/min, faster demo, less ownership.

**Quote capability is not automatic.** Whatever stack is chosen must expose a **tool/function** that calls Print Junkie pricing math (sizes, substrates, quantities, finish, ship zone) and returns a structured quote; the LLM must be forbidden from inventing prices. Transfer-to-human remains the safety net for edge jobs.

**Recommended phased plan (GO):**
- **Phase 0 (same day, $0–5):** GV spam settings + optional Workspace auto-attendant; log 48h baseline of spam vs real.
- **Phase 1 (1–3 days, ~$5–20 setup + pennies/min):** Telnyx DID + Call Control/TeXML challenge + free screening; optional GV link-forward with press-1 bot.
- **Phase 2 (1–2 weeks):** Wire quote tool + voice agent (Patter or LiveKit) on Telnyx number only (parallel DID) — do not put agent on public GV until challenge metrics look good.
- **Phase 3 (after metrics):** Cut public marketing to still show GV but forward all; or soft-launch Telnyx DID on web/print.
- **Phase 4 (optional, 2–6 weeks later):** Port GV → Telnyx if forward path stable and you want one canonical carrier DID.

**Rough monthly $ at PJ volume (order-of-magnitude):** Assume 2–5 real business calls/day × 3–8 min + spam that dies in <30s screening. Carrier for screened spam is near-noise ($5–30/mo). Real answered AI minutes dominate: self-host stack ~$20–80/mo models+STT/TTS at low volume; hosted Vapi-class ~$50–200/mo. Number $1/mo. Not the cost center — **engineering time and quote correctness** are.

---

## plan

| task_id | title | status |
|---|---|---|
| T1 | GV native spam + forward/link + press-1 mechanics | done |
| T2 | Telnyx / Twilio / SignalWire screening, pricing, port | done |
| T3 | Challenge-gate efficacy vs robocall vs human scam farms | done |
| T4 | Voice-agent stacks (OSS + hosted) + quote tool-calling | done |
| T5 | Phased GO/NO-GO + cost model + pitfalls | done |
| T6 | Publish `/home/ice/know/research/...` + kanban complete | in progress |

**Assumptions:** US AZ small biz; high inbound spam mix of robocall + live scam agents; prefer Fedora self-host / existing Telnyx-Twilio skills; must keep GV DID reachable during transition; agent must compute quotes via tool, not vibe prices.

---

## findings

### A. Keep the number — spam defense on / in front of Google Voice

#### A1. What Google Voice can do natively

| Lever | What it does | Efficacy vs this problem | Notes |
|---|---|---|---|
| AI "Filter Spam" / suspected spam | Diverts known-bad to spam/VM | Partial | Workspace community (Jan 2026): admins still get spam through ring groups; "Filter Spam" UI hard to find / may have moved. Not 100%. |
| Auto attendant (Workspace Voice) | "Press 1 for …" before ring group | Strong vs **bots** | Product expert: kills ~"99%" automated spam that won't DTMF. **Weak vs human farms.** Ring group still needs a number attackers can dial if exposed. |
| Custom call forwarding rules | Contacts → ring; unknown → VM | Medium | User-level; good for known-client shops; bad if cold inbound sales are real customers. |
| Block numbers / spam report | Manual | Low at 10/hr | Doesn't scale. |
| Linked number forward | Rings external DID | Transport only | **Not spam filter.** Google docs: forwarding to automated systems is "unsupported" on consumer path; Workspace auto attendant is the supported automated path. |
| Screening of forwarded leg | GV asks **destination** to **press 1** before connect | Works for humans; must be automated for bots | AI receptionist vendors document "Pre-Call Digit Press = 1" to accept GV forwards. |
| Show GV as caller ID on forward | Masks real CLID | Turn **OFF** for spam analytics | Needed so downstream screening sees real ANI. |

**Google official constraints (supported):**
- Link up to 6 numbers; verify via SMS **or** "Verify by phone" (spoken code) — required for VoIP/AI numbers that can't SMS.
- "Forwarding calls from your Google Voice number to an automated system is unsupported" on the standard linked-number path — use Workspace auto attendant for supported automated routing, **or** accept grey-area link-forward to VoIP that automates press-1 (widely practiced by AI receptionist vendors; not Google-blessed).
- SIP Link users: linked-number feature N/A.

#### A2. Recommended keep-number architecture (no port yet)

```
[Caller] → [Google Voice DID] 
              ├─ (optional) Workspace Auto Attendant DTMF
              └─ linked forward → [Telnyx DID]
                                      ├─ Telnyx Inbound Call Screening (free; STIR/SHAKEN + blocklist)
                                      ├─ TeXML/Call Control challenge gate
                                      │     pass → Frank mobile / ring group / AI quote agent
                                      │     fail → hangup + optional block ANI
                                      └─ AMD optional ($0.002–0.0065/call) for outbound later
```

**Link-forward checklist (from AI-receptionist practice + GV docs):**
1. Buy Telnyx local DID (~$1/mo).
2. GV → Settings → Linked numbers → New → enter Telnyx DID → **Verify by phone** (not SMS).
3. Telnyx Call Control answers, captures spoken digits (or TeXML `<Gather>`), completes verification.
4. Enable forward to that linked number; disable "show my Google Voice number as caller ID when forwarding."
5. On every inbound forward, GV plays press-1 to the **Telnyx leg** — TeXML must auto-send DTMF `1` immediately (or use a short gather-accept), then start screening/challenge for the **caller** side as designed.
6. Parallel: keep voice.google.com / app as backup answer path during burn-in.

**Pitfall:** Double-audio / delayed connect while press-1 fires. Test from a non-Google phone. Some forwards present GV's number as ANI if toggle left on — breaks reputation filters.

#### A3. Challenge gates that work (and what fails)

| Gate | Kills robocalls? | Kills human scam farms? | Customer friction | Verdict for PJ |
|---|---|---|---|---|
| Silent answer / long pause | Partial | No | Low | Weak alone |
| "Press 1 to continue" | Strong | **No** (they press) | Low | Good first filter only |
| Random "press 4-7-2" | Strong | Weak–medium | Low–med | Better; rotate prompt |
| "Say the business name: Print Junkie" + STT match | Strong | Medium–strong | Low for real customers | **Recommended primary** |
| Speech CAPTCHA / acoustic verify | Strong | Stronger | Med | Best vs DTMF-emulation fraud; more build |
| Known-contacts whitelist only | N/A | Strong | High for retail print | Only if cold-call sales unwanted |
| STIR/SHAKEN A/B/C + Telnyx blocklist | Drops signed-bad / listed spam | Partial (farms use dirty DIDs) | None to good callers | **Always on (free on Telnyx)** |

**Industry notes:** Five9/TransNexus-style IVR CAPTCHA is standard advice: unpredictable tasks beat fixed "press 1." Academic/patent literature flags DTMF-emulation autodialers specifically. TNS 2025–2026 robocall reports: bad actors exploit weaker/non-IP paths — terminating on a serious CPaaS with screening beats consumer GV alone.

**Operational policy:** After N failed challenges from same ANI → temporary block. Never challenge known VIP CLID list (repeat PJ customers). Log disposition codes for weekly kill-rate.

### B. Migration options (second number, forward, port)

| Path | Keeps public DID? | Spam control | Time to value | Risk | When |
|---|---|---|---|---|---|
| **B0. GV-only harden** | Yes | Low–med | Hours | Low | Always do first |
| **B1. New Telnyx DID, no GV change** | Old DID still spammy | High on new DID only | Hours | Low | Soft launch / web |
| **B2. GV link-forward → Telnyx + screening** | Yes | High on answered path | 1–3 days | Med (GV auto-system grey area; press-1) | **Default Phase 1** |
| **B3. Workspace GV auto attendant → human/agent** | Yes | Med (DTMF) | Hours–days | Low | If already on Workspace Voice paid |
| **B4. Port GV → Telnyx/Twilio** | Yes (same E.164) | High (full control) | Days–weeks | Med–high (port rejects, downtime if cancel early) | Phase 4 after B2 proves out |
| **B5. Abandon GV get new DID** | **No** | High | Hours | High business (printed materials, Google Business, customers) | **NO-GO** unless number worthless |

#### Porting facts (evidence-backed)

**Personal Google Voice → another carrier (common path):**
1. Unlock number in GV (historically **$3** if GV-native number; free if previously ported-in) — widely documented by PBX/port guides; confirm current fee in-account before relying.
2. Screenshot unlock / account details for LOA.
3. Receiving carrier (Twilio/Telnyx) files port with matching **name + service address**.
4. Do **not** cancel/delete GV until winning carrier confirms complete.

**Twilio:** Explicit help article "Can I port my Google Voice number" — process is supported via Console porting; community/guides stress unlock PDF + LOA match. Timelines: Twilio schedules FOC date with losing carrier; not instant.

**Telnyx:** Port-in orders via Mission Control / Porting API — draft → docs → submit; async with losing carrier; times vary by carrier.

**Workspace / business Voice:** Admin gets PIN/code (US: PIN + zip of **primary Voice location**, not necessarily billing address). Receiving provider initiates; GTSP processes. Do not cancel Voice subscription until port complete.

**Twilio port timeline (general US):** multi-step FOC scheduling; business days not minutes. Plan overlap.

**GO/NO-GO on port now:** **NO-GO until Phase 1–2 metrics.** Port does not by itself stop spam — only moves the DID to a place you can filter. Forward-first achieves filtering without LOA risk.

### C. Carrier comparison (inbound screening focus)

Rates checked via vendor pages + Oct 2026 industry roundups (Fora Soft, Famulor). **Re-verify before buy** — rate cards move.

| Dimension | **Telnyx** | **Twilio** | **SignalWire** | Notes |
|---|---|---|---|---|
| US local number /mo | ~$1.00 | ~$1.15 | ~$0.50 | LiveKit Cloud also offers 1 free US inbound number on plans (Oct 2026 roundup) |
| Inbound US local (list) | SIP ~$0.0032/min; Voice API path ~$0.002 API + SIP ≈ **$0.0052** | Programmable ~$0.0085; Elastic SIP origination ~$0.0034 | Local ~$0.0066; SIP class ~$0.003 | Telnyx example bill: 40k in × $0.0032 = $128 |
| Outbound US local | ~$0.005 SIP + $0.002 API ≈ **$0.007** blended | ~$0.014 Programmable; Elastic SIP ~$0.010 | ~$0.008 local; SIP ~$0.003 | |
| **Inbound spam screening product** | **Yes — free, default ON** (STIR/SHAKEN + 300M+/day intel blocklist; Call Control + TeXML; not SIP trunking) | Reputation / SHAKEN ecosystem; no free Telnyx-equivalent "inbound call screening" product found in this pass | STIR/SHAKEN standard; DIY on Call Fabric | **Telnyx wins productized free screening** |
| Call Control / IVR | Voice API, TeXML (TwiML-like) | TwiML industry default | Compatibility API + deep SIP | All three can build challenge gates |
| AMD | $0.002 std / $0.0065 premium per call | Available (paid) | Available | Optional |
| AI agent adjacency | Conversation Relay $0.05/min; LiveKit-on-Telnyx; native STT/TTS menu | ConversationRelay ~$0.07/min (Oct 2026 comparison) | AI agent runtime (2026) | |
| Docs / ecosystem | Strong; Hermes already has Telnyx skills | Strongest general docs | FreeSWITCH DNA | |
| Fit for Frank | **Primary** | Secondary / if already deep in Twilio | If building FreeSWITCH PBX core | |

**Telnyx Inbound Call Screening detail (vendor docs, Mar 2025 feature, still documented 2026):**
- Free; no account toggle — works when number is attached to **Call Control or TeXML** application (not raw Elastic SIP trunking alone).
- Uses SHAKEN attestation + internal blocklist; blocked calls never hit your webhook (you may not see them — by design).
- Does **not** replace challenge gate for human-dialed spam.

**Implication:** Put PJ DID on **TeXML or Call Control**, not naked SIP, if you want free screening. SIP-only to LiveKit still works but you implement more filtering yourself (or screen on a Call Control app that then bridges).

### D. Voice-agent stacks (quote-capable)

#### D1. Comparison matrix

| Stack | License / host | Telephony | Tool calling for quotes | Latency path | Ops burden | $/min ballpark (ex-carrier) | Fit |
|---|---|---|---|---|---|---|---|
| **Patter** | MIT; self SDK | Twilio, **Telnyx**, Plivo | Yes (agent tools) | Designed sub-second; phone preamble helpers | Low–med | Models only | **Best quick OSS+Telnyx** |
| **LiveKit Agents** | Apache-2.0 server; Cloud optional | Official **Telnyx SIP** guides | Yes (function tools) | WebRTC-class; production | Med | Cloud SIP $0.003–0.004 + models; or self-host media | **Best production OSS media** |
| **Pipecat** | OSS (Daily-backed); v1.0 Apr 2026 | No native SIP — bridge via **Asterisk AudioSocket** or FS `mod_audio_fork` | Yes | Good if bridge tuned | Med–high | Self-host all-in ~$0.03–0.08 (CelloIP) | Best if you want PBX+AI |
| **Asterisk / FreeSWITCH alone** | GPL / MPL | Native SIP | Via AGI/ARI + your code | N/A alone | High | Carrier only | Screening IVR yes; full LLM agent needs bridge |
| **Vapi** | Hosted | Twilio / SIP | Yes | Managed | Low | ~$0.05 platform + models ≈ **~$0.10–0.12** all-in | Fast demo; $$ |
| **Retell** | Hosted | Own Twilio or SIP | Yes | Managed | Low | ~$0.11–0.135 all-in class | Fast demo |
| **Bland** | Hosted | Pass-through | Yes | Managed | Low | ~$0.11–0.14 + telephony | Fast demo |
| **Telnyx Conversation Relay** | Telnyx-hosted relay | Native | Via your app | Managed media | Low–med | $0.05/min + models | Middle ground |

#### D2. Quote-agent design (Print Junkie-specific)

**Must-have behaviors:**
1. **Identity + intent:** "Thanks for calling Print Junkie — are you calling about a new print quote, an existing order, or something else?"
2. **Spam residual:** If challenge already passed, still allow polite early hangup on clear scam scripts ("your warranty…", "Google Business listing…").
3. **Quote tool (authoritative):**
   - Inputs: product type, W×H, quantity, color (4/0, 4/4…), substrate, coating, turnaround, ship ZIP, optional finishing.
   - Output: line items, unit, total, validity window, assumptions, "not a final invoice."
   - **Never** let the LLM free-hand currency amounts.
4. **Human escape:** "press 0" or "speak to Frank" → dial Frank's cell / bridge; voicemail after hours with SMS callback link.
5. **SMS follow-up:** After quote, offer text of summary to caller's mobile (A2P 10DLC registration required for US application-to-person SMS — start early if using Twilio/Telnyx messaging).
6. **Logging:** recording optional ($0.002/min Telnyx); store transcript + tool I/O for dispute/training.

**Local AI Box note:** STT/TTS/LLM can run on existing local stack (Whisper/OmniVoice/etc. from SCOR research memory) to cut per-min cost and keep data on-box; telephony still needs public webhook/SIP reachability (Tailscale/funnel, or cloud edge worker).

#### D3. Reference wiring (Telnyx + LiveKit)

LiveKit docs: create Telnyx FQDN connection, associate number, optional HD codecs; paid Telnyx account required (trial limits). Pattern: PSTN → Telnyx SIP → LiveKit SIP service → Agent session with tools.

Patter: MIT SDK, Telnyx among first-class carriers, dual Py/TS — good match to Hermes/Python shop.

Pipecat: terminate SIP on Asterisk, `AudioSocket()` 8 kHz PCM frames to Pipecat transport — keep dialplan IVR for challenge, only escalate to AI on pass.

### E. Cost model (order-of-magnitude)

**Scenario assumptions (adjust with real CDR):**
- Spam: ~10 calls/hr × 10 hr/day × 22 day ≈ **2,200 spam calls/mo**, most die in 5–20 s after screening → ~15–150 Telnyx inbound min if answered briefly, **or ~0 min** if blocked before webhook.
- Real: 3 calls/day × 5 min × 22 ≈ **330 min/mo** answered.

| Line | Phase 1 screen-only | Phase 2 self-host agent | Phase 2 hosted Vapi-class |
|---|---|---|---|
| Telnyx DID | $1 | $1 | $1 |
| Inbound carrier mins | $1–10 | $2–20 | $3–25 |
| Screening | $0 | $0 | $0 |
| STT+LLM+TTS on real mins | $0 | ~$15–60 | included in platform |
| Platform | $0 | $0 (power/GPU sunk) | ~$0.05 × 330 ≈ $17 + models |
| **Est. total** | **~$5–15** | **~$20–80** | **~$50–200** |

Engineering time (not in table) dominates Phase 1–2.

### F. Pitfalls checklist

1. **GV "unsupported" automated forward** — may break without notice; Workspace AA is cleaner if available.
2. **Press-1 race** — bot must answer and DTMF fast or caller hears failure.
3. **Caller ID rewrite** — leave GV "show my number" off for downstream filters.
4. **Human farms pass DTMF** — don't celebrate press-1 kill charts alone.
5. **Port LOA mismatch** — name/address/PIN errors = reject; address must match GV primary location for Workspace.
6. **Cancel GV too early** — port dies; number risk.
7. **SIP trunk without Call Control** — lose free Telnyx inbound screening.
8. **LLM price hallucination** — catastrophic for print shop trust; hard-tool only.
9. **TCPA / recording consent** — if record or outbound AI, comply; inbound business answer generally OK with notice.
10. **A2P 10DLC** — required before bulk SMS quotes.
11. **E911 / address** — set correctly on Telnyx numbers.
12. **Double billing** — don't pay Telnyx recording + platform recording.
13. **Spam block false positives** — whitelist customer CLIDs; monitor "why didn't they get through."
14. **After-hours** — define VM vs AI; AI can quote 24/7 if tool-safe.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | Telnyx Inbound Call Screening is free, default-on for Call Control/TeXML numbers, uses STIR/SHAKEN + blocklist, and does not apply to plain SIP trunking the same way | E1, E2 | high | supported |
| C2 | Google Voice forwards can require the linked destination to press 1; AI systems automate this with pre-call DTMF | E3, E4 | high | supported |
| C3 | GV documents that forwarding to automated systems is unsupported on standard linked numbers; Workspace auto attendants are the supported automated routing feature | E4, E5 | high | supported |
| C4 | Workspace Voice spam still reaches ring groups without AA; experts recommend auto attendant + org spam filter | E5 | medium | supported |
| C5 | Fixed "press 1" gates are weak against live human scam agents and DTMF-emulating autodialers | E6, E7 | high | supported |
| C6 | US list rates (Oct 2026): Telnyx inbound local SIP ~$0.0032/min and Voice API $0.002/min add-on; Twilio Programmable inbound ~$0.0085; numbers ~$1 Telnyx / $1.15 Twilio / $0.50 SignalWire-Plivo class | E8, E9, E10 | high | supported |
| C7 | Personal GV numbers are commonly unlocked (~$3 historical) then ported to Twilio/Telnyx; Workspace uses admin PIN/code port-out | E11, E12, E13, E14 | medium | supported |
| C8 | Porting is multi-day and LOA-sensitive; should follow successful forward-based filtering, not lead | E12, E14, E15 | high | supported |
| C9 | Patter is MIT-licensed OSS voice-AI SDK with Telnyx support suitable for owned-stack phone agents | E16 | high | supported |
| C10 | LiveKit Agents + Telnyx SIP is a documented production pattern for phone agents | E17, E10 | high | supported |
| C11 | Pipecat needs Asterisk/FreeSWITCH audio bridge for PSTN; self-host all-in often cited ~$0.03–0.08/min | E18 | medium | supported |
| C12 | Hosted Vapi-class stacks land ~$0.10+/min all-in vs self-host models-only economics | E19, E10 | medium | supported |
| C13 | For PJ, authoritative quote tool-calling is mandatory; carrier $ is secondary to model+correctness | E10, analysis | high | supported |
| C14 | Recommended immediate path is Telnyx TeXML/Call Control screening + challenge, optional GV link-forward, defer port | synthesis | medium | partial |

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | Telnyx Help: Inbound Call Screening | https://support.telnyx.com/en/articles/8037040-inbound-call-screening | web_extract 2026-10-27 | Free; default enabled; Call Control + TeXML; STIR/SHAKEN + blocklist; blocked calls don't notify; not the same on pure SIP trunking |
| E2 | Telnyx release notes: screening free | https://telnyx.com/release-notes/inbound-call-screening-free | web_extract | Feature announcement: free screening on Voice API / TeXML |
| E3 | AI Receptionist: GV forward setup | https://ai-receptionist.com/blog/google-voice-ai-receptionist-setup/ | web_extract | Verify-by-phone; Pre-Call Digit Press 1; disable GV CLID rewrite |
| E4 | Google Voice: link numbers / forward | https://support.google.com/voice/answer/165221 | web_extract | Link ≤6; SMS or call verify; **forward to automated system unsupported**; SIP Link caveat |
| E5 | Workspace Admin Community spam thread | https://support.google.com/a/thread/405251094/spam-calls-to-workspace-voice | web_extract | Jan 2026: AA DTMF kills bots; Filter Spam; OP notes AA still leaves ring-group number |
| E6 | Five9: IVR CAPTCHA vs robocallers | https://community.five9.com/s/article/Administrator-Blocking-Robo-Callers-Using-an-IVR-Captcha | web_search | Unpredictable tasks differentiate humans vs bots |
| E7 | Acoustic/DTMF emulation fraud note | https://eureka.patsnap.com/triz-case/acoustic-call-verification-dtmf | web_search | DTMF confirmation schemes bypassed by autodialer emulation |
| E8 | Telnyx Voice API pricing | https://telnyx.com/pricing/voice-api | web_extract | $0.002/min API; SIP in $0.0032 / out $0.005 example; number $1/mo; recording $0.002; AMD $0.002/$0.0065; Conversation Relay $0.05 |
| E9 | Famulor Telnyx vs Twilio 2026 | https://www.famulor.io/blog/telnyx-vs-twilio-voice-ai-pricing-2026 | web_extract | Side-by-side US in/out API vs SIP; worked 10k min example |
| E10 | Fora Soft Twilio alternatives Oct 2026 | https://www.forasoft.com/blog/article/twilio-alternatives-voice-ai | web_extract + full cache | Rate table Twilio/Telnyx/SignalWire/Plivo/LiveKit; AI minute anatomy; Vapi $0.05 / Retell $0.055 platform |
| E11 | Reliable PBX: port out GV | https://support.reliablepbx.com/portal/en/kb/articles/port-out-a-google-voice-number | web_search | Unlock $3 unless previously ported in |
| E12 | Dialsheet: port GV/OpenPhone → Twilio | https://www.dialsheet.io/resources/port-your-number-to-twilio-from-openphone-or-google-voice | web_search | Unlock personal $3 / Workspace free unlock path; LOA match; screenshot unlock PDF |
| E13 | Google: port your GV number | https://support.google.com/voice/answer/1065667/ | web_extract | Personal vs Workspace paths; port-in $20 mobile-to-GV (different direction); admin for work accounts |
| E14 | Google Workspace: port out Voice numbers | https://support.google.com/a/answer/9530249 | web_extract | PIN/code; US PIN+zip; don't cancel until complete; primary location address |
| E15 | Telnyx port-in getting started | https://developers.telnyx.com/docs/numbers/porting/getting-started | web_search | Draft order, docs, async losing-carrier process |
| E16 | GitHub PatterAI/Patter | https://github.com/PatterAI/Patter | web_extract | MIT; Telnyx/Twilio/Plivo; Py+TS; ~1.1k stars; voice agent SDK |
| E17 | LiveKit Telnyx SIP docs | https://docs.livekit.io/telephony/start/providers/telnyx/ | web_extract | FQDN connection, number associate, paid Telnyx required |
| E18 | CelloIP Pipecat+Asterisk guide | https://celloip.com/blog/pipecat-asterisk-freeswitch-integration/ | web_extract | AudioSocket bridge; ~$0.03–0.08/min self-host; Pipecat v1.0 Apr 2026 |
| E19 | BetOnAI Vapi/Retell/Bland pricing Sep 2026 | https://betonai.net/vapi-vs-retell-vs-bland-ai-voice-agent-pricing-2026/ | web_extract | Vapi ~$0.105 realistic inbound stack; Retell ~0.11–0.135; Bland tiers |
| E20 | Google Voice custom call forwarding | https://support.google.com/voice/answer/11420769 | web_search | Rules for forward / VM by contact |
| E21 | Twilio help: port GV number | https://help.twilio.com/articles/223179728-Can-I-port-my-Google-Voice-number | web_search (page spinner on extract) | Twilio documents GV port eligibility via Console |

---

## gaps

1. **Exact account type of Frank's GV** (free personal vs Workspace Voice tier) not confirmed in-task — changes AA availability and port-out procedure.
2. **Live FOC timelines** for this specific AZ number's losing carrier not knowable until port eligibility check.
3. **Unlock fee** $3 cited by multiple secondary sources; confirm in-account before budgeting (Google primary port article emphasizes admin/personal split more than fee).
4. **Twilio Help Center full body** for GV port returned spinner-only on extract — rely on Dialsheet + Google + Twilio search snippet; re-open in browser at execution time.
5. **Empirical kill-rate** on Frank's actual spam mix (bot vs human farm) unknown until Phase 1 metrics.
6. **Print Junkie quote API/math endpoint** existence/shape not specified — may need a thin wrapper around existing shop calculator before agent tools work.
7. **SignalWire first-party screening product** not deeply documented in this pass (STIR/SHAKEN yes; free blocklist product unclear).
8. **Legal review** of call recording + AI disclosure for AZ business not done (ops should add one-line consent if recording).
9. **SMS A2P brand registration** status for PJ unknown.
10. No hands-on lab call placed in this research run (docs-only).

---

## confidence

| field | value |
|---|---|
| level | **medium-high** |
| rationale | Strong primary docs for Telnyx screening, GV link/forward limits, Telnyx/Twilio list pricing, LiveKit+Telnyx, Patter MIT, and port-out high-level flow. Medium where secondary blogs supply unlock-$3 and blended rate tables; challenge-efficacy is well-supported in principle but not measured on Frank's line; quote-tool integration depends on unpublished PJ calculator interface. |

---

## recommended_next_actions

### Ops (Frank / human) — this week

1. Confirm GV type: personal vs Workspace; screenshot spam settings; 48h tally: spam vs real vs voicemail.
2. If Workspace: enable/configure **Auto Attendant** DTMF in front of ring group; ensure org spam filter on.
3. Create/login **Telnyx** account, buy **1 local AZ-or-existing-NPA DID** (~$1), attach to **TeXML or Call Control** app (not bare SIP).
4. Implement TeXML challenge: say "Print Junkie" or random digit string → gather → connect or hangup; enable default screening.
5. Optional same week: link-forward GV → Telnyx with verify-by-phone + auto press-1; turn off GV CLID-on-forward.
6. Whitelist top 20 customer numbers.

### Build (swarm) — follow-up cards

1. **TeXML/Call Control challenge + metrics logger** (dispositions, ANI, duration).
2. **PJ quote tool API** wrapping shop math; golden-test 10 historical jobs.
3. **Patter or LiveKit agent** on Telnyx DID with quote tool + transfer-to-Frank.
4. **A2P 10DLC** registration if SMS quote receipts desired.
5. Only later: **port eligibility + LOA packet** rehearsal (no submit until GO).

### GO / NO-GO summary

| Decision | Verdict | Why |
|---|---|---|
| Abandon GV number | **NO-GO** | Printed/customer equity |
| GV spam settings only | **GO but insufficient** | Do it; won't stop 10/hr alone |
| Telnyx DID + free screening + challenge | **GO primary** | Best $/control fit; existing skills |
| GV → Telnyx link-forward | **GO conditional** | Fast keep-number; watch Google unsupported caveat |
| Port this week | **NO-GO** | Premature; LOA risk; filtering doesn't need port |
| Hosted Vapi/Retell for production | **NO-GO default** / GO for 48h demo only | $/min and lock-in vs self-host |
| Patter or LiveKit + quote tool | **GO Phase 2** | Owned stack; Telnyx-native |
| AI quotes without tool | **NO-GO** | Trust risk |

---

## appendix — minimal TeXML-shaped challenge (illustrative)

Not production-hardened; shows control flow for implementers:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Gather action="/voice/challenge" numDigits="1" timeout="5" language="en-US">
    <Say>Thanks for calling Print Junkie. If you are a real customer, press 5 now.</Say>
  </Gather>
  <Say>Goodbye.</Say>
  <Hangup/>
</Response>
```

Prefer upgrading Gather to speech with expected business-name grammar, or Call Control webhook that runs STT match, then `transfer` to Frank or agent application.

---

## appendix — phase timeline

| Day | Milestone |
|---|---|
| 0 | Baseline spam log; GV settings |
| 1 | Telnyx DID + screening + simple DTMF challenge live on **new** number |
| 2–3 | Optional GV link-forward; press-1 automation tested |
| 4–10 | Quote tool + agent on Telnyx DID; friends-and-family calls |
| 11–14 | Review kill-rate & quote accuracy; whitelist tuning |
| 15–30 | Marketing still on GV; forward stable → consider soft DID publish |
| 30–60 | Port GO only if metrics + ops appetite |

---

*End of report. Single durable archive path below.*

**Archive path:** `/home/ice/know/research/business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md`
