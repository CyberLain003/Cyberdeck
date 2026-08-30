# Display — Selection and Integration (Phase 6)

Status: **Phase-6 design reference — draft, partially verified** — 2026-08-30.

> ⚠️ **Panel change (2026-08-30, DEC-044):** display is now **Raystar RFU800G-AYH-MNN** (1125 nit, 800×1280, DSI 4-lane, OD 115.74×184.93×4.75 mm) replacing the Vu8S. Envelope widened per DEC-043 (~200×140×50, bezels 3/3/3/10). Vu8S-specific sections below are superseded; DSI/FPC/backlight-PWM integration still applies (same 4-lane DSI). Confirm OD/power/init on bench.

References: DEC-024 (DSI accepted), DEC-035 (cheapest option), **DEC-043 (envelope), DEC-044 (Raystar panel)**, A-028, RISK-018 (brightness/measure), RISK-024 (panel FPC), RISK-001 (2.5 W budget).
Related Phase-6 docs: `hardware/pcb.md` (DSI impedance/skew §3; DSI-FFC adapter BOM §11), `electrical/block-diagram.md` (EDGE-02).
Strict evidence rule: any value not measured in-house is `TBD` or carrier-dated with a source; no "Recommended" classification without a bench.

| Symbol | Convention |
|---|---|
| E-xxx | evidence references in `info/sources.md` |
| TBD | to verify (not measured / not confirmed) |
| 🔲 | open item for Phase 6 verification |

---

## 1. Summary

- **Selected: Hardkernel Vu8S (8", 800×1280, 4-lane MIPI-DSI)** per DEC-035 — cheapest usable 8" DSI option; official kit incl. panel + backlight + touch + mounting brackets, $39 (≈€59–65 landed, B-004).
- **Verified (vendor side, accessed 2026-08-30):** 4-lane DSI; active area 172.224 × 107.64 mm (as listed in landscape coordinates; native PORTRAIT 800×1280 / 189 ppi); **2.4 W ±10 % at 100 % backlight duty** → practically matches the 2.5 W runtime target (RISK-001) — but **brightness is not specified** (300–400-nit class, RISK-018) → bench item.
- **Critical integration items:**
  1. **M1S-specific connector (RISK-024):** the kit is only drivable through the ODROID-M1S/M2 DSI connector (J7, 0.5 mm FFC) → the carrier needs a **DSI FFC adapter** that brings Verdin DSI out to the J7 pinout (§3).
  2. **Mechanical gate:** 8" panel (long axis 172.2 mm; 202×153 mm incl. bracket) vs. 170 mm lid → **does not fit** without removing the bracket (§6.1). Must be resolved before mechanical sign-off ("lid frame" decision).
  3. The Hardkernel note "MIPI-DSI disables HDMI" applies to the **RK3566 SoC of the M1S** (memory bandwidth) and **not** to Verdin i.MX8MP — there DSI and HDMI are independent (§3.5).

---

## 2. Vu8S — Data (evidence)

Source: Hardkernel shop product page <https://www.hardkernel.com/shop/vu8s-8inch-mipi-lcd-for-m1s/> (accessed 2026-08-30; booking confidence **High** for verified rows); also E-016/E-030 in `info/sources.md`.

| Attribute | Value | Source / confidence |
|---|---|---|
| Panel | 8.0" TFT-LCD, wide-viewing-angle | HK spec 🔲 |
| Resolution (native) | **800(H) × 1280(V)** PORTRAIT (5:8), ≈ **189 ppi** | HK spec (verified) |
| Interface | **MIPI-DSI, 4-lane** | HK spec (verified) |
| Active area | 172.224 × 107.64 mm (as listed, landscape coordinates) | HK spec (verified) |
| Touch | 5-finger capacitive, pre-assembled (I2C; controller TBD) | HK spec 🔲 |
| Power | **2.4 W ±10 % at 100 % backlight duty** (incl. BL) | HK spec (verified; key for runtime) |
| Brightness | **not specified** (~300–400-nit class) → `TBD` measure | RISK-018 |
| Outer size (module incl. bracket) | **202.0(W) × 153.0(H) mm** | HK spec (verified) — **oversized** (see §6.1) |
| Kit contents | Panel + touch pre-assembled; "Vu8S LCD Frame Board"; 2× "I form Bracket Board"; M3 hardware | HK spec |
| Price | $39 + DHL ~$10–15 → landed ≈ €59–65 (B-004) | E-030, B-004 |
| Availability | in stock (Hardkernel) | E-016 |

> **Pixel-pitch sanity check:** 25.4/(172.224/800) = 118 ppi and 25.4/(107.64/1280) = 302 ppi are inconsistent → the HK-listed numbers are in the **landscape** frame of reference of the mounted assembly; natively the panel is PORTRAIT (800 wide × 1280 tall). Measured against the **portrait** active size ≈107.6(W) × 172.2(H) mm → 189 ppi. Always re-verify against physical measurement on the bench.

---

## 3. Driving it from Verdin (i.MX8MP) — concept

### 3.1 Signal chain

```
Verdin SoM (DSI_0, 4 lanes + CLK)   ← i.MX8MP B2B socket
        │  DSI_0_D0..D3 (diff pairs) + DSI_0_CLK
        ▼
CARRIER PCB (4–6 L, controlled impedance 100 Ω diff, DSI layers)
        │  short (<~50 mm) to the carrier-side FFC header
        ▼
CARRIER FFC ADAPTER (0.5 mm-pitch FFC/FPC, J7-pinout compatible)
        │  0.5 mm FFC, ≤150 mm (through hinge into lid)
        ▼
Vu8S panel unit (lid)
```

- **Adapter approach:** the carrier carries a **0.5 mm-pitch FFC/FPC header with an ODROID-M1S-J7-compatible pinout**. The inserted FPC carries DSI lanes + I2C (touch) + backlight PWM/analog + reset + GND. The Vu8S kit already has its own FPC/FFC interconnect; we only replace the source end (M1S → Verdin).
  - 🔲 **TBD:** exact J7 pin count / pinout (M1S schematic) must be checked against the Verdin DSI mapping (§3.2). RISK-024 → "custom DSI FPC adapter on carrier" is budgeted in the BOM.
- **No HDMI→DSI bridge chip needed** (Verdin has native DSI — E-009; fallback I-04). Do **not** populate a separately-priced HDMI→DSI board (E-017).

### 3.2 Pin mapping (tentative → TBD exact SODIMM numbers)

Verdin DSI pins come from the Verdin i.MX8MP datasheet, "Pinout / Electrical characteristics" section (login-gated PDF, docs.toradex.com/116795 — E-009). Until released as carrier detail, the SODIMM pin numbers are `TBD` (groups below as structured overview).

| Function | Verdin side (SODIMM group) | Adapter side (to Vu8S/M1S-J7 style) | Notes |
|---|---|---|---|
| Lane 0 | DSI_0_D0P / DSI_0_D0N | lane0± | diff, 100 Ω |
| Lane 1 | DSI_0_D1P / D1N | lane1± | |
| Lane 2 | DSI_0_D2P / D2N | lane2± | |
| Lane 3 | DSI_0_D3P / D3N | lane3± | |
| Clock | DSI_0_CLKP / CLKN | CLK± | |
| Touch I2C | spare I2C bus on SoM | touch SDA/SCL | touch GPIO-interrupt compat TBD |
| Backlight | Verdin PWM pin (e.g., SODIMM "backlight PWM" function) → BL driver/PWM | BL_PWM (or direct LED+/LED−) | 3.3 V logic max; check level |
| Enable / Reset | GPIO | DISP_EN / RESET | polarity from datasheet |
| Power | 5 V (or 3.3 V) filtered, from carrier DC-DC | VCC | current per §4; short run |
| GND | GND | GND | common, tied at FFC |

> **Process note:** obtain the exact **ODROID-M1S J7 schematic** and the **Verdin datasheet pinout**, then cross-check — this is verification step V-2 (§8). Until then the table is a useful structured group, not a committing pin list.

### 3.3 Init / timings (DT concept)

- **Single panel:** 800×1280 @ ~60 Hz (portrait, native). Pixel clock (with typical blanking) ≈ 70–75 MP/s → 24-bit ≈ 1.7–1.8 Gbit/s → **4 lanes × ≈430–450 Mbit/s per lane** — far below the i.MX8MP DSI ceiling (supports 1080p60). No lane-rate problem.
- **Linux DT panel node:** approach = **extract init sequence + timings from the ODROID BSP (M1S kernel)** and port into the panel node on the Verdin DT (`imx8mp-verdin.dtsi`), using the i.MX8MP DSI controller. This minimizes integration work.
  - 🔲 **TBD:** exact panel part number inside the Vu8S (not listed in HK spec); DCS init sequence available from the M1S BSP → port.
  - Mainline reference for the M1S panel node: `arch/arm64/boot/dts/rockchip/odroid-m1s.dts` (mainline, RK3566) — timings/init extractable; watch lane polarity / clock / Hz differences when porting to imx8mp/verdin.
- **Touch:** if the touch controller is I2C → use a mainline touchscreen driver (chipset unknown → `TBD` impact; possibly I2C touch on a spare SoM I2C bus, see §3.2).

### 3.4 Backlight PWM and brightness

- **Backlight PWM from the carrier** (not pre-programmed panel driver): PWM output on a Verdin PWM pin (backlight function) → PMIC/FET driver on the carrier → panel LED chain. Regular `pwm-backlight` DT node + `backlight` sysfs interface.
- **Default:** low duty (dim at boot) to avoid pulling 2.4 W during boot; brightness via standard `brightness` (0–255) sysfs path.
- **Working-brightness target:** 300–400-nit class → measurable point (see §8).

### 3.5 DSI AND HDMI in parallel

- The "MIPI-DSI disables HDMI" note (HK) applies only to the **RK3566 / M1S** memory-bandwidth limit. **Verdin i.MX8MP**: DSI (LCDIF + DSI) and the HDMI PHY are **independent** — both usable simultaneously (E-009/I-03/I-04). No architecture blocker.

---

## 4. Power budget at working brightness (TBD until measured)

| State | Power | Source |
|---|---|---|
| Full-duty backlight (100 %) | **2.4 W ±10 %** | HK spec (verified) |
| Working brightness (~50–70 % duty, target) | **≈1.2–1.9 W** (heuristic until measured) | `TBD` — bench (§8) |
| Runtime target (RISK-001 / A-008) | **≤2.5 W @ working brightness** | driven by 30 h math |
| Extra: touch controller | <0.1 W (typical; measure) | `TBD` |

- **Consequence:** Vu8S at working brightness is **very plausibly ≤2.5 W**; this substantially de-risks the display side of RISK-001 (panel power was the single largest line in the 30 h math). Verification still mandatory (§8): if >2.5 W, dim or revisit runtime trade.
- Brightness and power **must be measured together** (§8), because BL duty ↔ lumen ↔ wattage.

---

## 5. Upgrades (path if needed)

No change while Vu8S measures as expected. Levers:

| Upgrade | Ratio | Brightness | Interface | When to use | Source |
|---|---|---|---|---|---|
| (leading) **Vu8S** | 5:8 (portrait) | ~300–400 (TBD) | DSI 4-lane | default | E-016 |
| **HE080IA-01E** (Innolux) | **4:3** (1024×768, 160 ppi) | **bare cell — no BL** | DSI 4-lane | if 4:3 matters more than price; add BL (BL power TBD) | E-013 |
| **HOTHMI** HTM-H080D14-LVDS-A01R | 4:3 (1024×768) | **1000 nit** | LVDS | if brightness beats ratio preference → needs LVDS driver board | E-015 |
| **Raystar** RFU800G-AYH-MNN | 5:8 (portrait) | **1125 nit** | DSI 4-lane | brightest DSI at same format | E-014 |

- **Upgrade trigger (bench §8):** Vu8S < ~250 nit at ≤2.5 W → consider 4:3 / bright switch; RISK-018.
- All alternatives without the Vu8S bracket/FFC fit a standard 0.5 mm FFC → the carrier adapter stays; only pinout/panel node changes.

---

## 6. Mounting into the lid (→ mechanical)

### 6.1 FIT GATE (critical)

- Envelope: lid interior = 170 × 130 mm (REQ-ENC-01).
- Vu8S: 202 × 153 mm outer with the "I form bracket" boards; panel long axis ≈172 mm (portrait active ≈107.6 × 172.2 mm).
- **Bottom line: the Vu8S does not fit the 170 mm lid:**
  - the 172 mm panel axis exceeds the ~160–165 mm usable lid inner width,
  - the 202 mm bracket assembly exceeds it outright.
- **Action (Phase 6 mechanical, gate):**
  1. **Remove the M1S bracket boards** (they are M1S-case-specific hardware) → evaluate panel + slim bezel alone.
  2. **Evaluate:** bare panel (~185–195 × ~115 mm estimate, `TBD` until the part is in hand) against the lid window. Likely tight → either a flush/inset lid window or a panel-rotation consideration (portrait upright = 172 high > 130, not usable).
  3. **Escalate:** if not solvable within the 170 mm lid → **Display-Fit Open Decision** (lid override / envelope change / 7"-class panel) — requires **user decision**, because REQ-ENC-01 is Hard.
- 🔲 **Handoff to `mechanical/`:** lid opening, mounting angle, hinge FPC run (≤150 mm), clamping, EMI on BL wiring.

> This is the most important **unresolved mechanical** question of the display integration — the datasheet facts above govern, the construction follows.

### 6.2 Connection steps (adapter topology §3.1)

- Carrier side: 0.5 mm FFC header (top side, short DSI run) → shielded/tightly twisted FFC through the hinge → panel unit in the lid. FFC ≈ ≤150 mm.
- Panel unit in the lid: aluminum frame (CNC) with slots for the panel/touch bonding; optional cover glass (not mandatory).

---

## 7. Cost estimate

| Item | € (landed) | Source |
|---|---|---|
| Vu8S kit ($39 + DHL) | ≈ **59–65** | B-004 (≈62) |
| Carrier FFC adapter (header + FFC) | ≈ **2–6** (BOM leftover) | estimate, part of PCB |
| (Fallback) 4:3 option: HE080IA-01E + BL | 25–45 + BL | E-013 `TBD` |

---

## 8. Schedule / measurement plan (bench)

Goal: validate brightness, panel power, DSI timing/init, FFC-length viability on the real part **before** finalizing carrier routing.

| Step | Action | Deliverable | Due |
|---|---|---|---|
| V-1 | Order Vu8S from Hardkernel (test unit) | part in hand | start of Phase 6 |
| V-2 | Obtain M1S J7 schematic/pinout; cross-check vs Verdin DSI pinout | adapter mapping released | V-1 + 1 week |
| V-3 | **Lux meter bench:** full-duty luminance (cd/m²), 25/50/75/100 % duty; power at each (watt-meter on 5 V / BL branch) | luminance + power curve | V-1 + 2 weeks |
| V-4 | Extract panel init/timings from M1S DT; test on Verdin eval board / carrier proto | working DSI panel node on Verdin | after V-3 |
| V-5 | Lid fit test (brackets removed) + FFC routing prototype | mechanical gate released | V-3 |
| V-6 | Soak: 60 min at working brightness via the 30 h workload profile; watt-meter | power figures into runtime model (RISK-001) | V-4 |

Gates: **V-3 and V-5 must be green**, otherwise activate the upgrade path (§5) or the lid decision (§6.1).

---

## 9. Open items

1. 🔲 **Lid fit (6.1):** Vu8S panel/bracket vs 170 mm lid — decision needed (override / envelope / panel swap). **P1.**
2. 🔲 J7 pinout vs Verdin SODIMM pinout (3.2) — exact pin numbers to be fixed.
3. 🔲 Measure luminance in cd/m² (2.4 W power verified, nit value = TBD).
4. 🔲 Panel part number / DCS init from M1S BSP (3.3).
5. 🔲 Touch controller chipset (I2C) + mainline driver (3.3/3.2).
6. 🔲 BL driver topology (PWM→FET) and default duty (3.4).