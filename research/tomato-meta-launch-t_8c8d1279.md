# Tomato Meta launch research — exact API/key/scopes checklist

**Task:** `t_8c8d1279`  
**Product:** Tomato (approval-first social command center)  
**Scope:** Facebook Page + Instagram professional account publishing, then metrics  
**Stack assumption:** local-first Next.js app; Frank pastes credentials later  
**Research date:** 2026-09-02  
**Graph API versions cited:** v25.0 / v26.0 (docs currently show both; pin one version in code)

---

## objective

Produce an exact, implementation-ready Meta/Facebook/Instagram publishing + metrics checklist so Su can wire Tomato end-to-end without Frank, leaving only key paste + dashboard confirmations for Frank at wake-up.

## summary

Tomato should implement **Facebook Login for Business** (not pure Instagram Login) as the primary path because it covers **Page publishing + Page-linked IG professional accounts + insights** from one OAuth flow and one Page access token family. Required Page publish scopes are `pages_manage_posts`, `pages_read_engagement`, `pages_show_list` (plus `pages_manage_metadata` / `pages_manage_read_engagement` per Pages get-started). IG publish scopes (FB Login path) are `instagram_basic`, `instagram_content_publish`, `pages_read_engagement`. Insights need `read_insights` (Page) and `instagram_manage_insights` (IG). Media for IG must be on a **public HTTPS URL** Meta can cURL — **localhost is not acceptable for IG image_url**. OAuth redirect URIs can use localhost during Development mode (confirm in dashboard). Tokens: short-lived user → long-lived user (~60d) → long-lived Page token (no fixed expiry when derived from long-lived user). Minimal FB photo = `POST /{page-id}/photos`; minimal IG photo = container → status poll → `media_publish`. Development/Standard Access is enough while Frank is the only user with an app role; App Review + Business Verification only if non-role users need the app.

---

## 0. Recommended architecture for Tomato (decision)

| Choice | Recommendation | Why |
|---|---|---|
| Login product | **Facebook Login for Business** | Required for Page-linked IG Graph API path; one flow for FB + IG |
| Token used for publish | **Page access token** | Page posts + IG content publish (FB Login path) use Page token |
| Host for Graph calls | `graph.facebook.com` | FB Login path (not `graph.instagram.com`) |
| Dev mode strategy | Stay in **Development / Standard Access** while only Frank (app role) uses it | No App Review needed for self-owned assets |
| Media hosting | Tomato must expose **public HTTPS** media URLs (tunnel, object storage, or prod CDN) | IG Content Publishing cURLs `image_url` from Meta servers |
| OAuth style | Authorization code flow, server-side exchange | App secret never in browser; long-lived exchange server-side |

**UNVERIFIED (Frank must confirm in Meta dashboard):** exact App type / use-case labels shown in 2026 App Dashboard UI (use-case builder vs classic app types). Functionally request the permissions listed below regardless of UI grouping.

---

## 1. Required Meta developer app setup steps

### A. One-time Meta account + assets (Frank)

1. Facebook personal account with developer access at [developers.facebook.com](https://developers.facebook.com/).
2. Create / claim a **Facebook Page** Frank can perform `CREATE_CONTENT` (and ideally `MANAGE`, `MODERATE`, `ANALYZE`) on.
3. Convert / ensure **Instagram professional account** (Business or Creator).
4. **Link IG professional account to that Facebook Page** (required for Instagram API with Facebook Login).
5. Optional but recommended: Meta Business / Business Manager portfolio that owns the app, Page, and IG asset.
6. Complete **Page Publishing Authorization (PPA)** preemptively if the Page ever requires it — IG publish blocks until PPA is done. Docs: Content Publishing + [PPA one-pager](https://www.facebook.com/business/m/one-sheeters/page-publishing-authorization).

Sources:  
- https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-facebook-login/get-started.md  
- https://developers.facebook.com/documentation/instagram-platform/content-publishing.md  
- https://developers.facebook.com/documentation/instagram-platform/overview.md  

### B. Create the Meta app (Frank or Su with Frank present)

1. Create app at App Dashboard → choose use case covering Pages + Instagram (or Other → Business-type if use-case UI does not fit).
2. Record **App ID** and **App Secret** (Settings → Basic).
3. Add products:
   - **Facebook Login** / **Facebook Login for Business**
   - Instagram Graph / Instagram product as offered by current dashboard
4. Settings → Basic:
   - App domains (when you have a real domain)
   - Privacy Policy URL / Terms (required before Live / Advanced Access; can be stubs in Dev)
   - Data deletion instructions / callback URL (required before publish for many use cases)
5. Facebook Login → Settings:
   - **Client OAuth login:** ON
   - **Web OAuth login:** ON
   - **Valid OAuth Redirect URIs:** exact Tomato callback URLs (see §3)
   - **Deauthorize callback URL** (optional but recommended)
6. Add Frank (and any testers) under **App Roles** (Administrator / Developer / Tester).
7. Keep app in **Development mode** (non-business) or **Standard Access** until external users are needed.

Sources:  
- https://developers.facebook.com/docs/development/build-and-test/  
- https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow/  
- https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-facebook-login/get-started.md  
- https://developers.facebook.com/docs/development/release/  

### C. Tomato code scaffolding (Su — no secrets needed yet)

1. Implement OAuth start + callback routes (see §3).
2. Implement server-only token exchange + long-lived + Page token derivation (see §8).
3. Implement encrypted token store (see §8).
4. Implement provider adapters: `facebook-page`, `instagram-professional`.
5. Implement public media URL issuer for publish jobs (see §4).
6. Define env var names (see §5) and a “paste keys” settings UI for Frank.

---

## 2. Exact products / permissions / scopes

### 2.1 Facebook Page post publishing

**Official Pages get-started permissions:**

| Permission | Purpose |
|---|---|
| `pages_show_list` | List Pages user can manage (`/me/accounts`) |
| `pages_manage_posts` | Create posts / photos on Page |
| `pages_manage_metadata` | Listed on Pages get-started |
| `pages_manage_read_engagement` | Listed on Pages get-started (naming as in docs) |

**Pages API posts guide also lists:**

| Permission | Purpose |
|---|---|
| `pages_manage_posts` | Publish |
| `pages_read_engagement` | Read engagement / related Page content ops |
| `pages_manage_engagement` | Manage engagement (comments etc.) |
| `pages_read_user_engagement` | User engagement reads |
| `publish_video` | **Only if** publishing video |

**Page photo create requirements (reference):**

- Page access token from user with `CREATE_CONTENT` task
- `pages_read_engagement`
- `pages_manage_posts`
- `pages_show_list`

**Tomato v1 minimum scope set (Page text + photo):**

```
pages_show_list
pages_manage_posts
pages_read_engagement
pages_manage_metadata
```

Add later if needed: `pages_manage_engagement`, `pages_read_user_engagement`, `publish_video`.

**Note on naming drift:** Docs alternate `pages_manage_read_engagement` vs `pages_read_engagement`. Treat **`pages_read_engagement`** as the canonical Graph permission string used by Page Photos reference; if dashboard only shows `pages_manage_read_engagement`, **UNVERIFIED — Frank must confirm exact checkbox label in App Dashboard / Graph Explorer** and grant whichever string the token debugger shows.

Sources:  
- https://developers.facebook.com/docs/pages-api/getting-started/  
- https://developers.facebook.com/documentation/pages-api/posts.md  
- https://developers.facebook.com/docs/graph-api/reference/page/photos/  
- https://developers.facebook.com/docs/permissions/  

### 2.2 Instagram professional account media publishing

**Path A — Instagram API with Facebook Login (recommended for Tomato):**

| Permission | Required |
|---|---|
| `instagram_basic` | Yes |
| `instagram_content_publish` | Yes |
| `pages_read_engagement` | Yes |
| `ads_management` + `ads_read` | **Only if** Page role was granted via Business Manager |

**Access token type:** Facebook **Page** access token  
**Host:** `graph.facebook.com` (resumable video also `rupload.facebook.com`)

**Path B — Instagram API with Instagram Login (optional later):**

| Permission | Required |
|---|---|
| `instagram_business_basic` | Yes |
| `instagram_business_content_publish` | Yes |

**Token:** Instagram User access token  
**Host:** `graph.instagram.com`  
**Does not replace Page publishing.**

Sources:  
- https://developers.facebook.com/documentation/instagram-platform/content-publishing.md  
- https://developers.facebook.com/documentation/instagram-platform/overview.md  

### 2.3 Page / IG metrics ingestion

**Facebook Page insights:**

| Permission / requirement | Notes |
|---|---|
| `pages_read_engagement` | Required |
| `read_insights` | Required |
| Page access token | Requester must have **ANALYZE** task on Page |

**Instagram insights (FB Login path):**

| Permission | Notes |
|---|---|
| `instagram_basic` | Yes |
| `instagram_manage_insights` | Yes |
| `pages_read_engagement` | Yes |
| `ads_management` / `ads_read` | If Page role via Business Manager |

**IG insights (Instagram Login path):** `instagram_business_basic` + `instagram_business_manage_insights`

**Access level:** Standard if only assets you own/manage + app roles; Advanced if serving others’ accounts.

Sources:  
- https://developers.facebook.com/docs/platforminsights/page/  
- https://developers.facebook.com/docs/instagram-platform/insights/  
- https://developers.facebook.com/docs/graph-api/overview/access-levels/  

### 2.4 Consolidated OAuth `scope` string for Tomato v1

```
public_profile,pages_show_list,pages_manage_posts,pages_read_engagement,pages_manage_metadata,instagram_basic,instagram_content_publish,instagram_manage_insights,read_insights
```

Optional later: `pages_manage_engagement`, `publish_video`, `ads_management`, `ads_read`.

---

## 3. Exact redirect / callback URL patterns Tomato should implement

### 3.1 OAuth endpoints Tomato must expose

| Route | Method | Role |
|---|---|---|
| `/api/meta/oauth/start` | GET | Builds Facebook dialog URL, sets CSRF `state`, redirects |
| `/api/meta/oauth/callback` | GET | Receives `?code=&state=`, exchanges code, stores tokens |
| `/api/meta/oauth/deauthorize` | POST | Deauthorize callback (signed_request) |
| `/api/meta/data-deletion` | POST | Data deletion request callback |
| `/api/meta/webhooks` | GET+POST | Optional early; required later for realtime |

### 3.2 Valid OAuth Redirect URIs to register in App Dashboard

Register **exact** strings (scheme + host + port + path; no partial matches):

**Local development:**

```
http://localhost:3000/api/meta/oauth/callback
http://127.0.0.1:3000/api/meta/oauth/callback
```

**If Tomato uses another port / host alias, register each variant.**

**Staging / production:**

```
https://{TOMATO_PUBLIC_HOST}/api/meta/oauth/callback
```

### 3.3 Dialog URL pattern

```
https://www.facebook.com/v26.0/dialog/oauth
  ?client_id={META_APP_ID}
  &redirect_uri={URL_ENCODED_EXACT_CALLBACK}
  &state={CSRF_STATE}
  &scope={COMMA_SEPARATED_SCOPES}
  &response_type=code
```

### 3.4 Code exchange pattern (server-side only)

```
GET https://graph.facebook.com/v26.0/oauth/access_token
  ?client_id={META_APP_ID}
  &redirect_uri={SAME_EXACT_CALLBACK}
  &client_secret={META_APP_SECRET}
  &code={CODE}
```

`redirect_uri` in exchange **must match** the one used in the dialog and the dashboard allowlist.

Sources:  
- https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow/  
- https://developers.facebook.com/documentation/facebook-login/guides/access-tokens/get-long-lived.md  

### 3.5 Localhost redirect note

Official manual flow requires registering Valid OAuth Redirect URIs. Community threads show intermittent localhost allowlist friction. **Supported practice for local Next.js:** register `http://localhost:{port}/...` while app is in Development mode. If dashboard rejects localhost, use a stable HTTPS tunnel URL (ngrok/cloudflared) registered as the redirect URI instead.

**UNVERIFIED:** whether Meta currently allows plain `http://localhost` in every app type’s OAuth settings — Frank must try save in Facebook Login → Settings and screenshot result.

---

## 4. Public media URL requirements (and localhost)

### Instagram Content Publishing

> “We cURL media used in publishing attempts, so the media must be hosted on a publicly accessible server at the time of the attempt.”

| Scenario | Acceptable? |
|---|---|
| `https://cdn.example.com/media/abc.jpg` | Yes |
| `https://tomato.example.com/api/media/signed/...` (publicly reachable) | Yes |
| `http://localhost:3000/...` | **No** — Meta’s servers cannot fetch it |
| Private VPC URL | **No** |
| Expired signed URL before Meta fetch completes | **No** (common failure) |

**IG image constraints (official):**

- **JPEG only** for IG image containers (MPO/JPS not supported)
- Filters / shopping tags not supported on this API path
- Rate limit: **100 API-published posts / 24h moving window** per IG account (`media_publish`); check `GET /{ig-user-id}/content_publishing_limit`

### Facebook Page photos

- Accepts upload via `url` **or** multipart `source`
- File types: `.jpeg, .bmp, .png, .gif, .tiff`
- Max **10MB** (PNG recommended ≤1MB)
- Multipart upload can be done from Tomato server without a public URL
- URL-based publish still needs a fetchable URL

**Tomato implementation rule:**

1. Always store originals in Tomato object storage / local disk.
2. For IG: mint a **short-lived public HTTPS URL** (or put object in public bucket) before container create; keep valid ≥ few minutes.
3. For FB Page: prefer **server-side multipart upload** to avoid public URL requirement; fall back to `url` if already public.
4. Local dev for IG: use **tunnel or public bucket**, never localhost media URLs.

Sources:  
- https://developers.facebook.com/documentation/instagram-platform/content-publishing.md  
- https://developers.facebook.com/docs/graph-api/reference/page/photos/  

---

## 5. Required env vars (names only — no secrets)

### App identity (Frank pastes)

```
META_APP_ID=
META_APP_SECRET=
META_GRAPH_API_VERSION=v26.0
META_OAUTH_REDIRECT_URI=http://localhost:3000/api/meta/oauth/callback
META_OAUTH_SCOPES=public_profile,pages_show_list,pages_manage_posts,pages_read_engagement,pages_manage_metadata,instagram_basic,instagram_content_publish,instagram_manage_insights,read_insights
```

### Optional webhook / compliance

```
META_APP_CLIENT_TOKEN=
META_WEBHOOK_VERIFY_TOKEN=
META_DEAUTHORIZE_CALLBACK_URL=
META_DATA_DELETION_CALLBACK_URL=
```

### Runtime / storage (Tomato-generated, not from Meta)

```
META_TOKEN_ENCRYPTION_KEY=
# or reuse app-level:
TOMATO_SECRETS_MASTER_KEY=
TOMATO_PUBLIC_MEDIA_BASE_URL=
TOMATO_PUBLIC_APP_URL=
```

### Per-connection fields (DB, not env — after OAuth)

Store encrypted, not in `.env`:

```
meta_user_id
meta_user_access_token_encrypted
meta_user_token_expires_at
meta_page_id
meta_page_access_token_encrypted
meta_page_name
meta_ig_user_id
meta_ig_username
meta_granted_scopes
meta_token_type                # user | page
meta_connected_at
meta_last_refreshed_at
```

### Explicitly do NOT put in env long-term

- Short-lived user tokens  
- Long-lived user tokens  
- Page tokens  
- IG user tokens  

These belong in encrypted DB / secret store after OAuth.

---

## 6. Test account / test Page / test IG checklist

### Preferred path for Tomato (real assets, Development mode)

Because Tomato is local-first for Frank’s own brands, **real Page + real IG professional + Frank as app Admin** is simpler than Meta test users.

- [ ] Frank is **Administrator** on Meta app  
- [ ] Frank can perform `CREATE_CONTENT` (+ `ANALYZE`) on target Page  
- [ ] IG account is **Professional** (Business or Creator)  
- [ ] IG is **linked to the Facebook Page**  
- [ ] Verify link: `GET /{page-id}?fields=instagram_business_account` returns IG id  
- [ ] PPA completed if prompted  
- [ ] Graph API Explorer smoke test with Page token before wiring Next.js  

### Meta Test Users / Test Pages (optional isolation)

- [ ] App Dashboard → Roles → Test Users: create test user  
- [ ] Create Test Page as that test user  
- [ ] Link a test IG professional account if available for the test identity  
- [ ] Remember: test users **cannot** interact with real users; content only visible to test users + app roles  

Sources:  
- https://developers.facebook.com/docs/development/build-and-test/  
- https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-facebook-login/get-started.md  

### Graph Explorer preflight (Frank, 10 min)

1. Select app + User token  
2. Grant scopes from §2.4  
3. `GET /me/accounts` → copy Page id + Page token  
4. `GET /{page-id}?fields=instagram_business_account`  
5. `POST /{page-id}/photos` with a public jpeg `url` + caption  
6. `POST /{ig-id}/media` then poll status then `media_publish`  

---

## 7. App Review / Advanced Access vs Development mode

| Mode | Who can grant permissions | What works |
|---|---|---|
| **Development mode** (classic non-business apps) | Only users with **app roles** | Any permission for role users; features active for role users |
| **Live mode** | Anyone, but only **App Review–approved** permissions/features | Unapproved permissions inactive for non-role users |
| **Standard Access** | Only app-role users (or managed assets depending on product) | Enough for Frank-only Tomato |
| **Advanced Access** | Any customer user | Requires **App Review** + usually **Business Verification** |

**Tomato launch implication:**

1. **Phase 0 (now):** Development + Standard Access + Frank as Admin → full publish/metrics on Frank’s Page/IG. **No App Review required.**
2. **Phase 1 (multi-tenant / other brands’ logins):** App Review for each permission in §2, Business Verification, Live/Advanced Access, privacy policy, data deletion callback, screencasts.

**Development mode visibility caveat (historical / operational):** posts made while limited may have reduced visibility to non-admins depending on product rules — verify with a real publish on Frank’s Page. Mark any “only admins see it” behavior as dashboard/policy confirmation.

Sources:  
- https://developers.facebook.com/docs/graph-api/overview/access-levels/  
- https://developers.facebook.com/docs/development/build-and-test/  
- https://developers.facebook.com/docs/development/release/  
- https://developers.facebook.com/docs/permissions/  

---

## 8. Token lifecycle

### 8.1 Types

| Token | Lifetime | How obtained | Used for |
|---|---|---|---|
| Short-lived **User** token | ~1 hour (web login) | OAuth code exchange | Exchange only |
| Long-lived **User** token | ~60 days | Server exchange w/ app secret | Derive Page tokens; refresh window |
| **Page** token from long-lived user | **No fixed expiry** (invalidated on password change, permission revoke, etc.) | `GET /{user-id}/accounts` | Page publish + IG publish (FB Login path) |
| Instagram User token (IG Login path) | short → long (~60d) | Separate IG login product | Only if Path B adopted |

### 8.2 Exchange flows (exact)

**A. Short → long user token (server only):**

```
GET https://graph.facebook.com/{version}/oauth/access_token
  ?grant_type=fb_exchange_token
  &client_id={app-id}
  &client_secret={app-secret}
  &fb_exchange_token={short-lived-user-token}
```

Response:

```json
{
  "access_token": "{long-lived-user-access-token}",
  "token_type": "bearer",
  "expires_in": 5183944
}
```

**B. Long-lived Page token:**

```
GET https://graph.facebook.com/{version}/{app-scoped-user-id}/accounts
  ?access_token={long-lived-user-access-token}
```

Each Page object includes `access_token`, `id`, `name`, `tasks`.

**C. Debug token:**

```
GET https://graph.facebook.com/debug_token
  ?input_token={token-to-inspect}
  &access_token={app-access-token-or-valid-token}
```

### 8.3 Tomato sequence after callback

1. Verify `state`  
2. Exchange `code` → short-lived user token  
3. Exchange → long-lived user token (server)  
4. `GET /me` → store `user_id`  
5. `GET /me/accounts` with long-lived user token → pages[]  
6. User picks Page (or auto if one)  
7. Persist **Page token** (primary publish credential)  
8. `GET /{page-id}?fields=instagram_business_account` → `ig_user_id`  
9. Store encrypted tokens + expiry + scopes  
10. Schedule re-auth reminder before user token ~60d expiry (Page token usually lasts until invalidation, but user token refresh path should exist)

### 8.4 Secure storage notes

- **Never** expose `META_APP_SECRET` to the browser or client bundle  
- Encrypt tokens at rest (AES-GCM with `META_TOKEN_ENCRYPTION_KEY`)  
- Encrypt backups; never log raw tokens  
- Store `expires_in` / `expires_at`; Facebook will **not** notify on invalidation  
- On API error 190 / subcodes 458/460/463/467 → mark connection `needs_reauth`  
- Prefer Page token for publish jobs; keep long-lived user token for re-listing pages / re-deriving page tokens  
- Do not reuse one long-lived user token across multiple independent browser clients without the client_code flow  

Sources:  
- https://developers.facebook.com/documentation/facebook-login/guides/access-tokens/get-long-lived.md  
- https://developers.facebook.com/docs/facebook-login/access-tokens/debugging-and-error-handling/  
- https://developers.facebook.com/docs/graph-api/guides/error-handling/  

---

## 9. Minimal publish sequences

Assume `GRAPH=https://graph.facebook.com/v26.0` and Page token as `PAGE_TOKEN`.

### 9.1 Facebook Page photo post (minimal)

**Option A — public URL:**

```bash
curl -X POST "$GRAPH/{page-id}/photos" \
  -F "url=https://public.example.com/photo.jpg" \
  -F "caption=Hello from Tomato" \
  -F "published=true" \
  -F "access_token=$PAGE_TOKEN"
```

**Success shape:**

```json
{
  "id": "{photo_id}",
  "post_id": "{page_post_id}"
}
```

**Option B — multipart from Tomato server (no public URL):**

```bash
curl -X POST "$GRAPH/{page-id}/photos" \
  -F "source=@/path/to/photo.jpg" \
  -F "caption=Hello from Tomato" \
  -F "access_token=$PAGE_TOKEN"
```

**Text-only feed post:**

```bash
curl -X POST "$GRAPH/{page-id}/feed" \
  -H "Content-Type: application/json" \
  -d "{\"message\":\"Hello\",\"access_token\":\"$PAGE_TOKEN\"}"
```

**Success:** `{ "id": "{page_post_id}" }`  
**Permalink:** `https://www.facebook.com/{page_post_id}`

**Page photo specs:** jpeg/bmp/png/gif/tiff; ≤10MB.

Sources:  
- https://developers.facebook.com/documentation/pages-api/posts.md  
- https://developers.facebook.com/docs/graph-api/reference/page/photos/  
- https://developers.facebook.com/docs/pages-api/getting-started/  

### 9.2 Instagram photo post (container → status → publish)

Prereq: `IG_USER_ID` from `/{page-id}?fields=instagram_business_account`, JPEG at **public HTTPS** `IMAGE_URL`, Page token with IG publish scopes.

**Step 1 — create container**

```bash
curl -X POST "$GRAPH/{ig-user-id}/media" \
  -H "Authorization: Bearer $PAGE_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"image_url\":\"$IMAGE_URL\",\"caption\":\"Hello from Tomato\"}"
```

**Success:** `{ "id": "{creation_id}" }`

**Step 2 — poll status until FINISHED**

```bash
curl -G "$GRAPH/{creation_id}" \
  -d "fields=status_code" \
  -d "access_token=$PAGE_TOKEN"
```

**status_code values:**

| Code | Meaning |
|---|---|
| `IN_PROGRESS` | Still processing |
| `FINISHED` | Ready to publish |
| `ERROR` | Failed |
| `EXPIRED` | Not published within 24h |
| `PUBLISHED` | Already published |

Docs recommend polling **~once per minute, ≤5 minutes** (esp. for video). For images, poll more frequently with backoff (e.g. 2s/5s/10s) but respect rate limits.

**Step 3 — publish**

```bash
curl -X POST "$GRAPH/{ig-user-id}/media_publish" \
  -H "Authorization: Bearer $PAGE_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"creation_id\":\"{creation_id}\"}"
```

**Success:** `{ "id": "{ig_media_id}" }`

**Optional — rate limit check**

```bash
curl -G "$GRAPH/{ig-user-id}/content_publishing_limit" \
  -d "access_token=$PAGE_TOKEN"
```

Sources:  
- https://developers.facebook.com/documentation/instagram-platform/content-publishing.md  
- https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-facebook-login/get-started.md  

### 9.3 Tomato provider state machine (implement this)

```
draft → approved → media_ready → (fb: publishing | ig: container_created → container_finished) → published → insights_pending → insights_synced
                                      ↘ failed(error_code, fbtrace_id, raw)
```

---

## 10. Minimal metrics sequence + first metrics to store

### 10.1 Resolve IDs (once per connection)

```bash
# Pages + page tokens
GET $GRAPH/me/accounts?access_token=$USER_LONG_TOKEN

# IG business account id
GET $GRAPH/{page-id}?fields=instagram_business_account&access_token=$PAGE_TOKEN
```

### 10.2 Page-level insights (daily rollup job)

```bash
GET $GRAPH/{page-id}/insights
  ?metric=page_impressions_unique,page_impressions_paid
  &period=day
  &access_token=$PAGE_TOKEN
```

Requires `read_insights` + `pages_read_engagement` + ANALYZE task.

**Note:** Page Insights historically require meaningful Page size for some metrics; empty datasets possible. Docs note metric deprecations through 2026 — pin metric names and handle `100` invalid metric errors.

### 10.3 Page post insights (per Tomato-published post)

```bash
GET $GRAPH/{page-post-id}/insights
  ?metric=post_reactions_like_total,post_reactions_love_total,post_reactions_wow_total
  &access_token=$PAGE_TOKEN
```

### 10.4 Instagram account insights

```bash
GET $GRAPH/{ig-user-id}/insights
  ?metric=impressions,reach,profile_views
  &period=day
  &access_token=$PAGE_TOKEN
```

### 10.5 Instagram media insights (per published IG media id)

```bash
GET $GRAPH/{ig-media-id}/insights
  ?metric=engagement,impressions,reach
  &access_token=$PAGE_TOKEN
```

(Lifetime metrics often return `period=lifetime`.)

### 10.6 Recommended first metrics Tomato should store

| Entity | Metric key | Period | Why |
|---|---|---|---|
| FB Page | `page_impressions_unique` | day | Core reach |
| FB Page post | `post_reactions_like_total` | lifetime/as returned | Simple engagement |
| FB Page post | `post_reactions_love_total` | lifetime/as returned | Engagement mix |
| IG account | `impressions` | day | Top-of-funnel |
| IG account | `reach` | day | Unique viewers |
| IG account | `profile_views` | day | Profile interest |
| IG media | `impressions` | lifetime | Per-post |
| IG media | `reach` | lifetime | Per-post |
| IG media | `engagement` | lifetime | likes+comments aggregate |

**Storage shape (suggested):**

```
provider, account_id, object_type, object_id, metric, period, end_time, value, fetched_at, raw_json
```

**Job cadence:** hourly for last 48h posts; daily for account rollups; skip if connection `needs_reauth`.

Sources:  
- https://developers.facebook.com/docs/platforminsights/page/  
- https://developers.facebook.com/docs/instagram-platform/insights/  
- https://developers.facebook.com/docs/graph-api/reference/insights/  

---

## 11. Common failure modes + exact error shapes

### 11.1 Canonical Graph error envelope

```json
{
  "error": {
    "message": "Message describing the error",
    "type": "OAuthException",
    "code": 190,
    "error_subcode": 460,
    "error_user_title": "A title",
    "error_user_msg": "A message",
    "fbtrace_id": "EJplcsCHuLu"
  }
}
```

Tomato should persist: `code`, `error_subcode`, `type`, `message`, `error_user_msg`, `fbtrace_id`.

### 11.2 Auth / token errors

| code | subcode | Meaning | Tomato action |
|---|---|---|---|
| 190 / OAuthException | — | Token expired/revoked/invalid | Re-auth |
| 190 | 458 | App not installed / not authorized | Re-auth |
| 190 | 460 | Password changed / session invalid | Re-auth |
| 190 | 463 | Expired | Re-auth |
| 190 | 467 | Invalid access token | Re-auth |
| 190 | 459 / 464 | User checkpointed / unconfirmed | Tell user to fix at facebook.com |
| — | 492 | Page token user lacks Page role | Reconnect with correct user |

### 11.3 Permission errors

| code | Meaning | Tomato action |
|---|---|---|
| 10 | Permission denied | Request missing scopes (`auth_type=rerequest`) |
| 200–299 | Permission-related | Same |
| 200 | Permissions error (photos) | Check Page tasks + scopes |
| 283 | Needs pages_read_engagement and/or related | Add scopes |

### 11.4 Publish / media errors

| code | Meaning | Tomato action |
|---|---|---|
| 324 | Missing or invalid image file | Validate image; FB types / IG JPEG |
| 506 | Duplicate post | Change content |
| 368 | Policy / abusive block | Back off; surface to user |
| 1609005 | Link scrape failed | Fix URL |
| IG status `ERROR` | Container processing failed | Surface; recreate container |
| IG status `EXPIRED` | >24h without publish | Recreate container |
| Public URL fetch fail | Meta cannot cURL media | Fix public HTTPS media |

### 11.5 Throttle / limits

| code | Meaning | Tomato action |
|---|---|---|
| 4 / 17 / 341 | Too many calls | Exponential backoff |
| 80001 | Too many calls to this Page | Backoff |
| IG 100 posts/24h | content publishing limit | Pre-check limit endpoint; queue |

### 11.6 Insights errors

| Signal | Meaning | Tomato action |
|---|---|---|
| Empty `data` | Often missing `read_insights` or unavailable metric | Check scopes; don’t store fake zeros blindly |
| 100 “valid insights metric” | Bad/deprecated metric name | Update metric list (2026 deprecations) |
| 3001 / subcode 1504028 | No metric specified | Always pass `metric=` |

Sources:  
- https://developers.facebook.com/docs/graph-api/guides/error-handling/  
- https://developers.facebook.com/docs/facebook-login/access-tokens/debugging-and-error-handling/  
- https://developers.facebook.com/docs/graph-api/reference/page/photos/  
- https://developers.facebook.com/docs/platforminsights/page/  
- https://developers.facebook.com/documentation/instagram-platform/content-publishing.md  

---

## Implementation checklist for Su (ordered)

### Code (no Frank)

- [ ] Env schema with names from §5  
- [ ] OAuth start/callback/deauthorize/data-deletion routes (§3)  
- [ ] Server-side code→token→long-lived→page token pipeline (§8)  
- [ ] Encrypted token repository  
- [ ] Page picker UI (from `/me/accounts`)  
- [ ] IG id resolution via `instagram_business_account`  
- [ ] FB photo publisher (`/{page-id}/photos`)  
- [ ] IG publisher state machine (container/status/publish)  
- [ ] Public media URL minting for IG  
- [ ] Metrics pull job for §10.6 metrics  
- [ ] Error mapper for §11 codes → `needs_reauth` | `retry` | `user_fix` | `fatal`  
- [ ] Fake-provider parity tests using recorded Graph fixtures  

### Frank at wake-up (paste + dashboard only)

- [ ] Create/open Meta app; paste `META_APP_ID`, `META_APP_SECRET`  
- [ ] Add Valid OAuth Redirect URIs for local + prod callbacks  
- [ ] Ensure Page + IG professional linked  
- [ ] Add self as app Admin  
- [ ] Run Tomato “Connect Meta” → approve scopes  
- [ ] Confirm test FB photo + IG photo publish  
- [ ] Confirm insights rows appear  

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | IG Content Publishing requires publicly reachable media URLs because Meta cURLs them | E1 | high | supported |
| C2 | FB Login path IG publish permissions include `instagram_basic`, `instagram_content_publish`, `pages_read_engagement` | E1 | high | supported |
| C3 | Page photo create requires Page token + `pages_manage_posts`, `pages_read_engagement`, `pages_show_list` + CREATE_CONTENT | E2 | high | supported |
| C4 | Long-lived user tokens last ~60 days; Page tokens from long-lived user tokens have no fixed expiry | E3 | high | supported |
| C5 | IG publish flow is create `/{ig-id}/media` → poll `status_code` → `/{ig-id}/media_publish` | E1 | high | supported |
| C6 | IG API publish rate limit is 100 posts / 24h moving window | E1 | high | supported |
| C7 | Standard/Development access is enough when only app-role users (Frank) use the app on owned assets | E4 E5 | high | supported |
| C8 | Page insights require `read_insights` + `pages_read_engagement` + ANALYZE task | E6 | high | supported |
| C9 | IG insights (FB Login) require `instagram_basic`, `instagram_manage_insights`, `pages_read_engagement` | E7 | high | supported |
| C10 | OAuth redirect_uri must exactly match dashboard allowlist and code exchange | E8 | high | supported |
| C11 | Graph errors use `{error:{message,type,code,error_subcode,fbtrace_id,...}}` | E9 | high | supported |
| C12 | IG image format for publishing is JPEG-only | E1 | high | supported |
| C13 | `pages_manage_read_engagement` vs `pages_read_engagement` naming is consistent across all Meta surfaces | E2 E10 | medium | partial |
| C14 | `http://localhost` is always accepted as Valid OAuth Redirect URI in current dashboard | E8 E11 | low | unsupported |
| C15 | Recommended first metrics list is product-optimal for Tomato UX | E6 E7 | medium | partial |

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | IG Content Publishing | https://developers.facebook.com/documentation/instagram-platform/content-publishing.md | web_extract markdown | Public media cURL requirement; FB Login permissions table; container/publish/status; JPEG-only; 100 posts/24h |
| E2 | Page Photos reference v26 | https://developers.facebook.com/docs/graph-api/reference/page/photos/ | web_extract | Create requirements; specs; url/multipart upload; error codes 324/190/200/283 |
| E3 | Long-lived tokens | https://developers.facebook.com/documentation/facebook-login/guides/access-tokens/get-long-lived.md | web_extract | fb_exchange_token; ~60d user; Page token via /accounts; no fixed Page expiry |
| E4 | Access levels | https://developers.facebook.com/docs/graph-api/overview/access-levels/ | web_extract | Standard vs Advanced; Advanced needs BV; Standard for role users |
| E5 | Build and Test / modes | https://developers.facebook.com/docs/development/build-and-test/ | web_extract | Development vs Live; role-gated permissions in Dev |
| E6 | Page Insights guide | https://developers.facebook.com/docs/platforminsights/page/ | web_extract | read_insights + pages_read_engagement; metric examples; common errors |
| E7 | IG Insights guide | https://developers.facebook.com/docs/instagram-platform/insights/ | web_extract | Permission tables; account + media metric examples |
| E8 | Manual login flow | https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow/ | web_extract | dialog/oauth; redirect_uri; code exchange; Valid OAuth Redirect URIs |
| E9 | Graph error handling | https://developers.facebook.com/docs/graph-api/guides/error-handling/ | web_extract | Error envelope; codes 190/10/4/17/506/368; auth subcodes |
| E10 | Pages get-started | https://developers.facebook.com/docs/pages-api/getting-started/ | web_extract | pages_manage_metadata/posts/manage_read_engagement/show_list; /me/accounts |
| E11 | Pages posts guide | https://developers.facebook.com/documentation/pages-api/posts.md | web_extract | feed/photos publish; permission list including pages_read_engagement |
| E12 | IG API with FB Login get-started | https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-facebook-login/get-started.md | web_extract | me/accounts → page → instagram_business_account → media |
| E13 | Token debug/errors | https://developers.facebook.com/docs/facebook-login/access-tokens/debugging-and-error-handling/ | web_extract | debug_token; expired/invalid sample JSON |
| E14 | IG Platform overview | https://developers.facebook.com/documentation/instagram-platform/overview.md | web_extract | FB Login vs IG Login comparison; Standard/Advanced; token lifetimes |
| E15 | Publish/release | https://developers.facebook.com/docs/development/release/ | web_extract | App Review + Business Verification before non-role users |

---

## gaps

1. **OAuth localhost allowlist** — confirm current App Dashboard accepts `http://localhost:3000/...` for this app type (C14).  
2. **Exact dashboard permission labels** — reconcile `pages_read_engagement` vs `pages_manage_read_engagement` in the live permission picker (C13).  
3. **Use-case wizard path** — 2026 app creation UI may auto-bundle permissions differently; map checklist scopes onto whatever use case Frank selects.  
4. **Page Insights metric deprecations (through mid-2026)** — final Tomato metric enum should be re-checked against Insights reference near implementation.  
5. **Whether Development-mode Page posts are fully public** — operational confirmation on Frank’s Page.  
6. **Business Manager–granted Page roles** — if Frank’s Page access is BM-only, may need `ads_management`/`ads_read` for IG calls.  
7. **Webhook subscriptions** — not required for v1 publish; needed later for comment/realtime efficiency.  
8. **Tomato repo current env conventions** — `/srv/apps/tomato` not mounted in this research container; Su should align env names with existing T0 scaffold.

---

## confidence

```json
{
  "level": "high",
  "rationale": "Load-bearing publish, permission, token, and error claims are grounded in official Meta markdown/HTML docs extracted 2026-09-02 (Graph v25/v26). Residual uncertainty is limited to App Dashboard UI labeling, localhost OAuth allowlisting, and 2026 insights metric deprecation details — all marked UNVERIFIED/partial."
}
```

---

## recommended_next_actions

1. **Su:** Implement Meta provider skeleton against this checklist (OAuth + token vault + FB photo + IG container pipeline + metrics job) using fake fixtures first.  
2. **Frank (wake-up, ~15 min):** Create app, paste `META_APP_ID`/`META_APP_SECRET`, register redirect URIs, connect Page/IG, run dual publish smoke test.  
3. **Su/Frank:** Run Graph Explorer preflight (§6) before first production-shaped publish.  
4. **Geraldo/Su:** After first live smoke test, file a short addendum confirming C13/C14 and final metric names.  
5. **Defer App Review** until Tomato must onboard non-role Meta users.

---

## Appendix A — Copy-paste endpoint cheat sheet

```
# OAuth dialog
https://www.facebook.com/v26.0/dialog/oauth?client_id=APP&redirect_uri=CB&state=S&scope=SCOPES&response_type=code

# Code exchange
GET /v26.0/oauth/access_token?client_id=APP&redirect_uri=CB&client_secret=SECRET&code=CODE

# Long-lived user
GET /v26.0/oauth/access_token?grant_type=fb_exchange_token&client_id=APP&client_secret=SECRET&fb_exchange_token=SHORT

# Pages
GET /v26.0/me/accounts

# IG user id
GET /v26.0/{page-id}?fields=instagram_business_account

# FB photo
POST /v26.0/{page-id}/photos  (url|source, caption, published)

# IG container / status / publish
POST /v26.0/{ig-id}/media
GET  /v26.0/{creation-id}?fields=status_code
POST /v26.0/{ig-id}/media_publish  {creation_id}

# Metrics
GET /v26.0/{page-id}/insights?metric=...
GET /v26.0/{page-post-id}/insights?metric=...
GET /v26.0/{ig-id}/insights?metric=impressions,reach,profile_views&period=day
GET /v26.0/{ig-media-id}/insights?metric=engagement,impressions,reach
```

## Appendix B — Frank paste card (no secrets in repo)

```
META_APP_ID=
META_APP_SECRET=
META_OAUTH_REDIRECT_URI=http://localhost:3000/api/meta/oauth/callback
META_GRAPH_API_VERSION=v26.0
```

Dashboard checks:

- [ ] Facebook Login → Valid OAuth Redirect URIs includes callback  
- [ ] App Roles includes Frank as Admin  
- [ ] Page linked to IG professional  
- [ ] Development mode OK for solo use  

---

*Report path: `/home/ice/fed/docs/tomato/research-meta-t_8c8d1279.md`*  
*Also mirrored to research-output and kanban attachment on completion.*
