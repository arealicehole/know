# Agent-visibility research process (Bible on-point → replicate → maximize)

**Owner frame (Frank 2026-10-30):** This is **not** the AVP sales process.  
It is the loop that keeps the **SEO Bible** honest and lets us **replicate + maximize** what PJ/Maza already get from answer engines:

```
probe fleet → research (Geraldo) → measure/test → findings →
  Frank decides: Bible 3.0.x patch | more research | site experiment →
  Romulus ships doctrine/site only after go → re-measure → new open questions
```

**Locked doctrine:** `/workspace/seo-bible/SEO_BIBLE_3.0.md`  
**Research SoR:** `arealicehole/know` → `research/`  
**Assembly:** Romulus only after Frank greenlight · **no silent Bible rewrites from vibes**

---

## Rules of the loop

1. **Outcome research first** (principles, levers, net-new, still-unknown) — not merge-gate theater.  
2. **Tests produce data**, not placement claims. Label **API baseline** vs **consumer-app UI**.  
3. Every completed battery/ablation **opens** a still-unknown list → next card.  
4. Bible patches are **additive 3.0.1+ appendices** unless Frank orders a structural rewrite.  
5. Site experiments (ablation, content, GBP) are **separate web cards**; research designs them, web may implement on staging.  
6. **Do not claim** single-lever causation until ablation (or labeled correlational triangulation) says so.

---

## What we already closed (seed knowledge)

| Pack | What it unlocked |
|------|------------------|
| `ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1` | Identity vs transaction surfaces; competitor scorecard |
| `chat-ui-discovery-bible-section-refresh-20261029` | Doctrine refresh inputs; **still-unknown** list |
| `avp-causal-verification-multi-engine-citation-battery-t_70d29518` | Battery + ablation **design** |
| `multi-engine-citation-battery-first-execution-t_49a81e49` + open-cells delta | **First API baseline** — Maza 3/3; PJ P1 1/4 Claude |
| `pj-gmc-ucp-eligibility-20261029` | GMC/Google UCP limits for service/custom |
| Bible **3.0 lock** | Doctrine + measurement protocol + baseline footnote |

---

## Open research backlog (prioritized)

### P0 — Causal / maximize (blocks “which lever to double down on”)

| ID | Question | Why it matters | Method | Status |
|----|----------|----------------|--------|--------|
| **R1** | Which **single** surface moves citations most? | Maximize spend of eng time | Staged ablation V0–V6 per t_70d29518; need staging host | **OPEN** |
| **R2** | API battery ≠ consumer ChatGPT/Claude/Gemini **app** UI? | Replicate real human path | Same prompts in real apps (Frank or ops screenshots) vs API | **OPEN** |
| **R3** | Full **12×5** (or Tier-1 P1–P7 × 5 engines) matrix | Parity gaps, not one-prompt luck | Extend battery; fill empty cells | **OPEN** (partial only) |

### P1 — Replicate pattern / close gaps on proof fleet

| ID | Question | Why it matters | Method | Status |
|----|----------|----------------|--------|--------|
| **R4** | Why PJ **loses narrative** to AlphaGraphics/Mousegraphics on P1 while still “readable”? | Maximize PJ chat wins | Competitor content/GBP/directory/citation autopsy + intent-page gap | **OPEN** |
| **R5** | Post-dual-stack **re-score** PJ+Maza vs same competitors | Baseline moved (identity+UCP both live) | Re-run 10-pt scorecard + spot battery | **OPEN** |
| **R6** | Directory/GBP as **answer sources** — depth required | Bible §3/directories | Probe Yelp/GBP fields engines quote | **OPEN** (doctrine weak on depth) |

### P2 — Measurement hygiene / ops for continuous loop

| ID | Question | Why it matters | Method | Status |
|----|----------|----------------|--------|--------|
| **R7** | Retrospective triangulation (referrals + bot logs + ship dates) | When ablation host missing | GA4 + edge logs + timeline | **OPEN** |
| **R8** | Grok/xAI crawler identity + robots honor | robots policy honesty | Log UA + controlled disallow test (non-prod) | **OPEN** |
| **R9** | ChatGPT **product** search path vs generic chat | PJ commerce prompts | Separate prompt tier | **OPEN** |

### P3 — Expand beyond two brands (replicate)

| ID | Question | Why it matters | Method | Status |
|----|----------|----------------|--------|--------|
| **R10** | Extant-site Path 2 (edge inject) works in the wild? | Client replication | Design + one pilot when Frank picks a thin site | **DEFERRED** |
| **R11** | Media §8 specimen still correct post-3.0? | Other ruler classes | R&R / JWAZ spot battery | **DEFERRED** |

---

## Wave plan (execute in order unless Frank reprioritizes)

### Wave 1 — *now* (measurement depth)
1. **Full Tier-1 battery expansion** (P1–P7 × engines) + empty-cell fill — API labeled.  
   - **DONE 2026-10-30** (`t_d08327bc`): 35/35 cells, ~$1.40. Report: `citation-battery-tier1-p1-p7-expansion-t_d08327bc.md`  
   - Snapshot: Maza P3 **5/5**; PJ P1 **3/5** (Gemini flipped HIT); P2/P7 **0/5**; P5 only Perplexity names $75; Edge 24/24=200; Maza **11/15** vs PJ **4/20**  
   - Prior “PJ 1/4 on P1” obsolete for this snapshot  
2. **PJ miss autopsy** (R4) using battery miss transcripts + competitor probes.  
   - **DONE 2026-10-30** (`t_f1118fbb`): `pj-chat-discovery-miss-autopsy-20261030.md`  
   - Headline: PJ wins **offer/machine** surface; chains win **place/authority** (street NAP, franchise URL farm, Yelp/dir density, ratings/tenure). Not robots/UCP.  
   - Live PJ holes: no `streetAddress` (llms.txt admits it), no `aggregateRating`, thin sameAs vs AG/Mouse. $75 ambient (Eyeconic). P2/P7 = specialty SERP.  
   - Status: R4 **diagnosed** — maximize = identity pin + citation graph, not more dual-stack alone.  
3. **Dual-stack rescore** (R5) — cheap, updates the replication blueprint. (`t_4237e704` ready; seed autopsy)

### Wave 2 — *causal*
4. **Host decision (2026-10-30):** PJ = public Vercel feature-branch previews; Maza = SSO-walled previews → clone project or log-only. Doc: `ablation-host-decision-20261030.md`.  
5. Cut PJ **V0–V3** preview branches (**never merge to main**); bot-UA 200 proof. Card `t_4c2afe0d`.  
6. Geraldo ablation battery on those URLs only — label **preview/domain-shifted**, not prod causation; open-discovery **and** URL-forced legs. Card `t_6e47f5ab` (blocked until URLs).  
7. Independent review (`deleg_22c96ed7`): host OK; Wave 2 not “running” until variants live; `maza-mediterranean.vercel.app` is **prod alias**.  
8. Maza causal waits clone **or** R7 retrospective. Optional separate defect: `www.mazahalalfood.com` TLS hostname mismatch.

### Wave 3 — *human path + maximize*
7. Consumer-app pack (R2) — Frank paste or guided screenshot protocol.  
8. Directory/GBP depth (R6).  
9. Bible **3.0.1** appendix only after Frank reads Wave 1–2 packs.

---

## Card discipline

- Board: **research** · assignee **geraldov21**  
- Title prefix: `[GIU][Bible-loop]`  
- Body: goal → method → still-unknown out → **no Bible write** · no AVP $  
- Parent-link any future **Bible 3.0.1 assemble** card to completed research ids  
- After each report: Romulus TLDR to Frank → **wait for go** before doctrine/site change

---

## Done definition for “Bible on point”

Not “no open questions.”  
**On point** = every doctrine claim is either (a) verified, (b) labeled likely, or (c) explicitly still-unknown with a next test.  
**Maximize** = ablation/ranking tells us what to double on the live fleet.  
**Replicate** = scorecard + install checklist still match what wins on the battery.

---

*Romulus — process spine 2026-10-30. Living doc; append wave results, do not erase open rows until closed with a report path.*
