# Plan — Tokpie Exchange Listing Strategy & Risk (t_c6c19df1)

task_id: t_c6c19df1
objective: Assess whether using Tokpie as a listing venue is a reasonable way to get a
  token onto CoinGecko, and what risks / red flags are attached to that path.
success_criteria:
  - CoinGecko's documented listing + delisting/untracking + volume-integrity rules
    are cited from primary (coingecko.com / support.coingecko.com) sources.
  - Tokpie's own listing offering (pricing, "free volume assistance", CG/CMC claims)
    is cited from primary (tokpie.io / CMC) sources.
  - Third-party sentiment (Bitcointalk, Trustpilot) is captured as sentiment, not fact.
  - A verdict + red-flag list + next actions are produced.
  - Every material claim maps to a ledger id and verbatim quote (evidence gate passes).
assumptions:
  - "Tokpie" = tokpie.io / tokpie.com centralized exchange (founded 2018, Panama).
  - The user's goal is a CoinGecko token listing; the token is assumed to be a
    real, non-honeypot asset (the analysis is about the *exchange path*, not the
    token's own quality).
  - CoinGecko's internal thresholds are undisclosed; we reason from the published
    criteria, not the hidden rubric.
tasks:
  - task_id: T1
    title: Map CoinGecko listing + delisting + volume-integrity rules
    goal: Extract the official criteria for token/exchange listing, the delisting/
      untrack rules, and the anti-inflation (wash-trading / Trust Score) rules.
    depends_on: []
    recommended_mode: direct
    sources: [docs: coingecko.com/en/methodology, support.coingecko.com, coingecko.com/en/listing_terms, glossary/wash-trade]
    deliverable: Cited rules for listing criteria, unpublish right, volume blacklist,
      outlier detection, Trust Score, wash-trade definition, delisting triggers.
    done_definition: Each rule quoted verbatim and attached to a ledger id.
    status: done
  - task_id: T2
    title: Map Tokpie's listing offering + pricing + CG/CMC claims
    goal: Capture Tokpie's advertised listing package, "free volume assistance",
      market-maker partnerships, and its CG/CMC eligibility claims + pricing.
    depends_on: []
    recommended_mode: direct
    sources: [docs: tokpie.io blog, CMC exchange page, CG exchange page]
    deliverable: Cited Tokpie claims (free volume, CG/CMC eligibility, pricing, VA
      definition) + its actual CG Trust Score (6/10) and anomaly flags.
    done_definition: Each Tokpie claim quoted verbatim; CG page anomaly + trust score captured.
    status: done
  - task_id: T3
    title: Capture third-party sentiment (community)
    goal: Collect Bitcointalk scam-thread + Trustpilot sentiment on Tokpie, clearly
      labeled as sentiment, not verified fact.
    depends_on: [T2]
    recommended_mode: direct
    sources: [community: bitcointalk.org topic 5376009, trustpilot.com/review/tokpie.com]
    deliverable: Quoted community complaints, explicitly marked as sentiment.
    done_definition: At least one verbatim community complaint captured; labeled sentiment.
    status: done
  - task_id: T4
    title: Synthesis + risk assessment
    goal: Combine T1–T3 into a verdict: how CG treats exchange volume, what "free
      volume assistance" likely is, the delisting risk, and red-flag list.
    depends_on: [T1, T2, T3]
    recommended_mode: verification
    sources: [synthesis of T1/T2/T3]
    deliverable: Final report with claims, evidence_index, gaps, confidence, next actions.
    done_definition: Evidence gate (--evidence) passes; verdict + red flags + next actions present.
    status: done

open_questions:
  - Exact internal numeric thresholds for CG's "inorganic volume" / anomaly detection
    are undisclosed (CG explicitly withholds them to prevent manipulation).
  - Whether "free volume assistance" is wash-trading or legitimate market making is
    not independently verifiable from public sources — treat as unverified mechanism.
  - Tokpie's current (2026) on-chain/regulatory status (ADGM/MiCA claims) is unverified.
