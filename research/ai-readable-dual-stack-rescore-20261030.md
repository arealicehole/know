# AI-readable dual-stack rescore — PJ + Maza vs local competitors (2026-10-30)

**Task:** `t_4237e704` (board: research, prefix [GIU])  
**Parent wave:** Wave 1C after citation battery `t_d08327bc` + miss autopsy `t_f1118fbb`  
**Baseline audit:** `research/ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md` (2026-10-28)  
**Absorbed inputs:** Tier-1 battery `t_d08327bc`, PJ miss autopsy `pj-chat-discovery-miss-autopsy-20261030.md`, Wave 2A ablation `pj-ablation-battery-v0-v3-20261030.md` (`t_6e47f5ab` DONE)  
**Prepared:** 2026-10-30 (all competitor/reference HTTP probes live this date)  
**Prepared by:** Geraldo v2.2  

**Doctrine labels (hard):**
- Dual-stack = **offer reliability when retrieved**, not proven **citation cause**.
- Wave 2A ablation is **preview-only** (`*.vercel.app`) — **no prod causation**.
- Street address on PJ remains **deferred by owner** (not an oversight gap).
- Do **not** rewrite SEO Bible from this scorecard alone.
- Optional new chat-battery cells **not re-run** this card (cost); cite Wave 1A/2A instead.

---

## objective

Re-score Print Junkie AZ and Maza Mediterranean (now dual-stack) against the same Phoenix/Chandler competitor set as the Oct 28 AI-readable audit, using the same binary signal rubric plus an explicit **dual-axis** split (**machine/offer** vs **place/authority**). Record movement since Oct 28, map remaining gaps to retrieval vs parse layers, and keep stack claims causal-safe given Wave 2A.

## summary

**Both references still lead the local field on combined AI-readability**, and the lead widened on the offer axis:

| Site | Oct 28 legacy | 2026-10-30 `/11` | Machine/offer `/6` | Place/authority `/6` | One-line |
|------|---------------|------------------|--------------------|----------------------|----------|
| **Maza** | **9/10** | **10/11** | **4/6** | **6/6** | Identity apex intact; **UCP upgraded** from discovery stub → **ucp/1.0 with 63 priced products** (shipped). Still **0 visible `$` in HTML body**. |
| **PJ** | **8/10** | **9/11** | **6/6** | **4/6** | Offer apex complete (`llms.txt` shipped vs 404 on Oct 28; UCP+specs+`$`+Offer schema). Place axis still capped: **no street** (owner-deferred), **no aggregateRating**. |
| Best non-ref | César 6 | **PHX Screen / AZ Hot Tees 7** | platform UCP/llms creep | PHX Screen **6/6 place** | Locals are catching **identity + platform agent files**, still not PJ-style SKU UCP. |

**Dual-axis verdict (stable):**
1. **Machine/offer surface** (bots allow, `llms.txt`, UCP, `/api/specs`, visible `$`, Offer/ItemList schema) → **PJ wins 6/6**. Maza 4/6 (no HTML `$`, no Offer JSON-LD) despite full UCP prices in `.well-known/ucp`.
2. **Place/authority surface** (typed schema, street NAP, hours, rating, sameAs, knowsAbout) → **Maza wins 6/6**. PJ 4/6. Several print competitors (PHX Screen, AZ Precision, Extreme, AG location) match or beat PJ on **place** alone while losing **offer**.

**Wave 2A absorption (preview, forced-URL):** OpenAI/Gemini extract brand+phone+$75 on **all** V0–V3 → once a URL is retrieved, **HTML price chips + core NAP** carry offer facts; **llms.txt removal impact 0.00**; UCP/identity-thin pooled impact **0.20** (low-confidence, Perplexity noise). Open-discovery never hit Vercel previews → **cannot** score discovery levers from that battery. Compete path remains **intent landings + GBP/entity pack** for retrieval; dual-stack remains **offer insurance when cited**.

**Competitor movement (material):**
- **Maza UCP:** stub (847 B) → **full ucp/1.0**, 63 products w/ tiers, checkout/fulfillment notes, `lastUpdated` 2026-10-30 — closes the Oct 28 “roadmap not shipped” gap on the wire.
- **PJ `llms.txt`:** 404 → **200** shop-facts file (still states street unpublished).
- **SpeedPro `llms.txt`:** was 200 → **404 now**; still **blocks GPTBot + Google-Extended** while allowing OAI/Perplexity/Claude-Search.
- **PHX Screen Printing:** shipped hand-authored **shop-facts `llms.txt`** → place-complete local (7/11).
- **Shirt52:** shop-facts `llms.txt` + richer schema → 3→6 on `/11` scale.
- **AZ Hot Tees / PHX Sign Shop:** previously 429/unverified → reachable; **Shopify platform `llms.txt` (Agent Instructions) + shopping UCP** (not shop-NAP facts). Score 7 and 6 but **platform_mcp** kind ≠ Maza/PJ hand facts.
- **César Graphics:** still has Wix-style `llms` + shopping UCP stub; **visible dollar catalog not found on home/printing paths this run** (Oct 28 had rich `$` body — possible JS/Wix render or page change; scored `$=0` this run).
- **Restaurant lane (Spitz, Greeks):** still ~0 server-readable schema (Greeks ~JS shell).
- **AlphaGraphics:** location page strong place signals; **brand root `llms.txt` ~43 KB franchise NAP directory** (265 locations) — authority density weapon, weak machine commerce.

---

## plan

```yaml
objective: Dual-stack rescore PJ+Maza vs Oct28 competitor set; dual-axis matrix; movement; causal-safe labels
success_criteria:
  - Live HTTP probes for refs + prior competitor URLs
  - Same 11 binary signals as t_64b6e1c1 (+ dual-axis rollup)
  - Before/after deltas; gaps mapped retrieval vs parse
  - Ablation doctrine absorbed; no Bible rewrite claim
  - Publish /home/ice/know/research/ai-readable-dual-stack-rescore-20261030.md
assumptions:
  - Rubric bits are presence/parseability, not SERP rank
  - Franchise root llms counts for AG brand row (noted)
  - Owner-deferred street ≠ “forgot to ship”
  - No new OpenRouter battery on this card
tasks:
  - T1: load Oct28 rubric + autopsy + ablation
  - T2: live probe matrix
  - T3: score + dual-axis + deltas
  - T4: Final Report Contract publish + kanban complete
```

---

## method

### Rubric (reproducible — same signals as Oct 28 §2.1)

Binary **1/0** each (sum = **score /11**). Oct 28 labeled “Max 10” while listing these 11 signals; this rescore uses honest **/11** and maps legacy rows by signal identity, not by renaming the denominator.

| # | Signal | Counts 1 when |
|---|--------|----------------|
| 1 | robots AI whitelist | Explicit Allow for GPTBot / OAI-SearchBot / Google-Extended (or peer) **without** Disallow `/` on GPTBot+Google-Extended |
| 2 | `llms.txt` | HTTP 200 body that is not an HTML 404 shell |
| 3 | UCP | `/.well-known/ucp` JSON 200 |
| 4 | `/api/specs` | JSON specs at `/api/specs` or per-SKU specs pattern (PJ) |
| 5 | typed schema | Business `@type` (LocalBusiness, Restaurant, Store, ProfessionalService, WholesaleStore, …) |
| 6 | street address | `streetAddress` in JSON-LD |
| 7 | hours | `openingHours` / `openingHoursSpecification` |
| 8 | aggregateRating | present |
| 9 | sameAs ≥ 1 | present |
| 10 | knowsAbout | present |
| 11 | visible `$` prices | ≥1 meaningful `$NN` in stripped homepage (or primary cards landing for PJ) |

### Dual-axis rollup (operator-required)

| Axis | Signals | Max |
|------|---------|-----|
| **Machine / offer** | robots AI + llms + UCP + specs + visible `$` + **bonus** Offer/AggregateOffer/ItemList/Product in JSON-LD | **/6** |
| **Place / authority** | typed schema + street + hours + aggregateRating + sameAs + knowsAbout | **/6** |

**Interpretation rule:** high offer + low place ⇒ parse/checkout-ready when found, weak pin/“best of” copy. High place + low offer ⇒ engines can name/locate but cannot reliably quote SKUs/checkout from first party.

### Probe set (2026-10-30)

References: `printjunkieaz.com`, `mazahalalfood.com`  
Prior matrix: César, AlphaGraphics PHX (us011), PHX Screen, AZ Precision, Extreme Print Lab, Shirt52, Mesa Sign, SpeedPro Phoenix, PHX Sign Co, Spitz Chandler, Greeks Grill, AZ Hot Tees, PHX Sign Shop  
Autopsy extras: Mousegraphics; AG brand `llms.txt` / `robots.txt` at root  

Workspace captures (ephemeral): `/home/ice/.hermes/kanban/boards/research/workspaces/t_4237e704/probes/`  
Durable archive: this file only.

---

## findings

### 1) Dual-axis leaderboard (live 2026-10-30)

| Rank | Site | Lane | `/11` | Offer `/6` | Place `/6` | robots | llms | UCP | specs | schema | street | hours | rating | sameAs | knows | `$` |
|-----:|------|------|------:|-----------:|-----------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | **Maza Mediterranean** | restaurant | **10** | 4 | **6** | Y | shop | **full 63** | Y | Restaurant | **Y** | Y | **Y** | 9 | Y | N |
| 2 | **Print Junkie AZ** | print | **9** | **6** | 4 | Y | shop | **full 8** | Y | LocalBusiness | **N** | Y | N | 3 | Y | **Y** |
| 3 | AZ Hot Tees | apparel | 7 | 3 | 5 | N | platform | shopify | N | LocalBusiness | Y | Y | N | 12 | Y | N |
| 3 | PHX Screen Printing | shirts/print | 7 | 2 | **6** | N | **shop** | N | N | LocalBusiness+ | Y | Y | Y | 4 | Y | N |
| 5 | Shirt52 | apparel | 6 | 3 | 4 | N | shop | N | N | LocalBusiness | Y | Y | N | 5 | N | min |
| 5 | PHX Sign Shop | signs/apparel | 6 | 2 | 4 | N | platform | shopify | N | WholesaleStore | Y | Y | N | ≥1 | N | N |
| 5 | AlphaGraphics PHX | print franchise | 6 | 1 | 5 | N* | **franchise dir** | N | N | LocalBusiness | **Y** | Y | **Y** | 6 | N | N |
| 5 | AZ Precision | apparel | 6 | 1 | **6** | N | N | N | N | LocalBusiness | Y | Y | Y | 4 | Y | N |
| 9 | Extreme Print Lab | DTF | 5 | 1 | 5 | N | N | N | N | LocalBusiness | Y | Y | Y | 4 | N | N |
| 10 | César Graphics | print | 4 | 2 | 2 | N | shop/wix | wix cart | N | LocalBusiness | Y | N | N | 0 | N | **N†** |
| 10 | Mousegraphics | print | 4 | 0 | 4 | N | N | N | N | LocalBusiness | **Y** | Y | N | ≥1 | N | N |
| 12 | Mesa Sign Co | signage | 3 | 0 | 3 | N‡ | N‡ | N‡ | N | LocalBusiness | Y | Y | N | 0 | N | N |
| 12 | SpeedPro Phoenix | signage | 3 | 0 | 3 | **anti** | **gone** | N | N | ProfessionalService | N | Y | N | 1 | N | N |
| 14 | PHX Sign Company | signage | 0 | 0 | 0 | — | N | N | N | none | N | N | N | 0 | N | N |
| 14 | Spitz Chandler | restaurant | 0 | 0 | 0 | — | N | N | N | none | N | N | N | 0 | N | N |
| 14 | Greeks Grill | restaurant | 0 | 0 | 0 | — | N | N | N | none | N | N | N | 0 | N | N |

\*AG location `robots.txt` 404; brand root robots = generic `/*_api/` disallow, no AI UA lines.  
†César: Oct 28 captured rich on-page `$` catalog; this run’s stripped home + `/printing/business-cards` showed **no `$NN` strings** (likely client-rendered Wix). UCP/llms still present.  
‡Mesa Sign: homepage 200; `robots`/`llms`/`ucp`/`sitemap` returned **Cloudflare 522** this run — secondary paths degraded.

**llms kind legend:** `shop` = hand NAP/facts (PJ/Maza/PHX Screen/Shirt52); `franchise dir` = AG multi-location dump; `platform` = Shopify/Wix “Agent Instructions” / MCP shopping guide (not local NAP card).

### 2) Reference deep dive — what changed since Oct 28

#### Print Junkie AZ

| Signal | Oct 28 | Oct 30 | Notes |
|--------|--------|--------|-------|
| robots AI | GPT+OAI+GExt+ChatGPT | **same** | unchanged |
| llms.txt | **404** | **200 ~1.8 KB shop facts** | Ships Facts + Core products + Agent endpoints + Links |
| UCP | ucp/1.0, 8 products | **same class** (~5 KB) | checkout stripe payment_link; address city/ZIP only |
| specs | per-product | **yes** (`/api/specs/business-cards` 200) | root `/api/specs` may 404; pattern intact |
| schema | LocalBusiness | + **Offer / AggregateOffer / ItemList / Product** | offer-schema bonus |
| street | null | **still absent** | llms: “full street address not published on-site” — **owner-deferred** |
| hours | Y | Y | Mon–Sat 09–17 |
| rating | N | N | still missing |
| sameAs | 2 | **3** | + Yahoo Local |
| knowsAbout | Y | Y | |
| visible `$` | Y on landings | **Y** home chips $65/$200/$125/$75/$149; cards page $65/$75/$125/$150 | |
| sitemap | 17 | **17** | still no city URL farm |

**Scores:** offer **6/6** (ceiling) · place **4/6** · total **9/11** (+1 vs Oct 28 via llms).

#### Maza Mediterranean

| Signal | Oct 28 | Oct 30 | Notes |
|--------|--------|--------|-------|
| robots AI | GPT+OAI+GExt | **same** (still no ChatGPT-User line) | minor |
| llms.txt | 200 shop facts | **200** | full street + phone + hours + delivery links |
| UCP | **847 B discovery stub** | **~37 KB ucp/1.0**, `productCount: 63`, tiers/prices, capabilities checkout/fulfillment/menu_discovery, endpoints, `lastUpdated` **2026-10-30T07:57Z** | **largest ship delta** |
| specs | 63 items | **63 items** | `pricing: true` utility still |
| schema | Restaurant+rating+menu | **same class** | still no Offer JSON-LD |
| street | Y | Y | 3491 W Frye Rd Suite 2 |
| visible `$` in HTML | 0 | **0** | prices live in UCP/specs JSON, not body chips |
| sitemap | 51 | **53** | local-intent farm intact |

**Scores:** offer **4/6** · place **6/6** · total **10/11**. Offer gap = HTML `$` + Offer schema (UCP now supplies machine prices).

### 3) Dual-axis reading (Bible-safe)

```
                 place/authority →
            low                         high
          ┌───────────────────────────┬───────────────────────────┐
 offer  hi│ PJ (6,4)                  │ *open cell: none local*   │
    ↓     │ offer insurance when hit  │ ideal dual-stack apex     │
          ├───────────────────────────┼───────────────────────────┤
        lo│ signage zeros, Greeks     │ Maza (4,6); PHX Screen;   │
          │ Spitz; thin shells        │ AZ Precision; AG place    │
          └───────────────────────────┴───────────────────────────┘
```

- **No local competitor occupies high-offer + high-place** except the two references approaching from opposite corners.
- **PJ’s remaining scoreboard debt is almost entirely place/authority** (street deferred, rating, denser sameAs/Maps/Yelp, URL farm) — matching miss-autopsy retrieval thesis, not missing UCP.
- **Maza’s remaining offer debt** is HTML-visible prices + Offer JSON-LD (UCP product tiers now exist).
- **Ablation alignment:** forced retrieval did not need llms/UCP to read `$75`/phone from HTML — scorecard must not promote llms/UCP as discovery causes. They remain **manifest + agent-endpoint insurance** and Maza-style identity packaging.

### 4) Competitor movement detail

| Competitor | Oct 28 | Now | Movement |
|------------|--------|-----|----------|
| Maza | 9 | **10/11** | UCP full product manifest shipped |
| PJ | 8 | **9/11** | llms.txt shipped; Offer schema richer |
| PHX Screen | 5 | **7/11** | **New shop-facts llms.txt**; strong place |
| Shirt52 | 3 | **6/11** | llms.txt + schema/sameAs growth |
| AZ Hot Tees | unverified/429 | **7/11** | Reachable; Shopify agent llms + UCP |
| PHX Sign Shop | unverified/429 | **6/11** | Same platform pattern |
| AlphaGraphics | 4 | **6/11** | Credit brand llms directory; location rating/street strong |
| AZ Precision | 5 | **6/11** | Place-complete; no agent files |
| Extreme | 4 | **5/11** | Slight schema richness |
| SpeedPro | 5 | **3/11** | **llms.txt disappeared**; still blocks GPTBot/GExt |
| César | 6 | **4/11** | llms+UCP remain; **$ catalog not observable** this probe |
| Mesa Sign | 4 | **3/11** | 522 on secondary endpoints |
| Mouse | (autopsy only) | **4/11** | Street NAP, no agent stack |
| Spitz / Greeks / PHX Sign Co | 2/1/1 | **0/11** | Still non-readable shells / empty schema |

**Platform creep finding:** Shopify “Agent Instructions” `llms.txt` + `dev.ucp.shopping` manifests are spreading among apparel/sign shops (AZ Hot Tees, PHX Sign Shop). These raise binary llms/UCP bits but are **not** equivalent to hand-authored local Facts cards (Maza/PJ/PHX Screen). Scoreboard inflation risk — dual-axis + `llms_kind` tag required when reading the table.

### 5) Gaps → retrieval vs parse (PJ focus)

| Gap | Axis | Blocks | Evidence |
|-----|------|--------|----------|
| No `streetAddress` | place | Pin-style “good place near me” copy | schema + llms explicit unpublished; owner-deferred |
| No `aggregateRating` | place | “Best / top-rated” language | schema absent; Maza/AG/PHX Screen have it |
| Thin Maps/Yelp `sameAs` | place | Entity resolution / citation graphs | 3 sameAs (IG/FB/Yahoo) vs Maza 9 |
| No city/service URL farm | place/retrieval | Intent coverage P2/P7 style | sitemap 17 vs Maza 53 |
| Directory/GBP density | place/retrieval | Perplexity/Grok P1 misses | autopsy citation graphs — not re-probed as battery here |
| llms/UCP missing | offer | **Mostly closed** on PJ | both 200; ablation: not load-bearing once URL forced |
| Visible `$` / Offer schema | offer | **Closed** on PJ | HTML chips + ItemList/AggregateOffer |
| Phone never in Tier-1 answers | packaging | Soft | present on site; engines prefer street+price (Wave 1A) |

**Maza gaps:** HTML `$` chips; Offer JSON-LD; optional ChatGPT-User robots line; first-party checkout still external (SkyTab/Slice) — UCP documents that honestly.

### 6) Citation / ablation cross-read (no new cells)

| Source | What it supports | What it does **not** support |
|--------|------------------|------------------------------|
| Wave 1A Tier-1 `t_d08327bc` | Maza discovery strong (P3 5/5); PJ P1 3/5 specialist slot; P2/P7 0/5 specialty lanes; edge 24/24=200 | Stack caused citations |
| Miss autopsy | Chains win place/NAP/directory density; PJ offer usable when retrieved | llms/UCP as P1 root cause |
| Wave 2A ablation | Offer extraction survives −llms and −UCP on forced preview URLs (OAI/Gemini o3 stable) | Prod discovery lever ranking; Bible rewrite |

**Optional spot battery:** **skipped** on this card (ablation + Tier-1 already burned budget the same day). Re-run only if prod-domain ablation or consumer-app legs are scoped separately.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | PJ machine/offer axis is at ceiling **6/6** on this rubric (robots, llms, UCP, specs, visible `$`, Offer schema). | E1 E2 E3 | high | supported |
| C2 | PJ place/authority remains **4/6** — missing street + aggregateRating (street owner-deferred). | E1 E4 E5 | high | supported |
| C3 | Maza place/authority is **6/6**; still the local identity reference. | E6 E7 | high | supported |
| C4 | Maza UCP is no longer a stub: live **ucp/1.0** with **63** priced products as of probe time 2026-10-30. | E8 | high | supported |
| C5 | Maza still has **0** visible `$` strings in homepage stripped text; offer axis **4/6**. | E6 | high | supported |
| C6 | Combined `/11`, Maza **10** and PJ **9** remain #1/#2 vs all probed locals. | E9 | high | supported |
| C7 | Several competitors improved agent files (PHX Screen shop llms; Shirt52 llms; Shopify platform llms/UCP on AZ Hot Tees & PHX Sign Shop). | E10 E11 | high | supported |
| C8 | SpeedPro lost `llms.txt` (404) and still blocks GPTBot/Google-Extended — score down. | E12 | high | supported |
| C9 | Dual-stack should be labeled **offer reliability when retrieved**, not citation cause — consistent with Wave 2A forced-leg null llms impact and open-discovery non-retrieval of previews. | E13 E14 | high | supported |
| C10 | No probed local competitor matches PJ’s SKU-tier UCP + visible price + specs pattern. | E9 E15 | high | supported |
| C11 | High place + low offer competitors (PHX Screen, AZ Precision, AG) can win narrative/NAP without UCP — autopsy-compatible. | E9 E14 | high | supported |
| C12 | César on-page dollar catalog was **not** reproducible on this run’s HTML extract (delta vs Oct 28 may be render path). | E16 | medium | partial |

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / note |
|-------------|--------------|------------|------------|----------------|
| E1 | PJ home + signals | https://printjunkieaz.com/ | live HTTP 2026-10-30; `probes/PJ_home.txt` | LocalBusiness; Offer/ItemList; dollars $65/$75/…; no streetAddress |
| E2 | PJ llms.txt | https://printjunkieaz.com/llms.txt | 200, 1801 B | “full street address not published on-site”; agent endpoints UCP+/api/specs |
| E3 | PJ UCP | https://printjunkieaz.com/.well-known/ucp | 200 ~4982 B | protocol ucp/1.0; 8 products; stripe payment_link; address city/ZIP only |
| E4 | PJ cards landing | https://printjunkieaz.com/business-cards-phoenix | 200 | $65/$75/$125/$150 chips |
| E5 | Owner street deferral | operator note on t_4237e704 + llms line | kanban mid-run + E2 | Not scored as ship defect |
| E6 | Maza home | https://mazahalalfood.com/ | 200 | Restaurant + AggregateRating + street + 9 sameAs; 0 `$` in stripped text |
| E7 | Maza llms.txt | https://mazahalalfood.com/llms.txt | 200, 1562 B | Full NAP + hours + delivery links |
| E8 | Maza UCP | https://mazahalalfood.com/.well-known/ucp | 200 ~36880 B | productCount 63; sample gyro $10.91; lastUpdated 2026-10-30T07:57:01Z |
| E9 | Full rescore matrix | workspace `probes/rescore_matrix.json` | derived from live probes | Leaderboard §1 |
| E10 | PHX Screen llms | https://phxscreenprinting.com/llms.txt | 200 shop-facts | Custom screen printing Phoenix/Glendale card |
| E11 | AZ Hot Tees llms/UCP | https://azhottees.com/llms.txt + /.well-known/ucp | 200 platform | “Agent Instructions — AZ Hot Tees”; Shopify UCP shopping |
| E12 | SpeedPro robots + llms | https://www.speedpro.com/phoenix/robots.txt ; …/llms.txt | robots 200; llms **404** | GPTBot Disallow /phoenix/; Google-Extended Disallow |
| E13 | Wave 2A ablation report | `research/pj-ablation-battery-v0-v3-20261030.md` | t_6e47f5ab | V1 impact 0.00; V2/V3 0.20; OAI/Gemini o3 all variants |
| E14 | Miss autopsy | `research/pj-chat-discovery-miss-autopsy-20261030.md` | t_f1118fbb | Retrieval = place graph; offer ≠ default-pick |
| E15 | Competitor UCP spot checks | César/AZHotTees/PHXSignShop UCP bodies | live | Platform cart schemas; not 8-SKU print tiers |
| E16 | César price extract | cesargraphics.com home + /printing/business-cards | live strip | No `$NN` matches this run |
| E17 | Oct 28 baseline | `research/ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md` | t_64b6e1c1 | PJ 8 / Maza 9 legacy table |
| E18 | Tier-1 battery | `research/citation-battery-tier1-p1-p7-expansion-t_d08327bc.md` | t_d08327bc | P1 3/5 PJ; P3 5/5 Maza; edge open |

## gaps

1. **Consumer-app** citation legs still out of scope (API-proxy only in Wave 1A/2A).
2. **Prod-domain ablation** (same apex, remove llms/UCP on printjunkieaz.com) unrun — preview battery cannot rank discovery.
3. **César price surface** needs browser/JS render pass before claiming permanent loss of `$` catalog.
4. **Mesa Sign 522** / intermittent edges — treat secondary path health as soft.
5. **Franchise scoring:** AG root llms credited on brand row; per-location path 404 — document ambiguity.
6. **Binary rubric inflation** from platform MCP llms — keep `llms_kind` in any Bible table.
7. **No new P1/P5 spot battery** on this card — freshness depends on earlier same-day runs.
8. **Street publish decision** is product/legal, not SEO checkbox.

## confidence

```yaml
level: high
rationale: >
  Reference and competitor HTTP probes collected live 2026-10-30 with saved bodies;
  dual-axis math is deterministic from binary signals; Maza UCP productCount and PJ llms
  are direct file reads. Medium only on César $ disappearance (render path) and on any
  implication about ranking/citations (explicitly not claimed; deferred to batteries).
```

## recommended_next_actions

1. **Scorecard hygiene:** Publish dual-axis (offer `/6`, place `/6`) + `/11` in Bible annex; tag `llms_kind`; label stack “offer reliability when retrieved.”
2. **PJ maximize lane (retrieval):** intent landings + GBP/Yelp/Maps sameAs + review velocity — **not** more UCP fields. Accept street deferred until owner publishes.
3. **PJ place quick wins without street:** `aggregateRating` when eligible; Google Business / Yelp `sameAs`; city/service URL farm (Maza-style).
4. **Maza offer polish:** optional HTML `$` chips or Offer JSON-LD mirroring UCP tiers; keep full UCP (now shipped).
5. **Watchlist:** PHX Screen (shop llms + place-complete), Shopify agent-llms shops, SpeedPro policy inconsistency, César price render.
6. **Research follow-ups (separate cards):** prod-domain ablation; consumer-app battery; browser pass on Wix price DOM — do not fold into Bible merge from this file alone.
7. **Non-goals:** Do not claim dual-stack caused Wave 1A hits; do not treat Vercel ablation as prod discovery proof.

---

## appendix

| Field | Value |
|-------|-------|
| objective | Dual-stack AI-readability rescore vs Oct 28 set |
| references | PJ offer 6/6 place 4/6 total 9/11; Maza offer 4/6 place 6/6 total 10/11 |
| top non-ref | AZ Hot Tees 7, PHX Screen 7 |
| biggest ship deltas | Maza full UCP 63 SKUs; PJ llms.txt; PHX Screen llms |
| stack doctrine | Offer insurance when retrieved — not citation cause |
| artifact | `/home/ice/know/research/ai-readable-dual-stack-rescore-20261030.md` |
