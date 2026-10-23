# Node.js October 2026 Security Releases + eMudhra Intermediate CA Compromise

**Task:** `t_c5b02058`  
**Published:** 2026-10-23  
**Archive path:** `/home/ice/know/research/nodejs-emudhra-ca-oct2026-security-releases-t_c5b02058.md`  
**Primary sources retrieved:** 2026-10-23 (curl/markdown extraction; Firecrawl keyless 403 fallback)

---

## objective

Produce a source-grounded technical research brief on:

1. The Node.js **Wednesday, 21 October 2026** security releases for lines **22.x / 24.x / 26.x** (patched floors **22.23.4**, **24.22.1**, **26.12.1**).
2. The related **eMudhra (emSign) intermediate CA private-key compromise** (reported 17 Oct 2026; public root-program statements 20 Oct 2026).
3. Operator impact: what is fixed in Node, what is **not** fixed by a Node bump alone, EOL/unsupported lines, and concrete upgrade guidance.

---

## summary

On **21 October 2026**, the Node.js project shipped security releases for its three supported lines:

| Line | Patched floor | Label (as of release day) |
|------|---------------|---------------------------|
| 22.x | **22.23.4** | LTS (codename Jod) |
| 24.x | **24.22.1** | LTS (codename Krypton; “Latest LTS” badge on nodejs.org) |
| 26.x | **26.12.1** | Current |

The release bundle addresses **ten CVE-tracked issues** (four High, three Medium, three Low) plus **dependency / trust-store** updates. Highest severity class stated by the project is **HIGH** (not Critical on the Node advisory itself). The standout High is **CVE-2026-111497**: Brotli decoder **out-of-bounds write** that Node states **permits remote code execution** when attacker-controlled Brotli is decoded.

The same releases also remediate the **eMudhra intermediate CA compromise** inside Node’s TLS trust path by:

- Updating bundled roots to **NSS 3.131.1** (root store that drops **emSign ECC Root CA - G3**).
- **Rejecting TLS chains** that contain the compromised intermediate **public keys**.

The eMudhra incident itself is a **Web PKI trust failure**, not a Node-only bug: eight intermediate CAs (six TLS, two code-signing) had **catastrophically weak P-384 private keys** (recoverable as small integers from the public certificates). Browsers blocked via CRLSet/OneCRL within hours of the 17 Oct report; non-browser runtimes and apps that vendor their own CA bundle remain exposed until **those** bundles/runtimes are updated. Chrome’s full removal of **emSign ECC Root CA - G3** from the Chrome Root Store is expected to reach all dependent devices on the morning of **Tuesday 27 October 2026 (PDT)**.

**Operator bottom line:** upgrade every Node 22/24/26 runtime to the floors above **now**. Node 20 and older are **EOL** and **will not** receive these fixes (platform advisories explicitly call this out). A Node bump is necessary but **not sufficient** for full eMudhra exposure: also refresh OS `ca-certificates`, language CA bundles (certifi, Go toolchain, PHP ca-bundle, etc.), container base images, and any vendored `cacert.pem` / `NODE_EXTRA_CA_CERTS`.

---

## plan (executed)

```yaml
objective: Source-backed brief on Node Oct 2026 sec releases + eMudhra CA incident
success_criteria:
  - Exact patched versions and line status
  - Full CVE list with severity and affected lines
  - eMudhra timeline, mechanism, browser vs non-browser impact
  - What Node fixed vs residual gaps
  - Upgrade guidance including EOL note
assumptions:
  - Primary authority is nodejs.org vulnerability + release pages
  - Secondary: Vercel bulletin, Rescana/Sanka trackers, eMudhra statement
  - No production inventory of this swarm's Node pin was in scope
tasks:
  - T1: Pull official Node advisory + release notes (curl; Firecrawl 403)
  - T2: Map CVE table + dependency updates
  - T3: Cross-source eMudhra timeline and trust-store remediation
  - T4: Platform operator notes (Vercel etc.)
  - T5: Structured report + publish to know archive
open_questions:
  - Exact Active LTS vs Maintenance LTS phase labels for 22 vs 24 beyond "LTS" badges
  - Whether NVD has published full CVSS vectors yet for all ten CVEs
  - Host/org Node pin inventory (out of scope for this card)
```

---

## claims

### C1 — Patched floors and release date
- **claim_id:** C1  
- **statement:** Node.js published security releases on **2026-10-21** for **v22.23.4 (LTS)**, **v24.22.1 (LTS / Latest LTS)**, and **v26.12.1 (Current)**.  
- **evidence_refs:** [E1], [E2], [E3], [E4], [E5]  
- **confidence:** high  
- **status:** supported  

### C2 — Highest severity is HIGH; ten CVE-class issues
- **claim_id:** C2  
- **statement:** The Oct 21 advisory lists ten vulnerabilities with severities High×4, Medium×3, Low×3; project summary states highest severity fixed is **HIGH**.  
- **evidence_refs:** [E1]  
- **confidence:** high  
- **status:** supported  

### C3 — CVE-2026-111497 Brotli OOB write → RCE (High)
- **claim_id:** C3  
- **statement:** **CVE-2026-111497** is a High-severity Brotli decoder out-of-bounds write affecting 22.x/24.x/26.x; Node states it permits **remote code execution**. Exposure requires code/libs that **decode attacker-controlled Brotli** (request body or fetched response). Fixed only by runtime upgrade.  
- **evidence_refs:** [E1], [E6]  
- **confidence:** high  
- **status:** supported  

### C4 — Additional High TLS/DNS issues
- **claim_id:** C4  
- **statement:** Other High issues in the same drop: **CVE-2026-105482** (`dns.resolve` NUL truncation, 22/24/26), **CVE-2026-105483** (TLS hostname bypass via fullwidth digit IP form, **24/26 only**), **CVE-2026-106069** (TLS hostname bypass when host loosely resolves to IP, 22/24/26).  
- **evidence_refs:** [E1]  
- **confidence:** high  
- **status:** supported  

### C5 — Medium and Low issues
- **claim_id:** C5  
- **statement:** Medium: keep-alive response-before-request (**CVE-2026-105481**), non-canonical IPv4 / DNS SAN IP auth (**CVE-2026-106072**), snapshot/SEA entropy reuse for `randomUUID`/`randomInt` (**CVE-2026-106074**). Low: ZipFile prefix after failed header (**CVE-2026-106070**, **26 only**), Permission Model wildcard radix over-grant (**CVE-2026-106071**, **26 only**), Permission Model `fs.realpathSync` bypass (**CVE-2026-106073**, 22/24/26).  
- **evidence_refs:** [E1]  
- **confidence:** high  
- **status:** supported  

### C6 — Dependency / trust-store updates in the same releases
- **claim_id:** C6  
- **statement:** Releases also update: **c-ares** (22/24/26), **ca-cert / eMudhra intermediate CA compromise** remediation (22/24/26), **OpenSSL** (22/24), **SQLite** (22), **Undici** (22/24/26).  
- **evidence_refs:** [E1]  
- **confidence:** high  
- **status:** supported  

### C7 — eMudhra mechanism and scope
- **claim_id:** C7  
- **statement:** Eight eMudhra (emSign) intermediate CA private keys were recoverably weak (P-384 keys effectively tiny integers from a **2025-04-30** key ceremony). Six TLS + two code-signing intermediates; Sectigo cross-certs for the same keys also revoked. Keys enable fraudulent cert issuance for domains clients would trust.  
- **evidence_refs:** [E7], [E8], [E9], [E10]  
- **confidence:** high  
- **status:** supported  

### C8 — Browser vs non-browser residual risk
- **claim_id:** C8  
- **statement:** Chrome CRLSet and Firefox OneCRL blocked the intermediates on **17 Oct 2026** (hours after report). Residual risk is **non-browser clients and runtimes with private trust stores** that lack live revocation/CT. Chrome removes **emSign ECC Root CA - G3** for all Chrome-Root-Store consumers on **2026-10-27 PDT morning**. eMudhra states it is not aware of pre-disclosure exploitation; CT reportedly showed no end-entity TLS certs under the six TLS intermediates before browser block (still verify against current trackers).  
- **evidence_refs:** [E7], [E8], [E9], [E10]  
- **confidence:** high  
- **status:** supported  

### C9 — What Node specifically fixed for eMudhra
- **claim_id:** C9  
- **statement:** Node **22.23.4 / 24.22.1 / 26.12.1** update bundled roots to **NSS 3.131.1** and **reject chains containing the blocked public keys**. Electron fixed floors cited by trackers: **44.9.1 / 43.8.1 / 42.11.16**.  
- **evidence_refs:** [E7], [E8], [E1]  
- **confidence:** high  
- **status:** supported  

### C10 — EOL lines and platform guidance
- **claim_id:** C10  
- **statement:** **Node 20.x and older are EOL** and do **not** receive this security drop. Operators on 20.x must move to a supported line (22 or 24 recommended for LTS stability; 26 for Current). Managed platforms (e.g. Vercel) may auto-roll patch levels on 22/24 but still require version-line changes off 20.x; WAF rules do **not** replace patching (especially for **outbound** Brotli decode).  
- **evidence_refs:** [E5], [E6]  
- **confidence:** high  
- **status:** supported  

### C11 — Upgrade is multi-ecosystem for eMudhra
- **claim_id:** C11  
- **statement:** Full eMudhra mitigation is **not** “Node only”: also update OS CA packages, Python certifi (**≥2026.10.19**), pip/uv pins, Go toolchains with the related fix, PHP `composer/ca-bundle`, Rust webpki-roots, container rebuilds, and scan for vendored CA files / `NODE_EXTRA_CA_CERTS` / `SSL_CERT_FILE`. Some ecosystems (e.g. Bun stable, several JDK LTS lines as of 22 Oct tracker checks) lagged.  
- **evidence_refs:** [E7], [E8]  
- **confidence:** high  
- **status:** supported  

### C12 — Line status snapshot from nodejs.org previous-releases table
- **claim_id:** C12  
- **statement:** As of the 2026-10-23 pull of nodejs.org previous-releases: **v26 Current** (last updated Oct 21), **v24 LTS**, **v22 LTS**, **v25/v23/v21/v20 and older EOL** (v20 last updated Mar 24, 2026). Fine-grained “Active LTS vs Maintenance LTS” labels for 22 vs 24 were not isolated beyond LTS badges on release pages.  
- **evidence_refs:** [E5], [E2], [E3], [E4]  
- **confidence:** medium  
- **status:** partial  

---

## Full CVE table (from official advisory)

| CVE | Title (short) | Severity | 22.x | 24.x | 26.x |
|-----|---------------|----------|------|------|------|
| CVE-2026-105482 | `dns.resolve` truncates names containing NUL | High | Y | Y | Y |
| CVE-2026-105483 | TLS hostname bypass — IP written with fullwidth digits | High | N | Y | Y |
| CVE-2026-106069 | TLS hostname bypass — host loosely resolves to IP | High | Y | Y | Y |
| CVE-2026-111497 | Brotli decoder OOB write → RCE | High | Y | Y | Y |
| CVE-2026-105481 | Keep-alive processes response written before request | Medium | Y | Y | Y |
| CVE-2026-106072 | Non-canonical IPv4 hostnames / DNS SAN IP auth | Medium | Y | Y | Y |
| CVE-2026-106074 | `randomUUID`/`randomInt` entropy reused via snapshots/SEAs | Medium | Y | Y | Y |
| CVE-2026-106070 | ZipFile attacker-controlled prefix after failed ZIP header | Low | N | N | Y |
| CVE-2026-106071 | Permission Model filesystem over-grant via wildcard radix | Low | N | N | Y |
| CVE-2026-106073 | Permission Model bypass with `fs.realpathSync` | Low | Y | Y | Y |

**Dependency updates called out on the advisory:** c-ares (22/24/26); ca-cert / eMudhra (22/24/26); OpenSSL (22/24); SQLite (22); Undici (22/24/26).

---

## eMudhra timeline (UTC unless noted)

| When | Event |
|------|--------|
| 2025-04-30 | Flawed key ceremony for the eight intermediates |
| 2026-10-17 ~18:05 | Researcher reports recoverable keys + proof to root programs + eMudhra |
| 2026-10-17 ~19:45 | Chrome CRLSet blocks intermediates by SPKI hash |
| 2026-10-17 20:06 | Mozilla receives report; OneCRL block starts |
| 2026-10-17 ~21:48 | Mozilla independently recovers keys |
| 2026-10-17 ~22:59 | OneCRL distribution complete for default Firefox |
| 2026-10-18 | eMudhra revokes eight certs (`caCompromise`) |
| 2026-10-19 | Sectigo revokes eight cross-certs; Apple blocks |
| 2026-10-20 | Microsoft blocks; Mozilla public disclosure; NSS **3.131.1**; root distrust of **emSign ECC Root CA - G3** path announced |
| 2026-10-21 | Node.js **22.23.4 / 24.22.1 / 26.12.1** security releases |
| 2026-10-27 PDT morning | Chrome Root Store removal of **emSign ECC Root CA - G3** expected to reach all dependent devices |

**eMudhra public position (statement):** incident limited to specific intermediates from the April 2025 KGC; root keys not affected; claims most affected intermediates were not in production and that **no end-entity certificates** were issued (corroborated by CT in their telling). Operators should still treat non-browser trust stores as urgent and plan for **root removal breakage** of any remaining legitimate chains to that root before Oct 27.

---

## Upgrade guidance (operators)

### Immediate (Node)
1. Inventory every Node process, container `FROM node:`, CI image, serverless runtime pin, and Electron app.
2. Raise floors:
   - **22.x → ≥ 22.23.4**
   - **24.x → ≥ 24.22.1**
   - **26.x → ≥ 26.12.1**
3. **Exit Node 20.x / older** — EOL, unpatched for this drop and for Brotli RCE.
4. Rebuild and redeploy; do not assume “minor auto-patch” on self-managed images.
5. Prefer **24.x LTS** for long-lived production unless you need Current 26 features; keep 22 only if you still require that LTS train and can stay current on patches.

### Immediate (eMudhra / TLS trust — beyond Node)
1. Upgrade OS CA packages (Debian/Ubuntu/Alpine advisories per tracker).
2. Refresh language bundles: certifi, Go toolchain rebuilds, PHP ca-bundle, Rust webpki-roots, etc.
3. Grep lockfiles and images for vendored CA material; strip **emSign ECC Root CA - G3** and the listed intermediate fingerprints.
4. Audit `NODE_EXTRA_CA_CERTS`, `SSL_CERT_FILE`, `REQUESTS_CA_BUNDLE`.
5. Before **2026-10-27**, identify any of *your* public sites/APIs still chaining to the dying root and re-issue certs.

### Brotli RCE exposure check
- App/library decodes Brotli from **untrusted request bodies**, or
- Client stack auto-decodes **Brotli responses** from untrusted servers (`fetch`, HTTP clients with brotli support).
- Network WAF on ingress **does not** protect outbound decode paths.

### Verify after upgrade
```bash
node -v   # expect >= floors above for your line
# Confirm process binary path matches the upgraded runtime
# Rebuild multi-stage Docker images from scratch (no stale layer cache on node base)
```

---

## evidence_index

| evidence_id | source_label | source_url | provenance | support note |
|-------------|--------------|------------|------------|--------------|
| E1 | Node.js Oct 21 2026 Security Releases | https://nodejs.org/en/blog/vulnerability/october-2026-security-releases | curl 2026-10-23; HTML→text extract (Firecrawl 403) | Full CVE list, severities, affected lines, dependency updates, download links to three releases |
| E2 | Node.js 22.23.4 release | https://nodejs.org/en/blog/release/v22.23.4 | curl 2026-10-23 | Security release; LTS label |
| E3 | Node.js 24.22.1 release | https://nodejs.org/en/blog/release/v24.22.1 | curl 2026-10-23 | Security release; LTS / Latest LTS; Krypton |
| E4 | Node.js 26.12.1 release | https://nodejs.org/en/blog/release/v26.12.1 | curl 2026-10-23 | Security release; Current; notes CVE-2026-106069/106072 TLS IP work |
| E5 | Node.js previous releases table | https://nodejs.org/en/about/previous-releases | curl 2026-10-23 | v26 Current last upd Oct 21; v24/v22 LTS last upd Oct 20–21; v20 EOL |
| E6 | Vercel KB Node.js security advisory | https://vercel.com/kb/bulletin/node-js-security-advisory | curl 2026-10-23 | Brotli RCE framing; patch floors; Node 20 EOL on Vercel; WAF ≠ patch |
| E7 | Sanka research — eMudhra CA compromise | https://sanka.com/research/incidents/emudhra-intermediate-ca-compromise/ | curl 2026-10-23 | Timeline, fingerprints, Node fixed versions, NSS 3.131.1, multi-ecosystem table |
| E8 | Rescana — eMudhra CA key compromise | https://www.rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026 | curl 2026-10-23 | Confirms Node 21 Oct fixes; certifi/pip/uv updates; non-browser residual risk |
| E9 | eMudhra official statement | https://emudhra.com/en/post/certificate-security-incident | curl 2026-10-23 | Vendor narrative: Mozilla timing, scope claims, no EE certs claim |
| E10 | Secondary news/trackers (index only) | https://thehackernews.com/2026/10/eight-emudhra-ca-keys-compromised.html ; https://blog.talosintelligence.com/compromised-intermediate-certificates/ ; https://ctlogs.dev/blog/emudhra-ca-compromise | web_search titles/snippets 2026-10-23 | Corroborate eight keys, TLS vs code-signing split, browser blocks |

---

## gaps

1. **CVSS base scores / NVD enrichment** for all ten CVEs not pulled (Node labels High/Medium/Low only in the advisory body extracted).
2. **Active LTS vs Maintenance LTS** distinction for 22 vs 24 not confirmed beyond generic LTS badges and previous-releases “LTS” status column.
3. **Exact OpenSSL / c-ares / Undici / SQLite version bumps** inside each Node tarball not diffed from changelogs beyond the advisory’s “updated” list.
4. **Org-specific inventory** (which hosts still on Node 20 / unpatched 22.23.3 etc.) was out of scope.
5. **Exploit PoC / weaponization details** intentionally omitted (defensive briefing only).
6. Some secondary vendor pages (Treasure, Critical Softworks) returned empty/blocked fetches; not required given primary Node + Sanka/Rescana coverage.
7. eMudhra’s “no EE certs issued” claim is **vendor + CT-aligned in secondary reporting** but should be re-checked against live CT if used for risk acceptance.

---

## confidence

```yaml
level: high
rationale: >
  Core version floors, CVE inventory, and severity classes come directly from the
  official Node.js vulnerability page and matching release posts retrieved live.
  eMudhra timeline and Node trust-store remediation are cross-checked across
  Sanka, Rescana, eMudhra statement, and the Node advisory dependency line.
  Residual uncertainty is limited to CVSS vectors, fine-grained LTS phase labels,
  and host inventory.
```

---

## recommended_next_actions

1. **Ops:** Sweep all Node runtimes/images/CI to ≥ **22.23.4 / 24.22.1 / 26.12.1**; force-migrate anything on **≤20.x**.
2. **Sec:** Treat **CVE-2026-111497** as patch-now for any service that decodes Brotli from untrusted input or auto-decompresses brotli HTTP.
3. **PKI:** Before **2026-10-27**, inventory certs chaining to **emSign ECC Root CA - G3** and re-issue; refresh non-Node CA bundles the same day as Node bumps.
4. **Eng:** Pin Docker bases to digest of patched tags; invalidate caches; include Electron apps if any.
5. **Optional follow-up card:** host-level Node version audit + SBOM CA-bundle scan across swarm/prod images.
6. **Do not** treat Vercel/WAF or browser CRLSet as covering self-hosted Node or outbound fetch decode paths.

---

## findings (concise narrative)

The 21 Oct 2026 Node security drop is a **combined application-security and trust-store** event. The most urgent pure-runtime issue is the **Brotli decoder RCE** (High). Three additional High issues touch **DNS truncation** and **TLS hostname verification**, so reverse proxies and IP-literal edge cases deserve regression tests after upgrade. Independently, the **eMudhra intermediate CA key failure** forced every major browser root program into emergency blocks and forced runtimes that ship Mozilla/NSS-derived or static roots—including Node—to ship **NSS 3.131.1** and explicit **compromised-key rejection**. Patching Node is mandatory for supported lines; **completing** eMudhra defense requires the broader CA-bundle ecosystem and attention to the **27 Oct 2026** Chrome root-removal date.

---

*Geraldo v2.2 / GIU — kanban task t_c5b02058*  
*Single durable publish path: this file under `/home/ice/know/research/`*
