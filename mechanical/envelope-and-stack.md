# Mechanical — Envelope, Vertical-Board Stack and Lid (Phase 6)

Status: **Phase-6 mechanical design draft** — 2026-08-30. All dimensions are CAD-input planning values derived from the approved baseline (DEC-001/002/003/016/018/023/033/035/037/043/044/052/053/054/055/056/057/058/060) and Phase-4 part evidence. Figures marked **TBD** are unverified and must be confirmed by part datasheets or measurement (Phase 8) before CNC quote.

> ⚠️ **MAJOR architecture rework (2026-08-30, DEC-052/053/054/055/056/057/058/060).**
> The previous "flat horizontal carrier mid-deck" layout (DEC-018 reading, old ADO §4.2/§4.3) is **superseded**. Approved architecture:
> - **Three VERTICAL boards** (edge-on in the z-stack): motherboard/carrier (hosts the SoM), rear daughterboard (all low-frequency ports + power management + battery contacts), keyboard PCB (own board). Interconnect via FPC/J-connectors + cables. **There is no flat horizontal carrier plane.**
> - **SoM lives in the right palm-rest area** (keyboard PCB occupies only the front ~50% of deck depth), freeing the space around/above it for the **thermal tower**: 5–7 mm finned heatsink over the SoM + a Sunon 30×30 mm fan, rising upward from the palm-rest zone.
> - **ThinkPad-class hinges**: one-finger open on a desk without lifting the base, carry the whole deck by the opened screen, reinforced multi-point mounts, ~1 N·m-class friction per side.
> - **Lid**: plastic bezel + plastic lid-top sheet behind the display (RF-transparent — Wi-Fi/BT/LTE antennas inside the lid); aluminum edges/rims retained.
> - **Rainproof for multiple hours** with keyboard drainage (T480-style weep channels to the bottom edge).
> - **Batteries** ride on rails machined into the **middle chassis spine** (drill-tool-pattern slot-in/out, DEC-060); rear daughterboard contacts co-mate on insertion; latch + polarity key + hot-swap retained.
> - Envelope **≈ 200 × 140 × 50 mm (≤55)**, lid bezels **3/3/3/10 (T/L/R/B)** for the **Raystar RFU800G-AYH-MNN** panel.
>
> Old flat-board references (RF windows in the aluminum deck, horizontal carrier tray, Vu8S, RTL8821CU module, mPCIe LTE slot) are superseded; where kept they are noted. See DEC-052…060 and A-032…038.

> Convention used throughout: **X** = deck depth (front/typing edge = 0, rear/hinge = 140), mm; **Y** = deck width (left = 0, right = 200), mm; **Z** = height above the inside of the bottom plate (0 at floor), mm. Naming per `info/terminology.md`. Envelope is **200 (Y, width) × 140 (X, depth) × 50 (Z, closed)** — the 200 mm axis is width and the 140 mm axis is depth; see §1.2 for the orientation proof.

---

## 1. External envelope verification (REQ-ENC-01, DEC-043)

### 1.1 Envelope table

| Dimension | Approved (REQ-ENC-01) | Design target | Interior (3 mm wall) | Verdict |
|---|---|---|---|---|
| Width (Y) | ≤ 200 | 200 | 194 | ✅ |
| Depth (X) | ≤ 140 | 140 | 134 | ✅ |
| Height (Z, closed) | ≤ 50 (≤ 55 w/ 7 mm HS) | 50 (base deck top 42 + lid 8) | — | ✅ (≤55 headroom reserved for thermal tower, §3.4) |

### 1.2 Why 200 mm is the width — panel-fit proof (DEC-043/044, A-030/031)

- Raystar RFU800G-AYH-MNN: **OD 115.74 × 184.93 × 4.75 mm**, active 107.64 × 172.22 mm (800×1280 portrait-native). Used **landscape** (as the Vu8S-class kit does: long axis across the lid width → 16:10 when open).
- **Long axis fit:** 184.93 + 6 (3+3 L/R bezel) = **190.93 ≤ 194 interior** ✅ (and 190.93 + 2×4.5 rim = 199.9 ≈ 200 exterior).
- **Short axis fit:** 115.74 + 13 (3 top + 10 bottom bezel) = **128.74 ≤ 134 interior** ✅.
- Two battery packs 70 wide side-by-side: 2×70 + 10 spine + 2×2 clear = **~154 ≤ 194** ✅ (old 170 mm envelope could not hold the panel; opening the width to 200 was the approved resolution).
- Keyboard block ≥ 150 mm wide (14 columns) fits 194 interior ✅.
- ⇒ Reverse reading (200 = depth) cannot hold two packs across 134 nor a 172 mm keyboard — rejected. **200 = Y (width), 140 = X (depth).**

Notes:
- Interior height: floor to top deck = **3 (bottom) + 27 (bay) + 5 (spine/rails) + 0 … 42** — real usable base interior ≈ **39 mm** above the floor in the rear (vertical-board zone) and ≈ **29–32 mm** above the mid-chassis spine in the front (battery level). This is a z-stack budget, not a volume bookkeeping figure (§3).
- No margin exists for a part wider than spec; tolerances must be held with pass/fit sketches before CNC (RISK-002).
- Envelope may rise to **≤ 55 mm** over the palm-rest thermal tower if the 7 mm heatsink + fan require headroom (DEC-043).

---

## 2. Board architecture — three vertical boards (DEC-052)

### 2.1 Roles

| Board | Orientation / position | Contents | Connectors out |
|---|---|---|---|
| **Motherboard / carrier** | **Vertical**, edge-on, in the rear-right **palm-rest column** (X ≈ 108–137, Y ≈ 96–197, Z 4–39) | SoM (Verdin i.MX 8M Plus 8 GB "WB", DEC-019/034/046) via B2B (TE 2309409-2, 5.2 mm); M.2 2280 NVMe M-key (DEC-033); LTE tiny-LGA (≤ ~30 mm, DEC-058); optionally the audio/DC rails; **all high-speed routes** | HDMI, USB3-A, USB2-OTG, RJ45 (on-module GbE PHY), SD, AUX — at rear/right of the palm-rest column (TBD wall, §11 M-3) |
| **Rear daughterboard** | **Vertical**, against rear wall (X ≈ 116–137, Y 3–197, Z 4–39) | **All low-frequency ports** (magnetic pogo UART REQ-UART-01, USB-C PD 65 W power-only, AUX line-out, power button, LEDs); power-manager MCU; STUSB4500 PD sink + BQ25713 charger; DC-DC 12/5/3.3 V split; battery contact blocks at its **front face** (X ≈ 110); fan driver | PD USB-C, pogo, AUX, LED/button window |
| **Keyboard PCB** | **Vertical**, edge-on near the front (X ≈ 3–10, plane across Y, Z 29–39) | STM32G0-class USB-HID deck MCU; membrane matrix scan; trackball sensor (ADNS-9800, SPI) via short pigtail flex to the ball well; optional backlight driver | USB-HID → carrier via FPC/cable (no external port) |

- **Interconnect ("FPC / J-connectors + cables", DEC-052):**
  - Carrier ↔ daughterboard: both vertical in the same rear band → **short horizontal links, gap < 50 mm**: signal FPC (0.5 mm, ≥ 26–30 p, 1.8/3.3 V, I2C/UART/PWM/GPIO/buttons/wake) + separate **power harness** (12/5/3.3 V + GND, 2.54 mm or Mini-Fit; rating per hardware/pcb.md §12.4, REQ-PWR-05 gate). High-speed never crosses (RISK-015).
  - Carrier ↔ keyboard PCB: deck USB2 + 3.3 V/GND FPC along the left spine or under-deck channel, bend ≥ 3 mm.
  - Carrier ↔ lid: **DSI FPC through the hinge** (4× lane + clk + I2C + BL PWM + power, ≤ 150 mm, dynamic bend loop ≥ 10 mm) + antenna coax (u.FL 1.13 mm × 3–4) + ALS flex. Lid sensors ride spare pins.

---

## 3. Z-stack (floor → mid-chassis spine → vertical boards → top deck → lid; thermal tower)

### 3.1 Stack budget (bottom → top)

| # | Layer | Z range (mm) | Notes |
|---|---|---|---|
| Z0 | Bottom plate 6061-T6 | 0–3 | machined floor, spine bosses, vent valleys |
| Z1 | Battery bays (front zone only) | 3–27 | pack 24 high, cells z ≈ 5–27 |
| Z2 | **Mid-chassis spine + drill rails** (front half, DEC-060) | 27–29 rails / 29–32 plate | rails machined **into the spine walls**; plate carries keyboard PCB base |
| Z3 | **Vertical boards** (rear zone + above spine) | 4–39 | daughterboard, carrier, keyboard PCB (edge-on; no horizontal plane) |
| Z4 | SoM + **thermal tower** (palm-rest column) | SoM 10–18, HS 18–25, fan 25–35 | rises over the upper edge of the vertical carrier; fan under the palm-rest grill |
| Z5 | Top deck (Al, 3 mm) | 39–42 | typing zone + palm-rest grill + bay mouths at front |
| Z6 | Keys (deck-relative) | 42–44 | keycap tops ~2 mm above deck |
| Z7 | Lid cavity (closed) | 42–50 (8 mm) | panel 4.75 + bezel + plastic lid-top + antenna gap |

Sum closed: **50 mm** exactly at the planning level; **≤ 55 mm** ceiling if the palm-rest deck needs to rise for the 7 mm heatsink/fan stack (DEC-043). **This is the first envelope revision where the battery is no longer the z-driver.**

### 3.2 Constraint check (the binding math)

- **Vertical board height:** top of carrier/daughterboard = `deck_underside = 42 − 3 = 39` → boards ≤ ~35 mm of component height above their bottom supports, comfortable for 1.6 mm board + connectors.
- **SoM B2B stacking (TBD measured):** TE 2309409-2 = board↔module 2.62 mm, module 6.0 mm → module top ≈ **8.6–9 mm over carrier** (hardware/pcb.md §4). Carrier top at Z4 end ≈ 18 → heatsink base at SoM top ≈ 18–20.
- **Heatsink fin height (DEC-017): 5–7 mm** → HS top ≈ 23–25 in the nominal 5 mm case, 25–27 for 7 mm.
- **Fan (DEC-028/038):** Sunon HA30101V4-1000U-A99, 30 × 30 × 10 → fan top ≈ **35 (5 mm HS) or 37 (7 mm HS)** — both < deck underside 39 ✅ at nominal closed 50. If measured SoM stack exceeds expectations, the palm-rest deck rises (≤ 55 ceiling) instead of enlarging the whole envelope.

### 3.3 Thermal construct (DEC-053, "heatsink+fan rise upward from the palm-rest zone")

- The SoM sits low in the right palm-rest column; its exposed upper edge carries the **5–7 mm finned heatsink** (mounted on Verdin S1–S4 bosses, hardware/pcb.md §9) and the **30×30×10 axial fan** on top — the tower rises vertically and is covered by the palm-rest deck grill (z 39–42).
- Intake: palm-rest grill (louvers, §7). Exhaust: **right-side wall louvers** (Y = 200, X ≈ 115–137) with a drain weep — keeps the rear daughterboard un-obstructed and keeps rain entry paths short.
- Design point: ~10–14 W continuous internal dissipation budget (feasibility §6) with fan ≤ 0.3–0.5 W; charge-while-use ≤ ~16 W → fan-at-max + charge taper on temp (RISK-016).
- CFM/noise/pressure and the exact fan axis (blow-down vs blow-across fins) are **TBD bench** (RISK-016).

### 3.4 Why the old plan-view split fails → resolved by the vertical board + z-column

| Zone (old flat layout) | Min depth (X, mm) |
|---|---|
| Battery bay (pack 105 + rail travel) | 108 |
| Carrier tray (horizontal) + keyboard | ≥ 40 |
| Daughterboard strip | ≥ 21 |
| **Sum vs interior 134** | **108+40+21 = 169 > 134** ✗ |

→ In-plane depth stacking is infeasible. The approved resolution (DEC-052): boards stand **upright in the z-column**, the battery bays occupy the front floor, and the palm-rest column at rear-right hosts the carrier + SoM + thermal tower. The keyboard PCB (front) and daughterboard (rear) are likewise vertical. This is the one arrangement satisfying REQ-ENC-01 at 200×140×50.

---

## 4. XY plan (base, plan view; front at top; X depth 0–140, Y width 0–200)

```
        Y →  0   10     92 102      180  197
     X ↓        ┌─────────┬────────┬───────────────┐
   3, front      │  e.g. KEYBOARD PCB (vertical, edge-on, front half)  │
   70            │  membrane 6×14 + trackball well  │  (front 50% depth)│
   ──────────────┼──────────┼────────┼──────────────┤
  108            │  BAY A   │  spine │  BAY B      │  right margin     │
   (bays full    │ 70×105   │rails   │ 70×105      │  (chase / spare)  │
   front width)  │          │GEAR060 │             │                   │
   ──────────────┴───┬──────┴───┬────┴──────────────┼───────────────────┤
  116                │ DAUGHTERBOARD (vertical, rear)                  │
  137, rear          └──────────┴──────────────────────────────────────┘
   (hinge barrels at Y ≈ 8–34 and 166–192 in the rear rim)
```

### 4.1 Battery bays (front, on the middle-chassis spine rails, DEC-060/A-038)

- **Bay A:** X 3–108, Y 10–92 (pack 70 wide centered). **Bay B:** X 3–108, Y 102–180.
- **Center spine:** Y 92–102 (10 mm) running X 3–137 — machined as part of the chassis; the **drill-style pack rails are machined into its side walls** (a 24-zone: two horizontal ribs per bay plus a key groove). Bays also gain upper guide ribs from the mid-chassis plate underside.
- **Insertion:** packs slide in from the front (X− direction along X+ travel), like a power-tool battery; positive stop at X ≈ 108 = **rear daughterboard contact face**; spring contacts co-mate in the last ~6 mm (staggered GND→ID/NTC→P+GND, §7 battery doc). Latch at the front bay mouth; polarity key = asymmetric spine groove + pack tab (T-rail), only one orientation possible.
- Bay Y clearances: pack 70 + ~4 taper = 74 per bay; total 148 + 10 spine + 2×2 walls/rubs ≈ 162 ≤ 194 ✅.

### 4.2 Palm-rest / SoM column (rear-right, DEC-053)

- Zone X ≈ 108–137, Y ≈ 96–197. Clear of battery bays (bays end X = 108) and of the rear daughterboard (which runs the full width).
- Vertical **carrier** board stands in this column (plane across Y, edge-on to the z-stack), z 4–39. **SoM** mounted above its lower edges (z 10–18); **M.2 2280 NVMe**, **LTE tiny-LGA**, deck-USB/LF cross-FPCs on the same column.
- **Thermal tower** rises through the palm-rest grill immediately above the SoM (X ≈ 115–135, Y ≈ 120–180). The keyboard stops at X ≈ 70 → no typing surface above the tower ✅.

### 4.3 Rear daughterboard (all low-frequency + power, DEC-052)

- X 116–137, z 4–39, spanning Y 3–197. Front-face battery contacts at X ≈ 108–110 for both bays; rear-face pushes the low-freq connectors + PD + AUX + pogo through the rear wall (X = 137/R = 140).
- Carrier high-speed ports exit near the rear-right (HDMI/USB3/RJ45/SD) — exact wall split is **TBD §11 M-3**.

### 4.4 Keyboard PCB (front, vertical, own board)

- Front half of depth (X 3–~70), plane across Y, standing z 29–39 on the mid-chassis spine plate. Membrane lies on the top deck over it (flat-flex into the vertical board top edge); trackball sensor at the well (X ≈ 62–70, Y ≈ 90–110) via a short horizontal pigtail flex. Deck MCU + HID output → carrier (USB2 FPC along the left spine).

---

## 5. Lid construction (DEC-055, A-033)

### 5.1 Materials / split

| Part | Material | Role |
|---|---|---|
| Lid aluminum rims/edges | black matte anodized 6061 | structure, hinge mounts, stiffness, grounding ring |
| Bezel (frame around display) | **plastic** (PC/ABS, TBD) | RF-transparent frame, holds panel, mounts to rims |
| Lid-top sheet (behind display) | **plastic sheet** (PC or PMMA, 1–1.5 mm) | closes the lid rear face; **RF-transparent** so antennas mount inside behind the panel |
| Panel | Raystar RFU800G-AYH-MNN | DSI, OD 115.74×184.93×4.75, active 107.64×172.22 |

### 5.2 Bezel math (DEC-043)

| Item | Value | Basis |
|---|---|---|
| Panel OD (landscape) | 184.93 (Y) × 115.74 (X) | candidates §4 |
| Bezel 3/3/3/10 (L/R/T/B) | outer bezel = 184.93+6 × 115.74+13 = **190.93 × 128.74** | DEC-043 |
| Aluminium rim wall | (200−190.93)/2 ≈ **4.5 mm L/R**; (140−128.74)/2 ≈ **5.6 mm T/B** | envelope − bezel |
| Exterior lid | 200 × 140 | matches base |

### 5.3 Antenna placement behind plastic (RISK-013/021 → superseded solution)

- Wi-Fi/BT (Verdin WB on-module, DEC-046) + LTE (tiny-LGA) RFC coax run from the palm-rest column → hinge → into the lid cavity; terminate on **antenna elements (PCB/adhesive flexible) mounted on the inside of the plastic lid-top sheet**, between the sheet and the panel.
- Keep-out: antenna elements **≥ 8 mm from the aluminum rims** (ground edge effect) and **≥ 4 mm from the panel back** (panel/rear-shield detuning) — gap provided by the antenna standoff/pocket in the plastic bezel. **TBD bench** RSSI/throughput vs position (RISK-021).
- No physical RF windows needed in the lid aluminum — the plastic cell is fully transparent (133 % better than the old deck-window plan; A-033).

### 5.4 Sensors (DEC-057, A-035)

- **Lid-open Hall sensor:** on the carrier/daughterboard at the rear-right (X ≈ 132, Y ≈ 150, z ≈ 36); **magnet in the lid rim** at the same Y ≈ 150; closed → field present, open → absent. Bond-gap ≤ 3 mm via trim.
- **Ambient-light sensor (ALS):** on a tiny flex/PBC mounted to the lid-top sheet inner face, **photodetector aimed away from the panel** (outward through the plastic top sheet) → measures ambient without panel glare; I2C + spare pin to the carrier via the lid FPC (auto-brightness, REQ-DISP-02). **TBD part + confirm bleed-gap position.**

---

## 6. Hinge zone and ThinkPad-class mounting (DEC-054)

### 6.1 Placement

- Two multi-point friction hinges, barrels integrated in the **rear wall / rim** at **Y ≈ 8–34 (left) and 166–192 (right)**, axis Y parallel to the rear edge, centered at Y ≈ 20 / 179; barrel zone z 42–50 (at/above top deck).
- Mounting lugs on the base: **reinforced bosses** machined into the rear wall + mid-chassis spine (≥ 3 fasteners per side, M2.5–M3, threaded inserts of ≥ 4 mm engagement); lid side: **through the aluminum rim** (≥ 4 screws per side into TBD threaded bosses) with a doubler plate.

### 6.2 Torque math (per side)

| Quantity | Value | Formula |
|---|---|---|
| Lid mass (panel + plastic bezel + lid-top + antennas) | ~0.25–0.30 kg | estimate, TBD weighed |
| Lid CoM arm (hinge → lid CoM, open) | ~60 mm | geometry |
| Required holding torque (2 hinges) | T_hold = m·g·d/(2) = 0.28·9.81·0.060/2 ≈ **0.082 N·m** | per hinge |
| **Spec friction torque** | **~0.6–1.0 N·m per side (≈6–12× hold)** | ThinkPad-class, DEC-054 |
| One-finger open on desk, no base lift (ceiling) | ≈ T_no_lift ≈ m_base·g·x_arm/(2) ≈ (1.7·9.81·0.05)/2 ≈ **0.42 N·m per side** | base pivot arm ~50 mm |
| Carry-by-opened-screen criterion | friction must hold lid @ any angle ≥ 3–5× hold | torque balance |

→ Design friction **~0.8 N·m/side nominal (user: ~1 N·m-class)**, ideally with **adjustable torque** (set-screw) so the bench can trim to just under the no-lift ceiling (0.4–0.8) for the desk-open use while ≥ carry hold. **TBD vendor + factory torque curve, life ≥ 15k cycles, ≤ 20 % tolerance** (assembly doc §4).

- FPC/coax through the hinge axis (central channel) with a **≥ 10 mm dynamic radius loop** in the lid channel; hinge removed before flex service (assembly §3).

---

## 7. Rain / water routing (DEC-056, A-034)

### 7.1 Keyboard drainage (T480-style weep)

- Water enters through key wells → lands on a molded drainage boss/trough under the membrane → **gutters molded into the top deck underside**, sloping aft/bottom (≥ 2°), collected at a **side/bottom weep rail** → exits at the **bottom edge weep slots** (front corners + under-bay), clear of the battery bays and rear ports.
- Drain sizing: T480-class ≈ 2×(4–8) mm slots **per key row**, ≥ 12 slots total, with a one-way flap/labyrinth so air passes but water does not re-enter; openings ≥ 0.5 mm to stop amphids; **TBD** flow test (drip + cup test, assembly §5).

### 7.2 Sealed vents with drain

- Fan intake (palm-rest grill) and exhaust (side louvers): **labyrinth louvers + fine mesh + replaceable filter pocket**; each vent plenum has a **low-point drain weep to the exterior** so any ingressed water exits before reaching the fan/duct.
- Keep net slot area ≥ 0.5–0.7× fan area (30×30 mm ≈ 900 mm² → ≥ ~500–600 mm² effective).

### 7.3 Closure and port sealing

| Opening | Seal | Notes |
|---|---|---|
| Lid-to-deck closure line | continuous closed-cell foam gasket (1–1.5 mm crushed) | also acoustic/dust |
| Hinge FPC/coax channel | foam gasket ring + soft bend-cap | keep dynamic radius ≥ 10 mm |
| Rear connector cut-outs | conductive EMI gasket + IP-rated bead per connector | spatter/IP4X |
| Pogo port | silicone/foam plug or magnetic cap-washer (default signal+GND, REQ-UART-01) | do not energize until electrical review |
| Battery bay mouths | soft lip seal + close-cover over front bezel | latched; no water path to contacts |
| Overall | **rainproof for multiple hours; IP4X splash + dust (TBD bench**), not submersible | DEC-056 |

---

## 8. Clearances, cables, connectors, bend radii

| Routing | Spec / allowance | Note |
|---|---|---|
| DSI FPC (carrier → lid, through hinge) | 0.3–0.5 mm pitch FFC; ≥ 4-lane + clk; **bend ≥ 5–6× thickness; dynamic hinge loop ≥ 10 mm**; ≤ 150 mm total | custom FFC + Raystar pinout, **TBD** (RISK-024) |
| Antenna coax u.FL 1.13 mm × 3–4 (carrier → lid) | bend ≥ 3 mm; keep ≥ 10 mm from fan/heatsink metal; ≥ 8 mm from lid rims | Wi-Fi ×2, LTE ×1–2 |
| Carrier ↔ daughterboard (both vertical, same band) | signal FPC 0.5 mm ≤ 80 mm die; power harness separate; high-speed never crosses (RISK-015) | short horizontal links |
| Carrier ↔ keyboard PCB (USB2) | 0.5 mm FFC, bend ≥ 3 mm, along left spine/under-deck channel | — |
| Battery interconnect (pack → DB front face) | spring contacts, ≥ 3 mm terminal separation; 14.4 V, ≤ 5 A rated; staged-length pins (GND first) | DEC-004, battery doc |
| Fan/air duct | duct walls ≥ 2 mm; no pinch < 1.5 mm; grill-to-fan gap ≥ 3 mm | noise/dust/drain |
| Board-to-chassis keep-outs | ≥ 1.0 mm machined faces; ≥ 0.5 mm pads/gaskets | RISK-002 margin |
| Verticals to spine plate | board bottom ≥ 1 mm above plate (isolate + flex); M2.5 standoffs | — |

---

## 9. Mass estimate and center of mass

### 9.1 Mass table (planning; reconciled with DEC-033/038/040/041/044/058)

| Item | Mass (g) | X centroid (mm) |
|---|---|---|
| Battery cells 8×71 (P50B) in bay A+B | 568 | 55 |
| Pack HW ×2 (BMS, shells, rails tabs, latches) | 70 | 55 |
| Keyboard + membrane + trackball + deck MCU | 130–160 | 35 |
| Carrier PCB + SoM (Verdin 8GB WB) + B2B | 95–115 | 118 |
| Heatsink 5–7 mm + Sunon fan | 50–70 | 116 |
| NVMe 2280 (user's) | 50–70 | 118 |
| LTE tiny-LGA + SIM + antennas (in lid) | 20–35 | 110 |
| Wi-Fi (on-SoM) + lid antennas/flex | 10–15 | 110 |
| Rear daughterboard + PD + pogo + power | 110–130 | 126 |
| Display (Raystar + BL) | 110–130 | 70 |
| Plastic bezel + lid-top sheet | 35–55 | 70 |
| Al rims (lid) + hinge assemblies ×2 | 60–90 | 130 |
| Chassis (base + mid-spine + deck) | 400–550 | 70 |
| Screws / cables / gaskets / misc | 110–150 | 70 |
| **Total** | **≈ 1,830 – 2,120** | — |

Mid-range ≈ **1,970 g** → the mass budget band **1.6–2.1 kg** is met (feasibility §7; aluminum shell + plastic lid keeps it bounded).

### 9.2 CoM (X = depth, 0 front … 140 rear)

- Closed, both packs: Σ(m·x) ≈ **104,500 g·mm ⇒ X_CoM ≈ 53 %** (≈ 74 mm) — in the 45–55 % neutral band (front-loaded battery offset by rear I/O/thermal) ✅.
- **Lid open** (~110°): lid + rear hardware centroid shifts + ~45 mm → ≈ **56 %, still inside footprint** (rear feet ≈ 126–133) ✅.
- **Both packs removed + lid open** (hot-swap event): Σm ≈ 1,330 g, X_CoM ≈ **85 mm = 61 %** — no tip-back (needs CoM < ~126 mm; margin ~40 mm) ✅.
- Y-CoM symmetric ≈ 100 (centered); Z-CoM ≈ 26–30 mm (low, favorable).

---

## 10. Keep-out / bounding boxes (source-verified core dims)

| Component | Keep-out box (X×Y×Z mm) | Source |
|---|---|---|
| Battery pack ×2 | 105 × 70 × 24 each (bay 108 × 74 × 28) | P50B Ø21.55×70.15, candidates §1 |
| Verdin i.MX8MP WB | 69.6 × 35.0 × 6.0 + B2B 5.2 (module top ≈ 8.6–9 over carrier, **TBD measured**) | candidates §2, hardware/pcb.md §4 |
| Carrier (vertical) | ~20 × 100 × 35 (stand in palm-rest column) | layout §4.2 |
| Rear daughterboard | 21 × 194 × 35 | layout §4.3 |
| Keyboard PCB (vertical) | 8 × ~194 × 10 (edge-on above spine) | layout §4.4 |
| M.2 2280 NVMe (M-key) | 22 × 80 × ≤ 3.7 (+standoff) | Wikipedia M.2, 2026-08-30 |
| LTE tiny-LGA | ≤ ~30 × ~30 × ~2.9 (EC200U/BG95/SIM7000-class, DEC-058) | **TBD part** |
| Sunon fan | 30 × 30 × 10 | candidates §7 |
| Thermal stack over SoM | ~38 × 40 × 27 (SoM+HS+fan) | §3.3 |
| Raystar panel | 115.74 × 184.93 × 4.75 (active 107.64 × 172.22) | candidates §4 |
| Antenna elements (lid) | 2× (30–40 × 6 × 1) + 1× LTE | §5.3 |
| Hinge assemblies | ~26 × 45 × 12 each | §6 |

---

## 11. Findings summary

1. Envelope 200×140×50 with 3/3/3/10 bezels is satisfied **only** by the vertical-board stack (DEC-052): boards edge-on in the z-column, no horizontal carrier, palm-rest column carries SoM + thermal tower (DEC-053).
2. Raystar landscape fit proven (§1.2): 190.93 ≤ 194 wide, 128.74 ≤ 134 deep.
3. Battery bay math closes at ≤ 194 width with a mid-chassis drill-rail spine (DEC-060); front insertion, rear contacts, hot-swap retained.
4. Z-stack closes at **50 mm nominal / 55 mm ceiling**; heatsink 5–7 mm and the SoM B2B stack (~9 mm) are the gating values (**TBD measured**).
5. Lid = plastic bezel + plastic lid-top + aluminum rims (DEC-055); antennas mount behind the plastic panel-cavity (RISK-013/021 superseded). Hall + ALS placed per DEC-057.
6. ThinkPad-class hinges ~1 N·m-class/side, with a bench-trim to the no-lift ceiling (DEC-054).
7. Rainproof hours with keyboard weep drainage + sealed, drained vents + gasketed closure (DEC-056).
8. Mass ≈ 1.8–2.1 kg; CoM 53 % closed → ≤ 61 % with packs out + lid open — **no tip-back**.

---

## 12. Open mechanical decisions needing user input

| # | Decision | Options | Blocks |
|---|---|---|---|
| M-1 | Carrier plane orientation / SoM face + thermal-tower axis final (vertical-board geometry) | tower over SoM upper edge (baseline) vs tower beside column; fan blow-down vs across fins | carrier layout, deck grill |
| M-2 | Heatsink 5 vs 6 vs 7 mm (depends on measured SoM B2B stack) | 5 default → 7 needs ≤ 55 ceiling (DEC-043) | thermal, envelope headroom |
| M-3 | High-speed port exit wall for carrier (HDMI/USB3/RJ45/SD) | rear segment (Y 130–197) vs right side (Y=200) | rear-wall CNC, connector order |
| M-4 | Rear daughterboard low-freq port order + pogo exact magnet/contact spec | TBD, Mill-Max 0964-class | DB layout |
| M-5 | Hinge vendor + torque trim (0.6 vs 0.8 vs 1.0 N·m/side) | bench-trim, adjustable set-screw | lid build |
| M-6 | Deck rise over palm rest (nominal 50 vs ≤55 w/7 mm HS + fan) | closed-height budget | CNC, thermal |
| M-7 | Keyboard PCB tilt/angle + trackball sensor pigtail arrangement | flat membrane vs slight pitch; pigtail well | keyboard assembly |
| M-8 | Battery drill-rail profile + spine width (8 vs 12 mm) + contact vendor | Mill-Max / CNC leaf / pogo; drop-test latch | bay + assembly |
| M-9 | ALS + Hall exact parts + placement (prism/bleed-gap) | photodiode vs ambient sensor IC | lid flex, Phase 7 |

All **TBD/unverified** items must be closed by Phase-8 measurement (panel OD, SoM stack, antenna RSSI, hinge torque, seal test, mass).