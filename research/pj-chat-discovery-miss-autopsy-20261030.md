# Print Junkie AZ — Chat-Discovery Miss Autopsy (P1 narrative gap)

**Task:** t_f1118fbb (board: research, prefix [GIU])  
**Wave input:** t_d08327bc — Tier-1 citation battery P1–P7 × 5 engines (2026-10-30)  
**Prior baselines:** t_528f7761 (open cells), t_49a81e49 (first battery), t_64b6e1c1 (AI-readable dual-stack audit), t_70d29518 (protocol SoR)  
**Process spine:** `bible-research-loop-process-20261030.md` R4  
**Prepared:** 2026-10-30  
**Prepared by:** Geraldo v2.2  
**Method:** Mine API-proxy miss/HIT transcripts + citation graphs from t_d08327bc captures; live HTTP probe competitor vs PJ dual-stack surfaces (same day); separate **machine-readable** gaps from **human-narrative / entity-authority** gaps.

**Hard caveats (operator + battery SoR):**
- API-proxy (OpenRouter native web) ≠ consumer ChatGPT/Gemini/Perplexity app UI.
- Single-shot per cell — Gemini×P1 flipped MISS→HIT vs t_528f7761; **do not claim permanence**.
- **Do not claim** dual-stack (llms/UCP/schema) *caused* the 3/5 P1 hits.
- Edge is open: **24/24 = HTTP 200** for AI bot UAs on PJ/Maza (t_d08327bc) — misses are **not robots blocks**.

---

## objective

Explain why franchise/chain and long-tenure local shops **win the P1 “good place for business cards in Phoenix/Tempe” narrative** on hard-miss engines (Perplexity, Grok) — and often still own “best overall” even when OpenAI/Claude/Gemini *do* name Print Junkie — while PJ is **machine-readable** (prices, UCP, llms.txt, bot allow) but is not the default pick. Separate actionable site/entity gaps from specialty-lane non-competes (P2/P7) and price-without-attribution (P5).

## summary

**Scoreboard to use (Wave 1A / t_d08327bc, obsolete prior “PJ 1/4 on P1”):**

| Lane | Score | Note |
|------|-------|------|
| PJ P1 cards | **3/5** | OpenAI / Claude / **Gemini HIT** (Gemini variance vs prior MISS) |
| PJ P5 price 1k | **1/5** | Only Perplexity names PJ + **$75**; others emit ~$75 bands **without name** |
| PJ P2 wraps | **0/5** | Specialty wrap shops only |
| PJ P7 signs | **0/5** | FASTSIGNS / Signarama lane |
| Perplexity×P1 / Grok×P1 | **MISS** | Stable hard misses across baselines |
| Edge | 24/24 = 200 | Not a robots problem |
| PJ phone in any Tier-1 answer | **0** | Never surfaced this run |

**One-line autopsy:** PJ already wins the **transaction/offer surface** (visible $65/$75, UCP product tiers, Stripe order links, agent endpoints in `llms.txt`). Chains and multi-decade locals win the **discovery/authority surface** that Perplexity/Grok (and OpenAI’s “best overall” slot) optimize for: **street-level NAP**, **multi-location / franchise page farms**, **directory+Yelp density**, **embedded ratings/tenure narratives**, and **dedicated geo-intent service URLs** that dominate the citation graph. PJ’s live schema still **omits `streetAddress`** (city/ZIP only), **omits `aggregateRating`**, and `llms.txt` itself states the street is unpublished — so even a fully readable offer stack cannot supply the pickup-pin identity engines copy from AlphaGraphics Camelback (4444 N 24th) or Mousegraphics (1414 W 14th).

**Machine-readable gap vs narrative gap:** Dual-stack is largely **shipped** for offers (and improved vs t_64b6e1c1: `llms.txt` now 200, `ItemList`/`AggregateOffer` present). Remaining first-party holes are **identity/authority** (street, rating, Maps/Yelp `sameAs`, review graph), not “can an agent parse price.” Competitor win on miss engines is **citation-graph + SERP entity density**, not superior UCP.

---

## plan (executed)

```yaml
objective: P1 miss autopsy — narrative vs machine-readable; live competitor probe
success_criteria:
  - Mine P1/P5/P2/P7 transcripts + citation domains from t_d08327bc
  - Live probe AG / Mouse / Cesar / Eyeconic / PJ dual-stack
  - Claims with evidence; no stack-causation overclaim
  - Publish know/research/pj-chat-discovery-miss-autopsy-20261030.md
assumptions:
  - Wave 1A scoreboard is SoR (P1 = 3/5)
  - API-proxy only
tasks:
  - T1: load battery report + captures
  - T2: citation-graph mine miss engines
  - T3: live competitor + PJ surface probe
  - T4: synthesize R4 autopsy + next actions
```

---

## findings

### 1) What “miss” actually means after Wave 1A

| Engine | P1 | Role of PJ when present | Default narrative winner |
|--------|----|-------------------------|---------------------------|
| **OpenAI** | **HIT** | Section **“Best for fast turnaround”** with $65/100, $75/1k + intent URL `?utm_source=openai` | **“Best overall local option: AlphaGraphics University Drive”** (full street 815 W University) |
| **Claude** | **HIT** | Named Tempe + 2–3 day / next-day / same-day pickup + link | Co-list with Eyeconic + AlphaGraphics |
| **Gemini** | **HIT** (delta vs prior MISS) | Named + ~$75/1k + domain in grounding | Multi-shop list led by **AlphaGraphics Camelback** vibe/rush/32pt story; Mousegraphics owns Tempe “community favorite since 1989” |
| **Perplexity** | **MISS** | Absent | **AlphaGraphics Camelback** + **CesarGraphics** + **UPS Store** location farm |
| **Grok** | **MISS** | Absent | **AlphaGraphics** multi-location lead + Outta PHX / Splash / JC + **Yelp search SERPs** |

**Interpretation:** On engines that retrieve PJ, PJ is a **specialist slot** (speed/value), not the **default recommendation**. On Perplexity/Grok, PJ never enters the retrieved set — the answer is assembled from franchise service pages, chain location pages, and directory SERPs.

### 2) Citation graphs (miss engines vs HIT)

#### Perplexity × P1 (MISS) — citation domain counts (from capture)

| Domain class | Count (approx) | Role in answer |
|--------------|----------------|----------------|
| `locations.theupsstore.com` | **10** | Neighborhood convenience graph |
| `alphagraphics.com` (local service URLs) | **3** | Lead “best all-around” + NW Valley alt |
| `cesargraphics.com` | **1** | Next-day + **$45** budget narrative |

**Answer pattern:** street addresses + rush hours + “best all-around = AlphaGraphics Camelback”; budget = Cesar $45 next-day. **Zero** `printjunkieaz.com`.

#### Grok × P1 (MISS)

- Lead claim: AlphaGraphics multiple PHX/Tempe locations + JC / Splash / Outta PHX.
- Citations include AG Tempe + Camelback **service URLs** and **Yelp search** pages (`find_desc=Business+Card+Printing&find_loc=Phoenix|Tempe`).
- Retrieval path is **franchise + UGC directory**, not first-party indie offer pages.

#### OpenAI × P1 (HIT) — still AG-led narrative

- Explicit ranking: AG University Drive = **best overall** (with street); PJ = **best for fast turnaround** (Tempe service area, **no street**); Mousegraphics = traditional design shop (**1414 W 14th**).
- Citations mix AG, PJ intent, Mouse home, Blend4, FedEx, Staples.

**Provenance path:** engines that “like” structured local answers copy **NAP triples from location pages**. AG and Mouse supply them; PJ supplies prices + phone + city but not a pin.

### 3) Live competitor vs PJ dual-stack (probed 2026-10-30)

| Signal | Print Junkie AZ | AlphaGraphics (Camelback / Tempe cards) | Mousegraphics | CesarGraphics | Eyeconic Prints (P5 rival) |
|--------|-----------------|------------------------------------------|---------------|---------------|---------------------------|
| robots AI allow explicit | GPTBot, OAI-SearchBot, Google-Extended, ChatGPT-User | Generic allow; no AI UA lines; blocks `/*_api/` | Generic | Wix default | (not fully matrixed) |
| `llms.txt` | **200** ~1.8 KB hand facts + agent endpoints | **200** **~43 KB franchise directory** incl. **14 AZ NAP lines** (Camelback, University Drive, Elliot, Chandler, etc.) | **404** | **200** Wix MCP agent doc (not shop facts) | — |
| `.well-known/ucp` | **200** product tiers + checkout/fulfillment | **404** | **404** | **200** Wix shopping UCP stub (cart/checkout schemas; not PJ-style SKU tiers) | — |
| Visible $ on cards page | **$65 / $75 / $125 / $150** | Sparse / marketing; some promo $29.95-class strings in HTML | **No $ catalog** | **Rich:** next-day **$45**, standard **$114**, foil/spot UV **$400**, etc. | **$55** 1-side / **$75** 2-side @ 1k |
| Schema street | **MISSING** (`PostalAddress` city Tempe + 85281 only) | **Present** (4444 N 24th / 815 W University) | **Present** (1414 W 14th) | **Present** (2610 W Baseline #112) | **Present** (1749 E McDowell) |
| Schema phone | **+1-602-730-2135** | Present on location pages | **(480) 894-1992** | Present | **602-688-4023** |
| `aggregateRating` | **No** | **Yes** (Organization + reviews on AG pages) | Google embed **5.0 / 27 reviews** on cards page | No in prior audit style | No on cards page types seen |
| `sameAs` / directories | IG, FB, Yahoo Local (3) | Franchise web + Maps ecosystem | Strong local/Maps/Yelp presence in SERP | Local | Local |
| Sitemap scale | **17** `<loc>` | AG root sitemap sample **511** `<loc>` (franchise-scale) | Multi-page WP | Large Wix | — |
| Tenure / narrative hooks on page | “since 2023” / volume claims on home | **Onsite 24–48h**, **32-point multi-layer**, designers on staff, “printed here not out of state” | **Family-owned since 1989**, “best in town” review quotes | Specialty finishes catalog | Family-owned since 2008 |

**PJ `llms.txt` smoking gun (live):**  
`Location: Tempe, AZ 85281 … (pickup shop; full street address not published on-site)`

That is an **intentional identity hole** relative to every competitor that wins P1 narrative.

**Delta vs t_64b6e1c1 (2026-10-28):** PJ has **shipped** `llms.txt`, richer home `ItemList`/`AggregateOffer`, and PostalAddress city/ZIP (still no street). Prior “no llms.txt / address null” is **partially obsolete** — remaining gap is **street + rating + entity graph**, not “zero identity file.”

### 4) Ranked autopsy themes (operator focus)

#### (1) Why chains win P1 narrative on Perplexity/Grok while OAI/Claude/Gemini can cite PJ

**Supported mechanisms (correlated, not causal proof):**

1. **Location-page density.** AlphaGraphics publishes per-city **business-cards** URLs with full street + phone + rush copy. Franchise `llms.txt` enumerates AZ stores as a NAP list. One brand → many retrievable entities for “Phoenix or Tempe.”
2. **Directory/SERP gravity.** Grok cites Yelp *search* URLs; Perplexity floods UPS Store location card URLs. Indie sites without strong Yelp/Maps/review graph lose when the engine’s search tool returns directory packs first.
3. **Copy-able narrative units.** AG: “24–48 hours or less,” “32-point,” “designers on staff,” “onsite not out of state.” Mouse: “since 1989,” Google 5.0 quotes. Cesar: next-day **$45**. PJ has strong price/turnaround copy **when retrieved**, but weaker multi-source corroboration.
4. **NAP completeness.** Engines paste **street** into “good place” answers. PJ cannot be pasted as a pin from first-party schema/llms; AG/Mouse/Cesar can.
5. **Even on HIT engines, ranking slot is secondary.** OpenAI’s own wording assigns **best overall → AG**, **fast → PJ**. Citation ≠ category win.

**Not the mechanism:** robots blocking (edge 200); missing UCP on competitors (they win *without* UCP); lack of `$` on PJ cards page (prices are visible).

#### (2) P5 ambient $75 without attribution — price page not uniquely owned

| Engine | Names PJ? | Price behavior |
|--------|-----------|----------------|
| OpenAI | N | Band ~$50–$80; cites Biltmore ~$52, Eyeconic $55/$75, NVST $75.99, etc. |
| Claude | N | ~$55–$80; Eyeconic $55 @ 1k dominant |
| Gemini | N | ~$55–$100; **$75 appears as market/Eyeconic double-sided** |
| **Perplexity** | **Y** | **“Print Junkie: $75 for 1,000”** + intent link — only clean attribution |
| Grok | N | ~$55–$150; Eyeconic-heavy |

**Live Eyeconic:** 1,000 cards **$55 one-sided / $75 double-sided** at `eyeconicprints.com/business-cards/` with McDowell street — **same $75 headline** as PJ’s most-popular tier.

**Implication:** `$75` is **ambient market language**, not a PJ-owned fact. Measurement rule from battery holds: **price-band answer ≠ merchant citation.** To own P5, PJ needs either unique price positioning, stronger “Print Junkie $75” co-occurrence off-site, or query-shaped content that forces entity+price binding (still no causation claim).

#### (3) P2 / P7 — intent coverage vs accept specialty non-compete

- **P2 (wraps/vehicle graphics):** 0/5. Citation domains = PRI Graphics, Custom Graphix, Limitless Graphix, Fast-Trac, Vinyl Vixen, PHX Screen, car-wrap specialists. These are **category pure-plays** with wrap-intent URLs. PJ UCP lists window-wraps SKUs, but discovery queries resolve to wrap shops.
- **P7 (signs/banners near Chandler):** 0/5. FASTSIGNS Chandler, Signarama Chandler, Sir Speedy, Print ’N Ship, AlphaGraphics Chandler, chamber listings. **Franchise sign density** near the geo term.

**Policy choice for Bible/roadmap (not decided here):**  
(A) invest in specialty intent pages + GBP categories + before/after proof to compete, or  
(B) **accept non-compete** on pure-play wrap/sign prompts and defend **generalist print + install** queries where dual-stack helps.  
Battery evidence supports “not a crawl bug”; it does **not** prescribe A vs B.

#### (4) PJ phone never surfaced this Tier-1 run

- Ground truth phone `+1-602-730-2135` is in schema, FAQ answers, `llms.txt`, UCP vendor block, and `tel:` CTAs.
- Across P1–P7 captures mined for this autopsy: **no engine quoted the PJ phone** in the scored Tier-1 set (battery report + capture sniffs).
- By contrast, Maza P3 often reproduces full NAP including phone — restaurant identity pattern.
- Print answers prefer **shop name + street + price/turnaround**; phone is secondary unless user asks “how do I call.”

**Gap type:** authority/NAP packaging (street missing amplifies “no place to put phone”), not absence of phone string on site.

#### (5) Do not claim stack caused P1 hits

- Gemini P1 was **MISS** on t_528f7761 and **HIT** on t_d08327bc with same model class — **variance**.
- Competitors without UCP/llms (Mouse) or with thin AI robots (AG) still dominate miss engines and “best overall.”
- Ablation (t_70d29518 §B) still unrun.
- Safe claim: dual-stack makes PJ **usable when retrieved** (accurate $65/$75, turnaround, order URL). Unsafe claim: dual-stack **causes** retrieval.

### 5) Machine-readable gaps vs human-narrative / authority gaps

| Layer | PJ status (live 2026-10-30) | Blocks default-pick? | Fix class |
|-------|-----------------------------|----------------------|-----------|
| Bot edge | Open (200) | No | — |
| Prices in HTML | Strong on cards landing | No for parse; weak for unique P5 ownership | Positioning / off-site co-occurrence |
| UCP + `/api/specs` | Strong | No (competitors win without it) | Keep; not the P1 miss driver |
| `llms.txt` | Present; **admits no street** | Partially — helps agents that fetch it; doesn’t create SERP entity | Add street when publishable; add Maps/Yelp links |
| Schema `streetAddress` | **Missing** | **Yes** for pin-style answers | Publish real pickup address in schema + pages |
| `aggregateRating` + review graph | Missing | **Yes** for “top-rated / best” language | GBP/Yelp/Google reviews → schema when eligible |
| `sameAs` Maps/Yelp | Thin (social + Yahoo) | **Yes** for entity resolution | Add Google Business / Yelp / Maps URLs |
| Multi-location / franchise page farm | Single shop, 17 URLs | **Yes** vs AG/UPS density | City/service URL farm (Maza-style) + citations |
| Directory/Yelp rank | Weak in miss-engine citation graphs | **Yes** on Perplexity/Grok | Local SEO / citations / reviews ops |
| Specialty wrap/sign intent URLs | Thin vs pure-plays | P2/P7 zeros | Intent pages or accept non-compete |
| Tenure narrative | Younger brand (2023+) | Soft — Mouse/Biltmore “since 198x” wins trust copy | Proof volume, installs, review velocity — not fake tenure |

**Dual-stack apex comparison (updated):**  
PJ remains the **offer/checkout reference** (UCP SKU tiers + Stripe links). Maza-style **identity apex** is still incomplete on PJ: street pin, rating, dense `sameAs`, local URL farm. AlphaGraphics is the **narrative/authority reference** for print P1 despite weaker machine commerce surface.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | Wave 1A P1 score for PJ is **3/5** (OpenAI, Claude, Gemini); Perplexity and Grok miss — prior “1/4” narrative obsolete for this snapshot. | E1, E2 | high | supported |
| C2 | Perplexity×P1 citation graph is dominated by UPS Store location pages + AlphaGraphics + Cesar; no PJ domain. | E3 | high | supported |
| C3 | Grok×P1 leads with AlphaGraphics multi-location and uses Yelp search SERPs; no PJ. | E4 | high | supported |
| C4 | Even OpenAI×P1 HIT assigns **best overall** to AlphaGraphics University Drive (with street) and slots PJ as **fast turnaround** with prices but no street pin. | E5 | high | supported |
| C5 | Live PJ schema has phone, hours, city/ZIP PostalAddress, offers, and `llms.txt`+UCP, but **no `streetAddress`**; `llms.txt` states street not published. | E6, E7 | high | supported |
| C6 | Live AlphaGraphics card pages expose street NAP, rush/32pt narrative, and schema rating patterns; franchise `llms.txt` lists many AZ locations with NAP — authority/page-farm advantage without UCP. | E8, E9 | high | supported |
| C7 | Mousegraphics wins Tempe “local favorite since 1989” narrative with street + Google review embed; no llms/UCP required. | E10, E11 | high | supported |
| C8 | P5 $75 is not PJ-unique; Eyeconic publishes $55/$75 at 1k; only Perplexity attributed $75 to PJ this run. | E12, E13, E1 | high | supported |
| C9 | P2 and P7 are 0/5 with specialty/franchise competitors; edge 200 ⇒ intent/SERP competition not crawl block. | E1, E14 | high | supported |
| C10 | PJ phone did not appear in Tier-1 answer text this run despite on-site presence. | E1, E15 | high | supported |
| C11 | Dual-stack cannot be claimed as the cause of P1 hits (single-shot variance; competitors win without stack; ablation unrun). | E1, E16 | high | supported |
| C12 | Primary remaining first-party gap for default-pick is **identity/authority** (street, ratings, directory graph, URL farm), not machine price parseability. | E6–E11, E3–E5 | medium-high | supported (synthesis) |

---

## evidence_index

| evidence_id | source_label | source_url / path | provenance | excerpt / support note |
|-------------|--------------|-------------------|------------|------------------------|
| E1 | Tier-1 battery report | `/home/ice/know/research/citation-battery-tier1-p1-p7-expansion-t_d08327bc.md` | know archive; API-proxy 2026-10-30 | Scoreboard P1 3/5, P5 1/5, P2/P7 0/5; edge 24/24; caveats |
| E2 | Operator Wave 1A note | kanban comment on t_f1118fbb | mid-run steering | Use 3/5; ranked autopsy focus; no stack-causation |
| E3 | Perplexity P1 capture | `.../t_d08327bc/captures/perplexity_p1.txt` | battery workspace | AG Camelback + Cesar + UPS; domains UPS×10, AG×3 |
| E4 | Grok P1 capture | `.../captures/grok_p1.txt` (+ json) | battery workspace | AG-led; Yelp search citations; no PJ |
| E5 | OpenAI P1 capture | `.../captures/openai_p1.txt` | battery workspace | Best overall AG University Dr street; PJ fast + $65/$75 + utm link |
| E6 | PJ home JSON-LD live | `https://printjunkieaz.com/` | HTTP 200 probe 2026-10-30 | PostalAddress without streetAddress; telephone present; no aggregateRating |
| E7 | PJ llms.txt live | `https://printjunkieaz.com/llms.txt` | HTTP 200 | Explicit “full street address not published on-site”; phone; agent endpoints |
| E8 | AG Camelback cards live | `https://www.alphagraphics.com/us-arizona-phoenix-us011/print-services/business-cards` | web_extract + HTTP | 4444 N 24th; 24–48h; 32-point; designers; onsite |
| E9 | AG llms.txt live | `https://www.alphagraphics.com/llms.txt` | HTTP 200 ~43KB | ≥14 Arizona location NAP lines incl. Tempe University + Elliot |
| E10 | Mouse cards live | `https://mousegraphics.com/page/business-cards` | web_extract | 1414 W 14th; (480) 894-1992; Google 5.0 review wall; since 1989 home SERP |
| E11 | Web SERP Tempe cards | web_search “business cards Phoenix Tempe AlphaGraphics Mousegraphics” | 2026-10-30 | Mouse home #1; AG Tempe cards #2; Mouse cards #3 — PJ not in top organic pack |
| E12 | Eyeconic cards live | `https://www.eyeconicprints.com/business-cards/` | HTTP 200 | $55 / $75 @ 1k; 1749 E McDowell |
| E13 | P5 captures | `.../captures/{openai,claude,gemini,perplexity,grok}_p5.txt` | battery | Only Perplexity names PJ+$75; others bands/Eyeconic |
| E14 | P2/P7 captures | `.../captures/*_p2.txt`, `*_p7.txt` | battery | Wrap pure-plays; FASTSIGNS/Signarama |
| E15 | Phone scan | capture text mine P1/P5 + battery notes | this task | No 602-730-2135 in answer bodies reviewed |
| E16 | Prior Gemini miss | `citation-battery-open-cells-delta-t_528f7761.md` | know | Gemini×P1 MISS AlphaGraphics-led — variance vs E1 HIT |
| E17 | Dual-stack audit | `ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md` | know 2026-10-28 | Historical split identity vs transaction; baseline before llms ship |
| E18 | Cesar cards + UCP/llms | cesargraphics.com probes | HTTP 2026-10-30 | Rich $ catalog; Wix MCP llms; shopping UCP stub |
| E19 | Sitemap scale | pj sitemap vs ag sitemap | HTTP | PJ 17 loc; AG sample 511 loc |

---

## gaps

1. **Consumer-app legs** still unrun — Perplexity/Grok app UI may differ from API-proxy.
2. **Multi-shot stability** (2–3×) owed before treating Gemini P1 HIT as durable.
3. **Ablation** (strip llms/UCP/schema) unrun — cannot quantify stack contribution.
4. **GBP/Yelp/Maps live pack ranking** for PJ vs AG/Mouse not fully scraped (Yelp SERP HTML noisy); citation graphs used as proxy.
5. **Whether street address can be published** is a business/privacy ops decision — technical gap is clear; publishability not confirmed here.
6. **P2/P7 strategy choice** (compete vs accept non-compete) needs owner decision.
7. **Off-site citation building** (chambers, sponsor pages, local blogs) not inventoried.
8. **Cesar Wix MCP/UCP** — present but unknown whether any engine used MCP path (unlikely in this battery); monitor as future agent surface.

---

## confidence

| level | rationale |
|-------|-----------|
| **high** on descriptive autopsy of Wave 1A misses/HITs and live first-party gaps (street, rating, citation domains) | Direct captures + HTTP probes same day |
| **medium** on “directory/franchise density *causes* Perplexity/Grok miss” | Strong correlation via citation graphs; no controlled retrieval ablation |
| **low** on any claim that shipping more schema alone flips Perplexity/Grok | Contradicted by competitors winning without UCP and by single-shot variance |

```json
{
  "level": "high",
  "rationale": "Transcript-level and live-probe evidence tightly support the machine-vs-narrative split and the street/NAP/directory mechanisms; causation of engine retrieval remains medium and explicitly caveated."
}
```

---

## recommended_next_actions

### P0 — identity pin (unblocks “place” answers)

1. **Publish pickup `streetAddress`** in JSON-LD, visible contact/about, GBP, and rewrite `llms.txt` line that currently admits absence.
2. Add **Google Maps + Yelp (and Apple Maps if listed)** to `sameAs`; align NAP everywhere.
3. Pursue **review velocity → `aggregateRating`** when eligible (don’t fake).

### P1 — authority / citation graph (Perplexity & Grok surface)

4. Local SEO ops: chamber, sponsor, “best of” lists, consistent categories on GBP (Printing Services / Banner / etc.).
5. Expand **city + service intent URL farm** (Tempe/Phoenix/Scottsdale business-cards, banners, wraps) — Maza pattern; today sitemap ≈ 17 URLs vs franchise hundreds.
6. Strengthen Yelp presence so Grok-style directory retrieval can land PJ entities, not only search SERPs of competitors.

### P1 — price attribution (P5)

7. Treat **Eyeconic $55/$75** as the ambient competitor; differentiate (stock/finish included, same-day pickup Tempe, install bundle) in on-page H2/FAQ so entity+price co-occur.
8. Keep measuring **named+$** separately from bare $75 bands (Bible metric hygiene).

### P2 — specialty lanes

9. Owner decision: **compete** (dedicated wrap/sign landing + photos + GBP secondary categories) vs **accept non-compete** on pure-play P2/P7 prompts.
10. If compete: mirror competitor URL shapes (`/vehicle-wraps-phoenix`, `/signs-chandler`) rather than only UCP SKUs.

### Measurement

11. Re-run **P1×Perplexity/Grok multi-shot** after street+Maps sameAs ship (still API-proxy labeled).
12. Do **not** mark R4 closed on stack ship alone; close when miss engines retrieve **name+street** or default-pick rate moves under multi-shot.
13. Optional: consumer-app spot checks for Perplexity/Grok P1 only (expensive/manual).

### Bible / process

14. Update R4 status in `bible-research-loop-process-20261030.md` from OPEN → **diagnosed** with pointer to this file.
15. Keep dual-stack as **offer reliability** control, not as the sole discovery lever.

---

## appendix (for operators)

```
P1 narrative miss ≠ unreadable site.
PJ = readable offers; AG/Mouse = pasteable places + denser citation graph.
Ship street + Maps/Yelp sameAs + reviews before expecting default-pick.
$75 is market ambient (Eyeconic); attribution ≠ band.
P2/P7 = category SERP, not robots.
Stack didn’t get a causation medal — don’t award one.
```

---

*End of report. Single durable archive path: this file under `/home/ice/know/research/`.*
