# Display — Selection and Integration (Phase 6)

Status: **Phase-6 design reference — draft, partially verified** — 2026-08-30.

> ⚠️ **Panel change (2026-08-30, DEC-044):** display is now **Raystar RFU800G-AYH-MNN** (1125 nit, 800×1280, DSI 4-lane, OD 115.74×184.93×4.75 mm) replacing the Vu8S. Envelope widened per DEC-043 (~200×140×50, bezels 3/3/3/10). Vu8S-specific sections below are superseded; DSI/FPC/backlight-PWM integration still applies (same 4-lane DSI). Confirm OD/power/init on bench.

References: DEC-024 (DSI accepted), DEC-035 (cheapest option — superseded by DEC-044), **DEC-043 (envelope), DEC-044 (Raystar panel)**, A-028, RISK-018 (brightness/measure), RISK-024 (panel FPC), RISK-001 (2.5 W budget).
Related Phase-6 docs: `hardware/pcb.md` (DSI impedance/skew §3; DSI-FFC adapter BOM §11), `electrical/block-diagram.md` (EDGE-02).
Strict evidence rule: any value not measured in-house is `TBD` or carrier-dated with a source; no "Recommended" classification without a bench.

| Symbol | Convention |
|---|---|
| E-xxx | evidence references in `info/sources.md` |
| TBD | to verify (not measured / not confirmed) |
| 🔲 | open item for Phase 6 verification |

---

## 1. Summary

- **Selected: Raystar RFU800G-AYH-MNN (8", 800×1280, 4-lane MIPI-DSI)** per DEC-044 — the brightest option found (1125 cd/m²) and the panel that drove the envelope widening to **~200×140×50 mm / bezels 3/3/3/10 (DEC-043)**.
- **Verified (vendor side, accessed 2026-08-30):** DSI 4-lane; 800×1280 (5:8 portrait, 16:10 landscape), 189 ppi; **OD 115.74 × 184.93 × 4.75 mm**; active 107.64 × 172.22 mm.
- **Power:** backlight ~4.32 W max @ full 1125-nit duty; **~2.2 W @ ~50 % PWM (≈560 nit)** — fits the ≤2.5 W runtime target at a daylight-visible working brightness (RISK-001). **Measure on bench (RISK-018).**
- **Critical integration items:**
  1. **Panel FPC connector (RISK-024):** the carrier brings Verdin DSI (4-lane) out via a **0.5 mm FFC adapter** to the Raystar panel's DSI connector; exact pitch/pin-count **TBD until the panel datasheet (or a sample) is in hand** (§3).
  2. **OD/power gate:** confirm OD fits the lid window and backlight power at working brightness before CNC + layout freeze (§6.1/§8).
  3. DSI and HDMI are **independent** on Verdin i.MX8MP — both usable simultaneously (§3.5).
  4. **Vu8S is historical (§2.1)** — superseded as leading by DEC-044; retained only as a low-cost fallback.

---

## 2. Selected panel — Raystar RFU800G-AYH-MNN (evidence)

Source: vendor quote / spec per DEC-044 and E-014 (`info/sources.md`, accessed 2026-08-30; confidence **High** for vendor rows, price **stale**).

| Attribute | Value | Source / confidence |
|---|---|---|
| Panel | 8.0" TFT-LCD | vendor spec 🔲 |
| Resolution (native) | **800(W) × 1280(H)** PORTRAIT (5:8), ≈ **189 ppi** | vendor (verified) |
| Interface | **MIPI-DSI, 4-lane** (native to Verdin) | vendor (verified) |
| Brightness | **1125 cd/m²** | vendor (verified, E-014) |
| Active area | 107.64 × 172.22 mm (portrait) | vendor 🔲 |
| Outer size (OD) | **115.74(W) × 184.93(H) × 4.75(T) mm** (portrait) | vendor (verified; DEC-044) |
| Touch | none required (REQ-ENC-03-friendly) | — |
| Power (backlight) | ~**4.32 W max** @ full 1125-nit duty; **~2.2 W @ ~50 % PWM (≈560 nit)** | DEC-044 estimate 🔲 measure |
| Price | **quote-only ~$35–60 (stale)** | E-014 🔲 re-quote |
| Availability | vendor quote (TBD) | 🔲 |

> **Fit sanity (DEC-043):** landscape OD = 184.93 (W) × 115.74 (H) mm vs lid window ≈ **188 × 121 mm** (interior 194×134 minus 3/3/3/10 bezels) → **fits with a small margin**; confirm OD on the received part before CNC (§6.1).

### 2.1 Historical — Hardkernel Vu8S (superseded by DEC-044)

The **Hardkernel Vu8S** (8", 800×1280, DSI 4-lane, official kit incl. backlight + touch, $39) was the DEC-035 cheapest pick; **superseded as leading by DEC-044** (Raystar is ~3× brighter at similar power). **Historical/fallback only.** Data below is retained from the Phase-5 research:

| Attribute | Value | Source / confidence |
|---|---|---|
| Panel | 8.0" TFT-LCD, wide-viewing-angle | HK spec 🔲 |
| Resolution (native) | 800(H) × 1280(V) PORTRAIT (5:8), ≈ 189 ppi | HK spec (verified) |
| Interface | MIPI-DSI, 4-lane | HK spec (verified) |
| Power | 2.4 W ±10 % at 100 % backlight duty | HK spec (verified) |
| Brightness | **not specified** (~300–400-nit class) → `TBD` measure | RISK-018 |
| Outer size (module incl. bracket) | 202.0(W) × 153.0(H) mm | HK spec (verified) — oversized |
| Price | $39 + DHL → landed ≈ €59–65 (B-004) | E-030, B-004 |
| Availability | in stock (Hardkernel) | E-016 |

---

## 3. Driving it from Verdin (i.MX8MP) — concept

### 3.1 Signal chain

```
Verdin SoM (DSI_0, 4 lanes + CLK)   ← i.MX8MP B2B socket
        │  DSI_0_D0..D3 (diff pairs) + DSI_0_CLK
        ▼
CARRIER PCB (4–6 L, controlled impedance ~100 Ω diff, DSI layers)
        │  short (<~50 mm) to the carrier-side FFC header
        ▼
CARRIER FFC ADAPTER (0.5 mm-pitch FFC/FPC → Raystar panel connector)
        │  0.5 mm FFC, ≤150 mm (through hinge into lid)
        ▼
Raystar RFU800G panel unit (lid)
```

- **Adapter approach:** the carrier carries a **0.5 mm-pitch FFC/FPC header** matched to the Raystar panel's DSI connector. The inserted FPC carries DSI lanes + backlight PWM/analog + reset + GND (+ touch I2C if a touch variant is used).
  - 🔲 **TBD:** exact Raystar connector pitch / pin-count must be checked against the Verdin DSI mapping (§3.2). RISK-024 → "custom DSI FPC adapter on carrier" is budgeted in the BOM (pcb.md §11).
- **No HDMI→DSI bridge chip needed** (Verdin has native DSI — E-009; fallback I-04 is not needed). Do **not** populate a separately-priced HDMI→DSI board.

> Note for anyone reusing the older Vu8S adapter notes: the Vu8S kit was only drivable through the ODROID-M1S/M2 J7 connector; the Raystar panel is a **standard DSI panel** (no board-kit interposer) — only the connector pitch/pin-count needs locking.

### 3.2 Pin mapping (tentative → TBD exact SODIMM numbers)

Verdin DSI pins come from the Verdin i.MX8MP datasheet, "Pinout / Electrical characteristics" section (login-gated PDF, docs.toradex.com/116795 — E-009). Until released as carrier detail, the SODIMM pin numbers are `TBD` (groups below as structured overview).

| Function | Verdin side (SODIMM group) | Adapter side (to Raystar panel) | Notes |
|---|---|---|---|
| Lane 0 | DSI_0_D0P / DSI_0_D0N | lane0± | diff, ~100 Ω |
| Lane 1 | DSI_0_D1P / D1N | lane1± | |
| Lane 2 | DSI_0_D2P / D2N | lane2± | |
| Lane 3 | DSI_0_D3P / D3N | lane3± | |
| Clock | DSI_0_CLKP / CLKN | CLK± | |
| Touch I2C (if used) | spare I2C bus on SoM | touch SDA/SCL | touch GPIO-interrupt compat TBD |
| Backlight | Verdin PWM pin (e.g., SODIMM "backlight PWM" function) → BL driver/PWM | BL_PWM (or direct LED+/LED−) | 3.3 V logic max; check level |
| Enable / Reset | GPIO | DISP_EN / RESET | polarity from datasheet |
| Power | 5 V (or 3.3 V) filtered, from carrier DC-DC | VCC | current per §4; short run |
| GND | GND | GND | common, tied at FFC |

> **Process note:** obtain the **Raystar datasheet/FPC pinout** and the **Verdin datasheet pinout**, then cross-check — this is verification step V-2 (§8). Until then the table is a useful structured group, not a committing pin list.

### 3.3 Init / timings (DT concept)

- **Single panel:** 800×1280 @ ~60 Hz (portrait, native). Pixel clock (with typical blanking) ≈ 70–75 MP/s → 24-bit ≈ 1.7–1.8 Gbit/s → **4 lanes × ≈430–450 Mbit/s per lane** — far below the i.MX8MP DSI ceiling (supports 1080p60). No lane-rate problem.
- **Linux DT panel node:** approach = **extract the DCS init sequence + timings from the Raystar datasheet (request from vendor)** and port into the panel node on the Verdin DT (`imx8mp-verdin.dtsi`), using the i.MX8MP DSI controller.
  - 🔲 **TBD:** exact panel part number / DCS init sequence from the vendor datasheet; port to `imx8mp`.
  - Many 800×1280 portrait DSI panels use near-identical 4-lane init — mainline panel bindings are a good cross-check, but the vendor datasheet is authoritative.
- **Touch:** only if a touch variant is used; mainline I2C touch driver then applies (chipset TBD).

### 3.4 Backlight PWM and brightness

- **Backlight PWM from the carrier** (not pre-programmed panel driver): PWM output on a Verdin PWM pin (backlight function) → PMIC/FET driver on the carrier → panel LED chain. Regular `pwm-backlight` DT node + `backlight` sysfs interface.
- **Default:** low duty (dim at boot) to avoid pulling ~2.2–4.3 W during boot; brightness via standard `brightness` (0–255) sysfs path.
- **Working-brightness target:** **~50 % PWM ≈ 560 nit** (≈2.2 W) — daylight-visible and within the power budget (§4).

### 3.5 DSI AND HDMI in parallel

- The "MIPI-DSI disables HDMI" note on the Vu8S kit applies only to the **RK3566 / M1S** memory-bandwidth limit — that caveat (now historical) does **not** apply to Verdin i.MX8MP: DSI (LCDIF + DSI) and the HDMI PHY are **independent** — both usable simultaneously (E-009/I-03/I-04). No architecture blocker.

---

## 4. Power budget at working brightness (TBD until measured)

| State | Power | Source |
|---|---|---|
| Full-duty backlight (100 %, ≈1125 nit) | **~4.32 W max** (vendor class) | DEC-044 🔲 measure |
| Working brightness (~50 % PWM, ≈560 nit) | **≈2.2 W** (DEC-044 estimate, heuristic until measured) | `TBD` — bench (§8) |
| Runtime target (RISK-001 / A-008) | **≤2.5 W @ working brightness** | driven by 30 h math |
| Extra: touch controller (if used) | <0.1 W (typical; measure) | `TBD` |

- **Consequence:** Raystar meets the target at ~50 % PWM with **~2.4× more headroom brightness than the Vu8S** (which was ~300–400-nit at 2.4 W full duty). This substantially de-risks the display side of RISK-001. Verification still mandatory (§8): if >2.5 W at the working brightness, dim or revisit the runtime trade.
- Brightness and power **must be measured together** (§8), because BL duty ↔ lumen ↔ wattage.

---

## 5. Alternatives / upgrade path (only if Raystar fails)

| Option | Ratio | Brightness | Interface | When to use | Source |
|---|---|---|---|---|---|
| **(chosen) Raystar RFU800G-AYH-MNN** | 5:8 (portrait) | **1125 nit** | DSI 4-lane | default | DEC-044, E-014 |
| **HE080IA-01E** (Innolux) | **4:3** (1024×768, 160 ppi) | **bare cell — no BL** | DSI 4-lane | if 4:3 matters more than brightness; add BL (BL power TBD) | E-013 |
| **HOTHMI** HTM-H080D14-LVDS-A01R | 4:3 (1024×768) | **1000 nit** | LVDS | if brightness beats ratio preference → needs LVDS driver board | E-015 |
| **Hardkernel Vu8S** (historical) | 5:8 (portrait) | ~300–400 (TBD) | DSI kit | cheap fallback (DEC-035, superseded by DEC-044) | E-016 |

- **Switch trigger (bench §8):** Raystar < ~300 nit at ≤2.5 W → consider a 4:3 / Vu8S / LVDS switch; RISK-018.
- All alternatives use a standard 0.5 mm FFC/DSI path → the carrier adapter approach stays; only pinout/panel node changes.

---

## 6. Mounting into the lid (→ mechanical)

### 6.1 FIT CHECK (bench gate)

- Envelope: **~200×140×50 mm closed** (DEC-043); lid interior = **194 × 134 mm**; bezels 3/3/3/10 → usable lid window ≈ **188 × 121 mm**.
- **Raystar (landscape OD = 184.93 W × 115.74 H × 4.75 mm) fits the window with margin**, clearing the 3/3/3/10 bezel in all four directions.
- (Historical) the Vu8S module incl. bracket was **202 × 153 mm** and its 172 mm panel axis could not fit the old 170 mm lid — that is exactly why the envelope widened (DEC-043) and the brighter Raystar was chosen (DEC-044).
- **Action (Phase-6 bench, gate):**
  1. Confirm the **received OD** (184.93 × 115.74 mm) and required relief around the FPC/ZIF tail and backlight wiring.
  2. Lock clamping, hinge FPC run (≤150 mm), and EMI on BL wiring with `mechanical/`.
- 🔲 **Handoff to `mechanical/`:** lid opening, mounting angle, hinge FPC run (≤150 mm), clamping, EMI on BL wiring.

### 6.2 Connection steps (adapter topology §3.1)

- Carrier side: 0.5 mm FFC header (top side, short DSI run) → shielded/tightly twisted FFC through the hinge → panel unit in the lid. FFC ≈ ≤150 mm.
- Panel unit in the lid: aluminum frame (CNC) with slots for the panel; optional cover glass (not mandatory). The plastic bezel + RF-transparent plastic lid-top (DEC-055) sit around/behind the panel for the antennas.

---

## 7. Cost estimate

| Item | € (landed) | Source |
|---|---|---|
| **Raystar RFU800G** (quote ~$35–60, stale) | ≈ 32–52 + shipping — **re-quote** | E-014 🔲 |
| Carrier FFC adapter (header + FFC) | ≈ **2–6** (BOM leftover) | estimate, part of PCB |
| (Fallback) Vu8S kit ($39 + DHL) | ≈ **59–65** | B-004 (≈62), historical |
| (Fallback) 4:3 option: HE080IA-01E + BL | 25–45 + BL | E-013 `TBD` |

---

## 8. Schedule / measurement plan (bench)

Goal: validate brightness, panel power, DSI timing/init, FFC-length viability on the real part **before** finalizing carrier routing.

| Step | Action | Deliverable | Due |
|---|---|---|---|
| V-1 | Order Raystar RFU800G sample (vendor quote) | part in hand | start of Phase 6 |
| V-2 | Obtain Raystar datasheet/FPC pinout; cross-check vs Verdin DSI pinout | adapter mapping released | V-1 + 1 week |
| V-3 | **Lux meter bench:** full-duty luminance (cd/m²), 25/50/75/100 % duty; power at each (watt-meter on 5 V / BL branch) | luminance + power curve | V-1 + 2 weeks |
| V-4 | Extract init/timings from the Raystar datasheet; test on Verdin eval board / carrier proto | working DSI panel node on Verdin | after V-3 |
| V-5 | Lid fit test (OD 184.93×115.74 vs 188×121 window) + FFC routing prototype | mechanical gate released | V-3 |
| V-6 | Soak: 60 min at working brightness via the 30 h workload profile; watt-meter | power figures into runtime model (RISK-001) | V-4 |

Gates: **V-3 and V-5 must be green**, otherwise activate the switch path (§5).

---

## 9. Open items

1. 🔲 **OD/power bench (6.1):** confirm Raystar OD (184.93×115.74) + backlight power at working brightness — gate before CNC/layout freeze. **P1.**
2. 🔲 Raystar connector pinout vs Verdin SODIMM pinout (3.2) — exact pin numbers to be fixed.
3. 🔲 Measure luminance in cd/m² (≈1125 vendor; power ≈ TBD measured).
4. 🔲 Panel part number / DCS init sequence from the vendor datasheet (3.3).
5. 🔲 BL driver topology (PWM→FET) and default duty (3.4).
6. 🔲 Re-quote Raystar price (E-014 quote is stale).