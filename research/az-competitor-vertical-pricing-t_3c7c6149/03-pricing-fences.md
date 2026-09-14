# 03 — Pricing fences (decision memo)

**Audience:** Frank (lock vs adjust in ~10 min) + Dick Jones (commercial architecture)  
**Date:** 2026-09-14  
**Draft anchors tested:** HR remote ~$175; on-site 1.25–1.5×; P1 1.5×; P0 2× · A-MO intro $400 → list $550 · A-SETUP ~$3k · B-MO 8h ~$2k · AB modest discount · P cost+5–10%

**Overall recommendation:** **LOCK the draft architecture with minor fences below.** Do not race AI-receptionist floors ($179) or MSP seat math. Defend **bundle uniqueness** (A+B+P), not lowest unit price.

---

## 1. Executive verdict (5 bullets)

1. **HR remote $175 is solid** — sits above Phoenix break-fix floor ($100) and inside MSP project bands ($150–$250); below fractional-CTO luxury ($200–$400).  
2. **A-SETUP ~$3k is competitive** — below AutomateNexus-class one-time builds ($7.5k published; $2.5k “from” marketing) and above free-setup AI receptionist plays; keep **$2.5k hard floor** for real agent systems.  
3. **A-MO $400 intro → $550 list is defensible** only when scoped as **managed agent keep-alive**, not “ChatGPT wrapper”; expect sticker shock vs $179–$500 pure receptionist SKUs—win on scope, not race to $179.  
4. **B-MO 8h ~$2k ($250/hr effective) is premium** vs Mobile Tech $100 and MBPS $150 T&M; **lock $2k as list** for multi-stack senior work, but allow **a documented 10–12h @ $2k–2.2k intro** only if needed—never price B as cheap break-fix.  
5. **P cost+5–10% is correct broker discipline**; do not use print margin to discount A/B.

---

## 2. HR — hourly remote / on-site / urgency

### Market anchors (cited)

| Anchor | Signal | Source conf. |
|--------|--------|--------------|
| Phoenix break-fix | **$100/hr** + **$20** travel; 1-hr on-site min | Mobile Tech Support rates — **high** |
| MSP project | **$150/hr** std · **$300/hr** after-hours | MBPS pricing — **high** |
| Break-fix band (provider blog) | **$150–$250/hr** | ITS / itsasap.com — **med** |
| Aggregator Phoenix | Project hourly **$150–$250** | itreviews.co 2026 guide — **med** |
| On-demand MSP hours | **$65/hr** (outlier low) | AZ MSP UNLIMITED — **high** as published, **low** as quality peer |
| Fractional CTO hourly | **$200–$400/hr** | ctoondemand.com 2026 — **med** (different role) |

### Draft vs recommendation

| SKU | Draft | Recommendation | Rationale |
|-----|-------|----------------|-----------|
| **HR-Remote** | ~$175 | **LOCK $175 list** | Above $100 floor; aligned $150 MSP T&M; room under CTO rates |
| **HR-On-site (local AZ)** | 1.25–1.5× | **LOCK 1.35× default** ($236); allow 1.25× only for multi-hour scheduled blocks | Covers travel friction beyond $20 flat fee model |
| **P1 (priority / after-hours-ish)** | 1.5× | **LOCK 1.5×** remote base | MBPS after-hours is **2×** ($300/$150); 1.5× still fair |
| **P0 (emergency)** | 2× | **LOCK 2×** | Matches MBPS after-hours multiple |

### Do not go below (HR)

| Line | Floor | Why |
|------|-------|-----|
| Remote standard | **$150/hr** | Below this you are pricing like generic MSP T&M without their seat revenue |
| On-site local | **$190/hr** (~1.25×$150) | Protects against becoming Mobile Tech at white-label rates |
| Never match | **$65/hr** on-demand MSP | Race-to-bottom; quality signal collapse |
| Emergency P0 | **never discount below 1.75×** in writing | After-hours is scarce inventory |

**Confidence:** **high** on remote $175 lock; **med-high** on multipliers.

---

## 3. A — agent systems (setup + monthly)

### Market anchors

| Offer type | Public signal | Source conf. |
|------------|---------------|--------------|
| AI receptionist monthly | **$179/mo** (Lithium Phoenix page); **$500/mo** (A³) $0 setup | High |
| One-time agent build | **$7,500** build + **$30–150/mo** AI direct (AutomateNexus Phoenix); pilot assessment **$500**; marketing “from **$2,500**” | Med-High |
| Shop SaaS substitute (V1) | Printavo **$109 / $244** mo | High |
| Menu board SaaS (V2) | **$7–$30/screen** common; MBM **$89–$299** | High–Med |

### Draft vs recommendation

| SKU | Draft | Recommendation | Notes |
|-----|-------|----------------|-------|
| **A-SETUP** | ~$3,000 | **LOCK $2,500–$4,500 band**; **list $3,000** for single-hero workflow (H1 *or* H2); **$4,500+** if both heroes or multi-location | Below $7.5k ownership builds; above free-setup bots |
| **A-MO intro** | $400 | **LOCK $400 for first 90 days** max, written end date | Needed vs $179–500 receptionist sticker shock |
| **A-MO list** | $550 | **LOCK $550** for T1 managed private keep-alive (monitoring, prompt/ops tweaks, uptime, light change budget) | Must itemize inclusions so it is not “ChatGPT tax” |
| **A-MO T2 self-host** | (unspecified) | **$250–$400/mo** keep-alive *or* pure T&M after setup | Lower fixed fee if client owns runtime; do not give T1 work at T2 price |
| **Add-on screens / seats** | — | Optional **$15–40/endpoint-equivalent** only if you absorb board SaaS-like load; else client pays Kitcast/etc. direct | Avoid becoming cheap digital signage |

### Do not go below (A)

| Line | Floor | Why |
|------|-------|-----|
| A-SETUP real agent system | **$2,500** | Below this you are building free-setup receptionist economics |
| A-SETUP “audit only” | **$500** OK (matches AutomateNexus pilot) — **do not call it setup** | Preserve setup SKU integrity |
| A-MO managed (T1) | **$350** steady-state | Below ~$350 you cannot fund human keep-alive |
| Never | Price A-MO to **beat $179** Lithium | Different product; you will lose and train bad buyers |
| Never | Unlimited scope inside $550 | Cap monthly change hours (e.g. 2–4 hrs included) |

**Positioning fence:** Sell A against **estimator hours / wrong prices / missed catering**, not against Printavo $244 or Kitcast $7.

**Confidence:** **med-high** on setup band; **med** on $550 list (depends on inclusion card Dick Jones writes).

---

## 4. B — tech guy retainer

### Market anchors

| Model | Signal | Conf. |
|-------|--------|-------|
| Fractional IT 10–20 hrs | **$1,500–3,000/mo**; packages **from $2,000/mo** | Med (LocalEdge) |
| Fractional CTO retainer | **$5,000–15,000/mo** (10–20 hrs/wk) — different altitude | Med |
| Effective $/hr at $2k / 8h | **$250/hr** | Math |
| Phoenix T&M | **$100–$150** common professional floor/mid | High |

### Draft vs recommendation

| SKU | Draft | Recommendation |
|-----|-------|----------------|
| **B-MO 8h bank** | ~$2,000 | **LOCK list $2,000** ($250/hr) for **named systems list + proactive hygiene + priority remote** |
| **B-MO value framing** | — | Publish as **“8 hour bank + systems ownership”**, not “$250/hr prepay” |
| **B intro (optional)** | — | **$1,600–1,800 for first month only** if close requires (effective $200–$225/hr)—time-boxed |
| **Per-job B** | — | Use **HR rates**; minimum **2h remote / 3h on-site** |

### Do not go below (B)

| Line | Floor | Why |
|------|-------|-----|
| 8h bank list | **$1,600** (intro only) / **$1,800** ongoing absolute floor | Below ~$1.6k you are cheaper than marketed fractional packages *and* leave no premium vs MSP T&M |
| Hours in bank | Do not sell **“unlimited”** under B-MO | Classic MSP trap without seat revenue |
| Scope | B is **not** full MSP (no pretending 24/7 NOC) | Or you will be compared to $150/user unfairly |

**Confidence:** **med-high**. $2k is correct *premium SMB tech retainer*; wrong if sold as break-fix prepay.

---

## 5. AB bundle

| Draft | Recommendation |
|-------|----------------|
| Modest discount | **LOCK 8–12% off sum of A-MO list + B-MO list**, **not** off setup |
| Example | A $550 + B $2,000 = $2,550 → **AB-MO $2,250–$2,350** |
| Setup | A-SETUP stays full or **5% AB courtesy max** |
| Rule | Discount rewards **commitment**, not cherry-picking receptionist scope |

### Do not go below (AB)

- **AB-MO floor $2,100** for full A-MO list scope + 8h B  
- Never AB-price a **receptionist-only** A at bundle rates  
- If client wants only A thin + B, sell **separately** (no fake bundle)

**Confidence:** **med** (architectural; little public AB comps—by design unique).

---

## 6. P-PERK — print near-cost

| Draft | Recommendation |
|-------|----------------|
| Cost +5–10% | **LOCK cost +8% default**; **+5%** only for high-volume active AB retainers; **+10–12%** for B-only or low volume |
| Art | **Always separate** (as drafted) |
| Eligibility | **Active B or AB only**; grace ≤15 days on lapse then retail |
| Who competes | PRI, AlphaGraphics, Artisan, PHX Signs — quote-driven incumbents |

### Do not go below (P)

- **Cost +5% absolute floor** (below this courier/defect/admin risk)  
- Never **at-cost or below-cost** “to win A”  
- Never let P margin **subsidize** A-MO discounts in the same quote without CFO-visible line

**Confidence:** **med** (no public broker cards; discipline is internal).

---

## 7. Fly-out / remote 10% thesis — travel fences

Geography: ~90% AZ · ~10% remote + fly-out (Frank can fly cheap; travel billed).

### Market practice signals

| Pattern | Signal | Conf. |
|---------|--------|-------|
| Local travel fee | **$20** flat + hourly (Mobile Tech) | High local |
| Consultant travel time | Often **50% rate** for travel hours; or day-rate inclusive; expenses pass-through | Med (industry forums / practice) |
| After-hours premium | **2×** hourly (MBPS) | High |
| Fractional day packaging | Day/ half-day common for non-local | Med |

### Recommended Frank travel card

| Element | Fence | Notes |
|---------|-------|-------|
| **Expenses** | **100% pass-through** (airfare, ground, hotel, reasonable meals) pre-approved over $X | No markup needed if HR/day strong |
| **Travel time** | **50% HR-Remote** portal-to-portal **or** waived if full **day rate** chosen | Pick one model per engagement; put in SOW |
| **On-site day rate (fly-out)** | **8 × HR-On-site** (use 1.35× remote) **≈ $1,888** at $175 remote · **or package $1,800–$2,200/day** | Cleaner than nickeling hours |
| **Half day** | **0.6 × day rate** (not 0.5) | Protects short trips |
| **Minimum fly-out** | **1 billable day + expenses** even if “two hour job” | Or refuse trip |
| **Remote-first default** | Fly only when **paid** and outcome needs hands/eyes | Matches thesis |
| **Portal-to-portal vs site-only** | Prefer **portal-to-portal at 50%** *or* day rate; avoid unpaid airport time | |

### Do not go below (travel)

- Unpaid same-day fly for “relationship”  
- Day rate **under $1,500** out of state when remote list is $175  
- Absorb airfare “in the rate” without written day-rate uplift  

**Confidence:** **med** (practice-based; lock into SOW template).

---

## 8. AZ-specific notes (COL / density)

| Factor | Observation | Pricing effect |
|--------|-------------|----------------|
| Phoenix MSP density | Many publishers of per-user ranges; transparent seats common | Buyers **educated** on $100–200/user—don’t fight seat math; stay out of MSP category |
| AI agency SEO density | High programmatic “Phoenix AI” pages | Expect **$179–500/mo** mental anchors for “AI” |
| Break-fix still alive | $100/hr + travel published | Floor for pure hands-on |
| Print production dense | PRI, AlphaGraphics, signs shops | P is relationship game; no public rate war online |
| COL vs coasts | Guides claim Phoenix MSP ≈ national, below LA/NY | **No need to discount below national AI setup bands** for AZ COL |

**Confidence:** **med**.

---

## 9. One-page lock card (copy into commercial architecture)

```
HR-Remote list ............. $175/hr
HR-On-site local ........... 1.35 × remote (≈ $236)
P1 ......................... 1.5 ×
P0 ......................... 2.0 ×
HR floors .................. $150 remote / never match $65 MSP on-demand

A-SETUP list ............... $3,000 (band $2,500–$4,500)
A-SETUP floor .............. $2,500
A-MO intro (≤90d) .......... $400
A-MO list T1 ............... $550 (capped included hours)
A-MO floor T1 .............. $350
A-MO T2 self-host .......... $250–$400 or T&M

B-MO 8h list ............... $2,000
B-MO floor ongoing ......... $1,800 ($1,600 intro-only)
B framing .................. hours bank + systems list (not cheap T&M)

AB-MO ...................... 8–12% off A-MO list + B-MO list
AB-MO floor example ........ ~$2,100+
A-SETUP in AB .............. full or ≤5% courtesy

P-PERK ..................... cost +8% default (floor +5%; B-only +10–12%)
Art ........................ always separate
Eligibility ................ active B/AB only

Fly-out day rate ........... ~$1,800–$2,200 + expenses 100% PT
Travel time ................ 50% portal-to-portal OR included in day rate
Min fly-out ................ 1 day + expenses
```

---

## 10. Adjust triggers (when to reopen fences)

| Trigger | Action |
|---------|---------|
| Win-rate &lt;20% on A-MO at $550 after 10 pitches with full scope card | Test $475 list—not $300 |
| Consistent loss to $179–500 receptionist **on same scope** | You are underscoping demos—fix sales, not price |
| B clients burning &gt;8h monthly | Raise bank or overage at HR; do not silently expand |
| Print defects / freight eating +8% | Move default to +10% |
| True AZ peer publishes agent+retainer public card | Re-benchmark within 30 days |

---

## 11. Confidence summary

| Block | Level | Rationale |
|-------|-------|-----------|
| HR | **High** | Multiple primary Phoenix rate cards |
| A-SETUP / A-MO | **Med-High** | Primary AI agency pages + SaaS substitutes; product uniqueness limits comps |
| B-MO | **Med-High** | Fractional packages + local T&M; 8h@$2k is premium by design |
| AB | **Med** | Few public bundles like this |
| P | **Med** | Logic/discipline; no public broker cards |
| Travel | **Med** | Practice norms + local travel fee |

**Gaps that do not block lock:** private Clutch quotes; PJ internal print COGS; restaurant marketing-agency retainers AZ; mystery-shopped MSP proposals.
