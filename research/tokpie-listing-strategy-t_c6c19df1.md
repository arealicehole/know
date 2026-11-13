# Tokpie Exchange Listing Strategy & Risk — CoinGecko Path Assessment

task_id: t_c6c19df1
date: 2026-11-13
author: Geraldov21 (GIU) — Kanban-native deep-research
companion files: PLAN-tokpie-listing-strategy-t_c6c19df1.md, evidence-t_c6c19df1/ (verbatim quotes), ledger-t_c6c19df1.json

## Objective

Determine whether using Tokpie (tokpie.io / tokpie.com) as the listing venue is a
reasonable path to a CoinGecko (CG) token listing, and catalogue the risks and red
flags around that path. The analysis is about the *exchange path* and how CG treats
exchange volume — not the intrinsic quality of any specific token.

## Executive summary

Getting *on* CG is cheap and mechanical; *staying on* CG depends on data-integrity
signals that Tokpie's own marketing is engineered to defeat [3][7][8]. Tokpie sells a low-cost
CEX listing (base fee 990 USDT + $250 in tokens) and, as its headline perk,
"perpetual free volume support" via market makers, and it claims that listing on
Tokpie "guarantees" CG/CMC tracking eligibility [7][8]. CG, by contrast, treats
inorganic / wash-traded volume as a data-integrity violation, actively detects it
through outlier + benchmark-consistency filters and its Trust Score, and reserves
the right to unpublish or delist without notice [1][2][3]. Tokpie's own CG
exchange page currently shows a Trust Score of only 6/10 and explicit "Anomaly —
Trading price or volume is an outlier against the average" flags on two of its pairs
[10]. Community reports (Bitcointalk 2021, Trustpilot) describe broken CG/CMC
deliveries and withdrawal problems — sentiment, not verified fact [11] and Trustpilot (unverified).

Net: Tokpie is a *plausible* low-cost first listing that can get a token *eligible*
for CG [4][10], but the "free volume assistance" model is exactly the behavior CG is
designed to penalize [6], and CG does not guarantee approval from eligibility alone [12]. The
dominant risk is a **short-lived CG listing that gets delisted** [13] (and, if the volume
is synthetic, a data-integrity / reputational hit [6][3]) — not a durable listing [13].

---

## Part 1 — How CoinGecko actually works (primary sources)

### 1.1 Listing prerequisites (token)

- A token must be "actively tradable on a cryptocurrency exchange tracked by
  CoinGecko" before it can be listed [4]. Tokpie is itself a CG-tracked exchange
  [10], so a Tokpie pair does satisfy this prerequisite mechanically.
- CG evaluates each token listing on "an internal set of criteria, which includes
  liquidity, team presence, project maturity, as well as several other evaluation
  factors that are undisclosed" [12]. Eligibility on a tracked exchange is therefore
  necessary but not sufficient [12].
- CG states it "will determine in our discretion the suitability of your Activity
  Post for listing on our Site" and "we are not obligated to provide any reasons
  for such refusal" [5]. Approval is discretionary even when published criteria are
  met [12].

### 1.2 Delisting / untracking (how a listing is lost)

- CG "reserves the rights to publish or unpublish any listed cryptoasset, ICO, or
  exchange on our site without prior notice if we feel that any of the presented
  information is inaccurate in any way" [1].
- CG "periodically review[s] existing listings, and if they fail to meet our
  requirements, then they will be deactivated or removed" [13]. The published
  criteria are "NOT exhaustive" [13].
- Documented token-rejection / risk signals include "a lack of organic attention
  for the project" and "Rug-pull Risk" (unlocked liquidity / not traded on
  exchanges) [12]. For exchanges, documented non-listing / rejection indicators are
  "suspicious or inorganic trading volumes, which conflict with our data integrity
  standards" and "anomalous volumes" under "Mal-intent / Suspicious application" [2].

### 1.3 Volume-integrity detection (anti-inflation)

This is the core of the risk: CG does not take reported volume at face value [1][3].

- CG's own glossary defines wash trading as "a way to artificially inflate the
  trading volume of any tradable assets on an exchange" done "by a 'market maker'
  to help a coin appear appealing to trader or by an exchange itself to trick
  traders into thinking they are trading in an active exchange" [6].
- In price aggregation, CG's "calculation algorithm for an exchange volume excludes
  trading pairs that have been blacklisted for inconsistent data and have not been
  updated for over 3 hours" [1]. Price outlier detection drops tickers that deviate
  beyond a median-absolute-deviation band (or >100x for <3-ticker coins) [1].
- The Trust Score exists precisely because "exchanges may artificially inflate their
  reported trading volume"; it scores Liquidity (50%), Cybersecurity (20%),
  Regulation (15%), Incident (10%), Proof of Reserves (5%), recalculated weekly [3].
- On the Liquidity component CG checks "volume consistency" against a reference
  benchmark set and order-book depth, and "Exchanges whose data deviates
  significantly will be penalized for displaying abnormal patterns"; an
  "exchange-level quality gate" means a few clean pairs are not enough if the rest
  of the book is anomalous, and "a human review process may also take place" [3].
- In short: reported volume that is *inconsistent with the market* is exactly the
  pattern CG's filters and Trust Score are built to catch [1][3] — i.e., the very
  thing "volume assistance" manufactures [7][8].

### 1.4 CG's own wash-trade framing

CG explicitly documents wash trading as a recognized fraud pattern and ties it to
both "market maker[s]" and "an exchange itself" [6]. This matters because Tokpie's
headline perk is *free* volume from its market-maker partners — which, if the
volume is synthetic or self-dealing, sits inside CG's published definition of the
behavior CG penalizes [6][3].

---

## Part 2 — What Tokpie actually offers (primary sources)

### 2.1 The listing package + "free volume assistance"

- Tokpie's listing pitch: "Enjoy free liquidity and volume support through our
  partnerships with multiple market makers" [7].
- On liquidity: "Tokpie partners with reliable market makers to provide perpetual
  free volume support, which provides sufficient liquidity for trading a token" [7].
- Its 2026 fee guide defines the term: "VA / Volume assistance — Paid support
  intended to help exchange-side trading activity" [8]; and reports Tokpie's own
  "forever-free volume assistance service is provided at no cost" [8].
- Pricing (quoted, email-confirmed per the guide): "Listing fee is 990 USDT + $250
  in tokens", with optional market making at "49 USDT per month, or 294 USDT for 6
  months" → minimum 6-month total "1,284 USDT + $250 in tokens" [8].

Note the internal tension: Tokpie calls the volume "free" and "perpetual" [7][8],
yet its own fee taxonomy describes "volume assistance" as *paid* support
"intended to help exchange-side trading activity" [8]. "Free" here is the marketing
frame; the mechanism is exchange-side generated trading activity, which is
indistinguishable in shape from the wash-trading CG defines [6][3].

### 2.2 The CG / CMC "guarantee" claim

- Tokpie states: "listing your token on Tokpie guarantees that it will become
  eligible for tracking on CoinMarketCap and CoinGecko. Tokpie is a recognized
  exchange, ensuring automatic eligibility for tracking" [7].
- Two things are true in that sentence and both are important:
  1. **Eligibility ≠ approval.** Being on a CG-tracked exchange makes a token
     *eligible* to be submitted [4][12]; CG still applies its discretionary,
     undisclosed criteria and "we are not obligated to provide any reasons" for
     refusal [5].
  2. **Eligibility is fragile.** If CG detects inorganic volume on that exchange
     [2][3], the token (and/or the exchange) can be "deactivated or removed" [13].

### 2.3 Tokpie's actual standing on CoinGecko (live page, 2026-11-13)

- CG's Tokpie exchange page shows a **Trust Score of 6/10** and 24h volume of
  ~$8.6M [10].
- The same page flags pairs with "**Anomaly — Trading price or volume is an
  outlier against the average**" (observed on ETH/USDT and UNI/ETH at capture time)
  [10].
- CMC's Tokpie page lists a **Profile Score of 55.0%** and describes it as a
  "Bounty Stakes Trading solution" with spot volume ~$3.8–5.4M [9].

Interpretation: Tokpie is a small, mid-tier CEX with a middling CG Trust Score and
*live* anomaly flags — exactly the "abnormal pattern" profile CG's Trust Score
penalizes [3][10]. A token whose volume on Tokpie is generated by Tokpie's own
"free volume assistance" [7][8] inherits that anomaly risk [10].

---

## Part 3 — Community / third-party sentiment (labeled as sentiment, not fact)

- **Bitcointalk (Dec 2021, "TOKPIE Exchange - scam"):** an OP states Tokpie
  "convince[d] [him] that they list on CoinGecko, CoinMarketCap, etherscan and
  provide liquidity (trading volume) for a lifetime on their exchange", that he
  "paid (0.343ETH) on that moment is 750$", and that "Non of this things wasn't
  done" with no refund [11]. This is a self-reported dispute, not a verified
  finding — but it documents the *exact* promise/delivery gap relevant here: the
  CG/CMC listing that was promised did not materialize [11].
- **Trustpilot (tokpie.com):** mixed/low reviews; a sampled complaint alleges a blocked
  withdrawal and a demand for more crypto as a "withdrawal fee". The full review body
  could not be captured verbatim in this environment, so it is cited as unverified
  reputation signal only, not attached as evidence [unverified].
- **R/Token subreddit:** self-promotional "Tokpie Safe and Secure Exchange" threads
  are present — promotion, not independent endorsement [unverified].

Sentiment weight: the community record is *anecdotal* and partially promotional, so
it is reported as reputation signal only [11]. It does not establish that Tokpie is a scam [11];
 it does show a documented 2021 case where the CG/CMC delivery failed and the
"lifetime volume" promise was central to the deal [11].

---

## Part 4 — Verdict & risk assessment

### 4.1 Will Tokpie get the token on CoinGecko?

- **Eligibility: yes, mechanically.** Tokpie is CG-tracked [10], so a token on
  Tokpie satisfies CG's "actively tradable on a CG-tracked exchange" prerequisite
  [4].
- **Approval: not guaranteed and not Tokpie's to give.** CG decides on its own
  undisclosed criteria [12][5]; a token can sit unlisted after the free window
  [12]. Tokpie's "guarantee" is of *eligibility only* [7].
- **Durability: the real risk.** A CG listing whose underlying volume is
  Tokpie-generated "free volume assistance" is exposed to CG's inorganic-volume
  detection [2][3], blacklist filters [1], and the right to "unpublish … without
  prior notice" [1] / "deactivated or removed" [13]. Expect a short, fragile
  listing rather than a durable one [13][1].

### 4.2 Red flags (what to watch before paying)

1. **"Free/perpetual volume" = the behavior CG defines as wash trading** [6][7][8].
   If the volume is synthetic, CG's outlier + benchmark + Trust Score machinery is
   designed to flag it [1][3].
2. **Tokpie's own CG Trust Score is only 6/10 with live "Anomaly" pair flags** [10] —
   the exchange is already in CG's "abnormal pattern" band [3].
3. **Internal contradiction:** the fee guide calls "volume assistance" *paid* [8]
   while the pitch calls it *free* [7] — ambiguity about what is actually being
   sold, and who bears the compliance risk [8][7].
4. **CG disclaimers cut both ways:** CG won't explain rejections [5] and its
   criteria are "NOT exhaustive" [13] — you cannot audit why a Tokpie-backed listing
   was denied or dropped [5][13].
5. **Documented 2021 failure of the same CG/CMC + lifetime-volume promise** [11] and
   withdrawal-related complaints (Trustpilot, unverified) — reputation signal that
   the package can fail to deliver the headline benefit [11].
6. **No approval guarantee, non-refundable fast-pass model at CG** (CG Fast Pass is
   a paid 24h *review*, not an approval promise — see CG's listing guide) [4].

### 4.3 What this does NOT establish

- It does not prove Tokpie is fraudulent — it is a real, CG-tracked, CMC-tracked
  exchange [9][10].
- It does not prove a specific token's Tokpie volume is wash-traded [7][8] — the mechanism
  of "volume assistance" is not independently verifiable from public sources
  [7][8]. Treat "is the volume synthetic" as an open question, not a finding [7][8].

## Claims

(citations = ledger ids; full verbatim evidence in `evidence-t_c6c19df1/` and the
ledger's `quotes`)

| id | claim | confidence | status | evidence |
|----|-------|-----------|--------|----------|
| C1 | A token must be tradable on a CG-tracked exchange to be listed on CG. | high | supported | [4][12] |
| C2 | Tokpie is a CG-tracked exchange, so a Tokpie pair makes a token *eligible* for CG. | high | supported | [10][4] |
| C3 | CG approval is discretionary; CG is not obligated to explain rejections. | high | supported | [5][12] |
| C4 | CG may unpublish/delist a token or exchange without prior notice if data is inaccurate. | high | supported | [1][13] |
| C5 | CG periodically reviews existing listings and deactivates/removes those that fail requirements; criteria are non-exhaustive. | high | supported | [13] |
| C6 | CG detects and penalizes inorganic/suspicious volume ("data integrity standards", "anomalous volumes"). | high | supported | [2][3] |
| C7 | CG's Trust Score (50% liquidity, 20% cybersecurity, 15% regulation, 10% incident, 5% PoR, weekly) scores exchanges on volume consistency vs a benchmark and flags "abnormal patterns". | high | supported | [3] |
| C8 | CG's volume algorithm blacklists inconsistent-data pairs and drops price outliers. | high | supported | [1] |
| C9 | CG defines wash trading as volume inflation by a market maker or the exchange itself. | high | supported | [6] |
| C10 | Tokpie advertises "perpetual free volume support" / "free liquidity and volume support" via market-maker partners. | high | supported | [7][8] |
| C11 | Tokpie quotes a base listing fee of 990 USDT + $250 in tokens; optional MM 49 USDT/mo (294 for 6 mo). | high | supported | [8] |
| C12 | Tokpie claims listing on Tokpie "guarantees" CG/CMC *eligibility* (not approval). | high | supported | [7] |
| C13 | Tokpie's CG Trust Score is 6/10 with live "Anomaly" flags on at least two pairs (ETH/USDT, UNI/ETH). | high | supported | [10] |
| C14 | CMC lists Tokpie at Profile Score 55.0%, ~$3.8–5.4M 24h spot volume, "Bounty Stakes Trading". | medium | supported | [9] |
| C15 | A 2021 Bitcointalk dispute reports Tokpie promised + priced CG/CMC + lifetime volume, delivered none, refused refund. | medium | supported (as reported) | [11] |
| C17 | Whether Tokpie's "volume assistance" is synthetic/wash-traded is not independently verifiable from public sources. | — | unverified | [7][8] (gap) |
| C18 | A Tokpie-backed CG listing is more likely short-lived/fragile than durable, because its volume inherits Tokpie's anomaly profile. | medium | partial | [10][3][13] |

## Evidence index

See the rendered `## Sources` block below (ledger-generated). Each id maps 1:1 to a
fetched page; verbatim quotes are attached in the ledger (`ledger-t_c6c19df1.json`)
and copied to `evidence-t_c6c19df1/`.

## Gaps / unresolved questions

- **G1** CG's internal numeric thresholds for "inorganic volume" and its anomaly
  detector are undisclosed by design ("revealing specific thresholds … could open
  the door to manipulation" — CG) [3]. So we cannot state the exact line a Tokpie
  volume must cross to be flagged.
- **G2** The mechanism of Tokpie "volume assistance" (real 2-sided MM vs self-dealing
  wash trades) is not publicly verifiable [7][8] — C17 is the central open question.
- **G3** Tokpie's current regulatory posture (ADGM/MiCA claims in its blog) is
  unverified; CMC shows "Reserve data unavailable" [9], and CG's Trust Score
  Regulation component would score it low if unverified [3].
- **G4** Tokpie's live 24h volume fluctuates ($3.8M on CMC vs $8.6M on CG at
  capture) — the divergence itself is a soft signal of inconsistent feeds [9][10].
- **G5** No independent audit of Tokpie order-book depth/spread quality was
  performed; the CMC +2%/−2% depth figures are the only depth data available [9].

## Confidence

- level: **medium**
- rationale: The *rules* (how CG lists, delists, and detects inorganic volume) are
  well-grounded in primary CG documentation and are high-confidence [1][2][3].
  Tokpie's *offerings* are grounded in Tokpie's own pages + CMC/CG live pages
  [7][8][9] — high confidence that this is what Tokpie sells and how CG rates
  it. The *interpretive* layer (C17/C18: that "free volume assistance" is likely
  synthetic and that the resulting CG listing is fragile) is a reasoned inference
  from CG's published anti-inflation stance + Tokpie's anomaly flags + community
  reputation, not a directly observed fact — so the overall verdict is medium, not
  high.

## Recommended next actions

1. **Do not treat Tokpie as the durable CG path.** Use it only as a low-cost
   *first* listing to obtain CG *eligibility* [4][10], then plan a second,
   higher-trust CEX (green Trust Score) before relying on the CG listing for rankings/market-cap [3][10].
2. **Ask Tokpie, in writing, what "free volume assistance" means:** are trades
   real 2-sided retail/institutional flow, or are they generated by Tokpie-affiliated
   bots [7][8]? Get it in the contract [7]. A refusal to answer is a red flag (C17) [7].
3. **Before paying, demand:** (a) the exact CG/CMC deliverable as a written SLA with
   a cure period, (b) a refund clause for non-delivery (the 2021 dispute had no
   refund [11]), and (c) evidence of Tokpie's current CG Trust Score trajectory
   (currently 6/10 with anomaly flags [10]).
4. **Prefer organic volume over "assistance."** If the goal is CG *retention*, real
   liquidity/spread beats manufactured volume, because CG penalizes the latter
   [2][3] and will drop the listing otherwise [13].
5. **Verify the token's own quality independently** (locked supply, no honeypot,
   working website, team) — CG rejects on those grounds regardless of exchange
   [12][4].
6. **Monitor the CG anomaly flag on the Tokpie pairs** [10] after listing; if CG
   starts blacklisting Tokpie pairs for inconsistent data [1], the token's CG
   volume will drop and the listing risks removal [13].

---

## Sources

[1] https://www.coingecko.com/en/methodology — CoinGecko Methodology (listing criteria, volume aggregation, outlier filters)
    > "CoinGecko reserves the rights to publish or unpublish any listed cryptoasset, ICO, or exchange on our site without prior notice if we feel that any of the presented information is inaccurate in any way."
    > "A working exchange website with actual trading volume that matches information in API."
[2] https://support.coingecko.com/articles/2131953804-why-is-my-exchange-not-listed-on-coingecko — CoinGecko: Why is my exchange not listed (inorganic volume, mal-intent)
    > "We have detected suspicious or inorganic trading volumes, which conflict with our data integrity standards."
    > "The exchange may exhibit suspicious links, anomalous volumes, or insufficient disclosure of its leadership team."
[3] https://support.coingecko.com/articles/6652614191-trust-score-methodology — CoinGecko Trust Score Methodology (Basilisk)
    > "exchanges may artificially inflate their reported trading volume"
    > "Exchanges whose data deviates significantly will be penalized for displaying abnormal patterns."
[4] https://support.coingecko.com/articles/9489827481-how-to-list-a-new-cryptocurrency-on-coingecko — CoinGecko: How to List a New Cryptocurrency
    > "your cryptocurrency must be actively tradable on a cryptocurrency exchange tracked by CoinGecko"
[5] https://www.coingecko.com/en/listing_terms — CoinGecko Terms & Conditions for Listing
    > "We will determine in our discretion the suitability of your Activity Post for listing on our Site."
    > "you undertake to fully indemnify, defend, hold harmless and reimburse our Company for any and all losses and claims resulting from your breach of the aforegoing warranty"
    > "de-listing or modifying your Activity Post"
[6] https://www.coingecko.com/en/glossary/wash-trade — CoinGecko Glossary: Wash Trade
    > "Wash trading is a way to artificially inflate the trading volume of any tradable assets on an exchange."
    > "This is ussually done either by a "market maker" to help a coin appear appealing to trader or by an exchange itself to trick traders into thinking they are trading in an active exchange."
[7] https://tokpie.io/blog/token-listing-on-tokpie-exchange — Tokpie: Token & Coin Listing (free liquidity/volume support, CG/CMC)
    > "Enjoy free liquidity and volume support through our partnerships with multiple market makers."
    > "Tokpie partners with reliable market makers to provide perpetual free volume support, which provides sufficient liquidity for trading a token."
    > "Yes, listing your token on Tokpie guarantees that it will become eligible for tracking on CoinMarketCap and CoinGecko."
[8] https://tokpie.io/blog/crypto-exchange-listing-fees-2026 — Tokpie: CEX Listing Fees 2026 (VA/liquidity definitions)
    > "Listing fee is 990 USDT + $250 in tokens."
    > "forever-free volume assistance service is provided at no cost."
    > "Paid support intended to help exchange-side trading activity."
[9] https://coinmarketcap.com/exchanges/tokpie — CMC: Tokpie exchange page (volume, profile score)
    > "Tokpie provide a Bounty Stakes Trading solution."
[10] https://www.coingecko.com/en/exchanges/tokpie — CoinGecko: Tokpie exchange page
    > "Anomaly - Trading price or volume is an outlier against the average"
[11] https://bitcointalk.org/index.php?topic=5376009.0 — Bitcointalk: TOKPIE Exchange - scam (2021 dispute)
    > "They convince me, that they list on CoinGecko, CoinMarketCap, etherscan and provide liquidity(trading volume) for a lifetime on their exchange."
[12] https://support.coingecko.com/articles/5547344066-why-is-my-token-not-listed-on-coingecko — CoinGecko: Why is my token not listed (rejection criteria)
    > "a lack of organic attention for the project will result in the application being rejected."
[13] https://support.coingecko.com/articles/4520117410-why-can-t-i-find-this-coin-token-listing-anymore — CoinGecko: Why can't I find this coin/token listing anymore (untrack)
    > "we periodically review existing listings, and if they fail to meet our requirements, then they will be deactivated or removed."
