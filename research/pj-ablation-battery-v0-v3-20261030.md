# PJ prospective ablation battery V0–V3 (2026-10-30)

task_id: `t_6e47f5ab`  
parents: `t_f1118fbb` (host decision) · protocol `t_70d29518` §B · Tier-1 prompts `t_d08327bc`  
run_date_utc: 2026-10-30  
scope: **preview-only / domain-shifted (`*.vercel.app`)** — **no prod causation**

---

## objective

Run a prospective single-variable ablation battery on Print Junkie AZ **preview** hosts:

| Variant | Host | Intended single change |
|---------|------|------------------------|
| **V0** control | https://web-qgfwqxqjz-tricon-digital.vercel.app | full stack |
| **V1** no llms.txt | https://web-9zr136bqb-tricon-digital.vercel.app | `llms.txt` → 404 |
| **V2** no UCP | https://web-bg4t8s6td-tricon-digital.vercel.app | `/.well-known/ucp` → 404 |
| **V3** identity thin | https://web-hd8c3mgrl-tricon-digital.vercel.app | JSON-LD identity thinned |

Same Tier-1 prompts **P1** (discovery) and **P5** (1k card price). Engines: OpenAI, Claude, Gemini, Perplexity, Grok via OpenRouter native web.

**Two legs**

1. **URL-forced (PRIMARY for stack-lever ranking)** — prompt injects the variant URL and asks the model to use it as primary source.
2. **Open-discovery (SECONDARY)** — bare Tier-1 prompts; no preview URL. Expect weak/zero vercel retrieval (autopsy-compatible).

Operator mid-run note (t_f1118fbb autopsy): if open-discovery is flat across V0–V3, that is **compatible** with miss engines never retrieving `*.vercel.app` / PJ without street NAP. Do **not** read “llms/UCP didn’t move open-discovery” as “stack worthless.” Stack ≈ offer reliability **when retrieved**; retrieval ≈ place/authority graph.

---

## summary

- **Hosts verified** under `OAI-SearchBot`: V0 llms+ucp 200; V1 llms **404** / ucp 200; V2 llms 200 / ucp **404**; V3 both 200. V3 is a **partial** identity thin (LocalBusiness **PostalAddress removed**; name/phone/hours/products/FAQ JSON-LD **retained**) — not a full schema wipe.
- **50/50 cells completed** (5 engines × 2 prompts × [1 open + 4 forced variants]). OpenRouter spend ≈ **$2.69** this matrix (plus prior session usage).
- **Open-discovery:** no engine retrieved any `tricon-digital.vercel.app` preview. Print Junkie appeared only sporadically via **prod/index** paths (OpenAI P1 named+`printjunkieaz.com`; Gemini/Perplexity P5 named or $75). Uniform non-retrieval of previews is **autopsy-compatible**, not a stack-null result.
- **URL-forced (ranking leg):** OpenAI and Gemini extracted **brand + phone + ~$75** at offer_score=3 on **all four** variants (including no-llms and no-UCP). Visible HTML / remaining schema carried offer facts without llms.txt or UCP. Grok stably named the brand (offer≈1) but rarely phone/$75. Claude **systematically failed to fetch** vercel staging (search-only tool path) → near-zero offer signal, not informative for lever rank. Perplexity often could not pull page body (“Tricon Digital” label, thin facts); noisy V0→V2/V3 drop on P1 only.
- **Pooled forced impact** `avg_offer(V0) − avg_offer(Vx)` on 0–3 scale: **V1 = 0.00**, **V2 = 0.20**, **V3 = 0.20** (n=10 cells/variant). Effects are **small**, **engine-specific**, and **not** a clean multi-engine kill. Honest rank under preview+forced conditions: **no lever is load-bearing once URL is known for OpenAI/Gemini**; residual pooled gap is mostly Perplexity fetch noise + tiny V2/V3 means.
- **Doctrine update (preview-scoped):** when the engine can read the page, **on-page price chips + core LocalBusiness (name/tel)** dominate offer reliability; **llms.txt and UCP are not required for forced-URL offer extraction** on these hosts. That does **not** prove they are worthless for **discovery/retrieval** on prod domains — this battery cannot measure prod retrieval causation.

---

## plan

```
objective: prospective V0–V3 ablation on PJ preview hosts; rank stack levers
success_criteria: host verify; P1+P5 × 5 engines × open+forced; scored matrix; ranked impact; preview-only labels
assumptions: OpenRouter native-web ≈ prior Tier-1; Vercel previews crawlable when URL-forced; V3 = identity thin as deployed
tasks:
  T1 host signal verify (done)
  T2 open-discovery battery P1/P5 × 5 engines (done)
  T3 URL-forced battery V0–V3 × P1/P5 × 5 engines (done; primary)
  T4 strict rescore + impact ranking + Final Report Contract publish (this doc)
open_questions:
  - Would full LocalBusiness wipe (true V3) move OpenAI/Gemini?
  - Prod-domain ablation (same apex) needed for discovery causation
  - V4 visible-$ and V5/V6 bot-block still unrun
```

---

## method

### Engines (requested = returned on success)

| Engine | Model | Search path |
|--------|-------|-------------|
| OpenAI | `openai/gpt-5.6-luna` | `plugins:[{id:"web",engine:"native"}]` |
| Claude | `anthropic/claude-haiku-5.5` | native web plugin |
| Gemini | `google/gemini-3.5-flash` | native web plugin (`max_tokens` capped on retries) |
| Perplexity | `perplexity/sonar-pro` | built-in search |
| Grok | `x-ai/grok-4.3` | native web plugin |

- API: `POST https://openrouter.ai/api/v1/chat/completions`
- temperature 0.2; fresh session per cell; schedule shuffled (SEED=42)
- Workspace captures: `/home/ice/.hermes/kanban/boards/research/workspaces/t_6e47f5ab/captures/`
- Runner: `run_ablation_battery.py` (same tree)

### Prompts (verbatim Tier-1)

| ID | Text |
|----|------|
| P1 | What's a good place to get business cards printed in Phoenix or Tempe, Arizona? |
| P5 | How much does it cost to get 1,000 business cards printed in Phoenix? |

### URL-forced wrapper

> `{prompt}` + “Please open and carefully use this specific website as your primary source… `{variant_url}` … Extract identity, contact, and any price facts… Cite the URL you used.”

### Scoring (strict, post-hoc)

`offer_score` ∈ {0,1,2,3} = `named(Print Junkie)` + `phone(602-730-2135)` + `visible $75`  
`extracted` = named ∧ (phone ∨ $75)  
`host` = forced variant host appears in answer/citations  
`fetch_fail` = model states it could not open/retrieve the staging URL  

impact(s) = mean_offer(V0) − mean_offer(variant_s) on the forced leg (protocol §B.5).

### Host verification (OAI-SearchBot, run-time)

| Path | V0 | V1 | V2 | V3 |
|------|----|----|----|-----|
| `/` | 200 | 200 | 200 | 200 |
| `/llms.txt` | 200 | **404** | 200 | 200 |
| `/.well-known/ucp` | 200 | 200 | **404** | 200 |
| LocalBusiness JSON-LD | full + PostalAddress | same as V0 | same as V0 | **no PostalAddress**; name/tel/hours kept |
| Product/FAQ ld+json | present | present | present | present |

---

## findings

### 1) Open-discovery leg (secondary)

| Engine | P1 named PJ | P5 named PJ | Preview URL ever | Notes |
|--------|-------------|-------------|------------------|-------|
| OpenAI | **Y** (+ printjunkieaz.com) | N | N | Prod/index path, not preview |
| Claude | N | N | N | Competitors only |
| Gemini | N | **Y** (+$75 mention) | N | Likely index/memory, not vercel |
| Perplexity | N | **Y** (+$75, prod-ish) | N | |
| Grok | N | N ($75 competitor noise possible) | N | |

**Interpretation:** Engines do not organically surface `*.vercel.app` PJ previews on P1/P5. Occasional PJ hits track **production brand/index**, not the ablation host. Flat open-discovery across “variants” is expected and **does not rank** llms/UCP/schema. Matches autopsy: retrieval ≠ stack body.

### 2) URL-forced matrix — offer_score (named+phone+$75)

Legend: `o{0-3}` offer · `h` forced-host cited · `e` extracted brand∧(phone∨$75) · `F` fetch_fail

| Engine/Prompt | V0 | V1 (no llms) | V2 (no UCP) | V3 (addr thin) |
|---------------|----|--------------|-------------|----------------|
| openai/P1 | o3 h e | o3 h e | o3 h e | o3 h e |
| openai/P5 | o3 h e | o3 h e | o3 h e | o3 h e |
| gemini/P1 | o3 h e | o3 h e | o3 h e | o3 h e |
| gemini/P5 | o3 h e | o3 h e | o3 h e | o3 h e |
| grok/P1 | o1 h | o1 h | o1 h | o1 h |
| grok/P5 | o1 h | o1 h | o1 h | o1 h |
| claude/P1 | o1 h F | o2 h e F | o1 h F | o1 F |
| claude/P5 | o0 F | o0 F | o0 F | o0 F |
| perplexity/P1 | o2 h | o1 h | o0 h | o0 h |
| perplexity/P5 | o1 h | o1 | o1 | o1 h |

### 3) Pooled forced means (n=10 ok cells / variant)

| Variant | avg offer | named rate | phone rate | $75 rate | host rate | extracted rate | fetch_fail rate |
|---------|-----------|------------|------------|----------|-----------|----------------|-----------------|
| V0 | **1.80** | 0.80 | 0.40 | 0.60 | 0.90 | 0.50 | 0.30 |
| V1 | **1.80** | 0.70 | 0.60 | 0.50 | 0.80 | 0.50 | 0.20 |
| V2 | **1.60** | 0.70 | 0.40 | 0.50 | 0.80 | 0.40 | 0.20 |
| V3 | **1.60** | 0.70 | 0.40 | 0.50 | 0.80 | 0.40 | 0.20 |

**impact vs V0:** V1 **0.00** · V2 **0.20** · V3 **0.20**

Paired mean Δ (same engine×prompt): V1 0.00 · V2 0.20 · V3 0.20 — **driven almost entirely by Perplexity/P1** (Δ=+2 on V2 and V3). OpenAI/Gemini/Grok paired Δ = 0 on every cell.

### 4) P5-only (price prompt) forced — flat

All four variants: mean offer **1.60**, p75 rate **0.60**, phone rate **0.40** (n=5). **UCP removal did not drop $75 extraction** for engines that can read HTML (OpenAI/Gemini still o3 on V2 P5 with explicit $75 + phone from page).

### 5) Engine behavior notes (forced)

- **OpenAI / Gemini:** Reliably fetch vercel HTML; quote Print Junkie AZ, 602-730-2135, $75 / 1,000 even when llms.txt or UCP is 404. Example (V2 OpenAI P5): “1,000 business cards cost $75 at Print Junkie AZ… Phone: 602-730-2135” citing `web-bg4t8s6td-…/business-cards-phoenix`.
- **Grok:** Names Print Junkie / uses host; shallow on NAP/price facts across all variants (stable o1).
- **Claude Haiku 5.5:** Repeated “I couldn't open the staging site… tools only run web searches.” **Not a valid ablation instrument** for vercel previews on this tool path.
- **Perplexity:** Frequently fails body fetch; labels shop “Tricon Digital”; mixes competitor prices. Treat V0→V2/V3 P1 drop as **low-confidence / fetch artifact**, not clean UCP causation.

### 6) Signal ranking (forced leg, preview-only)

Protocol hypothesis was schema > llms > UCP for identity/discovery. **This run’s forced-leg evidence (preview hosts):**

| Rank (this run) | Signal | impact | Confidence | Caveat |
|-----------------|--------|--------|------------|--------|
| 1 (tie / weak) | UCP (V2) | 0.20 | **low** | Pooled only; OpenAI/Gemini unchanged; Perplexity-dominated |
| 1 (tie / weak) | Identity thin (V3) | 0.20 | **low** | V3 only removed PostalAddress — phone/name remain |
| 3 | llms.txt (V1) | 0.00 | **medium** that it’s *not* required for forced offer extraction | Does not test discovery |

**Stronger qualitative finding than the 0.2 deltas:** for URL-known offer reliability on these pages, **HTML price chips + core LocalBusiness name/tel** are sufficient for top engines; ablating llms.txt or UCP does not flip OpenAI/Gemini outcomes.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | V0–V3 preview hosts match intended llms/UCP HTTP status matrix under OAI-SearchBot | E1 | high | supported |
| C2 | V3 is PostalAddress-thin only; LocalBusiness name/telephone and other ld+json blocks remain | E2 | high | supported |
| C3 | Open-discovery never cited `tricon-digital.vercel.app` for any engine×P1/P5 cell | E3 | high | supported |
| C4 | Open-discovery PJ hits (when present) track prod/index, not ablation previews | E3, E4 | medium | supported |
| C5 | URL-forced OpenAI and Gemini achieve offer_score=3 on all V0–V3 for P1 and P5 | E5 | high | supported |
| C6 | Removing llms.txt (V1) does not reduce forced-leg offer_score for OpenAI/Gemini/Grok vs V0 | E5, E6 | high | supported |
| C7 | Removing UCP (V2) does not prevent OpenAI/Gemini from quoting $75 + phone from HTML on P5 | E5, E7 | high | supported |
| C8 | Pooled forced impact is 0.00 (V1) / 0.20 (V2) / 0.20 (V3); small-N, Perplexity-heavy | E6 | medium | supported |
| C9 | Claude Haiku native-web path systematically fails to fetch these vercel previews → non-informative for lever rank | E8 | high | supported |
| C10 | Results are preview/domain-shifted only; **no production causation** for printjunkieaz.com citation rates | E9 | high | supported |
| C11 | Autopsy framing holds: open-discovery flatness ≠ “stack worthless”; forced leg isolates offer reliability when retrieved | E3, E5, E10 | high | supported |
| C12 | Hypothesis “UCP is primary price-discovery trigger” is **not supported** for forced-URL OpenAI/Gemini on these hosts | E5, E7 | medium | supported |
| C13 | Hypothesis “llms.txt is primary identity-discovery trigger” is **not testable** on open-discovery here and **not supported** for forced-URL offer extraction | E3, E5, E6 | medium | supported |

---

## evidence_index

| evidence_id | source_label | source_url / path | provenance | excerpt / note |
|-------------|--------------|-------------------|------------|----------------|
| E1 | Host HTTP probe | V0–V3 vercel URLs | curl OAI-SearchBot 2026-10-30 | V1 llms 404; V2 ucp 404; others 200 |
| E2 | JSON-LD diff V0 vs V3 | home HTML ld+json block0 | python extract | V3 drops `address`/`PostalAddress`; keeps name, telephone, hours, products |
| E3 | Open-leg captures | `captures/open_OPEN_*_{p1,p5}.{json,txt}` | OpenRouter battery | zero preview host strings; sparse prod PJ |
| E4 | OpenAI open P1 | `open_OPEN_openai_p1` | OpenRouter | names Print Junkie AZ Tempe + printjunkieaz.com |
| E5 | Forced matrix strict score | `captures/analysis_strict.json` + per-cell json | rescore script | OpenAI/Gemini o3 all variants |
| E6 | Pooled + paired deltas | analysis_strict | computed | V1 impact 0; V2/V3 0.2; paired Perplexity-driven |
| E7 | OpenAI forced V2 P5 | `forced_V2_openai_p5.txt` | OpenRouter | $75 + phone + Print Junkie despite ucp 404 |
| E8 | Claude forced samples | `forced_V0_claude_p5.txt` etc. | OpenRouter | “I couldn't open the staging site…” |
| E9 | Scope / domain | host decision t_f1118fbb · this task body | kanban | preview `*.vercel.app` ≠ prod apex |
| E10 | Operator autopsy note | kanban comment on t_6e47f5ab / t_f1118fbb | mid-run OOB | URL-forced primary; open flat = compatible |
| E11 | Runner + raw cells | workspace `t_6e47f5ab/` | local | 50 json/txt captures; cost ≈ $2.69 |

---

## gaps

1. **V3 under-ablated** relative to protocol “strip all LocalBusiness/Restaurant schema” — only PostalAddress removed. A true identity wipe may still matter.
2. **V4 (hide visible $), V5/V6 (bot block)** not deployed / not run.
3. **Claude tool path** cannot fetch vercel previews → gap for Anthropic-class forced ablation (need browser-capable route or different model/tooling).
4. **Perplexity body-fetch** unreliable on these hosts → high noise on the only engine showing V2/V3 deltas.
5. **No prod-apex single-variable** ablation — cannot claim discovery/citation causation on printjunkieaz.com.
6. **Small N** (2 prompts × 5 engines); single run; no multi-day repeat.
7. **Open Gemini P1** required max_tokens clamp after credit pressure; still completed.
8. OpenRouter **in-flight / credit** 402s mid-run caused Gemini retries; final matrix is complete but latency uneven.
9. Brand confusion: previews still say Print Junkie in content but some engines say “Tricon Digital” from hostname/meta — confounds named scoring slightly (strict scorer requires “Print Junkie”).

---

## confidence

```json
{
  "level": "medium",
  "rationale": "Forced-leg OpenAI/Gemini invariance across V0–V3 is high-confidence and replicated across P1+P5. Pooled V2/V3 impact is low-confidence (small, Perplexity-dominated). Open-discovery non-retrieval of previews is high-confidence but intentionally non-ranking. Preview/domain shift blocks any prod causal claim. V3 incomplete vs protocol ideal."
}
```

---

## recommended_next_actions

1. **Do not treat this battery as “drop llms/UCP from prod.”** It shows they are unnecessary for *forced* offer extraction when HTML already carries price+NAP — not that they fail at *discovery* on a real local domain.
2. **Ship a true V3b** (strip all LocalBusiness + Product JSON-LD; keep visible HTML) and re-run forced OpenAI/Gemini P1/P5 only — cheap causal check on schema.
3. **V4 visible-$ hide** on preview: forced P5 only — isolates HTML price chips vs schema/UCP machine prices (OpenAI/Gemini are the right instruments).
4. **Prod-domain path** (Cloudflare worker or staging subdomain on `printjunkieaz.com`) required before any discovery-rank claim; keep autopsy retrieval work on NAP/place graph separate.
5. **Skip Claude Haiku+native-web** for vercel forced tests until a browser/fetch-capable path exists; or pin a Claude model with stronger URL fetch.
6. **Optional:** repeat forced OpenAI/Gemini-only matrix 2× on another day for stability (cost ~$1–2).
7. Archive note for Julio/Huly only if Frank wants durable issue trail — kanban + know path is the live handoff.

---

## appendix

| Field | Value |
|-------|-------|
| task_id | t_6e47f5ab |
| report | `/home/ice/know/research/pj-ablation-battery-v0-v3-20261030.md` |
| captures | `/home/ice/.hermes/kanban/boards/research/workspaces/t_6e47f5ab/captures/` |
| cells | 50/50 ok |
| cost_usd_approx | 2.69 |
| primary_leg | URL-forced |
| ranking_headline | V1 impact 0.00; V2≈V3 impact 0.20 (low conf); OpenAI/Gemini invariant o3 |
| causation_label | **preview-only — no prod causation** |
