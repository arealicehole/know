# Streetwear × Cannabis DTF Style & Motif Bible
**Project:** Trapper Dan / Print Junkie design packs  
**Task:** t_1eefb647  
**Date:** 2026-10-03  
**Model target:** Qwen-Image-2.1 (prompt kit also notes Qwen-Image-2512 community practices)  
**Legal stance:** Parody *energy* only. No real Supreme / Stüssy / Bape / Carhartt / Thrasher marks, wordmarks, or logo clones. Original wording only.

---

## objective
A durable, promptable style system for **streetwear × cannabis enthusiast** full-front **DTF** tee graphics: controlled variation, unmistakable DTF isolation grammar (anti-AOP), ≥12 style families, strain→style matrix, and machine-usable prompt modules.

## summary
Streetwear graphic culture is a **signal language** (subculture type, density, placement, palette), not a pile of cannabis clichés.[2][7] DTF is an **opaque transfer with white underbase** that demands hard edges, transparent PNG, 300 DPI at size, and **no soft fade-to-transparent**—the opposite of all-over print / sublimation fill art.[1][3][4][5] This bible locks DTF layout grammar, 12 visual families with energy-only references, motif + type + color systems, strain aesthetic mapping (classics through 2020s–2026 candy-gas), pack structure, and a Qwen prompt kit (locked prefix/negative + family modules + variation levers + ≥20 full prompts).

---

## 1. Audience / culture map

| Tribe | Who | Visual hunger | Default family lean | Avoid |
|---|---|---|---|---|
| **Classic head** | Legacy smokers, landrace lore, reggae/Rasta adjacent, “old school gas” | Heritage leaf, roots type, earth greens, travel-poster sincerity | reggae_roots, psych_70s, workwear_patch | Neon candy overload, pure chrome Y2K |
| **Hype / drop culture** | 16–24 trap + SoundCloud + resale-aware; stacks AF1/Jordan energy | Stacked type, box-plaque, scarcity flex, clean center chest | box_logo_hype, luxury_monogram, camo_plaque | Tiny left-chest corporate marks; full-bleed AOP |
| **Reggae roots** | Caribbean diaspora + one-love lifestyle | Rasta triad, lion energy, sun/leaf, script + block mix | reggae_roots, mascot_beast | Fake celebrity likeness; sacred symbol abuse |
| **Skate** | Board/graphic tee heritage, loose silhouette | Flame, wheel, handstyle, distressed ink, anti-polish | skate_flame, handstyle_script, punk_grunge | Luxury serifs, monogram denseness |
| **Luxury-street** | Elevated basics, quiet flex | Controlled mono/metallic, monogram field *as plaque not AOP*, serif + grotesque | luxury_monogram, varsity_crest | Loud neon, sticker-bomb chaos |
| **Workwear** | Trades + utility aesthetic crossover | Patch badges, canvas texture cues, stamped type, muted earth | workwear_patch, camo_plaque | Gloss chrome, candy pastels |
| **Y2K candy** | Early-2000s nostalgia, metallics, pop saturation | Chrome, star, bubble type, pink/lime/cobalt candy | y2k_chrome_candy, box_logo_hype | Heavy blackletter-only, workwear drab |
| **Punk/DIY** | Anti-polish, zine, Xerox | Torn edges *as solid shapes*, ransom collage (hard masks), high contrast B/W + one accent | punk_grunge, handstyle_script | Soft airbrush gradients, luxury gold |

**Trapper Dan audience anchor (prior research, May 2026):** primary 16–24, male-skew, $30–75K HHI band, dual tribes SoundCloud-rap + hypebeast; stack with utility/workwear and sneaker culture.[U1] Phoenix underserved for trap streetwear hometown hero.[U2]

---

## 2. DTF layout grammar (NON-NEGOTIABLE)

### 2.1 What DTF is (vs AOP / sublimation)
| | **DTF transfer** | **AOP / sublimation fill** |
|---|---|---|
| Object | Isolated graphic pressed onto garment | Dye/print covering panels or whole garment |
| Background | **Must be transparent** — fabric shows around art | Art *is* the surface |
| White | RIP builds **white underbase** under opaque pixels | No underbase; light poly only for sub |
| Edge | Hard silhouette; soft alpha → **white halo** | Soft fades OK / desired |
| Feel | Thin film layer on fabric | Zero hand (sub) / full coverage (AOP) |
| Design intent | **Chest/back hit** with white margins | Edge-to-edge pattern |

DTF = print on film + powder adhesive + heat press peel; white underbase enables any garment color.[1][3][4] Sublimation dyes into light polyester only and has zero hand—different product category.[4][5]

### 2.2 File & edge rules (prompt + post)
1. **Transparent background only** — never white/colored artboard; white bg becomes a printed rectangle.[1]
2. **300 DPI at final print size** — e.g. 12″ front → ≥3600 px wide.[1]
3. **Hard edges** — no semi-transparent fringe, no feather, no outer glow to 0% alpha (white halo / powder fail).[1][3]
4. **Fades:** convert to **halftone** or trap inside a hard shape; do not fade to transparency.[3]
5. **Line weight:** keep strokes bold; thin hairlines and tiny text fail powder hold (industry guidance ~≥0.35 mm / ~1 pt practical floor; prefer thicker for AI gens).[3][6]
6. **RGB / sRGB** for design → RIP; avoid designing in CMYK for DTF workflows that expect RGB.[1][3]
7. **Trim canvas to artwork** — no huge empty transparent padding that confuses placement.[1]
8. **Common front sizes:** left chest 3–4″; standard full front **10–12″**; oversized ~12–14″; back up to ~14″.[1]
9. **Gang sheet ops (shop):** ~0.25–0.5″ gutters between designs; outer safe margin on film.[6][8]

### 2.3 Silhouette rules for *generation* (anti-AOP)
**Always say in prompts:**
- `isolated chest graphic`, `centered composition`, `single motif cluster`
- `transparent background`, `clean cut edge`, `hard silhouette`, `no background fill`
- `ample empty margin around artwork`, `not all-over print`, `not edge-to-edge pattern`
- `DTF transfer ready`, `sticker-like transfer shape` (metaphor for isolation—not a sticker product)

**Never say (or put in negatives):**
- all-over print, seamless pattern tile, full-bleed, wraparound, garment mockup fill, model wearing shirt (unless separate mockup job), fabric texture as background field, infinite repeating monogram across frame

### 2.4 DTF do / don't (operators)
| DO | DON'T |
|---|---|
| One hero mass with clear outer contour | Scattered sprites to canvas corners |
| Solid fills + controlled halftone interiors | Soft smoke dissolving into alpha |
| 2–5 color discipline (plus white) for most families | 20-color photo soup without posterization plan |
| Knockouts that are true holes (shirt shows) | Grey “fake transparent” pixels |
| Readable type at thumbnail | Micro legal text as decoration |
| Original wordmarks | Real brand boxes / ape heads / workwear logos |

---

## 3. Visual style families (≥12)

Energy references only — **describe grammar, never clone marks.**

### F01 — box_logo_hype
- **Energy:** Scarcity flex; rectangular plaque; stacked hierarchy; instant thumbnail read (box-logo *grammar*, not any brand).[7][9]
- **Palette:** High-contrast pairs — red/white, black/white, forest/cream, purple/black; max 3 colors.
- **Type:** Heavy grotesque / condensed sans; slight italic OK; all-caps short word (4–10 letters).
- **Motifs:** Rectangle plaque, bar stack, small secondary line, optional tiny leaf *as* bullet not photo.
- **Density:** Low–medium; lots of negative space *inside* the transfer bounds.
- **DTF silhouette:** Single rounded or sharp rectangle + optional bottom bar; no corner ornaments floating away.
- **Do:** Perfect alignment, optical center, thick counters. **Don't:** Futura-in-red-box clones; trademarked slogans.
- **Prompt core:** `centered rectangular box logo plaque, bold condensed sans all-caps wordmark in quotes, high contrast two-color fill, thick border, isolated chest graphic, hard edges, transparent background`

### F02 — handstyle_script
- **Energy:** Tag as signature; flow, slant, one flourish; street lettering anatomy.[10][11]
- **Palette:** Black/white + one accent (gold, lime, blood red) OR single color with outline.
- **Type:** Custom script / marker handstyle; consistent slant; 1 finish move only.
- **Motifs:** Underline slash, crown dots, small leaf accent, drips as *solid* teardrops not soft alpha.
- **Density:** Medium; lettering is the art.
- **DTF:** One continuous word cluster; outline stroke ≥ medium weight; no hairline connections.
- **Do:** Rhythm + readability test at small size. **Don't:** Illegible wildstyle for retail SKUs; fake celebrity tags.
- **Prompt core:** `original graffiti handstyle script wordmark, consistent slant, thick marker stroke, one flourish tail, outline and fill, isolated, hard silhouette, transparent background`

### F03 — varsity_crest
- **Energy:** Chenille/letterman grammar — shield, arched type, year, twin icons.[2][7]
- **Palette:** Navy/gold, crimson/cream, forest/gold, black/silver.
- **Type:** Chunk varsity block serif + arched banner sans.
- **Motifs:** Shield/crest, twin leaves or twin lions (original), stars, establishment year.
- **Density:** Medium-high but **bounded** by crest edge.
- **DTF:** Crest is the cut path; no rays extending softly.
- **Do:** Symmetry, thick borders. **Don't:** Real university marks; NFL/NBA lockups.
- **Prompt core:** `varsity athletic crest shield, arched banner text in quotes, twin cannabis leaf supporters original, bold outline, collegiate colors, isolated badge, hard edge, transparent background`

### F04 — workwear_patch
- **Energy:** Union label / chore coat / stamped utility.[2][7]
- **Palette:** Ochre, olive, brick, navy, cream, black — muted.
- **Type:** Condensed industrial sans, stamped serif small caps.
- **Motifs:** Oval/octagon patch, dashed stitch border, bolt, wrench *as abstract*, lot number, “DEPT” lines (original).
- **Density:** Low–medium; badge not mural.
- **DTF:** Patch-shaped silhouette; stitch ring as solid dashes.
- **Do:** Worn-ink *texture inside hard mask*. **Don't:** Real Carhartt/Dickies logos.
- **Prompt core:** `vintage workwear woven-style patch badge, oval with dashed stitch border, industrial condensed type in quotes, muted earth tones, isolated patch, hard cut edge, transparent background`

### F05 — camo_plaque (NOT AOP)
- **Energy:** Military/utilitarian trap — camo **inside a shape**, not shirt-fill.[2]
- **Palette:** Classic woodland, desert, urban grey, or night camo; 3–5 camo colors max.
- **Type:** Stencil sans over or under plaque.
- **Motifs:** Rounded rectangle / dog-tag / shield filled with camo; small leaf stencil knockout.
- **Density:** High *inside* plaque only.
- **DTF:** Outer contour hard; camo stops at border; **no** full-frame camo field.
- **Do:** Camo as fill. **Don't:** Seamless camo AOP prompts.
- **Prompt core:** `camouflage pattern fill confined inside a rounded rectangular plaque with thick border, stencil wordmark in quotes, not all-over print, isolated chest graphic, hard silhouette, transparent background`

### F06 — skate_flame
- **Energy:** Board graphic / thrash / late-90s–00s skate flame & wheel language.[2]
- **Palette:** Black base + orange/yellow flame; or teal/pink alt flame; high contrast.
- **Type:** Soft sans or sliced italic; sometimes no type.
- **Motifs:** Twin flames, wing flash, wheel, bolt, barbed forms as solids.
- **Density:** Medium; directional motion.
- **DTF:** Flame tips must be **closed solid shapes**, not wispy alpha smoke.
- **Do:** Bold cartoon fire. **Don't:** Real Thrasher logo flame lockup.
- **Prompt core:** `bold skate graphic twin flame motif, solid cel-shaded flames, optional wheel icon, thick black outlines, high contrast, isolated, hard edges, transparent background`

### F07 — luxury_monogram
- **Energy:** Quiet flex; monogram tile **as bordered field or scarf plaque**, not infinite AOP.[2]
- **Palette:** Black/gold, cream/brown, navy/silver, monochrome.
- **Type:** Interlocking monogram letters (original initials only) + small clean sans.
- **Motifs:** Diamond tile monogram inside rounded rect; thin double-line frame; tiny leaf as period.
- **Density:** Medium inside frame; generous outer margin.
- **DTF:** Frame = cut edge; monogram does not bleed off artboard.
- **Do:** Optical balance. **Don't:** LV/Gucci pattern clones; brand monograms.
- **Prompt core:** `luxury streetwear monogram plaque, original interlocking initials pattern inside double-line frame, gold on black, not seamless all-over, isolated chest graphic, hard border, transparent background`

### F08 — y2k_chrome_candy
- **Energy:** Early digital nostalgia — chrome, stars, bubble type, candy neons.[2][12]
- **Palette:** Hot pink, lime, cobalt, chrome silver, glossy black, soft purple.
- **Type:** Bubble fat sans, Y2K techno display, outlined chrome type.
- **Motifs:** Chrome leaf, star bursts (solid), smiley *original*, loading bars, glitter as dots not soft sparkle alpha.
- **Density:** Medium-high but clustered center.
- **DTF:** Chrome as cel + highlight shapes; no photographic bokeh background.
- **Do:** One loud anchor. **Don't:** Full UI screenshot AOP.
- **Prompt core:** `Y2K streetwear chrome candy graphic, bubbly chrome wordmark in quotes, neon pink and lime accents, star motifs, glossy highlights as solid shapes, isolated center graphic, hard cut edge, transparent background`

### F09 — reggae_roots
- **Energy:** Roots reggae / Rasta color grammar + uplift; landrace sincerity (not celebrity likeness).
- **Palette:** Red / gold / green triad + black/cream; occasional Ethiopian flag-inspired bands (respectful, non-parodic sacred misuse).
- **Type:** Rasta-adjacent display, friendly bold sans, or warm script.
- **Motifs:** Sun disc, original lion (not copyrighted marks), leaf, sound-system bars, concentric rings.
- **Density:** Medium; circular or arched compositions work well.
- **DTF:** Circle/sun badge silhouette preferred.
- **Do:** Joyful geometry. **Don't:** Real artist faces/logos; lazy “Rasta clipart” stereotypes.
- **Prompt core:** `reggae roots chest badge, red gold green black palette, sun disc and original cannabis leaf, bold arched type in quotes, circular hard silhouette, isolated, transparent background`

### F10 — psych_70s_travel
- **Energy:** 1970s psychedelic poster / travel poster / Blacklight-adjacent, but **solid posterized** for DTF.
- **Palette:** Avocado, mustard, burnt orange, violet, cream, teal.
- **Type:** Curvy display, stacked wavy baselines, split fountain *simulated with hard bands*.
- **Motifs:** Ornate leaf mandala, mountains, highway sun, ornate frames.
- **Density:** High ornamental but single poster rectangle.
- **DTF:** Poster frame = edge; no bleed-off mandala infinity.
- **Do:** Posterize gradients to bands. **Don't:** Soft airbrush-only without poster edges.
- **Prompt core:** `1970s psychedelic travel poster style cannabis graphic, ornate frame, wavy display type in quotes, posterized color bands, avocado mustard violet palette, isolated poster plaque, hard rectangle edge, transparent background`

### F11 — punk_grunge_diy
- **Energy:** Zine, Xerox, ransom, torn flyer — authenticity via imperfect solids.[2]
- **Palette:** B/W + one neon or blood red; newsprint cream.
- **Type:** Mixed weights, stamped, roughly aligned but **each letter solid**.
- **Motifs:** Safety pin (abstract), cracks as shapes, barcode parody (original numbers), leaf stamped.
- **Density:** High chaos *inside* a rough rectangle or torn-paper hard mask.
- **DTF:** “Torn paper” = jagged **opaque** polygon, not frayed alpha.
- **Do:** High contrast. **Don't:** Real band logos; unreadable mess for main SKU.
- **Prompt core:** `punk DIY xerox zine graphic, high contrast black white red, ransom-style original lettering in quotes inside jagged paper shape, grunge texture inside hard mask, isolated, transparent background`

### F12 — mascot_beast
- **Energy:** Character IP — beast/bud buddy/lion/dragon original mascot as brand engine.[2]
- **Palette:** Family-dependent; usually 3–5 cel colors + thick ink outline.
- **Type:** Optional arched name under mascot.
- **Motifs:** Anthropomorphic leaf-beast, smiling ogre bud, armored wolf with leaf crest — **original designs only**.
- **Density:** Character-centered medium.
- **DTF:** Character contour is cut path; bold outlines; no wispy fur alpha.
- **Do:** Cel animation read. **Don't:** Existing IP (cartoons, sports mascots, ape head brands).
- **Prompt core:** `original streetwear mascot character, bold comic ink outlines, cel shading, cannabis-inspired creature not copyrighted IP, optional arched wordmark in quotes, isolated full mascot, hard silhouette, transparent background`

---

## 4. Motif library (cannabis + street)

### Cannabis motifs (use with intent)
| Motif | Good use | Cliché warning |
|---|---|---|
| Single iconic leaf | Crest supporter, bullet, monogram period | Giant leaf + “420” only = lazy |
| Bud cluster (stylized) | Mascot body, badge fill | Photoreal nugs on every SKU |
| Trichome sparkle | Dot highlights inside hard shapes | Soft glitter alpha haze |
| Joint / blunt silhouette | Punk/skate only if brand-OK | Default “stoner starter pack” |
| Grinder / papers | Workwear utility icons | Over-literal product shot |
| Mountain / farm sun | Psych travel, roots | Generic “farm fresh” stock |
| Smoke wisps | **Halftone or solid ribbons only** | Soft smoke to transparent (DTF fail) |
| Terp flavor icons (candy, gelato swirl, marker, pine) | Strain series | Literal food brands/logos |

### Street motifs
| Motif | Family fit | Notes |
|---|---|---|
| Box / plaque | F01 F05 F07 | Core DTF friend |
| Crest / shield | F03 F09 | |
| Flame | F06 F08 | Solid tips |
| Camo fill | F05 | Confined |
| Handstyle | F02 F11 | |
| Stars / chrome | F08 | |
| Stitch / patch | F04 | |
| Barcode / xerox | F11 | Original data |
| Sound bars / sun | F09 | |
| Monogram tile | F07 | Framed only |

### Cliché ban list (default reject)
- Leaf + marijuana word in Comic Sans  
- Red eyes stoner face stock  
- Peace sign + leaf lazy combo without craft  
- “It’s 4:20 somewhere” without design system  
- Fake federal warning labels as whole design  
- Real brand parody so close it confuses source  

---

## 5. Typography system (print-safe)

| Role | Spec | DTF note |
|---|---|---|
| Hero display | Heavy grotesque, varsity slab, blackletter (sparingly), bubble Y2K | Thick stems; open counters |
| Script | Handstyle / brush with outline | Outline ≥ fill legibility |
| Support | Condensed industrial sans | Small sizes still bold |
| Micro | Avoid < ~12–14 pt equivalent at print | Prefer omit micro legal |

**Rules:** Quote exact strings in prompts (`"TRAPPER"`).[13] Prefer short stack (1–3 lines). Outline fonts conceptually (no hairline serifs). Blackletter: watch filled counters on dark underbase.[7]

---

## 6. Color systems (print-safe RGB intent)

| System | Hex intent (approx) | Use |
|---|---|---|
| **Hype mono** | #000000 #FFFFFF #E10600 | F01 |
| **Gas OG** | #0B1A0F #3D5C3A #C7B299 #D4A017 | Classic head |
| **Candy Runtz** | #7B2D8E #FF4DC4 #A8E600 #F5F5F5 | F08 candy-gas |
| **Gelato cream** | #5B2C6F #F7C6C7 #F4E1C1 #2E2E2E | Dessert strains |
| **Marker night** | #1A1020 #5E2A84 #FF6A00 #E8E8E8 | Permanent Marker vibe |
| **LCG jewel** | #2F0A3A #1F4D2A #FF7A1A #F2F2F2 | Lemon Cherry Gelato |
| **Roots triad** | #C8102E #FCD116 #007A3D #111111 | F09 |
| **Work drab** | #5C4033 #6B7B3C #C4A574 #1C1C1C | F04 |
| **Psych 70s** | #568203 #E1AD01 #CC5500 #6F2DA8 #F5E6C8 | F10 |
| **Chrome** | #C0C0C0 #5A5A5A #FFFFFF #0A0A0A + 1 neon | F08 |

**Print note:** Slightly oversaturate on-screen; RIP/underbase shifts neons and greens.[3] Prefer solid brandable swatches over 16-bit photo noise for core SKUs.

---

## 7. Qwen-Image prompt kit

### 7.1 Locked prefix (always first)
```
Subject: isolated DTF chest transfer graphic for streetwear t-shirt
Medium: bold vector-like apparel graphic, cel shading or clean flat print design
Background: pure transparent background, no scene, no fabric, no mockup
Edges: hard silhouette, clean cut edge, no feathering, no outer glow
Composition: centered single graphic, generous empty margin, not all-over print, not seamless pattern
```

### 7.2 Locked negative (always)
```
all-over print, AOP, seamless pattern, full bleed, edge-to-edge, repeating tile, garment mockup, model wearing shirt, photograph of t-shirt, fabric texture background, soft smoke fading to transparent, feathered edges, outer glow, white square background, grey checkerboard, blurry, lowres, watermark, real brand logos, Supreme box logo, Stussy, Bape, Carhartt, Thrasher, Nike, Adidas, copyrighted cartoon characters, extra limbs, messy microtext, illegible letters
```

### 7.3 Assembly order (Qwen-friendly)
1. Locked prefix  
2. Family module  
3. Strain color/motif module  
4. Exact text in `"quotes"` with font style  
5. Density + palette  
6. DTF edge reminders  
7. Locked negative  

Community practice for Qwen-Image family: **structure over long narrative**, subject first, quote text, keep concise, use negatives.[13]

### 7.4 Variation levers (controlled packs)
| Lever | Values | Effect |
|---|---|---|
| `pack_role` | hero / minimal / parody / crest / mascot | Composition budget |
| `density` | sparse / medium / packed | Motif count |
| `palette_mode` | mono / duo / triad / candy / roots / psych | Color system |
| `type_mode` | none / single_word / stacked / arched / handstyle | Text load |
| `edge_mode` | sharp_rect / badge / circle / jagged_diy / character_contour | Cut path |
| `finish` | flat / cel / halftone_shade / chrome_cel / distressed_inside_mask | Texture |
| `scale_intent` | left_chest / standard_front / oversized_front | Detail level |
| `parody_energy` | none / sports_broadcast / warning_label_gag / luxury_flex / tourist_poster | Tone (original copy only) |
| `strain_lock` | (name) | Palette + motif bias |

---

## 8. Strain → style mapping matrix

Visual/cultural cues from public strain writeups; **palette is design fiction layered on described bag appeal / lore**, not medical claims.

| Strain | Era / notes | Bag-appeal / lore cues | Primary families | Palette system | Motif bias |
|---|---|---|---|---|---|
| **OG Kush** | Classic 2000s SoCal gas | Lemon-pine-fuel lore; dark green frost archetype[15] | F01, F04, F05 | Gas OG | Pine sprig, fuel drop (abstract), plaque |
| **Lamb's Bread / Breath** | Jamaican landrace sativa lore; light green “wool” buds; Rasta culture association[16] | Roots, uplift | F09, F10 | Roots triad + light sage | Sun, spear cola stylized, lion original |
| **Gelato** | Cookie-family dessert; green/purple + orange pistils; lifestyle branding era[17] | Creamy dessert | F07, F08, F12 | Gelato cream | Swirl, spoon abstract, soft purple leaf |
| **Runtz** | Zkittlez × Gelato; candy-gas; purple/lime/magenta bag appeal; 2020 SOTY cultural peak[14][17] | Candy hype | F08, F01, F12 | Candy Runtz | Candy gems (original), chrome leaf |
| **Permanent Marker** | Biscotti × Jealousy × Sherb Bx; dark purple frost; marker-ink nose; 2023 SOTY[18][19] | Night candy-gas | F01, F07, F11 | Marker night | Cap abstract, stripe, deep purple bud stylized |
| **Lemon Cherry Gelato (LCG)** | Sherbet × GSC story; purple/green/orange jewel frost; loud lemon-cherry-cream[20] | Jewel candy | F08, F03, F12 | LCG jewel | Cherry + lemon discs (original), frost dots |
| **White Runtz / Pink Runtz** | Runtz phenos | Creamier / pinker candy | F08, F01 | Candy variants | Pastel chrome |
| **Jealousy** | Gelato41 × Sherb; gas-cream parent to Marker[18] | Frost + gas | F07, F05 | Marker night lighter | Diamond frost plaque |
| **Biscotti** | Gelato25 × SF OG | Cookie dough + gas | F04, F07 | Gelato + gas | Cookie geometry abstract |
| **Zkittlez** | Candy fruit parent to Runtz[14] | Rainbow candy | F08, F10 | Candy rainbow restrained | Fruit gems original |
| **Sour Diesel** | East Coast fuel classic | Neon acid green + black | F06, F11 | Hype mono + acid green | Fuel nozzle abstract, flame |
| **Northern Lights** | Legacy indica night | Deep green/violet stars | F10, F09 | Psych + night | Aurora bands posterized |
| **Blue Dream** | Legacy hybrid | Blue-green dreamy but DTF-solid | F10, F02 | Psych teal/blue | Wave script |
| **Girl Scout Cookies / GSC** | Dessert cornerstone | Thin mint / dessert | F07, F03 | Cream + forest | Crest dessert |
| **Wedding Cake** | Modern dessert | Vanilla icing white/purple | F07, F01 | Cream mono + purple | Cake tier abstract |
| **GMO / Garlic Cookies** | Funk savory counter-candy | Olive, mustard, black | F04, F11 | Work drab + funk | Skull-free; garlic bulb abstract optional |
| **Hash Burger** | Savory wave contrast to candy[19] | Brown, mustard, black | F04, F11 | Work drab | Burger abstract *careful cliché* |
| **London Pound Cake** | Dessert UK-US | Gold, cream, purple | F07, F03 | Luxury cream | Crest |
| **Gary Payton** | Modern hype athlete-name energy — **use original athletic crest, no real likeness/IP** | Sport hype | F03, F01 | Hype mono + royal | Varsity number original |
| **Ice Cream Cake** | Dessert | Pastel cream | F08, F12 | Gelato cream | Mascot scoop beast |
| **Mac 1** | Modern boutique | Alien green, cream | F01, F07 | Mono green luxury | Minimal plaque |
| **Super Boof / contemporary 2024–26 hype names** | Menu hype | Loud candy + gas | F08, F01 | Candy + marker | Stack type |
| **Tropicana Cookies / orange strains** | Citrus | Orange, green, cream | F06, F08 | Orange candy | Citrus wheel solid |
| **Purple Punch** | Grape dessert | Violet heavy | F10, F09 | Psych violet | Circle badge |

**Rule:** Strain name in `"quotes"` as type when SKU is strain-led; else encode strain only via palette/motif.

---

## 9. Design pack structure

### 9.1 Roles per strain or drop
| Role | Goal | Typical family | Type load |
|---|---|---|---|
| **Hero** | Main seller, max signal | F01/F03/F08/F12 | 1–2 lines |
| **Minimal** | Everyday tee | F01/F07 sparse | 1 word or icon |
| **Parody energy** | Humor without trademark | F11/F01 | Original slogan |
| **Crest** | Collegiate collectible | F03/F09 | Arched + year |
| **Mascot** | Character IP seed | F12 | Name under |

### 9.2 File naming
```
TD_{strainSlug}_{familyId}_{packRole}_{palette}_{vN}.png
# example
TD_permanentMarker_F01_hero_markerNight_v03.png
TD_lcg_F08_mascot_lcgJewel_v01.png
```
Companion prompt: same basename `.txt` or sidecars in pack YAML.

### 9.3 Review checklist (human + PJ)
- [ ] Transparent bg; no white box  
- [ ] Hard outer silhouette; no feather halo  
- [ ] Readable at phone thumbnail  
- [ ] No real brand marks / confusingly similar logos  
- [ ] Strain palette matches matrix (if strain-locked)  
- [ ] Type quoted strings spelled correctly  
- [ ] Not AOP/seamless  
- [ ] Line weights print-safe  
- [ ] Margins: graphic is a unit, not corner confetti  
- [ ] Pack has hero+minimal at minimum  

---

## 10. Example prompt cores (family) — see also `prompt_modules.yaml` full list (≥20)

1. F01 OG: box plaque `"OG"` gas palette  
2. F02 handstyle `"BREATH"`  
3. F03 LCG crest  
4. F04 workwear `"GAS DEPT"`  
5. F05 camo plaque `"TRAP"`  
6. F06 skate flame leaf  
7. F07 monogram `"TD"`  
8. F08 Runtz chrome candy `"RUNTZ"`  
9. F09 Lamb's Bread sun badge  
10. F10 psych travel `"NORTHERN"`  
11. F11 punk `"MARKER"`  
12. F12 mascot Gelato beast  

(Full 24 assembled prompts live in YAML `example_prompts`.)

---

## claims (structured)

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | DTF requires transparent PNG, ~300 DPI at size, RGB workflow, and hard edges; semi-transparent edges cause white halos because of white underbase | E1 E3 | high | supported |
| C2 | Soft fades to transparency are a DTF failure mode; use hard edges or halftone | E3 | high | supported |
| C3 | DTF is opaque transfer suitable for cotton/dark garments; sublimation is dye-in-poly light garments — different design grammar than AOP fill | E4 E5 | high | supported |
| C4 | Standard full-front transfer widths commonly ~10–12″; left chest ~3–4″ | E1 | high | supported |
| C5 | Streetwear aesthetics cluster into recognizable families (skate, workwear, Y2K, luxury, grunge, etc.) usable as design signals | E2 E7 | medium | supported |
| C6 | Handstyle depends on flow, slant consistency, weight, limited flourishes | E10 E11 | medium | supported |
| C7 | Qwen-Image family responds well to structured prompts, quoted text, concise specs, negatives | E13 | medium | supported |
| C8 | Runtz associated with candy-gas, colorful purple/lime bag appeal, Zkittlez×Gelato | E14 | medium | supported |
| C9 | Permanent Marker associated with dark frost, purple hues, marker-ink nose, dessert-gas lineage, 2023 Leafly SOTY mentions | E18 E19 | medium | supported |
| C10 | LCG associated with purple/green/orange jewel frost and lemon-cherry-cream sensory identity | E20 | medium | supported |
| C11 | Lamb's Bread/Breath framed as Jamaican landrace-type sativa with light green wool-like buds and Rasta cultural association in public sources | E16 | medium | partial |
| C12 | Trapper Dan demo leans 16–24 hype + SoundCloud trap (internal May 2026 research) | U1 | medium | supported (internal) |

---

## gaps
- Print-Junkie brief file and `aibox-dtf` sample outs **not mounted** in Geraldo Docker; brief reconstructed from kanban body.  
- Exact live Trapper Dan SKU strain roster beyond named set unknown — matrix covers brief names + common menu peers; extend when roster provided.  
- Qwen-**Image-2.1** exact card less documented than **2512** community guides; prompt kit uses shared Qwen-Image practices — validate on local 2.1.  
- RGB hexes are **design intent**, not ICC-calibrated shop profiles.  
- Cultural symbols (Rasta, religious) need brand-owner taste pass.  
- No primary interviews with TD customers in this run.

## confidence
- **level:** medium-high for DTF grammar + family taxonomy; medium for strain visual mapping; medium for Qwen-2.1 specifics  
- **rationale:** DTF rules triangulated across multiple 2025–2026 prep guides; streetwear families from editorial indexes; strain colors from public strain articles (not lab photography owned by us); model prompting from community Qwen-Image guides.

## recommended_next_actions
1. Print Junkie: load `prompt_modules.yaml`; generate 1 strain × 5 pack_roles smoke test on Qwen-Image-2.1.  
2. Calibrate negatives if model still emits mockups/AOP.  
3. Frank/TD: confirm strain roster + banned words/symbols.  
4. Add shop ICC notes post first press.  
5. Optional: skillize this folder for PJ auto-load.

## operator summary (Print Junkie — 10 lines)
1. Read DTF grammar §2 before any gen — isolation + hard edge + transparent bg.  
2. Pick family F01–F12; don’t freestyle density.  
3. Lock strain row from §8 for palette/motif.  
4. Assemble prompt: prefix + family + strain + `"TEXT"` + negative.  
5. Use variation levers for pack roles (hero/minimal/parody/crest/mascot).  
6. Reject AOP/mockup/soft smoke immediately.  
7. No real brand marks — energy only.  
8. Export trim-to-art PNG; aim 10–12″ @ 300 DPI.  
9. Name files `TD_{strain}_{family}_{role}_{palette}_vN.png`.  
10. Paths: `/home/ice/know/streetwear-cannabis-dtf-style-guide/` (`STYLE_GUIDE.md`, `prompt_modules.yaml`, `sources.md`).
