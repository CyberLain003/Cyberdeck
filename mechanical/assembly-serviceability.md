# Mechanical — Assembly and Serviceability (Phase 6)

Status: **Phase-6 mechanical design draft** — 2026-08-30. All torque/fastener values are typical CNC-aluminum practice; each **TBD** item must be confirmed against the purchased parts before build. Mass/CoM cross-references: `envelope-and-stack.md` §9. Battery packs and hot-swap electronics are safety-critical (REQ-PWR-05) — this file stays in sync with the electrical design (`hardware/…` Phase 6, TBD) and the risk register (RISK-002/004/008/013/015/016/020/021/023).

> ⚠️ **MAJOR architecture rework (2026-08-30, DEC-052…060).** The previous assembly flow for a **flat horizontal carrier tray** is **superseded**. Approved architecture is **three vertical boards** (carrier with SoM, rear daughterboard with all low-frequency ports + power management + battery contacts, keyboard PCB), a **SoM in the right palm-rest column** with the thermal tower rising from it, **drill-tool-pattern battery rails in the middle chassis spine** (DEC-060), **ThinkPad-class multi-point hinges (~1 N·m-class/side)**, a **plastic bezel + plastic lid-top** behind the display, **rainproof-with-keyboard-drainage** sealing. Assembly below is re-sequenced around these: boards are **inserted vertically**, interconnected by FPC/J-connectors + cables, then the top deck, palm-rest and lid close over them. See `envelope-and-stack.md` §2/§3/§4 for zone coordinates and z-values.

---

## 1. Assembly order (bottom-up, base first, lid last)

Chassis approached as a normal clamshell laptop (DEC-002) reinterpreted for vertical boards. Base assembly proceeds in **7 sub-groups**; each line item is one physical step with its own fastener/torque set. Screws come from the **base/bottom** (serviceability requirement) except where noted.

### Sub-group A — chassis, spine & bay prep
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| A1 | Inspect + deburr CNC base, mid-chassis spine, top deck, lid rims. Verify spine drill-rails, palm-rest grill, weep slots, vent louver passes. | base, spine, deck, lid (6061-T6) | manual, no screws |
| A2 | Mask anodize-off contact points (see §7 ESD) — tape before anodize | all Al parts | — |
| A3 | **[Drill rails, DEC-060]** Verify the two battery rail pairs machined into the spine side-walls (left bay Y 10–92, right bay Y 102–180); deburr rail edges; check a reference pack slides freely (entry force, no binding) | spine + rails | manual gauges; pack-slide test |
| A4 | Fit bay rear contact blocks (spring-pin arrays, staggered-length) onto the daughterboard front face or a dedicated front bracket | contact blocks ×2 | M2 × 4, 0.5 N·m (TBD) |
| A5 | Fit rail wear strips (POM/POM-C or delrin, 0.2 mm crush) + bay entry chamfers | 4 rails + 2 ramps | M2.5 × 5 + thread-lock; ~0.35 N·m (TBD) |

### Sub-group B — rear daughterboard (vertical, rear wall)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| B1 | Stand the daughterboard **vertically** against the rear wall (X 116–137) on its bottom standoffs (z ≥ 4); screw to rear-wall bosses. | daughterboard | M2 × 4 c/s steel, 0.5 N·m (TBD) |
| B2 | Fit side/center standoffs + board-to-spine bracing | standoffs | M2 × 4 |
| B3 | Cable the low-frequency ports (USB-C PD 65 W power-only, magnetic pogo UART, AUX, power button, LEDs) to the board rear face; battery contact blocks already on its front face (A4) | — | — |
| B4 | Apply lid-FPC gasket ring to the rear central channel | foam gasket | — |

### Sub-group C — battery bays + drill-rail contacts (front, DEC-060)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| C1 | Confirm bay routing: pack slides in from front along the spine rails, positive stop at X ≈ 108 (daughterboard front contacts co-mate in last ~6 mm travel, staggered GND→ID/NTC→P/GND) | bays A/B | rail check per A3 |
| C2 | Fit bay latch + polarity key + optional magnet pocket at the front bezel (drill-style push-guide; polarity tab cannot be bypassed) | latch/cam + key | M2 × 4 |
| C3 | Dry-slide 2 reference packs (fully-dummy) both bays: entry force, latch click, contact wipe, "seated" LED go/no-go | reference packs | — |

### Sub-group D — vertical carrier + SoM + thermal tower (right palm-rest column)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| D1 | Build up the vertical carrier (on a bench at the column zone X 108–137, Y 96–197): SoM-to-B2B socket, M.2 2280 (M-key NVMe), LTE tiny-LGA (DEC-058), WB antenna feeds on board; keying + silkscreen check (DEC-033) | carrier + SoM + NVMe + LTE | — |
| D2 | Mount carrier **vertically** on its floor standoffs (plane across Y, z 4–39), SoM-to-palm-rest facing up; verify column clearance to rear daughterboard | carrier | M2 × 4 + standoffs, 0.5 N·m (TBD) |
| D3 | Fit thermal tower: thermal pad on SoM, 5–7 mm finned HS on S1–S4 bosses, Sunon 30×30×10 fan on top (z 25–35) with duct wall ≥ 2 mm; fan PWM/tach routed to daughterboard | HS, fan, pad | M2 × 4 + thermal pad; fan M2 × 4 |
| D4 | Route/interconnect (FPC + cables, DEC-052): carrier↔daughterboard signal FPC (≤ 80 mm die) + power harness (12/5/3.3 V + GND, separate from signal FPC); DSI FPC + 3–4 u.FL coax → rear central channel (hinge loop later); keyboard HID flex → right side of spine | FPCs, harness, coax | — |
| D5 | ESD/lid-FPC gasket over the rear channel opening before deck close | foam gasket | — |

### Sub-group E — keyboard PCB + top deck + membrane (front ~50 %)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| E1 | Stand the **keyboard PCB vertically** (plane across Y, z 29–39) on the mid-chassis spine plate; connect the deck USB-HID FPC/cable to the carrier | keyboard PCB | M2 × 4 + standoffs, 0.5 N·m (TBD) |
| E2 | Fit membrane + trackball well assembly on the top deck; trackball sensor pigtail to the vertical keyboard PCB; route key matrix flex into board top edge | membrane, trackball | adhesive + key guides |
| E3 | Screw the top deck down over the whole base (front ~50 % typing zone + palm-rest grill over the thermal tower; bay mouths exposed at front) | top deck | M2.5 × 6 countersunk, 0.45 N·m (TBD) from below |
| E4 | Verify keyboard travel, trackball read, palm-rest grill mesh/filter fit, drain gutters aligned under membrane | — | — |

### Sub-group F — lid (plastic lid-top + bezel + panel; DEC-055)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| F1 | Build the lid rear face: **plastic lid-top sheet** (RF-transparent) mounted to the aluminum rims; mount Wi-Fi/BT + LTE antenna elements on the inner face of the sheet behind the panel (≥ 8 mm from rim, ≥ 4 mm from panel back) | lid-top sheet, antennas | M2 × 4 + adhesive |
| F2 | Install panel (Raystar RFU800G-AYH-MNN) into the **plastic bezel**; bezel frames the panel with 3/3/3/10 (T/L/R/B) and snaps/screws to the rims | panel + bezel | M2 × 4 (TBD) |
| F3 | Fit ALS (aimed away from panel, outward through plastic lid-top) + lid-magnet for Hall (Y ≈ 150) | ALS flex, magnet | adhesive pockets |
| F4 | Run the DSI FPC + coax into the lid channel with **≥ 10 mm dynamic radius bend-cap**; attach hinge flanges to the lid rims | FPC, hinges | — |

### Sub-group G — hinges + closure + sealing
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| G1 | **Install ThinkPad-class hinges (DEC-054):** multi-point friction, ~1 N·m-class/side nominal (0.6–1.0), barrels z 42–50, Y 8–34 / 166–192; base side ≥ 3 fasteners/side into reinforced rear-wall + spine bosses (M2.5–M3, ≥ 4 mm thread engagement, threaded inserts); lid side ≥ 4 screws/side into rim doubler | hinges ×2 | M3 + inserts; set-screw torque per §4.2 |
| G2 | Loop DSI FPC + coax **before** tightening hinge (order fixed: flex out first, then hinge screws) | — | — |
| G3 | Apply closure gasket to lid-to-deck line, port gaskets, palm-rest grill filter/mesh | gaskets | — |
| G4 | Final: close deck screws + front bezel over battery mouths; power-on test, thermal smoke, lid-hold torque check, drain test (§5) | — | M2.5 countersunk, 0.5 N·m (TBD) |

**Service access summary (screws from base):** remove **lid** (2 hinges, flex first) → remove **top deck** (bottom screws) → vertical **carrier + thermal tower** (bottom standoffs, D2/D3) → **SoM/NVMe/LTE** on the carrier; **keyboard PCB** under deck front; **front bezel + bays** to reach battery contacts and daughterboard front face. Battery packs need **only the front latch**, no deck removal (hot-swap, REQ-PWR-03/DEC-060).

---

## 2. Battery bay + drill-rail design (front, DEC-060; hot-swap REQ-PWR-03)

### 2.1 Bay layout (drill-tool pattern, A-038)

- Two bays, each **70 (Y) × 105 (X) × 24 (Z)** pack, housed at **X 3–108, Y 10–92 (A) / 102–180 (B)**, z ≈ 3–27; center **middle-chassis spine Y 92–102** carries the machined rails on both side walls (DEC-060).
- **Rails:** two horizontal machined channels per bay (on the spine wall + outer wall), pack slides along them front-to-rear; a **polarity key groove** (asymmetric T-rail) makes reverse insertion physically impossible.
- Travel: front mouth (X=3) → positive stop at X ≈ 108 = daughterboard front contact block; rails + chamfer ensure the pack self-aligns in the last 10 mm.

### 2.2 Retention + contact design

| Item | Design |
|---|---|
| Retention | Spring-loaded front latch + **drill-style push-guide** with audible click; optional magnet pocket for final positive stop (shear-only, withdraw-proof on drop — verify by drop test, else remove magnet; latch hold ≥ 8 N re-test) |
| Hot-swap contacts | Rear block = **staggered-length spring pins** (GND→ID/NTC→P/GND) so OR-ing sees ground first, then power, with pre-charge on sense (electrical detail → Phase 6, RISK-004) |
| Contact type | Gold-plated spring pins, ≥ 3 A continuous per pin, ≥ 2 pins in parallel on the main rail; **TBD** vendor (Mill-Max 0964-class or bespoke CNC) |
| Sense/balancing | Dedicated low-current Kelvin sense pins |
| DB front-face position | Contacts co-mate **on insertion** (last ~6 mm travel) — no separate plug action (DEC-060) |

### 2.3 Insertion direction & double-check

1. Pack inserts **front-to-rear (X+)**; key engages within first 10 mm (cannot be backwards).
2. Green/amber "seated" LED on the deck (sense-pin wired) + latch click; co-mate verified by contact wipe.

---

## 3. Swapping / replacement procedures

### 3.1 Battery packs (hot-swap, both removable while powered — REQ-PWR-03, DEC-060)
1. Sub-group C tools only (front bezel + latch). Release latch, **pull pack straight out along X−** on the spine rails.
2. Insert new pack front-first on the rails, push to the positive stop; polarity key prevents reversal; hear/feel latch + LED go/no-go.
3. No deck removal, no shutdown. **Process verified Phase-8 bench** (inrush/throttling, RISK-004).

### 3.2 NVMe M.2 2280 (REQ-COMP-03, DEC-033)
1. Open lid + top deck (G4/E3 screws from bottom).
2. Remove thermal tower (fan + HS, D3) for access if needed.
3. **Unclip/unscrew M.2 M-key socket; remove standoff screw M2.** Slide drive out across the carrier.
4. Replace, re-torque M2 standoff **0.4 N·m (TBD)**; re-seat FPC/fan; reassemble.
5. Silkscreen/keying verify (M-key only, "NVMe 2280 ONLY", DEC-033/A-027).

### 3.3 SoM (Verdin i.MX 8M Plus, B2B 260-pin)
1. Deck open + tower off (3.2.1–3.2.2).
2. Release the B2B lock levers on both sides (0.5 mm SODIMM-family); lift module straight up along Z off the vertical carrier.
3. Inspect socket; replace; press to seat, re-lock.
4. Re-apply thermal pad/HS/fan; cross-tighten carrier standoff screws evenly.

### 3.4 LTE tiny-LGA (DEC-058) — rework-class, not swappable by hand
1. Deck open + tower off; **reflow-remove** the tiny-LGA (≤ ~30 mm, ESP32-WROOM footprint-class) on the vertical carrier with hot-air (temperature profile TBD by solder alloy).
2. Clean pads; apply fresh paste; place + reflow; inspect joints (X-ray if available).
3. Re-assemble tower/deck. Note: alt simpler path = socketed LGA option if available; **TBD**.

### 3.5 Display / panel (plastic lid-top, DEC-055)
- Open lid; remove bezel (plastic frame); unseat panel; disconnect DSI FPC + BL at the carrier end (flex first). Antennas on the plastic lid-top stay in place unless damaged.

---

## 4. Lid hinge + friction torque (DEC-054; ThinkPad-class)

### 4.1 Spec & rationale
- Two **multi-point friction hinges**, **~1 N·m-class per side** nominal (0.6–1.0 N·m), barrels z 42–50 at Y 8–34 / 166–192; **reinforced mounts**: base side ≥ 3 fasteners/side into rear-wall + spine bosses with threaded inserts (≥ 4 mm engagement); lid side into rim doubler plates (≥ 4 screws/side).
- **Holding requirement:** lid ≈ 0.25–0.30 kg, CoM arm ≈ 60 mm ⇒ T_hold ≈ 0.082 N·m per side; spec friction 6–12× hold for carry-by-screen + hold at any angle.
- **One-finger-open / no-base-lift ceiling:** ≈ 0.42 N·m/side. ⇒ **Design nominal ~0.8 N·m/side, adjustable** (set-screw) to trim between carry-hold and no-lift on the desk. ≥ 15k cycles, ≤ 20 % tolerance. **TBD vendor + torque curve** (candidates B-022 replaced by TP-class spec).

### 4.2 Installation (TP-practice)
- Align barrel axis parallel to rear rim; equalize torque per side (mismatch = lid twist). Set-screw torque **TBD** (order: hinge loose → route flex loop → snap lid → set friction → verify 45–135° hold). Final: verify one-finger open, no base lift, carry by screen on bench.

---

## 5. Sealing, keyboard drainage + rain test (DEC-056)

### 5.1 Keyboard drainage (T480-style weep)
- Membrane sits over molded drain troughs in the deck; water entering key wells drains to **bottom-edge weep slots** (≥ 12 slots, ≥ 2° slope, one-way flaps) clear of bays and rear ports. Verify gutter continuity at assembly (E4).

### 5.2 Sealed vents / ports
| Opening | Seal | Notes |
|---|---|---|
| Palm-rest grill (fan intake) | louver + mesh + replaceable filter pocket + low-point drain weep | intake side |
| Side/rear exhaust | louver bank + drain weep | outlet, low pressure |
| Lid-to-deck closure | continuous closed-cell gasket | keep flex radius ≥ 10 mm |
| Hinge FPC/coax channel | foam ring + soft bend-cap | dynamic loop |
| Rear connector cut-outs | conductive EMI gasket + IP bead | spatter/IP4X |
| Pogo port | silicone plug / magnetic cap (signal+GND default) | REQ-UART-01 |
| Battery bay mouths | soft lip + front bezel cover over latches | no water to contacts |
| Overall target | rainproof **hours**, IP4X splash+dust (not submersible) | **TBD bench** |

### 5.3 Drain / rain test procedure (bench)
1. **Drip test:** 2 L water over the top deck + keyboard area, 2 mm nozzle, 20 min; verify no water reaches carrier/daughterboard; all water exits weep slots/vent drains.
2. **Row-wise cup test:** pour 100 mL into each key row gutter inlet; verify ~90 % exits within 60 s (T480-class target), the remainder trapped in drain tray.
3. **Vent drain check:** spray palm-rest grill + exhaust louvers; confirm low-point drains clear before fan duct.
4. **Closure seal test:** lid closed, 5 min continuous spray on the seam; no ingress beyond gasket.
5. Log results; iterate weep geometry if > 10 % retained (**TBD** targets).

---

## 6. Keyboard deck mounting (REQ-KB-01, DEC-021/025/045)

- **Top deck:** 3 mm anodized Al; front ~50 % holds the 6-row membrane (US legends + ISO-DE Enter, ~12 mm pitch) + trackball well between G-H-B (DEC-025); rear palm-rest carries the grill over the thermal tower.
- Keyboard **PCB is vertical** (own board, plane across Y, z 29–39) on the spine plate; membrane flat-flex + trackball sensor pigtail (ADNS-9800, RISK-020 ball-size caveat — **must test-mount first**; keep well as interchangeable insert) connect into it; STM32G0 deck MCU scans → USB-HID → carrier (DEC-045).
- Mounting: membrane adhesive, deck via shoulder screws from underside; net key travel ~5.5 mm below lid — verify against lid underside (gasket). Drain gutters under the membrane are part of the deck (E4).

---

## 7. ESD / grounding with the plastic lid-top (DEC-055)

- **Grounding budget (A2 masking):** bare-Al zones at battery contact pad areas (spine rails + rear block bosses), connector screw bosses, hinge lugs, pogo port frame, grill/vent frame edges, ground standoffs (carrier/daughterboard D-columns), SIM/antenna grounds, rim doubler bosses.
- **The aluminum rims/edges are the reference:** ground the metal rim continuously (rims are structure + grounding ring); the **plastic lid-top is dielectric**, so antennas behind it are fine (no RF window needed behind the panel) — antenna grounds return through coax shields to the carrier RF GND, which stitches to the chassis star.
- **Ground strategy:** chassis-star ground at the daughterboard; battery-rail GND ties via bay contact blocks; each board ground plane stitched to its mounting bosses with ≥ 2 screws; FPC/cable shields terminated to chassis at both ends; hinge barrels equalize lid rim ↔ base via finger-spring/conductive tape.
- **ESD test note:** ±8 kV air/contact discharge to all user-accessible metal (keys, trackball, edges, connector shells, rim) with deck grounded in PD/AC mode — Phase-8.
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
| Drain/rain test | drip rig + flow meter + 100 mL cup (per §5.3) |
| Assembly | ESD mat, wrist strap, M2.5 tap for dry-fit thread cutting |

---

## 9. Safety

1. **Sharp edges:** every user-handled edge (deck, keys, bays, lid rims, front bezel) gets a **0.3–0.5 mm chamfer/fillet**; no 90° knife edges on palm rest or hinge zone; thin rubber feet/edge tapes on lid rims (TBD).
2. **Battery polarity / latch anti-pop (REQ-PWR-05):**
   - Asymmetric spine-rail key prevents reverse insertion (§2/DEC-060).
   - Latch must hold pack against contact wipe (≥ 8 N) under vibration — re-tested with v0 prototype (RISK-008/023).
   - Each pack BMS has reverse-polarity protection + polyfuse; daughterboard OR-ing is ideal-diode (no back-feed, RISK-004). Never assemble damaged-wrap cells.
3. **Battery transport/UN38.3:** custom packs — carry path/declaration per RISK-008/023; SoC ≤ 30 % before flight.
4. **Live-deck opening:** remove power before any deck/board work **except** the hot-swap battery step (design + bench-verified).
5. **High-current PD tray:** PD 20 V/65 W isolated + fuse-protected, connector-recessed; electrical review gate (REQ-PWR-05) before first power.
6. **Thermal tower:** fan + HS are live-adjacent to SoM; torque HS mount evenly; keep fingers clear of blade; redirect airflow away from skin-contact surfaces (palm-rest grill ≥ 60 °C-surface design review TBD).

---

## 10. Open mechanical decisions (service side)

| # | Decision | Options | Blocks |
|---|---|---|---|
| S-1 | Contact block vendor/type (drill-rail rear block) | Mill-Max 0964-class / CNC leaf / pogo (verify) | bay + daughterboard |
| S-2 | Drill-rail latch type + polarity key profile | slide vs cam vs push-guide; T-rail key geometry | bay retention |
| S-3 | Hinge part + torque trim (0.6 vs 0.8 vs 1.0 N·m/side) | adjustable set-screw; lid-flex risk | lid build |
| S-4 | Top-deck thickness with grill + drain troughs (3 vs 2.5 mm) | stiffness vs mass/CoM | deck CNC |
| S-5 | Pogo port seal (cap vs plug) + level-shift enable | strictly signal+GND until electrical review | UART port |
| S-6 | LTE LGA socketed vs reflow-only (affects swappability) | land-pad vs socket (DEC-058) | swap service path |
| S-7 | Vent drain geometry + grill mesh pitch | louver 0.5 vs 0.8 mm; drain location | rain test |

All **TBD/unverified** items must be closed by Phase-8 measurement (mass, contact wipe, rail slide force, hinge torque, water retention, ESD).