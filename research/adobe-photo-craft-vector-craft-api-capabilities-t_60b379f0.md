# Adobe “Photo Craft” / “Vector Craft” API capabilities map

**Task:** `t_60b379f0`  
**Agent:** Geraldo v2.2 (GIU)  
**Date (UTC):** 2026-10-13  
**Archive path:** `/home/ice/know/research/adobe-photo-craft-vector-craft-api-capabilities-t_60b379f0.md`

---

## objective

Map what Adobe actually exposes programmatically for generative **photo** and **vector** editing—especially fill/inpaint, expand/outpaint, recolor, text-to-vector, and related ops—so product/eng can choose stack boundaries. The task used informal labels “Photo Craft” and “Vector Craft”; those are **not** Adobe product names.

## summary

1. **Identity (critical):** There are **no Adobe products** named Photo Craft or Vector Craft. Web hits for those strings resolve almost entirely to **ArtCraft / Storytold** open-source apps (**PhotoCraft**, **VectorCraft** on getartcraft.com / GitHub `storytold/*`). Adobe’s matching surface is **Firefly Services**: Firefly API + Photoshop API + Illustrator API (+ Lightroom/InDesign/etc.), plus in-app Firefly features in Photoshop/Illustrator/Firefly web.

2. **Photo / generative raster (API-ready):** Firefly API at `https://firefly-api.adobe.io` exposes async **Fill Image** (`POST /v3/images/fill-async`) and **Expand Image** (`POST /v3/images/expand-async`) with mask/prompt/seeds/size controls, plus generate, generate-similar, object/adaptive/precise composite, upscale, video generate, custom models. Photoshop API separately bills a **Fill Mask Area** op (inpaint-style) under Firefly Services Photoshop suite—not the same endpoint as Firefly fill.

3. **Vector (API-partial):** Illustrator API (`https://illustrator-api.adobe.io`) today is **template automation**: data merge, rendition, and **Image Trace** (`/v1/trace-image`) raster→SVG. OpenAPI has **zero** “recolor” or generative text-to-vector endpoints. **Text to Vector Graphic** exists as a Firefly **product** feature (web/Illustrator UI) and is listed for **IP indemnity eligibility** (as of 2024-10-14), but **no public Firefly OpenAPI path** for text-to-vector was found in the published `firefly-api.json` (paths are image/video only).

4. **Billing:** Enterprise **Shared Credits / Operations** rate card: Fill Image, Expand Image, Generate Image, Generate Similar, Object Composite = **1 Operation per generated image**; Illustrator Trace Image = **0.5 Operation per call**; conversion **1 API Operation ≈ 100 Generative Credits** (fractional allowed). Dollar list price is **not** published—contract/Admin Console.

5. **Commercial / IP:** Firefly models trained on licensed (e.g. Adobe Stock) + public-domain content; enterprise content not used to train foundation models. Outputs treated as Customer Content between Adobe and customer; commercial use generally allowed for non-beta features; **IP indemnification only on qualifying plans/entitlements** with exclusions (customer mods, combined materials, misuse, custom-training content, etc.). Content Credentials (C2PA) applied on qualifying export/download; Firefly APIs export event = consume Operations **and** download output.

6. **Integration posture:** OAuth S2S (IMS) + `x-api-key` client id; async job status/cancel; storage upload or presigned cloud URLs (allowlisted hosts). Prefer Firefly fill/expand for generative photo ops; Illustrator for deterministic vector template/trace; do **not** assume API parity with every Firefly web/Illustrator generative UI feature.

---

## plan

| task_id | title | status | notes |
|--------|--------|--------|-------|
| T1 | Product identity / name disambiguation | done | Adobe vs ArtCraft collision |
| T2 | Firefly Services surface inventory | done | Firefly + PS + AI + siblings |
| T3 | Photo endpoints: fill, expand, related | done | OpenAPI + tutorials |
| T4 | Vector endpoints: T2V, recolor, trace | done | OpenAPI + helpx; T2V API gap |
| T5 | Pricing / credits / ops | done | Shared Credits + ops rate card |
| T6 | License / commercial / indemnity | done | Product description + legal FAQs |
| T7 | Structured report + publish | done | this file |

**Assumptions:** “Photo Craft / Vector Craft” meant Adobe generative photo + vector capabilities, not ArtCraft OSS. Scope = public docs/OpenAPI as of extraction date; private ETLA SKUs may differ.

**Success criteria:** Clear Adobe mapping; endpoint-level fill/expand; honest vector API gaps; pricing unit model; commercial-use caveats with provenance.

---

## claims

### C1 — No Adobe products named Photo Craft or Vector Craft
- **statement:** Adobe does not market products called “Photo Craft” or “Vector Craft.” Those names belong primarily to ArtCraft/Storytold open-source crafting apps; the Adobe programmatic analogue is Firefly Services (Firefly / Photoshop / Illustrator APIs) and Firefly-powered features inside Creative Cloud apps.
- **evidence_refs:** [E1], [E2], [E3], [E4]
- **confidence:** high
- **status:** supported

### C2 — Firefly Services is the developer umbrella
- **statement:** Adobe Firefly Services bundles Firefly APIs, Photoshop APIs, Lightroom APIs, Content Tagging, and related creative APIs for generative + creative automation at scale.
- **evidence_refs:** [E3], [E5]
- **confidence:** high
- **status:** supported

### C3 — Fill (inpaint) is first-class on Firefly API v3
- **statement:** `POST https://firefly-api.adobe.io/v3/images/fill-async` (operationId `fillImagesV3Async`) requires `image` + `mask`; optional `prompt` (≤1024 chars), `negativePrompt`, `numVariations` (1–4), `seeds`, `size`, `promptBiasingLocaleCode`. Mask supports `invert`. Auth: bearer access token + `x-api-key`. Async via `/v3/status/{jobId}` and `/v3/cancel/{jobId}`.
- **evidence_refs:** [E6], [E7], [E8]
- **confidence:** high
- **status:** supported

### C4 — Expand (outpaint / generative expand) is first-class on Firefly API v3
- **statement:** `POST /v3/images/expand-async` requires `image`; optional `mask`, `placement` (alignment/inset; cannot combine placement with mask), `prompt`, `seeds`, `numVariations`, `size` (docs cite max output 3999×3999 in schema description; default 2048×2048). Same auth/async pattern as fill.
- **evidence_refs:** [E6], [E9], [E8]
- **confidence:** high
- **status:** supported

### C5 — Adjacent Firefly image APIs useful for “photo craft” pipelines
- **statement:** Published Firefly OpenAPI also includes generate (`/v3/images/generate-async`, `/v4/images/generate-async`), generate-similar, generate-object-composite-async, adaptive-composite, precise-composite, upscale, storage upload `/v2/storage/image`, custom-models list, and video generate.
- **evidence_refs:** [E6], [E3]
- **confidence:** high
- **status:** supported

### C6 — Photoshop API offers a separate Fill Mask Area capability
- **statement:** Firefly Services operations rate card lists Photoshop **Fill Mask Area** (“Fill in a masked area to complete an image”) at 1 Operation per API call, distinct from Firefly Fill Image. Docs also describe Photoshop “Fill Masked Areas” / inpainting guides under the Photoshop API tree.
- **evidence_refs:** [E10], [E11]
- **confidence:** medium-high
- **status:** supported

### C7 — No public Firefly API text-to-vector endpoint in current OpenAPI
- **statement:** Official `firefly-api.json` path list contains no vector-generation route. Text to vector is documented as a **Firefly web / Illustrator product** workflow (including partner models in UI) and appears on the Firefly IP indemnification eligible-features table (Text to Vector Graphic as of 2024-10-14), but programmatic parity is **not** evidenced in the public Firefly OpenAPI snapshot used here.
- **evidence_refs:** [E6], [E12], [E13], [E14]
- **confidence:** high (for “not in public OpenAPI”); medium (for “no private/partner API exists anywhere”)
- **status:** supported (public API absence); partial (absolute product-wide absence)

### C8 — Illustrator API vector surface is merge / rendition / image trace — not generative recolor
- **statement:** Illustrator API OpenAPI paths: `/v1/create-rendition`, `/v1/merge-data`, `/v1/trace-image`, status routes. Rate card: Data Merge and Rendition = 1 Operation per output file; **Trace Image** = 0.5 Operation per call (PNG/JPEG → editable SVG via Image Trace). Spec text search found **0** occurrences of “recolor” and **0** of “generative” as API ops.
- **evidence_refs:** [E15], [E16], [E10]
- **confidence:** high
- **status:** supported

### C9 — Operations / Shared Credits pricing model (not public $/call)
- **statement:** Enterprise consumption uses **Operations** under Shared Credits terms. Firefly Fill/Expand/Generate Image/Similar/Object Composite: **1 Operation = 1 generated image** (multi-variation calls multiply). Credit conversion published: **1 API Operation = 100 Generative Credits** (fractional). Public consumer Firefly plans are separate (generative credits in apps); API dollar rates are commercial/contract, not a public self-serve rate card in the sources reviewed.
- **evidence_refs:** [E10], [E17]
- **confidence:** high (units); low-medium (absolute USD without contract)
- **status:** supported (units); partial (USD)

### C10 — Commercial use, training data, ownership, indemnity (conditional)
- **statement:** Adobe positions Firefly foundation training on licensed + public-domain content; does not train foundation models on enterprise customer content (Custom Models fine-tune only for that customer). Outputs are Customer Content as between Adobe and customer; Adobe does not assert IP in outputs; outputs not guaranteed unique; cannot use outputs to train/improve AI/ML models. Commercial use allowed for non-beta features subject to agreement. **IP indemnification** for eligible features/surfaces requires **qualifying entitlement**; eligible features include Text to Image, Generative Fill, Generative Expand/Expand Image, Text to Vector Graphic, etc.; Firefly APIs surface export event = Operations consumption + download. Exclusions include customer modifications, combinations, agreement violations, context of use, custom training content, non-Adobe/partner models, beta.
- **evidence_refs:** [E14], [E18], [E19], [E20]
- **confidence:** high (structure of terms); medium (whether a specific org’s SKU qualifies—contract-specific)
- **status:** supported

### C11 — Input URL / upload constraints matter for integration
- **statement:** Firefly fill/expand image sources accept upload IDs or URLs only from allowlisted domains (schema text lists amazonaws.com, windows.net, dropboxusercontent.com, storage.googleapis.com). Binary upload via `/v2/storage/image` for jpeg/png/webp/tiff/jxl.
- **evidence_refs:** [E6], [E8]
- **confidence:** high
- **status:** supported

### C12 — Name collision risk with ArtCraft PhotoCraft/VectorCraft
- **statement:** Independent OSS suite ArtCraft markets PhotoCraft and VectorCraft as Adobe-replacement creative apps (Rust, open source). Any internal shorthand “Photo Craft / Vector Craft” should be disambiguated in tickets to avoid pulling wrong repos, licenses, or roadmaps.
- **evidence_refs:** [E1], [E2]
- **confidence:** high
- **status:** supported

---

## evidence_index

| evidence_id | source_label | source_url | provenance | support note / excerpt |
|-------------|--------------|------------|------------|------------------------|
| E1 | ArtCraft VectorCraft product | https://getartcraft.com/apps/vectorcraft | web_search + listing | VectorCraft described as ArtCraft crafting app; not Adobe |
| E2 | GitHub storytold/vectorcraft | https://github.com/storytold/vectorcraft | web_search | OSS vector app by ArtCraft team |
| E3 | Firefly API overview | https://developer.adobe.com/firefly-services/docs/firefly-api/ | web_extract 2026-10-13 | Firefly API capabilities: generate, expand, fill, similar, composites, custom models, video; base `firefly-api.adobe.io` |
| E4 | Firefly Services docs hub | https://developer.adobe.com/firefly-services/docs/guides/ | web_extract | Firefly + Lightroom + Photoshop + Content Tagging under Firefly Services |
| E5 | Photoshop API overview | https://developer.adobe.com/firefly-services/docs/photoshop/ | web_search/docs | Photoshop API v2 within Firefly Services |
| E6 | firefly-api.json OpenAPI 3.1 | https://raw.githubusercontent.com/AdobeDocs/ffs-firefly-api/main/static/firefly-api.json | curl + local parse | Paths include `/v3/images/fill-async`, `expand-async`, generate*, composites, upscale, storage, status/cancel, videos; servers production `https://firefly-api.adobe.io`; security `AccessToken` bearer S2S + `X-Api-Key` |
| E7 | Fill Image API tutorial | https://developer.adobe.com/firefly-services/docs/firefly-api/guides/how-tos/firefly-fill-image-api-tutorial | web_extract | End-to-end fill workflow for background replacement |
| E8 | Fill/Expand request schemas | same OpenAPI as E6 (`FillImageRequestV3`, `ExpandImageRequestV3`) | local JSON resolve | Required fields, mask invert, placement vs mask mutual exclusion, size notes, prompt limits |
| E9 | Expand Image API tutorial | https://developer.adobe.com/firefly-services/docs/firefly-api/guides/how-tos/firefly-expand-image-api-tutorial | web_extract | Expand workflow documentation |
| E10 | Firefly Services operations rate card | https://www.adobe.com/go/firefly-api-operation-definitions (helpx enterprise shared-credits ops) | web_extract full cache | Tables for Firefly Fill/Expand/Generate = 1 op/image; PS Fill Mask Area = 1 op/call; AI Trace = 0.5 op; 1 op = 100 generative credits |
| E11 | Photoshop Fill Mask guide | https://developer.adobe.com/firefly-services/docs/photoshop/guides/using-fill-mask/ | web_search | “Fill Masked Areas API” inpainting guide |
| E12 | Firefly Text to vector helpx | https://helpx.adobe.com/firefly/web/generate-vectors/text-to-vector/generate-vectors-using-text-prompts.html | web_extract | Product UI text-to-vector; not an API reference |
| E13 | Illustrator API docs | https://developer.adobe.com/firefly-services/docs/illustrator/ | web_extract | Data merge, rendition, image trace focus |
| E14 | Adobe Firefly product description (IP indemnity) | https://helpx.adobe.com/legal/product-descriptions/adobe-firefly.html | web_extract 2026-08-27 pub | Eligible features incl. Generative Fill, Expand, Text to Vector Graphic; Firefly APIs export event = Operations + download |
| E15 | illustrator-api.json | https://raw.githubusercontent.com/AdobeDocs/ffs-illustrator-api/main/static/illustrator-api.json | curl + parse | Paths only create-rendition, merge-data, trace-image, status; server `https://illustrator-api.adobe.io` |
| E16 | Image Trace guide | https://developer.adobe.com/firefly-services/docs/illustrator/guides/image-trace/ | web_extract | Raster→vector via Illustrator API |
| E17 | Shared Credits product description | https://helpx.adobe.com/legal/product-descriptions/shared-credits.html | web_extract | Shared Credits pool; Firefly Services consume via Operations; effective terms referenced Aug 2026 in secondary sources |
| E18 | Firefly approach for business | https://business.adobe.com/products/firefly-business/firefly-ai-approach.html | web_extract | Commercial safety claims; training permissions; no train on customer CC/AEC content; indemnity if entitled |
| E19 | Firefly Legal FAQs enterprise PDF | https://www.adobe.com/cc-shared/assets/pdf/enterprise/firefly-legal-faqs-enterprise-customers-2024-06-11.pdf | web_extract | Ownership, no train-on-outputs, indemnity scope/exclusions, commercially safe meaning |
| E20 | Content Credentials / gen credits helpx | https://helpx.adobe.com/firefly/web/get-started/learn-the-basics/… (credits + credentials pages attempted) | web_extract partial | C2PA on qualifying Firefly outputs; app generative credits model separate from API ops |

**Extraction paths:** Firecrawl/`web_extract` for Adobe HTML docs; direct `curl` of AdobeDocs GitHub OpenAPI JSON; PDF extract for legal FAQ; `web_search` for discovery only.

---

## capability matrix (decision aid)

| Capability (task language) | Adobe product feature | Public API? | Primary endpoint / surface | Billable unit (ops card) | Notes |
|----------------------------|----------------------|-------------|----------------------------|--------------------------|-------|
| Generative fill / inpaint | Firefly Generative Fill; Firefly Fill Image | **Yes** | `POST /v3/images/fill-async` | 1 op / image | Mask required; prompt optional |
| Photoshop mask fill | Photoshop Fill Mask Area | **Yes** (PS API) | Photoshop Fill Masked Areas API | 1 op / call | Parallel path; confirm v1 vs v2 in implementation |
| Generative expand / outpaint | Generative Expand / Expand Image | **Yes** | `POST /v3/images/expand-async` | 1 op / image | placement XOR mask |
| Text-to-image | Firefly Generate Image | **Yes** | `/v3` and `/v4` generate-async | 1 op / image | Model via headers (`x-model-version`) |
| Generate similar | Generate Similar | **Yes** | `/v3/images/generate-similar-async` | 1 op / image | |
| Product composite | Object / Adaptive / Precise Composite | **Yes** | respective `/v3/images/*` | 1 op / image (object composite) | |
| Upscale | Firefly upscale | **Yes** | `/v3/images/upscale` | check current rate card row | Present in OpenAPI |
| Text-to-vector | Firefly / Illustrator Text to Vector | **UI yes; public Firefly OpenAPI no** | firefly.adobe.com / Illustrator | N/A in Firefly API paths | Indemnity-eligible feature list includes it |
| Vector recolor (generative) | Illustrator generative recolor (app) | **Not in Illustrator OpenAPI** | — | — | No `recolor` in API spec |
| Raster → vector | Illustrator Image Trace | **Yes** | `POST /v1/trace-image` | 0.5 op / call | Deterministic trace, not generative T2V |
| AI template personalization | Illustrator Data Merge | **Yes** | `/v1/merge-data` | 1 op / output file | |
| AI → export formats | Illustrator Rendition | **Yes** | `/v1/create-rendition` | 1 op / output file | |

---

## integration sketch (non-secret)

```
Auth: Adobe IMS S2S client_credentials
  -> access_token (Authorization: Bearer)
  -> x-api-key: <client_id>

Photo fill:
  1) POST /v2/storage/image  OR  host assets on allowlisted URL
  2) POST /v3/images/fill-async  { image.source, mask.source, prompt?, size?, seeds? }
  3) poll GET /v3/status/{jobId}  until succeeded
  4) download result URLs (triggers API “export event” for indemnity accounting)

Photo expand:
  POST /v3/images/expand-async  { image, size, placement? | mask?, prompt? }

Vector today:
  POST illustrator-api /v1/trace-image | /v1/merge-data | /v1/create-rendition
  + status polling
```

Do not embed client secrets in apps; use server-side only.

---

## gaps

1. **Exact USD list price** for Firefly Services operations — contract/Admin Console only; secondary blogs are non-authoritative.
2. **Text-to-vector and generative recolor public APIs** — not in published OpenAPI; unknown if private beta/partner APIs exist.
3. **Photoshop Fill Mask Area** full request schema not deep-extracted this pass (rate card + guide title only)—implementers should open current Photoshop OpenAPI before build.
4. **Partner models** (GPT Image, Ideogram, etc. in Firefly UI) — indemnity table excludes non-Adobe-trained models; API exposure of partner models not fully mapped.
5. **v3 vs v4 generate** behavioral delta and default model IDs — changelog page partially retrieved; pin versions in eng spike.
6. **ArtCraft** roadmap/API — out of scope except name collision warning.
7. **Whether “Photo Craft/Vector Craft” was a mis-remembered internal codename** — no Adobe evidence; if Frank meant a specific deck/SKU, re-run with that artifact.

---

## confidence

- **level:** **high** on identity, Firefly fill/expand API shape, Illustrator API limits, ops unit economics, and indemnity *structure*.
- **rationale:** Claims anchored on Adobe developer OpenAPI JSON, official developer tutorials, helpx legal product description, enterprise ops rate card, and enterprise legal FAQ PDF. Residual uncertainty is commercial USD pricing, unpublished vector generative APIs, and org-specific entitlement fit.

---

## recommended_next_actions

1. **Naming hygiene:** In kanban/Huly, replace “Photo Craft / Vector Craft” with **Firefly Services — Photo (Fill/Expand)** and **Illustrator API — Trace/Merge** (or explicitly ArtCraft if OSS evaluation is intended).
2. **Eng spike (1–2 days):** S2S auth → storage upload → fill-async + expand-async → download; log operation counts vs Admin Console Shared Credits.
3. **Vector decision:** If generative text-to-vector is required **via API**, treat as **blocked on Adobe public API**—options: (a) human-in-loop Illustrator/Firefly UI, (b) Trace Image from raster generations, (c) ask Adobe sales/partner for unreleased T2V API, (d) non-Adobe vector model with separate legal review.
4. **Legal:** Confirm whether current ETLA includes Firefly IP indemnity SKU and whether API export events are in scope before shipping customer-facing gens.
5. **Optional follow-up research card:** Deep OpenAPI extract for Photoshop v2 Fill Mask + any unlisted Firefly vector endpoints after next AdobeDocs repo revision; monitor `ffs-firefly-api` changelog.

---

## findings (narrative)

### Product identity
Search for Adobe “Photo Craft” / “Vector Craft” does not surface Adobe SKUs. It surfaces **ArtCraft** PhotoCraft/VectorCraft and generic Firefly vectorizer marketing pages. For this task, capabilities were mapped from **Adobe Firefly Services**.

### Photo
Programmatic generative photo editing is mature on **Firefly API**: mask-based **fill** and canvas **expand** are documented, schema’d, tutorialized, and billed per output image. Broader pipeline pieces (generate, similar, composites, upscale, custom models) sit on the same API host and auth model. Photoshop Services add document-centric automation plus a billed **Fill Mask Area** path for mask completion inside PS-oriented workflows.

### Vector
**Asymmetry:** product UI has Text to Vector and (in-app) generative vector tools; **public developer API** for Illustrator is classical **trace / merge / rendition**. Teams expecting “Vector Craft API = full Illustrator generative recolor + T2V” will over-estimate API coverage if they only read Firefly marketing.

### Money and risk
Budget in **Operations** (and Shared Credits conversion), not ad-hoc per-app credits alone. Legal posture is relatively strong versus open web image models **if** the contract includes indemnity and workflows stay on eligible Adobe features/surfaces—but indemnity is **not** automatic for every plan, excludes partner/beta models and many post-edits, and still forbids using outputs to train other AI models.

---

## provenance stamp

| item | value |
|------|-------|
| task_id | t_60b379f0 |
| workspace | `/home/ice/.hermes/kanban/boards/research/workspaces/t_60b379f0` |
| openapi snapshots | `sources/firefly-api.json`, `sources/illustrator-api.json` |
| report | `/home/ice/know/research/adobe-photo-craft-vector-craft-api-capabilities-t_60b379f0.md` |
| method | plan → web_search → web_extract → OpenAPI curl/parse → legal/ops extract → structured claims |

---

*End of report.*
