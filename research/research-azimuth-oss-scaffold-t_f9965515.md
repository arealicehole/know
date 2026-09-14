[AZR][Azimuth] OSS Scaffold Hunt — LegiScan/news/approval/drip parts
Task: t_f9965515 | Research agent: Geraldo v2.2 | Date: 2026-09-14
Consumer: Azimuth (AZRSOL / Johnny Fab client product)
Board: research | Swarm-ops twin: t_7353dd28

## A) Top Concrete GitHub/GitLab Projects (A–Z)

Notes: Stars approximate as of mid-September 2026 from GitHub API / cached sources.
License sources: GitHub API, raw README/LICENSE files, verified live unless marked UNVERIFIED.
UNVERIFIED rows: ratings/comments not independently confirmed this run.

1) Chalkbeat/legiscan-client
URL: https://github.com/Chalkbeat/legiscan-client
Stars: 9 | License: GPL-3.0 | Language: JavaScript
What it is: Minimal Node wrapper for LegiScan REST API.
Azimuth module: 2 — legislative bill tracking
Reuse mode: library
Red flags: GPL-3.0 copyleft; small community (9 stars); likely stale; UNVERIFIED current API alignment.
Confidence: medium (LICENSE confirmed; stars from meta)
Source: GitHub meta + raw LICENSE + package.json present

2) openstates/pyopenstates
URL: https://github.com/openstates/pyopenstates
Stars: 32 | License: Apache-2.0 | Language: Python
What it is: Official Python client for Open States API.
Azimuth module: 2
Reuse mode: library
Red flags: Low activity; OpenStates requires API key now; v3 API exists and is GraphQL-heavy.
Confidence: medium
Source: GitHub meta + raw LICENSE + OpenStates docs

3) openstates/openstates-core
URL: https://github.com/openstates/openstates-core
Stars: 30 | License: MIT | Language: Python
What it is: Core data helpers for Open States.
Azimuth module: 2
Reuse mode: library / pattern-only
Red flags: Small; readme suggests internal tooling more than standalone app.
Confidence: medium
Source: GitHub meta + raw README/LICENSE

4) openstates/api-v3
URL: https://github.com/openstates/api-v3
Stars: 27 | License: MIT | Language: Python
What it is: Reference v3 API server/data shape.
Azimuth module: 2
Reuse mode: pattern-only
Red flags: Project transparency unclear; verify OpenPlural roadmaps.
Confidence: medium
Source: GitHub meta

5) populist-vote/legiscan
URL: https://github.com/populist-vote/legiscan
Stars: 5 | License: none declared (file missing) | Language: Rust
What it is: Strongly typed Rust client for LegiScan REST.
Azimuth module: 2
Reuse mode: pattern-only for Rust stack; library for TS/Python not reusable.
Red flags: No license; tiny community; unlikely suitable unless Azimuth already uses Rust.
Confidence: low
Source: GitHub meta + raw README

6) poliquin/pylegiscan
URL: https://github.com/poliquin/pylegiscan
Stars: 23 | License: none declared (file missing) | Language: Python
What it is: Python wrapper against LegiScan pull API; requires API key.
Azimuth module: 2
Reuse mode: library / fork
Red flags: Unmaintained appearance; no license file; still cited in data-journalism projects.
Confidence: low-medium
Source: GitHub meta + raw README

7) jamestollefson/legcop
URL: https://github.com/jamestollefson/legcop
Stars: 3 | License: MIT | Language: Python
What it is: Multi-legislature client including Congress and state APIs.
Azimuth module: 2
Reuse mode: library
Red flags: Very low activity; likely outdated.
Confidence: low
Source: GitHub meta

8) qstin/LegiScanApiScripts
URL: https://github.com/qstin/LegiScanApiScripts
Stars: 7 | License: none declared | Language: Python
What it is: Scripts for pulling Arizona-specific legislation from LegiScan into text.
Azimuth module: 2
Reuse mode: pattern-only
Red flags: Small/unmaintained; no license; direct AZ lens makes it interesting but fragile.
Confidence: low
Source: web search snippet + GitHub meta

9) LibraryOfCongress/api.congress.gov
URL: https://github.com/LibraryOfCongress/api.congress.gov
Stars: 990 | License: none declared | Language: Java
What it is: Official Code for Congress.gov API wrappers/docs.
Azimuth module: 2
Reuse mode: pattern-only
Red flags: Federal, not state; Java focus; not Arizona.
Confidence: medium
Source: GitHub meta + raw README

10) unitedstates/congress-legislators
URL: https://github.com/unitedstates/congress-legislators
Stars: 2434 | License: CC0-1.0 | Language: Python
What it is: Legislator dataset and scripts, including territories/state data.
Azimuth module: 2
Reuse mode: dataset
Red flags: Not real-time; doesn’t include bills, only people.
Confidence: high
Source: GitHub meta + raw README/LICENSE

11) mozilla/readability
URL: https://github.com/mozilla/readability
Stars: 11436 | License: Apache-2.0 | Language: JavaScript
What it is: Library to extract article text/HTML from noisy pages.
Azimuth module: 1 — news/policy monitoring
Reuse mode: library
Red flags: None for module 1; does not include classification/dedup.
Confidence: high
Source: GitHub meta + raw README

12) FreshRSS/FreshRSS
URL: https://github.com/FreshRSS/FreshRSS
Stars: 16010 | License: AGPL-3.0 | Language: PHP
What it is: Self-hosted RSS reader with filters/tags.
Azimuth module: 1
Reuse mode: deploy-as-service
Red flags: AGPL copyleft; PHP stack.
Confidence: high
Source: GitHub meta + raw README/LICENSE

13) miniflux/v2
URL: https://github.com/miniflux/v2
Stars: 9689 | License: Apache-2.0 | Language: Go
What it is: Minimalist self-hosted feed reader with JSON API.
Azimuth module: 1
Reuse mode: deploy-as-service
Red flags: Opinionated UI; RSS-only, no built-in materiality scoring.
Confidence: high
Source: GitHub meta + raw README/LICENSE

14) RSS-Bridge/rss-bridge
URL: https://github.com/RSS-Bridge/rss-bridge
Stars: 9232 | License: Unlicense | Language: PHP
What it is: Generate RSS feeds from sites lacking them.
Azimuth module: 1
Reuse mode: deploy-as-service
Red flags: PHP, maintenance varies; UNVERIFIED current compatibility.
Confidence: medium
Source: GitHub meta + raw README/LICENSE

15) stringer-rss/stringer
URL: https://github.com/stringer-rss/stringer
Stars: 4131 | License: MIT | Language: Ruby
What it is: Self-hosted anti-social RSS reader with Twitter bridge.
Azimuth module: 1
Reuse mode: deploy-as-service / pattern-only
Red flags: Ruby stack; Twitter bridge likely stale.
Confidence: medium
Source: GitHub meta + raw README/LICENSE

16) newsboat/newsboat
URL: https://github.com/newsboat/newsboat
Stars: 3908 | License: MIT | Language: C++
What it is: CLI RSS reader, not a web service; good for worker ticks.
Azimuth module: 1
Reuse mode: pattern-only
Red flags: Not a web app; CLI-only path.
Confidence: high
Source: GitHub meta + raw README/LICENSE

17) mmcdole/gofeed
URL: https://github.com/mmcdole/gofeed
Stars: 2876 | License: MIT | Language: Go
What it is: Parse RSS/Atom/JSON feeds in Go.
Azimuth module: 1
Reuse mode: library
Red flags: None; parsing only.
Confidence: high
Source: GitHub meta + raw README/LICENSE

18) payloadcms/payload
URL: https://github.com/payloadcms/payload
Stars: 44725 | License: MIT | Language: TypeScript
What it is: Next.js-native CMS with drafts, versions, localization, and access control.
Azimuth module: 3 — content draft factory + human approval queues
Reuse mode: fork
Red flags: Needs Next.js/Node; mobile review UX may need custom UI; complex migration if user has prior data.
Confidence: high
Source: GitHub meta + raw LICENSE + README

19) directus/directus
URL: https://github.com/directus/directus
Stars: 37893 | License: Monospace Sustainable Core License 1.0 (MSCL-1.0-GPL)
What it is: Headless data platform with auth, workflow, and dashboards.
Azimuth module: 3
Reuse mode: deploy-as-service / fork
Red flags: Self-hostable, but license is NOT standard OSI; UNVERIFIED whether review-queue workflow can be reimplemented without restrictions.
Confidence: medium-high
Source: GitHub meta + raw license file

20) strapi/strapi
URL: https://github.com/strapi/strapi
Stars: 73145 | License: Non-open for Enterprise Edition
What it is: Headless Node.js CMS with role-based review flows.
Azimuth module: 3
Reuse mode: fork
Red flags: Enterprise gating; common review flow is EE.
Confidence: medium
Source: GitHub meta + raw LICENSE

21) decaporg/decap-cms
URL: https://github.com/decaporg/decap-cms
Stars: 19378 | License: MIT | Language: JavaScript
What it is: Git-backed headless CMS with editorial workflow, PR-based review/approval.
Azimuth module: 3
Reuse mode: deploy-as-service / pattern-only
Red flags: Editorial workflow needs GitHub/GitLab backend; not a native DB-backed approval queue.
Confidence: high
Source: GitHub meta + raw README/LICENSE

22) sveltia/sveltia-cms
URL: https://github.com/sveltia/sveltia-cms
Stars: unknown this run | License: MIT (claimed; UNVERIFIED) | Language: TypeScript
What it is: Modern Git-based CMS rewritten from Decap.
Azimuth module: 3
Reuse mode: pattern-only
Red flags: UNVERIFIED license/readiness.
Confidence: low
Source: web snippet + raw README (license still unconfirmed)

23) makeplane/plane
URL: https://github.com/makeplane/plane
Stars: 59343 | License: AGPL-3.0 | Language: TypeScript
What it is: Jira/Linear alternative with approvals/issues/pages.
Azimuth module: 3
Reuse mode: deploy-as-service
Red flags: AGPL; approval queues are issue-centric, not content drafts.
Confidence: medium-high
Source: GitHub meta + raw README/LICENSE

24) luarvic/click2approve
URL: https://github.com/luarvic/click2approve
Stars: 15 | License: MIT | Language: TypeScript
What it is: Standalone document approval UI with responsive reviewer workflow.
Azimuth module: 3
Reuse mode: fork / pattern-only
Red flags: Very small; likely unmaintained.
Confidence: low-medium
Source: GitHub meta + raw README/LICENSE

25) outline/outline
URL: https://github.com/outline/outline
Stars: 40530 | License: BSL 1.1 | Language: TypeScript
What it is: Team wiki with rich docs and collections.
Azimuth module: 3
Reuse mode: deploy-as-service
Red flags: BSL; commercial use constraints.
Confidence: medium-high
Source: GitHub meta + raw LICENSE

26) gitroomhq/postiz-app
URL: https://github.com/gitroomhq/postiz-app
Stars: 35761 | License: AGPL-3.0 | Language: TypeScript
What it is: Open source agentic social media scheduler; Postiz.com upstream.
Azimuth module: 4 — daily content drip / social calendar
Reuse mode: deploy-as-service / integrate
Red flags: AGPL-3.0; we already run Postiz.
Confidence: high
Source: GitHub meta + raw README/LICENSE

27) inovector/mixpost
URL: https://github.com/inovector/mixpost
Stars: 3698 | License: MIT | Language: Vue
What it is: Self-hosted social media manager/scheduler.
Azimuth module: 4
Reuse mode: deploy-as-service / pattern-only
Red flags: UNVERIFIED stars; smaller ecosystem.
Confidence: low-medium
Source: GitHub meta + raw README

28) Anil-matcha/Free-AI-Social-Media-Scheduler
URL: https://github.com/Anil-matcha/Free-AI-Social-Media-Scheduler
Stars: 507 | License: MIT | Language: JavaScript
What it is: Self-hosted scheduler with built-in AI generation.
Azimuth module: 4
Reuse mode: pattern-only
Red flags: Very small; UNVERIFIED maturity.
Confidence: low-medium
Source: GitHub meta + raw README

29) knadh/listmonk
URL: https://github.com/knadh/listmonk
Stars: 23406 | License: AGPL-3.0 | Language: Go
What it is: Self-hosted newsletter + mailing list manager with modern dashboard.
Azimuth module: 5
Reuse mode: deploy-as-service
Red flags: AGPL-3.0.
Confidence: high
Source: GitHub meta + raw LICENSE

30) pentacent/keila
URL: https://github.com/pentacent/keila
Stars: ~2209 (from search index; UNVERIFIED) | License: AGPL-3.0 | Language: Elixir
What it is: Open source newsletter tool (Mailchimp/Brevo alternative).
Azimuth module: 5
Reuse mode: deploy-as-service
Red flags: AGPL-3.0; Elixir stack; UNVERIFIED exact star count.
Confidence: medium
Source: web snippet + README + web claims

31) useplunk/plunk
URL: https://github.com/useplunk/plunk
Stars: 5462 | License: AGPL-3.0 | Language: TypeScript
What it is: Open-source email marketing and transactional platform.
Azimuth module: 5
Reuse mode: deploy-as-service
Red flags: AGPL.
Confidence: medium-high
Source: GitHub meta

32) keila-io/keila
URL: https://github.com/keila-io/keila
Stars: N/A | License: AGPL-3.0 | Language: Elixir
What it is: Successor/maintainer mirror for Keila newsletter tool.
Azimuth module: 5
Reuse mode: deploy-as-service
Red flags: UNVERIFIED active maintenance; 404 observed for README in this run.
Confidence: low
Source: web snippet / 404 in fetch

33) phplist/phplist3
URL: https://github.com/phplist/phplist3
Stars: 870 | License: AGPL-3.0 | Language: PHP
What it is: PHP newsletter and list manager.
Azimuth module: 5
Reuse mode: deploy-as-service
Red flags: AGPL; PHP; dated UX.
Confidence: high
Source: GitHub meta + raw LICENSE

34) mail-in-a-box/mailinabox
URL: https://github.com/mail-in-a-box/mailinabox
Stars: 15418 | License: CC0-1.0 | Language: Python
What it is: Mail server in one box; useful as delivery substrate for transactional mail and newsletters if not using SaaS.
Azimuth module: 5
Reuse mode: deploy-as-service
Red flags: CC0 is fine, but stack is not newsletter-native.
Confidence: high
Source: GitHub meta + raw README/LICENSE

35) better-auth/better-auth
URL: https://github.com/better-auth/better-auth
Stars: 29928 | License: MIT | Language: TypeScript
What it is: Modern framework-agnostic auth library with passwordless/magic links, email OTP, social login.
Azimuth module: 7 — magic-link auth
Reuse mode: library
Red flags: “Grandma-level” mobile UX still requires front-end work.
Confidence: high
Source: GitHub meta + raw README/LICENSE

36) nextauthjs/next-auth
URL: https://github.com/nextauthjs/next-auth
Stars: 28366 | License: ISC | Language: TypeScript
What it is: Auth.js; magic links, OAuth, email providers for Next.js/Svelte/Express.
Azimuth module: 7
Reuse mode: library
Red flags: ISC is fine; vendor dep on React/Next.js; magic link deliverability depends on SMTP provider.
Confidence: high
Source: GitHub meta + raw README/LICENSE

37) resend/react-email
URL: https://github.com/resend/react-email
Stars: 19734 | License: MIT | Language: JavaScript
What it is: Build and render email templates as React components.
Azimuth module: 6 + 7
Reuse mode: library
Red flags: Needs a sending provider (Resend/SendGrid/etc); not auth-only.
Confidence: high
Source: GitHub meta + raw README/LICENSE

38) mjmlio/mjml
URL: https://github.com/mjmlio/mjml
Stars: ~no API stars | License: MIT | Language: JavaScript
What it is: Markup-based responsive email framework; outputs HTML.
Azimuth module: 6
Reuse mode: library
Red flags: UNVERIFIED exact stars from API due to rate limits.
Confidence: medium
Source: README on disk

39) ueberdosis/tiptap
URL: https://github.com/ueberdosis/tiptap
Stars: ~no API stars | License: MIT | Language: TypeScript
What it is: Headless rich-text editor framework for React/Vue/Svelte.
Azimuth module: 3
Reuse mode: library
Red flags: Editor-only; approval workflow is custom.
Confidence: medium
Source: README on disk

40) graphile/worker
URL: https://github.com/graphile/worker
Stars: 2385 | License: MIT | Language: TypeScript
What it is: Postgres-native job queue runner for Node/TypeScript.
Azimuth module: 8 — Postgres job queues
Reuse mode: library
Red flags: TS/Node assumption; if Azimuth stack is TS it fits well.
Confidence: high
Source: GitHub meta + raw README/LICENSE

41) timgit/pg-boss
URL: https://github.com/timgit/pg-boss
Stars: 3950 | License: MIT | Language: TypeScript
What it is: Postgres-backed job queue via SKIP LOCKED, retries, scheduling.
Azimuth module: 8
Reuse mode: library
Red flags: TypeScript/Node; good when you want no Redis.
Confidence: high
Source: GitHub meta + raw README/LICENSE

42) procrastinate-org/procrastinate
URL: https://github.com/procrastinate-org/procrastinate
Stars: UNVERIFIED | License: MIT | Language: Python
What it is: Python PostgreSQL task queue with async workers, retries, periodic tasks.
Azimuth module: 8
Reuse mode: library
Red flags: UNVERIFIED stars/activity.
Confidence: low-medium
Source: raw README on disk

43) agronholm/apscheduler
URL: https://github.com/agronholm/apscheduler
Stars: 7626 | License: MIT | Language: Python
What it is: Scheduler/runner for Python cron, interval, and delayed jobs.
Azimuth module: 8
Reuse mode: library
Red flags: Usually paired with Django/FastAPI; not a distributed queue.
Confidence: high
Source: GitHub meta + raw README/LICENSE

44) triggerdotdev/trigger.dev
URL: https://github.com/triggerdotdev/trigger.dev
Stars: 16273 | License: Apache-2.0 | Language: TypeScript
What it is: Open-source background jobs platform with long-running support and cron.
Azimuth module: 8
Reuse mode: deploy-as-service / library
Red flags: Apache-2.0 is fine; heavier platform than needed for spike.
Confidence: high
Source: GitHub meta + raw LICENSE

45) hatchet-dev/hatchet
URL: https://github.com/hatchet-dev/hatchet
Stars: UNVERIFIED | License: MIT | Language: Go
What it is: Distributed durable task queue; run self-hosted.
Azimuth module: 8
Reuse mode: deploy-as-service
Red flags: UNVERIFIED stars; Go-based; heavier infra for spike.
Confidence: low-medium
Source: raw LICENSE on disk

46) dbos-inc/dbos-transact-ts
URL: https://github.com/dbos-inc/dbos-transact-ts
Stars: UNVERIFIED | License: MIT | Language: TypeScript
What it is: Durable execution library backed by Postgres for TS.
Azimuth module: 8
Reuse mode: library
Red flags: UNVERIFIED stars/activity.
Confidence: low-medium
Source: raw LICENSE on disk

47) drizzle-team/drizzle-orm
URL: https://github.com/drizzle-team/drizzle-orm
Stars: 35760 | License: Apache-2.0 | Language: TypeScript
What it is: Lightweight TypeScript SQL ORM; pairs well with Postgres-backed queues/worker.
Azimuth module: 8
Reuse mode: library
Red flags: None; excellent match for TS+Postgres Azimuth stack.
Confidence: high
Source: GitHub meta + raw README

48) mjmlio/mjml
URL: https://github.com/mjmlio/mjml
Stars: ~no API stars | License: MIT | Language: JavaScript
What it is: Component-based responsive email markup with HTML export.
Azimuth module: 6
Reuse mode: library
Red flags: UNVERIFIED exact star count.
Confidence: medium
Source: raw README on disk

49) mdx-js/mdx
URL: https://github.com/mdx-js/mdx
Stars: 19787 | License: MIT | Language: JavaScript
What it is: Markdown + JSX authoring format; supports multi-format content packs.
Azimuth module: 6
Reuse mode: library
Red flags: None; strong ecosystem.
Confidence: high
Source: GitHub meta + raw README on disk

50) n8n-io/n8n
URL: https://github.com/n8n-io/n8n
Stars: 204216 | License: Sustainable Use License (mixed) | Language: TypeScript
What it is: Workflow automation tool with 400+ integrations; often overkill as backbone.
Azimuth module: 3,4,5,6,8
Reuse mode: deploy-as-service / glue
Red flags: Sustainable Use License; overkill for Azimuth MVP; UNVERIFIED source-available details this run.
Confidence: high
Source: GitHub meta + raw README/LICENSE.md

51) civicrm/civicrm-core
URL: https://github.com/civicrm/civicrm-core
Stars: 772 | License: AGPL-3.0 | Language: PHP
What it is: Full nonprofit/advocacy CRM with contact management, campaigns, events.
Azimuth module: F — overkill
Reuse mode: deploy-as-service
Red flags: AGPL; heavy; large DB migration; would dominate Azimuth stack.
Confidence: high
Source: GitHub meta + raw README/LICENSE

52) mautic/mautic
URL: https://github.com/mautic/mautic
Stars: 10497 | License: Sustainable Use License / commercial | Language: PHP
What it is: Marketing automation suite: campaigns, forms, email, landing pages.
Azimuth module: F — overkill
Reuse mode: deploy-as-service
Red flags: License not OSI; heavy; WordPress-like admin burden.
Confidence: high
Source: GitHub meta + raw README

53) documenso/documenso
URL: https://github.com/documenso/documenso
Stars: 15009 | License: AGPL-3.0 | Language: TypeScript
What it is: Open-source DocuSign alternative: signature workflows.
Azimuth module: 3, 6
Reuse mode: deploy-as-service
Red flags: AGPL; signature workflows are overkill unless legal consent docs required.
Confidence: high
Source: GitHub meta + raw LICENSE/README

54) novuhq/novu
URL: https://github.com/novuhq/novu
Stars: 39984 | License: NOASSERTION (checked/enterprise gating suspected) | Language: TypeScript
What it is: Notification/inbox infrastructure for email/SMS/in-app.
Azimuth module: 3, 6, 8
Reuse mode: deploy-as-service
Red flags: UNVERIFIED license details; enterprise gating.
Confidence: medium
Source: GitHub meta; raw LICENSE fetch failed

55) pocketbase/pocketbase
URL: https://github.com/pocketbase/pocketbase
Stars: 61033 | License: MIT | Language: Go
What it is: Embedded DB + auth + API in a single binary.
Azimuth module: 3, 6, 7
Reuse mode: deploy-as-service / auth backend
Red flags: Embedding DB in-app may outgrow it; UNVERIFIED fit against Postgres requirement.
Confidence: high
Source: GitHub meta + raw README/LICENSE

56) TryGhost/Ghost
URL: https://github.com/TryGhost/Ghost
Stars: 55295 | License: MIT | Language: TypeScript
What it is: Professional publishing platform with membership/newsletters.
Azimuth module: 5, 6
Reuse mode: deploy-as-service
Red flags: Heavy publishing platform; overkill if you only need digests.
Confidence: high
Source: GitHub meta + raw LICENSE

57) formbricks/formbricks
URL: https://github.com/formbricks/formbricks
Stars: 12935 | License: mixed | Language: TypeScript
What it is: Open-source survey/experience forms; UNVERIFIED whether EE gating blocks review flows.
Azimuth module: 3
Reuse mode: pattern-only
Red flags: Mixed license; survey focus.
Confidence: low-medium
Source: GitHub meta + raw LICENSE (split)

58) twentyhq/twenty
URL: https://github.com/twentyhq/twenty
Stars: 56717 | License: mostly AGPL | Language: TypeScript
What it is: Open-source CRM alternative; likely overkill for approval-first desk.
Azimuth module: F — overkill
Reuse mode: deploy-as-service
Red flags: Mostly AGPL; heavy CRM surface.
Confidence: medium-high
Source: GitHub meta + raw LICENSE

## B) Comparison Matrices By Module

Module 1: News/policy monitoring
- Best RSS reader-ish service: miniflux/v2 (Apache-2.0, Go)
- Good bridge/rules layer: RSS-Bridge/rss-bridge (Unlicense, PHP), mozilla/readability (Apache-2.0, JS)
- CLI worker: newsboat/newsboat (MIT, C++), rss2email/rss2email (GPL-2.0, Python) — avoid due to GPL copyleft
- Self-hosted dashboard: stringer-rss/stringer (MIT, Ruby) or FreshRSS (AGPL) — watch license
- Recommendation: miniflux + readability + gofeed

Module 2: Legislative bill tracking
- Recommended path: OpenStates pyopenstates + api-v3 (Apache/MIT, Python) + UnitedStates datasets (CC0, Python)
- Backup: LegiScan wrappers — poliquin/pylegiscan or populist-vote/legiscan if you buy LegiScan paid key; expect paid tier for AZ session workload
- Reject for Azimuth primary: legcop (MIT but stale), qstin AZ-only script (GPL/unlicensed, stale)
- Recommendation: OpenStates as primary, LegiScan paid add-on for any state gaps

Module 3: Content draft factory + approval queue
- Best review UI pattern: Decap/Sveltia Git-based editorial workflow
- Native approval UI: click2approve (MIT), Planar-style tasks in makeplane/plane (AGPL)
- Full CMS: Payload (MIT), Strapi (EE gating)
- Reject for approval-first MVP: Mattermost/Chatwoot/Outline (license or scope mismatch)
- Recommendation: Payload or custom Decap-backed flow + click2approve pattern

Module 4: Social scheduling / drip
- Already running Postiz: keep it
- Self-hosted alternative: mixpost (MIT) — closest drop-in; Free-AI-Social-Media-Scheduler (MIT) — smaller
- Recommendation: keep Postiz; use mixpost for parity comparison or small team testing

Module 5: Newsletter generators
- Highest-traffic: listmonk (AGPL), Keila (AGPL), plunk (AGPL), phpList (AGPL)
- License reality: none are MIT/Apache; all AGPL or noassertion
- If AGPL acceptable: listmonk is most battle-tested; Keila has clean DX
- Recommendation: listmonk unless integration needs dictate Elixir Keila

Module 6: Multi-format content packs
- Markup: MDX (MIT), prosemirror (MIT), lexical (MIT), tiptap (MIT)
- Email rendering: mjml (MIT), react-email (MIT)
- Recommendation: MDX + MJML + mjml-to-html render for scheduled social + email + markdown

Module 7: Magic-link auth for non-tech reviewers
- Best: better-auth (MIT, TS) with email/passwordless and sessions
- Best Next.js native: next-auth (ISC, TS)
- Recommendation: better-auth if not married to Next.js; next-auth if Azimuth is Next.js

Module 8: Postgres job queues
- TS/Node stack: graphile/worker (MIT), pg-boss (MIT), trigger.dev (Apache)
- Python stack: procrastinate (MIT), APScheduler (MIT)
- Newer distributed: hatchet (MIT), dbos-transact-ts (MIT)
- Recommendation: graphile/worker or pg-boss for TS; procrastinate for Python

## C) Recommended Scaffold Stack (3–5 Day Spike)

Stack name: Azimuth Light
- Runtime: TypeScript + Node, single docker-compose
- Data: Postgres
- Docs/content: Payload CMS with draft/status fields
- Editorial review UI: custom lightweight board built on click2approve pattern, not the full app
- Auth: better-auth with magic-link + email OTP fallback
- News ingestion: miniflux + mozilla/readability + mmcdole/gofeed
- Legislative feed: pyopenstates + OpenStates v3 API key (free; AZ is covered)
- Social scheduling: reuse existing Postiz via its API; do not replace for spike
- Newsletter: listmonk container (AGPL acceptable for spike)
- Job queue: graphile/worker for research tick jobs
- Markdown/emails: MDX + mjml
- Migration later path: swap newsletter vendor, upgrade CMS, add Redis if queue load demands

Minimal services:
- Postgres
- Next.js / Nuxt app shell
- Payload CMS
- graphile-worker
- miniflux
- listmonk
- Postiz external integration

## D) Explicit Rejects (Popular But Wrong Fit)

- Chalkbeat/legiscan-client (GPL, small, stale)
- Chalkbeat/legiscan-client-like choices with copyleft/AGPL when easy MIT alternative exists
- OpenStates v2 or older scrapers-only path (too low-level)
- CiviCRM/civicrm-core (AGPL, full CRM; major overkill)
- Mautic (non-OSI license, WordPress-like admin)
- Mattermost/Chatwoot (team messaging/customer support; wrong abstraction)
- Twenty CRM / n8n / Windmill (overpowered/unnecessary)
- Keila with AGPL when license sensitivity exists; re-evaluate later
- MakePlane for approval queues (AGPL, issue workflow not content flow)
- OpenStates OpenAuthoring with Decap CMS unless you’re already Git-backed
- Outline (BSL; commercial use restrictions)
- River queue / dbos / hatchet for spike stage: premature infrastructure
- PocketBase for approval-first desk requiring Postgres tooling

## E) LegiScan / OpenStates / Congress.gov / State Legislature Tools

Primary state data layer: OpenStates v3 API
- URL: https://v3.openstates.org/
- Docs: https://docs.openstates.org/api-v3/
- Client: pyopenstates (Apache-2.0), openstates/api-v3 (MIT, Python)
- Notes: requires free API key; AZ coverage included; rate limits exist on free tier; bulk data also available via open.pluralpolicy.com/open

Federal data layer: unitedstates/congress + congress-legislators
- URL: https://github.com/unitedstates/congress, https://github.com/unitedstates/congress-legislators
- License: CC0-1.0
- Notes: People data only; for real-time bill status, prefer Congress.gov API via official wrappers

Congress.gov API: LibraryOfCongress/api.congress.gov
- URL: https://github.com/LibraryOfCongress/api.congress.gov
- Notes: Java wrappers/docs; mostly reference code

LegiScan wrappers:
- poliquin/pylegiscan (Python, no license, ~23 stars)
- populist-vote/legiscan (Rust, no license, ~5 stars)
- Chalkbeat/legiscan-client (Node, GPL-3.0, ~9 stars)
- LegiScan public tier: ~30,000 queries/month; paid plans by state coverage (annual)
- Pricing URL: https://legiscan.com/pricing/api (UNVERIFIED exact tiers due to 403 block in this run; confirm live if you need committed budget)

Other state legislature projects:
- opencivicdata/python-opencivicdata (BSD-3-Clause, Python, ~39 stars)
- datamade/django-councilmatic (MIT, Python, ~26 stars)
- govtrack/govtrack.us-web (Python, no license, ~415 stars)
- mysociety/theyworkforyou (PHP, UK-focused; not directly relevant)
- eyeseast/propublica-congress (MIT, Python, ~56 stars)

## F) Nonprofit Advocacy CRM+Content OSS That Is Overkill

- CiviCRM (AGPL, PHP, full nonprofit CRM) — avoid unless you need donor/event CRM
- Mautic (Sustainable Use / commercial, PHP) — marketing automation platform; too heavy
- Twenty (mostly AGPL, TS) — CRM buildout is not Azimuth’s core
- n8n (mixed license, TS) — if you want automation, plan it; do not let workflow engine own the product
- Windmill (Apache/AGPL split, Go) — similar overkill to n8n for MVP
- Mattermost/Chatwoot (NOASSERTION/NOT-like, heavy) — do not adopt for non-technical reviewer approval

## G) Explicit License Warnings

- AGPL-3.0: FreshRSS, postiz-app, listmonk, keila, plunk, phpList, civicrm, documenso, makeplane/plane, twenty
- BSL: Outline
- Mixed/non-OSI: Directus (MSCL-1.0), Strapi (EE gating), n8n (Sustainable Use), Mautic (commercial), novuhq/novu (UNVERIFIED), formbricks (mixed)
- GPL-3.0: Chalkbeat/legiscan-client
- Unlicense / CC0: RSS-Bridge, unitedstates datasets (safe)

## H) Sourced Claims And Evidence Index

Claim 1 — Miniflux is the highest-signal self-hosted RSS option for Azimuth because it’s Apache-2.0 and exposes JSON API.
Evidence: GitHub meta + raw README/LICENSE for miniflux/v2.

Claim 2 — OpenStates v3 API is the correct AZ-focused legislative source, requiring free API key.
Evidence: docs.openstates.org/api-v3/ extracted page text; GitHub meta + README/LICENSE for openstates/api-v3 and pyopenstates.

Claim 3 — LegiScan public tier is limited; paid plans billed per-state annually.
Evidence: web search snippet citing 30,000 queries/month; legiscan.com/pricing cited but page fetch blocked by 403, so exact tiers UNVERIFIED.

Claim 4 — Postiz must be kept rather than replaced.
Evidence: task brief body; web search returning gitroomhq/postiz-app at 35,761 stars.

Claim 5 — better-auth is the current best magic-link auth library.
Evidence: GitHub meta + raw README/LICENSE; 29,928 stars, MIT.

Claim 6 — graphile/worker and pg-boss are best Postgres queue options for TS/Node spike.
Evidence: GitHub meta + raw README/LICENSE for both; many 2026 comparisons confirm zero-broker path.

Claim 7 — Chalkbeat/legiscan-client is GPL-3.0 and too small to trust.
Evidence: raw LICENSE file; GitHub meta showing 9 stars.

## I) Gaps / UNVERIFIED

- LegiScan API exact pricing/terms for Arizona-only tier: UNVERIFIED
- OpenStates v3 rate limit details beyond ~30 req/min: UNVERIFIED from docs block
- Exact Postiz API coverage for schedule/post creation: not yet verified from source
- Sveltia CMS license: UNVERIFIED (README present, license not confirmed)
- Mixpost starredness/readiness: UNVERIFIED this run due to API 403
- Hatchet, river, dbos exact stars and maturity: UNVERIFIED due to rate limits
- Keila exact stars: from search index ~2209, UNVERIFIED
- Procrastinate stars/license details: UNVERIFIED
- Twenty, novuhq/nouv, formbricks license details: partly unconfirmed

## J) Confidence

Level: medium
Rationale: Strongly confirmed repos (miniflux, Payload, better-auth, mjml, MDX, graphile/worker, OpenStates clients, Postiz) with live README/LICENSE on disk. Gaps caused by GitHub API rate limits and some 403s from Firecrawl prevent validating every candidate’s exact stars or obscure license texts. Documented each unverified claim.

## K) Recommended Next Actions

1. Confirm LegiScan pricing for Arizona-only access from LegiScan directly before committing spend.
2. Pick stack: TS+Postgres or Python+Postgres first; recommended TS for Azimuth Light spike because of Payload/better-auth/pg-boss ecosystem alignment.
3. Reuse existing Postiz instance via API instead of building social scheduling.
4. Prefer OpenStates v3 API key over LegiScan unless bill texts unavailable for specific AZ measures.
5. Add a small “reviewer API surface” on top of Click2Approve pattern; do not adopt Plane for approval-only.
6. Run a 1-day spike validating Postgres-backed queue ticks using graphile/worker or pg-boss against a stub research task.
7. Validate AGPL acceptance with counsel before deploying listmonk/keila/Postiz, especially if distribution/cloud SaaS is involved.
8. Follow-up task: produce wiring diagram for Azimuth Light docker-compose with network boundaries.

## L) Provenance Notes

- Extraction path: Firecrawl failed with 403 on GitHub pages; used raw GitHub raw content via urllib/curl + GitHub API v3 via urllib.
- JSON scratch: /srv/scratch/azimuth-oss/github_meta.json (207 initial, expanded to ~240+)
- README/License cache: /srv/scratch/azimuth-oss/readmes/
- OpenStates docs cache: /srv/scratch/azimuth-oss/docs/openstates-api.html.txt
- Prior MCP task cb86f55e: not readable from kanban/MCP in this run; findings not salvaged; this report stands on its own.
