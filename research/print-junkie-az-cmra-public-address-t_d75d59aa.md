# Print Junkie AZ — CMRA / Public Business Address Options (Phoenix–Tempe)

**Task:** `t_d75d59aa`  
**Project:** Print Junkie AZ / Tricon Web  
**Date:** 2026-11-14  
**Researcher:** Geraldo (GIU)  
**Audience:** Frank (decision) + Tricon implementation

---

## Frank TLDR (read this first)

1. **Do not put a CMRA / virtual mailbox / PO Box on the public Google Business Profile pin.** Google’s own guidelines say remote mailboxes and virtual offices you don’t operate from are **not acceptable**. For a home-based / go-to-customer print shop, run GBP as a **service-area business and hide the address**.
2. **Still get a CMRA street address** for everything else: website footer/contact, LocalBusiness schema, Yelp/Facebook/directories, invoices, vendor forms, package receiving — so home stays private and NAP can match.
3. **Best default pick for PJ AZ:** open a **The UPS Store private mailbox in Tempe** (preferred: **3116 S Mill Ave, Tempe AZ 85282**, store #5040, 480-889-0382). Real local street address, all-carrier packages, walk-in pickup, Form 1583 on site. Expect roughly **~$15–40/mo** depending on box size/term (franchise-set; call for exact quote) + setup/key fees.
4. **Address format on public pages:** always include **PMB or #** (never fake “Suite” unless the CMRA literally assigns Suite and USPS format allows it). Example: `3116 S Mill Ave PMB 123, Tempe, AZ 85282`.
5. **Same address everywhere** = website + schema + citations + invoices + social. **Exception:** GBP public display stays **hidden** (service areas only). Do not invent a second “marketing address.”

---

## objective

Choose a compliant, low-cost public mailing address strategy for Print Junkie AZ (Phoenix/Tempe/Mesa area) that:

- Keeps a home address off public web/citation surfaces
- Supports stable NAP for local SEO (non-GBP channels)
- Does **not** create Google Business Profile suspension risk
- Fits a small print / apparel / custom merch business that is primarily service-area / online rather than a walk-in storefront

---

## summary

Print Junkie needs **two different address behaviors**, not one magic address:

| Surface | What to use | Why |
|---|---|---|
| **Google Business Profile (public)** | Service-area profile; **hide address**; list PHX metro service areas | Google disallows PO Boxes, remote mailboxes, and unstaffed virtual offices as the profile location |
| **Website, schema, Yelp, FB, directories, invoices, shipping labels return address** | One **CMRA street address** with PMB/# | Professional AZ street address, privacy, package receipt, citation consistency |
| **Banks / some processors / registered agent** | Often **reject** CMRA; may need home or true office | KYC / CMRA detection (USPS DPV CMRA table). Plan separately from SEO address |

**Recommended architecture:** Tempe UPS Store CMRA as the single public mailing NAP + GBP as service-area with address hidden.

**Not recommended as primary AZ NAP:** out-of-state virtual mailbox brands with no Tempe/Phoenix street (e.g. PhysicalAddress.com’s published footprint is multi-state but **not Arizona**). Fine for pure remote mail scanning; weak for “local Tempe print shop” citations.

---

## plan

| task_id | title | status |
|---|---|---|
| T1 | Compare CMRA vs PO Box vs virtual mailbox vs virtual office | done |
| T2 | Extract Google GBP address / SAB rules from official Help | done |
| T3 | Enumerate Tempe/Phoenix UPS Store and virtual-mailbox options + cost bands | done |
| T4 | USPS Form 1583 / PMB format rules | done |
| T5 | Same-address checklist + avoid list + recommended pick | done |

---

## claims

### C1 — Google forbids remote mailboxes and non-operated virtual offices on GBP
- **statement:** Google Business Profile guidelines require a precise real-world location; **“P.O. boxes or mailboxes located at remote locations aren't acceptable.”** A business that **rents a physical mailing address but doesn't operate out of that location (virtual office) is not eligible**. Co-working only qualifies with **clear signage, customers received during hours, and staffed by the business’s own staff**. Service-area businesses **should hide** the address from customers.
- **evidence_refs:** [E1], [E2], [E3]
- **confidence:** high
- **status:** supported

### C2 — CMRA is still the right tool for public non-GBP NAP and mail
- **statement:** A CMRA (Commercial Mail Receiving Agency) provides a real street address under USPS rules via PS Form 1583; virtual mailbox products are typically CMRAs. They are appropriate for mail, packages (when the center accepts all carriers), website contact, and many citations — **separate** from Maps pin eligibility.
- **evidence_refs:** [E4], [E5], [E6]
- **confidence:** high
- **status:** supported

### C3 — USPS requires PMB or # on CMRA customer addresses
- **statement:** Mail to a CMRA customer must include **PMB** or **#** plus the private mailbox number. Using a bare street or a decorative “Suite” that hides the CMRA nature can break USPS delivery rules and is a common compliance failure. If the CMRA’s own address already uses suite/#, customer box must use **PMB**.
- **evidence_refs:** [E4], [E7]
- **confidence:** high
- **status:** supported

### C4 — Local UPS Store mailbox is the strongest default for PJ AZ
- **statement:** The UPS Store private mailbox is a staffed local CMRA with a real street address, multi-carrier package acceptance, optional pickup, and on-site Form 1583/ID process. Multiple Tempe locations exist; **3116 S Mill Ave (85282)** is a strong first call for brand locality (Mill corridor). National secondary sources put typical rents roughly **$15–30/mo** common band, broader **$10–60/mo** by size/market; franchises set price — **call for Tempe quote**.
- **evidence_refs:** [E6], [E8], [E9], [E10], [E11]
- **confidence:** high (fit); medium (exact monthly $ until store quotes)
- **status:** supported (fit); partial (exact local price)

### C5 — Pure virtual mailbox networks are optional add-ons, not GBP fixes
- **statement:** Anytime Mailbox / iPostal1-class products start roughly **~$5–15/mo** average marketing floors (location-variable; some vanity addresses much higher) and add scan/forward. Useful if Frank wants remote mail handling. They do **not** unlock a public GBP pin. PhysicalAddress.com plans from **$7.98/mo** but published locations are **not AZ** — poor choice for Tempe-local NAP.
- **evidence_refs:** [E12], [E13], [E14]
- **confidence:** medium-high
- **status:** supported

### C6 — Banks and some fintechs detect/reject CMRA addresses
- **statement:** CMRA addresses are often identifiable via USPS data; some banks and platforms reject virtual/commercial mailbox addresses for account address fields. SEO mailing address and banking address may need to differ — do not assume the CMRA works for Mercury/business banking KYC.
- **evidence_refs:** [E4]
- **confidence:** medium (platform-specific; verify at account open)
- **status:** partial

### C7 — Dual public addresses (home on one site, CMRA on another) create citation debt
- **statement:** Local SEO wants one stable NAP. Splitting website vs Yelp vs Facebook vs invoices invites map packs / directory mismatch. GBP hidden address is the intentional exception; all **visible** citations should match the CMRA line exactly (including PMB/# punctuation).
- **evidence_refs:** [E1], [E2]
- **confidence:** high
- **status:** supported

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | Google — Guidelines for representing your business | https://support.google.com/business/answer/3038177 | browser navigate + Address zippy expand + `document.body` text, 2026-11-14 | “P.O. boxes or mailboxes located at remote locations aren't acceptable.” Virtual office not eligible if you don’t operate there. Co-working needs signage + customers + own staff. SAB: “you should hide your business address from customers.” |
| E2 | MailboxMath — Virtual mailbox & GBP (quotes Google) | https://mailboxmath.com/guides/virtual-mailbox-google-business-profile/ | web extract/python HTML text, checked page dated 2026-11-02 | Tabular restatement of Google mailbox / virtual office / SAB / eligibility lines; concludes virtual mailbox not eligible as Maps address. |
| E3 | GBP Guardian / BusinessAnywhere / Otepsphere secondary | https://gbpguardian.com/google-business-profile-address-rules ; https://businessanywhere.io/how-to-use-virtual-address-google-business-profile/ ; https://otepsphere.com/resources/po-box-mailbox-google-business-profile/ | web_search + HTML text | Independent restatements aligning with E1; suspension risk theme for ineligible addresses. |
| E4 | AddressAccepted — What is a CMRA address | https://addressaccepted.com/guides/what-is-a-cmra-address/ | HTML text, page dated 2026-10-27 | CMRA = USPS DMM 508.1.8 business receiving mail for others; Form 1583; PMB/# format; GBP rejects CMRA as shown address; banks/Stripe may flag CMRA. |
| E5 | USPS Postal Explorer Pub 28 §285 (PMB) | https://pe.usps.com/text/pub28/28c2_040.htm | web_search + fetch (JS-heavy; rule confirmed via secondary + USPS form refs) | Private mailbox addresses at CMRAs must use PMB or # with box number. |
| E6 | The UPS Store — Mailboxes product | https://www.theupsstore.com/mailboxes | web_search + brand pages | Markets real street address vs PO Box; package receiving; franchise pricing varies. |
| E7 | Innbocks / AddressAccepted PMB format guidance | https://www.innbocks.com/compliance/pmb-addressing ; E4 | HTML/search | PMB vs Suite misuse; two allowed designator positions; “Suite” cosplay is a common break. |
| E8 | UPS Store Tempe location directory | https://locations.theupsstore.com/az/tempe | curl directory HTML 2026-11-14 | Listed Tempe stores include Broadway, Mill, Rural, McClintock, Elliot. |
| E9 | UPS Store #5040 South Mill Ave | https://locations.theupsstore.com/az/tempe/3116-south-mill-ave | browser 2026-11-14 | **3116 South Mill Ave, Tempe, AZ 85282**; (480) 889-0382; store #5040; Mon–Fri 8–6, Sat 9–5; mailbox open process: two IDs + agreement + size. |
| E10 | ThePricer — UPS mailbox cost | https://www.thepricer.org/ups-po-box-rental-cost/ | HTML/JSON-LD, modified 2026-04-30 | TLDR ~**$15–30/mo** most franchises; case: medium ~$25/mo + ~$20 setup + ~$10 key; Form 1583; no national fixed chart. |
| E11 | MailboxAvenue — UPS mailbox pricing ranges | https://mailboxavenue.com/blogs/the-mailbox-blog/how-much-are-ups-mailboxes | HTML text, 2025-06-30 | Small ~$10–30, medium ~$20–40, large ~$30–60 /mo; longer terms discount; Form 1583 + photo ID. |
| E12 | Anytime Mailbox marketing / reviews | https://www.anytimemailbox.com/ ; review summaries 2026 | web_search | Plans often from ~$4.99–$9.99 average marketing; location-priced; up to ~$50 vanity; 2500+ locations; Form 1583 at US sites. |
| E13 | PhysicalAddress.com pricing + location footprint | https://physicaladdress.com/pricing/ ; third-party location notes | web_search 2026-10 | Plan 30 **$7.98/mo** upward; scan allowances; locations cited as ID/NV/DE/GA/TX/FL/CA/NY — **not a Phoenix street**. |
| E14 | iPostal1 plans | https://ipostal1.com/compare-plans.php | web_search | Plans marketed from ~**$9.99/mo**; large location network; CMRA model. |
| E15 | Other Tempe UPS Stores (phones/zips) | locations.theupsstore.com/az/tempe/* | curl 2026-11-14 | 1753 E Broadway 85282 #2020 (480) 829-3900; 7650 S McClintock 85284 #3530 (480) 777-2777; 315 W Elliot 85284 #5570 (480) 889-0060; 5136 S Rural 85282 #8408. |
| E16 | Phoenix UPS Store density | https://locations.theupsstore.com/az/phoenix | curl directory | Dozens of Phoenix addresses if Tempe boxes are full — same CMRA model. |

---

## findings

### 1. Option comparison (decision matrix)

| Option | Real street? | All-carrier packages | Local AZ face | Typical $ | GBP public pin OK? | Best use for PJ |
|---|---|---|---|---|---|---|
| **USPS PO Box** | No (PO BOX n) | USPS-centric | Local ZIP yes | Often cheapest | **No** | Only if mail is USPS-only and no citation need |
| **UPS Store CMRA (Tempe)** | Yes | Yes (store network) | **Strong** | ~$15–40/mo band + fees | **No** (still a mailbox) | **Primary recommendation** for NAP + packages |
| **Virtual mailbox (Anytime / iPostal1 at local center)** | Yes (center’s street) | Usually | Depends on chosen center | ~$10–25+ loc-variable | **No** | If remote scan/forward matters more than walk-in |
| **PhysicalAddress.com etc. (non-AZ)** | Yes but **wrong city/state** | Provider-dependent | **Weak for Tempe SEO** | From ~$8/mo | **No** | Avoid as public PJ address |
| **Virtual office / bare cowork desk** | Sometimes | Varies | Maybe | $50–300+/mo | **Only if** true staffed office w/ PJ signage & customers | Overkill / high GBP risk if faked |
| **Home address public** | Yes | Yes | Accurate but privacy loss | $0 | Only if customers actually visit & signage rules met; else hide as SAB | Privacy fail for public web |

### 2. Google Business Profile rules (operative)

From Google Help (E1), verified in-browser 2026-11-14:

- Eligible if customers visit you **or** you travel to customers.
- Address must be actual real-world location; **no PO Box / remote mailbox**.
- Rented mailing address you don’t operate from (**virtual office**) → **not eligible**.
- Showing address implies **permanent fixed signage** of the business name.
- **Service-area businesses should hide address** (example: plumber working from home).
- Hybrid (shop + mobile) can show storefront **only if** staffed and able to receive customers during hours.
- One profile per business location; don’t duplicate.

**PJ implication:** Treat GBP as **service-area** covering greater Phoenix (Tempe, Mesa, Chandler, Phoenix, Scottsdale, etc. as actually served). Enter whatever Google requires privately for verification if needed, but **do not publish** CMRA or home on the Maps card unless/until there is a true staffed storefront with signage.

### 3. Local options — Tempe UPS Store shortlist

Call 2–3 stores Monday–Saturday; ask: mailbox availability, small/medium monthly on 6- and 12-month terms, setup/key fees, package limits, whether they issue **PMB vs #**, and hours for Form 1583 / notary if needed.

| Priority | Address | ZIP | Store # | Phone | Notes |
|---|---|---|---|---|---|
| **1 — call first** | 3116 S Mill Ave | 85282 | 5040 | (480) 889-0382 | Mill & Southern; strong “Tempe” brand signal; hours Mon–Fri 8–6, Sat 9–5 (verified page) |
| 2 | 1753 E Broadway | 85282 | 2020 | (480) 829-3900 | Central Tempe 85282 |
| 3 | 5136 S Rural Rd | 85282 | 8408 | (see locator) | 85282 Rural corridor |
| 4 | 7650 S McClintock Dr | 85284 | 3530 | (480) 777-2777 | South Tempe |
| 5 | 315 W Elliot Rd | 85284 | 5570 | (480) 889-0060 | South Tempe / Ahwatukee-adjacent |

**Phoenix fallback:** large network on `locations.theupsstore.com/az/phoenix` if Tempe boxes waitlisted — prefer still **East Valley** streets if PJ brand is Tempe-centric.

**Open process (Mill Ave FAQ):** two valid IDs, mailbox agreement, choose size; then receive mail/packages (E9). USPS still requires **PS Form 1583** for CMRA receipt (E4, E10, E11).

### 4. Virtual mailbox networks (if Frank wants scanning)

- **Anytime Mailbox:** marketplace of local centers; price per address; marketing average ~$9.99/mo; confirm a **Tempe/Phoenix** operator before buying.
- **iPostal1:** similar CMRA network; plans advertised from ~$9.99/mo.
- **PhysicalAddress.com:** cheap scan plans ($7.98+) but **not an AZ street** in published footprint → **do not** use as Print Junkie public NAP.

Virtual mailbox is a **mail UX upgrade**, not a Google loophole.

### 5. Cost bands (planning numbers, not quotes)

| Item | Planning range | Source class |
|---|---|---|
| UPS Store small–medium box | ~$15–40 /mo | E10, E11 (national franchise ranges) |
| Setup + key | often ~$20–40 one-time combined (varies) | E10 case |
| Anytime/iPostal1 entry | ~$10–20 /mo typical; location swings | E12, E14 |
| PhysicalAddress Plan 30 | $7.98 /mo | E13 — wrong geography for PJ |
| True staffed virtual office | often $50–300+ /mo | market norm (not primary path) |

**Action:** get two written Tempe UPS quotes before locking budget.

### 6. Canonical address string (after box assigned)

Use **exactly** one of these patterns (match what the store prints on 1583 / welcome sheet):

```text
Print Junkie AZ
3116 S Mill Ave PMB ###
Tempe, AZ 85282
```

or

```text
Print Junkie AZ
3116 S Mill Ave ####
Tempe, AZ 85282
```

Rules:

- Same spelling of street (`South` vs `S`) everywhere — copy from the store’s official line.
- Never drop PMB/# on public pages “to look more legit.”
- Phone for NAP should be the **business** number (Google Voice/local), not the UPS Store phone.
- Do not put “The UPS Store” in the business name field anywhere.

### 7. Same-address rollout checklist (non-GBP)

After box is live and first piece of mail received:

1. **Website** contact page + footer + any “Visit/Ship to” blocks  
2. **LocalBusiness / Organization JSON-LD** `address`  
3. **Invoice / estimate / packing slip** templates  
4. **Email signature**  
5. **Yelp, Facebook, Instagram business, Apple Business Connect** (as applicable)  
6. **Citation base** (Data Axle / BrightLocal / Neustar localez / Bing Places mailing fields — where a mailing address is collected **without** implying a public storefront pin)  
7. **Supplier accounts** (print blanks, apparel, ink) return address  
8. **Domain WHOIS / LLC mailing** if currently home and privacy is the goal (coordinate with formation docs; some states require RA street separately)  
9. **Stripe / shop “support address”** — test; if CMRA blocked, keep support email/phone primary and use acceptable address fields per processor  
10. **Internal single source of truth** doc: one canonical multiline address + date adopted  

**GBP-specific checklist:**

1. Profile type: service area / hide address  
2. Service areas = real drive-time markets only (Google notes ~2 hours as a guideline for many service businesses)  
3. Categories honest (e.g. custom t-shirt shop / embroidery service — match actual offer)  
4. Never set CMRA as public pin “just to rank in Tempe”  
5. If Google mails a postcard: use an address you can actually receive verification at (CMRA can receive many verification letters — still does **not** make the CMRA a displayable storefront)

### 8. What to avoid

| Avoid | Why |
|---|---|
| Public GBP pin on UPS Store / virtual mailbox | Policy violation → suspension / quality flags (E1) |
| PO Box as only public address | No street form; weaker ops; still banned on GBP |
| “Suite 123” cosplay on a CMRA without authorization | USPS PMB rules; detection risk (E4, E7) |
| Out-of-state vanity address as PJ NAP | Breaks local relevance; looks like spam stack |
| Different addresses on web vs Yelp vs invoices | Citation fragmentation |
| Cowork “virtual office” marketed as GBP-safe without staff + signage + customers | Explicitly called out as ineligible (E1) |
| Assuming bank KYC accepts CMRA | Often false (E4) |
| Putting home on GBP “just for now” while also publishing home on the site | Privacy loss; if SAB, hide it |

---

## recommended_pick

### Primary (do this)

1. **Call The UPS Store #5040 — 3116 S Mill Ave, Tempe AZ 85282 — (480) 889-0382**  
   Ask for small or medium business mailbox, 12-month price, package policy, PMB number format, Form 1583 steps.  
2. If full/pricey, call **1753 E Broadway (480-829-3900)** and **7650 S McClintock (480-777-2777)** same day.  
3. Complete **PS Form 1583** + two IDs; receive box number.  
4. Lock **canonical NAP string** (with PMB/#).  
5. Roll website + schema + citations to that string.  
6. Configure **GBP as service-area, address hidden**, service areas = real East Valley / PHX coverage.  
7. Keep home off public HTML.

### Optional upgrade

- If Frank travels or hates pickup runs: add **scan/forward** via a virtual-mailbox product **only if** it is the **same** street/PMB (some UPS Stores participate in Anytime-style networks) — do not create a second public address.

### Not the pick

- PhysicalAddress.com (or any non-AZ street) as the public Print Junkie address.  
- Paying for a “GBP-passing virtual office” unless there will be real staff, signage, and customer visits.

---

## gaps

1. **Exact Tempe monthly rent** — franchise-specific; not published nationally; needs phone quote (gap is operational, not conceptual).  
2. **Whether store #5040 has open small boxes this week** — inventory unknown.  
3. **Stripe/bank acceptance** of the chosen CMRA for PJ’s specific accounts — verify at change time.  
4. **Arizona LLC / statutory agent** interaction — RA may need a different address than marketing CMRA; not fully scoped here.  
5. **Live Anytime Mailbox Tempe operator list + prices** — site is app/JS heavy and Cloudflare-gated in this environment; confirm in browser at purchase time.  
6. **Parent SEO Bible 3 checklist path** referenced on the card was not fully readable from this container; recommendations above align with standard SAB + NAP practice and explicit Google text.

---

## confidence

```json
{
  "level": "high",
  "rationale": "Core GBP prohibitions and SAB hide-address rule were read from Google Help in-browser. CMRA/PMB/1583 rules are consistent across USPS-oriented secondary sources. Tempe UPS Store identities/phones verified from official locator pages. Exact local mailbox dollar amounts remain medium confidence until store quotes."
}
```

---

## recommended_next_actions

1. **Frank (15 min):** Call Mill Ave UPS Store + one backup; choose box size; complete 1583.  
2. **Tricon (after box # known):** Patch printjunkie site contact + schema to canonical CMRA line; sweep footer/templates.  
3. **Tricon/SEO:** Set or confirm GBP service areas + address hidden; do **not** geopin the UPS Store.  
4. **Ops:** Update invoice/ship-from/return address; test one ordered package to the PMB.  
5. **Finance (separate track):** Confirm bank/processor address rules before moving KYC fields.  
6. **Optional:** If scan-from-phone is desired, evaluate Anytime/iPostal1 **only** against the same center or accept a deliberate second *private* mail workflow without publishing a second address.

---

## appendix A — quick glossary

- **CMRA:** Commercial Mail Receiving Agency — business that accepts USPS mail for others (UPS Store mailbox, most “virtual mailboxes”).  
- **PMB:** Private Mail Box number designator required on CMRA addresses.  
- **PS Form 1583:** USPS application authorizing the CMRA to receive your mail; ID verification required.  
- **SAB:** Service-area business on Google — serves at customer locations; should hide address.  
- **NAP:** Name, Address, Phone consistency across the web.

## appendix B — source priority

1. Google Business Profile Help (E1) — policy  
2. USPS CMRA/PMB/1583 materials (E4, E5) — mail compliance  
3. Official UPS Store locators (E8, E9, E15) — local options  
4. Pricing secondaries (E10–E14) — budget planning only  

---

*Durable archive path: `/home/ice/know/research/print-junkie-az-cmra-public-address-t_d75d59aa.md`*  
*Kanban task: `t_d75d59aa` · Board: research · Prefix: [GIU]*
