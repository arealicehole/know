# Tomato TikTok Spike Research — Exact API / Keys / Scopes / Audit Checklist

**Task:** `t_b8a7d332`  
**Product:** Tomato (approval-first social command center, local-first Next.js)  
**Huly:** TOM-1  
**Research date:** 2026-09-02  
**Primary sources:** TikTok for Developers official docs (browser-extracted; last-updated stamps Aug 4–24, 2026)  
**Secrets policy:** env var *names only* — no key values in this document.

---

## objective

Produce an exact TikTok integration/spike checklist so Tomato can be built as far as possible offline; when Frank pastes API credentials he knows precisely what TikTok can/cannot do (direct public vs private-only vs upload fallback vs blocked), with official doc URLs on every load-bearing claim.

## summary

TikTok Content Posting API supports **two product modes**:

| Mode | Scope | Endpoint family | User experience |
|------|-------|-----------------|-----------------|
| **Direct Post** | `video.publish` | `/v2/post/publish/video/init/` (video) or `/v2/post/publish/content/init/` with `post_mode=DIRECT_POST` (photo) | Posts straight to profile (async process + status poll) |
| **Upload / draft** | `video.upload` | `/v2/post/publish/inbox/video/init/` (video) or `/v2/post/publish/content/init/` with `post_mode=MEDIA_UPLOAD` (photo) | Sends draft to creator **inbox**; user must open TikTok, edit, finish post |

**Unaudited apps cannot do public Direct Post.** Official guidelines: unaudited clients may only post `SELF_ONLY`, require **private TikTok accounts**, and are capped at **5 users / 24h**. Audit lifts private-viewership restriction. Photos for both modes are **PULL_FROM_URL only** (no FILE_UPLOAD) and require a **verified domain or URL prefix**. Videos may use FILE_UPLOAD or PULL_FROM_URL.

**Bottom line for Tomato spike expectation:** plan for **private-only Direct Post + MEDIA_UPLOAD fallback** until audit passes. Do **not** promise public scheduling until Frank completes TikTok app audit with compliant UX demo.

## Frank wake-up checklist (paste keys → run)

1. Create/open TikTok for Developers org + app → copy `client_key` / `client_secret`.
2. Add products: **Login Kit** + **Content Posting API**.
3. Enable **Direct Post** configuration on Content Posting API (in-app toggle).
4. Request scopes: `user.info.basic` (login), `video.publish` (direct), `video.upload` (inbox fallback).
5. Register **HTTPS** redirect URI(s) (static, no query/hash; max 10).
6. Verify **URL property** (domain or URL prefix) for media PULL_FROM_URL.
7. Configure webhook URL (optional but recommended) for publish events.
8. Paste env vars into Tomato (names below).
9. OAuth one private TikTok test account.
10. Run capability probe (section 11) → store result enum.
11. Only after successful private posts + compliant UX demo video → submit **app audit**.

---

## 1. Required TikTok developer app setup steps

Ordered setup (official Get Started + Login Kit + App Review):

1. **Register** on https://developers.tiktok.com and create an app under an organization.  
   Source: [Login Kit Web](https://developers.tiktok.com/docs/en/login-kit-web)
2. Obtain **client_key** and **client_secret** from Manage apps.  
   Source: same
3. Fill app details for eventual review: custom name/icon, public website (not bare login page), Privacy Policy + ToS visible without menu, description of intended audience use.  
   Source: [App Review Guidelines](https://developers.tiktok.com/docs/en/app-review-guidelines)  
   **Note:** “Apps that are still in development or testing will not be approved” and “Apps must not be for private or personal use” — Tomato must be framed as a product for creators/brands, not an internal-only utility. Content Sharing Guidelines also reject “utility tool to help upload contents to the account(s) you or your team manages.”  
   Source: [Content Sharing Guidelines](https://developers.tiktok.com/docs/en/content-sharing-guidelines)
4. Add product **Content Posting API**.  
   Source: [Get Started - Direct Post](https://developers.tiktok.com/docs/en/content-posting-api-get-started)
5. **Enable Direct Post configuration** on that product (required for direct posting).  
   Source: same
6. Add product **Login Kit**; register web redirect URI(s).  
   Source: [Login Kit Web](https://developers.tiktok.com/docs/en/login-kit-web)
7. Apply for scopes **`video.publish`** and/or **`video.upload`** (app must be approved for the scope; user must also authorize it).  
   Source: Direct Post + Upload get-started docs
8. Add **URL properties** (domain or URL prefix) and verify ownership (DNS signature recommended for domain).  
   Source: [Media Transfer Guide](https://developers.tiktok.com/docs/en/content-posting-api-media-transfer-guide)
9. (Optional) Register **webhook** endpoint for Content Posting events.  
   Source: [Get Post Status](https://developers.tiktok.com/docs/en/content-posting-api-reference-get-video-status)
10. Test in **sandbox** (required for first-time review demo videos).  
    Source: [App Review Guidelines](https://developers.tiktok.com/docs/en/app-review-guidelines)
11. After integration works privately → submit audit with end-to-end demo video (max 5 videos, 50 MB each) covering every selected product/scope.  
    Source: same + Content Sharing Guidelines

---

## 2. Exact scopes

| Capability | Scope | Notes |
|------------|-------|-------|
| Login / identity baseline | `user.info.basic` | Login Kit example scope; request only what you need |
| Creator info query (required before Direct Post UI) | **`video.publish`** | Creator Info API scope is `video.publish` only |
| Direct photo / video post | **`video.publish`** | App approved + user authorized |
| Upload to TikTok editing / inbox fallback (video or photo MEDIA_UPLOAD) | **`video.upload`** | App approved + user authorized |
| Photo endpoint either mode | `video.publish` **or** `video.upload` | Photo API lists both; mode decides which user grant is needed |

**OAuth authorize URL:** `https://www.tiktok.com/v2/auth/authorize/`  
**Scope param:** comma-separated string, e.g. `user.info.basic,video.publish,video.upload`  
Source: [Login Kit Web](https://developers.tiktok.com/docs/en/login-kit-web)

**Recommended Tomato OAuth request for full spike:**

```text
user.info.basic,video.publish,video.upload
```

Applying for a scope ≠ user grant. User can deny individual toggleable scopes. Store granted `scope` string from token response.

---

## 3. Redirect / callback URL patterns Tomato should implement

### Registration rules (Login Kit Web)

- Max **10** redirect URIs  
- Each URI length **&lt; 512** chars  
- Must be **absolute** and begin with **`https`**  
- Must be **static** — **no query parameters**  
- **No fragment** (`#`)  
- Callback `redirect_uri` must **exactly match** a registered URI  

Correct example from docs: `https://dev.example.com/auth/callback/`  

### Tomato route recommendations (local-first Next.js)

Register in TikTok portal (production / tunnel host — **not** bare `http://localhost`):

| Purpose | Suggested path pattern |
|---------|------------------------|
| OAuth callback | `https://{public-host}/api/auth/tiktok/callback` |
| Alternate env | `https://{staging-host}/api/auth/tiktok/callback` |
| Webhook receiver | `https://{public-host}/api/webhooks/tiktok` (configured separately as webhook URL, not Login Kit redirect) |

**App routes Tomato should implement:**

1. `GET /api/auth/tiktok` — start OAuth: create CSRF `state`, set httpOnly cookie, redirect to TikTok authorize URL.  
2. `GET /api/auth/tiktok/callback` — validate `state`, exchange `code` → tokens server-side, store encrypted refresh token.  
3. `POST /api/webhooks/tiktok` — verify + handle publish events (if configured).  
4. `GET /api/tiktok/creator-info` — proxy creator_info/query for export UI.  
5. Worker-only routes for init/status (never expose client_secret).

### Localhost reality

- Official docs require **https** redirect URIs.  
- Community reports that plain localhost often cannot be registered / fails (Stack Overflow historically).  
- **UNVERIFIED in dashboard:** whether TikTok currently accepts `https://localhost:...`, loopback with TLS, or only public domains.  
- **Frank must confirm** in Login Kit config what URIs the portal accepts. Practical local-first pattern: Cloudflare Tunnel / ngrok / Caddy public HTTPS hostname pointed at local Next.js, register that HTTPS URL.

### OAuth authorize query (application/x-www-form-urlencoded)

| Param | Value |
|-------|-------|
| `client_key` | app client key |
| `scope` | comma-separated scopes |
| `redirect_uri` | exact registered URI |
| `state` | CSRF token |
| `response_type` | `code` |
| `disable_auto_auth` | `0` or `1` (optional) |

Callback success params: `code`, `scopes`, `state`  
Callback error params: `error`, `error_description`  

Source: [Login Kit Web](https://developers.tiktok.com/docs/en/login-kit-web)

### Token exchange

- `POST https://open.tiktokapis.com/v2/oauth/token/`  
- Content-Type: `application/x-www-form-urlencoded`  
- Body: `client_key`, `client_secret`, `code` (URL-decoded), `grant_type=authorization_code`, `redirect_uri` (same as authorize)  
- Returns: `access_token` (~24h / `expires_in` 86400), `refresh_token` (~365d), `open_id`, `scope`, `token_type=Bearer`  
- Refresh: same endpoint, `grant_type=refresh_token` + `refresh_token`  

Source: [User Access Token Management](https://developers.tiktok.com/docs/en/oauth-user-access-token-management)

---

## 4. Public media URL + verified domain / URL-prefix requirements

### Photo posts (critical for Tomato photo path)

- Source **must** be `PULL_FROM_URL` only — **FILE_UPLOAD not allowed for photos**.  
- `photo_images`: up to **35** HTTPS URLs, publicly accessible, under verified property.  
- `photo_cover_index`: 0-based index of cover image.  
- Formats: **WebP, JPEG**; max **1080p**; max **20 MB** per image.  
Source: [Photo API](https://developers.tiktok.com/docs/en/content-posting-api-reference-photo-post), [Media Transfer Guide](https://developers.tiktok.com/docs/en/content-posting-api-media-transfer-guide)

### Video PULL_FROM_URL

- HTTPS URL, **no redirects** (3xx invalid).  
- URL must stay reachable for entire download; download **times out after 1 hour**.  
- Ingress bandwidth up to ~100 Mbps.  
- Formats: MP4 (rec), WebM, MOV; codecs H.264/H.265/VP8/VP9; FPS 23–60; min 360px side; max 4096; max **4 GB**; max init duration 10 minutes.  
Source: Media Transfer Guide

### Ownership verification

1. In TT4D app → **URL properties** widget → add Domain or URL Prefix.  
2. Must have manage/write access to property.  
3. **Domain:** DNS signature string recommended; once verified, domain + subdomains under it count.  
   - Verifying `static.example.com` covers `https://video.static.example.com/...`  
   - Does **not** cover `https://example.com/...`  
4. **URL Prefix:** `https://` + host + path + `/` (host = domain, not IP). Exact prefix match only.  
5. Failure error: `url_ownership_unverified` (403).  
6. Docs mention a convenient **test URL that works without verification** for Pull-from-URL testing — **UNVERIFIED exact URL string** (screenshot/widget-only on portal); Frank should copy from Media Transfer Guide page in dashboard session.  

Source: [Media Transfer Guide](https://developers.tiktok.com/docs/en/content-posting-api-media-transfer-guide)

### Tomato media hosting implication

Local-first app still needs **public HTTPS media origin** TikTok can pull (object storage / CDN / tunnel) with verified prefix, e.g.:

```text
https://media.tomato.example.com/exports/{userId}/{assetId}.jpg
```

Register prefix `https://media.tomato.example.com/exports/` or domain `media.tomato.example.com`.

---

## 5. Required env vars (names only)

```bash
# TikTok app credentials (server-only)
TIKTOK_CLIENT_KEY=
TIKTOK_CLIENT_SECRET=

# OAuth
TIKTOK_REDIRECT_URI=https://your.host/api/auth/tiktok/callback
# Optional explicit authorize/token bases if you abstract them
TIKTOK_AUTH_URL=https://www.tiktok.com/v2/auth/authorize/
TIKTOK_TOKEN_URL=https://open.tiktokapis.com/v2/oauth/token/
TIKTOK_API_BASE=https://open.tiktokapis.com

# Scopes requested at connect time
TIKTOK_SCOPES=user.info.basic,video.publish,video.upload

# Media pull origin (must match verified URL property)
TIKTOK_MEDIA_PUBLIC_BASE_URL=https://media.your.host/exports/
# Optional: signed URL TTL seconds for pull objects
TIKTOK_MEDIA_URL_TTL_SECONDS=3600

# Webhooks (optional)
TIKTOK_WEBHOOK_PATH=/api/webhooks/tiktok
# If TikTok provides a signing secret in portal — UNVERIFIED name:
# TIKTOK_WEBHOOK_SECRET=

# Encryption for stored tokens at rest (Tomato-side)
TIKTOK_TOKEN_ENCRYPTION_KEY=

# Feature / capability flags written by spike probe (non-secret)
TIKTOK_CAPABILITY_MODE=unknown
# one of: direct_public | private_only | upload_fallback | blocked
```

**Do not** put client_secret in client bundles or public repos. Content Sharing Guidelines: never share credentials / embed client_secret in open source.

Per-user secrets (DB, encrypted): `access_token`, `refresh_token`, `open_id`, `scope`, `access_token_expires_at`, `refresh_token_expires_at`.

---

## 6. Development / unaudited restrictions vs audit

### Unaudited Direct Post restrictions

From [Content Sharing Guidelines](https://developers.tiktok.com/docs/en/content-sharing-guidelines) and Direct Post / Photo docs:

| Restriction | Rule |
|-------------|------|
| Visibility | **SELF_ONLY only** |
| Account type | Posting user accounts **must be private** at time of post |
| User cap | Up to **5 users** may post per client per **24h** |
| Public lift | After posting private, owner must later set account public **and** change each post privacy to Everyone to surface content |
| Error if violated | `unaudited_client_can_only_post_to_private_accounts` — blocked at `/publish/content/init/` |

### Applies to audited **and** unaudited

| Cap | Rule |
|-----|------|
| Active creator cap | 24h active publishing users per client (from audit usage estimates) → error `reached_active_user_cap` |
| Per-creator posting cap | ~**15 posts / day / creator** via Direct Post (varies; shared across all API clients) → `spam_risk_too_many_posts` |
| Pending upload shares | At most **5 pending** inbox shares / 24h per user → `spam_risk_too_many_pending_share` |

### What audit is

- Submit app for review/audit verifying compliance with Terms of Service + Content Sharing UX guidelines.  
- App Review requires: complete app metadata, public website + policies, scope justification, **demo video** of full flow (sandbox for first approval).  
- Lifts **private viewing mode** restriction on Direct Post content.  
- Does **not** remove rate/spam caps.  
- **Intended use gate:** product must serve authentic creators posting original content to a wide audience — not bulk cross-post scrapers or internal multi-account upload tools.  
Sources: Content Sharing Guidelines, App Review Guidelines, Get Started Direct Post note.

### Tomato product implication

Until audit: scheduler may queue “public” intent but **must execute as SELF_ONLY** (or refuse public intent with clear UI). After audit: honor user-selected privacy from creator_info options.

---

## 7. Creator-info / privacy-level UI requirements

### API

```http
POST https://open.tiktokapis.com/v2/post/publish/creator_info/query/
Authorization: Bearer {user_access_token}
Content-Type: application/json; charset=UTF-8
```

**Scope:** `video.publish`  
**Rate:** **20 requests / minute / user access_token**

### Response fields Tomato must render

| Field | UI use |
|-------|--------|
| `creator_avatar_url` | Show account (TTL ~2h) |
| `creator_username` | Show handle |
| `creator_nickname` | **Required** on export page so user knows target account |
| `privacy_level_options` | **Only** these options in privacy dropdown |
| `comment_disabled` | Grey out / disable Allow Comment if true |
| `duet_disabled` | Grey out Duet (video only; ignore for photo-only) |
| `stitch_disabled` | Grey out Stitch (video only) |
| `max_video_post_duration_sec` | Block too-long videos |

**Privacy option sets:**

- Public account: `PUBLIC_TO_EVERYONE`, `MUTUAL_FOLLOW_FRIENDS`, `SELF_ONLY`  
- Private account: `FOLLOWER_OF_CREATOR`, `MUTUAL_FOLLOW_FRIENDS`, `SELF_ONLY`  

Source: [Query Creator Info](https://developers.tiktok.com/docs/en/content-posting-api-reference-query-creator-info)

### Mandatory UX (Content Sharing Guidelines)

1. Call creator_info **when rendering** Post-to-TikTok page (fresh).  
2. Show nickname.  
3. If creator cannot post more now → **stop** and prompt retry later.  
4. Privacy: dropdown from `privacy_level_options` only; **no default value**; user must manually select.  
5. Interactions: Comment / Duet / Stitch — **none checked by default**; grey out if creator disabled; **photos: Comment only**.  
6. Consent text before publish:  
   - Default: `By posting, you agree to TikTok's Music Usage Confirmation`  
   - Branded content: also Branded Content Policy  
7. Commercial disclosure toggle **off by default**; if on, require Your Brand and/or Branded Content; disable publish until chosen.  
8. Branded content **cannot** be SELF_ONLY / private.  
9. Preview content; editable title/hashtags; no promotional watermarks.  
10. Start send **only after express consent**.  
11. Notify that processing may take minutes.  
12. Poll status or webhooks.  
13. Map to API: `brand_organic_toggle` (Your Brand), `brand_content_toggle` (Branded Content).  

Mismatch → `privacy_level_option_mismatch` (treated as product-guidance violation signal).

---

## 8. Rate limits that matter for scheduler/worker

| Endpoint / action | Limit | Source |
|-------------------|-------|--------|
| Photo `/content/init/` | **6 req/min** per user access_token | Photo API |
| Video direct `/video/init/` | **6 req/min** per user access_token | Direct Post / search snippets + upload video ref |
| Inbox video `/inbox/video/init/` | **6 req/min** per user access_token | Upload video ref |
| Creator info query | **20 req/min** per user access_token | Creator Info API |
| Status fetch | **30 req/min** per user access_token | Get Post Status |
| Global | `rate_limit_exceeded` HTTP 429 | all posting docs |
| Unaudited users | **5 users / 24h** | Content Sharing Guidelines |
| Direct posts / creator | ~**15 / day** (varies) | Content Sharing Guidelines |
| Pending MEDIA_UPLOAD shares | **≤5 pending / 24h** | Photo API error text |
| Active publishing users / client | daily quota → `reached_active_user_cap` | Creator Info / Photo errors |
| Access token lifetime | **24h**; refresh token **~365d** | OAuth token docs |

**Worker design:**

- Per-`open_id` token bucket: ≤6 init/min, ≤30 status/min.  
- Daily counters for posts + pending uploads.  
- Exponential backoff on 429 / `internal_error`.  
- Status poll backoff: start ~2–5s, back off; moderation can take minutes–hours.  
- Prefer webhooks to reduce poll load.  
- Do not fan out multi-account bursts under unaudited 5-user cap.

---

## 9. Minimal direct photo post sequence

**Prereqs:** `video.publish` granted; Direct Post enabled; media URLs on verified property; creator_info fetched; user selected privacy (unaudited → only `SELF_ONLY`); consent + commercial rules satisfied.

```text
1. POST /v2/post/publish/creator_info/query/
2. Render export UI from response; collect privacy_level, disable_comment, title, description, commercial toggles
3. Ensure images hosted at verified HTTPS URLs (WebP/JPEG, ≤20MB, ≤1080p), ≤35 images
4. POST /v2/post/publish/content/init/
   Headers:
     Authorization: Bearer {access_token}
     Content-Type: application/json; charset=UTF-8
   Body:
   {
     "media_type": "PHOTO",
     "post_mode": "DIRECT_POST",
     "post_info": {
       "title": "...",                    // max 90 UTF-16 runes
       "description": "...",              // max 4000 UTF-16 runes
       "privacy_level": "SELF_ONLY",      // must be in privacy_level_options
       "disable_comment": false,
       "auto_add_music": true,
       "brand_content_toggle": false,
       "brand_organic_toggle": false
     },
     "source_info": {
       "source": "PULL_FROM_URL",
       "photo_cover_index": 0,
       "photo_images": ["https://verified.../1.jpg"]
     },
     "is_aigc": false
   }
5. Read data.publish_id (e.g. p_pub_url~v2....)
6. Loop POST /v2/post/publish/status/fetch/ { "publish_id": "..." }
   until status in { PUBLISH_COMPLETE, FAILED }
7. On PUBLISH_COMPLETE: store publish_id; publicaly_available_post_id may stay empty for private posts
```

**Note:** Photo API table marks `brand_*_toggle` as required=true while official curl examples omit them. Spike should try with explicit `false`/`false` first. Mark residual behavior **UNVERIFIED** if init rejects missing fields.

**Direct video** (same scope): `POST /v2/post/publish/video/init/` with `post_info` + `source_info` FILE_UPLOAD or PULL_FROM_URL; if FILE_UPLOAD, PUT chunks to `upload_url` then status fetch.

---

## 10. Minimal upload-to-TikTok-editing fallback sequence

**Prereqs:** `video.upload` granted; inform user they must open **inbox notification** in TikTok app to finish.

### Photo MEDIA_UPLOAD

```text
1. POST /v2/post/publish/content/init/
   {
     "media_type": "PHOTO",
     "post_mode": "MEDIA_UPLOAD",
     "post_info": { "title": "...", "description": "..." },
     "source_info": {
       "source": "PULL_FROM_URL",
       "photo_cover_index": 0,
       "photo_images": ["https://verified.../1.jpg"]
     }
   }
2. publish_id returned
3. Status → SEND_TO_USER_INBOX (notification delivered)
4. User completes in TikTok → PUBLISH_COMPLETE (may never happen if ignored)
5. UX must tell user to open TikTok inbox
```

**App version gate:** MEDIA_UPLOAD requires TikTok app **≥ 31.8** or `app_version_check_failed`.

### Video inbox upload

```text
POST /v2/post/publish/inbox/video/init/
{ "source_info": { "source": "FILE_UPLOAD"|"PULL_FROM_URL", ... } }
→ optional PUT upload_url
→ status fetch / webhooks post.publish.inbox_delivered
```

No privacy_level on inbox init — user sets privacy in TikTok editor.

---

## 11. What Tomato should store to prove capability

After OAuth + one controlled probe post, persist a **capability record** per app (and optionally per connected account):

```ts
type TikTokCapability =
  | "direct_public"      // audited path: PUBLIC_TO_EVERYONE accepted + public post_id path works
  | "private_only"       // unaudited: SELF_ONLY works; public rejected/forced
  | "upload_fallback"    // MEDIA_UPLOAD / inbox works; direct publish scope missing or blocked
  | "blocked";           // auth, scope, ownership, ban, or review blocks posting entirely

interface TikTokCapabilityProof {
  capability: TikTokCapability;
  probed_at: string; // ISO
  app_audit_status: "unaudited" | "audit_submitted" | "audited" | "unknown";
  scopes_granted: string[];          // from token
  scopes_missing: string[];
  creator_info_ok: boolean;
  direct_photo_init_ok: boolean;
  last_direct_privacy_attempted: string | null;
  last_direct_privacy_accepted: string | null;
  last_publish_id: string | null;
  last_status: string | null;
  last_error_code: string | null;
  last_fail_reason: string | null;
  media_url_ownership_ok: boolean;
  upload_inbox_ok: boolean;
  notes: string;
}
```

### Probe procedure (automated)

1. Token present? else `blocked` (`access_token_invalid`).  
2. Creator info with `video.publish`? if scope_not_authorized → try upload-only path.  
3. Init DIRECT_POST photo with `PUBLIC_TO_EVERYONE` (only if in options).  
   - Success path + later public post_id / webhook `post.publish.publicly_available` → candidate `direct_public`.  
   - `unaudited_client_can_only_post_to_private_accounts` or forced SELF_ONLY success → `private_only`.  
   - `privacy_level_option_mismatch` → retry SELF_ONLY.  
4. If direct fails for scope → MEDIA_UPLOAD with `video.upload` → success inbox → `upload_fallback`.  
5. `url_ownership_unverified` → `blocked` until domain verified (or capability `blocked` with reason ownership).  
6. Never claim `direct_public` without either audited confirmation **or** observed public availability event.

**Default expectation pre-Frank-audit:** `private_only` if publish works; else `upload_fallback`; else `blocked`.

---

## 12. Common failure modes and exact error shapes

### Envelope (Content Posting APIs)

```json
{
  "data": { },
  "error": {
    "code": "ok",
    "message": "",
    "log_id": "202210112248442CB9319E1FB30C1073F3"
  }
}
```

Success: `error.code === "ok"`. Any other `error.code` = failure. Always log `log_id`.

### OAuth token error shape (different)

```json
{
  "error": "invalid_request",
  "error_description": "Redirect_uri is not matched with the uri when requesting code.",
  "log_id": "..."
}
```

### Init / creator_info error codes (handle explicitly)

| HTTP | error.code | Meaning / Tomato action |
|------|------------|-------------------------|
| 400 | `invalid_param` | Fix payload; surface message |
| 400 | `app_version_check_failed` | MEDIA_UPLOAD needs TikTok ≥ 31.8 |
| 403 | `spam_risk_too_many_posts` | Daily direct post cap; reschedule next day |
| 403 | `spam_risk_user_banned_from_posting` | Do not retry |
| 403 | `spam_risk_too_many_pending_share` | Too many pending inbox uploads (max 5/24h) |
| 403 | `reached_active_user_cap` | Client daily active publisher quota |
| 403 | `unaudited_client_can_only_post_to_private_accounts` | Force private-only mode / mark `private_only` |
| 403 | `url_ownership_unverified` | Verify domain/prefix; fix media base URL |
| 403 | `privacy_level_option_mismatch` | Re-fetch creator_info; fix dropdown |
| 401 | `access_token_invalid` | Refresh token; re-OAuth if refresh fails |
| 401 | `scope_not_authorized` | Missing video.publish or video.upload grant |
| 429 | `rate_limit_exceeded` | Backoff |
| 5xx | `internal_error` | Retryable |

### Status `fail_reason` values

| fail_reason | Guidance |
|-------------|----------|
| `file_format_check_failed` | Bad media format |
| `duration_check_failed` | Video duration |
| `frame_rate_check_failed` | FPS |
| `picture_size_check_failed` | Image/video dimensions |
| `internal` | Retryable TikTok issue |
| `video_pull_failed` | URL pull failed/timeout 1h; check public access/bandwidth; retry if sure |
| `photo_pull_failed` | Same for photos |
| `publish_cancelled` | Cancel API used |
| `auth_removed` | User revoked access mid-flight; no retry |
| `spam_risk_too_many_posts` | Creator daily API post cap |
| `spam_risk_user_banned_from_posting` | Banned; no retry |
| `spam_risk_text` | Caption/text spam; no retry |
| `spam_risk` | Generic TnS block; no retry |

### Status enum

`PROCESSING_UPLOAD` | `PROCESSING_DOWNLOAD` | `SEND_TO_USER_INBOX` | `PUBLISH_COMPLETE` | `FAILED`

### Webhooks

| Event | Values | Meaning |
|-------|--------|---------|
| `post.publish.failed` | publish_id, reason, publish_type | Failed |
| `post.publish.complete` | publish_id, publish_type | User finished (upload path may multi-post) |
| `post.publish.inbox_delivered` | publish_id, publish_type=`INBOX_SHARE` | Inbox notified |
| `post.publish.publicly_available` | publish_id, post_id, publish_type | Public after moderation |
| `post.publish.no_longer_publicaly_available` | publish_id, post_id, publish_type | No longer public |

Note docs typo: `publicaly` / `publicaly_available_post_id`.

### Cancel pull

`POST /v2/post/publish/cancel/` with `publish_id` — best-effort; errors include `invalid_publish_id`, `publish_not_cancellable`, etc.

Sources: Photo API, Creator Info, Get Post Status, Media Transfer Guide, OAuth docs.

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | Direct Post requires scope `video.publish` (app + user) | Get Started Direct Post | high | supported |
| C2 | Upload/inbox requires scope `video.upload` | Get Started Upload; Photo Upload section | high | supported |
| C3 | Creator info query requires `video.publish`, 20 rpm | Creator Info API | high | supported |
| C4 | Photo init is `/v2/post/publish/content/init/` with media_type PHOTO and post_mode DIRECT_POST or MEDIA_UPLOAD | Photo API | high | supported |
| C5 | Photo source is PULL_FROM_URL only; ≤35 images | Photo API | high | supported |
| C6 | Unaudited Direct Post is SELF_ONLY + private accounts + 5 users/24h | Content Sharing Guidelines | high | supported |
| C7 | Audit required to lift private viewership restriction | Direct Post note; Content Sharing Guidelines | high | supported |
| C8 | Privacy dropdown must use creator_info options with no default | Content Sharing Guidelines | high | supported |
| C9 | Content init rate limit 6 rpm per user token | Photo API | high | supported |
| C10 | Status fetch 30 rpm; statuses include PROCESSING_*, SEND_TO_USER_INBOX, PUBLISH_COMPLETE, FAILED | Get Post Status | high | supported |
| C11 | Media domain/URL prefix must be verified; https no redirect | Media Transfer Guide | high | supported |
| C12 | Access token ~24h; refresh ~365d; token endpoint `/v2/oauth/token/` | OAuth User Access Token Management | high | supported |
| C13 | Redirect URIs must be https, static, ≤10 | Login Kit Web | high | supported |
| C14 | ~15 posts/day/creator Direct Post cap | Content Sharing Guidelines | medium | supported |
| C15 | brand_* toggles always required in JSON body | Photo table vs examples disagree | low | partial |
| C16 | Localhost http redirect works for Login Kit | Docs require https; community negative | low | unsupported — UNVERIFIED dashboard |
| C17 | Exact no-verify test media URL string | Docs allude; not copied as stable string | low | unsupported — UNVERIFIED |
| C18 | Webhook signing secret env name / algorithm | Events documented; crypto not fully extracted | low | partial — UNVERIFIED |

## evidence_index

| evidence_id | source_label | source_url | provenance | support note |
|-------------|--------------|------------|------------|--------------|
| E1 | Get Started - Direct Post | https://developers.tiktok.com/docs/en/content-posting-api-get-started | browser main.innerText 2026-09-02; updated Aug 4, 2026 | prereqs, video.publish, unaudited private note, creator_info, video/photo examples |
| E2 | Get Started - Upload | https://developers.tiktok.com/docs/en/content-posting-api-get-started-upload-content | browser; Aug 4, 2026 | video.upload, inbox init, MEDIA_UPLOAD photo |
| E3 | Photo API reference | https://developers.tiktok.com/docs/en/content-posting-api-reference-photo-post | browser; Aug 24, 2026 | fields, 6 rpm, errors, scopes |
| E4 | Query Creator Info | https://developers.tiktok.com/docs/en/content-posting-api-reference-query-creator-info | browser; Aug 4, 2026 | 20 rpm, privacy options, errors |
| E5 | Media Transfer Guide | https://developers.tiktok.com/docs/en/content-posting-api-media-transfer-guide | browser; Aug 4, 2026 | chunks, PULL_FROM_URL, domain/prefix, media limits |
| E6 | Content Sharing Guidelines | https://developers.tiktok.com/docs/en/content-sharing-guidelines | browser; Aug 4, 2026 | unaudited caps, UX, intended use, technical |
| E7 | Login Kit Web | https://developers.tiktok.com/docs/en/login-kit-web | browser; Aug 4, 2026 | redirect rules, authorize URL, callback params |
| E8 | User Access Token Management | https://developers.tiktok.com/docs/en/oauth-user-access-token-management | browser; Aug 4, 2026 | token/refresh endpoints and lifetimes |
| E9 | Get Post Status | https://developers.tiktok.com/docs/en/content-posting-api-reference-get-video-status | browser; Aug 4, 2026 | 30 rpm, statuses, fail_reason, webhooks |
| E10 | App Review Guidelines | https://developers.tiktok.com/docs/en/app-review-guidelines | browser; Aug 4, 2026 | review criteria, demo video, sandbox |
| E11 | Content Posting API product page | https://developers.tiktok.com/products/content-posting-api/ | search listing | product positioning direct vs draft |

## gaps

1. **UNVERIFIED:** Whether Login Kit accepts any localhost / `.local` HTTPS redirect — Frank must try in portal.  
2. **UNVERIFIED:** Exact “test URL without verification” string for PULL_FROM_URL.  
3. **UNVERIFIED:** Whether `brand_content_toggle` / `brand_organic_toggle` must always be present on DIRECT_POST photo init.  
4. **UNVERIFIED:** Webhook signature verification algorithm and secret field name in developer portal.  
5. **UNVERIFIED:** Sandbox vs production client key differences and whether unaudited caps differ in sandbox.  
6. **UNVERIFIED:** Exact numeric daily post cap per creator (docs say “typically around 15”).  
7. **UNVERIFIED:** Whether Tomato’s “approval-first multi-account scheduler for brands” narrative will pass intended-use review (guidelines hostile to internal multi-account upload utilities). Frank should draft review copy carefully.  
8. Repo `/srv/apps/tomato` not mounted in this research container — no code cross-check against existing scaffold.  
9. General platform rate-limit doc page 404’d at `/docs/en/rate-limit`; per-endpoint limits used instead.

## confidence

- **level:** high for core posting/OAuth/audit private-only path  
- **rationale:** All load-bearing posting, scope, UX, and restriction claims extracted from current official TikTok docs pages (Aug 2026 stamps) via live browser, not secondary blogs. Residual uncertainty is portal-only config (localhost, test URL, webhook crypto) and review narrative fit.

## recommended_next_actions

1. **Su / Tomato eng:** Implement OAuth + creator_info + DIRECT_POST photo (SELF_ONLY) + MEDIA_UPLOAD fallback + status poller + capability probe enum storage using env names above.  
2. **Frank:** Create TT4D app; enable Login Kit + Content Posting + Direct Post; request `video.publish` + `video.upload`; register HTTPS tunnel redirect; verify media URL property; paste keys only into env.  
3. **Frank:** Use a **private** TikTok account for first posts; expect `private_only` capability.  
4. **Product:** Build export UI matching Content Sharing Guidelines before audit (privacy no-default, commercial disclosure, music consent).  
5. **Frank:** After private posts work, record sandbox demo video and submit audit; only then flip product copy to public scheduling.  
6. **Optional:** Configure webhooks for `post.publish.*` to reduce worker polling.  
7. **Huly TOM-1:** Attach this doc path as research artifact for TikTok spike acceptance criteria.

---

## Appendix A — Endpoint cheat sheet

| Action | Method | Path |
|--------|--------|------|
| OAuth authorize (browser) | GET | `https://www.tiktok.com/v2/auth/authorize/` |
| Token / refresh | POST | `https://open.tiktokapis.com/v2/oauth/token/` |
| Creator info | POST | `/v2/post/publish/creator_info/query/` |
| Direct video init | POST | `/v2/post/publish/video/init/` |
| Inbox video init | POST | `/v2/post/publish/inbox/video/init/` |
| Photo direct or upload | POST | `/v2/post/publish/content/init/` |
| Status | POST | `/v2/post/publish/status/fetch/` |
| Cancel | POST | `/v2/post/publish/cancel/` |
| File bytes | PUT | `{upload_url}` with Content-Range |

Host for Open API: `open.tiktokapis.com`.

## Appendix B — FILE_UPLOAD chunk rules (video)

- Chunk size 5–64 MB (final chunk may exceed chunk_size up to 128 MB for trailing bytes)  
- Videos &lt; 5 MB: single whole upload, chunk_size = file size  
- Videos &gt; 64 MB: multi-chunk  
- 1–1000 chunks; sequential  
- Partial responses 206; final 201  
Source: Media Transfer Guide

## Appendix C — Spike acceptance criteria for Tomato T-TikTok

- [ ] Env vars documented in `.env.example` (names only)  
- [ ] OAuth connect + refresh works against registered HTTPS callback  
- [ ] creator_info drives privacy UI with no default  
- [ ] Direct photo SELF_ONLY succeeds on private account when unaudited  
- [ ] PUBLIC attempt surfaces `unaudited_client_can_only_post_to_private_accounts` or is withheld in UI  
- [ ] MEDIA_UPLOAD path works with inbox messaging  
- [ ] Status reaches PUBLISH_COMPLETE or FAILED with logged fail_reason  
- [ ] Capability proof row written: expected `private_only` or `upload_fallback` pre-audit  
- [ ] Domain verification documented for Frank’s media host  

---

*End of report. No secrets included.*
