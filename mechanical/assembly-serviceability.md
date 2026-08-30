# Mechanical — Assembly and Serviceability (Phase 6)

Status: **Phase-6 mechanical design draft** — 2026-08-30. All torque/fastener values are typical CNC-aluminum practice; each **TBD** item must be confirmed against the purchased parts before build. Mass/CoM cross-references: `envelope-and-stack.md` §6. Battery packs and hot-swap electronics are safety-critical (REQ-PWR-05) — this file stays in sync with the electrical design (`electric/…` Phase 6, TBD) and the risk register (RISK-002/004/008/023).

---

## 1. Assembly order (bottom-up, base first, lid last)

Chassis approached exactly as a normal clamshell laptop (DEC-002). Base assembly proceeds in **5 sub-groups**; each line item is one physical step with its own fastener/torque set. Screws come from the **base/bottom** (serviceability requirement) except where noted.

### Sub-group A — chassis & bay prep
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| A1 | Inspect + deburr CNC base, deck, mid-deck, lid. Verify RF windows/vents pass. | base, deck, mid-deck, lid (6061-T6) | manual, no screws |
| A2 | Mask anodize-off contact points (see §6 ESD) — tape before anodize | all Al parts | — |
| A3 | Fit bay slide rails (POM/POM-C or Al rails + 0.2 mm surface), pop-rivet/glue in place | 4 rails + 1 center divider | M2.5 × 5 button + thread-lock (LT-243) or structural adhesive; torque ~0.35 N·m (TBD) |
| A4 | Install rear-wall connector openings check (dry fit daughterboard) | daughterboard | — |

### Sub-group B — daughterboard (rear vertical board)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| B1 | Screw daughterboard onto rear wall bosses (D-columns) | daughterboard | M2 × 4 c/s steel, 0.5 N·m (TBD) |
| B2 | Fit side/center board-to-wall standoffs | standoffs | M2 × 4 |
| B3 | Cable the pogo module + AUX + SD + USB-C PD to board edges | — | — |
| B4 | Apply ESD/lid-FPC gasket to rear wall slot | foam gasket | — |

### Sub-group C — battery bay + contacts (front)
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| C1 | Install rear battery contact blocks (spring pogo/leaf array, staggered pin lengths) on daughterboard front face or on dedicated front bracket | contact block ×2 | M2 × 4, 0.5 N·m |
| C2 | Fit bay latch + magnet (optional retention) at front bezel | latch/cam + magnet pocket | M2 × 4 |
| C3 | Dry-slide a reference pack both bays (entry force, latch, contact wipe check) | 2 ref packs + 2 armed packs | — |

### Sub-group D — carrier mid-deck (top side) + thermal
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| D1 | Assemble carrier PCB: SoM-to-B2B socket, M.2 2280 (M-key NVMe), LTE mPCIe, Wi-Fi pad; keying + silkscreen check (DEC-033) | carrier + SoM + NVMe + LTE + Wi-Fi | — |
| D2 | Fit heat-sink (5–6 mm finned Al, gap pad to SoM) + Sunon fan beside it | heat-sink, fan | M2 × 4 + thermal pad |
| D3 | Route DSI FPC + 3 u.FL antenna coax + carrier↔daughterboard control FPC into hinge slot/rear gasket | FPCs, coax | — |
| D4 | Screw mid-deck (carrier plane) onto base bosses at ~31 mm z-level; verify pack clearance under it | mid-deck | M2.5 × 6, 0.45 N·m (TBD) |

### Sub-group E — deck, lid, closure
| Step | Operation | Part(s) | Fastener / torque |
|---|---|---|---|
| E1 | Fit keyboard membrane + trackball well + metal key guides into deck cutouts; route to carrier I²C/USB | deck, membrane, trackball | — |
| E2 | Close lid assembly: panel (+ glass, TBD) + driver board + FPC exit; door hinge attach | lid, hinges ×2 | hinge M2.5, 0.5 N·m |
| E3 | Attach lid to base via friction hinges (torque set in §4) | hinges | hinge M2.5; set screw torque per §4 |
| E4 | Final: close deck screws (base underside cover screws port-facing), run power-on test + thermal smoke + lid-hold torque check | — | M2.5 countersunk, 0.5 N·m (TBD) |

**Service access summary (screws from base):** remove **lid** (2 hinges, 4 screws + display flex) → remove **deck** (screws on bottom: sub-group E4 cover + deck) → remove **mid-deck** (D4) → carrier/m.2/soM accessible; remove **front bezel/bays** to access battery contacts and back of daughterboard. Battery packs need **only the front bezel/latch**, no deck removal (hot-swap, REQ-PWR-03).

---

## 2. Battery bay design (front insertion, DEC-018; hot-swap REQ-PWR-03)

### 2.1 Bay layout
- Two bays, each 70–72 (Y) × 102–107 (X) × 27 (Z-in) of clearance, cells at ~3 mm above floor.
- **Slide rails**: 4 rails (2/bay), POM or anodized Al, ±0.2–0.3 mm fit; entry chamfer on the pack shell ~5° so a pack cannot be mis-landed.

### 2.2 Retention + contact design
- **Retention**: spring-loaded **front latch** with a visible latch slot at the bezel; optional small magnet pocket for positive final stop + tactile "click" (magnet force ~0.5–1 N in shear only, sized so it never dislodges the pack during drop-shock — verify with drop test, else remove magnet).
- **Hot-swap plac/contact**: rear contact block = **staggered-length spring pins** (ground > power > sense/balance) so the battery-rail OR-ing sees ground first, then power, with a short pre-charge on the sense pin (electrical detail → Phase 6 electric, RISK-004).
- **Polarity/keying**: asymmetric rail/groove key (a "T" rail geometry) so a pack can only be inserted one way; electrical reverse-polarity diode + polyfuse on each pack BMS (pack-side).
- **Contact type**: pogo/leaf spring pair, **gold-plated**, ≥ 3 A continuous per signal, ≥ 2 contacts in parallel on the main rail. **TBD** exact vendor part (Mill-Max 0964-class spring pins, or bespoke CNC contacts).
- **Pack sense/balancing**: dedicated low-current sense pins so current-sense is Kelvin (no voltage drop error).
- **Anti-pop (see §9 safety)**: latch must keep the pack seated while the lid is open and under vibration; verify latch hold ≥ 8 N (typical) retest.

### 2.3 Insertion direction & double-check
Packs insert **front-to-rear (X+)** until the contacts wipe onto the daughterboard block. Two mechanical go/no-go checks on the dock:
1. Insertion key engages within the first 10 mm (can't be backwards).
2. A green/amber "seated" feedback (LED on the deck, wired to sense pin).

---

## 3. Swapping / replacement procedures

### 3.1 Battery packs (hot-swap, both removable while powered — REQ-PWR-03)
1. Sub-group C tools only (front bezel). Release latch (slide/finger), **pull pack straight out along X−**.
2. Insert new pack front-first, push to positive stop; hear/feel latch.
3. No deck removal, no shutdown. **Process verified in Phase-8 bench test** (inrush/throttling, RISK-004).

### 3.2 NVMe M.2 2280 (REQ-COMP-03, DEC-033)
1. Open lid + deck (4 screws + deck screws).
2. Remove mid-deck (D4 screws) and, if needed, heat-sink/fan (D2).
3. **Unclip/unscrew M.2 M-key socket; remove standoff screw M2 (1 pc).** Slide drive out on Y-axis (carrier layout §4.2).
4. Replace, re-torque M2 standoff **0.4 N·m (TBD)**; re-seat FPC/antenna; reassemble.
5. Silkscreen/keying verify (M-key only — a B+M SATA card must not physically fit, A-027/DEC-033).

### 3.3 SoM (Verdin i.MX 8M Plus, B2B 260-pin)
1. Steps 3.2.1–3.2.3, then **release the B2B lock levers** on both sides (0.5 mm pitch SODIMM-family socket).
2. Lift SoM card straight up along Z. Inspect socket; replace; press to seated, re-lock.
3. Re-apply thermal pad/heat-sink, reassemble. **Torque all M2.5 carrier screws evenly cross-tightened.**

### 3.4 LTE modem + SIM, Wi-Fi module
- LTE: remove mid-deck; unclip mPCIe card; anti-static; press-in; **insert nano-SIM into on-carrier slot** before seating.
- Wi-Fi: RTL8821CU pad module, unplug 1–2 u.FL (mark the two antenna feeds), re-plug accurately (crossed feeds = weak radio, RISK-021).

### 3.5 Display / panel
- Vu8S-class panel — see **open decision M-2 in `envelope-and-stack.md` §8.1**: current panel does not fit the envelope; serviceability of whichever display is chosen is part of the lid build.

---

## 4. Lid hinge + friction torque spec (DEC-002, Hinge zone in envelope doc §7)

### 4.1 Spec & rationale
- Two 30–40 mm friction hinges (candidates B-022), one per side, near X ≈ 125, z ≈ 40–50.
- **Required holding torque per hinge** ≈ _lid-weight × lever-arm / 2 hinges_ :
  - Lid mass ≈ 0.3 kg (display+glass/frame TBD), CoM arm ≈ 60 mm ⇒ T ≈ 0.3 × 9.81 × 0.06 ≈ **0.18 N·m** per hinge.
- **Design friction torque = 3–4× required** ≈ **0.6–0.8 N·m per hinge** (allows 45–135° hold + small portability bounce); spec vendor must state torque curve ≈ flat 0.6–0.8 N·m with ≤ 20 % tolerance, life ≥ 15k cycles.
- Laptop friction-hinge practice for a 1.6–1.9 kg deck: **30–40 mm hinge length, 3–5 kgf·cm (≈0.3–0.5 N·m) per hinge** is the common commercial band (AliExpress/stock, candidates B-022) — this lands inside our range at the high end; **rationale: short lid (small CoM arm) needs less than a 13–15" laptop; overshoot risks flexing the lid** — prefer the lower end (≈0.5 N·m/hinge) and add final hold-angle bench test.
- **Setting method**: hinge barrels carry a small set-screw or the friction is factory-set; if adjustable, adjust per side equally (mismatch causes lid twist). **TBD** exact hinge part & adjustment — verify before ordering (B-022 is a generic listing).

### 4.2 Cable loop through the hinge
- DSI FPC + antenna coax must pass a **≥ 10 mm dynamic radius loop** in the lid channel and a **soft bend-cap right at the hinge** (`envelope-and-stack.md` §5). Serviceability: the hinge must come off before pulling the FPC (order in E3 always: FPC out first, then hinge screws).

---

## 5. Keyboard deck mounting (REQ-KB-01, DEC-021/025)

- **Top deck**: 3 mm anodized Al; keywells CNC-machined (free CNC, DEC-023). **6 rows × ~12 mm pitch** (~72 mm keybody block + edges). **Trackball well below space bar, ~20 mm** recess between G/H/B (DEC-025).
- Membrane (custom, 6-row, US labels + ISO-DE Enter) sits on a 0.8–1 mm PET spacer on the deck; steel key guides (or precisely machined Al wells, depending on CNC tolerance) snap through.
- Trackball: small (~15–21 mm) recessed pool in the deck, sensor under it (ADNS-9800, **ball-size geometric-error risk RISK-020 — must test-mount first**; keep the well diameter a separate interchangeable insert so a different ball/sensor can drop in without re-cutting the deck).
- Mounting: membrane→deck is adhesive; deck→base via shoulder screws from underside so keys are not compressed. Net stack: deck 3 + membrane ~1.5 + key gap ≤ 1 = **~5.5 mm below lid when closed** — check against lid underside (gasket) — TBD verify with chosen keycaps; a palm-rest/hand hood region between last row and hinge must remain smooth (type comfort, few fasteners there).

---

## 6. Sealing (chip-entry path, vents, pogo port)

| Opening | Seal/approach | Notes |
|---|---|---|
| Rear wall vents | Slot array with 0.5 mm louver geometry (turns airflow, keeps chips out); optional fine mesh behind | exhaust side (cooler, low pressure) |
| Side (fan intake) | **Mesh + filter paper pocket** (replaceable) | main dust entry |
| Bottom vent cells | TBD — small louver slits, avoid under-pack water path | ingress-aware |
| Hinge FPC slot | Closed-cell foam gasket | keep bend ≥10 mm |
| Rear wall connector cut-outs | Conductive EMI gasket + light IP40 bead | keeps EMC + spatter |
| **Pogo port** | **Out-of-box the pogo port is `signal + GND only` (REQ-UART-01 safe default); seal with a silicone/foam plug or magnetic cap-washer behind a small rear slot; level-shift/protection circuit added only after electrical review** | Do not carry configurable 3.3/5 V until reviewed |
| Overall | Target **IP4X splash + dust** (not submersible) | TBD bench |

---

## 7. ESD / grounding (aluminum chassis as reference plane)

- **Anodize masking plan** (applied at A2, per anodizer's masking method): leave **bare-Al zones** at — battery contact pad areas, connector screw bosses, hinge attachment lugs, pogo port frame, vent-frame edges, ground standoffs (D-columns), mid-deck boss tops, SIM/antenna grounds. This is the **grounding budget that makes the chassis a real zero-ohm reference** (DEC-016/037; anodize is an insulator).
- **Ground strategy**: single **chassis-star ground** near the daughterboard; battery-rail GND ties via the bay contact blocks; each module ground plane stitched to its mounting boss with ≥ 2 screws; FPC/cable shields terminated to chassis at both ends.
- Add **finger-spring gaskets** at deck-to-base seam and around rear connectors; **conductive tape ratio** at hinge barrels to keep lid/chassis equalized.
- **ESD test note**: niCr/air discharge ±8 kV to all user-accessible metal (deck keys, trackball, edges, connector shells) with the device grounded in PD/AC mode — Phase-8; resp. second milestone — measure at +0 V chassis reference.
- **Tools/kit** for all swaps: grounded mat + wrist strap; never plug/unplug packs while bench not grounded.

---

## 8. Tool list

| Purpose | Tool |
|---|---|
| All body screws | **M2.5 torx T6 / Phillips #0** (pick per joint, standardize to torx) |
| M.2 / SoM fasteners | **M2 hex or PH0** + small torque driver (0.1–0.6 N·m) |
| Hinge set screws | T6/T8 + torque driver (per §4.1 band) |
| Bay latch / contacts | PH0 + non-marring prying tool |
| Displays/FPCs | nylon spudger, ESD-tweezers, anti-snag hook |
| Thermal | good-quality thermal pads + cleaning wipes + a small torque driver with ±10 % accuracy for CPU-heat-sink reuse |
| Assembly | ESD mat, wrist strap, M2.5 tap for any thread-cutting during dry fit |

---

## 9. Safety

1. **Sharp edges** (a classic for CNC boxes): every user-handled edge (deck, keys, bays, lid, front bezel) gets a **0.3–0.5 mm chamfer / fillet**; no 90° knife edges on the palm rest or hinge. Add thin rubber feet edge-protect tapes on lid edges (TBD).
2. **Battery polarity / latch anti-pop (REQ-PWR-05)**:
   - Asymmetric keyed rails prevent wrong-orientation insertion (§2.2).
   - Latch must hold the pack against the WIPE (≥ 8 N) so the contacts cannot "pop" open under vibration — re-tested with v0 prototype (RISK-008/023).
   - Each pack BMS has reverse-polarity protection + polyfuse on the rail; daughterboard OR-ing is ideal-diode (no back-feed between packs, RISK-004). Never assemble with cells whose wrapping is damaged.
3. **Battery transport/UN38.3**: packs are custom-built; carry path/declaration per RISK-008/023 — mark SoC ≤ 30 % prior to any flight.
4. **Live-deck opening**: remove power before any deck/mid-deck work except the hot-swap battery step (that one is designed & bench-verified).
5. **High-current PD tray**: keep PD 20 V routing isolated and fuse-protected; connector-recessed; electrical review gate (REQ-PWR-05) before first power.

---

## 10. Open mechanical decisions (service side)

| # | Decision | Options | Blocks |
|---|---|---|---|
| S-1 | Contact block vendor/type (pogo vs leaf, staggered-length) | Mill-Max 0964-class / CNC leaf / AliExpress 2.54 mm magnetic (verify first) | bay + daughterboard |
| S-2 | Latch type (slide vs cam vs push-push + magnet) | magnet shear force sized; drop-test | bay retention |
| S-3 | Hinge part + torque set (0.5 vs 0.8 N·m/hinge) | adjust per hinge; note lid-flex risk | lid build |
| S-4 | Top-deck material thickness (3 vs 2.5 mm) with keywells | stiffness vs mass/CoM | deck CNC |
| S-5 | Pogo port seal (cap vs plug) + level-shift enable | strictly signal+GND until electrical review | UART port |

All **TBD/unverified** items must be closed by Phase-8 measurement (mass, contact wipe, torque, ESD).