# Tomato env / public-URL / secrets contract

**Task:** `t_2d125f8e`  
**Product:** Tomato — approval-first social command center (TOM-1)  
**Date:** 2026-09-02  
**Author:** Geraldo v2.2 (GIU)  
**Audience:** implementers (Su / next coding pass) + Frank (paste-keys checklist)

---

## objective

Define the exact local/public runtime contract Tomato should use so Frank can wake up, paste API keys into `.env`, and run OAuth + publish against Meta (first) and TikTok (spike), with:

- stable public HTTPS for OAuth callbacks
- publicly fetchable media URLs (or an explicit upload alternative)
- encrypted-at-rest provider tokens in SQLite
- fail-loud validation when required keys are missing

## summary

**Use one permanent Cloudflare Tunnel hostname as the single public origin for both OAuth callbacks and media pulls.** Do not rely on `localhost` callbacks for TikTok; do not rely on `*.trycloudflare.com` quick tunnels for anything that must stay registered in provider consoles.

Canonical split of URLs:

| Concept | Env | Role |
|---|---|---|
| Operator-facing origin | `TOMATO_APP_URL` | What Frank opens in the browser (usually same as public) |
| Provider-facing origin | `TOMATO_PUBLIC_BASE_URL` | Exact HTTPS origin registered with Meta/TikTok and embedded in media URLs |
| Local bind | `TOMATO_HOST` + `TOMATO_PORT` | Where Next.js listens (tunnel target) |

**Recommendation for this server:** named Cloudflare Tunnel route  
`https://tomato.<your-cloudflare-zone>` → `http://127.0.0.1:3737` (or whatever port Tomato binds).  
Set **both** `TOMATO_APP_URL` and `TOMATO_PUBLIC_BASE_URL` to that HTTPS origin for MVP. Path strategy (stable, never change once registered):

```
/api/oauth/meta/callback
/api/oauth/tiktok/callback
/media/:assetId                 # public, unauthenticated GET of approved publish blobs
/api/health
```

Provider callbacks and provider media fetches **can and should share one public hostname** for a single-operator MVP. Separate media hostname is optional later (R2/CDN), not required for T0–Meta first.

Token storage: **AES-256-GCM** with a 32-byte key in env (`TOMATO_TOKEN_ENCRYPTION_KEY`, base64). Never store provider access/refresh tokens plaintext in SQLite or repo. Separate `TOMATO_SESSION_SECRET` for operator session cookies / passcode hashing material.

Boot validation: Zod (or equivalent) schema that **throws a single multi-line error listing every missing/invalid var by name** before the server accepts traffic. Optional provider blocks are gated by feature flags so Meta-only bring-up does not require TikTok keys.

## plan (executed)

1. Read task + TOM-1 pointer; attempt local repo `/srv/apps/tomato` (not visible from this worker).
2. Extract Meta Login + Instagram content-publishing URL requirements.
3. Extract TikTok Login Kit redirect rules + Content Posting media transfer (PULL_FROM_URL + FILE_UPLOAD).
4. Extract Cloudflare Tunnel publish-hostname pattern; compare Postiz self-host env patterns.
5. Synthesize env names, validation behavior, tunnel checklist, encryption recipe.

---

## findings

### 1. Recommended env var names + validation schema

#### 1.1 Core runtime (always required)

| Env var | Type | Example / generator | Notes |
|---|---|---|---|
| `NODE_ENV` | `development` \| `production` \| `test` | `development` | |
| `TOMATO_HOST` | string | `127.0.0.1` | Bind address for Next |
| `TOMATO_PORT` | int 1–65535 | `3737` | Tunnel service URL uses this |
| `TOMATO_APP_URL` | absolute URL, no trailing slash | `https://tomato.example.com` | Browser origin Frank uses |
| `TOMATO_PUBLIC_BASE_URL` | absolute **https** URL, no trailing slash | `https://tomato.example.com` | OAuth redirect base + media URL base |
| `TOMATO_SQLITE_PATH` | absolute path | `/var/lib/tomato/tomato.sqlite` | Outside repo; create parent dir on boot |
| `TOMATO_MEDIA_DIR` | absolute path | `/var/lib/tomato/media` | Local blob store for MVP |
| `TOMATO_MEDIA_PUBLIC_PATH` | path prefix | `/media` | Mounted public route; join with public base |
| `TOMATO_SESSION_SECRET` | string ≥ 32 chars | `openssl rand -base64 48` | Cookie signing / CSRF / session |
| `TOMATO_OPERATOR_PASSCODE_HASH` | string | argon2/bcrypt hash | **Not** plaintext passcode in env for prod; see §4 |
| `TOMATO_TOKEN_ENCRYPTION_KEY` | base64, decodes to **exactly 32 bytes** | `openssl rand -base64 32` | AES-256-GCM key |
| `TOMATO_TRUST_PROXY` | bool | `true` when behind Cloudflare Tunnel | So `x-forwarded-proto=https` makes cookies Secure |

**Derived (do not set separately unless override needed):**

- `mediaPublicBase = TOMATO_PUBLIC_BASE_URL + TOMATO_MEDIA_PUBLIC_PATH`
- Meta callback = `TOMATO_PUBLIC_BASE_URL + /api/oauth/meta/callback`
- TikTok callback = `TOMATO_PUBLIC_BASE_URL + /api/oauth/tiktok/callback`

Validation rules:

- `TOMATO_APP_URL` and `TOMATO_PUBLIC_BASE_URL` must parse as absolute URLs.
- In any mode where Meta or TikTok publish is enabled: `TOMATO_PUBLIC_BASE_URL` **must** be `https:` (not `http:`).
- Reject trailing `/` on base URLs (normalize or fail — prefer fail loud).
- `TOMATO_TOKEN_ENCRYPTION_KEY`: base64 decode length === 32, else fail with “expected 32-byte key (openssl rand -base64 32)”.
- Paths: writable at boot (touch test file or open SQLite); fail with path + errno.

#### 1.2 Draft brain — Nous API

| Env var | Required when | Notes |
|---|---|---|
| `NOUS_API_KEY` | `TOMATO_DRAFT_PROVIDER=nous` (default) | Server-only; never `NEXT_PUBLIC_` |
| `NOUS_API_BASE_URL` | optional | Default official Nous OpenAI-compatible base if documented in app |
| `NOUS_DRAFT_MODEL` | optional | e.g. model id string |
| `TOMATO_DRAFT_PROVIDER` | optional | `nous` \| `none` — `none` skips key requirement |

#### 1.3 Vision tagger (pluggable)

| Env var | Required when | Notes |
|---|---|---|
| `TOMATO_VISION_PROVIDER` | always (default `none`) | `none` \| `openai` \| `local` \| `custom` |
| `TOMATO_VISION_API_KEY` | provider ≠ `none` and needs key | |
| `TOMATO_VISION_BASE_URL` | `custom` / local OpenAI-compatible | |
| `TOMATO_VISION_MODEL` | provider ≠ `none` | |

#### 1.4 Meta OAuth / publish

| Env var | Required when | Notes |
|---|---|---|
| `TOMATO_META_ENABLED` | flag, default `true` for Meta-first MVP | |
| `META_APP_ID` | Meta enabled | Same app can cover FB Page + IG professional |
| `META_APP_SECRET` | Meta enabled | Server-only; used for code exchange + optional `appsecret_proof` |
| `META_GRAPH_VERSION` | optional | e.g. `v22.0` / `v26.0` — pin explicitly |
| `META_OAUTH_SCOPES` | optional override | Default set below |
| `META_CONFIG_ID` | if using Login for Business config | optional |
| `META_DEAUTHORIZE_PATH` | optional | `/api/oauth/meta/deauthorize` |
| `META_DATA_DELETION_PATH` | optional | `/api/oauth/meta/data-deletion` |

**Default scopes (Meta-first MVP, Facebook Login for Business path):**

- Page publish: `pages_show_list`, `pages_manage_posts`, `pages_read_engagement`
- IG content (FB login path): `instagram_basic`, `instagram_content_publish`, `pages_read_engagement`
- Add `business_management` if Page access goes through BM

Exact scope list should match the product choice: Instagram API with Facebook Login vs Instagram Login. TOM-1 targets Facebook Page + Instagram professional — prefer **Facebook Login for Business** single app (`META_APP_ID` / `META_APP_SECRET`) rather than a second Instagram-only app for MVP.

**Registered redirect URI (exact match, Strict Mode):**

```
{TOMATO_PUBLIC_BASE_URL}/api/oauth/meta/callback
```

Also set Meta App Domains to the hostname only (e.g. `tomato.example.com`).

#### 1.5 TikTok OAuth / publish (spike)

| Env var | Required when | Notes |
|---|---|---|
| `TOMATO_TIKTOK_ENABLED` | flag, default `false` until spike | |
| `TIKTOK_CLIENT_KEY` | TikTok enabled | TikTok “client key” (sometimes called client id) |
| `TIKTOK_CLIENT_SECRET` | TikTok enabled | Server-only |
| `TIKTOK_OAUTH_SCOPES` | optional | default `user.info.basic,video.upload,video.publish` (tune to audit status) |
| `TIKTOK_MEDIA_TRANSFER` | optional | `pull_from_url` \| `file_upload` (default `file_upload` for fewer moving parts; see §3) |

**Registered redirect URI (https, absolute, static, no query/fragment):**

```
{TOMATO_PUBLIC_BASE_URL}/api/oauth/tiktok/callback
```

TikTok allows max 10 redirect URIs; each < 512 chars; **must begin with `https`**.

#### 1.6 Zod-style schema (implementation sketch)

```ts
// lib/env.ts — fail at import / boot, not on first request
import { z } from "zod";

const urlNoSlash = z
  .string()
  .url()
  .refine((u) => !u.endsWith("/"), "must not end with /");

const base64Key32 = z.string().refine((s) => {
  try {
    return Buffer.from(s, "base64").length === 32;
  } catch {
    return false;
  }
}, "TOMATO_TOKEN_ENCRYPTION_KEY must be base64 for exactly 32 raw bytes");

const core = z.object({
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  TOMATO_HOST: z.string().default("127.0.0.1"),
  TOMATO_PORT: z.coerce.number().int().positive().default(3737),
  TOMATO_APP_URL: urlNoSlash,
  TOMATO_PUBLIC_BASE_URL: urlNoSlash,
  TOMATO_SQLITE_PATH: z.string().min(1),
  TOMATO_MEDIA_DIR: z.string().min(1),
  TOMATO_MEDIA_PUBLIC_PATH: z
    .string()
    .regex(/^\/[a-z0-9/_-]*$/i)
    .default("/media"),
  TOMATO_SESSION_SECRET: z.string().min(32),
  TOMATO_OPERATOR_PASSCODE_HASH: z.string().min(20),
  TOMATO_TOKEN_ENCRYPTION_KEY: base64Key32,
  TOMATO_TRUST_PROXY: z
    .enum(["true", "false"])
    .default("true")
    .transform((v) => v === "true"),
  TOMATO_DRAFT_PROVIDER: z.enum(["nous", "none"]).default("nous"),
  NOUS_API_KEY: z.string().optional(),
  NOUS_API_BASE_URL: z.string().url().optional(),
  NOUS_DRAFT_MODEL: z.string().optional(),
  TOMATO_VISION_PROVIDER: z.enum(["none", "openai", "local", "custom"]).default("none"),
  TOMATO_VISION_API_KEY: z.string().optional(),
  TOMATO_VISION_BASE_URL: z.string().url().optional(),
  TOMATO_VISION_MODEL: z.string().optional(),
  TOMATO_META_ENABLED: z.enum(["true", "false"]).default("true").transform((v) => v === "true"),
  META_APP_ID: z.string().optional(),
  META_APP_SECRET: z.string().optional(),
  META_GRAPH_VERSION: z.string().default("v22.0"),
  TOMATO_TIKTOK_ENABLED: z.enum(["true", "false"]).default("false").transform((v) => v === "true"),
  TIKTOK_CLIENT_KEY: z.string().optional(),
  TIKTOK_CLIENT_SECRET: z.string().optional(),
  TIKTOK_MEDIA_TRANSFER: z.enum(["pull_from_url", "file_upload"]).default("file_upload"),
});

export type TomatoEnv = z.infer<typeof core> & {
  metaCallbackUrl: string;
  tiktokCallbackUrl: string;
  mediaPublicBase: string;
};

export function loadEnv(raw = process.env): TomatoEnv {
  const parsed = core.safeParse(raw);
  if (!parsed.success) {
    const lines = parsed.error.issues.map(
      (i) => `  - ${i.path.join(".") || "(root)"}: ${i.message}`,
    );
    throw new Error(
      `Tomato env invalid — fix .env and restart:\n${lines.join("\n")}`,
    );
  }
  const e = parsed.data;
  const missing: string[] = [];

  if (e.TOMATO_PUBLIC_BASE_URL.startsWith("http://") && (e.TOMATO_META_ENABLED || e.TOMATO_TIKTOK_ENABLED)) {
    missing.push("TOMATO_PUBLIC_BASE_URL must be https when Meta or TikTok is enabled");
  }
  if (e.TOMATO_DRAFT_PROVIDER === "nous" && !e.NOUS_API_KEY) missing.push("NOUS_API_KEY (draft provider=nous)");
  if (e.TOMATO_VISION_PROVIDER !== "none" && !e.TOMATO_VISION_API_KEY && e.TOMATO_VISION_PROVIDER !== "local") {
    missing.push("TOMATO_VISION_API_KEY");
  }
  if (e.TOMATO_META_ENABLED) {
    if (!e.META_APP_ID) missing.push("META_APP_ID");
    if (!e.META_APP_SECRET) missing.push("META_APP_SECRET");
  }
  if (e.TOMATO_TIKTOK_ENABLED) {
    if (!e.TIKTOK_CLIENT_KEY) missing.push("TIKTOK_CLIENT_KEY");
    if (!e.TIKTOK_CLIENT_SECRET) missing.push("TIKTOK_CLIENT_SECRET");
  }
  if (missing.length) {
    throw new Error(
      `Tomato missing required secrets for enabled features:\n` +
        missing.map((m) => `  - ${m}`).join("\n") +
        `\n\nCopy .env.example → .env and fill the lines above. Do not commit .env.`,
    );
  }

  return {
    ...e,
    metaCallbackUrl: `${e.TOMATO_PUBLIC_BASE_URL}/api/oauth/meta/callback`,
    tiktokCallbackUrl: `${e.TOMATO_PUBLIC_BASE_URL}/api/oauth/tiktok/callback`,
    mediaPublicBase: `${e.TOMATO_PUBLIC_BASE_URL}${e.TOMATO_MEDIA_PUBLIC_PATH}`,
  };
}
```

**Runtime failure behavior (required UX):**

1. Parse all vars once at process start (`instrumentation.ts` or `server.ts` before listen).
2. On failure: print **red multi-line block** to stderr, exit code `1`, do not bind port.
3. Include a one-line hint: `See docs/tomato/research-env-contract-t_2d125f8e.md` or README section “Frank keys”.
4. Optional: `GET /api/health` returns `{ ok: true, publicBase, metaEnabled, tiktokEnabled }` with **no secrets**; if env failed, process is down so health is irrelevant.
5. Never log secret values — only var **names**.

---

### 2. Exposing local Next.js for Meta/TikTok on this server

#### 2.1 Preferred: permanent Cloudflare Tunnel (named hostname)

Already used on this host for other apps; matches privacy/self-host preference (origin stays private; CF terminates TLS).

**Steps (ops checklist):**

1. Cloudflare dashboard → Networking → Tunnels → existing tunnel (or create one).
2. Add **Published application** route:
   - Hostname: `tomato.<zone>` (stable; pick once)
   - Service URL: `http://127.0.0.1:3737` (Tomato bind)
3. DNS CNAME `tomato` → `<tunnel-id>.cfargotunnel.com` (proxied).
4. Set env:
   ```
   TOMATO_APP_URL=https://tomato.<zone>
   TOMATO_PUBLIC_BASE_URL=https://tomato.<zone>
   TOMATO_TRUST_PROXY=true
   ```
5. Register the **exact** callback URLs in Meta + TikTok consoles.
6. Frank always opens `https://tomato.<zone>` (not bare localhost) so cookies and OAuth land on the same host.

Official pattern: map public hostname → local service URL via tunnel ingress; optional quick tunnels exist but are for throwaway tests only (random `trycloudflare.com`, 200 concurrent limit, not for registered OAuth).

#### 2.2 Alternatives (ranked)

| Option | Use when | Drawback |
|---|---|---|
| Named CF Tunnel | **Default on this server** | Needs CF zone |
| CF quick tunnel `cloudflared tunnel --url http://127.0.0.1:3737` | 10-minute smoke test | Hostname changes → re-register OAuth every time |
| Tailscale Funnel | Prefer mesh over CF | Still public HTTPS; different DNS story |
| ngrok reserved domain | No CF | Extra vendor; paid for stable domain |
| Public bind :443 + Let’s Encrypt | Classic | Opens origin; fights “no public IP” preference |
| `next dev --experimental-https` localhost | Pure local UI only | Meta may accept `https://localhost` in dev; **TikTok web redirect still needs real https URI registered**; provider media fetch cannot reach localhost |

**Do not** use plain `http://localhost:…` as the production-style callback for TikTok Login Kit web.

#### 2.3 Exact hostname / path strategy

```
https://tomato.<zone>
├── /                         # operator UI (passcode gate)
├── /api/health
├── /api/oauth/meta/start
├── /api/oauth/meta/callback          ← Meta Valid OAuth Redirect URI
├── /api/oauth/meta/deauthorize       ← optional Meta deauth callback
├── /api/oauth/meta/data-deletion     ← optional data deletion callback
├── /api/oauth/tiktok/start
├── /api/oauth/tiktok/callback        ← TikTok Login Kit redirect URI
├── /media/:assetId                   ← public GET (publish pulls)
└── /api/...                          # authenticated operator APIs
```

**Path stability rule:** once a callback is registered with Meta/TikTok, treat it as frozen. Version behind query/`state`, not path churn.

**Media URL shape for providers:**

```
https://tomato.<zone>/media/<assetId>
```

- ASCII-only path (Meta docs strongly recommend US-ASCII URLs for IG media).
- No required auth cookies for GET (providers fetch server-to-server).
- Optional: short-lived signed query `?exp=&sig=` if you want unlisted URLs; then Meta/TikTok must receive the **full** signed URL. TikTok PULL_FROM_URL forbids redirects and requires ownership of prefix/domain — signed query on a verified prefix is OK if the prefix is verified without requiring static query (TikTok redirect URIs ban params; media URLs are different). Prefer **unguessable asset IDs** + rate limits for MVP simplicity.

---

### 3. One public hostname vs separate for callbacks vs media

| Concern | Shared hostname | Split (`tomato.` + `media.`) |
|---|---|---|
| OAuth redirect registration | One host | Still one host for app |
| Meta media cURL | Same origin OK | Fine |
| TikTok PULL_FROM_URL ownership | Verify one domain/prefix once | Verify media domain separately |
| Cookie scope | Natural | N/A for media |
| Blast radius / CDN | App + media coupled | Media can go R2/CDN later |
| MVP complexity | **Lower** | Higher |

**Verdict:** **Share one public hostname** for callbacks and media in the single-operator MVP.

**TikTok nuance:** if using `PULL_FROM_URL`, verify ownership of either:

- Domain `tomato.<zone>`, or  
- URL prefix `https://tomato.<zone>/media/`

in the TikTok developer portal URL properties widget. Media URL must be **https**, must **not redirect**, and must stay reachable for up to ~1 hour after pull starts.

**Mitigation if ownership is annoying:** set `TIKTOK_MEDIA_TRANSFER=file_upload` so Tomato PUTs bytes to TikTok’s `upload_url` and **does not need TikTok to fetch Tomato media**. Meta Instagram container creation still wants `image_url` / `video_url` on a public server **or** use Meta resumable upload (`upload_type=resumable` + `rupload.facebook.com`) to push bytes and reduce public-media dependency for large video.

**MVP practical combo:**

1. Always expose `/media/:id` publicly (Meta image/simple video path is easiest).
2. Meta: start with public URL containers; add resumable for large Reels if needed.
3. TikTok spike: prefer `FILE_UPLOAD` first; add `PULL_FROM_URL` after domain verification.

---

### 4. Token encryption key generation / storage (single-operator MVP)

#### 4.1 Goals

- No plaintext provider tokens in SQLite
- No secrets in git
- Frank can rotate by generating a new key (accepts re-link channels) or keep a backup of the key offline

#### 4.2 Recipe

```bash
# one-time on server (store only in .env, mode 600)
openssl rand -base64 32   # → TOMATO_TOKEN_ENCRYPTION_KEY
openssl rand -base64 48   # → TOMATO_SESSION_SECRET
```

**Algorithm:** AES-256-GCM (Node `crypto`).

Stored blob format (single TEXT column `token_blob`):

```
v1:<iv_b64>:<ciphertext_b64>:<tag_b64>
```

Encrypt JSON `{ access_token, refresh_token?, expires_at, token_type?, scopes? }`.

```ts
import { createCipheriv, createDecipheriv, randomBytes } from "node:crypto";

const VERSION = "v1";

export function seal(plain: string, keyB64: string): string {
  const key = Buffer.from(keyB64, "base64");
  const iv = randomBytes(12);
  const cipher = createCipheriv("aes-256-gcm", key, iv);
  const ct = Buffer.concat([cipher.update(plain, "utf8"), cipher.final()]);
  const tag = cipher.getAuthTag();
  return [VERSION, iv.toString("base64"), ct.toString("base64"), tag.toString("base64")].join(":");
}

export function open(blob: string, keyB64: string): string {
  const [v, ivB64, ctB64, tagB64] = blob.split(":");
  if (v !== VERSION) throw new Error("unknown token blob version");
  const key = Buffer.from(keyB64, "base64");
  const decipher = createDecipheriv("aes-256-gcm", key, Buffer.from(ivB64, "base64"));
  decipher.setAuthTag(Buffer.from(tagB64, "base64"));
  return Buffer.concat([
    decipher.update(Buffer.from(ctB64, "base64")),
    decipher.final(),
  ]).toString("utf8");
}
```

#### 4.3 Operator passcode

- Do **not** put raw passcode in `.env` long-term.
- Bootstrap: `TOMATO_OPERATOR_PASSCODE` only accepted on first boot if hash empty → hash with argon2id → write hash into SQLite settings + require removing plaintext env; **or** ship `scripts/hash-passcode.mjs` and set `TOMATO_OPERATOR_PASSCODE_HASH` only.
- Session cookie: `httpOnly`, `secure` when public URL is https, `sameSite: 'lax'`, signed with `TOMATO_SESSION_SECRET`.

#### 4.4 What stays in env vs DB

| Secret | Env | DB |
|---|---|---|
| Meta app secret | yes | no |
| TikTok client secret | yes | no |
| Nous / vision keys | yes | no |
| Token encryption key | yes | no |
| Session secret | yes | no |
| User access/refresh tokens | no | encrypted blob |
| Operator passcode | hash in env or DB | prefer DB after bootstrap |

#### 4.5 Meta app secret handling

Meta: never ship App Secret to the browser; exchange `code` server-side; optionally enable **Require App Secret** / `appsecret_proof` for Graph calls from Tomato backend only.

---

### 5. `.env.example` shape

```bash
# =============================================================================
# Tomato — copy to .env and fill. Never commit .env.
# Generate secrets:
#   openssl rand -base64 32   # TOMATO_TOKEN_ENCRYPTION_KEY
#   openssl rand -base64 48   # TOMATO_SESSION_SECRET
# Public URL must be the Cloudflare Tunnel hostname Frank actually opens.
# =============================================================================

NODE_ENV=development

# --- Bind (tunnel targets this) ---
TOMATO_HOST=127.0.0.1
TOMATO_PORT=3737
TOMATO_TRUST_PROXY=true

# --- Public / app URLs (no trailing slash; https required for providers) ---
TOMATO_APP_URL=https://tomato.example.com
TOMATO_PUBLIC_BASE_URL=https://tomato.example.com

# --- Local data (outside git) ---
TOMATO_SQLITE_PATH=/var/lib/tomato/tomato.sqlite
TOMATO_MEDIA_DIR=/var/lib/tomato/media
TOMATO_MEDIA_PUBLIC_PATH=/media

# --- Operator gate + crypto ---
TOMATO_SESSION_SECRET=REPLACE_WITH_openssl_rand_base64_48
TOMATO_OPERATOR_PASSCODE_HASH=REPLACE_WITH_argon2_hash
TOMATO_TOKEN_ENCRYPTION_KEY=REPLACE_WITH_openssl_rand_base64_32

# --- Draft brain (Nous) ---
TOMATO_DRAFT_PROVIDER=nous
NOUS_API_KEY=
# NOUS_API_BASE_URL=
# NOUS_DRAFT_MODEL=

# --- Vision tagger (pluggable) ---
TOMATO_VISION_PROVIDER=none
# TOMATO_VISION_API_KEY=
# TOMATO_VISION_BASE_URL=
# TOMATO_VISION_MODEL=

# --- Meta (Facebook Page + Instagram professional) ---
TOMATO_META_ENABLED=true
META_APP_ID=
META_APP_SECRET=
META_GRAPH_VERSION=v22.0
# Register EXACTLY: ${TOMATO_PUBLIC_BASE_URL}/api/oauth/meta/callback

# --- TikTok (spike; leave disabled until keys ready) ---
TOMATO_TIKTOK_ENABLED=false
TIKTOK_CLIENT_KEY=
TIKTOK_CLIENT_SECRET=
TIKTOK_MEDIA_TRANSFER=file_upload
# Register EXACTLY: ${TOMATO_PUBLIC_BASE_URL}/api/oauth/tiktok/callback
# If pull_from_url: verify https://tomato.example.com/media/ (or domain) in TikTok URL properties
```

---

### 6. Local-dev caveats checklist

| Topic | Rule |
|---|---|
| **Localhost callbacks** | Meta: often workable with `http://localhost:…` **in Development mode** only; Live + Enforce HTTPS need real https. TikTok web Login Kit: redirect URIs **must be https**, absolute, static. **Use tunnel for both.** |
| **HTTPS** | Meta requires HTTPS for OAuth (enforced for apps; “Use HTTPS” checklist). TikTok authorization page is https-only; redirects must be https. |
| **Strict redirect match** | Meta Strict Mode: full URI exact match (state param ignored). TikTok: no query/fragment on registered URI. |
| **Mixed content** | Operator UI must be served on the same https origin as API if Secure cookies are set. Don’t open `http://127.0.0.1:3737` while session cookie is `Secure` for tunnel host. |
| **Cookies** | `secure: true` when `TOMATO_PUBLIC_BASE_URL` is https; `httpOnly: true`; `sameSite: 'lax'`; path `/`. With tunnel, set `TOMATO_TRUST_PROXY=true` so framework sees https. |
| **CSRF on OAuth** | Meta + TikTok both require random `state`; store server-side (encrypted cookie or SQLite) and verify on callback. |
| **Provider media fetch** | Localhost media URLs **will fail** — Meta cURLs your `image_url`/`video_url`; TikTok PULL_FROM_URL downloads your URL. |
| **TikTok audit** | Unaudited apps: Direct Post often forced `SELF_ONLY`, limited users — spike expectation, not full prod. |
| **Meta App Mode** | Development: only roles/testers see some content; Live needed for broader visibility (Postiz ops note). |
| **ASCII media URLs** | Prefer hex/uuid asset ids; avoid spaces and non-ASCII. |
| **No trailing slash drift** | `https://host` vs `https://host/` breaks exact redirect match — normalize one way in env validation. |
| **Quick tunnels** | OK for disposable webhook tests; **not** for registered OAuth clients. |
| **Secrets in client bundles** | Prefix only non-secrets with `NEXT_PUBLIC_`. App secrets, tokens, encryption key: server-only. |
| **SQLite + media paths** | Keep under `/var/lib/tomato` (or `$HOME/var/tomato`), not inside Next project dir / git. |
| **Firewall** | Tomato binds `127.0.0.1` only; Cloudflare connects outbound via cloudflared. Do not publish 3737 on LAN unless intentional. |

#### Frank “paste keys and go” order

1. `cp .env.example .env && chmod 600 .env`
2. Generate session + encryption secrets; set passcode hash.
3. Confirm tunnel hostname healthy → set both base URLs.
4. Paste `NOUS_API_KEY`.
5. Paste `META_APP_ID` / `META_APP_SECRET`; register callback in Meta; enable Meta product.
6. `pnpm dev` / systemd — if boot errors, fix listed env names.
7. Open **tunnel URL**, unlock with passcode, connect Meta.
8. (Later) TikTok keys + enable flag + register callback (+ ownership if pull).

---

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | Instagram content publishing pulls media by cURL from a publicly accessible URL (`image_url` / `video_url`) unless using resumable upload paths. | E1, E2 | high | supported |
| C2 | Meta Facebook Login requires registering Valid OAuth Redirect URIs; Strict Mode exact-match; HTTPS required for modern apps. | E3, E4 | high | supported |
| C3 | TikTok Login Kit web redirect URIs must be https, absolute, static (no query/fragment), max 10, each < 512 chars. | E5 | high | supported |
| C4 | TikTok `PULL_FROM_URL` requires developer-owned verified domain or URL prefix; https; no redirects; available for download window. | E6 | high | supported |
| C5 | TikTok also supports `FILE_UPLOAD` chunked PUT to TikTok `upload_url`, avoiding inbound pull to Tomato. | E6, E7 | high | supported |
| C6 | Cloudflare Tunnel maps a public hostname to a local service URL; quick tunnels are ephemeral/dev-only. | E8 | high | supported |
| C7 | Single shared public hostname for OAuth + media is sufficient and preferred for single-operator MVP. | E1, E5, E6 + synthesis | medium | partial |
| C8 | AES-256-GCM with 32-byte env key is appropriate for encrypting OAuth tokens at rest in SQLite for this threat model. | industry standard + synthesis | medium | partial |
| C9 | Postiz-style env split (`FRONTEND_URL` exact access URL + provider `*_CLIENT_*` secrets) is a proven pattern for self-hosted social schedulers. | E9, E10 | high | supported |
| C10 | Tomato repo at `/srv/apps/tomato` was not readable from this worker; scaffold-specific names may need alignment on implement. | local probe | high | supported (gap) |

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | Meta Instagram Content Publishing | https://developers.facebook.com/docs/instagram-platform/content-publishing/ | web_extract 2026-09-02 | “We cURL media used in publishing attempts, so the media must be hosted on a publicly accessible server at the time of the attempt.” |
| E2 | Meta IG User Media reference | https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/reference/ig-user/media/ | web_extract 2026-09-02 | Create container via POST `/{ig-user-id}/media`; image/video specs; strongly recommend US-ASCII URLs. |
| E3 | Meta Manual Login Flow | https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow/ | web_extract 2026-09-02 | `redirect_uri` required; must match Valid OAuth redirect URIs in App Dashboard. |
| E4 | Meta Login Security | https://developers.facebook.com/documentation/facebook-login/security | web_extract 2026-09-02 | Strict Mode exact match; Use HTTPS (required); App Secret server-only; state param CSRF; Enforce HTTPS for redirects. |
| E5 | TikTok Login Kit Web | https://developers.tiktok.com/docs/en/login-kit-web | browser 2026-09-02 | Redirect URI https absolute static; max 10; no params; no fragment; state CSRF; client secret server-side. |
| E6 | TikTok Media Transfer Guide | https://developers.tiktok.com/docs/en/content-posting-api-media-transfer-guide | browser 2026-09-02 | FILE_UPLOAD vs PULL_FROM_URL; ownership verification domain/prefix; https no redirect; 1h download window. |
| E7 | Postproxy TikTok API guide | https://postproxy.dev/blog/how-to-post-to-tiktok-via-api/ | web_extract 2026-09-02 | init with PULL_FROM_URL or FILE_UPLOAD; scopes; audit/private restrictions. |
| E8 | Cloudflare Tunnel setup | https://developers.cloudflare.com/tunnel/setup/ | web_extract 2026-09-02 | Published application hostname → local service; quick tunnel trycloudflare.com testing only. |
| E9 | Postiz .env.example | https://raw.githubusercontent.com/gitroomhq/postiz-app/main/.env.example | web_extract 2026-09-02 | `FRONTEND_URL` must match access URL; `FACEBOOK_APP_*`, `TIKTOK_CLIENT_*`, `JWT_SECRET`, storage paths. |
| E10 | Postiz TikTok provider docs | https://docs.postiz.com/self-host/providers/tiktok | web_extract 2026-09-02 | HTTPS required; pull_from_url needs public media; localhost fails; verify media domain. |
| E11 | Postiz Facebook provider docs | https://docs.postiz.com/self-host/providers/facebook | web_extract 2026-09-02 | Redirect URI patterns; App ID/Secret env; Live vs Development visibility. |
| E12 | Huly TOM-1 | search_huly TOM-1 | MCP 2026-09-02 | Tomato Meta+TikTok approval-first MVP; blueprint path noted (not readable here). |
| E13 | Next.js cookies | https://nextjs.org/docs/app/api-reference/functions/cookies | web_extract 2026-09-02 | `secure`, `httpOnly`, `sameSite` options for session cookies. |

## gaps

1. **`/srv/apps/tomato` not mounted/visible** in this Docker worker — could not align names to existing scaffold code or any in-repo `.env.example`. Implementer should diff this contract against T0 code and rename only if scaffold already chose different identifiers (prefer this contract if greenfield).
2. **TOM-1 full issue body / blueprint file** (`/srv/app-data/huly-artifacts/su-v2/20260901-social-command-center-blueprint.md`) not readable from worker; hostname choice `tomato.<zone>` is a **recommendation**, not an existing DNS fact.
3. **Exact Cloudflare zone** used on this server not discovered from local configs in the sandbox — Frank/ops must pick the real hostname.
4. **Nous API base URL / model ids** not locked — leave optional overrides.
5. **Instagram Login vs Facebook Login** product path: contract assumes Facebook Login for Business single app; if product picks Instagram-user login, add `INSTAGRAM_APP_ID` / `INSTAGRAM_APP_SECRET` pair (Postiz standalone pattern).
6. **TikTok app audit** timeline and UX compliance UI requirements are out of scope for env contract but block public Direct Post.

## confidence

- **level:** high on provider URL/HTTPS/media-fetch rules; medium on Tomato-specific naming (no repo) and server DNS specifics.
- **rationale:** Core constraints come from official Meta/TikTok/Cloudflare docs extracted this session. Naming and path layout are design recommendations consistent with Postiz and single-operator ops, marked as such where not provider-mandated.

## recommended_next_actions

1. **Ops:** create CF Tunnel hostname `tomato.<zone>` → `http://127.0.0.1:3737`; put HTTPS origin in `.env`.
2. **Code:** add `lib/env.ts` boot validation + `.env.example` exactly as above; fail-fast message listing missing keys by name.
3. **Code:** implement AES-GCM seal/open for connection tokens; media route `GET /media/:id` from `TOMATO_MEDIA_DIR`.
4. **Code:** OAuth routes with frozen paths; export `metaCallbackUrl` / `tiktokCallbackUrl` on a debug “connection setup” screen (no secrets) so Frank can copy-paste into provider consoles.
5. **Meta first:** register one redirect URI; Development mode + tester roles; public `/media` for image publish test.
6. **TikTok spike:** `TOMATO_TIKTOK_ENABLED=true`, `TIKTOK_MEDIA_TRANSFER=file_upload` to avoid ownership ceremony initially.
7. **Optional follow-up card:** align contract with actual `/srv/apps/tomato` scaffold once path is visible to workers; file Huly note on TOM-1.

---

## appendix A — provider console paste sheet

```
Meta Valid OAuth Redirect URI:
  https://tomato.<zone>/api/oauth/meta/callback

Meta App Domain:
  tomato.<zone>

TikTok Login Kit Redirect URI:
  https://tomato.<zone>/api/oauth/tiktok/callback

TikTok URL property (only if PULL_FROM_URL):
  Domain: tomato.<zone>
  — or —
  URL prefix: https://tomato.<zone>/media/
```

## appendix B — threat model (MVP, honest)

| Asset | Control |
|---|---|
| Provider tokens at rest | AES-GCM + env key; disk perms on SQLite |
| Provider app secrets | env 600; never client |
| Operator session | httpOnly Secure cookie + passcode |
| Public media | unguessable ids; no directory listing; optional later signed URLs |
| Tunnel | origin not directly on WAN |

This is **not** multi-tenant SaaS hardening. Good enough for Frank-on-one-box; revisit before any second operator or remote untrusted access beyond CF Access if added.

---

*End of report — t_2d125f8e*
