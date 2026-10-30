# Wave 2 Ablation Host Decision — PJ + Maza

**Task:** t_b90ca0b0  
**Date:** 2026-10-30  
**Owner:** Romulus (tricon-web)  
**Scope:** Confirm non-prod host path only. Do **not** ablate production. Do **not** run the battery (Geraldo).

---

## 0) Spot-check (reconfirmed this run)

| Host | DNS | HTTP |
|------|-----|------|
| `staging.printjunkieaz.com` | **UNRESOLVED** | — |
| `preview.printjunkieaz.com` | **UNRESOLVED** | — |
| `staging.mazahalalfood.com` | **UNRESOLVED** | — |
| `preview.mazahalalfood.com` | **UNRESOLVED** | — |
| `printjunkieaz.com` / `www.` | resolves → Vercel | 200 |
| `mazahalalfood.com` | resolves → Vercel | 200 |
| `www.mazahalalfood.com` | resolves → Vercel | **TLS hostname mismatch** (cert not valid for www) |

**Conclusion:** There is **no** existing `staging.` / `preview.` subdomain for either site. Ablation must use another path.

**Blocked infra from this agent host (do not wait on these for Wave 2):**
- Namecheap API → `Invalid request IP` (egress `98.97.116.224` not whitelisted; Starlink CGNAT pattern).
- Cloudflare API token in `.agent.env` → **401 Invalid API Token**. CF Worker path is **not** viable until Fedoranzzi/Frank rotates a working token **and** zones are on CF NS (see below).

---

## 1) Project facts (live Vercel API + curl)

### Print Junkie AZ

| Field | Value |
|-------|-------|
| GitHub | `arealicehole/printjunkieaz` (private), production branch `main` |
| Vercel project | `web` (`prj_Qzv3EPmQC3eo32TZFSOqWHTl2Wgx`) |
| Team | `tricon-digital` (`team_GV7ch0ClYsZkkVjH5jJqnGgI`) |
| Root | `web/` (Next.js) |
| Prod domains | `printjunkieaz.com`, `www.printjunkieaz.com` |
| Prod aliases | also `web-tricon-digital.vercel.app`, `web-git-main-tricon-digital.vercel.app` |
| **SSO / password protection** | **`null` / none** — previews are public |
| Auth token scope | agent `VERCEL_TOKEN` (user `arealicehole`) |

**Verified public preview (no login wall):**
- Deployment URL: `https://web-rbnuvz16q-tricon-digital.vercel.app/llms.txt` → **200**
- Branch alias: `https://web-git-feat-agent-identity-layer-tricon-digital.vercel.app/robots.txt` → **200**
- Prod twin (do not ablate): `https://printjunkieaz.com`

### Maza Mediterranean

| Field | Value |
|-------|-------|
| GitHub | `mazahalal/maza-mediterranean`, production branch `main` |
| Vercel project | `maza-mediterranean` (`prj_mnOfbHeeGfnA8KYkrdnktreYl7by`) |
| Team | **`maza-s-projects`** (`team_2S5HVa4xM6Aww4Rm0FO8WIZZ`) — note: `.vercel/project.json` orgId `Kqmi94TEa79HrW3XzwtYmvlF` is **stale/wrong** |
| Prod domains | `mazahalalfood.com`, `maza-mediterranean.vercel.app` |
| **SSO protection** | **`all_except_custom_domains`** |
| Auth token scope | Maza `.env.local` `VERCEL_TOKEN` (user `info-45095564`) |

**Verified:**
- Prod custom domain public: `https://mazahalalfood.com/llms.txt` → **200**
- `https://maza-mediterranean.vercel.app/llms.txt` → **200**, **byte-identical** to apex (sha256 prefix `4513c55809341166`) → this is a **production** alias, not a sandbox
- Feature preview `https://maza-mediterranean-m2rsgvfef-maza-s-projects.vercel.app/...` → redirects to **Vercel SSO login**
- Branch alias `https://maza-mediterranean-git-feat-…-maza-s-projects.vercel.app/...` → **SSO login**
- Even `…-git-main-maza-s-projects.vercel.app` → **SSO login**

**Implication:** Maza default Vercel preview URLs are **not crawlable** by OAI-SearchBot / Claude-SearchBot / PerplexityBot. Unusable for §B ablation as-is.

---

## 2) Path evaluation (task options)

| Path | PJ | Maza | Notes |
|------|----|------|-------|
| A. Vercel preview on feature branch | **YES — ready now** | **NO** (SSO wall) | PJ: zero infra. Maza: bots never see content. |
| B. Cloudflare Worker rewrite | **NO (now)** | **NO (now)** | CF token dead; apex already on Vercel IPs; Namecheap DNS API IP-blocked from agent. |
| C. Ephemeral clone subdomain / project | Optional later | **YES — recommended** | New Vercel project under `tricon-digital` **without** SSO, or branch-scoped **custom domain** on Maza project (custom domains bypass SSO). |
| D. Prod ablation | **FORBIDDEN** | **FORBIDDEN** | Hard rule until Frank explicitly approves. |
| E. Retrospective triangulation (§B.4) | Available fallback | Available fallback | Maza has `NEXT_PUBLIC_GA_MEASUREMENT_ID` in env; PJ GA not in local env (confirm GA4 property access separately). |

---

## 3) PICKS (one path each)

### PJ — **Path A: Vercel feature-branch previews** (Tier 1)

**Why:** Already public, already linked to GitHub, no DNS, no CF, no Frank infra ticket. Matches §B.3 Tier 1 "preview".

**Limits (label in research):**
- Host is `*.vercel.app`, not `printjunkieaz.com` → **domain-authority / brand weight differs** from prod (Tier-3-like caveat on discovery prompts). Still valid for **URL-conditioned / crawl-surface** ablation (bots can fetch the exact signal state). For pure local-intent discovery (P1–P7 without URL), treat results as **relative / directional**, not identical to prod SERP behavior.
- Prefer **stable branch aliases** over ephemeral `web-<hash>-…` URLs so the battery can re-hit the same host across days.

**Stable URL pattern (PJ):**
```text
https://web-git-<branch-slug>-tricon-digital.vercel.app
```
Example (existing): `https://web-git-feat-agent-identity-layer-tricon-digital.vercel.app`

Also works: per-deployment `https://web-<id>-tricon-digital.vercel.app` (changes every push).

### Maza — **Path C: Ablation clone project on `tricon-digital` (no SSO)** (Tier 3 clone, public)

**Why:** Maza production project SSO blocks all non-custom-domain previews. CF Worker blocked. Custom subdomain needs DNS Frank can't get from this host automatically.

**Why not "just turn off SSO" on prod project:** touches production protection posture; needs Frank. Clone isolates risk.

**Alternative if Frank wants same-account DNS later (still Path C-ish / Tier 1):**
Add Vercel domain `ablation.mazahalalfood.com` with `gitBranch = <ablation-branch>` (custom domains are SSO-exempt). Requires DNS CNAME → `cname.vercel-dns.com` via Namecheap UI or IP-whitelisted API. **Not doable from this agent host today.**

**Clone URL pattern (after setup):**
```text
https://maza-ablation-tricon-digital.vercel.app
https://maza-ablation-git-<branch-slug>-tricon-digital.vercel.app
```

---

## 4) Absolute steps — PJ (ready to use; research can start)

**Do not merge ablation branches to `main`. Do not promote to production.**

### 4.1 One-time sanity
```bash
set -a
source /workspace/.agent.env
source /workspace/printjunkieaz/.env   # if present
set +a
cd /workspace/printjunkieaz
# confirm Vercel link
cat web/.vercel/project.json
# expect: projectName web, org team_GV7ch0ClYsZkkVjH5jJqnGgI, prj_Qzv3EPmQC3eo32TZFSOqWHTl2Wgx
```

### 4.2 Create a single-signal ablation branch (example: V1 kill llms.txt)
```bash
cd /workspace/printjunkieaz
git fetch origin
git checkout main && git pull origin main
git checkout -b ablation/v1-no-llms

# Implement EXACTLY one change, e.g. remove or force-404 app route for llms.txt
# (match §B.2 V1 — leave UCP/schema/prices/robots intact)

git -C web status   # or repo root depending on layout
# commit on branch only
git add -A && git commit -m "ablation(v1): force llms.txt 404 for citation battery"
git push -u origin ablation/v1-no-llms
```

### 4.3 Wait for Vercel preview; capture URLs
- GitHub PR optional (useful for Frank visibility) but **not required** for preview deploy if project auto-deploys branches.
- Preview appears under Vercel project `web` / team `tricon-digital`.
- Prefer branch alias:
  - `https://web-git-ablation-v1-no-llms-tricon-digital.vercel.app`
  - (Vercel slugifies `/` → `-`; confirm exact alias in Vercel deploy UI or API)

### 4.4 Crawlability gate (must pass before battery)
```bash
BASE=https://web-git-ablation-v1-no-llms-tricon-digital.vercel.app
curl -sS -o /dev/null -w '%{http_code}\n' -A 'OAI-SearchBot' "$BASE/"
curl -sS -o /dev/null -w '%{http_code}\n' -A 'OAI-SearchBot' "$BASE/llms.txt"          # expect 404 for V1
curl -sS -o /dev/null -w '%{http_code}\n' -A 'OAI-SearchBot' "$BASE/.well-known/ucp"  # expect 200
curl -sS -o /dev/null -w '%{http_code}\n' -A 'OAI-SearchBot' "$BASE/robots.txt"       # expect 200
# Fail gate if any answer-bot UA gets 401/403/302-to-login
```

### 4.5 Branch matrix naming (keep one signal per branch)
| Branch | Variant |
|--------|---------|
| `ablation/v0-control` | full stack (optional; can use latest `main` preview) |
| `ablation/v1-no-llms` | llms.txt 404 |
| `ablation/v2-no-ucp` | `/.well-known/ucp` 404 |
| `ablation/v3-no-jsonld` | strip JSON-LD |
| `ablation/v4-no-visible-prices` | hide `$` in HTML |
| `ablation/v5-block-oai-search` | robots Disallow OAI-SearchBot |
| `ablation/v6-block-ai-search` | robots Disallow all answer bots |

### 4.6 Hand Geraldo
- List of **live** `BASE` URLs per variant (branch aliases).
- Note: discovery prompts without URL are **domain-shifted**; recommend battery legs that (a) open-discovery and (b) **URL-forced** ("based on this site URL …") so signal ablation is measurable even when brand domain authority differs.

---

## 5) Absolute steps — Maza (clone project; ~15–30 min once Frank/token OK)

**Do not change SSO on production `maza-mediterranean` without Frank.**  
**Do not point `mazahalalfood.com` or `maza-mediterranean.vercel.app` at ablation branches.**

### 5.1 Create ablation project under TriCon (agent token)
```bash
set -a
source /workspace/.agent.env
set +a

# From a clean worktree of mazahalal/maza-mediterranean (or mirror under arealicehole)
# Option A — Vercel CLI (logged in as arealicehole / tricon-digital):
cd /path/to/maza-mediterranean-ablation-checkout
vercel link --yes --project maza-ablation --scope tricon-digital
# or: vercel project add maza-ablation
vercel deploy --yes   # first deploy creates project on tricon-digital

# Confirm SSO is OFF on new project (default for many hobby/team projects):
# Vercel → Project → Deployment Protection → ensure not "all_except_custom_domains"
# API check: ssoProtection should be null
```

### 5.2 Wire Git (recommended)
- Connect repo `mazahalal/maza-mediterranean` **or** a fork `arealicehole/maza-mediterranean-ablation` to project `maza-ablation`.
- Production branch for the **clone** can be `ablation/v0-control` (not client prod `main` alias).
- Copy **non-secret** build env needed for static signal pages (menu data is in-repo). **Do not** copy Twilio/Resend production secrets unless required for page render — prefer build-time public data only.

### 5.3 Variant branches (same matrix as PJ)
Push `ablation/v1-no-llms` … `ablation/v6-block-ai-search` to the connected repo; each gets:
```text
https://maza-ablation-git-<slug>-tricon-digital.vercel.app
```

### 5.4 Crawlability gate (must pass)
```bash
BASE=https://maza-ablation-git-ablation-v1-no-llms-tricon-digital.vercel.app
for UA in 'OAI-SearchBot' 'Claude-SearchBot' 'PerplexityBot' 'Googlebot'; do
  code=$(curl -sS -o /dev/null -w '%{http_code}' -A "$UA" "$BASE/")
  echo "$UA $code"   # must be 200, not 401/403/login
done
# Confirm SSO wall is gone:
curl -sI -A 'OAI-SearchBot' "$BASE/" | tr -d '\r' | grep -iE 'HTTP/|location:'
```

### 5.5 Optional later upgrade (Frank DNS) → true Tier 1 subdomain
If Frank adds DNS manually in Namecheap (agent API IP-blocked):
```text
Type: CNAME
Host: ablation
Value: cname.vercel-dns.com
```
Then Vercel project domain API / UI:
```text
Domain: ablation.mazahalalfood.com
gitBranch: ablation/v0-control   # or leave unset and use path/header variants
```
Because **custom domains bypass Maza SSO**, this could even live on the **existing** Maza Vercel project — but still **must not** attach to production branch `main`. Prefer dedicated clone until process is proven.

### 5.6 CF Worker path — deferred (not viable this run)
| Blocker | Detail |
|---------|--------|
| Token | `CLOUDFLARE_API_TOKEN` → 401 |
| DNS API | Namecheap rejects agent egress IP |
| Architecture | Apex already Vercel; Worker needs CF proxy on hostname |

Re-open only after: valid CF token + zone on CF NS (or hostname proxied) + Frank approval for edge rewrites on a **non-prod hostname**.

---

## 6) What is NOT viable (say so clearly)

| Claim | Reality |
|-------|---------|
| "Use staging.printjunkieaz.com" | **Does not exist** (NXDOMAIN) |
| "Use staging.mazahalalfood.com" | **Does not exist** (NXDOMAIN) |
| "Use Maza Vercel preview URL" | **SSO login wall** — bots cannot crawl |
| "Use maza-mediterranean.vercel.app as sandbox" | **Production alias** (identical body to apex) |
| "CF Worker rewrite this week from agent" | **Token dead + DNS API IP-blocked** |
| "Ablate prod robots/llms/ucp" | **Out of scope / forbidden without Frank** |

If Maza clone project is **not** stood up before Wave 2 wants causal runs:
→ **Maza Wave 2 causal = not ready** → research **falls back to retrospective triangulation** (§B.4: GA4 AI referrals + bot logs + signal ship dates).  
PJ can still run **prospective** ablation on Vercel previews immediately.

---

## 7) Recommendation summary (for Frank / Geraldo)

| Site | Chosen path | First ablation base URL (pattern) | Frank action needed? |
|------|-------------|-----------------------------------|----------------------|
| **PJ** | Vercel feature-branch preview | `https://web-git-ablation-<variant>-tricon-digital.vercel.app` | **No** — already public |
| **Maza** | New `maza-ablation` project on `tricon-digital` (no SSO) | `https://maza-ablation-git-ablation-<variant>-tricon-digital.vercel.app` | **Yes** — allow clone project + which git remote (mazahalal vs fork). Optional DNS later. |

**Prod hard rule remains:** never remove signals from `printjunkieaz.com` / `mazahalalfood.com` without explicit Frank approval.

**Research labeling:**
- PJ preview ablation = prospective, crawlable, **domain-shifted** (label relative).
- Maza until clone exists = **retrospective only**.
- After Maza clone = prospective, crawlable, **domain-shifted** (same label).

---

## 8) Evidence appendix (this run)

```text
PJ ssoProtection: null
PJ preview curl: web-rbnuvz16q-tricon-digital.vercel.app/llms.txt → 200
PJ branch curl: web-git-feat-agent-identity-layer-tricon-digital.vercel.app/robots.txt → 200

Maza ssoProtection: {deploymentType: all_except_custom_domains}
Maza preview curl: maza-mediterranean-m2rsgvfef-maza-s-projects.vercel.app → SSO login redirect
Maza prod alias: maza-mediterranean.vercel.app/llms.txt sha == mazahalalfood.com/llms.txt

DNS: staging|preview .(printjunkieaz|mazahalalfood).com → UNRESOLVED
Namecheap: Invalid request IP 98.97.116.224
Cloudflare token: 401 Invalid API Token
```

---

*Romulus — Wave 2 prep only. Battery execution = Geraldo. Bible edits = out of scope.*
