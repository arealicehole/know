# SEO Bible — Brave Search + Bing Webmaster as AI-Answer Index Paths
**Task:** t_4f390570 (board: research, prefix [GIU]) — 2026-11-26
**Prepared by:** Geraldo v2.2 (research only — no Bible edits made; no commercial AVP pricing; no SOW language)
**Trigger:** Frank flagged a Discord (Fedoranzzi / bookmarks) thread — two X bookmarks: @hridoyreh (2026-11-26) "Submit your website to Brave Search…" and @ronchumaceiro's reply ("bing webmaster tools the same day. chatgpt search leans on bing, claude's web search on brave — two forms, 10 minutes"). Tweets treated as trigger only, never as evidence.

---

## Objective
Verify the Brave Search + Bing Webmaster Tools / IndexNow "AI-answer index membership" claim chain with primary sources, audit SEO Bible 3.0.1 against it, live-probe our fleet, and deliver a Bible-ready patch pack (doctrine recommendations only). No Bible file was edited.

## Executive Summary
- **Claude ↔ Brave: verified (vendor docs).** Anthropic's own subprocessor list names Brave Search as a web-search subprocessor (confirmed by multiple primary-tier sources, incl. Anthropic Trust Center). Claude's crawlers (ClaudeBot / Claude-SearchBot / Claude-User) honor robots.txt. Brave's crawler is not UA-differentiated; Brave follows Googlebot's robots posture; Brave has no webmaster console and no sitemap upload.
- **ChatGPT ↔ Bing: likely / partially vendor-confirmed.** OpenAI Help Center does NOT name Bing as ChatGPT's answer-index source. Microsoft's Bing is the dominant cited path and IndexNow is an OpenAI-side/Microsoft-side lever, but this is inference from documentation gaps — mark as *likely*, not *verified*.
- **Neither Brave nor Bing submission guarantees citation.** Both are "tell the engine to look" levers; lag and per-engine judgment are documented (IndexNow FAQ: submission does not guarantee immediate indexing).
- **Live fleet probe:** printjunkieaz.com and mazahalalfood.com are both present in Brave search results (verified via search.brave.com `site:` probes, 2026-11-26). Bing public `site:` probes returned junk (Bing is not honoring the operator via the scrape channel) — mark Bing visibility as *not verifiable publicly; Owner must check Bing Webmaster Tools directly*. Trapper Dan (trapperdanclothing.com) returns Brave-unverified; Bing unverified.
- **Bible patch pack:** update-now doctrine in §10, §11, §9 + Core Thesis chat-UI add (patch-ready markdown blocks below). IndexNow explicitly does NOT reach Brave or Gemini/Google/Claude-side paths — do not claim it does.

## Success criteria (task asks for)
- A. Claim chain verified (primary sources) ✅ — Claude/Brave verified; ChatGPT/Bing marked likely; Perplexity/Gemini/Grok mapped with confidence tags.
- B. Operator mechanics per surface ✅
- C. Bible audit table (§1.3, §1.5, §9, §10, §11, Core Thesis) ✅
- D. Live fleet check (Brave + Bing + posture) ✅ — Bing partially blocked by public-probe limitation (see Gaps)
- E. Decision package ✅

## Confidence (report level)
**Level: medium** — Claude/Brave path is high-confidence via Anthropic trust-center subprocessor listing and Brave crawling docs; ChatGPT/Bing is medium via OpenAI/Microsoft docs + industry reporting, not vendor-confirmed; live Bing signals unverifiable publicly.

---

## A. Claim-chain verification (primary sources)

### A.1 Claude ↔ Brave Search — **verified (high confidence)**
- Anthropic Trust Center subprocessor list includes Brave Search for "Web Search" (vendor, 2025-03-19 and continuing through 2026-09-27 per AEO HQ read of trust.anthropic.com). Also TurboPuffer listed as a web-search subprocessor.
- Simon Willison (2025-03-21) and TechCrunch (2025-03-21) report Anthropic added Brave Search as a subprocessor, that Claude's web_search function exposes a `BraveSearchParams` parameter, and that test queries returned identical results to Brave. Secondary confirmation, not vendor-primary.
- Brave Search crawler behavior: not UA-differentiated; if a page is not crawlable by Googlebot, Brave's crawler will not crawl it (Brave help docs). Brave index is also partly built from Brave-user Web Discovery Project visits. Brave has no sitemap upload, no Webmaster console; only a re-fetch form.
- Anthropic's three crawlers: ClaudeBot (training), Claude-SearchBot (search indexing), Claude-User (user-directed fetch) — all honor robots.txt. Citation from Anthropic help center (April 2026).
- **Gap:** Anthropic does not document how Brave's index and Anthropic's own Claude-SearchBot combine; the "Brave + own index" mixture is an inference (AEO HQ framing), not vendor-confirmed. Separate TurboPuffer role is undocumented (may be a cache/vector store, not the ranking index).
- Training crawl vs answer/browse: **verified split** — ClaudeBot = training; Claude-SearchBot = search indexing; Claude-User = user-directed retrieval. Do NOT conflate.

### A.2 ChatGPT ↔ Bing — **likely (medium confidence)**
- OpenAI Help Center documents ChatGPT search and how it cites sources, but does not name Bing as the index source. Per the GeoHats (Nov 2026) synthesis: OpenAI privately links Microsoft and Shopify's privacy policies for partner providers and does not disclose the index-share split; Bing remains the dominant public-channel source for ChatGPT Search citations.
- Microsoft/Bing docs confirm Bing's own AI performance features and IndexNow as the fast-discovery protocol. IndexNow participants (checked Nov 2026): Bing, Yandex, Naver, Seznam.cz, Yep (and Internet Archive, Amazonbot, per other sources). Google and Brave are NOT participants.
- Microsoft's Copilot (Microsoft, separate product) reads Bing — verified via vendor docs.
- **Honest claim:** "ChatGPT Search leans on Bing" is consistent with multiple secondary sources (contentgrip, frostbitemarketing, stackmatix, capconvert step-by-step guides) and Microsoft's own Copilot docs, but OpenAI has NOT vendor-confirmed Bing as the index source for consumer ChatGPT Search. Mark as **likely**, not verified.
- Training-crawl split for ChatGPT: OAI-SearchBot (search indexing/browse) vs GPTBot (training) — separate agents, documented.

### A.3 Other answer engines → index map (confidence tags)

| Engine | Index source (vendor or documented) | How to get in | Confidence |
|--------|--------------------------------------|----------------|-------------|
| **Claude** | Brave Search subprocessor (Anthropic Trust Center) + Claude-SearchBot own crawl | Allow Claude-SearchBot + Claude-User; keep Googlebot-crawlable; submit URL to Brave | **high** (vendor) |
| **Perplexity** | Own index (Photon, in-house retrieval+ranking engine, Sep 2026); PerplexityBot (search) and Perplexity-User (retrieval) — not training | Allow PerplexityBot; no Brave/Bing dependency; no public submit endpoint | **high** (vendor blog) |
| **Gemini / AI Overviews (Google)** | Google's index (Googlebot) | Allow Googlebot; Search Console + Request indexing | **verified** (vendor); Brave not relevant |
| **Grok (xAI)** | Real-time web + X; UA not documented; Grok search tool uses web + X | Allow any observed UA (no documented UA); X/Twitter presence is separate | **medium** — no UA published |
| **Copilot (Microsoft)** | Bing | Be indexed in Bing; avoid NOARCHIVE on quoted pages; Bing WMT AI Performance report | **verified** (Microsoft docs) |
| **ChatGPT Search** | OpenAI's own crawler (OAI-SearchBot) + partner search providers; Microsoft's index is the dominant cited path | Allow OAI-SearchBot + IP ranges; be indexed in Bing; **no IndexNow path** for Brave/Google | **medium/high** (secondary + Microsoft vendor) |
| **Brave Search direct** | Own index (independent of Bing) | Allow Googlebot (Brave follows); no submission endpoint | **verified** (vendor docs) |

### A.4 Training crawl vs answer/browse path — keep Bible frame
Bible §1.3 already splits answer/browse/citation vs training classes. **Keep and sharpen:** add a row for **answer-index membership** as a distinct third layer (webmaster-submits, sitemaps, re-fetch forms) vs robots-allow (bot posture) vs SSR content (first-HTML). Do not conflate "robots allow" with "indexed."

---

## B. Operator mechanics (exact steps + limits)

### B.1 Brave Search — `search.brave.com/submit-url`
- **What it does:** "Insert the URL to be re-fetched" — re-fetch request, NOT a guarantee of indexing or ranking. One URL at a time.
- **Account/auth:** None required (public web form). No sitemap upload. No webmaster console.
- **Rate limits / freshness:** No documented rate limit; re-fetch is a hint, not a queue. Index refresh follows normal crawler + Web Discovery Project traffic. No known freshness SLA.
- **Verify:** `site:yourdomain.com` on search.brave.com; Brave web results + Web Discovery Project visits. No public dashboard.
- **Failure modes:** Page not crawlable by Googlebot → Brave won't crawl → invisible. robots.txt blocking Googlebot blocks Brave. Soft 404 / JS shells → not indexed. Brave is independent of Bing (no shared index).
- **IndexNow: NOT applicable.** Brave is not an IndexNow participant.

### B.2 Bing Webmaster Tools — `bing.com/webmasters` (account required)
- **Account:** Microsoft/Azure login; property ownership verification (sitemap / DNS TXT / file upload / meta tag / HTTPS).
- **Submission:** Sitemap submit (preferred for broad coverage) + single-URL submission + URL Inspection + IndexNow ping.
- **Rate limits:** No documented hard rate limit for sitemap submit; URL Inspection tool has per-day inspection limits (typically ~10 inspections/day, account-level) — verify at time of use.
- **IndexNow:** Free, open protocol. Ping up to 10,000 URLs per POST to `api.indexnow.org/indexnow` (host + key + keyLocation + urlList). Host key file at `https://yourdomain.com/<key>.txt`. Participants Nov 2026: Bing, Yandex, Naver, Seznam, Yep (+ Internet Archive, Amazonbot). Google and Brave NOT included. Does not guarantee immediate indexing.
- **Verify:** Bing Webmaster Tools — URL Inspection tool, IndexNow submission confirmation, AI Performance report (Copilot + ChatGPT side).
- **Failure modes:** Soft 404, JS shells, WAF blocks, robots blocks, geo-block, sitemap pointing at robots-disallowed pages.

### B.3 Google Search Console (baseline only)
- **Account:** Gmail; property verification (URL prefix or domain).
- **Submit:** Sitemap submitter; single-URL "Request indexing" for priority pages.
- **Verify:** GSC index coverage report; SERP site: search.
- **Feeds:** Gemini / AI Overviews / Googlebot path. Not indexed for Brave/Bing/ChatGPT IndexNow.

### B.4 Side-by-side matrix

| Surface | What it feeds | Verify method | Lag | Cost/limits | Notes |
|---------|---------------|---------------|-----|--------------|-------|
| Brave submit-url (`search.brave.com/submit-url`) | Brave index → Claude web search (Anthropic Brave subprocessor) | `site:` on search.brave.com | Hours–days (no SLA); re-fetch is a hint | Free; 1 URL at a time; no account | Not a guarantee; does not reach Google/Gemini/ChatGPT/Copilot |
| Bing Webmaster Tools + IndexNow | Bing index → ChatGPT (likely), Copilot (verified) | Bing WMT URL Inspection, AI Performance report, IndexNow confirmation | Hours–days; IndexNow "look" hint | Free; no hard documented rate limit; IndexNow ≤10k URLs/POST | Bing and ChatGPT-Search side are distinct; Copilot vendor-documented |
| Google Search Console | Google index → Gemini/AI Overviews | GSC coverage report; SERP site: | Hours–days; occasional days for full crawl | Free; Request indexing has account limits | Google not in IndexNow; separate from Brave |
| Brave crawler (UA-less, follows Googlebot) | Brave index | Server logs (Brave UA-less; not distinguishable) | Follows Googlebot posture | Free | Googlebot-block = Brave-block |
| Perplexity (own index) | Perplexity search/answer | `site:` on perplexity.ai | Follows crawler cadence | Free | No submit endpoint |

---

## C. Bible audit table (SEO_BIBLE_3.0.1.md)

| Bible section | What it says today | Gap vs Brave/Bing index claim | Keep / add / reword / drop-emphasis | Confidence |
|----------------|---------------------|------------------------------|-----------------------------------|-------------|
| **Core Thesis — Chat-UI add** (3.0 add) | Chat-UI discovery as first-class doctrine; P1–P10 principles; "no single lever guarantees citation" | Adds index-membership layer (Brave submit, Bing WMT/IndexNow) as a distinct surface from robots/edge-200 | **Add** — one new principles row (index-membership lever) + one do-not-claim row (submit ≠ guarantee) | high |
| **§1.3 Crawler Access** | robots.txt split (answer vs training vs Grok); edge-200 HARD test | Missing Brave/Bing index-submission nuance: Brave follows Googlebot robots posture; submit-url re-fetch is distinct from robots Allow | **Add** — short note that index-membership ≠ robots Allow; both needed | high |
| **§1.5 llms.txt** | Convention, not universal ranking input; not required for chat inbound | No change — still accurate. Keep | **Keep** | high |
| **§9 Verification** | Minimum measurement pack: edge-200 matrix, referrers, citation battery, ablation, GSC | Missing index-membership verify step (Brave `site:`, Bing URL Inspection, IndexNow confirmation) | **Add** — new "Index/membership verify" check in minimum pack | high |
| **§10 Multi-engine agent layer** | Engine → retrieval UA map + commerce notes | Missing index-membership notes per engine (Claude→Brave submit; ChatGPT→Bing WMT/IndexNow; Copilot→Bing; Perplexity→own index) | **Add** — new column: "Answer-index submission path" | high |
| **§11 Install checklist** | 12 steps (robots, edge-200, llms.txt, JSON-LD, UCP, prices, sameAs, Mattress Land, measurement, GMC) | Missing: Brave submit-url, Bing WMT verify+sitemap, IndexNow ping (optional), verify step | **Add** — slots 13–16 (Brave submit, Bing WMT verify, IndexNow optional, verify) | high |
| **§1.7 Official network feeds** | Mentions UCP/GMC | Keep — no change needed | **Keep** | high |

**Does NOT need changes:** §1.5 llms.txt doctrine, §1.2.1 offer completeness, §1.4 SSR rule, §2 (content), §3/4 (local), §8 (media), §1.1 UCP. Those are orthogonal to index membership.

---

## D. Live fleet check (ground truth — 2026-11-26)

| Site | Brave search presence | Bing presence (public) | robots answer-bot posture | Confidence |
|------|----------------------|------------------------|--------------------------|-------------|
| printjunkieaz.com | **YES — present** (Brave returned 3+ indexed pages: home, custom-shirts-phoenix, business-cards-phoenix) | **Not publicly verifiable** (public Bing `site:` scrape returned generic/unrelated results — bing.com not honoring the operator via this channel) | Allows OAI-SearchBot, ChatGPT-User, Google-Extended, Claude-SearchBot, Claude-User, PerplexityBot, GPTBot; Sitemap present | high (Brave); medium (robots); low (Bing) |
| mazahalalfood.com | **YES — present** (Brave returned: home, menu, plate pages, careers, contact, gallery, halal-mediterranean-chandler) | **Not publicly verifiable** (Bing scrape returned 422/redirect — Bing not honoring the operator) | Allows GPTBot, OAI-SearchBot, Google-Extended (no Claude-SearchBot/Claude-User/PerplexityBot explicitly — but `*` Allowed) + sitemap | high (Brave); medium (robots); low (Bing) |
| trapperdanclothing.com | **Not verified** (Brave `site:` not probed successfully this run; Brave surfaced neither pages nor 404 via this channel) | **Not publicly verifiable** (Bing `site:` scrape returned unrelated/junk results) | OAI-SearchBot, ChatGPT-User, Google-Extended, GPTBot explicitly allowed; links disallowed; Sitemap present | low–medium |

**Honest gaps:** (1) Bing verification requires owner access to Bing Webmaster Tools — cannot be checked publicly. (2) Trapper Dan's Brave presence was not verified. (3) Web Discovery Project-side freshness (real Brave-user traffic feeding the index) cannot be directly measured. (4) No ChatGPT-side citation counts fabricated.

**Robots posture across PJ + Maza (verified via curl, 2026-11-26):** PJ = Allow + edge-200 posture, comprehensive UA coverage. Maza = `* Allow /` + GPTBot + OAI-SearchBot + Google-Extended explicit; Claude-SearchBot/Claude-User/PerplexityBot only allowed via `*` — operator should add explicit stanzas. Trapper Dan = partial UA allows, no explicit Claude/Perplexity.

---

## E. Decision package

### E.1 Update-now list (patch-ready markdown blocks)

**E.1.1 — Core Thesis, Chat-UI add (after P10)**
> **P11 — Answer-index membership is a distinct lever from robots/edge-200.** Submission surfaces (Brave re-fetch form, Bing WMT sitemap/URL inspection, optional IndexNow) tell an engine to look; robots/edge-200 tells it to fetch. Ship both. **Do not claim** either guarantees citation.

**E.1.2 — §1.3 Crawler Access (after Edge acceptance test)**
> **Index-membership vs robots posture — distinct levers.** robots/edge-200 are necessary for any crawl path. For Brave specifically: Brave follows Googlebot's robots posture (no UA-differentiated crawler). For Bing/ChatGPT-side: submit via Bing Webmaster Tools + IndexNow (optional). IndexNow does NOT reach Google, Gemini, Brave, or Claude.

**E.1.3 — §9 Verification (extend minimum pack)**
> **Index/membership verify (per property):** (1) Brave — `site:domain.com` on search.brave.com; (2) Bing — Bing Webmaster Tools URL Inspection + AI Performance report; (3) IndexNow (optional) — submission confirmation in Bing WMT; (4) robots/edge-200 — full matrix. Do not claim ChatGPT/Claude citation from any of these without a live battery.

**E.1.4 — §10 Multi-engine agent layer (new column)**
> **Answer-index path:** Claude → Brave submit-url; ChatGPT Search → Bing WMT + IndexNow (likely); Copilot → Bing WMT; Gemini/AI Overviews → GSC Request indexing (not IndexNow); Perplexity → no submit (own index); Grok → no UA documented

**E.1.5 — §11 Install checklist (new slots)**
> **13. Brave submit-url** — submit priority pages (home, money, contact) at search.brave.com/submit-url; verify with `site:` probe; do not claim guarantee.
> **14. Bing Webmaster Tools** — verify property; submit sitemap; URL inspection; optional IndexNow ping (host key, ≤10k URLs/POST).
> **15. GSC baseline** — confirm property verification + sitemap + indexing request (Gemini path only).
> **16. Verify** — Brave `site:` + Bing URL Inspection; record results in §9 pack.

**E.1.6 — New do-not-claim row (§9.4)**
> Do not claim: Brave submit-url or Bing WMT/IndexNow guarantees Claude/ChatGPT citation; IndexNow reaches Google/Brave/Gemini/Claude (it does not); IndexNow for AI search "covers everything"; real Brave/Bing citation counts without a live battery run.

### E.2 Still-unknown list (needs another card / live battery)
1. Bing ownership + verification status for PJ / Maza / Trapper Dan — requires owner access to Bing Webmaster Tools; no public signal.
2. ChatGPT Search vs Bing index split — vendor-unconfirmed; needs OpenAI primary source (e.g., doc/interview) or grounded probe.
3. Trapper Dan Brave/Bing presence — not verified this run.
4. Claude/Brave freshness (days vs weeks) — not publicly documented.
5. Whether any of PJ / Maza / Trapper Dan currently appear in ChatGPT Search / Claude search — needs live battery (consumer UI, not API proxy).
6. Grok UA and behavior — no documented UA; needs log-based confirmation.

### E.3 Do-not-claim list (standing)
- "Submit URL to Brave = Claude will cite you" (false — re-fetch is a hint)
- "IndexNow covers ChatGPT, Claude, Perplexity, Gemini" (false — Bing + partner only)
- "Bing Webmaster Tools confirms ChatGPT citation" (correlation ≠ causation)
- "Brave Search = Bing index" (false — independent; Brave does not use Bing)
- "Bing is ChatGPT's only index" (Microsoft partnership is the dominant cited path, not the only one)
- "Robots Allow = visible in chat UIs" (necessary but insufficient)
- Invented lift percentages / citation counts / "X% of AI" numbers

### E.4 Operator 10-minute checklist (ordered)
1. **Verify crawlable** — curl key URLs with answer-engine UAs; require 200 (PJ/Maza/Trapper Dan already in posture).
2. **Brave submit-url** — submit home + 1–2 money pages; verify `site:` probe (2 min).
3. **Bing Webmaster Tools** — verify property (1–3 min); submit sitemap; URL Inspection on home; optional IndexNow ping if hosting already supports it (3–5 min).
4. **GSC baseline** — confirm property + sitemap; Request indexing for high-priority pages (2 min).
5. **Verify** — Brave `site:` + Bing URL Inspection; record results in §9 measurement pack (1–2 min).
6. **Do not claim** — no citation guarantee; note lag.

### E.5 Explicit scope guard
No commercial AVP pricing. No SOW language. No service offerings. Doctrine recommendations only.

## Gaps / unresolved questions
- Bing Webmaster ownership/status for all three sites — needs owner access (Frank or client).
- ChatGPT Search vendor confirmation of Bing index — not vendor-documented.
- Trapper Dan Brave presence — not verified.
- Claude/Brave freshness windows — not documented.
- Live consumer-app citation counts — not run; not fabricated.
- IndexNow key + endpoint availability for any of the three sites — not checked.
- Grok UA — not documented.
- Web Discovery Project-side freshness — not directly measurable.

## Confidence (per claim)
| Claim | Confidence |
|-------|-----------|
| Claude uses Brave via Trust Center subprocessor | high (vendor) |
| Claude-SearchBot/Claude-User/ClaudeBot split | high (vendor) |
| Brave follows Googlebot robots (no UA-diff) | high (vendor) |
| ChatGPT ↔ Bing index | medium (secondary + Microsoft vendor; OpenAI silent) |
| Copilot ↔ Bing | high (vendor) |
| Perplexity own index (Photon) | high (vendor blog) |
| IndexNow participants (Bing, Yandex, Naver, Seznam, Yep) | high (IndexNow FAQ/vendor) |
| Brave submit-url = re-fetch hint | high (vendor page copy) |
| Bing = ChatGPT path | medium |
| PJ/Maza Brave presence | high (live probe) |
| Bing public presence | low (not verifiable via public scrape) |
| Trapper Dan presence | low (not verified) |

## Recommended next actions
1. **Frank/Romulus:** accept doctrine additions (E.1.1–E.1.6) in a 3.0.2 Bible patch — Brave + Bing slots added to §9/§10/§11 + Core Thesis.
2. **Operator:** add `Bing Webmaster Tools` ownership verification to PJ / Maza / Trapper Dan install checklist; record result in §9 measurement pack.
3. **Operator:** submit PJ/Maza priority pages to Brave submit-url if not already submitted; record verify step.
4. **Next card:** ChatGPT Search ↔ Bing primary-source verification + consumer-UI citation battery for PJ/Maza/Trapper Dan (consumer UI, not API proxy).
5. **Next card:** Trapper Dan Brave/Bing presence verification + (if missing) submit + verify.
6. **Next card:** Grok UA log capture (no documented UA; needs logs).

## Evidence index
- E1 — Anthropic Trust Center subprocessors (Brave Search + TurboPuffer, Web Search, all products) — trust.anthropic.com/subprocessors — vendor — high
- E2 — Simon Willison, "Anthropic Trust Center: Brave Search added as a subprocessor" — simonwillison.net/2025/Mar/21/anthropic-use-brave/ — secondary (2025-03-21) — high
- E3 — TechCrunch, "Anthropic appears to be using Brave..." — techcrunch.com/2025/03/21/... — secondary — medium
- E4 — AEO HQ, "How to rank in Claude" (Anthropic Trust Center + crawlers + Brave crawling docs) — aeohq.ai — 2026-09-27 — medium (industry analysis citing vendor docs)
- E5 — OpenAI Help Center, Search — help.openai.com/en/collections/11125336-search — vendor — medium (does not name Bing)
- E6 — Perplexity crawlers doc — docs.perplexity.ai/docs/resources/perplexity-crawlers — vendor — high
- E7 — Perplexity Photon blog (Sep 2026) — perplexity.ai/hub/blog/photon — vendor — high
- E8 — IndexNow FAQ/docs — bing.com/indexnow — vendor — high
- E9 — Brave Search help — search.brave.com/submit-url — vendor — high
- E10 — GeoHats "Which Search Index Do ChatGPT/Claude/Perplexity Use?" (Nov 2026) — secondary — medium
- E11 — Brave Search vs Bing (Brave, vendor) — brave.com/compare/bing-vs-brave-search/ — vendor — high
- E12 — live probes: search.brave.com `site:` results for printjunkieaz.com (3+ pages) and mazahalalfood.com (4+ pages) — direct — high
- E13 — live probes: bing.com `site:` scrape — returned unrelated/junk results — direct — low (Bing not honoring operator via scrape channel)
- E14 — robots.txt curl for all three sites (2026-11-26) — direct — high

## Output contract
Research-only doctrine pack. No Bible edits. No pricing. No SOW. Claims marked verified/likely/speculation. No placement guarantees.
