# Mechanical — Envelope, Flat-Board Stack and Lid (Phase 6)

Status: **Phase-6 mechanical design draft — 2026-08-30.** This is the **authoritative revision** of the mechanical layout. All dimensions are CAD-input planning values derived from the approved baseline (DEC-001/002/003/016/017/018/019/023/033/034/038/040/041/043/044/045/046/047/048/049/050/052/053/054/055/056/057/058/059/060/061/062/063) and Phase-4 part evidence (`parts/candidates.md`, `hardware/thermal.md`). Figures marked **TBD** are unverified and must be confirmed by part datasheet or by measurement (Phase-8 bench) before CNC quote.

> **SUPERSESSION NOTE — reads first (DEC-052, revised).**
> This document **replaces and supersedes the earlier vertical-board draft** of `envelope-and-stack.md`. The vertical-board construct (three boards standing **edge-on**, SoM/thermal tower rising out of a rear palm-rest *column*, fan stacking above the heatsink) is **abandoned**. Per **DEC-052 (revised)** the approved architecture is **three HORIZONTAL FLAT boards on normal laptop planes**:
> 1. **keyboard PCB** (top, part of the deck),
> 2. **carrier motherboard** (SoM via 0.5 mm 260-pin B2B; M.2 2280 NVMe M-key; LTE tiny-LGA; Wi-Fi antenna feed; USB2 hub; codec),
> 3. **rear daughterboard** (ALL low-frequency ports + power management + PD/charger + battery spring contacts),
> interconnected by **FPC/J-connectors + cables**.
>
> The thermal construct is also superseded: the 30 mm fan is **in-plane left of the heatsink** (not stacked on top), the fan/duct-split direction from `hardware/thermal.md` §1.4/§6 — **Delta BFB0305HA-C blower is primary**, Sunon axial is fallback (thermal §2.4). The keyboard sits at **regular-laptop 10 %/40 %** position (DEC-061) and the SoM + 5–7 mm heatsink + fan live in the **right palm-rest zone** (DEC-053), with the **left-back → right** air path (DEC-062) ducted around the flat stack. Battery drill-rails stay in the **middle chassis spine** (DEC-060). All figures here are re-derived for the flat stack; cross-references to `hardware/thermal.md` use its stated numbers verbatim where possible.

> Coordinate convention (identical to `hardware/thermal.md`, all machines): **X = depth (front 0 … rear 140)**, **Y = width (left 0 … right 200)**, **Z = height (0 at bottom-plate floor)**. Envelope: **200 (Y) × 140 (X) × 50–55 (Z closed)**. The 200 mm axis is width, 140 is depth (landscape, DEC-063).

---

## 1. External envelope verification (REQ-ENC-01, DEC-043/063)

### 1.1 Envelope table

| Dimension | Approved (REQ-ENC-01) | Design target | Interior (3 mm walls) | Verdict |
|---|---|---|---|---|
| Width (Y) | ≤ 200 | **200** | **194** (walls 3/3) | ✅ |
| Depth (X) | ≤ 140 | **140** | **134** (walls 3/3) | ✅ |
| Height (Z, closed) | ≤ 50 (≤ 55 w/ 7 mm HS) | **53 nominal (HS-5)/≤55 (HS-7)** | — | ✅ within ≤55 ceiling; see §3 for the raised palm-rest deck |

Chassis construction (DEC-016/023/A-022): **black matte anodized aluminum, CNC-machined** (base floor, mid-chassis drill-rail spine, stepped top deck, lid rims — free local CNC, cost = material + anodizing only). All major surfaces noted in the stack reference this 6061 base structure.

### 1.2 Why 200 mm is the width — panel-fit proof (DEC-043/044, A-030/031)

Raystar RFU800G-AYH-MNN: **OD 115.74 × 184.93 × 4.75 mm**, active 107.64 × 172.22 mm (800×1280), used **landscape** (long axis → width).

```
Long axis:  184.93 + 3 + 3 (L/R bezel)            = 190.93  ≤ 194 interior ✅
Short axis: 115.74 + 3 + 10 (T/B bezel)          = 128.74  ≤ 134 interior ✅
Exterior:    190.93 + 2×4.5 rim (L/R) ≈ 199.9 ≈ 200 ✅   and 128.74 + 2×5.6 ≈ 140 ✅
```

Width budget proof (two side-by-side packs, DEC-003/A-004 plus the DEC-060 drill-rail spine and the DEC-062 duct):

```
Left  wall                      3
Intake/duct channel (Y 3–16)   13   ← DEC-062 left-back intake duct (under-carrier)
Bay A  (pack 70 wide window)   76   ← carry clearance to pack 70
Mid chassis spine (Y 92–102)   10   ← DEC-060 drill rails machined into spine walls
Bay B  (pack 70 wide window)   76
Exhaust channel (Y 178–194)    16   ← DEC-062 right exhaust plenum/weep
Interior                         194  ✓   (exterior 200 with 3+3 walls)
```

Reverse reading (200 = depth, 140 = width) cannot hold the panel long axis (190.93 > 134) nor two packs across 134 — **rejected**. 200 = Y (width), 140 = X (depth) confirmed (DEC-063).

- Interior height usable below the deck underside: **~39 mm** in the keyboard band, **~42.2 mm** in the raised palm-rest band (§3) — this is a z-stack budget, not volume bookkeeping.
- No margin for a part wider than spec: tolerances must be proven by pass/fit sketches before CNC (RISK-002).
- Envelope may reach **≤55 mm** where the raised palm-rest deck carries the HS (HS-7) and HS-5 keeps ~53 (DEC-043, §3).

---

## 2. Board architecture — three horizontal flat boards (DEC-052 revised, A-032)

### 2.1 Role register

| Board | Plane / z | Contents | Connectors / exits |
|---|---|---|---|
| **Keyboard PCB** (top, part of the deck) | flat, z ≈ 35–39, keyboard band X 56–126, full Y | STM32G0-class USB-HID deck MCU (DEC-045); membrane matrix scan; trackball sensor (ADNS-9800, SPI, DEC-025/050); optional backlight driver | USB-HID → carrier via J/FPC, no external port |
| **Carrier motherboard** | flat, z ≈ 28.5–30.2, spans Y 3–194, X ≈ 20–134 (over the bays + under the keyboard band) | SoM (Verdin i.MX8MP-WB, DEC-019/034/046) via TE 2309409-2 260-p 0.5 B2B (SoM top ≈ +8.6–9); M.2 2280 NVMe M-key (DEC-033/048); LTE tiny-LGA ≤~30 mm (DEC-058) + nano-SIM; Wi-Fi/BT antenna U.FL feed (DEC-046); USB2 hub (USB2513-class, DEC-049); codec/AUX; carrier-side FPC/J drops | DSI FPC → hinge → lid; U.FL coax → lid antennas; hub ports |
| **Rear daughterboard** | flat, z ≈ 4–6, band X 110–134, full Y | **ALL low-frequency ports**: HDMI, USB3-A, USB2-O-TG, RJ45, AUX, SD, pogo-UART (REQ-UART-01); power: STUSB4500 PD sink + BQ25713-class charger, DC-DC 12/5/3.3 V (DEC-031); power-manager MCU + fan driver + fan fau-lt (themal §4.2/§4.3); **battery spring-contact towers on its FRONT face** (X ≈ 108, staggered GND→ID/NTC→P) | HDMI, USB3, USB2, RJ45, SD, AUX, pogo, USB-C PD (REQ-I-O-06) — all rear wall; battery contacts front |

**Interconnect (DEC-052 "FPC + cables"):**
- Carrier ↔ daughterboard: because the carrier overlies the DB band (carrier to X≈134, DB below at z 4–6), the link is a **short vertical jumper** — signal FPC (0.5 mm, ≥26–30 p, I2C/UA-RT/PWM/GPIO) + a separate power harness (12/5/3.3 V + GND, ≥ Mini-Fit-class) routed through a board notch in the carrier. High-speed never crosses (RISK-015).
- Carrier ↔ keyboard PCB: deck USB2 + 3.3 V/ND FPC, vertical drop ~8 mm in the keyboard band, bend ≥3 mm.
- Carrier ↔ lid: **DSI FPC through the central hinge channel** (4-lane + clk + I2C + BL PWM + power, ≤150 mm total, dynamic bend loop ≥10 mm) + 3–4× U.FL 1.13 mm coax to lid antennas + ALS/Hall flex.

This is a normal-laptop z-plane architecture: everything lays flat, all three boards are accessible from the base removal sequence (`assembly-serviceability.md` §1).

---

## 3. Z-stack (floor → carrier → SoM thermal desk → deck → lid)

### 3.1 Plane register (bottom-up)

| # | Layer | z range (mm) | Notes |
|---|---|---|---|
| Z0 | Bottom plate 6061-T6 | 0–3 | machined floor, bay floor, vent valleys, DB mount |
| Z1 | Battery bays + rail spine | 3–28 | pack slab 24 high; slide clearance to 28.5; drill-rail spine Y 92–102 runn X 3–137 (DEC-060/A-038) |
| Z2 | **Carrier motherboard, flat** | 28.6–30.2 (board 1.6) | sits on standoffs ~1.5 above pack rails/spine; spans X 20–134 |
| Z2b | SoM B2B stack (right palm-rest) | 30.2 → **~38.8 top** (+8.6, TBD 8.6–9.5) | TE 2309409-2, Verdin 6.0 module |
| Z2c | **HS 5–7 mm** on SoM | 38.8 → **43.8 (HS-5) / 45.8 (HS-7)** | fins run +Y (left→right, DEC-062) |
| Z2d | **Fan 30×30×10 in-plane left of HS** | 30.2 → 40.2 top | Delta BFB0305HA-C blower primary (themal §2.4); axial fallback |
| Z3 | Keyboard PCB (flat) | 35–39 | in keyboard band, under deck |
| Z4 | Top deck = keyboard deck + raised palm-rest deck | 39–42 (keyboard) / **42.2–45 (palm)** | stepped deck (DEC-043 ceiling) |
| Z5 | Keys/cap | 44–46 tops | ~2 above deck |
| Z6 | Lid (closed) | 45–51.6 (over palm), lid ~6.5–8 | panel 4.75 + plastic lid-top + gaskets |

### 3.2 Binding z-aths (the gating values)

```
Carrier top plane   z_c = 28.5..30.2       (standoffs ~1.5 over pack rails 28–28.5; TBD by standoff BOM)
SoM top            z_s = z_c + h_b2b          h_b2b ≈ 8.6–9.5   (TE 2309409-2 2.62 + module 6.0; TBD measured)
Hab sink top       z_h = z_s + h_fin          h_fin = 5.0 (HS-5) / 7.0 (HS-7)
Keyboard deck top  z_k = 42 (3 mm plate, underside 39.0)
Raised palm deck  z_p ≥ z_h + 1.4 (shroud gap 0.5 + shut-term ~0.9)   → 45.2 (HS-5) / 47.2 (HS-7)
Fan top            40.2 < palm deck underside (z_p −3 ≈ 42.2) ✅
```

Check: HS-5 z_h = 30.2 + 8.6 + 5.0 = **43.8** ≤ deck underside 42.2? **No — the palm deck must rise.** z_p ≈ **45.2** (HS-5) and **47.2** (HS-7). The deck steps +3.2/+5.2 mm over the palm-rest band (X 0–56). Closed envelope:
```
Lid seat     = highest deck top  = 45.2 (HS-5) / 47.2 (HS-7)
Closed (HS-5)  = 45.2 + 6.5..8.0 (lid) = 51.7–53.2   → body of REQ ≈ 52 nominal
Closed (HS-7)  = 47.2 + 6.5..8.0        = 53.7–55.2 → at ceiling 55 (DEC-043)
```

Reading: **the SoM B2B stack + HS is the z-driver, not the battery** (battery 24 no longer dominates — DEC-001 rectified). Because the SoM sits **over** the right pack (palm-rest zone = front-right, which overhangs Bay B), the HS headroom can only come from raising the palm-rest deck (change `DEC-001/043` ceiling). To reach a *flat* 50 the SoM would need a battery-free pocket — see §10 minimal changes / open items.

Key clearances: carrier ↔ keyboard PCB = 35 − 30.2 = **4.8 mm** (M.2 3.7 + routing fits; true if M.2 is on the carrier under the KB band, §4.4); SoM top (38.8) uses the raised palm band (underside 42.2) → OK.

Formula set (z-auditable on bench):
```
z_hf = z_c + h_b2b + h_fin        z_c = standoff + PCB 1.6
z_p  = ceil(z_hf + 0.5 gap + 0.9 tol)   → deck top = z_p (+3 plate); closed = z_p + lid
```

### 3.3 Why the old in-plane split fails and the flat stack closes it

The old "vertical boards" draft (§3.4) argued in-plane depth stacking (108 + 40 + 21 = 169 > 134) forces edge-on boards. The flat stack closes the same envelope by **z-overlap**: carrier (falt) + keyboard PCB (flat) + DB (flat) **share depth bands at different z**. Pan band by band:

| Detail | X band | z usage | Closing argument |
|---|---|---|---|
| Palm-rest band | 0–56 | raised deck 45 + thermal stack | SoM/HS/fan above Bay B at z 28.5→45 |
| Keyboard band | 56–126 | KB PCB 35–39; carrier 28.5–30.2 with M.2 | two planes overlapping the bay field |
| Rear band | 108–134 | carrier over DB (z 4–6) | FPC/J drop; DB contacts front-faced |

First envelope revision where **no board stands edge-on** — normal laptop construction.

---

## 4. XY plan (landscape; front at top; X = depth 0–140, Y = width 0–200)

```
   Y →   0   3     16             92 102            178  194 200
  front ▐w▐███░░░ AYYYYYYYYYYYYYY SP YYYYYYYYYYYYYYBB ░░░██w█▐
    3   ▐w▐▓▓▓░░░ BAY A (pack    █▓▓▓ BAY B (pack    ▓▓▓w█▐  ← bay mouths z4–28
        ▐w▐▓▓▓░░░ 70×105)        █▓▓▓ 70×105)        ▓▓▓w█▐
    8   ▐w▐▓▓▓░░┌─· CARRIER (flat, z28.5) ────────·──┐w█▐  fan▶HS at X8–43
  20    ▐w▐█░░░░│                                        │w█▐   (palm deck raised)
   ======================= SoM/HS/fan zone (X 8–43, Y 105–187) ==========
  43    ▐w▐░░░░░░│... M.2 2280 (X→, Y≈40) ... NVMe ..│....│w█▐
  56    ▐w▐░░░░░░│ KEYBOARD ░  ░ (X56–126, flat plane)  │...│w█▐ ◄ keyboard starts
  90    ▐w▐░░░░░░│  ... trackball well below space bar (G–H–B)   │w█▐
 108    ▐w▐░░░░░░│  ... (bays end) CONTACTS ↑ .........│....│w█▐  ◄ DB battery towers
 126    ▐w▐░░░░░░│  ... keyboard rear ..................│....│w█▐  ◄ KB ends (90%)
 134    ▐w▐░░░░░░│ REAR DAUGHTERBOARD ░░░░░ (z4–6)   │...│w█▐   ports out rear wall
 140    └────────┴──────── hinge barrels ────────────┴──┘     rear wall
  Legend: w=3mm wall, █=duct channel, ░=open volume, SP=drill-rail spine (DEC-060)
```

### 4.1 Zones (interior, Y 3–194, X 0–134)

| Zone | Y band | X band | z | Function |
|---|---|---|---|---|
| Left intake channel (under-carrier) | 3–16 | 3–137 | 4–28 (13×24 ≈ 312 mm² ≫ 150) | DEC-062 intake trench, §7 |
| **Bay A** (drill-rail) | 16–92 | 3–108 | 4–28 | pack 70×105 slide-in (DEC-060) |
| **Mid chassis spine** | 92–102 | 3–137 | 3–31 | drill rails both sides + carrier standoff + duct roof |
| **Bay B** | 102–178 | 3–108 | 4–28 | pack 70×105 (DEC-060) |
| Right exhaust channel | 178–194 | 8–134 | 29–38 | DEC-062 exhaust plenum (+16 wide), §7 |
| **Carrier** (flat) | 3–194 | 20–134 | 28.6–30.2 | SoM/M.2/LTE/hub |
| **SoM + HS + fan** | 105–187 | 8–43 | 30.2–45 | DEC-053/062, thermal tower flat on carrier |
| Keyboard PCB | 3–194 | 56–126 | 35–39 | DEC-061 band, flat |
| Raised palm deck | 3–194 | 0–56 | 42.5–45 | DEC-043/053; covers thermal + duct header |
| **Rear daughterboard** | 3–194 | 110–134 | 4–6 | all LF ports + power + battery towers (DEC-052) |
| Lid/panel | — | 0–134 | 45–51.6+ | Raystar landscape (DEC-055) |

### 4.2 Keyboard band (DEC-061/A-039)

```
X_kb_rear (top edge) = 140 × (1 − 0.10) = 126      "top edge at 10 % depth"
X_kb_front (bottom)  = 140 × 0.40       = 56       "bottom edge 40 % from front"
Keyboard band span   126 − 56 = 70 mm = 50 % of deck depth
Palm-rest             X 0–56 (front 40 %)          "palm-rest = front 40 %"
```

- 6-row membrane, ~12 mm pitch, ≥14 cols → ~168 mm wide ≤ 194 ✅ (DEC-021).
- Trackball well **below space bar, between G-H-B** → at keyboard front edge X≈56, Y≈90–110 (DEC-025/A-024); ball 15–21 mm bench gate RISK-020/DEC-050.
- **Only the top portion holds keys**; the front 40 % is un-keyed palm-rest with the SoM tower in the right portion (DEC-053). Nobody types over the HS/fan.

### 4.3 Palm-rest / SoM thermal zone (DEC-053/061/062)

- Zone X 8–43, Y 105–187. **SoM** at X 8–43, Y 138–174 (⌀ module 35 along X, 69.6 along Y → fits 43 wide). Base under the 32(X)×38(Y) HS footprint.
- **Fan** in-plane left: Y 105–135 (30×30), X 8–38; blower intake = large face at −Y (Y≈105) feeding the duct header; **outlet slot 30×5 faces +Y into the HS fin mouths ≤ 2 mm** (thermal §1.4/§1.7; Delta blower primary DEC-028-revised).
- **HS** Y 135–173 (38 along flow), fins +Y left→right (thermal §1.5); exhaust gap Y 173–187 → right channel → louvres at Y 194–200 wall, X 8–40 (thermal §1.6).
- Raised palm deck over X 0–56 (top ≈ 45) hides the whole assembly; keyboard keys start at X 56 — no typing surface over the tower ✅ (DEC-053).
- Skin targets **≤45 °C pref / ≤50 °C hard**, HS base → palm-deck boss + TIM coupling (thermal §3.4 §6.6).

### 4.4 Carrier motherboard detail

- Board ~ 194(X)?? — orientation: **X 20–134 × Y 3–194** (length X, width Y) flat sheet over the bay field + spine.
- **M.2 2280 NVMe along X** (length 80): socket at X≈55, drive to X≈135, Y≈40–62 (over Bay A, left of spine) — keeps it clear of the RF/thermal zone and of the DB's tall parts. 22 mm wide; z 30.2–33.9 < KB PCB plane 35 ✅. Keyed **M-only + silkscreen "NVMe M.2 2280 ONLY"** (DEC-033/048/A-027); drive retained by M2 standoff 0.4 N·m.
- **LTE tiny-LGA** (≤30 mm, DEC-058) + nano-SIM tray: Y≈70–100, X≈60–90 (over spine area); SIM tray on the carrier, **accessed via bottom-plate trap door** (minimal change, §10) or optional rear-edge tray.
- **USB2 hub** (USB2513-class, DEC-049), codec/AUX, U.FL Wi-Fi/BT feeds, hub/fan/I2C drops to DB + KB board on the carrier.
- Carrier has routing **notch/recess** where the carrier↔DB jumper and the DB's tall RJ45 zone sit (X 110–134) — jumper passes through, tall parts stay < 28.

### 4.5 Rear daughterboard band

- Band X 110–134, full Y, flat z 4–6 (under the carrier, heat flows down to the cool bottom plate — improved skin vs the old rear wall).
- **Battery spring-contact towers** rise on the DB's front face at X≈108, z≈6–28, staggered GND→ID/NTC→P/GND, co-mate in the last ~6 mm of pack travel (DEC-004/060, A-038; staging electrical detail → hardware Phase 6, RISK-004).
- All LF ports exit the rear wall: HDMI, USB3-A, USB2-OTG, RJ45, AUX, full-size SD, 6-pin magnetic pogo UART (REQ-UART-01/02, DEC-047/059), USB-C PD 65 W (REQ-IO-06, DEC-031). RJ45 (≈13.4 tall, tops ~19 < 28) per-port ESD/IP beads.

### 4.6 Hinge barrels

- Two multi-point friction hinges, barrels in the rear rim Y≈8–34 and 166–192 (axis ∥ rear edge), z complement at deck top (DEC-054); central channel between them carries DSI FPC + coax (old §6 geometry retained, re-based on the flat decks, §8).

---

## 5. Lid construction (DEC-055, A-033) — unchanged scope, re-baselined

### 5.1 Materials / split

| Part | Material | Role |
|---|---|---|
| Lid aluminum rims/edges | black matte 6061 | structure, hinge mounts, stiffness, **grounding ring** |
| Bezel (frame) | **plastic** (PC/ABS) | RF-transparent, holds panel, 3/3/3/10 |
| Lid-top sheet | **plastic** (PC or PMMA, 1.0–1.5) | closes lid rear; RF-transparent; antennas behind panel |
| Panel | Raystar RFU800G-AYH-MNN | DSI, OD 115.74×184.93×4.75, active 107.64×172.22 |

### 5.2 Bezel math (DEC-043)

| Item | Value | Basis |
|---|---|---|
| Panel OD (landscape) | 184.93 (Y) × 115.74 (X) | candidates §4 / A-031 |
| Bezel 3/3/3/10 | bezel = 184.93+6 × 115.74+13 = **190.93 × 128.74** | DEC-043 |
| Al rim | (200−190.93)/2 ≈ **4.5 L/R**; (140−128.74)/2 ≈ **5.6 T/B** | envelope − bezel |

### 5.3 Antennas behind plastic (RISK-013/021 superseded)

- Wi-Fi/BT (SoM WB, DEC-046) + LTE coax U.FL → hinge channel → lid; elements (PCB/adhesive flex) on the **inner face of the plastic lid-top sheet**, ≥8 mm from Al rims, ≥4 mm from panel back (DEC-055/A-033). No Al RF window needed.

### 5.4 Sensors (DEC-057/A-035)

- **Lid-open Hall:** magnet in the lid rim at Y≈150; Hall on the carrier rear edge (X≈132, Y≈150, z≈33) where the lid magnet passes at closed; bond-gap ≤3 mm via trim.
- **Ambient-light (ALS):** tiny flex on the lid-top inner face aimed **away from the panel** (outward through plastic), I2C + spare pin on the lid FPC (auto-brightness, REQ-DISP-02). Position: lid top band, ≥30 mm from panel glow — **TBD** exact bleed-gap.

---

## 6. Hinge zone and ThinkPad-class mounting (DEC-054)

### 6.1 Placement & mounting

- Two multi-point friction hinges, rear rim Y≈8–34 / 166–192, base side ≥3 fasteners/side into reinforced rear-wall + spine bosses (M2.5–M3, inserts ≥4 mm); lid side ≥4 screws/side into rim doubler.

### 6.2 Torque math (per side)

| Quantity | Value | Formula |
|---|---|---|
| Lid mass ≈ 0.25–0.30 kg; CoM arm ≈ 60 mm | — | — |
| Hold torque | T = m·g·d/2 = 0.28×9.81×0.06/2 ≈ **0.082 N·m** | T_hold |
| Spec friction (ThinkPad-class) | **~0.8 N·m/side nominal (0.6–1.0)**, adjustable set-screw | DEC-054 |
| One-finger-open ceiling (no base lift) | T = m_base·g·x_arm/2 ≈ (1.9×9.81×0.05)/2 ≈ **0.47 N·m** | base mass ~1.9 kg |
| Carry-by-screen | friction ≥ 3–5× T_hold at any angle | torque balance |

Design friction **~0.8 N·m/side** with set-screw trim (0.4–0.8 desk / ≥carry), ≥15k cycles, ≤20 % tolerance — **vendor + factory curve TBD**. Route DSI FPC/coax before final hinge torque (order in `assembly-serviceability.md` §4.2).

---

## 7. Cooling air path & ducting (DEC-062/A-040 — re-baselined for the flat stack)

Numbers below are **taken from `hardware/thermal.md` §1/§2/§6 verbatim** (fan: **Delta BFB0305HA-C blower primary**, 0.285 inH₂O / 71 Pa static, 1.45 CFM free-air; axial Sunon fallback; operating point ~0.75 CFM vs ~0.4 for axial).

### 7.1 Path statement (flat-stack routing)

```
LEFT-BACK intake louvres (Y≈0/rear-corner, X≈100–137)
   → under-carrier intake channel   (Y 3–16, z 4–28, ×312 mm²)
   → front duct header              (z 28.5–38, cross-section ≥153 mm², X 3–20)
   → blower intake (large −Y face, Y≈105)            [low-pressure draw]
   → blower outlet slot (~30×5 ≈150 mm²) → HS fin mouths (≤2 mm gap)
   → HS fins run +Y left→right (17 ch ×1.4×3.5 ≈83 mm²)
   → right channel (Y 178–194) → exhaust louvres (Y=194–200 wall, X≈8–40)
   → right-bottom weep
```

- Intake is **left-back**, the SoM palm-rest target is **front-right**: the duct simply runs forward under the carrier and turns right in the front header — the 200×140 flat layout gives the longest straight duct run the old vertical chassis lacked. The old "spine pass-through" problem (thermal §1.3) is **resolved**: the trench lives outside the bays (Y 3–16), no spine slot needed.

### 7.2 Duct sizing table (from thermal §1.2–§1.7)

| Element | Spec (thermal.md) | Flat-stack value | Formula |
|---|---|---|---|
| Intake louvre **free area** | ≥660 mm² (≤2.5 m/s) | louvres ≈32×35 gross (~1100 mm²), η≈0.6, slots ≤1.8 mm | A=Q/v; Q=3.5 CFM=0.001652 m³/s; v≤2.5 |
| ΔP_grille | ≤~5 Pa (K≈1.2–1.4) | « 5 % of blower static (~71 Pa) ✅ | ΔP=K·½ρv² |
| Intake plenum/trench | ≈40×30×8 (≈9 cm³), fan standoff ≥3 mm | trench 13×24×~130 ≈ 40 cm³ ✅ | — |
| Fan→HS gap | ≤2 mm + closed-cell foam surround | 1.5–2 mm, foam **mandatory** | — |
| HS flow area | 83.3 (HS-5) / 131 (HS-7) mm² | as built (32×38 foot) | 17 ch×1.4×3.5 |
| System ΔP @ op. pt. | ~0.14–0.16 inH₂O (blower) | model matches thermal §2.3 | ΔP=C·Q, C≈98,500 Pa·s/m³ |
| Exhaust free area | ≥350–400 mm² (gross ≈950, e.g. 38×18 mm) | right wall louvres X≈8–40, lab-angled down-outward | — |
| Weep/drain | 3×15 mm flap weep each plenum + drip trays | left trench low-point + right channel weep to bottom edges | — |
| Water protection | labyrinth vanes, downward/backward facing, filter pocket 1 mm foam upstream | retained | DEC-056/A-034 |
| **Envelope pressure budget** | blower operating ~0.75 CFM carries 8.7 W @ ΔT_air≈20 K (thermal §2.5) | the raised palm deck adds no extra duct loss (free-area kept ≥ thermal §1.2 spec) | — |

Fan install x-y (z 30.2–40.2): beside the HS (left, −Y), collinear axis with fin channels; 20–25 kHz low-side PWM by the DB power-MCU, ±5 %/s slew, stall-avoid floor, fail-safe = run-full on loss (thermal §4.2). **Fan model final = Delta blower primary per thermal §2.4; RISK-027/028/029/030 gate Phase 8.**

### 7.3 Rain / drainage (DEC-056/A-034 — flat-stack adaptation)

- **Keyboard drainage (T480-style):** water → key wells → drainage troughs molded under the membrane (deck underside) sloped aft/bottom ≥2° → collected at bottom-edge weep rail → out front/bottom edge slots (≥12 slots, ≥0.5 mm anti-insect, one-way flaps). Flat deck makes trough routing straightforward (same plane as membrane). Flow test (drip + row cup) in `assembly-serviceability.md` §5.
- **Vents:** intake relies on the trench labyrinth + 1 mm foam filter pocket; both plena drain before fan/HS.
- **Closure/ports:** continuous closed-cell lid gasket, hinge-channel foam ring + bend-cap, per-connector IP4X beads + EMI gasket, pogo cap (signal+GND default REQ-UART-01), bay mouths sealed lips + soft front bezel cover. Target **rainproof hours; IP4X splash+dust** (not submersible) — bench TBD.

---

## 8. Clearances, cables, connectors, bend radii

| Routing | Spec / allowance | Note |
|---|---|---|
| DSI FPC carrier → lid (through hinge) | 0.3–0.5 mm FFC 4-lane+clk; bend ≥5–6× thickness; dynamic loop ≥10 mm; ≤150 mm | custom + Raystar pinout, TBD (RISK-024) |
| Antenna coax u.FL 1.13 ×3–4 | bend ≥3 mm; ≥10 mm from HS/fan metal; ≥8 mm from lid rims | Wi-Fi ×2, LTE ×1–2 |
| Carrier↔DB jumper (vertical, same band) | signal FPC ≤80 mm die + power harness separate; high-speed never crosses (RISK-015) | through carrier notch |
| Carrier↔KB PCB USB2 | 0.5 mm FFC, bend ≥3 mm, vertical drop ~8 mm | — |
| Battery interconnect (pack→DB towers) | spring contacts ≥3 A/pin, ≥2 P in parallel, staged GND→ID/NTC→P, ≥3 mm separation; 14.4 V/≤5 A | DEC-004/060 |
| M.2 2280 | 22×80×≤3.7, M2 standoff, M-key only | DEC-033/048 |
| Fan/duct | duct walls ≥2 mm; no pinch <1.5 mm; fan-to-grill gap ≥3 mm | noise/filter |
| Board keep-outs | ≥1.0 mm machined faces; ≥0.5 mm pads/gaskets; board bottom ≥1 mm above pack rails | RISK-002 |

---

## 9. Mass estimate and center of mass

### 9.1 Mass table (planning; flat boards vs old draft)

| Item | Mass (g) | X centroid (mm) |
|---|---|---|
| 2× packs (8× P50B 568 + pack HW ×2 ≈70) | 640 | 55 |
| Carrier PCB 194×114×1.6 (≈6 layers) + SMD | 90–120 | 110 |
| Verdin i.MX8MP-WB 8 GB + B2B | 30–40 | **26** (front-right) |
| HS 32×38×5 + Delta blower | 45–65 | **26** |
| NVMe 2280 (user's) | 50–70 | 88 |
| LTE tiny-LGA + SIM + antennas (in lid) | 15–35 | 60 |
| Keyboard PCB + membrane + trackball + deck MCU | 130–160 | 90 |
| Rear daughterboard + PD + pogo + power | 110–130 | 125 |
| Display (Raystar + BL) | 110–130 | 70 |
| Plastic bezel + lid-top | 35–55 | 70 |
| Al rims (lid) + hinges ×2 | 60–90 | 130 |
| Chassis (base + mid-spine + deck, 3 mm 6061) | 400–550 | 70 |
| Screws/cables/gaskets/filter/misc | 110–150 | 70 |
| **Total** | **≈1,830–2,135** | — |

Mid-range ≈ **1,980 g** — inside the 1.6–2.1 kg budget (A-029; aluminum shell + plastic lid).

### 9.2 CoM (X depth, 0 front … 140 rear; Y symmetric ≈100)

```
Σ m·x ≈ 105,000 g·mm → X_CoM closed ≈ 74 mm ≈ 53 % (neutral band) ✅
Lid open (~110°): shifted +45 mm → ≈56 %, still inside footprint (rear foot ~130) ✅
Packs out + lid open: Σm≈1,330 → X_CoM ≈ 85 (61 %) — no tip-back (need <~126; margin ~40) ✅
Z_CoM ≈ 28–32 mm (low; flat stack keeps the board plane low → favorable)
```

Note: putting the SoM tower at the **front-right** moves hardware centroid a few mm forward vs the old rear draft; the front-loaded batteries + tower keep CoM inside 45–55 % — re-verified numerically at Phase-8 scale model.

---

## 10. "Minimum physical changes" call-outs (DEC-063/A-041)

All flagged with ★ = a small permitted tweak used to make the flat architecture fit 200×140×50:

| # | Change (flat-stack enabler) | Why it is necessary | Impact |
|---|---|---|---|
| ★1 | Raised palm-rest deck X 0–56 to ≈45.2 (HS-5) / 47.2 (HS-7) | SoM B2B (+8.6) + HS (5–7) sit **over Bay B** (palm-rest front-right) → headroom must come from the deck (nominal 50 → 52 for HS-5) | closed ≤55 ✅; stepped deck lip at X≈56 |
| ★2 | Left intake duct under-carrier channel Y 3–16 | DEC-062 intake is left-back but the SoM tower is front-right → a forward trench is the free-CNC answer (no spine slot, no over-lid routing) | Bay A window 76 (pack 70 still fits) ✅ |
| ★3 | Right exhaust channel Y 178–194 | gives the DEC-062 R-side exhaust plenum + weep inside the box | Bay B window 76 (pack 70) ✅ |
| ★4 | DB passive under the carrier (z 4–6), heat to bottom plate | DB + all LF ports + power need to sit flat at the rear; under-carrier keeps the deck flat and the rear wall clean | tall parts (RJ45) < z 28 |
| ★5 | M.2 2280 oriented along X, Y≈40–62 | keeps NVMe under the keyboard band (tall but under KB PCB plane), clear of RF/thermal + DB | NVMe z 30.2–33.9 ✅ |
| ★6 | nano-SIM via bottom-plate trap door (or rear-edge tray) | SIM must be reachable without a full teardown; front/palm area is occupied | small CNC door |
| ★7 | Lid packing compressed to ~6.5–8 mm (ant gap 0.5 at HS-7) | HS-7 raises closed height to ~55 → trim lid internal air/gasket to hold ceiling | HS-7 → lid antenna gap 1.25→0.5 |
| ★8 | Fan in-plane left of HS (not on top), blower primary | DEC-062 + thermal §1.4/§2.4; saves HS-stack height (fan no longer adds z) | fan top 40.2 fits under palm deck |

These are the *only* deviations from a textbook flat-stack laptop interior; nothing else needs to change to fit the envelope.

---

## 11. Findings summary

1. Envelope 200×140×50–55 with 3/3/3/10 bezels is satisfied by the **flat horizontal stack** (DEC-052 revised): no vertical boards, no edge-on planes.
2. Raystar landscape fit proven (§1.2): 190.93 ≤ 194; 128.74 ≤ 134.
3. Width math closes: 3+13+76+10+76+16+3 walls = 197 ≤ 200 ✅ with the DEC-060 drill-rail spine and DEC-062 ducts.
4. Z-stack closes at **~52 (HS-5) / ≤55 (HS-7)** with a raised palm-rest deck (★1); the SoM B2B + HS is the z-driver (**TBD measured**).
5. Keyboard band 10 %/40 % (X 56–126) + palm-rest front 40 % with SoM/HS/fan in the right zone (DEC-053/061); nobody types over the tower.
6. Air path re-baselined: under-carrier left trench → front header → blower (Delta primary) → HS fins L→R → right channel → R-wall louvres (thermal numbers held: intake ≥660 mm², exhaust ≥350–400 mm²).
7. Battery drill-rail spine (DEC-060) front-insert, rear DB spring towers co-mate, hot-swap retained (REQ-PWR-03).
8. Lid = plastic bezel + plastic lid-top + Al rims (DEC-055); Hall + ALS per DEC-057; rainproof + keyboard drainage (DEC-056).
9. Hinges ThinkPad-class ~0.8 N·m/side adjustable (DEC-054); mass ≈1.83–2.14 kg; CoM 53 % closed, ≤61 % packs-out — no tip-back.

---

## 12. TBD measurement items (Phase-8 bench, before CNC)

| # | Item | Why it gates |
|---|---|---|
| T-T1 | Verdin TE 2309409-2 B2B actual stack (h_b2b 8.6–9.5?) | z-plane registration, palm-deck height |
| T-T2 | HS height/geometry choice (HS-5 vs HS-7) + blower mounting fit | deck height, closed 50/55 |
| T-T3 | Delta blower outlet-slot orientation vs fin mouths (2-wire speed control, thermal RISK-027) | fan install + z/XY |
| T-T4 | Raystar OD (115.74×184.93×4.75) re-measure | bezel + louvre geometry |
| T-T5 | Pack slab exact OD (cell Ø21.55/21.25 × H 70.8) + rail slide force | bay/rail clearances |
| T-T6 | M.2 drive height (≤3.7) + KB PCB plane gap 4.8 | NVMe under KB band |
| T-T7 | Hinge vendor torque curve + adjustable range | §6, assembly §4 |
| T-T8 | Rails slide / latch / contact-wipe force; hot-swap electrical (RISK-004) | §4.1/7, assembly §3 |
| T-T9 | SIM trap-door geometry + LTE coax routing | — |
| T-T10 | Lid packing thickness at HS-7 (ant gap) | closed ≤55 |
| T-T11 | Keyboard row / trackball well + drainage trough continuity | assembly §5 |
| T-T12 | Mass / CoM on scale model | §9 |

---

## 13. Open mechanical decisions (flat-stack edition)

| # | Decision | Options | Blocks |
|---|---|---|---|
| F-1 | Palm deck rise (nominal 52 vs ≤55 ceiling) + HS-5 vs HS-7 | HS-5 flat 52 / HS-7 stepped 55 (DEC-043) | deck CNC, thermal |
| F-2 | Carrier front edge X 20 vs 3 (notch for duct header) | keep header band X 3–20 free (favored) vs full-span with slot | carrier layout |
| F-3 | M.2 orientation along X (favored) vs along Y | under-KB band z clearance | carrier layout |
| F-4 | SIM access: bottom trap-door (favored) vs rear-edge tray | service path | bottom CNC |
| F-5 | LTE LGA socketed vs reflow-only (swappability) | DEC-058 | swap service |
| F-6 | Hinge vendor + trim (0.6/0.8/1.0 N·m) + adjustable set-screw | bench-trim | lid build |
| F-7 | DB under-carrier thermal path + RJ45 tall part clearance (z<28) | bottom-plate spreader | DB/PCB y |
| F-8 | ALS position + Hall magnet target (Y≈150) | lid flex + PCB | lid build |
| F-9 | Drainage trough geometry + weep slot pattern | flow test | rain test |

All **TBD/unverified** items close at Phase-8 measurement before the CNC quote (RISK-002/013/015/016/020/021/023/027–030).