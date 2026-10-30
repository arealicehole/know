# Citation Battery — Tier-1 Expansion (P1–P7 × 5 Engines)

**Task:** t_d08327bc (board: research, prefix [GIU])  
**Parent / SoR:** t_70d29518 — multi-engine citation battery protocol §A  
**Prior baselines:**  
- t_49a81e49 — first execution (API proxy; partial matrix)  
- t_528f7761 — open-cells delta (Gemini×Maza/PJ, Grok×PJ)  
**Run window (UTC):** 2026-10-30T07:06–07:21Z  
**Prepared by:** Geraldo v2.2  
**Method:** OpenRouter chat completions + native provider web search (API-proxy — not consumer-app UI)

---

## objective

Execute the multi-engine citation battery for **Tier-1 prompts P1–P7 × all available engines** (OpenAI, Claude, Gemini, Perplexity, Grok) against live Print Junkie AZ and Maza Mediterranean, producing a complete 35-cell matrix with provenance, fact-accuracy vs ground truth, competitor/trap notes, edge reachability, and explicit deltas vs prior baselines — labeled honestly as **API-proxy**, not consumer ChatGPT/Gemini app results.

## summary

**35/35 cells completed live** on 2026-10-30 via OpenRouter native web search (total cost ≈ **$1.40**). Edge access remains fully open: **24/24 = HTTP 200** across OAI-SearchBot, ChatGPT-User, PerplexityBot, Claude-SearchBot, xAI-SearchBot, Googlebot × PJ/Maza home + primary intent pages.

**Headline scores (this run, single-shot):**

| Lane | Score | Readout |
|------|-------|---------|
| **Maza discovery (P3)** | **5/5** | Every engine names + (usually) links Maza; OpenAI/Claude/Grok quote exact NAP |
| **Maza delivery (P4)** | **3/5** | Claude, Gemini, Grok cite; OpenAI + Perplexity prefer delivery-native competitors |
| **Maza hours (P6)** | **3/5** | OpenAI (exact NAP+hours), Gemini (named, **hours partially wrong**), Grok (named thin); Claude + Perplexity pick Green Corner / others on unscoped “best Mediterranean” |
| **PJ business cards (P1)** | **3/5** | OpenAI, Claude, **Gemini** cite (Gemini is a **delta HIT** vs prior MISS on t_528f7761); Perplexity + Grok miss |
| **PJ price 1k (P5)** | **1/5** | **Only Perplexity** names Print Junkie at **$75/1,000** (correct); OpenAI/Gemini quote market $75 ranges **without naming PJ** |
| **PJ window wraps (P2)** | **0/5** | Specialty wrap shops dominate; PJ absent on all engines |
| **PJ signs/banners near Chandler (P7)** | **0/5** | FASTSIGNS / Signarama / franchise sign lane wins; PJ absent |

**Biz totals this run:** Maza **11/15** cited cells; PJ **4/20** cited cells.  
**Strongest causal-adjacent signal (still correlation):** engines that cite Maza repeatedly reproduce **3491 W Frye Rd Ste 2**, **(480) 534-6550**, **Tue–Sun 10–10 / closed Mon** from first-party surface — ground-truth match, not hallucination.  
**Do not claim:** stack *caused* citations; consumer-app parity; P2/P7 “failure of robots” (access is open — this is ranking/intent coverage); Gemini P6 hours as accurate.

## plan (executed)

```yaml
objective: Fill Tier-1 P1–P7 × 5 engines; matrix + deltas + know archive
success_criteria:
  - 35 API cells with HTTP 200 + web grounding where supported
  - each cell scored cited Y/N, type, facts, competitors, cost
  - edge probe matrix for bot UAs
  - durable report at /home/ice/know/research/citation-battery-tier1-p1-p7-expansion-t_d08327bc.md
assumptions:
  - API-proxy path identical in spirit to t_49a81e49 / t_528f7761
  - single-shot per cell (stability 2–3× still owed)
  - consumer-app legs out of scope for this headless run
tasks:
  - T1: load SoR + prior baselines; verify OpenRouter credits/models
  - T2: edge UA probes PJ/Maza home + intent
  - T3: run 35 cells; save JSON/txt captures
  - T4: score + delta vs prior; Final Report Contract publish
```

---

## method

| Engine | Model (requested = returned) | Search path |
|--------|------------------------------|-------------|
| OpenAI | `openai/gpt-5.6-luna` | `plugins:[{id:"web",engine:"native"}]` |
| Claude | `anthropic/claude-haiku-5.5` | native web plugin |
| Gemini | `google/gemini-3.5-flash` | native web plugin (Vertex grounding titles) |
| Perplexity | `perplexity/sonar-pro` | built-in search |
| Grok | `x-ai/grok-4.3` | native web plugin |

- **API:** `POST https://openrouter.ai/api/v1/chat/completions`  
- **Auth:** `OPENROUTER_API_KEY` (non-free; pre-run usage ≈ $22; limit_remaining null)  
- **Params:** `temperature=0.2`, one user message = prompt verbatim, fresh session per cell  
- **Workspace captures (ephemeral):** `/home/ice/.hermes/kanban/boards/research/workspaces/t_d08327bc/captures/{engine}_{pN}.{json,txt}`  
- **Durable archive:** this file only (single-publish rule)

### Prompts (verbatim, protocol §A.3)

| ID | Biz | Prompt |
|----|-----|--------|
| P1 | PJ | What's a good place to get business cards printed in Phoenix or Tempe, Arizona? |
| P2 | PJ | Where can I get custom window wraps or vehicle graphics in the Phoenix area? |
| P3 | Maza | What's a good halal Mediterranean restaurant in Chandler, Arizona? |
| P4 | Maza | Where can I get gyro or shish kabab delivered in Chandler, Arizona? |
| P5 | PJ | How much does it cost to get 1,000 business cards printed in Phoenix? |
| P6 | Maza | What are the hours for the best Mediterranean restaurant in Chandler? |
| P7 | PJ | Is there a good sign or banner shop near Chandler, Arizona? |

### Ground truth (reused from t_70d29518 / live schema)

| Business | Phone | Key price | Hours | Address |
|----------|-------|-----------|-------|---------|
| Print Junkie AZ | +1-602-730-2135 | ~$75 / 1,000; from ~$65 / 100 | Mon–Sat 09:00–17:00 | Tempe service area (GeoCircle Phoenix in schema) |
| Maza Mediterranean | (480) 534-6550 | menu in JSON-LD | Tue–Sun 10:00–22:00, **closed Mon** | **3491 W Frye Rd Ste 2, Chandler, AZ 85226** |

---

## findings

### 1) Full matrix — our_business_cited (this run)

Legend: **Y** = named and/or first-party domain in answer/citations; **N** = absent. Citation type in parentheses.

| Prompt | OpenAI | Claude | Gemini | Perplexity | Grok | Hits |
|--------|--------|--------|--------|------------|------|------|
| **P1** PJ cards | **Y** (named+linked+price) | **Y** (named+linked) | **Y** (named+linked+$75) | N | N | **3/5** |
| **P2** PJ wraps | N | N | N | N | N | **0/5** |
| **P3** Maza halal | **Y** (NAP+rating) | **Y** (NAP named) | **Y** (named+addr+#1) | **Y** (named+linked) | **Y** (NAP+Yelp) | **5/5** |
| **P4** Maza delivery | N | **Y** (named+intent URL) | **Y** (named+addr+site order) | N | **Y** (named thin) | **3/5** |
| **P5** PJ $1k | N* | N | N* | **Y** (named+$75) | N* | **1/5** |
| **P6** Maza hours | **Y** (exact NAP+hours) | N (Green Corner) | **Y** (named; hours **wrong**) | N | **Y** (named thin) | **3/5** |
| **P7** PJ signs | N | N | N | N | N | **0/5** |
| **Engine total** | 3/7 | 3/7 | 4/7 | 2/7 | 3/7 | **15/35** |

\*N* = engine quoted a market price band that **includes ~$75** but **did not name Print Junkie** (price-without-attribution).

### 2) Edge reachability (P2 gate) — this run

| UA | PJ home | PJ /business-cards-phoenix | Maza home | Maza /halal-mediterranean-chandler |
|----|---------|----------------------------|-----------|-------------------------------------|
| OAI-SearchBot | 200 | 200 | 200 | 200 |
| ChatGPT-User | 200 | 200 | 200 | 200 |
| PerplexityBot | 200 | 200 | 200 | 200 |
| Claude-SearchBot | 200 | 200 | 200 | 200 |
| xAI-SearchBot | 200 | 200 | 200 | 200 |
| Googlebot | 200 | 200 | 200 | 200 |

**24/24 = 200.** Access is not the blocker for P2/P7 zeros or PJ misses.

### 3) Per-prompt narratives (material cells)

#### P1 — Business cards Phoenix/Tempe (PJ)

| Engine | Result | Notes |
|--------|--------|-------|
| **OpenAI** | **HIT** | Section **“Best for fast turnaround: Print Junkie AZ”**; quotes **$65/100** and **$75/1,000**; link `printjunkieaz.com/business-cards-phoenix?utm_source=openai`. AlphaGraphics still “best overall.” |
| **Claude** | **HIT** | Named **Print Junkie AZ (Tempe)** with 2–3 day ship / next-day rush / same-day pickup; linked intent page. Eyeconic + AlphaGraphics co-listed. |
| **Gemini** | **HIT (delta vs t_528f7761 MISS)** | Named **Print Junkie AZ** with premium stocks + **~$75 for 1,000**; first-party domain in grounding. Still multi-shop list (AlphaGraphics, Mousegraphics, etc.). |
| **Perplexity** | MISS | AlphaGraphics, CesarGraphics, UPS Store — no PJ. |
| **Grok** | MISS | AlphaGraphics-led + Outta PHX / Splash / JC — no PJ. |

#### P2 — Window wraps / vehicle graphics (PJ)

**0/5.** Answers dominated by specialty wrap shops (PRI Graphics, Custom Graphix, Limitless Graphix, Fast-Trac Designs, PHX Screen, etc.). **No Print Junkie** on any engine. Interpretation: PJ’s wrap/graphics intent surface is not winning retrieval against pure-play wrap SERP entities — not an edge-block issue.

#### P3 — Halal Mediterranean Chandler (Maza)

**5/5 HIT — strongest row in the battery.**

| Engine | Rank / role | Facts quoted (vs GT) |
|--------|-------------|----------------------|
| OpenAI | Strong pick | address ✓ phone ✓ hours ✓ **4.9** ✓; link home |
| Claude | **“best-supported pick”** | address ✓ phone ✓ hours ✓; plates $16.55–$28.99 |
| Gemini | **#1** “Best for Traditional Middle Eastern & Kebabs” | address ✓; Greeks Grill / Arabella / BelAfia also listed |
| Perplexity | Named + linked | address path; Green Corner co-cited |
| Grok | Lead recommendation | address ✓ phone ✓ hours ✓; Yelp co-cited |

#### P4 — Gyro/shish delivery Chandler (Maza)

| Engine | Result | Notes |
|--------|--------|-------|
| OpenAI | MISS | Green Corner, BelAfia, delivery apps — no Maza |
| Claude | **HIT** | Maza with Frye Rd address; notes delivery unconfirmed; also cites **intent URL** `/halal-mediterranean-chandler` in capture path |
| Gemini | **HIT** | Maza #3-ish with address + **order on mazahalalfood.com** + UE/DD |
| Perplexity | MISS | King of Gyros, Greeks Grill delivery-first |
| Grok | **HIT (thin)** | Maza named in “others” kebab list without deep facts |

**Readout:** delivery-intent favors entities with strong marketplace/delivery-graph presence; Maza wins when engine still retrieves first-party restaurant identity.

#### P5 — Cost 1,000 business cards Phoenix (PJ)

| Engine | Names PJ? | Price behavior |
|--------|-----------|----------------|
| OpenAI | N | Band ~$50–$80; competitors named |
| Claude | N | No PJ |
| Gemini | N | Band $55–$100; **$75 appears as market number without PJ** |
| **Perplexity** | **Y** | **“Print Junkie: $75 for 1,000 cards”** + link intent page — **only clean price attribution HIT** |
| Grok | N | Market/competitor pricing |

**Inversion vs P1:** Perplexity **misses** anonymous discovery (P1) but **hits** explicit price query (P5). OpenAI/Gemini can emit **$75** as ambient market knowledge without credit — measurement implication: price-band answers ≠ merchant citation.

#### P6 — Hours for “best” Mediterranean Chandler (Maza)

Unscoped “best” (no *halal*) changes the game:

| Engine | Result | Notes |
|--------|--------|-------|
| **OpenAI** | **HIT + exact** | Picks **Maza** as top-rated; hours Tue–Sun 10–10, closed Mon; address; phone; `?utm_source=openai` |
| Claude | MISS | **Green Corner** hours Mon–Sat 11–9 / Sun 11–7; Pitta Souvli alt — **no Maza** |
| Gemini | **HIT name, BAD hours** | Lists Maza #4; quotes **Tue–Wed 10–8, Thu–Sun 10–10, Mon closed** — **does not match GT Tue–Sun 10–10**. Fact accuracy = **incorrect hours** |
| Perplexity | MISS | Non-Maza hours path |
| Grok | HIT thin | Mentions Maza as alt without full hours block |

**Trap insight (P6 ≈ soft P11):** removing “halal” drops Claude/Perplexity off Maza; OpenAI still locks entity+hours. Gemini retrieves entity but **corrupts hours** — do not treat Gemini hours as schema-faithful without verification.

#### P7 — Sign/banner shop near Chandler (PJ)

**0/5.** FASTSIGNS Chandler, Signarama Chandler, Print ’N Ship, AlphaGraphics Chandler, Sir Speedy, Display Solutions Group dominate. PJ never appears. Signage franchise density beats generalist print discovery on this prompt.

### 4) Deltas vs prior baselines

| Cell / theme | Prior (t_49a81e49 / t_528f7761) | This run (t_d08327bc) | Delta |
|--------------|----------------------------------|------------------------|-------|
| Gemini × P1 PJ | **MISS** (AlphaGraphics-led) | **HIT** named + ~$75 + domain | **Improved** (single-shot variance or index move — do not over-claim permanence) |
| OpenAI × P1 PJ | Linked w/ UTM; weak narrative | **Named section + $65/$75 + UTM** | **Improved narrative** |
| Claude × P1 PJ | HIT name+price | HIT name+turnaround | **Stable HIT** |
| Perplexity × P1 | MISS | MISS | Stable miss |
| Grok × P1 | MISS | MISS | Stable miss |
| Maza P3 multi-engine | 3/3 or 4/4 on partial sets | **5/5** | **Confirmed / expanded** |
| Gemini × P3 | HIT #2 + address | HIT **#1** + address | Stable/stronger |
| Grok × P3 | HIT NAP (first run) | HIT NAP | Stable |
| New rows P2/P4/P5/P6/P7 | mostly unrun | **filled** | **Matrix expansion complete for Tier-1** |
| utm_source=openai | Observed on PJ cards | Re-observed on PJ P1 + Maza P6 home link | **Stable measurement channel** |

**Prior “PJ 1/4 on P1” narrative is obsolete for this snapshot:** P1 is now **3/5** (OpenAI, Claude, Gemini). Perplexity+Grok remain the hard misses on discovery; price query uniquely unlocks Perplexity.

### 5) Cost & ops

| Engine | Σ cost (7 prompts) |
|--------|---------------------|
| OpenAI | $0.163 |
| Claude | $0.132 |
| Gemini | $0.692 |
| Perplexity | $0.069 |
| Grok | $0.346 |
| **Total** | **≈ $1.402** |

Gemini dominates spend (long grounded answers). Perplexity is cheapest per cell.

### 6) What this does *not* prove

1. **Causation** — ablation (t_70d29518 §B) still unrun.  
2. **Consumer-app parity** — API-proxy only.  
3. **Stability** — single-shot; Gemini P1 flip vs yesterday shows variance.  
4. **P2/P7 = broken site** — edge is 200; intent/SERP competition explains zeros.  
5. **Gemini hours fidelity** — P6 hours wrong despite entity hit.  
6. **llms.txt / UCP as citation cause** — not separable here.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | Tier-1 matrix P1–P7 × 5 engines was fully executed live (35/35) on 2026-10-30 via OpenRouter native web search. | E_matrix, captures/* | high | supported |
| C2 | Maza anonymous halal discovery (P3) is **5/5** engines cited this run. | E_p3_* | high | supported (API-proxy) |
| C3 | OpenAI, Claude, and Grok reproduced Maza NAP (address and/or phone and/or correct hours) on P3; OpenAI also on P6 with full exact hours. | E_p3_openai, E_p3_claude, E_p3_grok, E_p6_openai | high | supported |
| C4 | PJ business-card discovery (P1) is **3/5** this run (OpenAI, Claude, Gemini); Perplexity and Grok miss. | E_p1_* | high | supported |
| C5 | Gemini × P1 flipped from prior **MISS** (t_528f7761) to **HIT** with ~$75 quote — treat as unstable until multi-shot. | E_p1_gemini, prior t_528f7761 | medium | supported (delta; variance caveat) |
| C6 | PJ specialty lanes P2 (wraps) and P7 (signs/banners) are **0/5** — competitors only; edge still 200. | E_p2_*, E_p7_*, E_edge | high | supported |
| C7 | Only Perplexity attributed **Print Junkie $75/1,000** on P5; other engines often emit $75-ish bands without naming PJ. | E_p5_* | high | supported |
| C8 | Unscoped “best Mediterranean hours” (P6) drops Claude/Perplexity off Maza; OpenAI keeps exact hours; Gemini names Maza but quotes **incorrect** midweek hours. | E_p6_* | high | supported |
| C9 | Edge bot access remains fully open (24/24 HTTP 200) on home + intent pages. | E_edge | high | supported |
| C10 | OpenAI continues appending `utm_source=openai` on cited merchant URLs (P1 PJ intent; P6 Maza home). | E_p1_openai, E_p6_openai | high | supported |
| C11 | Maza delivery intent (P4) is partial (3/5); delivery-graph competitors win OpenAI/Perplexity. | E_p4_* | high | supported |
| C12 | Overall citation density: Maza **11/15** vs PJ **4/20** on Tier-1 this run — parity gap persists, concentrated in PJ non-card + Perplexity/Grok discovery. | E_matrix | high | supported |
| C13 | Causal claim that agent-visibility stack *caused* citations remains **unproven** without ablation. | protocol §B | high | supported (boundary) |
| C14 | Results are **API-proxy baselines**, not consumer ChatGPT/Claude/Gemini/Grok app guarantees. | method | high | supported (boundary) |

---

## evidence_index

| evidence_id | source_label | source_url / path | provenance | support note |
|-------------|--------------|-------------------|------------|--------------|
| E_matrix | Full 35-cell scores | `.../workspaces/t_d08327bc/captures/matrix_rows.jsonl` + `matrix_final.json` | Live OpenRouter 2026-10-30T07:06–07:21Z | Machine-scored then human-spot-checked |
| E_edge | Bot UA curl matrix | `.../captures/edge_probes.txt` | curl -A bot UA -L 2026-10-30T07:13Z | 24/24 HTTP 200 |
| E_p1_openai | OpenAI×P1 capture | `captures/openai_p1.{json,txt}` | gpt-5.6-luna + native web; ~$0.024 | PJ named “fast turnaround”; $65/$75; utm_source=openai |
| E_p1_claude | Claude×P1 | `captures/claude_p1.*` | claude-haiku-5.5 + web | PJ named Tempe; intent URL |
| E_p1_gemini | Gemini×P1 | `captures/gemini_p1.*` | gemini-3.5-flash + web; ~$0.093 | PJ named + ~$75; delta vs prior miss |
| E_p1_perplexity | Perplexity×P1 | `captures/perplexity_p1.*` | sonar-pro | MISS |
| E_p1_grok | Grok×P1 | `captures/grok_p1.*` | grok-4.3 + web | MISS |
| E_p2_* | All engines ×P2 | `captures/*_p2.*` | same method | Universal MISS for PJ |
| E_p3_openai | OpenAI×P3 | `captures/openai_p3.*` | native web | Maza NAP+4.9 |
| E_p3_claude | Claude×P3 | `captures/claude_p3.*` | native web | Maza #1 narrative + full NAP |
| E_p3_gemini | Gemini×P3 | `captures/gemini_p3.*` | native web | Maza #1 + Frye address |
| E_p3_perplexity | Perplexity×P3 | `captures/perplexity_p3.*` | sonar-pro | named+linked |
| E_p3_grok | Grok×P3 | `captures/grok_p3.*` | native web | NAP + Yelp |
| E_p4_* | Delivery lane | `captures/*_p4.*` | | Claude/Gemini/Grok Y; OpenAI/Perplexity N |
| E_p5_perplexity | Perplexity×P5 | `captures/perplexity_p5.*` | sonar-pro | **Print Junkie $75/1000** + intent link |
| E_p5_other | OpenAI/Claude/Gemini/Grok×P5 | `captures/*_p5.*` | | No PJ name (price bands only) |
| E_p6_openai | OpenAI×P6 | `captures/openai_p6.*` | | Exact hours+NAP+utm |
| E_p6_claude | Claude×P6 | `captures/claude_p6.*` | | Green Corner; no Maza |
| E_p6_gemini | Gemini×P6 | `captures/gemini_p6.*` | | Maza named; **hours incorrect** vs GT |
| E_p6_grok | Grok×P6 | `captures/grok_p6.*` | | Thin Maza mention |
| E_p7_* | Signs lane | `captures/*_p7.*` | | Universal MISS; FASTSIGNS/Signarama-led |
| E_prior_1 | First battery | `/home/ice/know/research/multi-engine-citation-battery-first-execution-t_49a81e49.md` | 2026-10-29 | Baseline partial matrix |
| E_prior_2 | Open cells delta | `/home/ice/know/research/citation-battery-open-cells-delta-t_528f7761.md` | 2026-10-30 earlier | Gemini P1 was MISS |
| E_protocol | Battery SoR | `/home/ice/know/research/avp-causal-verification-multi-engine-citation-battery-t_70d29518.md` | | Prompts, schema, ablation owed |
| E_key | OpenRouter key preflight | GET `/api/v1/key` | 2026-10-30 | usable credits; not free tier |

---

## gaps

1. **Single-shot only** — protocol wants 2–3× for stability (especially Gemini P1 flip).  
2. **Consumer-app legs** still unrun (ChatGPT/Claude/Gemini/Grok product UI).  
3. **Tier-2/3** (P8–P12 named-entity + adversary traps) not in this task’s execution set (P11/P12 partially illuminated by P6 unscoped behavior only).  
4. **Ablation §B** unrun — no causal lever ranking.  
5. **Gemini Vertex redirect URLs** — host often from citation *title*, not bare href.  
6. **Workspace captures may GC** — scores/excerpts durable here; re-attach raw JSON pack only if operator needs binary audit.  
7. **Phone for PJ** never surfaced in Tier-1 hits this run (price/turnaround/location dominated).  
8. **P4 delivery confirmation** for Maza inconsistent across engines (Claude explicitly uncertain).  
9. **Bible process spine file** `bible-research-loop-process-20261030.md` was **not present** under know/research at run time — used t_70d29518 + prior battery reports as SoR instead.

---

## confidence

```yaml
level: high
rationale: >
  All 35 cells returned usable completions with saved full bodies; scoring
  spot-checked against raw text for every HIT and for surprising MISS/price
  cells; edge probes deterministic 24/24; deltas vs dated prior reports are
  explicit. Residual uncertainty is black-box index variance (Gemini P1
  flip), API-proxy≠consumer-app, and single-shot noise — not missing data
  or failed execution. Gemini P6 hours error is documented, not papered over.
```

---

## recommended_next_actions

1. **AVP / Bible 3.0 measurement doctrine:** Replace “PJ 1/4 on P1” with **P1 = 3/5 (API-proxy 2026-10-30)** and publish full Tier-1 heatmap (this report). Keep language as *measured citations*, never guarantees.  
2. **Multi-shot stability pass (cheap):** Re-run Gemini×P1 and Perplexity×P1/P5 ×3 to bound variance before product claims.  
3. **PJ remediation hypotheses to test (not assume):**  
   - P2/P7 zeros → dedicated crawlable intent pages + GBP categories for wraps/signs, or accept specialty-lane non-compete.  
   - Perplexity/Grok P1 miss → directory/GBP/citation-graph work vs AlphaGraphics-class entities.  
   - P5 ambient $75 without name → ensure price pages are the *only* crisp local 1k-card quote engines can attribute.  
4. **Maza:** Protect P3 5/5; improve delivery-graph presence for P4; treat unscoped “best Mediterranean” (P6) as known trap — content/reviews strategy if that query class matters.  
5. **Instrumentation:** GA4/server filter `utm_source=openai` (reconfirmed).  
6. **Still owed:** consumer-app gold run; ablation staging; Tier-2/3 battery.  
7. **Do not** ship Gemini hours as verified — P6 shows entity hit ≠ fact fidelity.

---

## appendix A — compact scoring rows

| run_date | engine | model | prompt | cited | type | facts | wrong/top competitors (sample) | cost_usd | latency_s |
|----------|--------|-------|--------|-------|------|-------|----------------------------------|----------|-----------|
| 2026-10-30 | OpenAI | gpt-5.6-luna | P1 | Y | named+linked+quoted | $65/100, $75/1k, Tempe | AlphaGraphics, Mousegraphics, Staples, FedEx | 0.0245 | 12.6 |
| 2026-10-30 | Claude | haiku-5.5 | P1 | Y | named+linked | Tempe, rush | Eyeconic, AlphaGraphics | 0.0121 | 5.9 |
| 2026-10-30 | Gemini | 3.5-flash | P1 | Y | named+linked+quoted | ~$75/1k | AlphaGraphics, Mousegraphics, FedEx, UPS | 0.0931 | 14.2 |
| 2026-10-30 | Perplexity | sonar-pro | P1 | N | not-present | — | AlphaGraphics, CesarGraphics, UPS | 0.0113 | 3.0 |
| 2026-10-30 | Grok | 4.3 | P1 | N | not-present | — | AlphaGraphics, Outta PHX, Splash, JC | 0.0387 | 31.0 |
| 2026-10-30 | all×5 | — | P2 | N | not-present | — | PRI/Custom Graphix/Fast-Trac/etc. | ~0.17 | — |
| 2026-10-30 | OpenAI | gpt-5.6-luna | P3 | Y | named+linked+quoted | addr, phone, hours, 4.9 | Greeks Grill | 0.0216 | — |
| 2026-10-30 | Claude | haiku-5.5 | P3 | Y | named+quoted | addr, phone, hours | Greeks Grill, Green Corner | 0.0120 | — |
| 2026-10-30 | Gemini | 3.5-flash | P3 | Y | named+linked+quoted | addr (#1) | Greeks, Arabella, BelAfia | 0.1042 | 20.3 |
| 2026-10-30 | Perplexity | sonar-pro | P3 | Y | named+linked | addr | Green Corner | 0.0093 | 2.5 |
| 2026-10-30 | Grok | 4.3 | P3 | Y | named+linked+quoted | addr, phone, hours | Green Corner, TurkDish, Yelp | 0.0423 | 13.3 |
| 2026-10-30 | OpenAI | gpt-5.6-luna | P4 | N | not-present | — | Green Corner, BelAfia, apps | 0.0347 | 19.7 |
| 2026-10-30 | Claude | haiku-5.5 | P4 | Y | named+linked | addr | Greeks, Green Corner, apps | 0.0235 | 9.4 |
| 2026-10-30 | Gemini | 3.5-flash | P4 | Y | named+linked | addr + site order | King of Gyros, apps | 0.0796 | 14.9 |
| 2026-10-30 | Perplexity | sonar-pro | P4 | N | not-present | — | King of Gyros, Greeks | 0.0101 | 3.0 |
| 2026-10-30 | Grok | 4.3 | P4 | Y | named | thin | King of Gyros, Greeks, apps | 0.0463 | 19.9 |
| 2026-10-30 | OpenAI | gpt-5.6-luna | P5 | N | not-present | band ~$75 w/o name | AlphaGraphics, Biltmore | 0.0232 | 10.1 |
| 2026-10-30 | Claude | haiku-5.5 | P5 | N | not-present | — | — | 0.0235 | 7.3 |
| 2026-10-30 | Gemini | 3.5-flash | P5 | N | not-present | band includes $75 w/o name | AlphaGraphics, FedEx, UPS | 0.1045 | 13.1 |
| 2026-10-30 | Perplexity | sonar-pro | P5 | Y | named+linked+quoted | **$75/1k** | Eyeconic, Staples, AlphaGraphics | 0.0090 | 2.2 |
| 2026-10-30 | Grok | 4.3 | P5 | N | not-present | band w/o name | AlphaGraphics, Outta PHX | 0.0658 | 19.5 |
| 2026-10-30 | OpenAI | gpt-5.6-luna | P6 | Y | named+linked+quoted | **exact hours+NAP** | Pitta Souvli alt | 0.0223 | 8.2 |
| 2026-10-30 | Claude | haiku-5.5 | P6 | N | not-present | Green Corner hours | Green Corner, Pitta Souvli | 0.0248 | 8.1 |
| 2026-10-30 | Gemini | 3.5-flash | P6 | Y | named+linked | **hours WRONG** vs GT | Pitta, Greeks, Green Corner | 0.1882 | 21.8 |
| 2026-10-30 | Perplexity | sonar-pro | P6 | N | not-present | — | non-Maza | 0.0079 | 1.9 |
| 2026-10-30 | Grok | 4.3 | P6 | Y | named | thin mention | Greeks, Green Corner | 0.0700 | 8.7 |
| 2026-10-30 | all×5 | — | P7 | N | not-present | — | FASTSIGNS, Signarama, Print N Ship, AlphaGraphics | ~0.13 | — |

---

## appendix B — honesty block (carry to AVP / Bible)

- Label every figure **API-proxy baseline (OpenRouter native search), single-shot 2026-10-30**.  
- **Do not** say “ChatGPT recommends Print Junkie 3/5” — say “OpenAI API web-grounded answer cited PJ on P1 this run.”  
- **Do not** claim Gemini P1 improvement is permanent.  
- **Do not** claim P2/P7 prove technical failure — they prove **intent non-win**.  
- **Do not** claim schema hours “work on Gemini” after P6 corruption.  
- **Do** claim Maza P3 multi-engine discovery is the strongest measured row to date (5/5 API-proxy).  
- **Do** track `utm_source=openai`.

---

*End of report — t_d08327bc. Single durable archive: `/home/ice/know/research/citation-battery-tier1-p1-p7-expansion-t_d08327bc.md`. Prepared by Geraldo v2.2. No double-publish.*
