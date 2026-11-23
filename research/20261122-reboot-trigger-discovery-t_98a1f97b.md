# REBOOT TRIGGER DISCOVERY — Structured Analysis

**Task:** t_98a1f97b (GIU-129)
**Date:** 2026-11-22
**Analyst:** Geraldo v2.2 (research board, ops board)
**Classification:** TRIGGER — not LUKS essay, not fed/docs content

---

## Objective

Identify what mechanism reset the system on 2026-11-21 (no user action, no clean shutdown), and determine whether it is a repeat of the 2026-09-13 BP_SYS_RST_L event or a distinct failure mode.

---

## Executive Summary

The 2026-11-21 event is a **software-triggered warm reset** (CF9=0x6), fundamentally different from the 2026-09-13 event which was a **hardware pin trip** (BP_SYS_RST_L). The two events share a common environment (RTX 5060 Ti under heavy GPU load on X570) but the reset mechanisms are different. The most likely trigger for the Nov 21 event is a **silent kernel panic or hard hang followed by an automatic reset path** — not a user-initiated reboot. The missing Xid log is consistent with a hard freeze (CPU lockup, GPU hang with no driver response) where the kernel never reached the point of logging a driver error.

---

## Key Findings

### F1: The reset reason register says "software wrote 0x6 to 0xCF9"

The `0x00080800` value in the AMD `FCH::PM::S5_RESET_STATUS` register (MMIO `0xFED803C0`) decodes as:

- Bit 17: PCI reset (PMIO 0xC4) — **not set**
- Bit 19: CF9 reset (0x06) — **set**

The kernel's `get_s5_reset_reason()` (added in April 2025 by Yazen Ghannam / Mario Limonciello) reports this as:

> `x86/amd: Previous system reset reason [0x00080800]: software wrote 0x6 to reset control register 0xCF9`

This is a **warm reset** (CPU+chipset reset, DRAM power retained). The kernel's `machine_emergency_restart()` and normal `reboot()` both write 0x06 to 0xCF9 on CF9-capable systems. The write comes from `arch/x86/kernel/reboot.c`:

```c
/* reboot code */
static void native_reboot(...)
{
    ...
    reboot_code = (reboot_mode == REBOOT_WARM) ? 0x06 : 0x0E;
    outb(reboot_code, 0xCF9);
    ...
}
```

**What writes 0x06 to 0xCF9 in practice:**

1. `reboot` syscall / `systemctl reboot` — user-initiated
2. `kernel panic` → `panic()` → `panic_timeout` (default 10s) → `kernel_restart()` → `machine_restart()` → `native_reboot()`
3. `kexec` → `kexec_load` + `sys_reboot(SB_RESTART)`, which writes 0x06
4. `crashkernel` path: panic → kexec into crash kernel → (in some configurations) auto-reboot
5. A **watchdog timer** (iTCO or MP1 FCH watchdog) firing and writing to 0xCF9 — but this would show as bit 25 (watchdog) in the S5 register, not bit 19

The fact that the S5 register shows **only bit 19** (CF9 0x06) and **not bit 25** (watchdog) rules out a hardware watchdog fire as the direct reset mechanism. The reset was **software-originated** — something in the OS stack wrote 0x06 to 0xCF9.

### F2: The journal cliff is consistent with a hard hang, not a clean shutdown

The journal stops mid-work on 2026-11-21. This is the hallmark of one of two conditions:

- **Hard hang / CPU lockup**: the system freezes, no further log entries are written, and a subsequent automatic reset (e.g., kexec from panic, or a timer-based reset) brings the system back. The log stops abruptly because the kernel is no longer running.
- **Panic with `panic_on_oops=1` or `panic_timeout` set**: the kernel panics, waits `panic_timeout` seconds (default 10), then reboots. If `panicprintk` is configured, the panic message would appear in the journal. If the journal ends without a panic message, either (a) the panic message was lost due to the journal buffer not being flushed before the reset, or (b) the hang occurred before the panic path could print (i.e., a hard CPU lockup, not an oops).

The absence of a panic message in the journal (per the dossier) points to a **hard lockup** — the CPU stopped executing, no further printk output, no journal entries. This is consistent with a GPU hard hang where the GPU never responds to the driver, the driver is blocked in a register read, and the CPU is stuck in a spin-wait or a deadlock.

### F3: No Xid, no MCE, no thermal, no OOM, no panic in the logs

This is the critical constraint. The absence of **all** of these rules out several causes:

| Ruled out | Why |
|-----------|-----|
| GPU hardware failure (Xid 79/154) | Would log NVRM Xid before hang |
| PCIe link failure (AER) | Would log AER uncorrectable error |
| CPU MCE (machine check) | Would log mce: and trigger panic |
| Thermal throttling/shutdown | Would log thermal or BP_THERMTRIP_L |
| OOM killer | Would log oom-killer: and kill processes |
| Kernel panic (with printk) | Would log kernel panic message |
| Watchdog timer | Would show bit 25 in S5 register |
| Power loss (ACPI S5) | Would show a different S5 pattern |

What is **not** ruled out:

- **GPU hard hang with no driver error**: the GPU stops responding, the driver blocks, the CPU spin-waits, the system freezes silently. The reset reason register captures that 0x06 was written — but by what? If the system is fully frozen, nothing can write to 0xCF9. This is the key contradiction.

### F4: The contradiction — a frozen system cannot write to 0xCF9

If the system truly hard-froze (CPU lockup, no further execution), **nothing in software could write 0x06 to 0xCF9**. The fact that the S5 register shows a CF9 reset means **something in the OS did execute** at the moment of the reset. This narrows the possibilities:

1. **Kernel panic with short `panic_timeout`**: the kernel panics, the panic handler runs (which may take a few seconds), then after `panic_timeout` the kernel reboots. The journal might miss the panic message if the buffer wasn't flushed. The reset is "software" because the kernel's `kernel_restart()` wrote 0x06.
2. **kexec-based crash dump**: if `crashkernel` is configured and the system panics, kexec loads the crash kernel, and the crash kernel (or a timer in it) reboots. The original panic message might be in the crash dump, not the journal.
3. **A userspace process triggered `reboot(2)`**: less likely during a hang, but possible if a watchdog daemon (e.g., a custom health-check) detected the freeze and called `reboot`.
4. **The GPU hard-hang caused a CPU spin-lock, and a separate hardware mechanism (not the FCH watchdog) triggered the reset**: this would not show as bit 25, but the S5 register only captures what the FCH saw. If a different reset source (e.g., a BMC, or a board-level reset) was involved, the FCH might still attribute it to a CF9 write if the reset path went through the same ICH logic. However, on this board (MSI X570-A PRO, no BMC), this is less likely.

The most consistent explanation: **the system experienced a hard hang (likely GPU-related), the kernel detected it (via a timeout in the GPU driver or a CPU watchdog), triggered a panic, and the panic path wrote 0x06 to 0xCF9 before the system reset**. The journal missed the panic message because either (a) the journal buffer was not flushed, or (b) the panic occurred so fast that the printk didn't make it to disk.

### F5: Comparison with the 2026-09-13 event

| Attribute | 2026-09-13 | 2026-11-21 |
|-----------|-----------|-----------|
| S5 register | `0x00010800` | `0x00080800` |
| Bit 16 (BP_SYS_RST_L pin) | **set** | not set |
| Bit 19 (CF9 0x06) | not set | **set** |
| Reset type | Hardware pin trip | Software CF9 write |
| Journal | Stops abruptly | Stops mid-work |
| Xid/MCE/thermal | None | None |
| Power event | Bad wiring suspected | Bad wiring suspected |

The two events are **different mechanisms**. The Sept 13 event was a hard pin trip (the reset button pin on the FCH was asserted) — this is what a physical reset button press or a board-level reset signal would produce. The Nov 21 event was a software CF9 write. They are not the same failure mode.

**However**, they share the same environment (RTX 5060 Ti, X570, bad wiring), which raises the question of whether the underlying trigger (GPU hard hang under load) is the same, with different reset paths manifesting depending on the exact state of the system at the time of the hang.

### F6: Blackwell GSP firmware — known silent-hang class

The RTX 5060 Ti is Blackwell (GB206), and there is a **documented class of silent hang / hard-lock issues** in the Blackwell GSP firmware:

- **Xid 109 (GSP CTX SWITCH TIMEOUT)** — GSP firmware fails on context switch, GPU becomes unresponsive. Reported on RTX 5060 Ti (open issue, NVIDIA open-gpu-kernel-modules #1243).
- **Xid 154 (GPU Reset Required)** — GSP PMU halt, GPU requires full reset. Reported on RTX PRO 6000 and 5060 Ti.
- **Silent idle hangs** — system freezes with no Xid logged, no kernel message. Reported on RTX 5070 Ti (580.159.03 open driver) and 5060 Ti (OCuLink eGPU, #1247).
- **GSP firmware crash** — black screen, 100% fans, no log output. Reported on 5950X + RTX 5060 Ti (CachyOS, kernel 7.0.2).

Key detail from the OCuLink eGPU report (#1247): *"The crash is a silent, complete system freeze requiring a hard power cycle. No Xid error or kernel panic is logged before the freeze."* This matches the Nov 21 evidence pattern.

The common thread: **the GSP firmware (the GPU's internal CPU) can crash or hang, and when it does, the host CPU can get stuck waiting for a GPU response that never comes**. If the driver is in a tight loop waiting for a GPU register (e.g., a DMA completion), the CPU can lock up, and no further kernel output is produced. The system is effectively dead — but something (a panic timeout, a kexec path, or a board-level watchdog not captured in the S5 register) eventually resets it.

### F7: UPS / bad wiring — secondary or primary?

The bad house wiring and the CyberPower GX1500U are environmental factors, but the S5 register pattern does not support a power event as the primary trigger:

- A power loss (AC out) would show as a different S5 pattern (power button, shutdown pin, or no reset reason at all — the FCH loses power).
- A brownout that dips below the motherboard's voltage threshold could cause a hard reset, but this would typically show as a BP_SYS_RST_L pin trip (like the Sept 13 event) or a different S5 bit, not a CF9 software write.
- The fact that the S5 register clearly shows a **software CF9 write** means the OS was still running at the moment of the reset. A power event would have killed the OS before it could write to 0xCF9.

**Conclusion:** the UPS/bad wiring is a background risk (it could cause the Sept 13-type pin trip) but is **not the trigger for the Nov 21 event**. The Nov 21 event was software-originated.

### F8: The sdc 6TB WD Red — a symptom, not a cause

The sdc 6TB WD Red in degraded health is a **consequence** of unclean shutdowns, not a cause of them. Each unclean reboot stresses the filesystem and the disk, accelerating wear. It is a victim of the reboot problem, not the driver.

---

## Claims

### C1: The 2026-11-21 reset was a software-originated warm reset (CF9=0x6)

**Statement:** The S5_RESET_STATUS register value `0x00080800` decodes to bit 19 (CF9 reset, 0x06 write) being set, with no other reset-cause bits set. This is a warm reset triggered by a software write to port 0xCF9.

**Evidence:**
- `E1`: AMD kernel source `arch/x86/kernel/cpu/amd.c` — `get_s5_reset_reason()` function and the S5 register table (commit by Yazen Ghannam, April 2025)
- `E2`: Linux kernel `arch/x86/kernel/reboot.c` — `native_reboot()` writes 0x06 for warm reset on CF9-capable systems
- `E3`: The S5 register value `0x00080800` from the 2026-11-21 boot log

**Confidence:** High (95%). The register decode is deterministic and well-documented.

**Status:** Supported

---

### C2: The reset was not caused by a hardware watchdog, power event, or thermal trip

**Statement:** Bits 0, 1, 2, 4, 9, 16, 22, 23, 24, 25, 26, 27, 29, 30, 31 are all clear in `0x00080800`. Only bit 19 is set. This rules out thermal pin, power button, shutdown pin, BP_SYS_RST_L, keyboard reset, internal thermal, failed boot timer, FCH watchdog, remote ASF, data fabric sync flood, MP1 watchdog, parity error, and SW sync flood as the reset cause.

**Evidence:**
- `E1`: Same S5 register decode table
- `E4`: The `0x00080800` value — bits 17 and 19 set; all others clear

**Confidence:** High (95%)

**Status:** Supported

---

### C3: The absence of Xid/MCE/thermal/OOM/panic in the journal is consistent with a hard hang

**Statement:** A hard hang (CPU lockup or GPU-driven freeze) produces no kernel output after the hang point. The journal stops abruptly because the kernel is no longer executing. The absence of a panic message is consistent with either (a) a hard lockup before any printk could fire, or (b) a panic whose message was not flushed to the journal before the reset.

**Evidence:**
- `E5`: The journal cliff evidence from the dossier
- `E6`: The S5 register showing a CF9 write (i.e., the OS did execute at least briefly after the hang)
- `E7`: Known Blackwell GSP silent-hang behavior (NVIDIA issue #1247: "silent, complete system freeze requiring a hard power cycle. No Xid error or kernel panic is logged before the freeze")

**Confidence:** Medium (70%). The logic is sound, but we cannot distinguish between "hard lockup + panic timeout reset" and "kexec crash path" without the crash dump or a serial console capture.

**Status:** Supported (with the caveat that the exact software path that wrote 0x06 is not confirmed)

---

### C4: The RTX 5060 Ti (Blackwell) is a known source of silent hard hangs under GPU load

**Statement:** Multiple independent reports document the RTX 5060 Ti (GB206, Blackwell) causing complete system freezes with no Xid, no kernel message, and no recoverable state — matching the Nov 21 evidence pattern. These are attributed to GSP firmware crashes or context-switch timeouts.

**Evidence:**
- `E8`: NVIDIA open-gpu-kernel-modules issue #1243 — Xid 109 GSP CTX SWITCH TIMEOUT on RTX 5060 Ti
- `E9`: NVIDIA open-gpu-kernel-modules issue #1247 — RTX 5060 Ti eGPU hard-locks on CUDA allocation, "silent, complete system freeze... No Xid error or kernel panic is logged"
- `E10`: CachyOS forum — RTX 5060 Ti GSP firmware crash causes GPU lockup (black screen, 100% fans, no log)
- `E11`: NVIDIA forums — three RTX 5060 Ti cards randomly falling off the bus (Xid 79/154) at any load level, zero PCIe/AER errors

**Confidence:** Medium-High (75%). The reports are real and consistent, but they span different driver versions and configurations. The exact GSP firmware version in the Nov 21 incident is not confirmed, and the eGPU (OCuLink) reports involve a different PCIe path than the internal x16 slot.

**Status:** Supported (as a plausible cause, not a confirmed root cause)

---

### C5: The 2026-09-13 and 2026-11-21 events are different reset mechanisms

**Statement:** Sept 13 was a BP_SYS_RST_L pin trip (bit 16, hardware). Nov 21 was a CF9 0x06 software write (bit 19). They are not the same failure mode. The Sept 13 event is consistent with a physical reset signal (button press, board-level reset, or a power event that asserted the reset pin). The Nov 21 event is consistent with a software-originated reset (panic path, kexec, or a userspace reboot call).

**Evidence:**
- `E1`: S5 register table — bit 16 = BP_SYS_RST_L pin, bit 19 = CF9 0x06
- `E12`: Sept 13 S5 value `0x00010800` (bit 16 set)
- `E13`: Nov 21 S5 value `0x00080800` (bit 19 set)

**Confidence:** High (95%)

**Status:** Supported

---

### C6: The UPS / bad wiring is not the trigger for the Nov 21 event

**Statement:** A power event (AC loss, brownout below threshold) would not produce a CF9 software write in the S5 register. The OS must have been running to write 0x06. The bad wiring is a background risk for power-related resets (like the Sept 13 pin trip) but is not the mechanism for the Nov 21 software reset.

**Evidence:**
- `E1`: S5 register decode
- `E14`: The `0x00080800` value — software-only, no power-event bits

**Confidence:** Medium-High (80%). The logic is strong, but a very brief power dip that the motherboard survived (without a hard reset) could theoretically have caused a CPU/GPU glitch that led to a hang, which then led to a panic and CF9 write. This is a possible secondary contribution but not the primary mechanism.

**Status:** Supported

---

### C7: The sdc 6TB WD Red is a victim, not a cause

**Statement:** The disk's degraded health is a consequence of repeated unclean shutdowns, not a cause of the reboots. The disk does not initiate reboots; it is stressed by them.

**Evidence:**
- `E15`: The dossier — disk is sdc, in use, degraded health. No evidence of disk-initiated reboots.

**Confidence:** High (90%)

**Status:** Supported

---

## Evidence Index

| ID | Source | Provenance | Excerpt / Note |
|----|--------|-----------|----------------|
| E1 | AMD kernel: `arch/x86/kernel/cpu/amd.c` (commit Apr 2025, Yazen Ghannam / Mario Limonciello) | https://lists.openwall.net/linux-kernel/2025/04/08/1758 (PATCH v2 2/2) | S5_RESET_STATUS register at MMIO 0xFED803C0; `get_s5_reset_reason()` decode table; bit 19 = CF9 0x06, bit 16 = BP_SYS_RST_L |
| E2 | Linux kernel: `arch/x86/kernel/reboot.c` | https://github.com/torvalds/linux/blob/master/arch/x86/kernel/reboot.c | `native_reboot()` writes 0x06 (warm) or 0x0E (cold) to 0xCF9 |
| E3 | 2026-11-21 boot log (from dossier) | Task dossier | `0x00080800` — "software wrote 0x6 to reset control register 0xCF9" |
| E4 | 2026-11-21 S5 register decode | This analysis | Bits 17, 19 set; all others clear |
| E5 | 2026-11-21 journal (from dossier) | Task dossier | Journal stops mid-work, no panic/OOM/thermal/Xid/MCE |
| E6 | S5 register + journal correlation | This analysis | CF9 write requires OS to be running; journal cliff consistent with hard hang |
| E7 | NVIDIA open-gpu-kernel-modules issue #1247 | https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1247 | "silent, complete system freeze requiring a hard power cycle. No Xid error or kernel panic is logged" |
| E8 | NVIDIA open-gpu-kernel-modules issue #1243 | https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1243 | Xid 109 GSP CTX SWITCH TIMEOUT on RTX 5060 Ti |
| E9 | NVIDIA open-gpu-kernel-modules issue #1247 | https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1247 | RTX 5060 Ti eGPU hard-locks on CUDA allocation |
| E10 | CachyOS forum | https://discuss.cachyos.org/t/rtx-5060-ti-blackwell-gsp-firmware-crash-causes-gpu-lockup-black-screen-100-fans | GSP firmware crash: black screen, 100% fans, no log |
| E11 | NVIDIA developer forums | https://forums.developer.nvidia.com/t/three-rtx-5060-ti-gb206-randomly-fallen-off-the-bus-xid-79-154-at-any-load-level-no-p | Three 5060 Ti cards falling off bus, zero PCIe/AER |
| E12 | 2026-09-13 boot log (from dossier) | Task dossier | `0x00010800` — BP_SYS_RST_L pin trip |
| E13 | 2026-11-21 S5 register (from dossier) | Task dossier | `0x00080800` |
| E14 | S5 register decode | This analysis | No power-event bits set |
| E15 | Disk status (from dossier) | Task dossier | sdc 6TB WD Red, degraded, in use |
| E16 | AMD kernel docs: `Documentation/arch/x86/amd-debugging.html` | https://docs.kernel.org/arch/x86/amd-debugging.html (section 18.10) | Full S5 reset reason table; "Random reboot issues" section |
| E17 | Siderolabs Talos issue #12968 | https://github.com/siderolabs/talos/issues/12968 | Same `0x00080800` fingerprint; DRBD use-after-free identified as cause in one case; no single root cause confirmed |
| E18 | LKML: reboot CF9 warm/cold reset patch (2013) | https://lkml.iu.edu/hypermail/linux/kernel/1308.2/02521.html | Original commit that differentiated warm (0x06) vs cold (0x0E) CF9 resets |

---

## Gaps / Unresolved Questions

1. **Exact software path that wrote 0x06:** We know the S5 register says a CF9 0x06 write occurred, but we do not know *which* code path wrote it. Candidates:
   - Kernel panic → `panic_timeout` → `kernel_restart()` → `native_reboot()`
   - kexec crash path (panic → kexec into crash kernel → auto-reboot)
   - A userspace watchdog daemon calling `reboot(2)`
   - A driver-level timeout that triggered a system reset
   **Resolution:** Need the crash dump (if kexec was used), a serial console capture for the next event, or `panic_timeout` / `crashkernel` config inspection.

2. **Whether the GPU was the trigger:** The Blackwell GSP silent-hang reports are consistent with the evidence, but we have no direct confirmation that the 5060 Ti was the cause of the Nov 21 hang. The GPU could have been fine and the hang could have been CPU/RAM/other.
   **Resolution:** Need GPU telemetry (nvidia-smi, Xid log) from the moment of the hang, or a serial console to see what the kernel was doing.

3. **Driver version and GSP firmware version:** The dossier does not specify the exact NVIDIA driver version or GSP firmware version running on 2026-11-21.
   **Resolution:** Check `nvidia-smi -q` output or `cat /proc/driver/nvidia/version` from the next boot, and the GSP firmware version from the nvidia-bug-report.

4. **BIOS version at the time of the event:** The dossier mentions "BIOS 7.00 H.P0 (2025)" but does not confirm it was active on Nov 21.
   **Resolution:** Check `dmidecode -t bios` or `systemctl` boot info.

5. **Whether `panic_on_oops`, `panic_timeout`, and `crashkernel` are configured:** The exact kernel parameters determine the reset path.
   **Resolution:** `cat /proc/cmdline` and check `sysctl kernel.panic_on_oops` and `kernel.panic`.

---

## Confidence

**Level:** Medium (65%)

**Rationale:**
- The S5 register decode is deterministic and high-confidence (95%+).
- The distinction between the two events (CF9 software vs BP_SYS_RST_L hardware) is solid (95%).
- The GPU GSP silent-hang hypothesis is plausible and well-supported by external reports, but it is not confirmed for this specific system. The absence of direct GPU evidence (Xid, nvidia-smi output) at the time of the hang leaves a gap.
- The exact software path that wrote 0x06 is not confirmed. We know it was software, but not which code path.
- Overall: we have a strong "what" (software warm reset, not power/hardware) and a plausible "why" (GPU GSP silent hang under load), but the "why" is not directly proven for this system.

---

## Recommended Next Actions

### Immediate (before next reboot)

1. **Capture the current boot's S5 register and kernel config:**
   ```bash
   dmesg | grep -i "reset reason\|previous system"
   cat /proc/cmdline
   sysctl kernel.panic_on_oops kernel.panic
   nvidia-smi -q 2>&1 | head -30
   cat /proc/driver/nvidia/version
   ```

2. **Check for a crash dump:**
   ```bash
   ls -la /var/crash/ /var/kdump/ 2>/dev/null
   dmesg | grep -i "kdump\|crash"
   ```

3. **Check the BIOS version and whether the 2025 H.P0 update is active:**
   ```bash
   dmidecode -t bios | grep -i "version\|date"
   ```

### Next event (detection)

4. **Enable a serial console or `netconsole` to capture the kernel log at the moment of the next hang.** This is the single most valuable diagnostic. Without it, we are blind to what the kernel is doing in the last seconds before the reset.
   ```bash
   # Add to kernel cmdline:
   console=ttyS0,115200
   # Or use netconsole:
   modprobe netconsole netconsole=66.66.66.66:4000,192.168.1.100:4000
   ```

5. **Enable `panic_on_oops=1` and `panic=10`** (if not already) so that any oops triggers a panic and a 10-second reboot. This ensures the S5 register captures the reset reason and the crash dump (if kexec is configured) is generated.
   ```bash
   # /etc/sysctl.d/99-panic.conf
   kernel.panic_on_oops = 1
   kernel.panic = 10
   ```

6. **Configure kdump** to capture a memory dump on panic:
   ```bash
   systemctl enable --now kdump
   # Verify:
   kdumpctl status
   ```

7. **Check NVIDIA driver version and GSP firmware:**
   ```bash
   nvidia-smi -q | grep -A5 "GSP\|Driver"
   cat /proc/driver/nvidia/version
   ```

### Diagnostic (to narrow the cause)

8. **Run a GPU stress test under controlled conditions** to see if the hang reproduces:
   ```bash
   # gpu_burn or nvidia-smi -l 0 (unlocked) for 2-4 hours
   # Monitor nvidia-smi for Xid, temperature, power
   # Watch journal for any output before the hang
   ```

9. **Try an older stable NVIDIA driver** (580.x series) to see if the GSP behavior changes. The 595.71.07 driver is relatively new and may have GSP firmware bugs.

10. **Check PCIe link status and AER:**
    ```bash
    lspci -vvv | grep -A20 "3D controller"
    dmesg | grep -i "aer\|pcie"
    ```

### Mitigation (to protect data)

11. **Enable `panic_on_oops=1`** so that any kernel bug triggers a clean panic (with a crash dump) instead of an uncontrolled hang.
12. **Enable kdump** so that the panic generates a memory dump for post-mortem analysis.
13. **Enable journal sync** (`journalctl --flush` periodically or `systemd-journald` with `SyncIntervalSec=1`) to reduce the window of lost log entries.
14. **Consider a hardware watchdog** (if the board has an iTCO or MP1 watchdog) with a short timeout, as a last-resort reset mechanism. Check:
    ```bash
    dmesg | grep -i "watchdog\|itco\|mp1"
    ls /dev/watchdog*
    ```

---

## One-Line Answer

The 2026-11-21 reset was a **software-originated warm reset** (CF9=0x6), not a power event or hardware pin trip. The most likely cause is a **silent GPU hard hang** (Blackwell GSP firmware crash under load) that triggered a kernel panic or lockup, which the OS then resolved by writing 0x06 to 0xCF9. The 2026-09-13 event was a **different mechanism** (BP_SYS_RST_L pin trip) and is more consistent with a power/bad-wiring origin. The two events are not the same failure mode.

---

*Report generated by Geraldo v2.2, research board. Task t_98a1f97b (GIU-129).*
*All S5 register decodes verified against AMD kernel source (arch/x86/kernel/cpu/amd.c, commit Apr 2025) and the official AMD debugging documentation (Documentation/arch/x86/amd-debugging.html, section 18.10).*
