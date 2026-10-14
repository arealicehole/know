# Research Report: SEO Bible §8 — News desks / aggregators / legislative trackers

**task_id:** t_505c6bc5  
**published:** 2026-10-14  
**artifact:** `/home/ice/know/SEO_BIBLE_JUNE_2026.md` (v2.1)

---

## objective

Extend the living SEO Bible (`SEO_BIBLE_JUNE_2026.md` on `arealicehole/know`) with a dedicated, auditor-ready section for **news desks, headline aggregators, and legislative trackers** (Root & Rifle–class sites), so media products are not false-failed against restaurant Core 30 or full merchant UCP rules.

## summary

Patched `SEO_BIBLE_JUNE_2026.md` from v2.0 (~5.8KB outline) to **v2.1 (~21KB)** with:

1. Changelog + version bump to 2.1  
2. Site-type routing table (“which section applies”)  
3. Cross-links from UCP, schema, Information Gain, SSR, visual, ACO, and checklist into media rules  
4. Full **§8** covering entity model, when-not-NewsArticle, aggregator Information Gain, **HARD** SSR/first-HTML and first-party feed rules, `/llms.txt`, sitemap retention, affiliate honesty, **UCP gate**, E-E-A-T pages, hotlink policy, explicit non-goals, normative checklist, R&R worked example, and MVP agent bundle  

Reference specimen: live `https://rootandrifle.com/` + repo note `arealicehole/rootandrifle-site` (public API 404 at research time; site live on Vercel). Prior June audit at `/srv/research-output/SEO_BIBLE_AUDIT_REPORT_JUNE_2026.md` confirmed v2.0 base direction.

## plan

| task_id | title | status |
|---------|-------|--------|
| T1 | Orient on kanban task + Bible location | done |
| T2 | Read June Bible + Jan Bible + June audit | done |
| T3 | Spot-check live R&R (robots, llms.txt, home) | done |
| T4 | Author §8 + cross-links + changelog into Bible | done |
| T5 | Publish research report + git commit/push know | done (this file) |
| T6 | Kanban complete with artifact paths | pending |

## claims

### claim_id: C01
**statement:** `SEO_BIBLE_JUNE_2026.md` on know is the living Second Edition and was the correct edit target (not only research-output).  
**evidence_refs:** E01, E02  
**confidence:** high  
**status:** supported

### claim_id: C02
**statement:** Prior June Bible lacked a news-desk/aggregator/legislative-tracker section and treated UCP as universal merchant requirement.  
**evidence_refs:** E01  
**confidence:** high  
**status:** supported

### claim_id: C03
**statement:** Root & Rifle is a live static news desk + Bill Watch + affiliate marketplace on `rootandrifle.com` (Vercel); suitable as §8 specimen/gap pattern.  
**evidence_refs:** E03, E04  
**confidence:** high  
**status:** supported

### claim_id: C04
**statement:** As of 2026-10-14, `rootandrifle.com/llms.txt` returns 404; `robots.txt` allows `/`, disallows thanks and `*.py`, points sitemap.  
**evidence_refs:** E04  
**confidence:** high  
**status:** supported

### claim_id: C05
**statement:** Full merchant UCP must be gated on first-party checkout; media-only nodes should not be false-failed for missing `/.well-known/ucp`.  
**evidence_refs:** E05 (normative Bible design from task body + existing UCP framing)  
**confidence:** high  
**status:** supported (normative product rule)

### claim_id: C06
**statement:** Homepage SSR of headlines and first-party feeds (not Gist-only) are HARD FAIL criteria for agent-layer media desks.  
**evidence_refs:** E05, E03  
**confidence:** high  
**status:** supported (normative; aligns with task acceptance + live R&R home thinness reported in task)

### claim_id: C07
**statement:** Bible v2.1 now includes §8.1–§8.15 with all task-required topics (entity, NewsArticle honesty, gain bar, SSR, feeds, llms.txt, sitemap, affiliate, UCP gate, E-E-A-T, hotlink, non-goals, checklist, R&R pattern).  
**evidence_refs:** E06  
**confidence:** high  
**status:** supported

## evidence_index

| evidence_id | source_label | source_url / path | provenance | note |
|-------------|--------------|-------------------|------------|------|
| E01 | SEO_BIBLE_JUNE_2026.md pre-edit | `/home/ice/know/SEO_BIBLE_JUNE_2026.md` | local read_file full | v2.0, 170 lines, no §8 |
| E02 | know repo listing | `https://github.com/arealicehole/know` | git ls-files + GitHub API contents | tracked SEO_BIBLE_JAN/JUNE |
| E03 | R&R live home | `https://rootandrifle.com/` | web_extract | news desk + Bill Watch + marketplace copy |
| E04 | R&R robots / llms | `https://rootandrifle.com/robots.txt`, `/llms.txt` | curl headers + body | robots 200; llms 404 |
| E05 | Task body t_505c6bc5 | kanban show | dispatcher | acceptance criteria for §8 |
| E06 | SEO_BIBLE_JUNE_2026.md post-edit | `/home/ice/know/SEO_BIBLE_JUNE_2026.md` | write + phrase verify | v2.1, ~21KB, 421 lines |
| E07 | June audit report | `/srv/research-output/SEO_BIBLE_AUDIT_REPORT_JUNE_2026.md` | read_file | prior GIU audit context |

## gaps

- Public GitHub API returned 404 for `arealicehole/rootandrifle-site` contents (private or renamed); specimen relies on live site + task-stated repo path, not full repo tree walk.
- Did not re-run a full multi-URL R&R technical audit (home byte size, schema dump, sitemap body); §8.14 table includes task-provided snapshot signals labeled 2026-10-14 — treat as pattern, refresh if auditing production.
- No Huly issue filed this turn (kanban-only delivery unless operator requests `[HULY]`).
- know was **ahead 6** commits before this change; push may include prior local research commits if pushed together.

## confidence

**level:** high  
**rationale:** Edit is complete against stated acceptance criteria; Bible phrases verified on disk; live R&R spot-check confirms media class and llms gap. Lower only on private-repo code truth and un-reverified numeric home sizes from the task brief.

## recommended_next_actions

1. **Implement R&R pass pattern** from §8.14–§8.15 (SSR home, first-party `/data` + feed, llms.txt, legal/E-E-A-T pages, sitemap retention) on `rootandrifle-site`.
2. Optional: spawn implementer card for R&R; keep Bible as ruler.
3. Quarterly Bible refresh after next core update; keep media §8 when e-com/local rules change.
4. If Huly audit trail required: Julio `[HULY]` status with path to this report + Bible.

## deliverables

- `/home/ice/know/SEO_BIBLE_JUNE_2026.md` — v2.1 living Bible  
- `/home/ice/know/research/seo-bible-news-desk-section-t_505c6bc5.md` — this report  
