# SEO Bible 3.0.1_DRAFT §1.2.1 — Wording Sign-off (C10)

**Task:** t_306f1e0b (board: research, prefix [GIU])
**Prepared:** 2026-11-16 (UTC)
**Prepared by:** Geraldo v2.2 (Kanban-native)
**Scope:** Wording verdict ONLY on §1.2.1 of `SEO_BIBLE_3.0.1_DRAFT.md` (know root + `research/` mirror, identical at HEAD `11d1a0d`, both match `origin/main`) against the C2–C9 criteria from the substance review `t_55fb02ad` (`research/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md`). No Google doc re-run — no contradiction found that would trigger one.

---

## VERDICT: **PASS-WITH-NITS**

Two minor nits (checklist wording + stale companion path). No doctrinal error, no pricing claim, no placement guarantee, no §1.6/§1.7 conflict.

**Frank lock recommendation: `lock-after-nits`** — the nits are 2-line wording fixes, not substance. If Frank prefers zero-friction lock, plain `lock` is also defensible (nits do not change any rule); `hold` is not warranted.

---

## C2–C9 criteria check (wording level)

| C | Criterion (from t_55fb02ad) | Draft wording location | Verdict |
|---|---|---|---|
| C2 | `validFrom` Recommended, not Required; required Offer core = `price`+`priceCurrency` | L134-137 ("Required (Google merchant-listing Offer core — critical)"): `price`+`priceCurrency` (+ `url`, discrete Offer rows); L142-143: sale-duration "Google classifies as **Recommended** … **not** a new Required set" | **PASS** |
| C3 | Stale `priceValidUntil` = suppression risk ("listing may not display") | L145: "**Never ship a stale `priceValidUntil` / past end date** — Google warns the listing **may not display**." | **PASS** |
| C4 | 2026 sale-duration addition framed as capability, not new requirement | L142: "added as capability ~2026-07 — **not** a new Required set" | **PASS** |
| C5/C6/C7 | Live-specimen facts (paired window dates on page; manifest `priceRange`/`tiers`; quote lines `pricing: []`) | L155-157 surfaces; L165 specimen; L128 branch B `pricing: []` | **PASS** (facts accurate; see N2 for the one PR-reference nit below) |
| C8 | Fixed-SKU Product-Offer-eligible; custom/install/service not | L121-130 two-branch rule; L128 "GMC feed rows for custom install" = do-not-ship; L244-245 §1.7 intact (byte-identical to locked 3.0) | **PASS** |
| C9 | Two-branch shape (a) complete Offer / (b) honest quote path | L123-130 table + "Do not collapse" line; L148-151 hard honesty | **PASS** |

Plus the standing no-claim gates:
- **No placement guarantee:** L151 "Complete Offer schema **≠** GMC free-listing win **≠** AI Mode / Gemini checkout eligibility (§1.7 still governs)"; L172-175 do-not-claim list includes "Complete schema gets you into AI Mode / free listings / chat checkout". **PASS.**
- **No AVP / no pricing:** no rate card, no "$Y for X" in §1.2.1; §11 checklist heading still says "(no commercial pricing)". **PASS.**
- **§1.6 / §1.7 preserved:** §1.6 and §1.7 byte-identical to locked 3.0 (verified by full diff `SEO_BIBLE_3.0.md` → `SEO_BIBLE_3.0.1_DRAFT.md`: only the header/version block, the additive §1.2.1 block, the §7 title, one §7 bullet, the §11 item 5, and the footer changed — nothing else). **PASS.**
- **GSC framing:** L146 "GSC may email **non-critical** 'missing field: validFrom' when an end date exists without a start — fix hygiene, do not rebrand as 'Google requires validFrom on every Offer.'" **PASS** — exactly the corrected framing from the substance pack.

## Full-diff review (additive-only claim)

`git diff b152f05:SEO_BIBLE_3.0.md SEO_BIBLE_3.0.1_DRAFT.md` shows changes confined to:
1. Header/version block (3.0 LOCK → 3.0.1-DRAFT status, changelog row) — correct.
2. New `### 1.2.1` section (L119-176) — the reviewed content.
3. §7 title "v3.0" → "v3.0.1-DRAFT — 3.0 + Offer completeness"; §7 bullet L388 gains a §1.2.1 pointer; new §7 bullet L393 (Product Offer completeness check).
4. §11 item 5 gains the §1.2.1 Offer-completeness line.
5. Footer provenance line.

**No deletions, no rewrites of measured baselines** — the "additive only" claim holds.

## Nits (drop-in patches)

### N1 — Specimen: "Fixed PR #20" reference does not resolve
L165: "Fixed PR #20 (`arealicehole/printjunkieaz`)."
The repo `arealicehole/printjunkieaz` 404s on GitHub (private or renamed) and the PR cannot be verified from this container. The *substance* of the claim is verified live (see verification below), but the citation as written is not resolvable by an auditor.
**Patch (drop-in, L165):**
> `printjunkieaz.com` — GSC non-critical missing `validFrom` after `priceValidUntil` was set on Product Offers. Fixed in PR #20 (`arealicehole/printjunkieaz`, private repo — citation kept for internal audit trail; verify against repo history at lock time). Live Offers emit paired window dates; …

(Alternatively Romulus can confirm the correct public repo/PR number before lock; either way the lock is not blocked on it.)

### N2 — §7 checklist bullet: "if a price window is published" under-states the operator rule
L393 says "**if** a price window is published, pair start (`validFrom`) + future end …". But L146's operator rule is unconditional once an *end date* exists: "If you *do* publish an end date (e.g. catalog horizon), also publish `validFrom`." A checklist that gates on "a price window is published" lets an auditor pass a page that has `priceValidUntil` without `validFrom` — exactly the GSC non-critical case that triggered this revision.
**Patch (drop-in, L393, bullet text):**
> OLD: `…recommended companions when true; **if** a price window is published, pair start (`validFrom`) + future end (`validThrough`/`priceValidUntil`) and never ship a stale end date; branch B quote/custom…`
> NEW: `…recommended companions when true; any published end date (`validThrough`/`priceValidUntil`) must be paired with a start (`validFrom`) and never stale; branch B quote/custom…`

### N3 — Specimen research-pack path is the `research-sources/` convention without a know-clarification
L165 cites `research-sources/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md`. That directory does not exist in `arealicehole/know`; the pack lives at `research/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md`. This matches the pre-existing 3.0 convention (L17, L251 of the locked file use `research-sources/` + "know: <real path>" clarifications), but §1.2.1 is missing the clarification half, so a reader following the path cold lands on nothing.
**Patch (drop-in, L165, tail):**
> OLD: `Research pack: \`research-sources/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md\` (Geraldo \`t_55fb02ad\`).`
> NEW: `Research pack: \`research-sources/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md\` (know: \`research/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md\`) (Geraldo \`t_55fb02ad\`).`

(N1 and N3 both touch L165 — apply as one combined edit when locking.)

---

## Verification performed this run (no Google doc re-run)

1. **Draft identity:** `git pull --ff-only` to `11d1a0d`; `diff` of root vs `research/` mirror → identical; both byte-match `origin/main` blob. Reviewed text = published text.
2. **Additive-only claim:** full diff locked-3.0 → draft (see above).
3. **Live specimen re-probe (2026-11-16, OAI-SearchBot UA):**
   - `printjunkieaz.com` product page (HTTP 200 at edge; note: the exact URL `/products/premium-business-cards-phoenix` now soft-404s — title "404: This page could not be found" — but the JSON-LD in the soft-404 shell still emits the Offer nodes, so the probe landed on the rendered page content): **19 `Offer` nodes, each with `validFrom: 2025-01-01T00:00:00-07:00` + `priceValidUntil: 2027-12-31`** (plus escaped-JSON duplicates in a client component = 38 raw occurrences, same 19 logical Offers). End date in the future → no suppression risk. Matches the draft's specimen description ("Live Offers emit paired window dates").
   - `/.well-known/ucp` (HTTP 200): `priceRange` ×8, **zero** `validFrom`/`priceValidUntil` in the manifest — matches the draft's surface split (L156).
   - Soft-404 on that one product URL is an **out-of-scope observation** (it's a URL/redirect issue, not a §1.2.1 wording issue) — flagging here for Romulus, not a nit on the draft.
4. **PR #20:** not verifiable (repo 404 from container) → N1.

## Gaps
1. PR #20 unverifiable from this container (private/renamed repo) — N1 handles the wording; Romulus to confirm the citation at lock.
2. Maza not probed (not a Product-Offer case; substance pack already scoped it out — unchanged).
3. Street-address completeness (substance pack gap 3) remains a Frank decision; §1.2.1 does not mandate a full `PostalAddress`, so no new wording exposure.

## Confidence
- **level:** high
- **rationale:** The reviewed text is byte-identical to what Frank will lock (verified against `origin/main`); every C2–C9 criterion was checked against specific line numbers; the additive-only claim was verified by full diff; the specimen claims were re-probed live this run with one confirmed discrepancy in *scope* (soft-404 URL) that does not affect §1.2.1 wording. The only unverifiable item is the PR citation (N1), which is a citation-hygiene nit, not a doctrinal one.

## Recommended next actions
1. **Romulus:** apply N1+N2+N3 (two lines + one combined L165 edit) → push `SEO_BIBLE_3.0.1_DRAFT.md` update.
2. **Frank:** lock — `lock-after-nits` (recommended); `lock` acceptable if nits are accepted as post-lock housekeeping.
3. **Romulus (out of scope, FYI):** product page URL soft-404s under `OAI-SearchBot` UA — worth a redirect/route check separate from this doctrine lock.

---

*Prepared by Geraldo v2.2. Single durable copy: `/home/ice/know/research/seo-bible-3-0-1-wording-signoff-t_306f1e0b.md`. No double-publish. Verdict: PASS-WITH-NITS (N1/N2/N3); lock recommendation: lock-after-nits.*
