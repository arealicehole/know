# Print Junkie–class GMC / Google UCP eligibility

**Date:** 2026-10-29  
**Author:** Romulus (ops assembly from primary Google docs)  
**Scope:** Can a local print/signage shop (Print Junkie AZ class) use Google Merchant Center free listings and Google’s UCP-powered checkout hub the way product retailers do?  
**Not in scope:** OpenAI ACP feeds, on-site `/.well-known/ucp` (already shipped), citation battery.

---

## Exec summary (Frank)

| Path | Verdict for PJ-class | One-liner |
|------|----------------------|-----------|
| **On-site UCP** (`/.well-known/ucp`) | **YES — already** | House/agent manifest; not the same as Google’s Merchant Center UCP hub |
| **Merchant Center free product listings** | **PARTIAL — only for real SKU products** | Fixed products with price, image, buy link, shipping/returns can list; pure “custom quote / install service” jobs generally **do not** fit product-feed rules |
| **Google UCP-powered checkout (AI Mode / Gemini)** | **NO / very unlikely for core service work** | Requires MC + **approved products** + checkout eligibility; Google explicitly marks **services**, personalized goods, install/setup bundles, special shipping as **ineligible for agentic checkout** |
| **Adjacent Google paths** | **YES — use these** | GBP + free local presence; Local Inventory where you stock finished goods; Local Services Ads / Performance Max local only if category/geo eligible (separate product) |

**Bottom line for Print Junkie:** Keep **on-site UCP + Stripe payment links + intent pages** as the agent transaction layer. Treat **Merchant Center** as optional for **catalog-like SKUs** (e.g. fixed business-card packs, standard banner sizes with set prices) if you can meet product feed + shipping/returns requirements. Do **not** promise Google AI Mode “Buy” checkout for custom wrap/install jobs.

---

## 1) Two different “UCPs” (do not conflate)

| Layer | What it is | PJ today |
|-------|------------|----------|
| **On-site / open UCP** | Machine manifest at `/.well-known/ucp` (and peers) for any agent that fetches your host | **Live** — products, prices, handoff links |
| **Google UCP + Merchant Center** | Google’s integration so **eligible product listings** can checkout inside **AI Mode / Gemini** via Merchant Center product data + APIs | **Not enrolled**; different bar |

Sources: [About UCP on Google (Merchant Center Help)](https://support.google.com/merchants/answer/16837055), [Prepare Merchant Center for UCP](https://developers.google.com/merchant/ucp/guides/overview/merchant-center), [ucp.dev](https://ucp.dev/).

---

## 2) Merchant Center free listings — eligibility shape

Google free listings are for **products** shown across Search, Shopping, Maps, Images, Lens, YouTube, Gemini, Business Profile product modules, etc.

**Requirements (high level):**
- Merchant Center account in good standing  
- Product data meeting [product data specification](https://support.google.com/merchants/answer/7052112) (id, title, description, link, image_link, price, availability, brand/gtin/mpn as applicable, etc.)  
- Policies, landing-page rules, **shipping** settings (required in US and many other countries), return policy strongly expected  
- Free listings often on by default once products are approved — **not a placement guarantee**

Source: [Free listings for products](https://support.google.com/merchants/answer/13889434).

### PJ-class mapping

| Offer type | Free listings fit? | Why |
|------------|--------------------|-----|
| Fixed pack: “100 premium business cards — $65” with product URL, image, price, buy/payment link | **Likely YES if feed quality + shipping/returns OK** | Looks like a normal product SKU |
| Standard A-frame / vinyl size with fixed price and cart/payment link | **Possible** | Same as above |
| “Custom window wrap — quote required” | **Poor / usually NO** | No stable price, often custom, not a clean product detail page |
| On-site installation labor | **NO as a product** | Service, not a shippable/buyable product row |
| Apparel packs with fixed price | **Possible** | Product-shaped if GTIN/MPN/brand handled |

**Verdict:** **Partial.** PJ can use GMC for **SKU-shaped** print products; not as a dump for every service line.

---

## 3) Google UCP-powered checkout — harder bar

Google’s UCP checkout feature (AI Mode / Gemini “Buy” on listings):

- US/CA/AU phase language in Help; **select merchants** / interest form  
- Requires **Merchant Center + approved products for free listings** first  
- Product attribute `native_commerce(checkout_eligibility)`  
- Return policy + customer support in MC  
- Checkout API / payments integration (Google Pay path)

### Explicit **ineligible** categories (Google developer guide)

From [Prepare your Merchant Center account (UCP)](https://developers.google.com/merchant/ucp/guides/overview/merchant-center) — product restrictions include:

- **Services** (lessons, classes, travel packages unless special travel rails)  
- **Personalized goods** (custom design, engravings, monograms)  
- **Bundled services** (warranties, **installation**, setup contracts)  
- **Special shipping** (freight, non-standard delivery fees)  
- Subscriptions, installments, pre-orders, final-sale/used, age-restricted, prohibited content, pure digital/virtual goods, etc.

### PJ-class mapping

| Offer type | Google UCP checkout | Why |
|------------|---------------------|-----|
| Fully custom design + print | **Ineligible** | Personalized goods |
| Install / wrap on-site | **Ineligible** | Bundled services / installation |
| Freight or special delivery banners | **Likely ineligible** | Special shipping |
| Simple fixed SKU, ship standard, returns allowed, no customization after order | **Maybe eligible if** MC + feed + checkout API + Google accepts merchant | Retail-like path only |
| Stripe payment-link handoff only (no Google checkout API) | **Does not unlock Google UCP Buy button** | Different integration |

**Verdict:** **No for core PJ service revenue.** **Maybe later** for a thin slice of non-custom, shippable SKUs if Frank wants the full Google retail integration cost.

---

## 4) Adjacent paths that *do* fit local print shops

| Path | Role | Notes |
|------|------|-------|
| **Google Business Profile** | Local discovery, hours, photos, products module sometimes | Already core Bible §3 |
| **Free local listings / local inventory** | In-store or local product availability | Only if you inventory finished goods at a place |
| **On-site UCP + specs + intent pages** | Agent transaction/identity | **Primary** for chat discovery (battery already shows engines reading prices/NAP) |
| **OpenAI ACP / product feeds** | ChatGPT commerce discovery when enrolled | Separate from GMC; still product-feed shaped |
| **Local Services Ads** | Lead ads for some home-service categories | Print/signage **may not** be a standard LSA category — check Google’s category list before promising; moving toward PMax local over 2026–27 per industry notes |

---

## 5) Recommended doctrine / AVP language

**Do say:**
- “We ship an on-site commerce manifest agents can fetch.”  
- “SKU-like products *may* be submitted to Merchant Center free listings if they meet Google’s product feed rules.”  
- “Google AI Mode checkout is a **retail product** program; custom print/install is usually out.”

**Do not say:**
- “Sign up for Merchant Center UCP and ChatGPT will check out your wraps.”  
- “Every local service is GMC/UCP eligible.”  
- “Free listings guarantee Shopping/AI placement.”

---

## 6) Practical next step for Print Junkie (optional, not done here)

1. Inventory offers that are **fixed price + image + product URL + ship or local pickup rules + returns**.  
2. If ≥5 clean SKUs: create MC, feed those only, enable free listings, watch approvals.  
3. Leave custom/install on **on-site UCP + phone/Stripe links** only.  
4. Skip Google UCP checkout integration until (if ever) a non-custom SKU line is worth the API work.

---

## Sources

- https://support.google.com/merchants/answer/16837055 — UCP + checkout on Google  
- https://developers.google.com/merchant/ucp/guides/overview/merchant-center — MC prep, **services/personalized/install ineligible**  
- https://support.google.com/merchants/answer/13889434 — free listings  
- https://support.google.com/merchants/answer/7052112 — product data specification  
- https://blog.google/products/ads-commerce/agentic-commerce-ai-tools-protocol-retailers-platforms/ — UCP launch context (2026-01-11)  
- Live PJ on-site UCP: https://printjunkieaz.com/.well-known/ucp  

---

*Romulus — eligibility research only. No Merchant Center account created. No Bible edit in this file.*
