# AI → PDF Conversion on Linux (No Adobe) — Research Report
**task_id:** t_bdf343f7  
**slug:** ai-to-pdf-conversion-linux  
**date_utc:** 2026-10-13T20:42:24Z  
**agent:** Geraldo v2.2 (GIU)  
**inventory_context (from parent handoff):** 2,596 `.ai` files scanned; **1,164** classified `pdf_compatible`; **1,432** classified `needs_conversion` (PostScript-era / non-PDF-compatible). Paths cited by parent: Sulemon Dropbox, Pierre Dropbox, Desktop Archives.  
**note:** `inventory.csv` / `needs_conversion.txt` attachments are host-only under `/home/ice/.hermes/kanban/boards/research/attachments/t_bdf343f7/` and were **not mounted** into the Geraldo Docker sandbox for this run. Counts below therefore rely on the parent task body, not a re-parse of the CSV.

---

## objective

Produce an operator-ready research brief for converting a large mixed archive of Adobe Illustrator `.ai` files to PDF on Linux **without Adobe Illustrator/Acrobat**, covering:

1. How `.ai` files are structured (PDF-compatible vs legacy PostScript / non-PDF-compatible).
2. Open-source and commercial-but-non-Adobe Linux tool matrix with install + CLI recipes.
3. Fidelity expectations (vectors, text, layers, effects, spot color, transparency).
4. A decision tree / batch pipeline for ~2.6k files given the 1,164 / 1,432 split.
5. Explicit gaps and what remains blocked without Illustrator or without sample bytes.

## summary

Since **Illustrator 9 (2000)**, native `.ai` is a **PDF container** that optionally embeds a full PDF page stream plus proprietary **AI private data** (`/AIPrivateData`). The save option **Create PDF Compatible File** (Adobe default ON) controls whether the PDF page stream holds real artwork or only the warning page *"This is an Adobe® Illustrator® File that was saved without PDF Content."* Nearly every non-Adobe consumer (InDesign place, Affinity, Inkscape, Ghostscript, Acrobat open-as-PDF, macOS Quick Look) reads **only the PDF stream**, not private data.

Therefore:

| Class | Approx. count (handoff) | Linux conversion reality |
|---|---:|---|
| **pdf_compatible** (modern AI with real PDF content) | **1,164** | **High success** with Ghostscript `pdfwrite`, Poppler, or “copy/rename + validate”. Often the `.ai` **is already a PDF** for visual purposes. |
| **needs_conversion** (legacy PS AI ≤8, or AI≥9 saved **without** PDF content, or broken dual-structure) | **1,432** | **Hard**. Open tools do **not** reverse-engineer full Illustrator private data. Legacy pure-PostScript AI can sometimes be interpreted by **Ghostscript** as PS; non-PDF-compatible modern AI generally yield **warning-only PDFs** unless re-saved from Illustrator (or another full AI engine). |

**Recommended batch strategy:**  
1) **Triage** every file with header + content heuristics (not extension).  
2) **Tier A — pdf_compatible:** `gs -sDEVICE=pdfwrite` (or validated copy) → PDF; optional PNG proof via Poppler/Cairo.  
3) **Tier B — legacy PostScript AI:** Ghostscript PS→PDF; fallback `pstoedit` / Inkscape where GS succeeds partially.  
4) **Tier C — non-PDF-compatible modern AI / opaque private data:** queue for **Illustrator re-save** (Create PDF Compatible File ON) or accept **unrecoverable** without Adobe; do not expect Inkscape/GS/Scribus to invent missing PDF content.  
5) **Never** treat conversion as lossless round-trip of live type, symbols, meshes, or spot workflows.

**Confidence (overall):** **medium-high** on format mechanics and tool behavior from primary docs; **medium** on exact inventory composition because CSV bytes were inaccessible this run; **low** on per-file success rates for the 1,432 without sampling real files on the host.

---

## plan

```yaml
objective: Linux AI→PDF conversion research without Adobe for ~2596-file archive
success_criteria:
  - format dual-structure explained with detection heuristics
  - tool matrix with install + CLI
  - fidelity matrix
  - decision tree mapped to 1164/1432 split
  - structured Final Report Contract fields
assumptions:
  - parent classification pdf_compatible vs needs_conversion is directionally correct
  - target is visual/print PDF deliverables, not full AI editability
  - host may install Ghostscript, Poppler, Inkscape; Adobe not available
tasks:
  - {task_id: T1, title: AI format dual-structure, status: done}
  - {task_id: T2, title: Detection heuristics, status: done}
  - {task_id: T3, title: Tool matrix + CLI recipes, status: done}
  - {task_id: T4, title: Fidelity + failure modes, status: done}
  - {task_id: T5, title: Batch decision tree for 2596 files, status: done}
  - {task_id: T6, title: Parse inventory attachments, status: blocked, note: host-only paths}
open_questions:
  - exact header mix inside the 1432 (true PS vs warning-PDF vs corrupt)
  - presence of EPS sidecars / PDF exports already next to AI
  - CMYK/spot requirements for downstream print
```

---

## claims

### claim_id: C1
- **statement:** Since Illustrator 9, `.ai` files use a PDF-based dual structure: a PDF wrapper plus Illustrator-native private data; Create PDF Compatible File controls whether the PDF page stream contains real artwork or only a warning page.
- **evidence_refs:** [E1, E2, E3, E4, E5]
- **confidence:** high
- **status:** supported

### claim_id: C2
- **statement:** Non-Adobe applications (InDesign, Affinity, Inkscape, Ghostscript, most PDF tools) consume the PDF-compatible stream only; they do not interpret full undocumented AI private data.
- **evidence_refs:** [E2, E3, E6, E7, E8]
- **confidence:** high
- **status:** supported

### claim_id: C3
- **statement:** Adobe’s documented meaning of Create PDF Compatible File is: “Saves a PDF representation of the document in the Illustrator file” for compatibility with other Adobe apps; it may embed fonts; background save is disabled when the option is off.
- **evidence_refs:** [E1]
- **confidence:** high
- **status:** supported

### claim_id: C4
- **statement:** Pre-Illustrator 9 `.ai` files are PostScript/DSC-based (not PDF-core); `file(1)` often reports “PostScript document”; they need a PostScript interpreter path (typically Ghostscript), not a pure PDF copy.
- **evidence_refs:** [E4, E9]
- **confidence:** high
- **status:** supported

### claim_id: C5
- **statement:** Ghostscript `pdfwrite` is the primary Linux CLI path to rewrite PDF-compatible `.ai` (and many PS/EPS inputs) into cleaner standalone PDFs; community automation (e.g. GitHub `ai-to-pdf-action`) is literally `gs -sDEVICE=pdfwrite ...`.
- **evidence_refs:** [E10, E11, E12]
- **confidence:** high
- **status:** supported

### claim_id: C6
- **statement:** Inkscape can open AI≥9 when PDF content exists (via internal PDF or Poppler/Cairo import) and export PDF/SVG; it fails or shows warning content when PDF compatibility was off; gradient meshes and some transparency modes are limited; native AI private-data import remains incomplete (Inkscape has explored private-stream work but PDF path is what works today).
- **evidence_refs:** [E7, E8, E13, E14]
- **confidence:** high
- **status:** supported

### claim_id: C7
- **statement:** Scribus can import PDF-based AI (and layers in some cases) and EPS; it is a GUI/layout-centric path, weaker as a headless bulk converter than Ghostscript.
- **evidence_refs:** [E15, E16]
- **confidence:** medium
- **status:** supported

### claim_id: C8
- **statement:** LibreOffice Draw lists AI among importable vector formats (EPS or PDF representation) but is not the highest-fidelity bulk AI→PDF engine; treat as secondary.
- **evidence_refs:** [E17]
- **confidence:** medium
- **status:** partial

### claim_id: C9
- **statement:** PDF content in a “PDF-compatible” AI may omit pasteboard objects outside artboards; fonts may be embedded for preview but not fully editable elsewhere; PDF stream can diverge from native AI due to bugs (including background-save related issues reported by practitioners).
- **evidence_refs:** [E2, E3, E1]
- **confidence:** high
- **status:** supported

### claim_id: C10
- **statement:** For this archive’s **1,164 pdf_compatible** files, automated Linux conversion to visual PDF is **feasible at scale** with Ghostscript (+ optional Poppler proofs). For the **1,432 needs_conversion**, expect a large residue requiring Illustrator re-save or declaring unrecoverable—open tools cannot lawfully/fully reverse private AI data.
- **evidence_refs:** [E2, E6, E7, E8, parent handoff counts]
- **confidence:** medium
- **status:** supported (strategy); partial (exact residue size)

### claim_id: C11
- **statement:** Reliable detection of PDF-compatible vs not is content-based: `%PDF-` header; presence of substantial page content vs the literal warning string; `/AIPrivateData` vs `/AIPDFPrivateData`; legacy `%!PS-Adobe` without PDF; optional `%AI5_FileFormat` version markers and XMP `illustrator:Type`.
- **evidence_refs:** [E3, E4, E9]
- **confidence:** high
- **status:** supported

---

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | Adobe Help — Save artwork (Create PDF Compatible File) | https://helpx.adobe.com/illustrator/using/saving-artwork.html | web_extract → `/root/.hermes/cache/web/helpx.adobe.com-053a540464.md` | “Saves a PDF representation of the document in the Illustrator file. Select this option if you want the Illustrator file to be compatible with other Adobe apps.” Background save does not work when option not selected. |
| E2 | CreativePro — Understanding Illustrator File Formats (Monika Gause) | https://creativepro.com/understanding-illustrator-file-formats/ | web_extract | Create PDF Compatible File embeds a PDF (fonts/images); off-artboard elements excluded from PDF; without option, other apps cannot open; place in InDesign/Affinity uses PDF part. |
| E3 | note.com / monokano — What Does Create PDF Compatible File Mean (2026-03) | https://note.com/monokano/n/n9df9cfc584f4 | web_extract | Dual-structure always PDF wrapper + native; unchecked → PDF contains only warning “saved without PDF Content”; other apps always use PDF data. |
| E4 | Tyler Thorsted — Adobe Illustrator and PDF (preservation, 2023-04-07) | https://preservation.tylerthorsted.com/2023/04/07/adobe-illustrator-and-pdf/ | web_extract | AI≤8 PostScript headers vs AI≥9 PDF headers; PRONOM; `/AIPrivateData` vs `/AIPDFPrivateData`; `%AI5_FileFormat` versioning; modern AI often misidentified as plain PDF by naive tools. |
| E5 | Adobe Community — When should I tick Create PDF Compatible File | https://community.adobe.com/questions-652/when-should-i-tick-create-pdf-compatible-file-in-illustrator-770680 | web_extract | Official-style: option saves PDF representation for other Adobe apps; also useful as corruption recovery of artwork. |
| E6 | Affinity Help — Import .AI | https://www.affinity.studio/help/import-ai/ | web_extract | Affinity opens embedded PDF Compatible File/Stream only; without it shows the standard Adobe warning message on canvas. |
| E7 | VectorCraft issue #537 | https://github.com/storytold/vectorcraft/issues/537 | web_extract | Clean-room tools read PDF-compatible part only; private data undocumented; without PDF compatibility cannot open. |
| E8 | Inkscape wiki — Inkscape for Adobe Illustrator users | https://wiki.inkscape.org/wiki/Inkscape_for_Adobe_Illustrator_users | web_extract | Inkscape opens AI since v9 and PDF with limits: gradient meshes approximated; transparency modes don’t work fully. |
| E9 | file(1) / preservation examples | same as E4 | web_extract | Example: AI8 → “PostScript document text conforming DSC level 3.0”; AI9 → “PDF document, version 1.4”. |
| E10 | Ghostscript — Using Ghostscript | https://ghostscript.readthedocs.io/en/latest/Use.html | web_extract | GS interprets PostScript, EPS, and PDF; high-level `pdfwrite` rebuilds PDF from marking operations. |
| E11 | jv-k/ai-to-pdf-action entrypoint.sh | https://raw.githubusercontent.com/jv-k/ai-to-pdf-action/master/entrypoint.sh | web_extract | `gs -sDEVICE=pdfwrite -dNOPAUSE -dQUIET -dBATCH -sOutputFile=... input.ai` |
| E12 | Artifex / Ghostscript blog — Optimizing PDFs | https://ghostscript.com/blog/optimizing-pdfs.html | web_extract | pdfwrite re-interprets input; output is not a byte-wise clone; metadata/private constructs may drop; still standard for “clean” PDFs. |
| E13 | Inkscape import AI guide (community) | https://salivity.github.io/inkscape/article/how-to-import-ai-files-directly-into-inkscape | web_extract | Requires Create PDF Compatible File; PDF Import Settings: Internal vs Cairo/Poppler; meshes/3D/brushes may flatten. |
| E14 | Inkscape membership blog — AI Extension 2023 | https://inkscape.org/*membership/blog/march-ai-extension-2023/ | web_search snippet | Project goal: high-fidelity import from embedded AI datastream beyond PDF; implies PDF path is current practical path. |
| E15 | Scribus wiki — Import vectors | https://wiki.scribus.net/canvas/Help:Manual_Importvectors | web_extract | PDF-based AI opened via File>Open can import AI/PDF layers; EPS widely supported. |
| E16 | Inkscape wiki — PDF Tools | https://wiki.inkscape.org/wiki/PDF_Tools | web_extract | Catalog: Ghostscript, Poppler, pstoedit, Scribus, ill2svg (older PS AI), etc. |
| E17 | LibreOffice Draw Guide — AI format note | https://books.libreoffice.org/en/DG76/DG7606-EditingImages.html | web_search | AI described as proprietary single-page vector in EPS or PDF formats; Draw has import filters among many graphics formats. |
| E18 | Parent kanban handoff t_bdf343f7 | task body | kanban_show | 2,596 AI; 1,164 pdf_compatible; 1,432 needs_conversion; folders Sulemon/Pierre/Desktop Archives. |

---

## findings

### 1. Format anatomy (operator view)

```
Legacy AI (≤ 8)          Modern AI (≥ 9)
─────────────────        ────────────────────────────────────────────
%!PS-Adobe-...           %PDF-1.x
PostScript body          PDF objects + page content stream(s)
EPS-like / DSC           + compressed AI private data (AIPrivateData)
                         + optional XMP (illustrator:Type)
                         + %AI5_FileFormat N markers inside
```

**Create PDF Compatible File = ON (default):** PDF page content ≈ artwork (artboards; pasteboard often excluded).  
**OFF:** PDF page content = warning placard only; native data still in private section; **only Illustrator** (or a full reverse-engineer) opens real art.

Saving as `.pdf` with **Preserve Illustrator Editing Capabilities** is the inverse sandwich (PDF public + AI private). PDF/X presets disable private forks.

### 2. Detection heuristics (bash-friendly)

Do **not** trust extension alone. Recommended classifiers:

```bash
# header class
head -c 16 "$f" | tr -d '\0'

# strong negative for "PDF compatible artwork"
grep -a -m1 -F 'without PDF Content' "$f" && echo NON_PDF_COMPATIBLE_WARNING

# private data markers (preservation)
grep -a -o 'AIPrivateData\|AIPDFPrivateData' "$f" | sort | uniq -c

# legacy PS
head -c 20 "$f" | grep -q '%!PS-Adobe' && echo LIKELY_PS_AI

# PDF shell
head -c 8 "$f" | grep -q '%PDF-' && echo PDF_SHELL

# Illustrator format version comment (often present)
grep -a -m1 '%AI5_FileFormat' "$f"
```

**Practical labels for pipeline:**

| Label | Rule of thumb |
|---|---|
| `pdf_shell_with_art` | `%PDF-` AND NOT warning string AND `pdfinfo`/`pdftotext` shows non-warning content |
| `pdf_shell_warning_only` | `%PDF-` AND warning string (needs Illustrator) |
| `ps_legacy` | `%!PS-Adobe` / EPS-ish |
| `unknown_binary` | zip/CFB/other — investigate |

Parent’s **1,164 pdf_compatible** ≈ `pdf_shell_with_art`.  
Parent’s **1,432 needs_conversion** ≈ mix of `ps_legacy` + `pdf_shell_warning_only` + oddities — **re-triage on host** before burning CPU.

### 3. Tool matrix (Linux, no Adobe)

| Tool | Install (Fedora-ish) | Role | Strengths | Weaknesses | Headless batch? |
|---|---|---|---|---|---|
| **Ghostscript (`gs`)** | `dnf install ghostscript` | Primary converter PS/PDF/AI→PDF | Battle-tested; used by AI→PDF GitHub Action; handles many PS legacy files | Rebuilds content (not bit-identical); drops many private constructs; warning-only AI → useless PDF | **Yes** |
| **Poppler** (`pdfinfo`, `pdftotext`, `pdftocairo`) | `dnf install poppler-utils` | Validate + raster/SVG proof | Fast triage; PNG/SVG previews | Needs PDF content; won’t fix non-compatible AI | **Yes** |
| **Inkscape ≥1.x** | `dnf install inkscape` | Import AI/PDF, export PDF/SVG | Better interactive fix-ups; CLI export | Heavy; meshes/transparency limits; needs PDF compat; slow for 2.6k | Yes but slow |
| **pstoedit** | `dnf install pstoedit` | PS/PDF → other vectors via GS | Niche vector recovery | Plugin/quality uneven | Partial |
| **Scribus** | `dnf install scribus` | Import AI/PDF/EPS, export PDF | Layers on PDF-AI sometimes | GUI-first; scripting heavier | Weak |
| **LibreOffice Draw** | `dnf install libreoffice-draw` | Import AI (EPS/PDF forms) | Already common | Fidelity not specialized for AI | `--headless` possible, quality uncertain |
| **qpdf / mutool** | `dnf install qpdf mupdf-tools` | PDF surgery, extract, clean | Good when input already PDF-OK | Cannot create art from private data | Yes |
| **ImageMagick `convert`** | often present | Raster fallback | Last-resort preview | Rasterizes; poor vector PDF | Yes |
| **Affinity (commercial)** | proprietary | Open PDF-compatible AI | Good UI consumer | Not a Linux server batch tool; same PDF-stream limit | No (desktop) |
| **Adobe Illustrator** | commercial | Only full native reader | Recovers non-PDF-compatible; best fidelity | **Out of scope / unavailable** | N/A |

**This sandbox snapshot:** `convert` (ImageMagick) and `file` present; **`gs` / `inkscape` / `pdftocairo` not installed** in the Geraldo container — install on the **host** that holds the Dropbox trees.

### 4. Conversion recipes (copy-paste)

#### 4.1 Tier A — PDF-compatible AI → PDF (Ghostscript primary)

```bash
# High-quality general purpose
gs -dSAFER -dBATCH -dNOPAUSE -dQUIET \
  -sDEVICE=pdfwrite \
  -dCompatibilityLevel=1.6 \
  -dPDFSETTINGS=/prepress \
  -sOutputFile="out/${base}.pdf" \
  "in/${base}.ai"

# Faster / smaller screen proofs
gs -dSAFER -dBATCH -dNOPAUSE -dQUIET \
  -sDEVICE=pdfwrite \
  -dPDFSETTINGS=/ebook \
  -sOutputFile="out/${base}.pdf" \
  "in/${base}.ai"
```

**Optional “already PDF” fast path** (only after validation): if `pdfinfo "file.ai"` succeeds and page content ≠ warning, you may `cp`/`qpdf --stream-data=compress` to `.pdf`. Prefer GS rewrite when consumers choke on AI private tails.

#### 4.2 Validation / proof

```bash
pdfinfo "out/${base}.pdf"
pdftotext -f 1 -l 1 "out/${base}.pdf" - | head
# raster proof
pdftocairo -png -r 150 "out/${base}.pdf" "proofs/${base}"
# or
gs -dSAFER -dBATCH -dNOPAUSE -sDEVICE=png16m -r150 \
  -sOutputFile="proofs/${base}.png" "out/${base}.pdf"
```

**Reject** outputs whose text layer or raster is only the Illustrator warning placard.

#### 4.3 Tier B — Legacy PostScript AI

```bash
# Let GS interpret as PS
gs -dSAFER -dBATCH -dNOPAUSE \
  -sDEVICE=pdfwrite \
  -sOutputFile="out/${base}.pdf" \
  "in/${base}.ai"

# If that fails, try EPS-style wrapping or pstoedit
pstoedit -f pdf "in/${base}.ai" "out/${base}.pdf"
```

#### 4.4 Inkscape CLI (when GS PDF needs vector edit or SVG bridge)

Inkscape 1.x style:

```bash
inkscape "in/${base}.ai" \
  --export-type=pdf \
  --export-filename="out/${base}.pdf" \
  --export-area-drawing
```

Use when operators must fix fonts/paths interactively; not first pass for 2.6k files.

#### 4.5 Tier C — Non-PDF-compatible modern AI

```text
NO reliable open-source bulk fix.
Actions:
  1. Mark UNRECOVERABLE_WITHOUT_ILLUSTRATOR
  2. If any machine has Illustrator: batch re-save with Create PDF Compatible File ON
     (and consider disabling Save in Background per practitioner reports of PDF-stream bugs)
  3. Search sidecars: same basename .pdf / .eps / .svg / .png
```

### 5. Fidelity matrix (expectations)

| Asset feature | GS pdfwrite on PDF-AI | Inkscape import PDF-AI | Legacy PS via GS | Non-PDF-compat AI |
|---|---|---|---|---|
| Basic vectors / fills | Good | Good | Variable | **None** (warning) |
| Live text editability | Often outlined or subset-embedded; not AI live type | Keep as text **or** paths (import choice); font sub risk | Font dependency high | N/A |
| Layers | Often flattened in rewrite | Partial / groups; AI layers not guaranteed | Rare | N/A |
| Pasteboard objects | Often **missing** (never in PDF part) | Same | N/A | N/A |
| Gradients | Usually OK | Usually OK | Variable | N/A |
| Gradient meshes | Flattened / approximated | Approximated lattices | Poor | N/A |
| Transparency / blend modes | Mostly flattened or approximated | Known limitations | Poor | N/A |
| Spot / Pantone | Risk of convert-to-process | Risk | Risk | N/A |
| Linked images | Need embed or missing | Need embed | Need embed | N/A |
| Symbols / live effects / brushes | Expanded or lost | Expanded or lost | Lost | N/A |
| CMYK print intent | Use `/prepress`; verify ICC | Verify | Verify | N/A |
| True AI edit round-trip | **No** | **No** | **No** | Only Illustrator |

### 6. Decision tree (mapped to 1,164 / 1,432)

```
                    ┌─────────────┐
                    │ each .ai    │
                    └──────┬──────┘
                           ▼
                 header / warning scan
                           │
           ┌───────────────┼────────────────┬─────────────────┐
           ▼               ▼                ▼                 ▼
   pdf_shell_art     pdf_shell_warn     ps_legacy        other/corrupt
   (~1164 target)    (subset of 1432)   (subset 1432)
           │               │                │                 │
           ▼               ▼                ▼                 ▼
    GS pdfwrite      sidecar search    GS pdfwrite      quarantine
    + pdfinfo OK     else ILLUSTRATOR  + pstoedit FB     manual
    + PNG proof      QUEUE             + PNG proof
           │               │                │
           ▼               ▼                ▼
         DONE         blocked/OS       DONE or queue
```

**Throughput sketch (host with GS):**  
- 1,164 Tier A @ ~0.5–5 s/file → roughly **10–90 minutes** wall time single-threaded; easy to GNU `parallel`.  
- 1,432 Tier B/C: expect **high failure/quarantine rate** without Adobe; budget operator time for sidecar hunt + sampling.

**Batch skeleton:**

```bash
#!/usr/bin/env bash
set -euo pipefail
IN="$1"; OUT="$2"; PROOF="$3"
mkdir -p "$OUT" "$PROOF" "$OUT/quarantine" "$OUT/logs"
classify() {
  local f="$1"
  if grep -a -q -F 'without PDF Content' "$f"; then echo warn; return; fi
  if head -c 8 "$f" | grep -q '%PDF-'; then echo pdf; return; fi
  if head -c 12 "$f" | grep -q '%!PS-Adobe'; then echo ps; return; fi
  echo other
}
export -f classify
process() {
  local f="$1" base b class
  base=$(basename "$f" .ai); b=$(basename "$f")
  class=$(classify "$f")
  case "$class" in
    pdf|ps)
      if gs -dSAFER -dBATCH -dNOPAUSE -dQUIET -sDEVICE=pdfwrite \
          -dPDFSETTINGS=/prepress -sOutputFile="$OUT/$base.pdf" "$f" 2>"$OUT/logs/$base.gs.err"
      then
        if pdftotext -f 1 -l 1 "$OUT/$base.pdf" - 2>/dev/null | grep -q 'without PDF Content'; then
          mv "$OUT/$base.pdf" "$OUT/quarantine/$base.warn.pdf"
          echo "$f WARN_OUTPUT" >>"$OUT/logs/failures.txt"
        else
          pdftocairo -png -r 72 -singlefile "$OUT/$base.pdf" "$PROOF/$base" 2>/dev/null || true
          echo "$f OK $class" >>"$OUT/logs/ok.txt"
        fi
      else
        echo "$f GS_FAIL $class" >>"$OUT/logs/failures.txt"
        cp -n "$f" "$OUT/quarantine/" || true
      fi
      ;;
    warn|other)
      echo "$f $class" >>"$OUT/logs/needs_illustrator.txt"
      cp -n "$f" "$OUT/quarantine/" || true
      ;;
  esac
}
export -f process
find "$IN" -type f -iname '*.ai' -print0 | parallel -0 process {}
```

### 7. Archive-specific notes (Sulemon / Pierre / Desktop)

Without CSV mount, only handoff metadata applies:

- **Mixed dropbox + desktop archives** → expect duplicates, partial exports, and renamed extensions.
- Before converting, run **hash-based dedupe** (`sha256sum`) across the three roots to avoid triple work.
- Prefer discovering existing `.pdf`/`.eps` sidecars beside `.ai` for the 1,432 — often designers exported once.
- Keep originals immutable; write outputs under a separate tree with mirrored relative paths.

### 8. Licensing / safety

- Ghostscript: AGPL (Artifex) — fine for internal conversion; be aware if redistributing modified GS.
- Do not ship AI private data parsers scraped from Adobe binaries (legal/ToS risk); stick to public PDF/PS interpreters.
- `-dSAFER` on GS for untrusted archives.

---

## gaps

1. **Inventory attachments not readable in worker container** — could not recompute counts, path histograms, or sample headers from `inventory.csv` / `needs_conversion.txt`. Parent counts used as-is (E18).
2. **No live byte sampling** of Sulemon/Pierre/Desktop files from this sandbox — success-rate estimates for the 1,432 are not empirically measured.
3. **Exact mix inside needs_conversion** (PS legacy vs warning-PDF vs corrupt vs misclassified) unknown until host re-triage.
4. **Print requirements** (PDF/X, spot retention, bleed) not specified — `/prepress` is a generic high-quality default only.
5. **Inkscape private-data importer** status is research/ongoing historically; not a production bulk path as of sources reviewed.
6. **PRONOM fmt/558 etc.** useful for archives but not required for conversion pipeline.
7. This container lacks `gs`/`inkscape`/`pdftocairo` — recipes are host-targeted.

---

## confidence

```yaml
level: medium
rationale: >
  Format dual-structure, Adobe option semantics, and non-Adobe PDF-stream
  limitation are triangulated across Adobe Help, CreativePro, Affinity,
  VectorCraft, Inkscape, and preservation write-ups (high confidence).
  Ghostscript as primary CLI converter is confirmed by vendor docs and
  production GitHub Action (high). Archive-specific throughput and the
  recoverable fraction of 1,432 needs_conversion remain medium/low until
  host-side header sampling and pilot conversion on real files.
```

### per-area

| Area | Level |
|---|---|
| AI dual-structure / PDF compatible option | high |
| Detection heuristics | high |
| GS/Inkscape/Scribus capabilities | high–medium |
| Fidelity edge cases (spot, mesh, pasteboard) | medium–high |
| This archive’s 1432 recoverable % | low (blocked on samples) |

---

## recommended_next_actions

1. **Host ops (Su/ops):** mount or copy `inventory.csv` + `needs_conversion.txt` into worker workspace **or** run classification script on host paths; attach a `triage_summary.json` (counts by `pdf|ps|warn|other` per root).
2. **Pilot 50+50:** 50 random `pdf_compatible` + 50 random `needs_conversion` through GS recipe; record OK/warn/fail + visual spot-check.
3. **Install on conversion host:** `ghostscript poppler-utils inkscape parallel` (optional `pstoedit qpdf`).
4. **Run Tier A bulk** on the 1,164 after pilot sign-off; mirror directory structure; store PNG proofs at 72–150 dpi for QA sampling.
5. **For failures + 1,432:** sidecar search → Illustrator re-save queue → mark residual `UNRECOVERABLE_WITHOUT_ILLUSTRATOR`.
6. **If print shop delivery:** define PDF/X target and re-run GS with appropriate PDF/A or PDF/X workflow (may require color profiles); do not assume spot colors survive.
7. **Optional follow-up card:** implement production converter service once pilot metrics known (assignee: devops/su — not research).

---

## appendix A — quick reference commands

```bash
# classify one file
python3 - <<'PY'
import sys
p=sys.argv[1]
data=open(p,'rb').read(2_000_000)  # first 2MB enough for markers
head=data[:64]
print('head',head[:20])
print('pdf', data.startswith(b'%PDF-'))
print('ps', head.startswith(b'%!PS-Adobe'))
print('warn', b'without PDF Content' in data)
print('AIPrivate', b'/AIPrivateData' in data)
print('AIPDFPrivate', b'/AIPDFPrivateData' in data)
PY
file.ai

# pdf-compatible → pdf
gs -dSAFER -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -dPDFSETTINGS=/prepress \
  -sOutputFile=out.pdf in.ai
```

## appendix B — what “success” means for this project

**In scope success:** openable multi-consumer PDF that visually matches artboard artwork for PDF-compatible sources; structured quarantine list for the rest.  
**Out of scope:** full layer-preserving, spot-perfect, live-type AI clone without Adobe.

---

## appendix C — provenance stamp

| Step | Path |
|---|---|
| Adobe save options | web_extract helpx.adobe.com |
| Dual-structure explainer | web_extract note.com/monokano + creativepro |
| Preservation / headers | web_extract preservation.tylerthorsted.com |
| GS usage | web_extract ghostscript.readthedocs + github ai-to-pdf-action |
| Inkscape | web_extract wiki.inkscape.org + salivity guide |
| Affinity limit | web_extract affinity.studio/help/import-ai |
| Inventory | parent task body only (attachments not mounted) |

---

*End of report — t_bdf343f7*
