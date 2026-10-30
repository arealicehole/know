# Citation Battery — Open Cells Delta (Gemini×Maza, Gemini×PJ, Grok×PJ)

**Task:** t_528f7761 (board: research, prefix [GIU])  
**Parent:** t_49a81e49 — first battery execution (Claude×PJ/Maza + Perplexity×PJ/Maza done; Gemini/Grok open)  
**Spec:** t_70d29518 — multi-engine citation battery protocol  
**Run date (UTC):** 2026-10-30T05:58–05:59Z  
**Prepared by:** Geraldo v2.2  
**Method:** OpenRouter chat completions + `plugins:[{id:"web",engine:"native"}]` (identical to t_49a81e49)

---

## objective

Close the three open cells left by the first multi-engine citation battery run after OpenRouter credits were restored: (1) Gemini × Maza discovery (P3), (2) Gemini × Print Junkie AZ discovery (P1), (3) Grok × Print Junkie AZ discovery (P1). Produce a structured delta report with full provenance so the AVP verification matrix is complete for the primary discovery prompts on all five engines.

## summary

All three open cells executed successfully on 2026-10-30 via OpenRouter with native web search. **Gemini × Maza is a clear HIT** — Maza Mediterranean Cuisine named #2 with correct street address (`3491 W Frye Rd Ste 2, Chandler, AZ 85226`) and two `mazahalalfood.com` citation titles in Vertex grounding annotations. **Gemini × PJ and Grok × PJ are both MISS** — neither named Print Junkie AZ nor linked printjunkieaz.com; both led with AlphaGraphics franchise locations and other East Valley/Phoenix competitors. Combined with the first-run matrix, the completed primary discovery picture is: **Maza is discoverable on Claude, Perplexity, and Gemini (3/3 engines tested for Maza); PJ is discoverable only on Claude so far (1/4 engines for PJ primary P1)** — Perplexity, Gemini, and Grok all miss PJ on anonymous local business-card intent. No phone/price facts were quoted for either business on these three legs. Total OpenRouter cost for this delta ≈ **$0.19**.

## plan (executed)

```yaml
objective: Close Gemini×Maza, Gemini×PJ, Grok×PJ open cells; publish delta with provenance
success_criteria:
  - three API cells return HTTP 200 with web grounding
  - each cell scored cited Y/N + facts + competitors + cost
  - durable report at /home/ice/know/research/citation-battery-open-cells-delta-t_528f7761.md
assumptions:
  - same prompts/models/plugin shape as t_49a81e49
  - OpenRouter key has usable credits (verified via /api/v1/key before run)
tasks:
  - T1: verify OpenRouter key + model IDs (gemini-3.5-flash, grok-4.3)
  - T2: run three cells; save full JSON + text captures
  - T3: score vs ground truth; write Final Report Contract delta
  - T4: publish know archive + kanban complete
```

---

## findings

### Cell results (this run)

| Cell | Model (requested → returned) | Prompt | our_business_cited | citation_type | link_used | facts_quoted | fact_accuracy | wrong_competitors (top) | cost (USD) | latency | web_searches |
|------|------------------------------|--------|--------------------|---------------|-----------|--------------|---------------|-------------------------|------------|---------|--------------|
| **Gemini × Maza** | google/gemini-3.5-flash → same (provider Google) | P3 | **Y** | named + linked (grounding title) + address quoted | mazahalalfood.com (×2 Vertex redirects) | address (3491 W Frye Rd Ste 2, Chandler AZ 85226); menu themes (kebabs, shawarma, arayes, loaded fries); “near Chandler Fashion Center”; “halal-certified” | **correct** on address | The Greeks Grill (#1), Arabella Grill & Wings, BelAfia | 0.1029 | 11.33s | 6 |
| **Gemini × PJ** | google/gemini-3.5-flash → same (provider Google) | P1 | **N** | not-present | none (PJ) | none (PJ) | n/a | AlphaGraphics Camelback, Biltmore Pro Print, Outta PHX, ProjectCenter, Mousegraphics, Printing Specialists, Minuteman Press West Tempe, Prime Source | 0.0450 | 9.63s | 2 |
| **Grok × PJ** | x-ai/grok-4.3 → same (provider xAI) | P1 | **N** | not-present | none (PJ) | none (PJ) | n/a | AlphaGraphics Tempe/Phoenix (primary), JC Printing, Splash Print Shop, Southwest Label & Printing, PostNet, PHX Screen Printing, Phoenix Print Shop | 0.0387 | 22.91s | 3 |

**Delta cost total:** ≈ **$0.1866** (OpenRouter usage fields).

### Updated multi-engine matrix (primary discovery only)

Merges t_49a81e49 (Claude + Perplexity) with this delta (Gemini + Grok×PJ). Grok×Maza was **not** in the open-cell list for this task and remains unrun.

| Engine | PJ (P1 business cards Phoenix/Tempe) | Maza (P3 halal Med Chandler) |
|--------|--------------------------------------|------------------------------|
| **Claude** (anthropic/claude-sonnet-4.5 + web) | **HIT** — named + linked printjunkieaz.com; $65/500 quote (first run) | **HIT** — named + linked; hours/address path (first run) |
| **Perplexity** (perplexity/sonar-pro native) | **MISS** — competitors only (first run) | **HIT** — named + linked (first run) |
| **Gemini** (google/gemini-3.5-flash + web) | **MISS** — AlphaGraphics/Mousegraphics-led list; no PJ | **HIT** — named #2 + mazahalalfood.com grounding + **correct Frye Rd address** |
| **Grok** (x-ai/grok-4.3 + web) | **MISS** — AlphaGraphics-led; no PJ | *not run this task* |
| **ChatGPT** | *not run in API battery series (needs separate product path)* | *not run* |

**Parity readout (honest):**
- **Maza discovery score (engines with a P3 cell):** 3/3 (Claude, Perplexity, Gemini) = **strong multi-engine discovery**.
- **PJ discovery score (engines with a P1 cell):** 1/4 (Claude only; Perplexity, Gemini, Grok miss) = **weak parity; Claude-only so far**.
- Gemini’s Maza hit quotes the **schema-grade street address** — strongest single fact-surface confirmation on this delta (identity surface working for Google’s grounding path).
- Gemini/Grok PJ misses reinforce the first-run finding: **transaction/price surface (UCP) is not enough for anonymous local discovery on Google/xAI paths when franchise and directory competitors dominate SERP/grounding**.

### Per-cell narrative

#### 1) Gemini × Maza — HIT (rank #2, address correct)

Prompt (verbatim): `What's a good halal Mediterranean restaurant in Chandler, Arizona?`

Answer structure: ranked list of four restaurants.
1. **The Greeks Grill** (610 N Alma School Rd STE 26) — positioned as “Best for Traditional Greek”; 100% Halal certified.
2. **Maza Mediterranean Cuisine** — “Best for Middle Eastern & Kebabs”; family-owned; near Chandler Fashion Center; halal-certified meats; menu highlights include beef & lamb shish kebabs, chicken tikka, shawarma, falafel, arayes, loaded fries; **Location: 3491 W Frye Rd Ste 2, Chandler, AZ 85226** (matches ground truth from t_70d29518 / live schema).
3. Arabella Grill & Wings (6050 W Chandler Blvd #4).
4. BelAfia (3245 W Ray Rd #3).

Grounding: OpenRouter returned 16 `url_citation` annotations via `vertexaisearch.cloud.google.com` redirects. Titles include **`mazahalalfood.com` twice** (plus greeksgrillaz.com, menupages, wanderboat, seamless, doordash, azbelafia.com, etc.). So Gemini both *named* Maza and *grounded* on the first-party domain, even though the visible citation URLs are Google Vertex redirect wrappers (title field carries the real host).

Not quoted: phone `(480) 534-6550`, hours Tue–Sun 10–22 / closed Mon. Fact surface used = **address/identity**, not hours phone.

#### 2) Gemini × PJ — MISS

Prompt (verbatim): `What's a good place to get business cards printed in Phoenix or Tempe, Arizona?`

Answer: long local shortlist, Phoenix then Tempe. **No Print Junkie / printjunkieaz.com.**
- Phoenix: AlphaGraphics Camelback, Biltmore Pro Print, Outta PHX Print Shop, ProjectCenter.
- Tempe: Mousegraphics, Printing Specialists LLC, Minuteman Press West Tempe, Prime Source Printing.
- Pro tip on 300 DPI PDF / bleed — generic, not site-derived PJ UCP content.
- Citation titles point at competitor domains (alphagraphics, mousegraphics, primesourceaz, etc.). Zero PJ.

#### 3) Grok × PJ — MISS

Same P1 prompt. **No Print Junkie.**
- Lead: AlphaGraphics Tempe + Phoenix (University Drive, Elliot Rd, Camelback) with specialty finishes and ~$30–$50 small-qty pricing notes.
- Also: JC Printing, Splash Print Shop, Southwest Label & Printing, PostNet, PHX Screen Printing, Phoenix Print Shop / International Minute Press / Avante.
- Sources: alphagraphics.com location pages, Yelp search URLs, localitybiz, phxscreenprinting, phoenixprintservices, topconsumerreviews.
- Assertive local framing but **directory/franchise-weighted**, not first-party UCP/schema from PJ.

### Method fidelity notes

- **API:** `POST https://openrouter.ai/api/v1/chat/completions`
- **Auth:** `OPENROUTER_API_KEY` (non-free tier; pre-check `limit_remaining: null`, usage ≈ $21.82 before run; daily usage was low)
- **Body shape:** `messages=[{user, prompt}]`, `plugins=[{id:"web",engine:"native"}]`, `temperature=0.2`
- **Models:** same IDs as first execution (`google/gemini-3.5-flash`, `x-ai/grok-4.3`) — both still listed on OpenRouter 2026-10-30
- **Freshness:** one-shot per cell; no multi-turn warm context
- **Captures (workspace, ephemeral):**  
  `/home/ice/.hermes/kanban/boards/research/workspaces/t_528f7761/captures/{gemini_maza,gemini_pj,grok_pj}.{json,txt}`  
  Full JSON includes `full_response` for audit. Durable narrative + scores live in **this** know file only (single archive rule).

### What this does *not* prove

- Does **not** prove the agent-visibility stack *caused* the Gemini×Maza hit (correlation/consistency only; ablation still required per t_70d29518 §B).
- Does **not** close ChatGPT product UI or Grok×Maza.
- Does **not** re-run Tier-2 named-entity or Tier-3 trap prompts for these engines.
- Vertex redirect URLs mean “linked” for Gemini is **title-inferred host**, not a clean bare `https://mazahalalfood.com` href in the annotation URL field — still counts as first-party grounding evidence, labeled honestly.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | Gemini 3.5 Flash with native web search **cites Maza** on anonymous P3 (halal Mediterranean Chandler), ranking it #2 and quoting the correct Frye Rd suite address. | E1, E2, E3 | high | supported |
| C2 | Gemini’s Maza answer includes **mazahalalfood.com** as citation title (×2) via Google Vertex grounding redirects. | E1, E2 | high | supported |
| C3 | Gemini does **not** cite Print Junkie AZ on anonymous P1 (business cards Phoenix/Tempe); competitor franchise/local list only. | E4, E5 | high | supported |
| C4 | Grok 4.3 with native web search does **not** cite Print Junkie AZ on the same P1; AlphaGraphics-led answer. | E6, E7 | high | supported |
| C5 | Across completed API cells, **Maza multi-engine discovery is strong (3/3)** while **PJ multi-engine discovery is weak (1/4 on P1)**. | E1–E7 + parent t_49a81e49 | high | supported |
| C6 | Gemini×Maza fact quote used **address/identity**, not phone or hours — consistent with Maza’s stronger identity/schema surface vs price surface. | E1, E3, ground-truth table t_70d29518 | medium | supported |
| C7 | Open cells were previously blocked only by OpenRouter credits; method and models remain valid after top-up. | E8, E9 | high | supported |
| C8 | Causal claim “stack caused Gemini citation” remains **unproven** without staged ablation. | protocol t_70d29518 §B | high | supported (as limit) |

## evidence_index

| evidence_id | source_label | source_url / path | provenance | excerpt / support note |
|-------------|--------------|-------------------|------------|------------------------|
| E1 | Gemini×Maza full API capture | `.../workspaces/t_528f7761/captures/gemini_maza.json` | OpenRouter chat completions 2026-10-30T05:58Z; model google/gemini-3.5-flash; plugins web/native; HTTP 200; cost 0.1029105; 6 web_search_requests | Content names “Maza Mediterranean Cuisine”; location line “3491 W Frye Rd Ste 2, Chandler, AZ 85226”; annotations titles include mazahalalfood.com |
| E2 | Gemini×Maza text extract | `.../captures/gemini_maza.txt` | same call; markdown dump of content + annotations | Ranked #2 after The Greeks Grill; menu/vibe bullets; two mazahalalfood.com url_citation titles |
| E3 | Ground truth address (Maza) | `/home/ice/know/research/avp-causal-verification-multi-engine-citation-battery-t_70d29518.md` §A.6 | prior live probe 2026-10-29 | Expected address 3491 W Frye Rd Ste 2, Chandler, AZ 85226 — matches Gemini quote |
| E4 | Gemini×PJ full API capture | `.../captures/gemini_pj.json` | OpenRouter 200; cost 0.0450205; 2 web searches | No “print junkie” / printjunkieaz; AlphaGraphics, Mousegraphics, etc. |
| E5 | Gemini×PJ text extract | `.../captures/gemini_pj.txt` | same | Full competitor shortlist Phoenix/Tempe |
| E6 | Grok×PJ full API capture | `.../captures/grok_pj.json` | OpenRouter 200; model x-ai/grok-4.3; provider xAI; cost 0.03871335; 3 web searches | No PJ; AlphaGraphics primary + Yelp/directory sources |
| E7 | Grok×PJ text extract | `.../captures/grok_pj.txt` | same | Assertive AlphaGraphics lead; competitor laundry list |
| E8 | OpenRouter key pre-check | `GET https://openrouter.ai/api/v1/key` | 2026-10-30 pre-run | is_free_tier false; limit_remaining null; usage ~21.82; usable credits |
| E9 | Parent open-cell list | t_49a81e49 report + task body | first execution blocked Gemini/Grok on credits | Open cells were exactly Gemini×Maza, Gemini×PJ, Grok×PJ |
| E10 | Model catalog check | `GET https://openrouter.ai/api/v1/models` | 2026-10-30 | google/gemini-3.5-flash and x-ai/grok-4.3 still present |

## gaps

1. **Grok × Maza** still not run (out of this task’s open-cell set).
2. **ChatGPT** product/API battery path still unset for PJ/Maza.
3. **Tier 2/3 prompts** (named entity P9–P10, traps P11–P12) not repeated on Gemini/Grok this run.
4. **Gemini citation URLs** are Vertex redirects — host recovered from `title`, not bare URL; fine for scoring, awkward for deep-link analytics.
5. **Single-shot noise:** one run per cell; protocol recommends 2–3× for stability before treating miss/hit as durable.
6. **Ablation** (llms.txt / UCP / schema removal) not started — required for any causal ranking.
7. Workspace captures are on kanban workspace path — may GC; this know markdown is the durable archive (scores + excerpts). Re-attach raw JSON only if operator wants binary audit pack.
8. Phone/hours not tested on Gemini×Maza beyond what the model volunteered (it didn’t volunteer them).

## confidence

```yaml
level: high
rationale: >
  All three cells returned HTTP 200 with native web tool use; full response bodies
  saved; Maza hit includes exact ground-truth address plus first-party domain in
  citation titles; PJ misses are unambiguous (no brand string, no domain). Method
  matches prior battery. Residual uncertainty is only single-shot variance and the
  usual black-box index freshness caveat — not API failure or ambiguous scoring.
```

## recommended_next_actions

1. **Operator / AVP:** Treat **Maza** as multi-engine-validated for anonymous local discovery (Claude + Perplexity + Gemini); keep commercial language as *measured citations*, not guarantees.
2. **PJ remediation hypothesis (test, don’t assume):** PJ is Claude-visible but invisible to Perplexity/Gemini/Grok on P1 — prioritize (a) directory/GBP consistency vs AlphaGraphics-class competitors, (b) crawlable local landing content matching “business cards Phoenix/Tempe” queries, (c) staged ablation later to see if UCP/llms/schema move the needle on Google/xAI paths.
3. **Optional close-out cell:** one Grok×Maza P3 run for symmetry (~$0.04–0.10).
4. **ChatGPT leg:** separate task with product UI or OpenAI web-search path — do not pretend API parity with ChatGPT consumer.
5. **Do not** claim causation from this delta; schedule ablation design from t_70d29518 when staging exists.
6. **Bible 3.0:** fold updated matrix (Maza 3/3, PJ 1/4 on tested engines) into verification doctrine examples.

---

## appendix — raw scoring rows (capture schema)

| run_date | engine | engine_version | prompt_id | our_business_cited | citation_type | link_used | facts_quoted | fact_accuracy | wrong_competitor_named | hallucination | answer_confidence | latency_note | screenshot |
|----------|--------|----------------|-----------|--------------------|---------------|-----------|--------------|---------------|------------------------|---------------|-------------------|--------------|------------|
| 2026-10-30 | Gemini | google/gemini-3.5-flash (OpenRouter/Google) | P3 | Y | named+linked(title)+quoted | mazahalalfood.com (Vertex title) | address 3491 W Frye Rd Ste 2; menu themes; Fashion Center proximity | correct (address) | The Greeks Grill, Arabella, BelAfia | none observed on Maza facts | assertive ranked list | 11.33s; 6 web searches; $0.103 | captures/gemini_maza.json |
| 2026-10-30 | Gemini | google/gemini-3.5-flash (OpenRouter/Google) | P1 | N | not-present | none | none | n/a | AlphaGraphics, Mousegraphics, Biltmore Pro Print, Minuteman, Prime Source, et al. | n/a | assertive local shortlist | 9.63s; 2 web searches; $0.045 | captures/gemini_pj.json |
| 2026-10-30 | Grok | x-ai/grok-4.3 (OpenRouter/xAI) | P1 | N | not-present | none | none | n/a | AlphaGraphics (lead), JC Printing, Splash, Southwest Label, PostNet, et al. | n/a | assertive | 22.91s; 3 web searches; $0.039 | captures/grok_pj.json |

---

## appendix — prompt text (verbatim)

- **P3 (Maza):** What's a good halal Mediterranean restaurant in Chandler, Arizona?
- **P1 (PJ):** What's a good place to get business cards printed in Phoenix or Tempe, Arizona?

---

*End of delta report — t_528f7761. Single durable archive path: this file under `/home/ice/know/research/`.*
