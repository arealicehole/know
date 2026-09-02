# Tomato / Postiz / Echo credential-pattern mining

**Task:** `t_2a7ac118`  
**Product:** Tomato — approval-first social command center (TOM-1)  
**Date:** 2026-09-02  
**Author:** Geraldo v2.2 (GIU)  
**Secrets policy:** env var **names**, paths, and patterns only — **no secret values**

---

## objective

Mine internal Postiz + Echo operational knowledge so Tomato implementers know:

1. What credential / env / callback patterns already work on this stack
2. What Frank likely already has from Postiz that can map into Tomato
3. Whether Tomato should integrate with Postiz, reuse patterns only, or stay standalone
4. What Tomato must **not** copy from Postiz/Echo product design

## summary

**Recommendation: Tomato stays standalone for OAuth + publish.** Reuse Postiz **provider-app credential patterns** (Meta App ID/Secret, TikTok client key/secret, HTTPS public origin, callback registration discipline). Do **not** route Tomato publish through Postiz API/MCP for the MVP — Postiz is a multi-channel scheduler with org-level API keys, Temporal workers, and agent middleware scoping; Tomato is an approval-first command center with its own encrypted token store and operator gate.

**Frank wake-up paste map (high confidence if Postiz providers were ever connected):**

| Frank likely has (Postiz / Meta / TikTok portals) | Tomato env (from sibling contract) |
|---|---|
| Meta **App ID** (`FACEBOOK_APP_ID` in Postiz `.env`) | `META_APP_ID` |
| Meta **App Secret** (`FACEBOOK_APP_SECRET`) | `META_APP_SECRET` |
| TikTok **Client Key** (`TIKTOK_CLIENT_ID` in Postiz) | `TIKTOK_CLIENT_KEY` |
| TikTok **Client Secret** (`TIKTOK_CLIENT_SECRET`) | `TIKTOK_CLIENT_SECRET` |
| Postiz public origin / tunnel hostname pattern | `TOMATO_PUBLIC_BASE_URL` + `TOMATO_APP_URL` (new hostname; **new** callback paths) |
| Postiz org API key / middleware tokens (`POSTIZ_API_KEY`, `pk_echo_full`, …) | **Not used by Tomato** (unless optional later “schedule via Postiz” adapter) |

**Critical:** Even if Frank reuses the **same Meta/TikTok developer apps**, he must **add Tomato’s callback URIs** alongside Postiz’s. Callbacks are path-specific and exact-match. Reusing app credentials ≠ reusing redirect URIs.

**Echo:** Not a separate credential vault. Echo is an agent consumer of **Postiz** (full-platform scoped middleware token `pk_echo_full`) plus Hermes social skills. Tomato should copy Echo’s **approval/scoping philosophy** (don’t give agents raw master keys), not Echo’s Postiz-shaped integration.

**Public URL pattern proven on this host:** named **Cloudflare Tunnel** hostname → local bind (documented in `research-env-contract-t_2d125f8e.md`). Prefer that over `*.trycloudflare.com` quick tunnels for anything registered in provider consoles.

## plan (executed)

1. Claim kanban task; read attachment + sibling Tomato research (`t_2d125f8e`, `t_8c8d1279`, `t_b8a7d332`).
2. Load internal skills: `postiz`, `postiz-deployment`.
3. Mine sessions for Postiz middleware / Echo / OAuth reconnect / social stack.
4. Mine durable research under `/srv/research-output/` and `/srv/scratch/know/postiz-*`.
5. Cross-check official Postiz provider docs (Facebook/Instagram/TikTok) for env names + callback paths.
6. Note container limits: `/srv/apps` and `/srv/app-data` **not mounted** in this worker — live Postiz `.env` not re-read; skill + prior research used as source of record.
7. Write this report; attach; findings comment; complete.

---

## findings

### 1. Postiz deployment footprint (this server)

| Item | Pattern / location (no secrets) |
|---|---|
| App code | `/srv/apps/postiz/` |
| Data | `/srv/app-data/postiz/` (`config`, `uploads`, postgres, redis, temporal…) |
| Compose | `docker compose -f docker-compose.yaml -f docker-compose.override.yaml` |
| UI / API | host port **4007** → container nginx **5000**; API under `/api` |
| Temporal UI | host **8080** |
| Registration | `DISABLE_REGISTRATION=true` after admin created |
| Storage | `STORAGE_PROVIDER=local` |
| Huly | card `[STACK] Postiz`; issue **AISB-57** (Done) |
| Org name (non-secret) | Fedoranzzi |
| Agent CLI env names | `POSTIZ_API_URL`, `POSTIZ_API_KEY` (often in ops profile / bashrc) |
| Preferred Hermes integration | Postiz MCP at `http://localhost:4007/api/mcp` with Bearer API key |
| Scoped multi-agent access | Middleware at `/srv/apps/postiz-middleware/` port **3456** |

**Core Postiz runtime env names (from deployment skill + prior research):**

- `JWT_SECRET`
- `DATABASE_URL`
- `REDIS_URL`
- `TEMPORAL_ADDRESS`
- `STORAGE_PROVIDER`
- `DISABLE_REGISTRATION`
- `MAIN_URL` / `NEXT_PUBLIC_MAIN_URL` (or `FRONTEND_URL` / `NEXT_PUBLIC_BACKEND_URL` depending on compose generation)
- `BACKEND_INTERNAL_URL`
- `IS_GENERAL` (some installs)
- Optional: `RESEND_API_KEY`, R2/S3 vars if not local storage

**API key recovery lesson (ops):** `Organization."apiKey"` in Postgres is often **hashed** — not usable as Bearer. Usable keys come from Postiz UI → Settings → Developers → Public API, or a previously saved `POSTIZ_API_KEY` in an ops profile env file.

### 2. Postiz provider env vars + callback patterns

Official Postiz docs (2026) confirm Meta FB+IG share one app; TikTok needs public HTTPS + verified media domain.

#### 2.1 Meta / Facebook / Instagram

| Concern | Postiz pattern |
|---|---|
| Env (FB + IG via Facebook Login) | `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET` |
| Env (IG standalone only) | `INSTAGRAM_APP_ID`, `INSTAGRAM_APP_SECRET` |
| FB callback | `{FRONTEND_URL}/integrations/social/facebook` |
| IG (FB business path) callback | `{FRONTEND_URL}/integrations/social/instagram` |
| IG standalone callback | `{FRONTEND_URL}/integrations/social/instagram-standalone` |
| Local docker examples | `http://localhost:5000/integrations/social/...` |
| Local dev examples | `http://localhost:4200/integrations/social/...` |
| FB scopes (Postiz docs) | `pages_show_list`, `business_management`, `pages_manage_posts`, `pages_manage_engagement`, `pages_read_engagement`, `read_insights` |
| IG scopes (FB path, Postiz docs) | `instagram_basic`, `pages_show_list`, `pages_read_engagement`, `business_management`, `instagram_content_publish`, `instagram_manage_comments`, `instagram_manage_insights` |
| Doc note | Same Meta app for Facebook + Instagram |

**Tomato sibling research alignment:** Tomato uses `META_APP_ID` / `META_APP_SECRET` and callback `{TOMATO_PUBLIC_BASE_URL}/api/oauth/meta/callback` — **different path family** than Postiz `/integrations/social/*`.

#### 2.2 TikTok

| Concern | Postiz pattern |
|---|---|
| Env | `TIKTOK_CLIENT_ID`, `TIKTOK_CLIENT_SECRET` (Client Key = ID; 16-char / 32-char formats cited in internal OAuth guide) |
| Callback | `{FRONTEND_URL}/integrations/social/tiktok` |
| HTTPS | **Required** for redirect URI (no plain http) |
| Products | Login Kit + Content Posting API; enable Direct Post |
| Scopes (Postiz docs) | `user.info.basic`, `video.create`, `video.publish`, `video.upload`, `user.info.profile` |
| Media | `pull_from_url` — media must be **public HTTPS**; localhost/`/uploads` private routes fail; verify domain in TikTok portal |
| Unaudited apps | Direct Post forced **SELF_ONLY**; ≤5 users / 24h; private accounts |

**Tomato naming:** `TIKTOK_CLIENT_KEY` / `TIKTOK_CLIENT_SECRET` (same portal values as Postiz’s `TIKTOK_CLIENT_ID` / `TIKTOK_CLIENT_SECRET`).

#### 2.3 X / Twitter (secondary for Tomato MVP)

Internal research + Postiz patterns:

- Env: `X_API_KEY` + `X_API_SECRET` (alt names sometimes `X_CLIENT` / `X_SECRET`)
- Callback pattern: `{FRONTEND_URL}/integrations/social/x` (internal OAuth guide)
- Needs OAuth 1.0a **and** OAuth 2.0 for media + v2 posting
- Paid API tiers are a cost flag for posting

#### 2.4 Other platforms (Postiz-only inventory; Tomato not MVP)

| Platform | Typical env names (internal docs) |
|---|---|
| LinkedIn | `LINKEDIN_CLIENT_ID`, `LINKEDIN_CLIENT_SECRET` |
| YouTube | `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET` |
| Reddit | `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET` |
| Pinterest | `PINTEREST_CLIENT_ID`, `PINTEREST_CLIENT_SECRET` |
| Threads | `THREADS_APP_ID`, `THREADS_APP_SECRET` |
| Slack | `SLACK_ID`, `SLACK_SECRET`, `SLACK_SIGNING_SECRET` |
| Discord | `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_BOT_TOKEN_ID` |
| Twitch | `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET` |
| GitHub | `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` |
| Mastodon | `MASTODON_URL`, `MASTODON_CLIENT_ID`, `MASTODON_CLIENT_SECRET` |

Generic SSO (if used): `POSTIZ_GENERIC_OAUTH`, `POSTIZ_OAUTH_*` family.

### 3. Postiz failures / fixes worth stealing as *ops* knowledge

From `postiz-deployment` skill + `/srv/research-output/postiz-oauth-auto-reconnect-research.md` + sessions:

| Failure | Fix / lesson for Tomato |
|---|---|
| OAuth `client_id=undefined` | Provider app env vars not loaded; restart after `.env` change |
| Callback mismatch | Exact match between provider console and public base + path; no trailing-slash drift |
| HTTP callbacks rejected (TikTok especially) | Permanent HTTPS public origin before registering |
| Instagram connect fails | Need Business/Creator IG linked to Page; tester roles; right scopes |
| TikTok media 403 / ownership | Public HTTPS media + verified domain/prefix; or FILE_UPLOAD |
| Token shows disconnected / null (X, IG, YT historical bugs) | Proactive refresh + health cron; don’t assume silent forever-tokens |
| Temporal workers die | Background refresh/post jobs stop; health-check workers |
| App Development mode | Posts only visible to app roles until Live / Advanced Access |
| Cloudflare R2 default storage | Local deploys must force `STORAGE_PROVIDER=local` |

**Tomato design takeaway:** implement **its own** refresh + encrypted store + health endpoints; do not depend on Postiz Temporal for Tomato-owned channels.

### 4. Echo profile / posting workflow

**What Echo is in this stack (reconstructed from skills + sessions):**

- Hermes/agent profile that posts via **Postiz**, not via direct Meta/TikTok SDKs as the primary path.
- Multi-agent social architecture:
  ```
  Agent (Echo / FURA / …)
      → Bearer scoped token (e.g. pk_echo_full, pk_fura_twitter)
      → postiz-middleware :3456
      → Postiz Public API :4007 (master key POSTIZ_MASTER_KEY)
      → platform OAuth tokens held inside Postiz
  ```
- Middleware token **names** (config file pattern `config/tokens.yaml` under postiz-middleware data/app dirs):
  - `pk_fura_twitter` → Twitter/X only
  - `pk_fura_instagram` → Instagram only
  - `pk_fura_linkedin` → LinkedIn only
  - `pk_echo_full` → all platforms
  - `pk_audit_readonly` → GET-only
- Alternative for Hermes: Postiz **MCP** (`integrationList`, `schedulePostTool`, …) with org API key — preferred when scoping not required.
- Adjacent skills referenced in ops history: `postiz-scoped-access`, `xitter` / `xurl` (direct X), FURA agent context (separate brand bot).

**What Tomato should copy from Echo:**

- Human/operator gate before publish (Tomato’s core product)
- Never hand agents the master provider secrets
- Channel/platform allow-lists conceptually (Tomato: per-channel encrypted tokens + approval queue)

**What Tomato should NOT copy:**

- Org-level Postiz API as the system of record for OAuth tokens
- Middleware `pk_*` token model as Tomato’s auth
- Temporal-based scheduler as Tomato’s publish engine
- “Connect channel in Postiz UI” as the only OAuth UX (Tomato owns `/api/oauth/*`)
- Multi-platform kitchen-sink (28 providers) — Tomato is Meta-first + TikTok spike

**Echo profile filesystem note:** this worker could not open a dedicated Echo profile tree under `/home/ice/.hermes/profiles` from the container view used here. Echo evidence is skill + session + middleware design, not a live Echo `.env` dump. **Gap:** if Echo keeps local secrets outside Postiz, they were not visible in this run.

### 5. Integrate with Postiz vs standalone

| Option | Verdict | Why |
|---|---|---|
| **A. Tomato standalone OAuth+publish** | **Recommended MVP** | Matches TOM-1 approval-first design; sibling env/Meta/TikTok research already specifies Tomato routes + encryption; avoids Postiz Temporal/token-refresh bugs as Tomato blockers |
| **B. Reuse Postiz only as optional sink** | Optional later | Tomato could “export approved post → Postiz schedule” via `POSTIZ_API_URL` + key; dual systems, dual token health |
| **C. Tomato is thin UI over Postiz** | **Reject for MVP** | Collapses product into scheduler; loses encrypted SQLite ownership, fake-provider loop, and operator model |

**Credential reuse policy:**

- **Yes reuse:** Meta App ID/Secret, TikTok client key/secret (same developer apps), Cloudflare zone/tunnel ops muscle, provider scope checklists, media public-HTTPS discipline.
- **No reuse as-is:** Postiz callback paths, Postiz `MAIN_URL` as Tomato public base, Postiz API keys as Tomato session auth, middleware tokens as Tomato operator auth.

### 6. Frank credential checklist (paste-ready, names only)

#### 6.1 From Meta developer portal (likely already used by Postiz)

1. App ID → `META_APP_ID`
2. App Secret → `META_APP_SECRET`
3. Add redirect: `https://tomato.<zone>/api/oauth/meta/callback` (exact)
4. App Domains: `tomato.<zone>` hostname only
5. Confirm Page + IG professional linked; app roles include Frank
6. Pin Graph version separately in Tomato (`META_GRAPH_VERSION`)

#### 6.2 From TikTok developer portal

1. Client Key → `TIKTOK_CLIENT_KEY` (Postiz called this `TIKTOK_CLIENT_ID`)
2. Client Secret → `TIKTOK_CLIENT_SECRET`
3. Add redirect: `https://tomato.<zone>/api/oauth/tiktok/callback`
4. Products: Login Kit + Content Posting; Direct Post toggle
5. Scopes: at least `user.info.basic,video.publish,video.upload`
6. Verify media domain/prefix if using PULL_FROM_URL; else prefer Tomato `TIKTOK_MEDIA_TRANSFER=file_upload`

#### 6.3 Tomato-only secrets (generate; not from Postiz)

- `TOMATO_TOKEN_ENCRYPTION_KEY` — `openssl rand -base64 32`
- `TOMATO_SESSION_SECRET` — `openssl rand -base64 48`
- `TOMATO_OPERATOR_PASSCODE_HASH`
- `NOUS_API_KEY` if draft brain enabled

#### 6.4 Postiz-only (do not paste into Tomato unless building adapter)

- `POSTIZ_API_KEY` / UI Public API key
- `POSTIZ_MASTER_KEY` (middleware)
- `pk_echo_full` and other middleware tokens
- `JWT_SECRET`, `DATABASE_URL`, etc.

#### 6.5 Where Frank can look without this report reprinting secrets

| Location | What |
|---|---|
| `/srv/apps/postiz/.env` | Provider app env names + values |
| `/srv/app-data/postiz/config/` | Mounted config if used |
| Postiz UI → Settings → Public API | Org API key |
| `/srv/apps/postiz-middleware/` + `config/tokens.yaml` | Scoped agent tokens |
| `~/.hermes/profiles/ops/.env` (and similar) | `POSTIZ_API_URL`, `POSTIZ_API_KEY` |
| Meta / TikTok developer consoles | Canonical app credentials |
| Cloudflare dashboard | Named tunnel hostname for tomato |

### 7. Public URL / tunnel patterns proven here

From sibling env contract + host ops notes:

1. **Preferred:** Cloudflare **named** tunnel  
   `https://tomato.<zone>` → `http://127.0.0.1:3737` (or Tomato bind)  
   Set both `TOMATO_APP_URL` and `TOMATO_PUBLIC_BASE_URL` to that origin; `TOMATO_TRUST_PROXY=true`.
2. **Avoid for OAuth registration:** `cloudflared tunnel --url …` quick tunnels (`*.trycloudflare.com`) — hostname churn breaks registered redirects.
3. **Postiz parallel:** Postiz uses stable host port 4007 locally; production OAuth historically needed public HTTPS (Akash ingress hostname appears in older research). Same rule: public HTTPS origin before TikTok/Meta Live flows.
4. **Media:** Tomato `/media/:assetId` on same public host for Meta image_url; TikTok prefer FILE_UPLOAD until domain verified.

### 8. Kanban / Huly / session provenance

| Source | Relevance |
|---|---|
| This task `t_2a7ac118` | Credential-pattern mining brief |
| Sibling `t_2d125f8e` | Tomato env / tunnel / encryption contract |
| Sibling `t_8c8d1279` | Meta scopes + publish checklist |
| Sibling `t_b8a7d332` | TikTok spike / audit checklist |
| Huly AISB-57 / Postiz stack card | Postiz deploy done |
| Huly TOM-1 | Tomato product issue |
| Skills `postiz`, `postiz-deployment` | Live ops truth for paths, middleware, MCP |
| `/srv/research-output/postiz-*`, `social-media-posting-stack-2026-05-19.md` | Prior GIU research |
| `/srv/scratch/know/postiz-social-media-oauth-integration-2025-01-14.md` | Full OAuth env/callback catalog |
| Sessions (e.g. middleware scoped keys Apr 2026; OAuth reconnect May 2026; social stack May 2026) | Architecture decisions |

**Kanban board mining limit:** full-text search across all historical kanban cards was partial (DB path not fully open in container). Primary internal truth came from skills + research-output + sibling tomato docs + session search.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C01 | Postiz is deployed on this host under `/srv/apps/postiz` with data under `/srv/app-data/postiz`, UI on :4007 | E01, E02 | high | supported |
| C02 | Postiz Meta env names are `FACEBOOK_APP_ID` / `FACEBOOK_APP_SECRET`; IG can share that app | E03, E04 | high | supported |
| C03 | Postiz TikTok env names are `TIKTOK_CLIENT_ID` / `TIKTOK_CLIENT_SECRET`; HTTPS callbacks required | E03, E05 | high | supported |
| C04 | Postiz OAuth callback path family is `/integrations/social/{platform}` | E03, E04, E05 | high | supported |
| C05 | Tomato callback path family is `/api/oauth/{meta\|tiktok}/callback` under `TOMATO_PUBLIC_BASE_URL` | E06 | high | supported |
| C06 | Tomato Meta env names are `META_APP_ID` / `META_APP_SECRET` (map from Postiz FACEBOOK_*) | E06, E07 | high | supported |
| C07 | Tomato TikTok env names are `TIKTOK_CLIENT_KEY` / `TIKTOK_CLIENT_SECRET` (map from Postiz TIKTOK_CLIENT_ID) | E06, E08 | high | supported |
| C08 | Multi-agent Postiz access uses middleware :3456 with tokens like `pk_echo_full` | E01, E09 | high | supported |
| C09 | Echo posts through Postiz (middleware/MCP), not as Tomato’s token store | E01, E09, E10 | medium | supported |
| C10 | Tomato MVP should stay standalone for OAuth/publish; optional Postiz export later | E06–E08, product brief | high | supported |
| C11 | Named Cloudflare Tunnel is the preferred public HTTPS pattern on this server | E06 | high | supported |
| C12 | Live Postiz `.env` values were not re-read this run (`/srv/apps` not mounted in worker) | E11 | high | supported |
| C13 | Organization.apiKey in DB may be hashed; UI-generated API key is the usable Bearer | E01 | high | supported |

## evidence_index

| evidence_id | source_label | source_url_or_path | provenance | note |
|---|---|---|---|---|
| E01 | postiz-deployment skill | `…/skills/devops/postiz-deployment/SKILL.md` | skill_view | Deploy paths, middleware, MCP, API key caveats |
| E02 | postiz agent skill | `…/skills/devops/postiz-agent/SKILL.md` | skill_view | CLI env names, ports, org labels |
| E03 | Postiz OAuth know doc | `/srv/scratch/know/postiz-social-media-oauth-integration-2025-01-14.md` | file | Env + callback catalog |
| E04 | Postiz Facebook/IG docs | https://docs.postiz.com/providers/facebook , `/instagram` | web_extract | Official env + redirects |
| E05 | Postiz TikTok docs | https://docs.postiz.com/providers/tiktok | web_extract | HTTPS, media pull, scopes |
| E06 | Tomato env contract | `/home/ice/fed/docs/tomato/research-env-contract-t_2d125f8e.md` | sibling task | Tomato env + tunnel |
| E07 | Tomato Meta research | `/home/ice/fed/docs/tomato/research-meta-t_8c8d1279.md` | sibling task | Scopes + Graph path |
| E08 | Tomato TikTok research | `/home/ice/fed/docs/tomato/research-tiktok-t_b8a7d332.md` | sibling task | Client key naming, audit |
| E09 | Session middleware design | `@session:geraldov21/20260415_062108_20ca08` | session_search | Scoped proxy + POSTIZ_MASTER_KEY |
| E10 | Social stack research | `/srv/research-output/social-media-posting-stack-2026-05-19.md` | file | Postiz as agent hub |
| E11 | Worker mount probe | terminal ls `/srv/apps` | this run | Apps/data not visible in container |
| E12 | OAuth reconnect research | `/srv/research-output/postiz-oauth-auto-reconnect-research.md` | file | Token refresh failures |

## gaps

1. **Live Postiz `.env` not opened** this run — cannot confirm which providers are currently connected or whether FACEBOOK_/TIKTOK_ vars are populated. Frank or host-side Su can `grep -E '^[A-Z_]+=' /srv/apps/postiz/.env | cut -d= -f1`.
2. **Echo profile directory** not found from this container — no Echo-local secret inventory beyond Postiz middleware naming.
3. **Kanban historical cards** not exhaustively SQLite-scanned — may miss older one-off OAuth tasks.
4. **Exact Cloudflare zone hostname** for tomato not chosen in this report (placeholder `tomato.<zone>`).
5. **Whether Frank’s Meta app is already Live** and which scopes were granted — dashboard confirmation only.
6. **Password discrepancy** between postiz vs postiz-deployment skills (admin login strings differ) — irrelevant to Tomato env map; do not treat skill passwords as source of truth without DB verify.

## confidence

- **level:** medium-high  
- **rationale:** Strong agreement across official Postiz docs, local skills, and sibling Tomato contracts on env **names** and callback **shapes**. Medium only because live `.env` and Echo profile files were unreachable from this worker; mapping Frank’s actual filled keys is inferred from “if Postiz providers were configured.”

## recommended_next_actions

1. **Su:** Implement Tomato env loader per `research-env-contract-t_2d125f8e.md`; document Frank paste table from §6 in README.
2. **Su:** OAuth routes under `/api/oauth/meta/*` and `/api/oauth/tiktok/*` — freeze paths before Frank registers them.
3. **Ops/Frank:** Create named CF tunnel `tomato.<zone>`; set public base URLs.
4. **Frank (wake-up):** From Meta/TikTok portals (or Postiz `.env` **names**), paste into Tomato `.env`; **add** Tomato callbacks without removing Postiz callbacks if Postiz stays live.
5. **Optional later:** Adapter skill `tomato → Postiz schedule` using `POSTIZ_API_URL` + key — only after standalone Meta path works.
6. **Do not** put Postiz middleware tokens or master API keys into Tomato operator auth.

---

## appendix A — side-by-side env map (MVP)

| Purpose | Postiz | Tomato |
|---|---|---|
| Public origin | `MAIN_URL` / `FRONTEND_URL` / `NEXT_PUBLIC_*` | `TOMATO_PUBLIC_BASE_URL`, `TOMATO_APP_URL` |
| Meta app id | `FACEBOOK_APP_ID` | `META_APP_ID` |
| Meta app secret | `FACEBOOK_APP_SECRET` | `META_APP_SECRET` |
| IG standalone id/secret | `INSTAGRAM_APP_ID` / `INSTAGRAM_APP_SECRET` | (prefer single Meta app; optional later) |
| TikTok client key | `TIKTOK_CLIENT_ID` | `TIKTOK_CLIENT_KEY` |
| TikTok client secret | `TIKTOK_CLIENT_SECRET` | `TIKTOK_CLIENT_SECRET` |
| Meta callback | `/integrations/social/facebook` (+ ig variants) | `/api/oauth/meta/callback` |
| TikTok callback | `/integrations/social/tiktok` | `/api/oauth/tiktok/callback` |
| User tokens at rest | Postiz DB + Temporal refresh | SQLite AES-GCM via `TOMATO_TOKEN_ENCRYPTION_KEY` |
| Agent API access | `POSTIZ_API_KEY` / `pk_*` middleware | Tomato session + approval queue (not Postiz) |

## appendix B — architecture sketch

```
Frank portals (Meta / TikTok)
        │ app id/secret (reuse OK)
        ▼
   Tomato .env  ──► OAuth start/callback on https://tomato.<zone>
        │
        ▼
 encrypted tokens in tomato.sqlite
        │
        ▼
 approval queue ──► Graph / TikTok publish
        │
        └── (optional later) Postiz Public API schedule sink

Echo / FURA agents (existing)
        │ pk_* scoped tokens
        ▼
 postiz-middleware:3456 ──► Postiz:4007 ──► same Meta/TikTok apps (different callbacks)
```

---

*End of report. No secret values included.*
