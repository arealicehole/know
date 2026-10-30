# PJ Ablation Preview Branches — V0–V3 (AVP Causal Battery)

**Date:** 2026-10-30  
**Kanban:** `t_4c2afe0d`  
**Repo:** `arealicehole/printjunkieaz`  
**Host decision:** `/workspace/seo-bible/research-sources/ablation-host-decision-20261030.md`  
**Design:** design pack §B / `avp-causal-verification-multi-engine-citation-battery-t_70d29518.md`  
**Production:** **NOT touched.** All arms are public Vercel **preview** deployments only.

## Baseline (control stack)

Cut from `origin/main` @ `2fd428bb` — full dual-stack agent identity already on main via PR #11:

| Signal | Path / surface | Present on main |
|--------|----------------|-----------------|
| llms.txt | `/llms.txt` (`web/public/llms.txt`) | yes |
| UCP | `/.well-known/ucp` | yes |
| LocalBusiness identity | root JSON-LD `PostalAddress` + `sameAs` (Yahoo Local) | yes |
| Offer catalog / FAQ / Dataset | other JSON-LD blocks | yes (left intact on all arms) |
| `/api/specs` | catalog index | yes |

## Branches (git)

| Variant | Branch | SHA | Diff vs main (signal only) |
|---------|--------|-----|----------------------------|
| **V0 baseline** | `experiment/pj-ablation-v0-baseline` | `14af173d` | marker only: `web/scripts/ABLATION_V0.txt` (no runtime effect; empty commit was skipped by Vercel) |
| **V1 no-llms** | `experiment/pj-ablation-v1-no-llms` | `cc279c5e` | **delete** `web/public/llms.txt` |
| **V2 no-ucp** | `experiment/pj-ablation-v2-no-ucp` | `91c4d091` | **delete** `web/src/app/.well-known/ucp/route.ts` |
| **V3 no-identity-schema** | `experiment/pj-ablation-v3-no-identity-schema` | `622ba93c` | strip `address`/`PostalAddress` + `sameAs` from root `LocalBusiness` in `web/src/app/layout.tsx` |

Each non-V0 arm is a **single-variable** kill. Other signals stay on.

## Public preview URLs

Prefer **deployment URL** (stable for this battery run). Branch aliases also work (V3 alias is truncated by Vercel DNS length limits).

| Variant | Deployment URL (READY) | Branch alias |
|---------|------------------------|--------------|
| **V0** | https://web-qgfwqxqjz-tricon-digital.vercel.app | https://web-git-experiment-pj-ablation-v0-baseline-tricon-digital.vercel.app |
| **V1** | https://web-9zr136bqb-tricon-digital.vercel.app | https://web-git-experiment-pj-ablation-v1-no-llms-tricon-digital.vercel.app |
| **V2** | https://web-bg4t8s6td-tricon-digital.vercel.app | https://web-git-experiment-pj-ablation-v2-no-ucp-tricon-digital.vercel.app |
| **V3** | https://web-hd8c3mgrl-tricon-digital.vercel.app | https://web-git-experiment-pj-ablation-v3-no-iden-a9ac0d-tricon-digital.vercel.app |

**Auth:** Project has `passwordProtection: null`, `ssoProtection: null`. Curl proofs show `auth_wall=0` on all arms — public, no Vercel login wall.

**Target:** all deployments `target: null` (preview). Production aliases (`printjunkieaz.com`, `web-git-main-…`) remain on main @ `2fd428bb`.

## Curl proofs (2026-10-30)

```
V0  llms=200  ucp=200  home=200  specs=200  LocalBusiness=1  PostalAddress=1  sameAs/yahoo=1
V1  llms=404  ucp=200  home=200  specs=200  LocalBusiness=1  PostalAddress=1  sameAs/yahoo=1
V2  llms=200  ucp=404  home=200  specs=200  LocalBusiness=1  PostalAddress=1  sameAs/yahoo=1
V3  llms=200  ucp=200  home=200  specs=200  LocalBusiness=1  PostalAddress=0  sameAs/yahoo=0
PROD (control intact) llms=200 ucp=200 PostalAddress present
```

Raw transcript: `/workspace/printjunkieaz-ablation-wt/scripts/ablation-curl-proofs.txt`

### One-liner re-check

```bash
for u in \
  https://web-qgfwqxqjz-tricon-digital.vercel.app \
  https://web-9zr136bqb-tricon-digital.vercel.app \
  https://web-bg4t8s6td-tricon-digital.vercel.app \
  https://web-hd8c3mgrl-tricon-digital.vercel.app
do
  printf '%s ' "$u"
  curl -sS -o /dev/null -w 'llms:%{http_code} ' "$u/llms.txt"
  curl -sS -o /dev/null -w 'ucp:%{http_code}\n' "$u/.well-known/ucp"
done
```

## Battery handoff notes

1. Point the multi-engine citation battery at the **four deployment URLs** above (not prod).
2. V0 is the control; compare citation_rate(V0) − citation_rate(Vn) per design §B.5.
3. Do **not** merge these branches to main. Do **not** promote previews to production.
4. After the battery window, branches can be deleted; keep this doc + curl proofs for the causal report.
5. V4–V6 (price hide / bot blocks) were **out of scope** for this card (host decision: V0–V3 first).

## Local worktree

`/workspace/printjunkieaz-ablation-wt` (git worktree on the experiment branches).

## Acceptance checklist

- [x] Four experiment branches pushed
- [x] Four public Vercel preview deploys READY
- [x] Curl proofs: V1 llms 404; V2 ucp 404; V3 PostalAddress+sameAs gone; V0 all present
- [x] Prod unchanged
- [x] URLs documented for battery runner
