# Research: Bookings app — STR AI-chat direct book (Scottsdale lead)

- **task_id:** `t_af826421`
- **board:** research
- **date:** 2026-10-28
- **researcher:** geraldov21 (Geraldo v2.2)
- **lead context:** Eric Estep (Print Junkie customer) — Scottsdale STRs; investors want direct booking lift via Claude / ChatGPT / Gemini vs Airbnb/VRBO
- **pack:** `/srv/scratch/handoffs/20261028-bookings-app/` (also `/srv/huly-artifacts/bookings-app-20261028/`)
- **scope:** technical feasibility research only — not a build, not directory submission, not pricing

---

## objective

Evidence-backed answer to whether/how a Scottsdale STR host (or thin multi-tenant product) can get **discoverable direct booking** inside ChatGPT / Claude / Gemini: inventory, auth, payments, discovery reality, comps, effort, and kill risks — so Dick can score KEEP / KILL / DEFER and shape a thin pilot.

## summary

**Bottom line (KEEP with narrow pilot, not full channel rebuild):**

1. **In-chat booking tools are real** on ChatGPT (Apps SDK / plugins on MCP) and Claude (remote MCP connectors). Gemini supports **user-added custom MCP apps** on the consumer Gemini web app and admin/enterprise MCP connectors — **not** a verified public third-party “apps directory + Suggested” path comparable to OpenAI/Anthropic as of this research.
2. **Discoverable ≠ guaranteed ranking.** Directory listing is necessary for Claude **Suggested Connectors** and for ChatGPT directory / contextual app suggestions; both platforms document **usage-based / quality** ranking and **do not guarantee** suggestion. Cold-start against Booking.com / Expedia / Tripadvisor is the core product risk.
3. **Money cannot move inside Claude connectors.** Anthropic Software Directory Policy + connector review criteria reject connectors that **transfer money / financial assets**. Booking.com’s own Claude connector is search → link-out only. Design must be **hold/quote → external checkout URL**.
4. **OpenAI commerce is physical-goods-first.** Plugin guidelines: commerce currently only for **physical goods**; digital goods/services and many high-risk travel patterns restricted; **external checkout** is the GA path; Instant Checkout is private beta for select marketplaces. **Lodging/STR as “physical goods” is UNVERIFIED** in official text — treat as review risk, not green light.
5. **Organic ChatGPT mention (Print Junkie story) is a different mechanism** from apps/connectors. It is closer to web/search citation (site + Maps + crawlers like OAI-SearchBot) and is **not transferable by “installing an MCP.”** Eric’s “first in line when someone asks Claude for Scottsdale rentals” is mostly **web presence + OTA inventory already in giant apps**, not a private MCP.
6. **Hospitable MCP is the clearest STR-native MCP today — host-ops, not guest discovery.** Official help: ChatGPT plugin listing + Claude custom connector URL `https://mcp.hospitable.com/mcp`; tools for properties, reservations, calendars, guest messages, etc. That is **operator AI**, not guest-facing “find Eric’s homes in chat.”
7. **Thinnest pilot:** one remote MCP with public catalog tools (`search_stays`, `get_listing`, `check_availability`, `create_hold` + checkout link) + solid direct-booking site + schema/GEO; submit Claude Community connector and optionally ChatGPT app **after** demo inventory works. **Do not** rebuild channel manager. **Do not** sell “we’ll rank you above Booking.com.”

**North-star flag:** Full multi-PMS channel + directory war = capital/attention sink. Owned income path is **fixed-scope pilot / retainer for one host’s direct book stack**, or a **thin “AI-ready direct book site + MCP” product** only if directory review and commerce classification clear.

---

## plan

```yaml
objective: Technical research on discoverable STR direct booking via ChatGPT/Claude/Gemini
success_criteria:
  - Platform matrix with official citations
  - Perplexity dump claim verify/contradict
  - Tool sketch + payment split within documented constraints
  - Discovery honesty + comps + effort + kill top 10 + thin pilot
  - Durable know path + kanban attach
assumptions:
  - Warm single lead (Eric); multi-tenant product not pre-approved
  - Research date 2026-10-28; platform policy can change
  - openai.com blog/policy HTML often 403 from this container; used developers.openai.com *.md and search snippets
tasks:
  - T1: pack intake (done)
  - T2: official OpenAI Apps/plugins docs
  - T3: official Anthropic connector docs + Software Directory Policy
  - T4: Gemini custom MCP / enterprise connectors
  - T5: Booking.com connector + Hospitable MCP comps
  - T6: discovery/GEO secondary sources labeled
  - T7: synthesize master report
open_questions:
  - Does OpenAI treat vacation-rental stays as allowed "physical goods" commerce?
  - Exact ChatGPT contextual-suggestion ranking features (not fully in public MD)
  - Eric's PMS/SoT and whether Hospitable (or other) already in stack
```

---

## 01 / A — Platform capability matrix

| Dimension | ChatGPT (OpenAI) | Claude (Anthropic) | Gemini (Google) |
|---|---|---|---|
| **Guest can use tools in chat** | Yes — Apps SDK / plugins built on **MCP**; tools + optional UI widgets | Yes — remote **MCP connectors** in claude.ai / Desktop / mobile / Code | Partial — **custom apps** via MCP server URL in Gemini web app; Workspace/Enterprise connectors admin-gated |
| **Auth model** | OAuth / signed-in app connection; directory apps disclose data shared; test account required for review | OAuth 2.0 if tools act on user account; or **no auth** for public data; fixed headers exist for custom connectors | OAuth-style sign-in patterns for connected apps; custom MCP URL add |
| **Public directory** | Yes — app/plugin directory; submissions opened (announced Dec 2025); org verification + Apps Management Write | Yes — Connectors Directory (Verified + Community); submit via directory manage on paid plans | **No equivalent verified public third-party directory for consumer discovery found.** Custom MCP is user-added. Gemini Enterprise custom MCP data stores are org admin. |
| **Contextual suggestion** | Documented product behavior (apps suggested mid-chat for relevant intents); quality/featured path in guidelines — **ranking formula not public** | **Directory connectors eligible for Suggested Connectors**; custom URL connectors **never** suggested; ranking **usage-based** | Connected apps may be used when linked; **no verified “Suggested third-party MCP” submission path** like Claude |
| **Install friction** | Directory: search/connect; custom/dev mode for testing | Directory: one-click connect; custom: paste URL (install links can prefill); **paid plan required for custom connectors** (Hospitable docs + Anthropic materials) | User must add custom app URL (or admin enables connectors) |
| **Money / checkout** | Commerce currently **physical goods only**; **external checkout** GA; Instant Checkout **private beta** select marketplaces; no arbitrary embedded third-party checkout in plugin UI; money-transfer style services prohibited | **Unsupported:** transfer money/crypto/financial assets via connector. Pattern in wild: search + **link out** (Booking.com) | Not researched to a lodging-specific policy page; no claim of in-chat payment for arbitrary third parties |
| **Write / book actions** | Allowed in principle via tools if policy/commerce allow; destructive actions need clear UX | Writes allowed if split tools + `destructiveHint`; **not** money movement | Depends on custom server; no public guest-booking marketplace path verified |
| **Review bar** | ZIP package, 5 positive + 3 negative test cases, video, test account (no MFA), privacy policy, automated + human review; one active review | Tool design checklist (split R/W, annotations, no prompt-injection in descriptions), test credentials, docs/privacy, auto scan → **Community** default; optional Verified escalation | N/A for public directory (gap) |
| **STR fit** | Possible as **inventory search + external book**; giant OTAs already live | Possible as **public catalog connector** or private host connector; money out-of-band | Weak for *discovery*; possible for power-user custom link |

### Confidence by cell group
- Claude directory vs custom + money ban: **high** (official docs)
- OpenAI physical-goods commerce + external checkout: **high** (plugin guidelines + monetization MD)
- OpenAI lodging classification: **low / UNVERIFIED**
- Gemini public discovery parity: **medium-high that it does *not* match** (support pages show custom add; no submit-to-directory doc found)
- ChatGPT suggestion ranking details: **medium** (product claims + secondary blogs; primary blog HTML 403 here)

---

## 02 / B — Reference architecture (stack-agnostic)

### Design principle
Separate **three surfaces** that people conflate:

| Surface | Who | Job | Discovery? |
|---|---|---|---|
| **S0 Direct book site** | Any guest | Canonical inventory, rates, photos, policies, checkout | Feeds web/AI search citations |
| **S1 Public guest MCP / app** | Travelers in ChatGPT/Claude | Search/quote/hold + link to S0 checkout | Only if **directory-listed** |
| **S2 Host-ops MCP** | Eric / VM | Calendar, messages, reservations (e.g. Hospitable) | None for guests |

Eric’s SMS ask is **S1 discovery**. Hospitable is **S2**. PJ “showed up in ChatGPT” is closer to **S0 + search**, not S1.

### Recommended components
1. **Inventory SoT** — Eric’s PMS or a thin DB mirrored from PMS (availability, min stay, rates, photos, house rules). Do not invent a second channel manager.
2. **Public read API** — listings + availability + quote (idempotent).
3. **Hold service** — short TTL hold / draft reservation **without capturing payment** in the LLM platform.
4. **Checkout on own domain** — Stripe/Lodgify/Hostaway checkout / existing direct-book engine; returns confirmation URL.
5. **Remote MCP server (HTTPS, Streamable HTTP)** — thin tool facade over (2)–(4).
6. **Optional ChatGPT UI widget** — card carousel of listings (Apps SDK); still link-out to pay.
7. **Observability** — tool call logs, hold conversion, review test tenant.

### Tool sketch (directory-safe)

| Tool | Side | Annotations (Claude) | Notes |
|---|---|---|---|
| `search_stays` | read | `readOnlyHint: true` | geo, dates, guests, amenities → ranked list + deep links |
| `get_listing` | read | `readOnlyHint: true` | detail, photos, policies |
| `check_availability` | read | `readOnlyHint: true` | dates → available / alternates |
| `price_quote` | read | `readOnlyHint: true` | taxes/fees breakdown; **not** charge |
| `create_hold` | write | `destructiveHint: true` | creates expiring hold; returns `checkout_url` |
| `get_hold_status` | read | `readOnlyHint: true` | paid / expired / cancelled |
| ~~`charge_card`~~ | — | — | **DO NOT** expose on Claude; avoid on ChatGPT until commerce class clear |
| ~~`api_request`~~ | — | — | Rejected by Claude review |

**Auth for guest catalog:** prefer **no user OAuth** for public search tools (Anthropic allows unauthenticated public data). If holds require guest identity, collect email in tool args and verify on checkout page — or OAuth only for “my trips.”

**Multi-tenant note:** Claude docs: directory expects **users connect to different URLs only if each URL is its own listing** (or use auth). One shared MCP with host_id in every tool is the scalable shape; per-host directory rows do not scale.

```
Guest prompt
    → Platform may suggest directory app/connector (not guaranteed)
    → search_stays / get_listing / check_availability / price_quote
    → create_hold → checkout_url (S0)
    → Guest pays on S0
    → Webhook → SoT confirmed booking
    → Optional get_hold_status in chat
```

---

## 03 / C — Discovery & distribution reality

### What “show up when someone asks for Scottsdale rentals” actually means

| Mechanism | Works without guest install? | Controlled by small host? | Evidence |
|---|---|---|---|
| **Web / ChatGPT Search citation** | Yes | Partially (site quality, GBP/Maps, allow OAI-SearchBot, reviews) | Secondary GEO sources; OpenAI primary search pages 403 here — treat mechanism as real but **rank UNVERIFIED** |
| **OTA inventory inside Booking/Expedia apps** | Yes (if listed on OTA) | Only by listing on OTA (defeats “direct”) | Booking.com + Expedia ChatGPT apps widely reported; Booking.com Claude connector official |
| **Directory app/connector suggestion** | After connect once; first time needs accept | Only if listed **and** ranked | Claude: official Suggested Connectors for directory only. ChatGPT: product + guidelines “enhanced distribution” |
| **Custom MCP URL / install link** | No — prior knowledge or host shares link | Yes | Claude directory-vs-custom; Gemini custom app URL |
| **Paid ChatGPT ads** | Possibly | Yes if ads product available | Mentioned in SMS by Frank; **not verified** in this pass |

### Honest answers to Eric-shaped questions
- **“Can guests book through Claude without installing something?”**  
  Not for *your* private server unless it is **directory-listed** and Claude **suggests** it — still a connect click first time. Even then payment is **off-Claude**.
- **“Will we be first in line vs Airbnb?”**  
  **No evidence** any platform promises local-host priority over OTAs. Usage-based ranking favors incumbents.
- **“Is PJ ChatGPT visibility the same product?”**  
  **No.** PJ is citation/recommendation of a local business website. STR direct book needs either citations to **S0** or a competitive **S1** app against OTAs.

### GEO / site baseline (still the highest-leverage cheap move)
- Fast, crawlable listing pages (unique URL per property), Availability/Lodging schema where valid, clear NAP/geo, allow AI search crawlers intentionally, Google Business Profile if applicable, fresh photos/policies.
- This supports **both** classic SEO and AI answers — without directory politics.

---

## 04 / D — Competitive / comparable landscape

| Comp | Type | Role | Lesson |
|---|---|---|---|
| **Booking.com in ChatGPT** | OTA app | Search/plan/book travel in chat | Incumbent distribution; not a small-host path |
| **Expedia in ChatGPT** | OTA app | Same | Same |
| **Booking.com Claude connector** | Official MCP | Search + filters; **bookings/payments not in Claude**; link out | **Gold-standard pattern** for Anthropic policy |
| **Hospitable MCP** | PMS host-ops | `https://mcp.hospitable.com/mcp`; ChatGPT plugin listing; Claude custom connector; props/reservations/messages/calendars/payouts | Proves STR MCP demand is **ops automation**, not guest discovery |
| **OpenTable etc. in Gemini connected apps** | Partner integrations | Reservations-like partners exist in Google ecosystems | Partner path ≠ open submit |
| **Guesty / Hostaway / Hostfully MCP** | — | No strong official guest-directory MCP found this pass | Don’t assume PMS gives guest ChatGPT placement |
| **“AI booking startup” wrappers** | Various | Often custom-GPT or link-in-bio grade | Install-first; weak discovery |

**Positioning implication:** Competing as “the Scottsdale Booking.com inside ChatGPT” is a **lose**. Competing as “Eric’s direct inventory + hold + checkout, reachable if suggested or linked, and strong on the open web” is the only honest wedge.

---

## 05 / E — Build effort & operating model (order-of-magnitude)

| Slice | Rough effort | Calendar | Notes |
|---|---|---|---|
| S0 direct site polish + quote/checkout | 3–10 eng-days if PMS already has direct book; 2–6 weeks if greenfield | Depends on PMS | Highest ROI regardless of AI |
| Public read API + holds | 3–8 eng-days | 1–2 weeks | Need TTL holds, idempotency, webhook |
| MCP server (6 tools, HTTPS, logs) | 3–7 eng-days | 1 week | TS or Python MCP SDK |
| Claude custom test + Community submit package | 2–5 days | + review wait UNVERIFIED | Test account mandatory |
| ChatGPT Apps SDK widget + directory package (5+3 cases, video) | 1–3 weeks | + review | Heavier than Claude |
| Gemini custom URL docs for power users | 0.5–1 day | — | Not a growth channel |
| Multi-tenant productization (billing, host onboarding, SLAs) | months | — | **Capital sink** if premature |
| Ongoing ops | on-call for MCP uptime, policy re-review, rate limits, support | continuous | Directory delist risk |

**Cost drivers:** review/test tenant with realistic Scottsdale inventory; payment/compliance on S0; availability correctness (double-book risk); ToS of Airbnb if any API scrape (don’t scrape).

**Who runs it:** For pilot — Frank/stack hosts MCP + S0 for Eric under retainer. For product — needs support rotation; not a pure “ship and forget” print-shop side quest.

---

## 06 / F — Kill risks (top 10) + thinnest pilot

### Kill / defer risks (ordered)
1. **Discovery mirage** — directory eligibility ≠ impressions; OTA apps dominate travel intents.
2. **Claude money ban** — any design that charges in-connector fails review / policy.
3. **OpenAI commerce class** — STR may not count as allowed physical-goods commerce; app rejected or limited to search-only.
4. **High-chargeback / abusive travel** language in OpenAI prohibited services — lodging refunds/disputes could trigger scrutiny (**interpretive risk**).
5. **Cold-start ranking** — usage-based Suggested Connectors need users you don’t have yet.
6. **Guest plan gate** — Claude custom connectors need paid Claude; even directory users may be a subset of travelers.
7. **PMS/SoT ambiguity** — wrong inventory → double bookings → reputation kill.
8. **Channel-manager rebuild temptation** — calendar sync across Airbnb/VRBO/direct is a company, not a pilot.
9. **Policy churn** — Apps SDK / connector rules moving fast (2025–2026).
10. **Gemini expectation mismatch** — investors lump three logos; Gemini path is weakest for open discovery.

### Thinnest valuable pilot (recommend)
**Name:** “Direct Book Spine + Claude Public Catalog MCP (Eric-only)”

**In scope**
1. Confirm PMS + existing direct-book checkout (or stand up minimal S0).
2. Implement read API + holds + checkout_url.
3. Ship remote MCP with tools above; no card data.
4. Add as **Claude custom connector** for demo; prepare Community directory materials (docs, privacy, icon, populated test tenant).
5. Measure: tool invocations, hold creates, paid conversions, support tickets — **not** vanity “rank.”
6. Parallel cheap GEO pass on property pages.
7. Optional: ChatGPT app submission **only if** OpenAI commerce counsel/review signal is acceptable.

**Out of scope**
- Multi-host marketplace, Airbnb undercutting automation, Gemini “directory,” in-chat capture of PANs, paid ads experiments as core bet, Print Junkie SEO myth transfer claims.

### KEEP / KILL / DEFER rubric for Dick
| Signal | Score |
|---|---|
| Eric has working direct checkout + clean availability API | KEEP pilot |
| Investors require “#1 in ChatGPT over Airbnb” as success metric | KILL / reframe |
| No PMS cooperation / spreadsheet inventory only | DEFER until SoT |
| Want multi-tenant SaaS immediately | DEFER product; KEEP single-host learnings |
| Only want Gemini parity | DEFER / KILL that leg |

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | Claude custom connectors are remote MCP servers added by URL; directory listing is separate and required for Suggested Connectors | E1 E2 E3 | high | supported |
| C2 | Claude directory rejects connectors that transfer money/crypto/financial assets | E4 E5 | high | supported |
| C3 | Booking.com’s Claude connector searches lodging and completes booking off-Claude via links | E6 | high | supported |
| C4 | OpenAI plugins/apps currently may conduct commerce only for physical goods; external checkout is the GA monetization path | E7 E8 | high | supported |
| C5 | OpenAI directory submission requires test account, 5 positive + 3 negative cases, video/release notes; owners need verification / Apps Management Write | E9 | high | supported |
| C6 | ChatGPT contextual app suggestion exists as a product behavior; exact ranking is not fully public | E10 E11 | medium | partial |
| C7 | Gemini allows adding custom MCP server URL as a custom app; enterprise has custom MCP connectors — not shown as open public directory parity | E12 E13 | medium | supported |
| C8 | Hospitable provides official STR PMS MCP aimed at host operations; listed in ChatGPT plugins; Claude via custom URL | E14 E15 | high | supported |
| C9 | Vacation rental stays are explicitly classified as allowed ChatGPT “physical goods” | — | low | unsupported |
| C10 | A small Scottsdale host can reliably outrank Booking.com/Expedia apps inside ChatGPT/Claude suggestions | — | low | unsupported |
| C11 | Print Junkie ChatGPT visibility proves an MCP will make STRs “first in line” | E16 | low | unsupported (category error) |
| C12 | Perplexity dump is broadly directionally right on Claude directory vs custom, money restriction, and ChatGPT apps — with citation noise and over-certainty on discovery | E17 + matrix | medium | partial |

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / note |
|---|---|---|---|---|
| E1 | Claude: directory vs custom | https://claude.com/docs/connectors/building/directory-vs-custom | curl MD 200, 2026-10-28 | Directory connectors eligible for Suggested Connectors; custom never suggested; ranking usage-based; per-customer URLs need separate listings |
| E2 | Claude: connectors directory | https://claude.com/docs/connectors/directory | curl MD 200 | Same catalog; Suggested Connectors; Verified vs Community |
| E3 | Claude: verification labels | https://claude.com/docs/connectors/verification | curl MD 200 | Community default after auto scan; Verified not a security audit |
| E4 | Claude: review criteria | https://claude.com/docs/connectors/building/review-criteria | curl MD 200 | Unsupported: transfer money/crypto/financial assets; split R/W tools; annotations; test credentials |
| E5 | Anthropic Software Directory Policy | https://support.claude.com/en/articles/13145358-anthropic-software-directory-policy | curl HTML 200 | Prohibits software that transfers money/financial assets or executes financial transactions autonomously without HITL safeguards (policy text) |
| E6 | Booking.com connector about | https://developers.booking.com/mcp/booking-connector/about | curl HTML 200 | Search/filter in Claude; open on Booking.com to complete booking; payments not in Claude |
| E7 | OpenAI plugin guidelines — commerce | https://developers.openai.com/plugins/plugin-guidelines | curl MD 200 | Commerce only for physical goods; external checkout; Instant Checkout beta select partners; prohibited categories include money transfers and abusive travel services |
| E8 | OpenAI monetization / checkout | https://developers.openai.com/plugins/build/monetization | curl MD 200 | External checkout recommended GA; physical-goods framing; payment sheet private beta |
| E9 | OpenAI plugin submission | https://developers.openai.com/plugins/deploy/submission | curl MD 200 | Test account; 5 positive / 3 negative cases; publish after approval; Apps Management Write |
| E10 | OpenAI introducing apps / submit apps | https://openai.com/index/introducing-apps-in-chatgpt/ ; https://openai.com/index/developers-can-now-submit-apps-to-chatgpt/ | web_search snippets (HTML 403 in container) | Apps in ChatGPT; directory submissions (Dec 2025 announcement) |
| E11 | OpenAI “what makes a great ChatGPT app” | https://developers.openai.com/blog/what-makes-a-great-chatgpt-app | curl MD 200 | Design for conversation/discovery (qualitative) |
| E12 | Gemini custom apps MCP | https://support.google.com/gemini/answer/17209137 | curl HTML 200 | Add custom app with MCP server URL in Gemini web app |
| E13 | Gemini Enterprise custom MCP | https://docs.cloud.google.com/gemini/enterprise/docs/connectors/custom-mcp-server/set-up-custom-mcp-server | curl HTML 200 | Admin/enterprise custom MCP data store path |
| E14 | Hospitable MCP help | https://help.hospitable.com/en/articles/14424057-connect-an-ai-agent-to-hospitable-using-mcp | curl HTML 200 | ChatGPT plugin; Claude custom URL https://mcp.hospitable.com/mcp; paid Claude note; ops data + guest messages |
| E15 | Hospitable MCP product | https://hospitable.com/features/mcp | curl HTML 200 | “First official MCP server in short-term rentals”; host/pro plans |
| E16 | Lead SMS screenshots | pack PNGs | vision_analyze 2026-10-28 | Eric: Scottsdale STRs; direct vs Airbnb/VRBO; Claude/chat/Gemini exposure; investors care |
| E17 | Perplexity export | pack `bookings_app.md` | local file | Hypothesis dump; many footnotes; some bad citations (e.g. paid-plan footnote pointing at Apache Superset) |
| E18 | Claude submission guide | https://claude.com/docs/connectors/building/submission | curl MD 200 | HTTPS remote only; OAuth or no auth; listing materials; paid plan to submit |
| E19 | Pack HANDOFF / RESEARCH-REQUEST | pack | local | Acceptance criteria and non-goals |

**Extraction path note:** Primary path = `curl` to official `*.md` developer docs (Firecrawl keyless 403). Google/Hospitable/Booking = HTML strip. OpenAI marketing HTML often 403 — supplemented with search snippets labeled accordingly.

---

## Perplexity dump verification (bookings_app.md)

| Perplexity claim | Verdict | Notes |
|---|---|---|
| Claude custom connector = remote MCP; add via Customize → Connectors | **Supported** | Matches Anthropic docs |
| OAuth or headers for custom connectors | **Supported** (auth doc); directory prefers OAuth or public no-auth | |
| Paid plan required for custom connectors on web | **Supported** by Hospitable + Anthropic submission “paid plan”; one Pplx footnote wrongly cited Superset — **citation noise** | |
| Directory vs custom; Suggested Connectors only for directory | **Supported** | E1 |
| Ranking usage-based; eligibility ≠ guarantee | **Supported** | E1 |
| ChatGPT apps on MCP / Apps SDK; directory submit | **Supported** | E9 E10 |
| ChatGPT mid-chat app suggestions (Zillow/travel examples) | **Partial** | Product real; examples often secondary blogs |
| Gemini murkier / no clear public submit+suggest path | **Supported** (this research) | E12 E13 |
| Claude: no money transfer via connector → reservation + external checkout | **Supported** | E4 E5; Booking.com pattern E6 |
| Tool design: split read/write, annotations, test account | **Supported** | E4 E18 |
| “No installation needed” for remote MCP | **Misleading** | No *app store install binary*, but **connect/directory accept** still required; discovery still hard |
| Build one MCP and submit both directories → guests find & book | **Overstated** | Technically plausible path; discovery/commerce risks underplayed |
| Footnote quality | **Weak** | Multiple irrelevant URLs in export — never trust Pplx footnotes without re-fetch |

---

## gaps

1. OpenAI Developer Apps Terms full HTML (money section 1.x) not retrieved (403) — guidelines MD used instead.
2. Whether lodging is “physical goods” under OpenAI review — **needs human/policy clarification before ChatGPT commerce spend**.
3. Live directory category density for “vacation rental host” apps — not manually browsed in authenticated ChatGPT/Claude UI this run.
4. Eric’s actual PMS, current direct-book stack, channel mix, ADR/occ — unknown.
5. ChatGPT Ads product availability/eligibility for STR — mentioned in SMS, not verified.
6. Exact Suggested Connectors trigger features (embedding? query classifier?) — undisclosed.
7. Gemini consumer UI evolution beyond support article — monitor only.
8. Legal: short-term rental licensing, occupancy tax collection on direct book in Scottsdale/AZ — out of scope but pilot-blocking in production.

---

## confidence

```yaml
level: medium
rationale: |
  High confidence on Claude connector mechanics, money ban, Booking.com pattern,
  OpenAI physical-goods commerce framing, submission artifacts, Hospitable ops MCP.
  Medium on ChatGPT suggestion economics and Gemini trajectory.
  Low on lodging commerce approval and any ranking/ROI promise.
  Primary OpenAI marketing pages blocked (403) in this environment; MD docs solid.
```

---

## recommended_next_actions

1. **Dick:** KEEP narrow pilot vs KILL “beat Airbnb in ChatGPT” narrative; decide single-host retainer vs product.
2. **Intake Eric (non-Geraldo):** PMS name, direct-book URL today, who owns calendar SoT, must-have guest languages, success metric investors will accept.
3. **If KEEP:** spawn implementation card for **S0+S1 spine** (not full apps submission first) — su-v2 / tricon-web when chosen.
4. **Policy checkpoint:** written note on OpenAI lodging commerce before ChatGPT directory burn.
5. **Do not** pitch Gemini parity; mention custom-MCP only as footnote.
6. **Optional follow research:** authenticated walkthrough of ChatGPT app directory travel category + Claude connector search for “vacation”/“rental” density (browser login session).

---

## deliverables map (request 01–06)

| Req | Section |
|---|---|
| 01 Platform matrix | §01 / A |
| 02 Architecture + tools + payments | §02 / B |
| 03 Discovery reality | §03 / C |
| 04 Comps | §04 / D |
| 05 Effort | §05 / E |
| 06 Kill risks + pilot | §06 / F |
| SOURCES | evidence_index + pack paths |

---

## pack provenance

- `/srv/scratch/handoffs/20261028-bookings-app/HANDOFF.md`
- `/srv/scratch/handoffs/20261028-bookings-app/00-RESEARCH-REQUEST.md`
- `/srv/scratch/handoffs/20261028-bookings-app/00-CONTEXT.md`
- `/srv/scratch/handoffs/20261028-bookings-app/bookings_app.md`
- Screenshots: `Screenshot_20261028-133639.png`, `Screenshot_20261028-133644.png`
- Scratch extracts: `/srv/scratch/t_af826421-research/sources/`

---

*End of report — research-bookings-app-t_af826421*
