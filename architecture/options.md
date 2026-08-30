# Architecture Options (Phase 3)

Status: **Options record, reconciled to Phase-6 decisions** — 2026-08-30. Leading rows reflect the approved current state where decided (e.g., DEC-033/044/046/058); options/alternatives are retained as research for trade-off context, not current design.

## 0. Evaluation criteria (all options scored on these)

1. Runtime impact (idle/system power) — REQ-PWR-02 is the tightest requirement.
2. NixOS + mainline Linux support — REQ-COMP-04/05.
3. Routable for the user's 4–6L carrier — REQ-COMP-06 (no BGA, B2B with practical pitch).
4. Native vs bridged interfaces (NVMe HDMI USB3 OTG ETH) — fewer chips → lower cost/power/risk.
5. Cost + regional availability.
6. Envelope/thermal fit in **200×140×50 mm** aluminum clamshell (DEC-043).

## 1. Compute (SoM family)

| Option | Architecture | Interfaces (native) | Idle power (est.) | NixOS/mainline | Carrier-routable | Rough cost | vs runtime | Verdict |
|---|---|---|---|---|---|---|---|---|---|
| **A. i.MX 8M Plus SoM (leading)** | 4×A53 @1.8 GHz | **PCIe Gen3 ×1 (NVMe)**, HDMI TX, USB3 host, USB2 OTG, RGMII, dual DSI, dual GbE | ~1.5–2.5 W (verify) | **Strong** (mainline U-Boot + extlinux, **no config.txt** — DEC-034) | 0.5 mm edge-socket B2B (SODIMM-like) — routable at 4–6L | mid | **Feasible for 30 h** | **Leading** (DEC-019/033/034) — no SATA; NVMe direct |
| B. Raspberry CM5 (BCM2712) | 4×A76 @2.4 GHz | PCIe x4, USB3 via RP1, 2×4K HDMI; no native SATA/ETH | ~2.5–3.5 W (verify) | Strong; generic arm64 NixOS | 100-pin 0.5 mm B2B — conceptually routable, many 4L carriers exist | low | Risk: idle too high | Conditional |
| C. RK3588 SoM | 2+6× big.LITTLE | SATA×3, HDMI, USB3, PCIe | ~3–5 W (verify) | Partial-mainline | 0.5/0.8 mm B2B available | mid | Likely fails 30 h | Conditional (reject if runtime priority) |
| D. i.MX 8M Mini | 4×A53 | No SATA (only MMC), no HDMI TX | ~1.5 W | Strong | 0.5 mm B2B | low-mid | Needs bridges for SATA + display | Rejected (bridges add power/cost) |
| E. i.MX 93 | 2×A55+M33 | No USB3 host, no SATA | very low | Emerging | B2B | low | SATA + USB3 impossible native | Rejected |

**Rationale:** Option A matches every constraint best: one module covers storage (PCIe→NVMe) + HDMI + USB3 + OTG + GbE, idles in the 1.5–2.5 W window that keeps 30 h reachable, boots mainline U-Boot/extlinux (no config.txt), and its edge-socket B2B fits the user's no-BGA routing constraint. Verified across Phase 4–6 (idle power, PCB-way B2B, mainline path); final price still to land (Phase 5).

## 2. Display path

| Option | Description | Fit | Verdict |
|---|---|---|---|
| **A. Native MIPI-DSI → Raystar RFU800G-AYH-MNN (8", 800×1280, 1125 nit)** | SoM has native DSI 4-lane; no driver board, no bridge chip; built-in backlight | Brightest DSI at this format; OD 115.74×184.93 fits the 200×140 lid (window 3/3/3/10) | **Leading (DEC-044)** |
| B. 4:3 DSI HE080IA-01E (1024×768) + matching backlight | True 4:3 ratio; bare cell, BL power TBD | 4:3 preference preserved as historical alt | **Alternative** (ratio-upgrade, not leading) |
| C. LVDS HOTHMI HTM-H080D14 (1000 nit) | Needs LVDS driver board | Brightness beats ratio | **Alternative** (E-015) |
| D. HDMI→eDP/LVDS XGA 4:3 + driver board | SOM has native HDMI; generic driver boards cheap | Adds a small board; no longer needed (DSI chosen) | Historical (superseded by DEC-044) |

Note (REQ-DISP-01): the original 4:3 XGA preference is preserved **as preference**; DEC-044 chose the brightest 800×1280 (5:8) DSI panel (Raystar) at the Phase-6 gate. 4:3 remains available as an acceptable ratio override if ever required.

## 3. Power topology

| Option | Description | Fit | Verdict |
|---|---|---|---|
| A. **Dual 4S packs, OR-ing ideal-diode bridge + pre-charge, PD 65 W sink, power-manager MCU** | ThinkPad-style hot-swap; one charger; sequential/priority charge; MCU sequencing | Cleanest for true hot-swap + charging-while-use | **Leading** (DEC-012/018) |
| B. Dual 2S2P packs | 7.4 V rail, higher currents | Higher loss/overs = more infra | Falls back only if 4S rails problematic |

## 4. Hot-swap power-bridge detail (leading option internals)

- Per-pack **ideal-diode OR FETs** + current sense (HS amplifier) + **pre-charge limit** on insert to avoid inrush back-feed.
- **Charge path:** single 65 W PD sink → charger IC(s) — sequential charge priority (one pack at a time fastest, or current-shared both) — under MCU control; taper on temperature (RISK-016).
- Behavior matrix (two packs, one pack, none, insert, remove) to be written in Phase 6, incl. "one pack back-feeds the other" exclusion.

## 5. Keyboard + trackball

| Option | Description | Fit | Verdict |
|---|---|---|---|
| A. Custom 6-row membrane, ~12 mm pitch, US-labelled + ISO-DE Enter, **≤20 mm optical trackball between G-H-B (below space bar)**, aluminum top-deck cutouts; keyboard 10% → 40% (palm rest = front 40%), SoM in right palm-rest | Only path to hybrid ISO-DE Enter legends | 6 rows fit the 140 mm depth with 10/40 placement; SoM + thermal tower in front-right 40% | **Leading** (DEC-021/025/045/053/061); deck MCU = STM32G0 USB-HID; final row-count/pitch with user in Phase 6 |
| B. Bought compact module (60%-style) | No hybrid legends; large | Rejected for REQ-KB-01 | Rejected |

## 6. Thermal (per user DEC-017/062)

- **Fan primary = Delta BFB0305HA-C blower** (30×30×10, 5 V, 0.65 W, 29 dBA, 1.45 CFM, 0.285 inH₂O); **Sunon HA30101V4 axial** (0.30 W, 15.1 dBA, 3.5 CFM) retained as **fallback** for a low-restriction arrangement (`hardware/thermal.md` §2).
- Air path (DEC-062): **left-back intake louvres → low-pressure draw → 30 mm fan → heatsink fins L→R → right-side exhaust**; fan is in-plane left of the 5–7 mm heatsink in the right palm-rest zone.
- Sustained internal dissipation target ≈ **10–14 W** (vs ~6–8 W sealed). Fan control (low-side PWM/voltage, tach TBD — RISK-027) by power-manager MCU.
- Charge-while-use (65 W / ~11–16 W internal) handled via fan-max + taper (RISK-016).

## 7. Chassis (per user DEC-016/023)

- **Black matte anodized aluminum**, CNC **locally free** (material + coating only) — enables full custom geometry: vents, RF windows/apertures, sunk key cutouts, hinge mounts, heatsink bosses.
- RF considerations: **aluminum is RF-opaque** ⇒ **plastic bezel + plastic lid-top behind the screen are RF-transparent** and host Wi-Fi/BT + LTE antennas (DEC-055); metal edges/rims stay aluminum for structure/hinge. (RISK-013)
- Structure doubles as heatsink; hinge at rear; intake/exhaust louvres left-back / right-side.
- Cost = material + anodizing + purchased hinges (est. €60–150), no labor. Budget flexibility approved for the shell (DEC-022).

## 8. Options matrix summary

| Area | Leading option | Alt | Remaining evidence needed |
|---|---|---|---|
| Compute | i.MX 8M Plus SoM (8 GB) — **NVMe on PCIe Gen3 ×1, no config.txt** | CM5, RK3588 | idle power, B2B pitch/size, dimensions, NixOS path, price |
| Display | **Native DSI → Raystar RFU800G (1125 nit, 800×1280)** | 4:3 DSI HE080IA / LVDS HOTHMI | panel OD/power on bench (DEC-044) |
| Power | Dual 4S OR-ing + PD65 + e-sys MCU | 2S2P | charger IC, OR/PFET part, connector CAD |
| Hot-swap | Drill-rail mid-chassis packs, pre-charge + ideal-diode OR FET | — | inrush sim + bench proto (DEC-060) |
| Keyboard | Custom membrane + trackball below space (G-H-B) + STM32G0 deck MCU | — | membrane cost/tooling, trackball module, small-ball gate |
| Thermal | **30 mm Delta blower + 5–7 mm heatsink, left-back intake → right exhaust** | Sunon axial | fan CFM/static-pressure/noise (RISK-027), heatsink fit |
| Chassis | Black matte anodized Al CNC | sheet-Al hybrid | quotes, antenna placement on plastic lid-top, hinges |