# Mechanical — Assembly and Serviceability (Phase 6, flat stack)

Status: **Phase-6 mechanical design draft — 2026-08-30.** This is the **authoritative revision** for assembly/service of the **flat horizontal stack** (DEC-052 revised, A-032). It supersedes the earlier vertical-board assembly draft. All torque/fastener values are typical CNC-aluminum practice; every **TBD** item must be confirmed against purchased parts before build. Mass/CoM reference: `envelope-and-stack.md` §9. Battery packs and hot-swap electronics are safety-critical (REQ-PWR-05) — this file stays in sync with `hardware/…` (Phase 6, TBD) and the risk register (RISK-002/004/008/013/015/016/020/021/023/027–030).

> **Architecture in one line:** three **flat boards on normal laptop planes** — keyboard PCB (top, part of the deck), carrier motherboard (SoM/M.2/LTE/hub), rear daughterboard (all LF ports + power + battery contacts) — interconnected by FPC/J-connectors + cables; SoM + HS + fan in the right palm-rest zone with the left-back→right air duct; drill-rail battery spine (DEC-060); ThinkPad-class hinges (DEC-054); plastic bezel + plastic lid-top (DEC-055); rainproof + keyboard drainage (DEC-056). Coordinates: X depth (front 0, rear 140), Y width (left 0, right 200), Z floor 0.

---

## 1. Assembly order (bottom-up, base first, lid last)

Chassis is a normal clamshell (DEC-002). Base assembles in **7 sub-groups**; each step is one physical operation with its own fastener/torque. Service screws come from the **base/bottom** except where noted (serviceability requirement).

### Sub-group A — chassis, rails & bay prep
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| A1 | Inspect + deburr CNC base, mid-chassis spine, top deck, lid rims; verify spine drill-rails, palm-rest step, weep slots, intake/exhaust louvre passes (envelope §4/§7) | base, spine, deck, lid (6061-T6) | manual, no screws |
| A2 | Mask anodize-off contact zones (see §7 ESD) — tape before anodize | all Al parts | — |
| A3 | **[Drill rails, DEC-060]** Verify both rail pairs machined into the mid-spine walls (Bay A Y 16–92, Bay B Y 102–178); deburr; dry-slide a reference pack (entry force, no binding) | spine + rails | manual gauges; pack-slide |
| A4 | Fit Bay **rear spring-contact towers** onto the DB front face at X≈108 (staggered GND→ID/NTC→P/GND, ≥3 A/pin) | contact towers ×2 | M2 ×4, 0.5 N·m (TBD) |
| A5 | Fit rail wear strips (POM 0.2 mm crush) + bay entry chamfers + polarity-key verify | 4 rails + 2 ramps | M2.5 ×5, thread-lock; ~0.35 N·m (TBD) |

### Sub-group B — rear daughterboard (flat, rear band X 110–134)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| B1 | Lay the **DB flat** on the floor bosses at X 110–134 (z 4–6), contact towers facing forward (X≈108 → into the bays) | DB | M2 ×4 c/s steel, 0.5 N·m (TBD) |
| B2 | Fit side/center standoffs + floor spreader pad (heat to bottom, envelope §4.5) | standoffs, pad | M2 ×4 |
| B3 | Cable the LF ports + USB-C PD (65 W, DEC-031) + pogo UART + AUX + power button/LEDs; **pre-wire battery towers** (A4) | — | — |
| B4 | Fit hinge-channel gasket ring to the rear central channel (DSI/coax pass) | foam gasket | — |

### Sub-group C — battery bays (front, DEC-060, hot-swap REQ-PWR-03)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| C1 | Confirm bay routing: pack slides in from front (X+), positive stop at X≈108 = DB contact towers co-mate in last ~6 mm (staged GND→ID/NTC→P) | bays A/B | per A3 |
| C2 | Fit bay latch + **polarity key** (asymmetric spine T-rail groove; reverse insertion physically impossible) + optional magnet pocket at front bezel | latch/cam + key | M2 ×4 |
| C3 | Dry-slide 2 reference packs (fully-dummy): entry force, latch click, contact wipe, "seated" LED go/no-go | reference packs | — |

### Sub-group D — carrier motherboard (flat, over the bays)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| D1 | Bench-build the carrier (z plane ~28.6–30.2): SoM-to-B2B (TE 2309409-2, 260-p, 0.5 mm), M.2 2280 M-key NVMe (DEC-033/048: keying + silkscreen "NVMe M.2 2280 ONLY"), LTE tiny-LGA (DEC-058), nano-SIM tray, USB2 hub (DEC-049), U.FL feeds, codec | carrier + SoM + NVMe + LTE + SIM | — |
| D2 | Lower the carrier flat onto the spine-top standoffs (X 20–134, Y 3–194); verify M.2 rides under the future KB-plane gap (envelope §4.4: z 30.2–33.9) | carrier | M2 ×4 + standoffs, 0.5 N·m (TBD) |
| D3 | **Carrier↔DB jumper:** vertical signal FPC (≤80 mm die) + power harness (12/5/3.3 V + GND, separate) through the carrier notch over the DB band | FPC, harness | — |
| D4 | Fit the **SIM trap-door** (bottom plate) under the SIM socket; verify SIM service without teardown | trap door | M2, countersunk |

### Sub-group E — SoM + heatsink + fan + duct (right palm-rest zone, DEC-053/062)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| E1 | Thermal pad on SoM (or base-boss + TIM); place **HS (32×38×5/7), fins running +Y (left→right)** on the SoM die zone (X 8–43, Y 138–174) | HS, pad | M2 ×4 + pad, cross-torque |
| E2 | Fan **in-plane LEFT of the HS** (−Y): Delta BFB0305HA-C blower (primary, thermal §2.4), inlet face (−Y) to duct header, **outlet slot (~30×5) ≤2 mm from fin mouths**; **closed-cell foam surround** around fan→HS interface (no bypass, thermal §1.4) | blower, foam | M2 ×4 |
| E3 | Duct/header: confirm left under-carrier intake channel (Y 3–16) → front header (X 3–20, z 28.5–38) → fan inlet; walls ≥2 mm, no pinch <1.5 mm; insert **1 mm foam filter pocket** upstream of fan (RISK-029) | duct parts, filter | M2 or adhesive |
| E4 | Route: carrier↔DB jumper FPC/harness (done D3); DSI FPC + 3–4 U.FL coax → rear central channel (hinge loop later); keyboard HID J/FPC → KB plane | FPCs, coax | — |

### Sub-group F — keyboard PCB + top deck + membrane (DEC-061 band X 56–126)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| F1 | Seat the **keyboard PCB flat** at z 35–39 (X 56–126) on deck standoffs; plug the USB-HID J/FPC (drop ~8 mm to carrier) | keyboard PCB | M2 ×4, 0.5 N·m (TBD) |
| F2 | Fit membrane + trackball well (G–H–B, X≈56, Y≈90–110, DEC-025/A-024; **bench-test ball first — RISK-020/DEC-050**, keep well interchangeable); map key matrix flex into KB PCB | membrane, trackball | adhesive + key guides |
| F3 | Screw the **top deck** (stepped: keyboard zone top 42, raised palm-rest zone X 0–56 top ≈45) down over the base; verify drain gutters align under membrane, bay mouths open at front | top deck | M2.5 ×6 c/s, 0.45 N·m (TBD) from below |
| F4 | Verify keyboard travel, trackball read, palm-rest louvre/filter fit, drainage trough continuity (§5.1) | — | — |

### Sub-group G — lid (plastic lid-top + bezel + panel; DEC-055)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| G1 | Build lid rear: **plastic lid-top sheet** (RF-transparent) to Al rims; mount Wi-Fi/BT + LTE antenna elements on its inner face behind the panel (≥8 mm from rim, ≥4 mm from panel back) | lid-top, antennas | M2 ×4 + adhesive |
| G2 | Install Raystar panel into the **plastic bezel** (3/3/3/10); bezel snaps/screws to rims | panel + bezel | M2 ×4 (TBD) |
| G3 | Fit **ALS** (aimed **away** from panel, outward through the plastic lid-top) + **lid magnet** for the Hall (Y≈150) | ALS flex, magnet | adhesive pockets |
| G4 | Run DSI FPC + coax into the lid channel with a **≥10 mm dynamic radius bend-cap**; attach hinge flanges to the rim doublers | FPC, hinges | — |

### Sub-group H — hinges + closure + sealing
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| H1 | **Install ThinkPad-class hinges (DEC-054):** ~0.8 N·m/side nominal (0.6–1.0), adjustable set-screw; barrels rear rim Y 8–34 / 166–192; base ≥3 fasteners/side into reinforced rear-wall + spine bosses (M2.5–M3, inserts ≥4 mm); lid ≥4/side into rim doublers | hinges ×2 | M3 + inserts; set-screw per §4.2 |
| H2 | Loop DSI FPC + coax **before** tightening hinges (flex first, then hinge screws) | — | — |
| H3 | Closure gasket: lid-to-deck line (continuous closed-cell), hinge-channel foam ring + bend-cap, port IP4X beads + EMI gaskets, palm-rest louvre mesh/filter, bay lip seals | gaskets | — |
| H4 | Final: deck screws + front bezel over bay mouths; power-on, thermal smoke, lid-hold torque check, drain test (§5) | — | M2.5 c/s, 0.5 N·m (TBD) |

**Service access summary (screws from base):** remove **lid** (2 hinges, FPC-first) → remove **top deck** (bottom screws; raised palm-rest steps down at X≈56) → **carrier + thermal tower** (bottom standoffs, D2/E1/E2) → **SoM/NVMe/LTE/SIM** on the carrier; **keyboard PCB** under deck front (F1); **front bezel + bays** to reach DB contact towers; **DB** under carrier at rear (B1). Battery packs need **only the front latch** — no deck removal (hot-swap, REQ-PWR-03/DEC-060). SIM via bottom trap-door (D4) without teardown.

---

## 2. Battery bay + drill-rail design (front, DEC-060; hot-swap REQ-PWR-03/A-038)

### 2.1 Bay layout (drill-tool pattern)
- Bays: each pack **105 (X) × 70 (Y) × 24 (Z)** at X 3–108; Bay A Y 16–92, Bay B Y 102–178; center spine Y 92–102 (10 mm) carries the machined rails on both side walls (DEC-060). Width check: left duct 13 + bay 76 + spine 10 + bay 76 + right channel 16 + walls = 194 (envelope §1.2).
- **Rails:** two horizontal machined channels per bay (spine wall + outer wall); pack slides along them front→rear; **polarity key groove** (asymmetric T-rail) makes reverse insertion physically impossible.

### 2.2 Retention + contact design
| Item | Design |
|---|---|
| Retention | Spring-loaded front latch + drill-style push-guide with audible click; optional magnet pocket for final stop (shear-only, withdraw-proof on drop — verify by drop test, else remove magnet; latch hold ≥8 N) |
| Hot-swap contacts | Rear block = **staggered-length spring pins** (GND→ID/NTC→P/GND) so OR-ing sees ground first, then power, with pre-charge on sense (electrical detail → hardware Phase 6, RISK-004/DEC-004) |
| Contact type | Gold-plated spring pins ≥3 A continuous/pin, ≥2 pins parallel on the main rail; **TBD vendor** (Mill-Max 0964-class or bespoke CNC) |
| Sense/balancing | Dedicated low-current Kelvin sense pins |
| DB front-face position | Contacts co-mate **on insertion** (last ~6 mm travel) — no separate plug action (DEC-060) |

### 2.3 Insertion direction
1. Pack inserts front→rear (X+); key engages within first 10 mm (cannot be backwards).
2. Green/amber "seated" LED on the deck (sense-pin wired) + latch click; co-mate verified by contact wipe.

---

## 3. Swapping / replacement procedures

### 3.1 Battery packs (hot-swap, both removable while powered — REQ-PWR-03, DEC-060)
1. Sub-group C tools only (front bezel + latch). Release latch, **pull pack straight out along X−** on the spine rails.
2. Insert new pack front-first on the rails → positive stop; polarity key prevents reversal; hear/feel latch + LED go/no-go.
3. No deck removal, no shutdown. **Process verified Phase-8 bench** (inrush/throttling, RISK-004).

### 3.2 NVMe M.2 2280 (REQ-COMP-03, DEC-033/048)
1. Open lid + top deck (H/G/E/F screws from bottom).
2. Remove thermal stack (fan + HS + foam, E1/E2) — usually not needed (drive is out of the thermal zone, X≈55–135 along Y≈40–62).
3. Unclip/unscrew M.2 M-key socket; remove standoff screw M2; slide drive out along X.
4. Replace, re-torque M2 standoff **0.4 N·m (TBD)**; re-seat FPC/fan; reassemble.
5. Silkscreen/keying verify (M-key only, "NVMe 2280 ONLY", DEC-033/A-027).

### 3.3 SoM (Verdin i.MX8MP-WB, B2B 260-pin)
1. Deck open + tower off (§3.1/3.2.1–.2).
2. Release B2B lock levers both sides (0.5 mm SODIMM-family); lift module straight up along Z off the flat carrier.
3. Inspect socket; replace; press to seat, re-lock.
4. Re-apply thermal pad/HS/fan + foam (E1/E2); cross-tighten carrier standoff screws evenly.

### 3.4 LTE tiny-LGA (DEC-058) — rework-class, not hand-swappable
1. Deck open + tower off; **reflow-remove** the ≤~30 mm LGA on the carrier with hot-air (profile TBD by solder alloy).
2. Clean pads; fresh paste; place + reflow; inspect joints (X-ray if available). **Alt: socketed LGA if available — TBD** (S-6).
3. Re-assemble. SIM tray keeps radio data unchanged on rework budget.

### 3.5 Display / panel (plastic lid-top, DEC-055) + SIM
- Lid: open lid; remove bezel (plastic frame); unseat panel; disconnect DSI FPC + BL at the carrier end (flex first). Antennas on the plastic lid-top stay unless damaged.
- SIM: bottom trap-door (D4) → re-seat SIM socket → close.

---

## 4. Lid hinge + friction torque (DEC-054; ThinkPad-class, flat-stack re-baseline)

### 4.1 Spec & rationale
- Two multi-point friction hinges, **~0.8 N·m/side nominal (0.6–1.0)** (envelope §6), barrels rear rim Y 8–34 / 166–192; reinforced mounts: base ≥3 fasteners/side into rear-wall + spine bosses (inserts ≥4 mm), lid ≥4/side into rim doublers.
- Hold: lid ≈0.25–0.30 kg, CoM arm ≈60 mm ⇒ T_hold ≈0.082 N·m/side; spec 6–12× hold for carry-by-screen + hold-at-angle.
- One-finger-open / no-base-lift ceiling ≈0.47 N·m/side (m_base ≈1.9 kg). ⇒ nominal **0.8 N·m adjustable set-screw**; bench-trim between carry-hold and no-lift. ≥15k cycles, ≤20 % tolerance. **TBD vendor + curve** (candidates B-022 replaced by TP-class spec).

### 4.2 Installation (TP-practice, torque trim)
- Align barrels ∥ rear rim; equalize torque per side (mismatch = lid twist); set-screw torque **TBD** (order: hinge loose → route FPC loop → snap lid → set friction → verify 45–135° hold). Final: one-finger open, no base lift, carry-by-screen on bench.

---

## 5. Sealing, keyboard drainage + rain test (DEC-056)

### 5.1 Keyboard drainage (T480-style weep) — flat-deck adaptation
- Membrane over molded drain troughs in the deck underside, sloped aft/bottom ≥2°; water exits **bottom-edge weep slots** (≥12, ≥0.5 mm anti-insect, one-way flaps) clear of bays and rear ports. Flat deck: troughs are straight-run between the KB band and the bottom lip — simpler than a vertical-board chassis.
- Verify gutter continuity at assembly (F4).

### 5.2 Sealed vents / ports
| Opening | Seal | Notes |
|---|---|---|
| Palm-rest raised deck louvre (HS intake) | louver + mesh + replaceable 1 mm filter pocket + low-point drain weep | intake side; DEC-062 |
| Right-wall exhaust louvre | louver bank (down-outward vanes, labyrinth) + low-point weep | DEC-062/056 |
| Under-carrier intake channel (left, Y 3–16) | labyrinth + drip tray + weep | intake path |
| Lid-to-deck closure | continuous closed-cell gasket | keeps FPC radius ≥10 mm |
| Hinge FPC/coax channel | foam ring + soft bend-cap | dynamic loop |
| Rear connector cut-outs | conductive EMI gasket + IP bead | spatter/IP4X |
| Pogo port | silicone plug / magnetic cap (signal+GND default) | REQ-UART-01 |
| Battery bay mouths | soft lip + front bezel cover over latches | no water to contacts |
| SIM trap-door | gasket + recessed M2 screws | — |
| Overall target | rainproof **hours**, IP4X splash+dust (not submersible) | **TBD bench** |

### 5.3 Drain / rain test procedure (bench) — flat targets
1. **Drip test:** 2 L water over the top deck + keyboard area, 2 mm nozzle, 20 min; no water reaches carrier/daughterboard; all water exits weep slots/vent drains.
2. **Row cup test:** 100 mL into each key-row gutter inlet; ~90 % out within 60 s (T480-class), remainder trapped in drain tray.
3. **Vent drain check:** spray palm-rest louvre + exhaust louvres; confirm low-point drains clear before the fan duct.
4. **Closure seal test:** lid closed, 5 min continuous spray on the seam; no ingress beyond gasket.
5. Log results; iterate weep geometry if >10 % retained (**TBD**).

---

## 6. Keyboard deck mounting (REQ-KB-01, DEC-021/025/045)

- **Top deck = stepped:** keyboard zone top 42 (X 56–126), raised palm-rest zone top ~45 (X 0–56, over the SoM/HS/fan + duct header). The keyboard band sits fully over the flat deck; the raised part is purely the palm-rest (DEC-053/061).
- Keyboard **PCB is flat** at z 35–39 (own plane, X 56–126); membrane flat-flex + trackball sensor pigtail (ADNS-9800, RISK-020 ball-size caveat — **must test-mount first**; keep well interchangeable) connect into it; STM32G0 deck MCU scans → USB-HID → carrier (DEC-045).
- Net key travel ~2 mm above deck; clearance to the closed lid bottom (stepped seat): keycap tops ~44 vs lid seat ≈45+ → ≥1 mm ✅ (verify with closed-lid gasket).
- Drain gutters under the membrane are part of the deck (F4).

---

## 7. ESD / grounding with the plastic lid-top (DEC-055)

- **Grounding budget (A2 masking):** bare-Al zones at battery contact tower areas (spine rails + rear blocks), connector screw bosses, hinge lugs, pogo port frame, louvre/vent frame edges, ground standoffs (carrier/DB), SIM/antenna grounds, rim doubler bosses.
- **The aluminum rims/edges are the reference:** ground the metal rim continuously (structure + grounding ring); the **plastic lid-top is dielectric**, so antennas behind it are fine (no RF window needed); antenna grounds return via coax shields to the carrier RF GND → chassis star (DEC-055/A-033).
- **Ground strategy:** chassis-star ground at the daughterboard; battery-rail GND ties via bay contact towers; each board ground plane stitched to its mounting bosses with ≥2 screws; FPC/cable shields terminated to chassis at both ends; hinge barrels equalize lid rim↔base via finger-spring/conductive tape.
- **ESD test:** ±8 kV air/contact discharge to all user-accessible metal (keys, trackball, edges, connector shells, rim) with deck grounded in PD/AC mode — Phase-8.
- Always work grounded (mat + wrist strap); never unplug packs on an ungrounded bench.

---

## 8. Tool list

| Purpose | Tool |
|---|---|
| All body screws | M2.5 torx T6 / Phillips #0 (standardize to torx) |
| M.2 / SoM / boards | M2 hex or PH0 + small torque driver (0.1–0.6 N·m) |
| Hinge set-screws | T6/T8 + torque driver (§4 band) |
| Drill-rail bay latch / contacts | PH0 + non-marring prying tool |
| LTE LGA rework | hot-air reflow station + stencils, temperature-controlled (°C TBD) |
| Displays/FPCs | nylon spudger, ESD tweezers, anti-snag hook |
| Thermal | thermal pads + wipes + torque driver ±10 % for HS reuse |
| Drain/rain test | drip rig + flow meter + 100 mL cup (§5.3) |
| Assembly | ESD mat, wrist strap, M2.5 tap for dry-fit thread cutting |

---

## 9. Safety

1. **Sharp edges:** every user-handled edge (deck, keys, bays, lid rims, front bezel) gets a **0.3–0.5 mm chamfer/fillet**; no 90° knife edges on palm rest or hinge zone; thin rubber feet/edge tapes on lid rims (TBD).
2. **Battery polarity / latch anti-pop (REQ-PWR-05):**
   - Asymmetric spine-rail key prevents reverse insertion (§2/DEC-060).
   - Latch must hold pack against contact wipe (≥8 N) under vibration — re-tested with v0 prototype (RISK-008/023).
   - Each pack BMS has reverse-polarity protection + polyfuse; DB OR-ing is ideal-diode (no back-feed, RISK-004). Never assemble damaged-wrap cells.
3. **Battery transport/UN38.3:** custom packs — carry path/declaration per RISK-008/023; SoC ≤30 % before flight.
4. **Live-deck opening:** remove power before any deck/board work **except** the hot-swap battery step (design + bench-verified).
5. **High-current PD tray:** PD 20 V/65 W isolated + fuse-protected, connector-recessed; electrical review gate (REQ-PWR-05) before first power.
6. **Thermal stack:** fan + HS are live-adjacent to SoM; torque HS mounts evenly; keep fingers clear of blade; redirect airflow away from skin-contact surfaces (palm-rest deck ≤45/50 °C design review).

---

## 10. Open mechanical decisions (service side, flat-stack edition)

| # | Decision | Options | Blocks |
|---|---|---|---|
| S-1 | Contact tower vendor/type (drill-rail rear block) | Mill-Max 0964-class / CNC leaf / pogo (verify) | bay + DB |
| S-2 | Rail-latch type + polarity key profile | slide vs cam vs push-guide; T-rail key geometry | bay retention |
| S-3 | Hinge part + torque trim (0.6/0.8/1.0 N·m/side) | adjustable set-screw; lid-flex risk | lid build |
| S-4 | Deck stepped height (palm X0–56 top 45 vs 47 HS-7; keyboard 42) + troughs | deck CNC + closed 50/55 | deck CNC |
| S-5 | Pogo port seal (cap vs plug) + level-shift enable | strictly signal+GND until electrical review | UART port |
| S-6 | LTE LGA socketed vs reflow-only (swappability) | land-pad vs socket (DEC-058) | swap service |
| S-7 | Vent drain geometry + mesh pitch; intake filter pocket | louver 0.5 vs 0.8 mm; drain location | rain test |
| S-8 | SIM trap-door vs rear-edge tray | bottom CNC vs rear | SIM service |
| S-9 | Fan model final (Delta blower primary vs Sunon axial) + PWM/tach path | thermal RISK-027 | fan install |

All **TBD/unverified** items must be closed by Phase-8 measurement (mass, contact wipe, rail slide force, hinge torque, water retention, ESD) before first-unit build — mirrored from `envelope-and-stack.md` §12.