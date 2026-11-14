# Tokpie ↔ CoinGecko Volume Integrity — Live Delta Refresh

task_id: t_45a9201e  
date: 2026-11-14T19:10:13Z  
author: Geraldov21 (GIU) — Kanban-native deep-research  
prior_brief: `/home/ice/know/research/tokpie-listing-strategy-t_c6c19df1.md` (commit 47ecde1, 2026-11-13)  
scope: Delta-only refresh of CG volume-integrity policies + Tokpie live posture + FURA market status. Not investment, legal, or listing advice.

---

## objective

Refresh, with live primary sources, whether CoinGecko still treats inorganic / wash-style volume as a listing and data-integrity risk, what Tokpie currently markets as “volume assistance / free MM,” how Tokpie itself scores on CG today, and what that implies for a FURA path that leans on Tokpie for CG visibility.

## summary

**Verdict unchanged from t_c6c19df1, with sharper live evidence.**

1. **CoinGecko still treats artificial volume inflation as a first-class integrity problem.** CG’s own glossary frames wash trading as volume inflation often done “by a ‘market maker’ … or by an exchange itself.”[1] Exchange rejection criteria still cite “suspicious or inorganic trading volumes, which conflict with our data integrity standards” and “anomalous volumes.”[2] Trust Score still exists because “exchanges may artificially inflate their reported trading volume,” with Liquidity at 50% weight, volume-consistency vs a trusted benchmark, an exchange-level quality gate, weekly recalc, and penalties for “abnormal patterns.”[3] Volume aggregation still “excludes trading pairs that have been blacklisted for inconsistent data and have not been updated for over 3 hours.”[4] Listings remain discretionary and revocable without prior notice; existing listings are periodically reviewed and can be deactivated/removed.[5][6][7]

2. **Tokpie still sells the product CG’s filters are designed to scrutinize.** Live Tokpie listing copy still offers “free liquidity and volume support through our partnerships with multiple market makers,” FAQ language of “perpetual free volume support,” a 2–14 day tracker timeline after listing, and a claim that listing “guarantees … eligible for tracking on CoinMarketCap and CoinGecko” (eligibility, not CG approval).[8][9] Fee table still lists base **990 USDT + $250 in tokens** with Total w/ MM/VA equal to base (VA bundled as free in Tokpie’s framing).[10]

3. **Tokpie remains CG-tracked but mid/low-trust with live anomaly flags.** Live CG API at capture: Trust Score **6/10**, rank **#91**, ~**150.6 BTC** 24h volume, 89 coins / 103 pairs.[11] Of 73 returned tickers, **4 pairs** carry `is_anomaly=true`: SOL/USDT, UNI/ETH, MANA/ETH, POL/USDT.[12] CMC still lists Tokpie active, Profile Completion Score **55%**, ~**$9.47M** spot 24h (quote 2781), description “Bounty Stakes Trading.”[13]

4. **FURA is not on CoinGecko.** CG search API for `FURA` / `FuraCoin` returned **zero coins**.[14] Project site points to ERC-20 `0xe163…8ad4` and Uniswap pool `0x400d…a34b`, still in Early Access / PinkSale narrative with “Buy FURA on Uniswap” framed as coming after launch.[15] Live GeckoTerminal: Uniswap v3 **FURA/USDT** pool exists since 2022-04-16 but is effectively **dead** — ~**$33.78** reserve, **$0** 24h volume, **0** txs across all windows; token `coingecko_coin_id: null`.[16]

**Net for operators:** Tokpie can still create *mechanical eligibility* for a CG application because Tokpie is a CG-integrated exchange.[11][5] It does **not** create a durable CG listing, does **not** override CG discretion, and its marketed free/perpetual volume support sits in direct tension with CG’s anti-inflation / anomaly machinery.[1][2][3][8] For FURA specifically, a near-empty Uniswap book plus no CG presence means any Tokpie-only volume story would be the *dominant* liquidity signal CG sees — which raises, rather than lowers, inorganic-volume scrutiny risk.[16][5][7]

---

## plan

| task_id | title | status |
|---------|--------|--------|
| T1 | Re-fetch CG primary policies (wash, trust, listing/delist, methodology volume) | done |
| T2 | Re-fetch Tokpie marketing (listing, fees, help) | done |
| T3 | Live Tokpie on CG API + CMC | done |
| T4 | FURA on CG + Uniswap/GT | done |
| T5 | Diff vs t_c6c19df1; write structured report | done |

Assumptions: analysis is about **exchange-path / volume-integrity risk**, not token fundamentals quality. “Volume assistance” mechanism (real two-sided MM vs circular flow) remains **not independently verifiable** from public pages.

---

## delta_vs_prior (t_c6c19df1 → t_45a9201e)

| dimension | 2026-11-13 prior | 2026-11-14 live | delta |
|-----------|------------------|-----------------|-------|
| CG wash / inorganic policy | Documented | Same primary pages still live; wash glossary + exchange rejection + Trust Score + unpublish language confirmed | **No material policy change** |
| Tokpie Trust Score | 6/10 | **6/10** | Unchanged |
| Tokpie CG rank | (page-era; ~mid 6/10 band) | **#91** of scored exchanges | Rank now explicit via API |
| Tokpie 24h vol (CG) | ~$8.6M / ~130–143 BTC (page) | **~150.6 BTC** (~API) | Same order of magnitude |
| Live anomaly pairs | Prior page: ETH/USDT, UNI/ETH (and other page anomalies in HTML scrape) | API `is_anomaly`: **SOL/USDT, UNI/ETH, MANA/ETH, POL/USDT** (4/73) | **Still anomalous**; pair set **rotated** (ETH/USDT not in current anomaly set; SOL/MANA/POL added) |
| Tokpie VA marketing | free/perpetual MM volume | Same claims still on listing page + fees table 990+$250 | Unchanged offer |
| “Guarantee” language | eligibility guarantee | Same FAQ answer live in page JSON-LD | Unchanged |
| FURA on CG | Not in prior scope deeply | **Not listed** (empty search); GT `coingecko_coin_id: null` | Confirmed gap |
| FURA Uniswap | Not centered | Pool live but **~$34 TVL, $0 vol** | Material: **no organic DEX depth** to diversify CG volume signals |
| CMC Tokpie | PCS 55%, ~$3.8–5.4M noted prior | PCS **55%**, spot ~**$9.47M**, status active | PCS stable; volume higher at this snapshot |

**Conclusion of delta:** Prior risk thesis is **reaffirmed**, not overturned. The main new operational fact is FURA’s **absent CG listing + dead DEX pool**, which makes a Tokpie-centric volume plan higher-stakes for data-integrity optics.

---

## findings

### 1. CoinGecko volume-integrity posture (live)

**Wash trading definition (CG glossary):**  
“Wash trading is a way to artificially inflate the trading volume of any tradable assets on an exchange… usually done either by a ‘market maker’ to help a coin appear appealing to trader or by an exchange itself…”[1]

**Exchange non-listing / rejection signals:**  
- “Insufficient or lack of organic trading volume — We have detected suspicious or inorganic trading volumes, which conflict with our data integrity standards.”[2]  
- “Mal-intent / Suspicious application — … anomalous volumes, or insufficient disclosure of its leadership team.”[2]  
- CG “reserves the rights to publish or unpublish any listed cryptoasset, ICO, or exchange … without prior notice if we feel that any of the presented information is inaccurate in any way.”[2][5]

**Token listing / survival:**  
- Must be on at least one active exchange CG has integrated; lack of **organic** attention (liquidity, volume, social sentiment) can reject.[5]  
- “Projects traded only on self-serviceable centralized/decentralized exchanges may be rejected due to security concerns and lack of liquidity.”[5]  
- Criteria non-exhaustive; discretionary listing rights.[5]  
- Existing listings: “we periodically review existing listings, and if they fail to meet our requirements, then they will be deactivated or removed.”[6]  
- Inactive page when CG “unable to track any trades of the asset on any exchanges.”[7]

**Trust Score (current methodology page):**  
- Motivated by artificial volume inflation.[3]  
- Weights: Liquidity **50%**, Cybersecurity **20%**, Regulation **15%**, Incident **10%**, Proof of Reserves **5%**; curved vs peer population; **weekly** recalc.[3]  
- Liquidity uses trusted-exchange **benchmark**, **volume consistency**, order-book depth, and an **exchange-level quality gate** so a few clean pairs cannot mask anomalous remainder; deviations → “abnormal patterns.”[3]  
- Thresholds intentionally undisclosed to limit gaming.[3]

**Methodology volume filter:**  
“CoinGecko’s calculation algorithm for an exchange volume excludes trading pairs that have been blacklisted for inconsistent data and have not been updated for over 3 hours.”[4]

### 2. What Tokpie markets (live)

From Tokpie listing guide (2026 page still live):  
- “Enjoy **free liquidity and volume support** through our partnerships with multiple market makers.”[8]  
- Section “Enjoy Liquidity and Volume Boost at No Cost” + FAQ “**perpetual free volume support**” with market makers.[8]  
- “After listing on Tokpie, listing on CoinMarketCap (CMC), CoinGecko, and other coin trackers takes **2 to 14 days**.”[8]  
- FAQ: “Yes, listing your token on Tokpie **guarantees that it will become eligible** for tracking on CoinMarketCap and CoinGecko. Tokpie is a recognized exchange, ensuring automatic **eligibility** for tracking.”[8]  
- Help center: “seamless access to top coin trackers like CoinMarketCap and CoinGecko, robust liquidity support…”[9]  
- Fees guide table: Tokpie **990 USDT + $250 in tokens** base and Total w/ MM/VA (VA framed as included/free vs paid MM on other CEXs).[10]

**Important parse:** Tokpie’s “guarantee” is **eligibility via a recognized/tracked exchange**, not a promise CG will approve or retain the token page.[8] CG’s own docs separate eligibility from discretionary approval and ongoing review.[5][6]

### 3. Tokpie on CoinGecko / CMC right now

| metric | value | source |
|--------|-------|--------|
| CG Trust Score | 6/10 | CG exchanges API[11] |
| CG trust_score_rank | 91 | [11] |
| CG 24h vol | 150.63 BTC | [11] |
| coins / pairs | 89 / 103 | [11] |
| country / year | Hong Kong / 2018 | [11] |
| `is_anomaly` pairs (API page1) | SOL/USDT (~$307k), UNI/ETH (~$198k), MANA/ETH (~$63k), POL/USDT (~$12k) | [12] |
| CMC status | active | [13] |
| CMC PCS | 55.0% | [13] |
| CMC spot 24h (USD quote) | ~$9.47M | [13] |
| CMC blurb | “Bounty Stakes Trading solution” | [13] |

Interpretation (inference, labeled): A stable 6/10 with **recurring anomaly flags on major-name pairs** is consistent with CG’s liquidity/quality-gate story of “abnormal patterns,” not with a green-trust venue.[3][12] Pair-level anomaly membership **rotates** day to day; the structural issue is that anomalies remain present, not which four tickers.

### 4. FURA market reality (live)

| check | result |
|-------|--------|
| CG search `FURA` / `FuraCoin` | **no coins**[14] |
| Contract (project site) | `0xe1631864495a2251Ae092d678111ACE32eE28aD4` (ETH ERC-20)[15] |
| Stated Uniswap pool | `0x400d569b45faef46df52a47efd3b7e3991eba34b`[15] |
| GT pool | FURA/USDT Uniswap v3 0.3%, created 2022-04-16[16] |
| GT reserve_in_usd | **~33.78**[16] |
| GT volume h24 / txs h24 | **0 / 0**[16] |
| GT token coingecko_coin_id | **null**[16] |
| DexScreener token pairs | **0** returned at capture[17] |
| Project narrative | Early Access → Seed / PinkSale → Public LP; Uniswap public path “being organized”[15] |

Implication: FURA does **not** currently present CG with multi-venue organic flow. A Tokpie listing would likely be the primary (or sole) CEX ticker CG could attach — concentrating integrity risk on that venue’s volume quality.[5][11][16]

### 5. Risk synthesis (facts vs inference)

**Supported facts**  
- CG fights fake/inorganic volume via Trust Score, pair blacklists, anomaly labels, organic-volume exchange criteria, and unpublish/periodic review powers.[1][2][3][4][5][6]  
- Tokpie openly sells free/perpetual MM volume support and tracker eligibility.[8][10]  
- Tokpie is still listed on CG at 6/10 with live anomalies.[11][12]  
- FURA is not on CG; DEX pool is dust-level.[14][16]

**Inferences (medium confidence)**  
- Using Tokpie’s free VA as the *main* path to “look liquid” on CG is **structurally misaligned** with CG’s published integrity model (C18-style fragility thesis from prior brief still holds).  
- Without independent organic venues (healthy DEX + higher-trust CEX), a FURA CG page—if obtained—would be **fragile** under periodic review and inactive-trade rules.[6][7][16]

**Not established**  
- That Tokpie’s MM program is literally wash trading (requires non-public order/flow analysis).  
- That CG will or will not approve any specific future FURA application.  
- Exact numeric anomaly thresholds (CG withholds them).[3]

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | CG defines wash trading as artificial volume inflation, including by market makers or exchanges. | E1 | high | supported |
| C2 | CG rejects/penalizes exchanges for suspicious/inorganic/anomalous volume under data-integrity standards. | E2 | high | supported |
| C3 | CG Trust Score weights Liquidity 50% and scores volume consistency vs a trusted benchmark, with quality gate and weekly updates; motivated by artificial volume. | E3 | high | supported |
| C4 | CG volume aggregation can blacklist/exclude inconsistent or stale (>3h) pairs. | E4 | high | supported |
| C5 | CG may unpublish assets/exchanges without prior notice; periodically deactivates listings that fail non-exhaustive criteria; inactive when no trades tracked. | E2 E5 E6 E7 | high | supported |
| C6 | Token listing requires a CG-integrated active exchange; organic attention/liquidity matter; approval is discretionary. | E5 | high | supported |
| C7 | Tokpie markets free liquidity/volume support and “perpetual free volume support” via market-maker partners. | E8 | high | supported |
| C8 | Tokpie states listing guarantees CMC/CG *eligibility* (recognized exchange), and quotes 2–14 days to tracker listing after Tokpie list. | E8 | high | supported |
| C9 | Tokpie published listing price remains 990 USDT + $250 in tokens (Total w/ MM/VA same in their fee table). | E10 | high | supported |
| C10 | Tokpie is currently CG-tracked at Trust Score 6/10, rank 91, ~150.6 BTC 24h volume. | E11 | high | supported |
| C11 | At 2026-11-14 ~19:07–19:10Z capture, CG marked 4 Tokpie pairs anomalous: SOL/USDT, UNI/ETH, MANA/ETH, POL/USDT. | E12 | high | supported |
| C12 | CMC still tracks Tokpie (active, PCS 55%, ~$9.47M spot 24h at capture). | E13 | high | supported |
| C13 | FURA/FuraCoin has no CoinGecko coin id / search hit at capture. | E14 E16 | high | supported |
| C14 | Project-linked Uniswap v3 FURA/USDT pool has ~$34 TVL and $0 24h volume (effectively non-market). | E15 E16 | high | supported |
| C15 | Whether Tokpie VA is synthetic wash vs bona fide MM is not publicly verifiable. | E8 E10 | — | unsupported (gap) |
| C16 | A Tokpie-primary CG path for FURA is more likely fragile than durable given CG integrity rules + Tokpie anomaly profile + empty DEX. | E3 E8 E11 E12 E16 | medium | partial |

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|-------------|--------------|------------|------------|------------------------|
| E1 | CG Glossary — Wash Trade | https://www.coingecko.com/en/glossary/wash-trade | curl 200 HTML 2026-11-14; text extract `/srv/scratch/t_45a9201e/txt/wash.txt` | “Wash trading is a way to artificially inflate the trading volume… usually done either by a \"market maker\" … or by an exchange itself…” |
| E2 | CG Support — Why is my exchange not listed | https://support.coingecko.com/articles/2131953804-why-is-my-exchange-not-listed-on-coingecko | curl 200 | “suspicious or inorganic trading volumes, which conflict with our data integrity standards”; “anomalous volumes”; unpublish without prior notice |
| E3 | CG Support — Trust Score Methodology | https://support.coingecko.com/articles/6652614191-trust-score-methodology | curl 200 | Artificial volume motive; weights 50/20/15/10/5; volume consistency vs benchmark; quality gate; weekly; thresholds withheld |
| E4 | CG Methodology | https://www.coingecko.com/en/methodology | curl 200 | “excludes trading pairs that have been blacklisted for inconsistent data and have not been updated for over 3 hours” |
| E5 | CG Support — Why is my token not listed | https://support.coingecko.com/articles/5547344066-why-is-my-token-not-listed-on-coingecko | curl 200 | Integrated exchange required; organic attention; discretionary; non-exhaustive; unpublish rights |
| E6 | CG Support — Why can’t I find this listing anymore | https://support.coingecko.com/articles/4520117410-why-can-t-i-find-this-coin-token-listing-anymore | curl 200 | Periodic review; deactivate/remove; criteria not exhaustive |
| E7 | CG Support — Why is my token/coin page inactive | https://support.coingecko.com/articles/4063932759-why-is-my-token-coin-page-inactive | search snippet + prior triangulation | Inactive when no trades tracked on any exchange |
| E8 | Tokpie — Token listing guide | https://tokpie.io/blog/token-listing-on-tokpie-exchange/ | curl 200 HTML+JSON-LD FAQ | Free liquidity/volume support; perpetual free volume support; 2–14 days; eligibility guarantee wording |
| E9 | Tokpie Help — Maximize listing visibility | https://help.tokpie.com/hc/en-us/articles/28309464078225-Maximize-Your-Token-s-Potential-List-on-Tokpie-Exchange-for-Unmatched-Visibility-and-Growth | curl 200 | CMC/CG access + liquidity support marketing |
| E10 | Tokpie — CEX listing fees 2026 | https://tokpie.io/blog/crypto-exchange-listing-fees-2026/ | curl 200 | Tokpie row: 990 USDT + $250 tokens; Total w/ MM/VA same; VA/MM vocabulary |
| E11 | CG API — exchanges/tokpie | https://api.coingecko.com/api/v3/exchanges/tokpie | curl JSON 2026-11-14T19:07Z | trust_score 6; rank 91; trade_volume_24h_btc 150.63; coins 89; pairs 103 |
| E12 | CG API — tokpie/tickers | https://api.coingecko.com/api/v3/exchanges/tokpie/tickers?page=1 | curl JSON | 73 tickers; is_anomaly on SOL/USDT, UNI/ETH, MANA/ETH, POL/USDT |
| E13 | CMC data-api exchange detail tokpie | https://api.coinmarketcap.com/data-api/v3/exchange/detail?slug=tokpie | curl JSON | active; PCS 55; spotVolume24h ~9.47e6 USD; Bounty Stakes blurb |
| E14 | CG API search FURA | https://api.coingecko.com/api/v3/search?query=FURA | curl JSON | `coins: []` |
| E15 | FuraCoin site | https://furacoin.io/ | curl 200 | Contract + Uniswap pool links; Early Access / PinkSale / public LP narrative |
| E16 | GeckoTerminal pool + token | https://api.geckoterminal.com/api/v2/networks/eth/pools/0x400d569b45faef46df52a47efd3b7e3991eba34b ; …/tokens/0xe163… | curl JSON | reserve ~$33.78; vol 0; txs 0; coingecko_coin_id null |
| E17 | DexScreener token | https://api.dexscreener.com/latest/dex/tokens/0xe1631864495a2251Ae092d678111ACE32eE28aD4 | curl JSON | pairs: 0 at capture |
| E18 | Prior GIU brief | https://github.com/arealicehole/know/blob/47ecde1/research/tokpie-listing-strategy-t_c6c19df1.md | local `/home/ice/know/research/tokpie-listing-strategy-t_c6c19df1.md` | Baseline claims C1–C18 for delta |

---

## gaps

- **G1** CG numeric thresholds for anomaly / inorganic volume remain undisclosed by design.[3]  
- **G2** Tokpie VA mechanism (inventory MM vs circular wash) not auditable from public web.[8][10][C15]  
- **G3** No on-chain forensic of Tokpie hot wallets or trade-graph clustering performed this run.  
- **G4** FURA may have other pools/chains not linked from homepage; DexScreener empty and GT top_pools empty for the stated contract, but exhaustiveness not proven.[16][17]  
- **G5** Community sentiment (Bitcointalk 2021 dispute, Trustpilot) not re-litigated this delta; prior brief still the pointer.[E18]  
- **G6** Firecrawl keyless extract 403 in this environment; pages retrieved via direct curl + CG/CMC/GT APIs (provenance stamped above).

---

## confidence

```json
{
  "level": "high",
  "rationale": "Policy claims and live venue metrics are grounded in primary CG support/methodology pages and live CG/CMC/GT APIs captured the same day. Medium only on the interpretive fragility claim (C16) and the unverifiable VA mechanism (C15). Overall report confidence high on the decision-relevant facts: CG still polices inorganic volume; Tokpie still sells free VA while scoring 6/10 with anomalies; FURA is off-CG with a dead DEX pool."
}
```

---

## recommended_next_actions

1. **Do not treat Tokpie free VA as a CG ranking strategy.** Treat it at most as temporary order-book cosmetics with explicit integrity risk under CG’s own rules.[3][8]  
2. **If a CEX is needed for CG eligibility,** prefer a higher Trust Score venue (or multi-venue) and assume Tokpie alone is a **weak durability** signal.[11][3]  
3. **Before any CG application for FURA:** seed **real** DEX liquidity (orders of magnitude above ~$34), document organic markets, and avoid single-venue synthetic volume as the headline metric.[16][5]  
4. **Contractual diligence on Tokpie (if still pursued):** written scope of “volume support” (no self-trade), who owns inventory, stop conditions, refund if CG never lists or delists, and current Trust Score/anomaly acknowledgment.[8][10][11]  
5. **Monitoring cadence:** weekly check CG `exchanges/tokpie` trust_score + `is_anomaly` tickers + FURA search until strategy locked; re-run this delta if Trust Score drops or Tokpie is untracked.[11][12]  
6. **Optional follow-up cards (not this task):** (a) higher-trust CEX shortlist for FURA; (b) wash-trade forensic methodology outline for Tokpie order flow (capability-heavy); (c) FURA multi-chain pool inventory.

---

## Sources

[1] https://www.coingecko.com/en/glossary/wash-trade — CoinGecko Glossary: Wash Trade  
[2] https://support.coingecko.com/articles/2131953804-why-is-my-exchange-not-listed-on-coingecko — Why is my exchange not listed on CoinGecko?  
[3] https://support.coingecko.com/articles/6652614191-trust-score-methodology — Trust Score Methodology  
[4] https://www.coingecko.com/en/methodology — CoinGecko Methodology (volume blacklist / stale pairs)  
[5] https://support.coingecko.com/articles/5547344066-why-is-my-token-not-listed-on-coingecko — Why is my token not listed on CoinGecko?  
[6] https://support.coingecko.com/articles/4520117410-why-can-t-i-find-this-coin-token-listing-anymore — Why can't I find this coin/token listing anymore?  
[7] https://support.coingecko.com/articles/4063932759-why-is-my-token-coin-page-inactive — Why is my token/coin page "inactive"?  
[8] https://tokpie.io/blog/token-listing-on-tokpie-exchange/ — Tokpie token listing guide + FAQ (eligibility guarantee, free/perpetual volume support)  
[9] https://help.tokpie.com/hc/en-us/articles/28309464078225-Maximize-Your-Token-s-Potential-List-on-Tokpie-Exchange-for-Unmatched-Visibility-and-Growth — Tokpie Help Center listing announcement  
[10] https://tokpie.io/blog/crypto-exchange-listing-fees-2026/ — Crypto exchange listing fees 2026 (Tokpie 990 USDT + $250 tokens)  
[11] https://api.coingecko.com/api/v3/exchanges/tokpie — Live Tokpie exchange object (trust 6, rank 91, vol)  
[12] https://api.coingecko.com/api/v3/exchanges/tokpie/tickers?page=1 — Live Tokpie tickers + is_anomaly flags  
[13] https://api.coinmarketcap.com/data-api/v3/exchange/detail?slug=tokpie — CMC Tokpie detail (PCS 55%, volume)  
[14] https://api.coingecko.com/api/v3/search?query=FURA — CG search empty for FURA  
[15] https://furacoin.io/ — FuraCoin official site (contract, pool, launch narrative)  
[16] https://api.geckoterminal.com/api/v2/networks/eth/pools/0x400d569b45faef46df52a47efd3b7e3991eba34b — Uniswap v3 FURA/USDT pool metrics  
[17] https://api.dexscreener.com/latest/dex/tokens/0xe1631864495a2251Ae092d678111ACE32eE28aD4 — DexScreener token pairs (none at capture)  
[18] `/home/ice/know/research/tokpie-listing-strategy-t_c6c19df1.md` — Prior GIU Tokpie listing strategy brief (t_c6c19df1)

---

## capture_metadata

- capture_time_utc: 2026-11-14T19:07–19:10Z  
- worker: geraldov21 / task t_45a9201e  
- scratch: `/srv/scratch/t_45a9201e/` (HTML + API JSON + txt extracts)  
- durable_report: `/home/ice/know/research/tokpie-cg-volume-integrity-delta-t_45a9201e.md`  
