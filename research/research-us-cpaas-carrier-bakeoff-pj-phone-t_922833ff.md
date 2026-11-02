# US CPaaS Carrier Bakeoff — Print Junkie Phone (Unbiased 2026)

**Task:** `t_922833ff`  
**Date:** 2026-11-02 (UTC)  
**Researcher:** Geraldo v2.2 / GIU  
**Related prior:** `/home/ice/know/research/business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md` (Telnyx-leaning stack design — treated as **input**, not locked conclusion)  
**Scope:** Cost-effective + feature-rich + API-accessible US voice carrier for (1) day-1 spam shield and (2) later self-host voice agent. Arizona local DID preferred. Google Voice path considered.

---

## objective

Run an **unbiased** US CPaaS bakeoff for Print Junkie business phone needs. Re-score Telnyx fairly against Twilio, SignalWire, Bandwidth, Vonage, Plivo, and dark horses. Produce a weighted scorecard, volume-model monthly costs, primary + runner-up, when-to-choose matrix, and explicit skill-bias check (existing Telnyx/Twilio ops skills must not inflate scores).

## summary

**Primary pick: Telnyx** — still wins on the **joint** objective (cheap programmable voice **plus** day-1 free network spam/reputation screening), not because of prior skill familiarity.

**Runner-up: SignalWire** — nearly tied on base telephony cost, stronger FreeSWITCH DNA, cheaper DID ($0.50), native AI Agent product as foil; **loses** the free inbound reputation screen (DIY/Robokiller path).

**Strong third / agent-path dark horse: Plivo** — list rates near Telnyx/SW, DID $0.50, **audio streaming listed as Included** on US voice pricing (big agent-path win). Lacks Telnyx-class free network spam screen.

**Twilio** remains best ecosystem/docs but ~1.5–2× telephony cost at list and no free inbound spam filter comparable to Telnyx’s Mar 2026 free Inbound Call Screening.

**Bandwidth** rates are competitive and network-owned, with a newer self-serve “Build” path; still weaker for small-biz day-1 spam shield and DID list transparency vs peers.

**Vonage** inbound list is excellent; outbound list is Twilio-class expensive unless domestic-sales rates unlock. Fine API, weaker spam story.

**Wavix / DIDWW / VoIP.ms:** not primary. Wavix inbound local ($0.01/min) is expensive for inbound-heavy PJ traffic; better as optional SIP termination hybrid than full CPaaS agent platform.

**Skill-bias verdict:** Re-opened bakeoff **still selects Telnyx**. Bias check did surface that **Plivo** and **SignalWire** beat or match Telnyx on some agent-path unit economics; Telnyx’s decisive edge is **free multi-provider inbound reputation + STIR/SHAKEN screening** (day-1 PJ requirement), not skill lock-in.

**Day-1 path:** Buy AZ local DID on Telnyx → enable Inbound Call Screening (Reject or Flag) → TeXML/Call Control IVR/DTMF → optional simultaneous ring / later SIP to FreeSWITCH.  
**Later agent path:** Media Streaming WebSocket ($0.0035/min) or SIP to self-host; TeXML eases Twilio-shaped code portability.

**Migration note:** TeXML ≈ TwiML; SignalWire Compatibility API is also TwiML-like. Switching Telnyx↔Twilio↔SignalWire is moderate; Plivo XML is similar-but-not-identical; Bandwidth BXML is another dialect.

---

## plan

```json
{
  "objective": "Unbiased 2026 US CPaaS bakeoff for PJ phone (cost + features + API; spam day-1; agent later)",
  "success_criteria": ["fresh cited rates", "weighted scorecard", "volume $ model", "primary+runner-up+when-to-choose", "skill-bias note"],
  "tasks_completed": ["T1 prior ingest", "T2 pricing", "T3 spam", "T4 API/media", "T5 port/DID", "T6 dark horses", "T7 scorecard", "T8 publish"]
}
```

Extraction path: Firecrawl keyless 403 → **curl HTML strip + browser snapshot + Telnyx `/pricing.md`** (provenance stamped per evidence item).

---

## Volume model (task-specified)

| Scenario | Inbound min | Outbound min |
|----------|------------:|-------------:|
| Low | 300 | 100 |
| Mid | 1,000 | 300 |
| High | 3,000 | 1,000 |

Assumptions: **1× US local DID**, list PAYG, no volume commit, no taxes/regulatory surcharges, no AI/LLM tokens, no recording unless noted. Telnyx programmable = Voice API $0.002 + SIP trunk leg.

### Base telephony monthly $ (DID + minutes)

| Vendor | In $/min | Out $/min | DID $/mo | Low | Mid | High |
|--------|---------:|---------:|---------:|-----:|-----:|------:|
| **Telnyx Call Control** | 0.0052 | 0.0070 | 1.00 | **$3.26** | **$8.30** | **$23.60** |
| Telnyx SIP-only | 0.0032 | 0.0050 | 1.00 | $2.46 | $5.70 | $15.60 |
| SignalWire | 0.0066 | 0.0080 | 0.50 | $3.28 | $9.50 | $28.30 |
| Plivo | 0.0055 | 0.0115 | 0.50 | $3.30 | $9.45 | $28.50 |
| Bandwidth | 0.0055 | 0.0100 | ~1.00* | ~$3.65 | ~$9.50 | ~$27.50 |
| Vonage list (intl card) | 0.00495 | 0.01446 | 1.06 | $3.99 | $10.35 | $30.37 |
| Vonage domestic-sales† | 0.00485 | 0.00849 | 1.06 | $3.36 | $8.46 | $24.10 |
| Twilio | 0.0085 | 0.0140 | 1.15 | $5.10 | $13.85 | $40.65 |
| Wavix local | 0.0100 | 0.0065 | ~1.00* | ~$4.65 | ~$12.95 | ~$37.50 |

\*DID list incomplete / quote-ish — modeled at $1 for comparison.  
†Vonage domestic zone rates via sales per Cozzo 2026-10-31 verification of vendor page — not self-serve default.

### Agent-path mid (1,000 in + 300 out) + media stream surcharge

| Stack | Stream $/min | Mid all-in (telephony+stream) |
|-------|-------------:|------------------------------:|
| **Plivo** (stream Included on price card) | 0.0000 | **~$9.45** |
| Telnyx Media Streaming | 0.0035 | ~$12.85 |
| SignalWire `<Stream>`/Tap | 0.0030 | ~$13.40 |
| Bandwidth bidirectional | 0.0040 | ~$14.70 |
| Vonage WebSocket leg | 0.00492 | ~$16.74 |
| Twilio Media Streams | 0.0040–0.0044 | ~$19.57 |

**Implication:** If spam screening is solved separately, **Plivo** can win pure agent unit economics. PJ day-1 requirement keeps Telnyx primary.

---

## Feature matrix (yes / partial / no)

| Capability | Telnyx | Twilio | SignalWire | Plivo | Bandwidth | Vonage | Wavix |
|------------|--------|--------|------------|-------|-----------|--------|-------|
| Self-serve PAYG API | yes | yes | yes | yes | yes (Build) | yes | yes |
| US local DID inventory | yes (~$1/mo) | yes ($1.15) | yes ($0.50) | yes ($0.50) | yes (quote/Build) | yes ($1.06) | yes |
| Programmable inbound (XML/CC) | yes TeXML + Call Control | yes TwiML | yes SWML + Compatibility | yes Plivo XML | yes BXML | yes NCCO | partial (SIP-first) |
| DTMF gather / IVR | yes | yes | yes | yes | yes | yes | partial |
| Blind/attended transfer | yes ($0.10 xfer invoke) | yes (Refer $0.10 SIP) | yes (SIP Refer $0.10) | yes | yes | yes | SIP |
| SIP trunk to FreeSWITCH | yes (native) | yes (SIP interface) | yes (FS roots) | yes | yes | yes | yes (strong) |
| Bidirectional media / WS to agent | yes $0.0035/min | yes $0.0040–0.0044 | yes $0.003 | **Included** | yes $0.0035–0.004 | yes ~$0.00492 | limited/unclear |
| **Network inbound spam/reputation screen** | **yes FREE** (Flag/Reject; CallerAPI+Nomorobo+YouMail; STIR) | no* | no* (DIY/Robokiller) | no* | partial (Number Reputation products; not free day-1 shield) | no* | no* |
| STIR/SHAKEN handling | yes (screen Attestation C/Invalid) | yes (attest outbound; Voice Integrity beta) | yes (network) | yes | yes (branded calling etc.) | yes | unclear |
| Recording | $0.002/min | $0.0025/min + storage | $0.002/min | **$0 listed** | $0.002/min | $0 listed (Cozzo) | paid suite |
| AMD | $0.002–0.0065/call | $0.0075/call | $0.006/call | **$0 listed** | $0.006/call | standard $0 / adv paid | AMD product |
| Number port-in | yes (portal/API) | yes (GV unlock documented) | yes (port-in $0; port-out $5) | yes | yes | yes | yes |
| Min monthly commit | $0 PAYG | $0 PAYG | $0 PAYG | $0 PAYG | $0 Build trial | $0 | Flex tiers |
| Vendor AI receptionist (foil only) | Conversational AI | Conversation Relay etc. | **AI Agent $0.16/min** | AI Agents ~$0.03/min + tel | limited | Voice API + partners | limited |

\*Inbound spam = you build allow/block + third-party reputation API, or reject manually. Not equivalent to Telnyx free network screen.

---

## Weighted scorecard

Weights (from task): Cost 25 · API richness 20 · Spam/screening 15 · Media/SIP 15 · Porting/DIDs/ops 10 · Min-spend/self-serve 10 · Vendor AI foil 5.

| Vendor | Cost 25 | API 20 | Spam 15 | Media/SIP 15 | Port/DID 10 | Self-serve 10 | AI foil 5 | **Total** |
|--------|--------:|-------:|--------:|-------------:|------------:|-------------:|----------:|----------:|
| **Telnyx** | 24 | 19 | **15** | 14 | 8 | 10 | 4 | **94** |
| SignalWire | 22 | 18 | 6 | 14 | 9 | 10 | **5** | **84** |
| Plivo | 22 | 16 | 5 | **15** | 8 | 10 | 4 | **80** |
| Twilio | 10 | **20** | 5 | 13 | 9 | 10 | 4 | **71** |
| Bandwidth | 21 | 14 | 6 | 12 | 6 | 8 | 2 | **69** |
| Vonage | 17 | 16 | 4 | 12 | 7 | 9 | 3 | **68** |
| Wavix | 11 | 10 | 3 | 11 | 6 | 9 | 1 | **51** |

### Scoring notes (anti-bias)

- **Cost:** Telnyx Call Control slightly ahead of SW/Plivo on mid/high; Twilio clearly last among majors. Plivo gains on agent-path because stream is $0 — reflected in Media/SIP, not double-counted into Cost.
- **API:** Twilio still richest ecosystem; Telnyx Call Control + TeXML + public pricing.md is excellent; SignalWire SWML/Compatibility strong.
- **Spam:** Only Telnyx gets full marks for **free, network-level, multi-DB reputation + STIR screen with Flag/Reject and webhook field** — verified Mar 2026 free forever (no per-number fee).
- **Skill bias:** Existing Telnyx telephony skills and prior GIU Telnyx recommendation were **not** used as scoring inputs. If spam weight were removed, **Plivo/SignalWire** would challenge or beat Telnyx on agent economics — called out explicitly so Frank can reweight.

---

## claims

### claim_id: C1
- **statement:** Telnyx Voice API list is $0.002/min platform fee; US SIP trunk list starts ~$0.0032/min inbound and ~$0.005/min outbound; combined programmable local ≈ **$0.0052 in / $0.007 out**; local DID marketed at **$1.00/number/mo**.
- **evidence_refs:** [E1][E2][E3]
- **confidence:** high
- **status:** supported

### claim_id: C2
- **statement:** Telnyx Inbound Call Screening is **free** as of **2026-03-17**, uses CallerAPI + Nomorobo + YouMail reputation plus number validation and STIR/SHAKEN attestation screening; modes Flag (SIP header + webhook `call_screening_result`) or Reject at network.
- **evidence_refs:** [E4][E5]
- **confidence:** high
- **status:** supported

### claim_id: C3
- **statement:** Twilio US Programmable Voice list (2026 page pull): outbound local **$0.0140/min**, inbound local **$0.0085/min**, local number **$1.15/mo**; Media Streams about **$0.0040–0.0044/min**; AMD $0.0075/call; recording $0.0025/min.
- **evidence_refs:** [E6]
- **confidence:** high
- **status:** supported

### claim_id: C4
- **statement:** SignalWire published US PSTN rates: inbound local **$0.0066/min**, outbound **$0.0080/min**, SIP/WebRTC **$0.003/min**, local DID **$0.50/mo**, TF DID **$0.80/mo**; Audio Stream/Tap **$0.003/min**; port-in free, port-out $5.
- **evidence_refs:** [E7]
- **confidence:** high
- **status:** supported

### claim_id: C5
- **statement:** Plivo US voice list: outbound local **$0.0115/min**, inbound **$0.0055/min**, local DID **$0.50/mo**; Call Recording / AMD / Conference listed **$0**; **Audio streaming & noise cancellation Included**.
- **evidence_refs:** [E8]
- **confidence:** high
- **status:** supported

### claim_id: C6
- **statement:** Bandwidth published US local voice: outbound **$0.0100/min**, inbound **$0.0055/min**; unidirectional media stream $0.0035/min, bidirectional $0.004/min; self-serve Build path exists (trial credits, no sales call claimed).
- **evidence_refs:** [E9][E10]
- **confidence:** medium
- **status:** supported

### claim_id: C7
- **statement:** Vonage US list (Cozzo verified vendor page 2026-10-31): inbound **$0.00495/min**, outbound **$0.01446/min** on international PSTN card; domestic-sales card ~**$0.00485 in / $0.00849 out**; local number **$1.06/mo**; SIP/WebSocket leg ~**$0.00492/min**.
- **evidence_refs:** [E11]
- **confidence:** medium
- **status:** supported

### claim_id: C8
- **statement:** SignalWire does **not** ship a Telnyx-equivalent free multi-provider network spam reputation toggle; public guidance is DIY blocklists / Robokiller integration.
- **evidence_refs:** [E12]
- **confidence:** medium
- **status:** supported

### claim_id: C9
- **statement:** Google Voice numbers can be ported to CPaaS carriers after unlock (Twilio documents unlock + screenshot LOA path; personal unlock historically ~$3); Telnyx accepts standard port-ins via portal/API. OTP apps may break if still bound to GV forwarding — operational risk, not carrier-unique.
- **evidence_refs:** [E13][E14][E15]
- **confidence:** medium
- **status:** partial

### claim_id: C10
- **statement:** Under the task volume model, Telnyx Call Control base mid ≈ **$8.30/mo** vs Twilio ≈ **$13.85** vs SignalWire ≈ **$9.50** vs Plivo ≈ **$9.45** (ex-tax). Agent mid with streams: Plivo ~$9.45, Telnyx ~$12.85, Twilio ~$19.57.
- **evidence_refs:** [E1][E6][E7][E8] + derived model
- **confidence:** high
- **status:** supported

### claim_id: C11
- **statement:** Wavix US local landline voice list shows outbound **$0.0065/min** and inbound **$0.01/min** — inbound-expensive relative to Telnyx/SW/Plivo for PJ’s inbound-heavy profile; better as SIP/DID adjunct than primary programmable carrier.
- **evidence_refs:** [E16]
- **confidence:** medium
- **status:** supported

### claim_id: C12
- **statement:** Primary recommendation remains **Telnyx** after unbiased re-score; runner-up **SignalWire**; agent-economics challenger **Plivo**. Prior `t_c4f16b85` Telnyx lean is **corroborated**, not skill-locked.
- **evidence_refs:** scorecard + C1–C11
- **confidence:** high
- **status:** supported

---

## evidence_index

| evidence_id | source_label | source_url | provenance | support note |
|-------------|--------------|------------|------------|--------------|
| E1 | Telnyx Voice API Pricing | https://telnyx.com/pricing/voice-api | browser_navigate snapshot 2026-11-02; curl HTML | $0.002/min Voice API; example SIP out $0.005 in $0.0032; transfer $0.10; media stream $0.0035; compare table Twilio/Vonage |
| E2 | Telnyx pricing.md | https://telnyx.com/pricing.md | curl markdown 2026-11-02 | SIP from $0.005 out / $0.0032 in; Voice API $0.002; media streaming $0.0035 |
| E3 | Telnyx Global Numbers Pricing | https://telnyx.com/pricing/numbers | curl HTML strip | $1.00 per number/mo headline |
| E4 | Telnyx HC Inbound Call Screening | https://support.telnyx.com/en/articles/8037040-inbound-call-screening | web_search snippet + curl | Free; Flag/Reject; SPAM LIKELY headers; webhook field; US/CA reputation |
| E5 | Telnyx release: screening free | https://telnyx.com/release-notes/inbound-call-screening-free | web_search + curl | 2026-03-17 free; CallerAPI+Nomorobo+YouMail |
| E6 | Twilio Voice pricing US | https://www.twilio.com/en-us/voice/pricing/us | curl HTML strip 2026-11-02 | $0.014 out / $0.0085 in; DID $1.15; streams ~$0.0040–0.0044; AMD $0.0075 |
| E7 | SignalWire pricing rates | https://signalwire.com/pricing and /pricing/rates | curl HTML strip | $0.0066/$0.0080; DID $0.50/$0.80; stream $0.003; AI Agent $0.16 |
| E8 | Plivo US Voice pricing | https://www.plivo.com/voice/pricing/us/ | curl HTML strip | $0.0115/$0.0055; DID $0.50; recording/AMD $0; streaming Included |
| E9 | Bandwidth pricing | https://www.bandwidth.com/pricing/ | curl HTML strip | $0.0100 out / $0.0055 in; media stream rates |
| E10 | Bandwidth Build signup | https://www.bandwidth.com/build-sign-up/ | web_search description | Self-serve Voice/SIP/SMS trial, no CC/sales call claimed |
| E11 | Cozzo Vonage price check | https://cozzo.io/tools/vonage | curl text | Verified 2026-10-31 vs Vonage page; rates as C7 |
| E12 | SignalWire spam guidance | https://signalwire.com/blog/how-to-block-spam-calls-with-robokiller-and-signalwire ; DIY Node blog | web_search | Partner/DIY spam — not free built-in reputation toggle |
| E13 | Twilio GV port help | https://help.twilio.com/articles/223179728-Can-I-port-my-Google-Voice-number | web_search | Unlock + screenshot path |
| E14 | Dialsheet GV→Twilio guide | https://www.dialsheet.io/resources/port-your-number-to-twilio-from-openphone-or-google-voice | web_search | Unlock $3 personal / free Workspace; LOA match |
| E15 | Telnyx port-in HC | https://support.telnyx.com/en/articles/1130634-port-numbers-to-telnyx | curl text | Portal + API port-in |
| E16 | Wavix voice pricing | https://wavix.com/pricing/voice | curl HTML strip | Local landline $0.0065 out / $0.01 in |
| E17 | Prior GIU report | `/home/ice/know/research/business-phone-spam-voice-agent-gv-pj-t_c4f16b85.md` | local read | Prior Telnyx architecture; screening was paid then — **superseded** by E5 free |

---

## Primary / runner-up / when-to-choose

### Primary — Telnyx
**Why:** Lowest (or joint-lowest) programmable all-in among majors **and** unique free network spam/reputation screen for day-1. Strong SIP + Call Control + TeXML + media streaming. $0 platform fee, $0 min. AZ inventory via standard US local DID search (confirm at provision time).

**Day-1 recipe:**  
1. Create Mission Control account, fund PAYG.  
2. Buy AZ local DID (~$1/mo).  
3. Enable Inbound Call Screening → **Reject** (hard block) or **Flag** (route spam to dead-end TeXML).  
4. Attach TeXML app: greet → DTMF menu → dial human / voicemail.  
5. Optional: simultaneous ring to cell; keep GV as backup until port FOC.

**Later agent:** WebSocket media streaming to self-host STT/LLM/TTS **or** SIP to FreeSWITCH/LiveKit; keep screening on number.

### Runner-up — SignalWire
**Why:** Cost within ~$1 of Telnyx at mid volume, DID half price, FreeSWITCH heritage, SWML + Compatibility API, strong AI Agent foil ($0.16/min all-in transport+AI marketing).  
**Gap:** No free Telnyx-class inbound reputation product — budget DIY filter or Robokiller.  
**Choose SW if:** You want FS-native control plane first and will own spam logic.

### Challenger — Plivo
**Why:** Best **agent-path list economics** if streaming truly remains Included; free recording/AMD; cheap DID.  
**Gap:** Spam shield DIY; outbound list higher than Telnyx/SW.  
**Choose Plivo if:** Agent media cost dominates and spam is handled upstream (e.g. only publish number to known clients).

### Twilio — choose when
Docs/SDK depth, largest example ecosystem, Flex/contact-center gravity, or existing Twilio estate. Accept ~1.5–2× voice bill and build/buy spam separately. Voice Integrity helps **outbound** spam-label remediation, not free inbound block.

### Bandwidth — choose when
You want facilities-based US carrier, enterprise insights/911, and Build self-serve is enough. Confirm DID MRC and any account gates before committing. Spam/reputation is productized but not the free day-1 toggle story.

### Vonage — choose when
Per-second billing + WebSocket AI streaming matter and you can unlock domestic rates (else outbound list hurts). Inbound-only heavy workloads look better on list card.

### Wavix / DIDWW / VoIP.ms — choose when
Hybrid: cheap international DIDs or SIP termination into your own switch. **Not** recommended as sole PJ programmable + spam platform. VoIP.ms is PBX-cheap but weak modern CPaaS agent APIs relative to majors.

---

## Skill-bias check (mandatory)

| Factor | Effect on this bakeoff |
|--------|------------------------|
| Prior `t_c4f16b85` Telnyx recommendation | Used as background only; rates/features re-pulled Nov 2026 |
| Existing Telnyx/Twilio Hermes skills | **Not** scored. Twilio would win if “skill familiarity” were a weight — it is not |
| What would change the winner? | (1) Drop spam weight → Plivo/SW contest; (2) Need max docs/hireability → Twilio; (3) Domestic Vonage rates confirmed self-serve → Vonage rises on cost |
| Honest tension | Plivo’s **Included streaming** is under-advertised in prior GIU work and is a real Telnyx competitor on agent $ |

---

## Porting / GV / AZ notes

- **New AZ DID (recommended day-1):** Instant provision on Telnyx/Twilio/SW/Plivo; verify 480/520/602/623 inventory in portal before account beauty contest ends.
- **Port GV → CPaaS:** Unlock in Google Voice → LOA/screenshot → carrier port (days, not minutes). Personal unlock fee often cited ~$3; Workspace may differ.
- **Forwarding interim:** GV → new DID possible but adds latency and GV spam policy quirks; prefer cutover after screening tested.
- **OTP risk:** Services bound to GV SMS may need number update; voice-only OTP usually fine after port if CNAM/STIR clean.
- **Min spend:** All majors above offer true PAYG at PJ volumes (low tens of $/mo). No enterprise minimum required for Telnyx/Twilio/SW/Plivo.

---

## gaps

1. **Live AZ DID search** not executed inside vendor portals (needs authenticated session) — inventory assumed available; verify before buy.
2. **Bandwidth exact DID MRC** not on public voice blurb — modeled $1.
3. **Wavix US DID MRC** not clearly extracted from numbers page (JS-heavy) — modeled $1.
4. **Vonage domestic rates** may require sales enablement — list card used for default score.
5. **Plivo “Included” streaming** — confirm in account ToS/metering that bidirectional WS is truly $0 at scale (price card says Included as of this pull).
6. **Regulatory/USF/carrier fees** excluded — real invoices higher; relative ranking usually stable.
7. **Firecrawl outage** forced curl/browser extraction — pages can A/B; re-check rate cards at purchase week.
8. **t_5f19c10d** related id not found on board — no extra handoff.

---

## confidence

- **level:** high  
- **rationale:** Core rate cards pulled from official pages the same day (Telnyx, Twilio, SignalWire, Plivo, Bandwidth) plus Telnyx free-screening primary docs. Soft spots isolated to DID quotes (Bandwidth/Wavix), Vonage domestic gate, and unauthenticated AZ inventory.

---

## recommended_next_actions

1. **Provision Telnyx** PAYG + search AZ local DIDs; enable Inbound Call Screening (start Flag 48h, then Reject if FP low).  
2. **TeXML day-1 IVR** (open / hours / ring human) — no AI required.  
3. **Parallel spike (2–4h):** Plivo streaming hello-world vs Telnyx media stream on same agent stub — validate Included $0 and latency.  
4. **Optional:** SignalWire Compatibility app spike if FreeSWITCH-first path preferred.  
5. **Do not** buy Twilio as primary unless ecosystem reasons dominate; keep as TeXML-compatible backup mental model.  
6. **GV:** keep number alive until new DID stable; port only after screening+IVR proven.  
7. **Ops:** document CNAM, STIR attestation level on outbound, and emergency address if E911 needed ($0.75/mo class fees on several vendors).

---

## Appendix A — All-in rate derivation (Telnyx)

```
Programmable inbound local = Voice API 0.002 + SIP inbound 0.0032 = 0.0052 $/min
Programmable outbound local = Voice API 0.002 + SIP outbound 0.005  = 0.0070 $/min
(SIP-only drops the 0.002 platform fee; loses hosted Call Control/TeXML unless self-built)
```

Matches Telnyx marketing example (100k min blended) and third-party 2026 comparisons citing ~$0.0052/$0.0070.

## Appendix B — Extraction provenance stamp

| Source class | Path used |
|--------------|-----------|
| Telnyx voice/numbers/SIP | browser_navigate + curl + pricing.md |
| Twilio / SignalWire / Plivo / Bandwidth / Wavix | curl HTML strip (Firecrawl 403) |
| Screening docs | web_search + curl support/release notes |
| Vonage | Cozzo secondary verifier of vendor page (Oct 31, 2026) |
| Volume math | local Python 2026-11-02 |

## Appendix C — Dark horse reject table

| Candidate | Verdict | Why |
|-----------|---------|-----|
| Wavix | Reject primary | Inbound $0.01 local; CPaaS-agent depth thinner |
| DIDWW | Reject primary | DID/SIP global specialist; spam screen not day-1 story |
| VoIP.ms | Reject primary | Cheap PBX trunking; weak modern agent WS story vs CPaaS |
| FreeSWITCH bare | Infra not carrier | Needs a DID/PSTN carrier underneath (Telnyx/SW/BW) |
| Google Voice stay | Reject | No real Call Control API; spam/policy limits; not business CPaaS |

---

*End of report — GIU t_922833ff*
