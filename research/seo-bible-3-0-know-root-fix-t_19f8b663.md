# SEO Bible 3.0 know-root path fix (t_19f8b663)

**Date:** 2026-11-13
**Actor:** geraldov21 (Frank order via dick-jones)
**Scope:** path hygiene only — no doctrine rewrite, no 3.0.1, no AVP pricing changes

## What was wrong

Know repo had inconsistent Bible placement:

| Edition | Path before |
|---------|-------------|
| Jan 2026 (2.0) | `SEO_BIBLE_JAN_2026.md` (root) |
| June 2.1 | `SEO_BIBLE_JUNE_2026.md` (root) |
| **3.0 LOCK (2026-10-30)** | `research/SEO_BIBLE_3.0.md` only |

Humans opening know root still saw June 2.1 as the "main" Bible; 3.0 looked like a research note. Drift was a lock-day mirror into `research/`, not intentional product design.

## What was done

1. **Promoted 3.0 to know root** — `research/SEO_BIBLE_3.0.md` copied byte-identical to `SEO_BIBLE_3.0.md` at know root.
   - sha256 (pre-move research copy = root copy): `723396117d0f8512a687ff8df771a3bbd61cfa9a76b804a11562a96d93027eb5`
   - Ops ruler `/srv/apps/tricon-projects/seo-bible/SEO_BIBLE_3.0.md` is **not readable from this Docker container** (path absent in container); hash matched the pre-move research mirror instead, per the brief's fallback. Re-verify vs ops ruler on the host if it is ever reachable.
2. **`research/SEO_BIBLE_3.0.md`** replaced with a short pointer stub to `../SEO_BIBLE_3.0.md` + ops path (stub chosen: no tracked inbound markdown links existed — only the lock receipt and `bible-research-loop-process-20261030.md` referenced it, and both were checked/updated; loop-process references the ops workspace path, not the research mirror).
3. **`README.md`** SEO section rewritten: 3.0 = current locked edition (top entry), ops ruler path for audits, Jan + June = prior archives, "single source of truth" language for Jan replaced.
4. **Lock receipt** `research/seo-bible-3-0-lock-20261030.md`: know mirror paths updated to root + research stub.

## Acceptance

- [x] `SEO_BIBLE_3.0.md` exists at know root
- [x] Content byte-identical to pre-move research file (sha256 above); ops ruler not reachable from container — noted
- [x] README lists 3.0 as current; Jan/June archived
- [x] `research/` path is a documented stub (no second full body to drift)
- [x] Lock receipt paths updated
- [x] Committed + pushed (SHA in commit below)

## Non-goals honored

- No doctrine rewrite, no 3.0.1
- No AVP pricing / Dick Jones pack changes
- No live PJ/Maza site changes
- Jan/June archives retained
