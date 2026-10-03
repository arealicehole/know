# Streetwear × Cannabis DTF Style Bible — index
**Task:** t_1eefb647 | **2026-10-03** | **geraldov21**

## Deliverables (this folder)
| File | Role |
|---|---|
| `STYLE_GUIDE.md` | Full style & motif bible (DTF grammar, 12 families, motifs, type/color, Qwen kit, strain matrix, packs, claims/gaps) |
| `prompt_modules.yaml` | Machine-usable modules: locked prefix/negative, 12 families, 22 strains, levers, 24 full example prompts |
| `sources.md` | Cited sources E1–E23 + internal U1–U4 |

## Print Junkie load path
```
/home/ice/know/streetwear-cannabis-dtf-style-guide/
```

## Operator 10-liner
1. DTF = isolated hard-edge transparent graphic — never AOP/soft alpha.  
2. Assemble: locked_prefix + family + strain + `"TEXT"` + locked_negative.  
3. Families F01–F12 cover hype/handstyle/varsity/workwear/camo-plaque/skate/luxury/Y2K/reggae/psych/punk/mascot.  
4. Strain matrix includes OG, Lamb's Breath, Gelato, Runtz, Permanent Marker, LCG + peers.  
5. Pack roles: hero / minimal / parody / crest / mascot.  
6. Variation levers in YAML for controlled packs.  
7. No real brand marks — energy only.  
8. Target ~10–12″ front @ 300 DPI after gen cleanup.  
9. Name: `TD_{strain}_{family}_{role}_{palette}_vN.png`.  
10. Validate YAML loads; smoke-test one strain × five roles on Qwen-Image-2.1.
