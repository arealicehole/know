# AI Box GPU1 bottom 3090 fan start/stop hunt — root cause + fix plan

**Task:** `t_d5555f3e`  
**Date:** 2026-09-17  
**Author:** Geraldo v2.2 (GIU)  
**Depth:** deep / multi-source  
**Target host:** AI Box dual Dell RTX 3090 (Omarchy), GPU1 = bottom `02:00.0`, display disabled  
**Symptom:** audible start/stop on bottom GPU (stop → few full rotations → stop → repeat); top GPU steady hum  
**Ops context:** Fedoranzzi NVML floor service (`ai-box-gpu-fan-floor.py` + timer) did **not** eliminate audible hunt; fan1 responds to NVML, fan0 looks sticky/mis-mapped; parking fan0 at 0% reduced but did not stop the sound

---

## objective

Produce a cited root-cause differential and ranked software→hardware fix plan for GPU1 dual-fan zero-RPM / sticky-fan hunting under idle-but-VRAM-resident llama TP2 on a headless dual-3090 Linux box (driver 610.x class), so ops can stop guessing.

## summary

The audible pattern (stop → spin a couple turns → stop) is **most consistent with a low-RPM mechanical/thermal thrash cycle**, not a pure software “one wrong API call” bug. Three mechanisms stack on GPU1:

1. **OEM/auto zero-RPM hysteresis near a temperature edge** (VRAM-resident idle keeps die warm enough to repeatedly cross the spin-up threshold), producing exactly the start/stop cadence users hear.[6][7][16]
2. **Manual PWM floors that are too low for reliable continuous spin** (~0–30%). On Ampere-class cards, **manual control typically cannot go below ~30%**, and values in the unreliable band cause RPM spikes / stall-restart rather than quiet steady spin.[5][6][7][13][16]
3. **Asymmetric dual-fan response on GPU1**: one fan header tracks NVML; the other stays at tach 0 or 0→~30 blips. That matches either (a) **bearing stiction / failing fan**, (b) **daisy-chained OEM shroud wiring** where one PWM/tach path is weak, or (c) **control-plane index mismatch** (NVML per-device fan0/fan1 vs nvidia-settings **global** fan indices).[4][10][11][12]

**Why NVML `SetFanSpeed` can return OK while fan0 tach stays 0:** NVML accepts the policy write into the driver; the return code does **not** guarantee the mechanical fan spun or that tach feedback is healthy. Manual fan policy also **persists after the setter process exits**, and `SetDefaultFanSpeed_v2` is reported not to fully restore auto behavior.[2][3]

**Why the existing floor service did not kill the hunt:** parking one fan at 0% or a low floor **keeps the card in the unstable band** (stiction / zero-RPM edge / windmill tach blips). A durable software fix is a **constant ≥35–45% (often ≥40%) on BOTH GPU1 fans**, continuous re-apply (or a proper NVML daemon), verify tach RPM > 0 and stable, then only escalate to hardware replace if one fan never leaves 0 RPM under ≥40% PWM.

**Preferred control plane on this headless box:** **NVML** (`nvmlDeviceSetFanSpeed_v2` / tools like NVFD or pynvml scripts) — no X required.[3][14] Coolbits + `nvidia-settings` is a valid second plane but needs headless X / Coolbits and careful **global** fan index mapping on multi-GPU.[4][8][9][15]

## Root-cause differential

| Hypothesis | Fits symptom? | Evidence | Confidence | How to confirm on box |
|---|---|---|---|---|
| **H1. Zero-RPM / low-RPM hysteresis thrash** (temp near spin threshold under VRAM-idle) | Yes — classic stop/spin/stop | Arch reports 0%↔30% instability below ~33%; FanControl/NV forums: auto zero-RPM vs manual 30% floor; Linux 470+ regression locked manual min ~30%[5][6][7][16] | **High** | Log 1 Hz: GPU1 temp, fan0/1 %, fan0/1 RPM for 10 min idle with TP2 resident; look for periodic crossings |
| **H2. Bearing stiction / dying fan0** | Yes — few rotations then die; fan1 OK | Ops observation fan1 responds, fan0 sticky; OEM dual-fan kits document separate Fan A/B headers[10][11] | **Medium-High** | Set **both** fans to 45% via NVML; if fan0 RPM stays 0 while fan1 >1000, hardware |
| **H3. NVML fan-index / dual-header mapping** | Partial — can explain one fan silent under “wrong” index | NVML exposes per-device `GetNumFans` + per-`fan` set/get; dual 3090 nvidia-settings uses **global** fan:0,1,2… not per-GPU[1][3][4][12] | **Medium** | `GetNumFans` on GPU1; set fan0=45, fan1=45 separately; read back target + RPM each |
| **H4. Windmill / tach ghost from airflow** | Partial — tach blips without sustained spin | Neighbor fan / case airflow can spin a free fan slowly; tach may blip 0→30 | **Low-Medium** | Visually/audibly isolate which physical fan; cover briefly (careful) or note phase vs fan1 PWM |
| **H5. Secondary-GPU / display-disabled driver quirk** | Weak primary, possible aggravator | Coolbits/X paths often miss non-display GPUs; NVML should still work headless[8][9][15] | **Low-Medium** | Compare NVML set on GPU0 vs GPU1; if only GPU1 fails under identical PWM, not pure X quirk |
| **H6. Floor service race / timer too sparse** | Aggravator | Manual policy can be overwritten; sparse timer allows hysteresis cycle between ticks[3] | **Medium** | journalctl of floor unit vs 1 Hz tach log; ensure continuous hold not one-shot |

**Most likely stack (ranked):** H1 + H2 on GPU1 fan0, with H3/H6 as process failures that left the card in the bad band. Top GPU “just hums” = continuous spin above stiction / not on the zero-RPM edge.

## Why NVML returns OK but fan0 tach is 0 or 0→30 blips

1. **API success ≠ mechanical success.** `nvmlDeviceSetFanSpeed_v2(device, fan, percent)` writes target policy. NVML also exposes `GetFanSpeed`, `GetFanSpeedRPM`, `GetTargetFanSpeed`, `GetMinMaxFanSpeed`, `GetNumFans`, `SetDefaultFanSpeed`, `SetFanControlPolicy`.[1][3][14] You must **read RPM back per fan index**.
2. **Per-fan indices are real.** Dual-fan cards expose fan 0 and fan 1 on the **same device handle**. Setting only one index leaves the other under auto/zero-RPM.[3][12][14]
3. **Manual policy sticks.** After `SetFanSpeed_v2`, the card stays manual even if the process dies; unclean exit can leave fans pinned or half-managed. NVFD documents this explicitly and uses `ExecStopPost=nvfd reset-fans`.[3]
4. **`SetDefaultFanSpeed_v2` is unreliable for “full auto resume”** per NVIDIA forum reports (speed may drop to a default but auto ramp under load does not fully return).[2]
5. **Driver min clamp.** On many Ampere+ Linux stacks, **user PWM below ~30% is clamped or rejected in practice**; requesting 0–25% can yield “success” with actual ~30% or weird tach.[5][7][13][16]
6. **Tach vs PWM mismatch on 3090.** Users report 30% target mapping to unexpectedly high RPM / loudness — OEM curves and tach scaling are imperfect.[13]
7. **nvidia-settings index trap (if mixed control planes).** Fan targets are **global** (`fan:0` … `fan:N` across the machine). Dual 3090 writeup: GPU1’s fan was `fan:2`, not `fan:1`. Wrong global index → you think you set GPU1 while moving GPU0’s second fan or a no-op.[4][12]

## Linux control methods that work headless (driver 610.x class)

### A. NVML (recommended primary)

| Item | Detail |
|---|---|
| Works without X? | **Yes** — preferred for AI Box |
| Key APIs | `nvmlDeviceGetNumFans`, `GetFanSpeed` / `GetFanSpeedRPM`, `GetMinMaxFanSpeed`, `SetFanSpeed_v2`, `SetDefaultFanSpeed_v2`, `SetFanControlPolicy`[1][3] |
| Tools | **NVFD** (daemon + TUI, multi-GPU, systemd, failsafe),[3] pynvml scripts (e.g. RoversX/Cippo95 lineage),[14] custom floor service (already present) |
| Privileges | Root/admin typically required for set |
| Persistence | Re-apply on boot via systemd; reset to auto on stop |
| Caveat | Manual min ~30%; do not aim for true 0 RPM in manual mode[5][7][16] |

### B. nvidia-settings + Coolbits (secondary)

| Item | Detail |
|---|---|
| Coolbits | Bit 2 (`4`) historically enables fan control page; Arch notes bit removed as documented path in 470.42.01 but field practice still uses Coolbits (often `4`, `12`, or `28`) per Device section; need **all GPUs** enabled[9][15] |
| Headless | Requires X: `xvfb-run`, or dedicated headless Xorg on `:2` bound to NVIDIA BusIDs (gpu-fanctl pattern)[4][8] |
| Multi-GPU | `nvidia-xconfig --enable-all-gpus --cool-bits=4`; map **global** fan indices experimentally[4][9][12] |
| Secondary GPU | Display-disabled GPU often needs its own Device section / Coolbits or NVML instead[8][15] |
| Caveat | Root/`nvidia-settings`, X session quirks; Arch marks some autostart recipes disputed[9] |

### C. Hybrid monitoring

- **Read path:** `nvidia-smi` / NVML always (no X).[9]
- **Write path:** NVML for compute box; keep coolbits path as fallback experiment only.

## Dell OEM RTX 3090 (subsystem 3880) fan/shroud notes

Public material is mostly **parts/replacement**, not VBIOS fan tables:

- Dell/Lenovo OEM 30-series dual-fan shrouds use **two physical fans** (Fan A often 4+2 pin for LED/tach daisy, Fan B 4-pin), ~88 mm, triangle mount 42 mm, example P/N **PLA09215B12H**, 12V.[10][11]
- Replacement guides: remove shroud (4–8 screws), disconnect board + daisy-chain connectors, swap fans.[10]
- No authoritative public doc found that subsystem `3880` uniquely disables NVML fan control; treat behavior as **generic Ampere OEM dual-fan + Linux driver policy**.
- Implication for AI Box: **software can drive both headers if hardware is healthy**; one dead bearing is a common failure mode on aged mining/OEM cards and matches “fan1 OK, fan0 sticky.”

*Gap:* Exact Dell VBIOS zero-RPM trip points for SSID 3880 were not found in public sources (see gaps).

## Safe operating envelope if one fan is dead

| Condition | Guidance | Confidence |
|---|---|---|
| **Idle / VRAM-resident TP2, one fan spinning ≥35%** | Often survivable short-term if GPU temp stays **&lt;70–75°C** and no thermal slowdown; still elevated hotspot risk | Medium |
| **Full load / sustained train or dense decode, one fan dead** | **Not safe long-term** on 350 W-class 3090 in dense dual-slot sandwich; expect throttle, VRAM heat, shortened life | High |
| **Mitigations while waiting for parts** | (1) Force surviving fan **≥60–80%**; (2) `nvidia-smi -pl` lower power limit on GPU1; (3) improve case intake between cards; (4) avoid long 100% jobs on GPU1; (5) monitor temp every 5s, abort &gt;83–85°C | Medium-High |
| **Replace threshold** | If any fan **cannot hold RPM &gt;0 at 45% PWM for 5 minutes**, schedule hardware replace; do not “tune around” a dead fan under load | High |

*Note:* These limits are engineering judgment from TDP/class behavior + community thermal practice, not a Dell OEM datasheet.[unverified] for exact hotspot margins.

## Ranked fix plan for Fedoranzzi (software first)

### P0 — Diagnose in 15 minutes (do before more tweaks)

```bash
# Identify GPUs / bus
nvidia-smi -L
nvidia-smi --query-gpu=index,pci.bus_id,name,temperature.gpu,fan.speed,power.draw,memory.used --format=csv

# Per-fan NVML probe (Python; needs root + pynvml or bundled nvidia-ml-py)
sudo python3 - <<'PY'
from pynvml import *
nvmlInit()
for i in range(nvmlDeviceGetCount()):
    h = nvmlDeviceGetHandleByIndex(i)
    name = nvmlDeviceGetName(h)
    try: name = name.decode()
    except: pass
    pci = nvmlDeviceGetPciInfo(h)
    n = nvmlDeviceGetNumFans(h)
    print(f"GPU{i} {name} bus={pci.busId} fans={n}")
    for f in range(n):
        try:
            sp = nvmlDeviceGetFanSpeed_v2(h, f)
        except Exception:
            sp = nvmlDeviceGetFanSpeed(h)
        try:
            rpm = nvmlDeviceGetFanSpeedRPM(h, f)
        except Exception as e:
            rpm = f"n/a ({e})"
        try:
            mn, mx = nvmlDeviceGetMinMaxFanSpeed(h, f)
        except Exception:
            mn = mx = "?"
        print(f"  fan{f}: speed%={sp} rpm={rpm} min/max={mn}/{mx}")
nvmlShutdown()
PY
```

**Pass criteria:** GPU1 reports **2 fans**. Note which index has RPM 0.

Confidence: **High** that this is the right first step.

### P1 — Software hold: BOTH GPU1 fans at constant ≥40% (kill the hunt band)

Rationale: leave the 0–30% thrash/stiction band; manual mode cannot do true silent zero-RPM anyway.[5][6][7][16]

```bash
# One-shot hold (adjust GPU index if bottom is not 1)
sudo python3 - <<'PY'
from pynvml import *
nvmlInit()
# Prefer PCI bus match for bottom card 02:00.0
target_bus = None  # e.g. b"00000000:02:00.0" if needed
for i in range(nvmlDeviceGetCount()):
    h = nvmlDeviceGetHandleByIndex(i)
    pci = nvmlDeviceGetPciInfo(h)
    bus = pci.busId.decode() if hasattr(pci.busId, 'decode') else pci.busId
    if "02:00.0" not in bus and i != 1:
        continue
    n = nvmlDeviceGetNumFans(h)
    for f in range(n):
        # 40–45% is the usual minimum reliable continuous spin band
        rc = nvmlDeviceSetFanSpeed_v2(h, f, 45)
        print("set", bus, "fan", f, "-> 45% rc", rc)
    for f in range(n):
        print(" read fan", f, "pct", nvmlDeviceGetFanSpeed_v2(h, f),
              "rpm", nvmlDeviceGetFanSpeedRPM(h, f))
nvmlShutdown()
PY

# Watch 5 minutes
watch -n1 'nvidia-smi --query-gpu=index,pci.bus_id,temperature.gpu,fan.speed --format=csv; echo; date'
```

**Update floor service semantics:**

- Apply to **GPU1 fan0 AND fan1** (never park one at 0% while “fixing” hunt).
- Floor **≥40%** (try 45; if still audible chirps, 50–55).
- Interval **≤5–10s** continuous hold (or long-running daemon), not sparse timer-only.
- On stop: `nvmlDeviceSetDefaultFanSpeed_v2` per fan **or** accept permanent manual until reboot; document which.
- Optional: adopt **NVFD** fixed mode for GPU1 only: `nvfd 1 manual 45` (syntax per NVFD help) while GPU0 stays auto/hum.[3]

**Success:** audible start/stop **gone**; both RPM steady non-zero; temp stable.

Confidence: **High** if hardware is OK; **Medium** if fan0 is mechanically dead (will prove H2).

### P2 — If hunt continues only on one physical fan at 45%+

Declare **hardware path**:

1. Confirm visually which fan is the clicker.
2. Order Dell/Lenovo OEM dual-fan set (PLA09215B12H-class / dual A+B kit).[10][11]
3. While waiting: power-limit GPU1, raise surviving fan to 70%+, avoid long full-load jobs.

Confidence: **High** for replace-when-RPM-never-rises.

### P3 — Optional coolbits plane (only if NVML blocked on this driver)

```bash
# Discover global fan targets (needs working X display :0 or headless :2)
DISPLAY=:0 nvidia-settings -q all | grep -i fan
# or
xvfb-run -a nvidia-settings -q fans

# Example dual-GPU — DO NOT copy fan indices blindly; probe first [4]
xvfb-run -a nvidia-settings \
  -a "[gpu:1]/GPUFanControlState=1" \
  -a "[fan:N]/GPUTargetFanSpeed=45" \
  -a "[fan:M]/GPUTargetFanSpeed=45"
```

Headless pattern: dedicated Xorg on `:2` with Coolbits 28 and PCI BusIDs (gpu-fanctl).[8]

Confidence: **Medium** (more moving parts than NVML).

### P4 — Do **not** chase true 0 RPM on Linux manual control

Auto mode is the only practical path to OEM zero-RPM silence; it reintroduces hysteresis near the trip point under VRAM heat.[5][7][16] For an always-on inference box, **steady low-mid PWM** is the correct acoustic tradeoff.

Confidence: **High**.

## Copy-paste: hardened floor service sketch

```python
#!/usr/bin/env python3
"""ai-box-gpu-fan-floor.py — hold BOTH fans on selected GPUs above stiction band."""
import os, time
from pynvml import *

# Bottom GPU PCI substring and floor percent
PCI_MATCH = os.environ.get("FAN_FLOOR_PCI", "02:00.0")
FLOOR = int(os.environ.get("FAN_FLOOR_PCT", "45"))  # >=40
INTERVAL = float(os.environ.get("FAN_FLOOR_INTERVAL", "5"))

def main():
    nvmlInit()
    while True:
        for i in range(nvmlDeviceGetCount()):
            h = nvmlDeviceGetHandleByIndex(i)
            bus = nvmlDeviceGetPciInfo(h).busId
            bus = bus.decode() if hasattr(bus, "decode") else str(bus)
            if PCI_MATCH not in bus:
                continue
            n = nvmlDeviceGetNumFans(h)
            for f in range(n):
                nvmlDeviceSetFanSpeed_v2(h, f, FLOOR)
                try:
                    rpm = nvmlDeviceGetFanSpeedRPM(h, f)
                except Exception:
                    rpm = -1
                pct = nvmlDeviceGetFanSpeed_v2(h, f)
                print(f"{time.time():.0f} {bus} fan{f} target={FLOOR} pct={pct} rpm={rpm}", flush=True)
                if rpm == 0:
                    print(f"WARN {bus} fan{f} RPM still 0 at {FLOOR}% — likely hardware", flush=True)
        time.sleep(INTERVAL)

if __name__ == "__main__":
    main()
```

systemd: `Type=simple` long-running service (not oneshot + sparse timer only); `Restart=always`; stop hook optional reset-to-auto.

## claims

| claim_id | statement | evidence_refs | confidence | status |
|---|---|---|---|---|
| C1 | NVML exposes per-device multi-fan get/set APIs including SetFanSpeed / GetNumFans / GetFanSpeedRPM | [1][3][14] | high | supported |
| C2 | Manual fan mode on modern NVIDIA Linux commonly enforces ~30% minimum; true 0 RPM is auto/VBIOS territory | [5][7][13][16] | high | supported |
| C3 | Low static PWM (&lt;~33%) can cause erratic start/stop thrash as temp oscillates near thresholds | [6] | high | supported |
| C4 | nvidia-settings fan indices are global across GPUs; dual 3090 may use non-contiguous fan numbers | [4][12] | high | supported |
| C5 | NVML manual policy outlives the process; unclean exit leaves fans managed | [3] | high | supported |
| C6 | SetDefaultFanSpeed_v2 may not fully restore auto algorithm as documented | [2] | medium | partial |
| C7 | Headless fan control works via NVML without X; coolbits path needs X/Xvfb/headless Xorg | [3][4][8][9][14] | high | supported |
| C8 | Dell/Lenovo OEM 3090 dual-fan shrouds use two replaceable fans (A/B), public P/Ns available | [10][11] | high | supported |
| C9 | GPU1 audible hunt after floor service is best explained by low-PWM/zero-RPM band + possible fan0 mechanical fault | task body + [6][7] | medium-high | partial |
| C10 | One dead fan under full 3090 load is unsafe long-term without power limit / replace | engineering | medium | partial |
| C11 | Arch Coolbits bit 4 (fan) marked removed in 470.42.01 docs path; field tools still use Coolbits | [9][8] | medium | partial |
| C12 | Subsystem 3880-specific public fan curve docs were not found | search gap | high | supported (gap) |

## evidence_index

| evidence_id | source_label | source_url | provenance | excerpt / support note |
|---|---|---|---|---|
| E1 | NVML device queries / API surface | https://docs.nvidia.com/deploy/nvml-api/group__nvmlDeviceQueries.html | web_extract | NVML query/command groups; fan-related API family documented in NVML tree |
| E2 | nvml.h fan symbols (Get/Set FanSpeed, NumFans, RPM, MinMax, Default, Policy) | https://raw.githubusercontent.com/NVIDIA/nvidia-settings/main/src/nvml.h | web_extract + grep | Confirmed symbols: `nvmlDeviceGetNumFans`, `GetFanSpeed`, `GetFanSpeedRPM`, `GetMinMaxFanSpeed`, `SetFanSpeed`, `SetDefaultFanSpeed`, `SetFanControlPolicy` |
| E3 | NVFD README — headless NVML daemon | https://github.com/Infinirc/nvfd | web_extract | NVML works X11/Wayland/headless; SetFanSpeed_v2 policy persists; reset on stop; multi-fan UI |
| E4 | Dual 3090 global fan index | https://davidrusseltrask.com/setting-manual-fan-speeds-on-dual-rtx-3090-gpus/ | web_extract | GPU2 fan was `fan:2` not `fan:1`; xvfb-run for headless nvidia-settings |
| E5 | 30% manual floor / no zero RPM | https://forums.developer.nvidia.com/t/any-workaround-to-the-30-fan-speed-limitation-on-newer-nvidia-gpus/291486 | web_extract | Manual/auto user floor discussions; moderator: reported range is user-settable range not VBIOS floor |
| E6 | Arch low-RPM thrash | https://bbs.archlinux.org/viewtopic.php?pid=2234005 | web_extract | Static &lt;33% erratic; 0%↔30% cycle with temp 55–56°C oscillation |
| E7 | FanControl 0RPM vs 30% issue | https://github.com/Rem0o/FanControl.Releases/issues/265 | web_search/prior extract | Manual curves won’t go under 30%; auto can 0 RPM |
| E8 | gpu-fanctl headless Xorg Coolbits | https://github.com/brat91tvoj/gpu-fanctl | web_extract | Headless X :2 Coolbits 28; manual min ~30%; auto for 0 RPM |
| E9 | ArchWiki Coolbits + fan CLI | https://wiki.archlinux.org/title/NVIDIA/Tips_and_tricks | web_extract | Coolbits bits; fan via `GPUFanControlState` + `GPUTargetFanSpeed`; enable-all-gpus |
| E10 | Dell OEM dual fan kit | https://gpuconnect.com/products/dell-rtx-3000-gpu-fan | web_extract | Fan A/B, PLA09215B12H, dual-fan replace procedure |
| E11 | Dell/Lenovo 3090 fan replacement listing | https://www.gpufanreplacement.com/products/dell-lenovo-rtx-3060-3070-3080-3090-fan-replacement | web_search | OEM dual-fan replacement market for Dell/Lenovo 3090 |
| E12 | Fan index vs GPU index forum | https://forums.developer.nvidia.com/t/how-to-control-fan-speed-based-on-gpu-index-instead-of-fan-index/173940 | web_search | nvidia-settings fan vs gpu targeting confusion |
| E13 | 3090 30% RPM loudness | https://forums.developer.nvidia.com/t/rtx-3090-fan-30-limit-acts-incorrect-too-high-and-hence-fan-is-too-loud/323325 | web_extract | 30% maps to high RPM on some 3090s; persists across driver updates |
| E14 | pynvml terminal fan control | https://closex.medium.com/the-simplest-linux-nvidia-gpu-fan-speed-control-tutorial-8d641e9efdff | web_extract | Headless pynvml curve; SetDefaultFanSpeed_v2 on exit |
| E15 | SetDefaultFanSpeed_v2 incomplete resume | https://forums.developer.nvidia.com/t/nvmldevicesetdefaultfanspeed-v2-does-not-resume-fan-speed-algorithm-please-fix/214430 | web_extract | After manual 100%, default call does not restore full auto ramp |
| E16 | 470+ zero-RPM regression | https://forums.developer.nvidia.com/t/fan-speed-regression-with-nvidia-beta-470-42-01-and-rtx-3080-fans-dont-stop-on-idle/183604 | web_extract | Manual cannot set below 30; auto zero-RPM broken vs 465 on some setups |

## gaps

1. **Live box telemetry not re-sampled in this research container** — brief attachments (`BRIEF-gpu1-fan-hunt.md`, fan-floor note) and `/home/ice/bin/ai-box-gpu-fan-floor.py` were **not mounted** into the researcher container; reconstruction used task body + Fedoranzzi comment. Ops should paste 60s of per-fan RPM after P0/P1.
2. **No public Dell SSID 3880 VBIOS fan-table dump** found — exact zero-RPM trip °C unknown.
3. **Driver 610.x exact min-fan behavior** on this Omarchy install not measured here — community floor is ~30% but card-specific mins vary (23–33% reported).
4. **Whether GPU1 fans are electrically daisy-chained vs independent PWM** needs eyes-on shroud or `GetNumFans` + independent RPM response test.
5. **Coolbits bit 4 “removed in 470”** vs tools still using Coolbits — driver README for installed 610.x should be checked on box.
6. **Thermal pad / sandwich airflow** between dual 3-slot 3090s may raise GPU1 baseline temp into hysteresis band even with healthy fans — not quantified.

## confidence

```json
{
  "level": "high",
  "rationale": "Control-plane facts (NVML multi-fan API, 30% manual floor, global nvidia-settings fan indices, headless NVML vs X coolbits, OEM dual-fan hardware) are multi-source corroborated. Root cause for THIS chassis is medium-high: strongest software fix is exit the low-PWM band on BOTH fans; residual risk is mechanical fan0 failure which P1 will falsify quickly. Live attachment telemetry was unavailable in-container."
}
```

## recommended_next_actions

1. **Ops P0/P1 today:** Run per-fan NVML dump on GPU1 (`02:00.0`); set **both** fans to **45%** continuous; listen 10 minutes; capture RPM log. (Confidence: high)
2. **Patch floor service:** Never park fan0 at 0%; floor both headers ≥40%; prefer long-running service ≤5s loop or NVFD manual on GPU1. (Confidence: high)
3. **If fan0 RPM stays 0 at ≥45%:** Order Dell OEM dual-fan kit; power-limit GPU1; raise fan1; do not run sustained full load. (Confidence: high)
4. **Only if NVML set fails:** Probe coolbits/headless X secondary path with **probed** global fan indices. (Confidence: medium)
5. **Do not optimize for 0 RPM** on an always-warm TP2 box — accept steady hum over hunt. (Confidence: high)

---

## Sources

[1] NVML Device Queries — https://docs.nvidia.com/deploy/nvml-api/group__nvmlDeviceQueries.html  
[2] nvmlDeviceSetDefaultFanSpeed_v2 does not resume algorithm — https://forums.developer.nvidia.com/t/nvmldevicesetdefaultfanspeed-v2-does-not-resume-fan-speed-algorithm-please-fix/214430  
[3] NVFD (Infinirc) — https://github.com/Infinirc/nvfd  
[4] Setting Manual Fan Speeds on Dual RTX 3090 GPUs — https://davidrusseltrask.com/setting-manual-fan-speeds-on-dual-rtx-3090-gpus/  
[5] Workaround to 30% fan limitation — https://forums.developer.nvidia.com/t/any-workaround-to-the-30-fan-speed-limitation-on-newer-nvidia-gpus/291486  
[6] Arch Linux Forums: NVIDIA Fan Speed thrash — https://bbs.archlinux.org/viewtopic.php?pid=2234005  
[7] FanControl issue: 0 RPM vs 30% — https://github.com/Rem0o/FanControl.Releases/issues/265  
[8] gpu-fanctl headless Coolbits — https://github.com/brat91tvoj/gpu-fanctl  
[9] ArchWiki NVIDIA Tips and tricks — https://wiki.archlinux.org/title/NVIDIA/Tips_and_tricks  
[10] DELL Lenovo OEM RTX 3070/3080/3090 fan set — https://gpuconnect.com/products/dell-rtx-3000-gpu-fan  
[11] Dell/Lenovo RTX 3060–3090 fan replacement — https://www.gpufanreplacement.com/products/dell-lenovo-rtx-3060-3070-3080-3090-fan-replacement  
[12] Control fan by GPU index vs fan index — https://forums.developer.nvidia.com/t/how-to-control-fan-speed-based-on-gpu-index-instead-of-fan-index/173940  
[13] RTX 3090 fan 30% limit too loud — https://forums.developer.nvidia.com/t/rtx-3090-fan-30-limit-acts-incorrect-too-high-and-hence-fan-is-too-loud/323325  
[14] CloseX pynvml fan control tutorial — https://closex.medium.com/the-simplest-linux-nvidia-gpu-fan-speed-control-tutorial-8d641e9efdff  
[15] NVIDIA nvml.h (nvidia-settings tree) — https://raw.githubusercontent.com/NVIDIA/nvidia-settings/main/src/nvml.h  
[16] Fan speed regression 470 / no idle 0 RPM — https://forums.developer.nvidia.com/t/fan-speed-regression-with-nvidia-beta-470-42-01-and-rtx-3080-fans-dont-stop-on-idle/183604  

---

## Appendix A — NVML fan-related symbols (from nvml.h grep)

- `nvmlDeviceGetFanControlPolicy`
- `nvmlDeviceGetFanSpeed` / `nvmlDeviceGetFanSpeedRPM`
- `nvmlDeviceGetMinMaxFanSpeed`
- `nvmlDeviceGetNumFans`
- `nvmlDeviceGetTargetFanSpeed`
- `nvmlDeviceSetDefaultFanSpeed`
- `nvmlDeviceSetFanControlPolicy`
- `nvmlDeviceSetFanSpeed` (+ `_v2` usage in community tools)

## Appendix B — Task provenance

- Kanban: `t_d5555f3e` `[RESEARCH] AI Box GPU1 bottom 3090 fan start/stop hunt`
- Dispatcher notes: Fedoranzzi; user confirms hunt still audible after floor service
- Attachments referenced but **not readable in researcher Docker mount**: `BRIEF-gpu1-fan-hunt.md`, `20260917-ai-box-gpu-fan-floor.md`
- Durable path required: `/home/ice/know/research/research-aibox-gpu1-fan-hunt-REPORT.md`
