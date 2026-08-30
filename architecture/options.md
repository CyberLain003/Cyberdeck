# Architecture Options (Phase 3)

Status: **Draft for approval** — 2026-08-30. Per the strict evidence rule, the "Leading" tag means *first priority to research in Phase 4*, **not** "Recommended". No claim here is verified until Phase-4 sourcing/measurement.

## 0. Evaluation criteria (all options scored on these)

1. Runtime impact (idle/system power) — REQ-PWR-02 is the tightest requirement.
2. NixOS + mainline Linux support — REQ-COMP-04/05.
3. Routable for the user's 4–6L carrier — REQ-COMP-06 (no BGA, B2B with practical pitch).
4. Native vs bridged interfaces (SATA HDMI USB3 OTG ETH) — fewer chips → lower cost/power/risk.
5. Cost + regional availability.
6. Envelope/thermal fit in 130×170×50 mm aluminum clamshell.

## 1. Compute (SoM family)

| Option | Architecture | Interfaces (native) | Idle power (est.) | NixOS/mainline | Carrier-routable | Rough cost | vs runtime | Verdict |
|---|---|---|---|---|---|---|---|---|
| **A. i.MX 8M Plus SoM (leading)** | 4×A53 @1.8 GHz | SATA3, HDMI TX, USB3 host, USB2 OTG, RGMII, dual DSI | ~1.5–2.5 W (verify) | Strong (mainline U-Boot, etnaviv GPU) | 0.5 mm edge-socket B2B (SODIMM-like) — routable at 4–6L | mid | **Feasible for 30 h** | **Leading** (DEC-019) |
| B. Raspberry CM5 (BCM2712) | 4×A76 @2.4 GHz | PCIe x4, USB3 via RP1, 2×4K HDMI; no native SATA/ETH | ~2.5–3.5 W (verify) | Strong; generic arm64 NixOS | 100-pin 0.5 mm B2B — conceptually routable, many 4L carriers exist | low | Risk: idle too high | Conditional |
| C. RK3588 SoM | 2+6× big.LITTLE | SATA×3, HDMI, USB3, PCIe | ~3–5 W (verify) | Partial-mainline | 0.5/0.8 mm B2B available | mid | Likely fails 30 h | Conditional (reject if runtime priority) |
| D. i.MX 8M Mini | 4×A53 | No SATA (only MMC), no HDMI TX | ~1.5 W | Strong | 0.5 mm B2B | low-mid | Needs bridges for SATA + display | Rejected (bridges add power/cost) |
| E. i.MX 93 | 2×A55+M33 | No USB3 host, no SATA | very low | Emerging | B2B | low | SATA + USB3 impossible native | Rejected |

**Rationale:** Option A matches every constraint best: one module covers SATA + HDMI + USB3 + OTG + ETH, idles in the 1.5–2.5 W window that keeps 30 h reachable, and its edge-socket B2B fits the user's no-BGA routing constraint. **Phase 4 must verify precise idle power, exact B2B pitch/size, dimensions, NixOS evidence (or a reproducible boot path), and price.**

## 2. Display path

| Option | Description | Fit | Verdict |
|---|---|---|---|
| A. 8" XGA 4:3 eDP/LVDS panel + **HDMI→eDP driver board** | SOM already has native HDMI; generic driver boards cheap; panel selection industrial | Brightness 500–800 nit achievable; adds a small board | **Leading** (DEC-020) |
| B. Native MIPI-DSI to a 4:3 8" panel | Larger signal-integration; fewer panels at this ratio | A 4:3 8" DSI low-power panel is uncommon | Investigate |
| C. LVDS direct | Older panels; driver board still needed | Medium fit | Alternative |

Note (REQ-DISP-01): true 720p is 16:9; a 4:3 panel is XGA (1024×768, 768 vertical lines) which satisfies "≥720 vertical lines" and "4:3". Reconfirm this interpretation to the user in Phase 4 gate.

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
| A. Custom 6-row membrane, ~12 mm pitch, US-labelled + ISO-DE Enter, **≤20 mm optical trackball**, aluminum top-deck cutouts | Only path to hybrid ISO-DE Enter legends | 6 rows × ~9.5 mm ≈ 57 mm of 130 mm; trackball at right/center | **Leading** (DEC-021); final row-count/pitch with user in Phase 6 |
| B. Bought compact module (60%-style) | No hybrid legends; large | Rejected for REQ-KB-01 | Rejected |

## 6. Thermal (per user DEC-017)

- **Micro fan** (25–30 mm blower/axial) + **heatsink 5–7 mm** over the SoM; heat path to aluminum chassis/frame; rear/side vents.
- Sustained internal dissipation target ≈ **10–14 W** (vs ~6–8 W sealed). Fan ~0.2–0.5 W continuous; PWM profile by power-manager MCU.
- Charge-while-use (65 W / ~11–16 W internal) handled via fan + taper (RISK-016).

## 7. Chassis (per user DEC-016/023)

- **Black matte anodized aluminum**, CNC **locally free** (material + coating only) — enables full custom geometry: vents, RF windows/apertures, sunk key cutouts, hinge mounts, heatsink bosses.
- RF considerations: **aluminum is RF-opaque** ⇒ antenna windows/apertures on top deck (WiFi/BT ceramic) and rear (LTE); elsewhere lid edge. (RISK-013)
- Structure doubles as heatsink; hinge at rear; vents rear/side/bottom.
- Cost = material + anodizing + purchased hinges (est. €60–150), no labor. Budget flexibility approved for the shell (DEC-022).

## 8. Options matrix summary

| Area | Leading option | Alt | Phase-4 evidence needed |
|---|---|---|---|
| Compute | i.MX 8M Plus SoM (8 GB) | CM5, RK3588 | idle power, B2B pitch/size, dimensions, NixOS path, price |
| Display | 8" XGA eDP/LVDS + HDMI→eDP driver | Native DSI | panel 500–800 nit stock, driver board, power |
| Power | Dual 4S OR-ing + PD65 + e-sys MCU | 2S2P | charger IC, OR/PFET part, connector CAD |
| Hot-swap | Pre-charge + ideal-diode OR FET | — | inrush sim + bench proto |
| Keyboard | Custom membrane + ≤20 mm trackball | — | membrane cost/tooling, trackball module |
| Thermal | 25–30 mm micro fan + 5–7 mm heatsink | mini-blower | fan CFM/noise/power, heatsink fit |
| Chassis | Black matte anodized Al CNC | sheet-Al hybrid | quotes, antenna windows, hinges |