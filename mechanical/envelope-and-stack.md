# Mechanical — Envelope and Z-Stack (Phase 6)

Status: **Phase-6 mechanical design draft** — 2026-08-30. All dimensions are CAD-input planning values derived from the approved baseline (DEC-001/002/003/016/018/023/033/035/037) and Phase-4 part evidence. Figures marked **TBD** are unverified and must be confirmed by part datasheets or measurement (Phase 8) before CNC quote.

> ⚠️ **Envelope change (2026-08-30, DEC-043):** envelope widened to **≈200 × 140 × 50 mm (≤55 w/ 7 mm HS)**, lid bezels **3 mm top/left/right + 10 mm bottom**, to fit the **Raystar RFU800G-AYH-MNN** panel (OD 115.74×184.93×4.75 mm, DEC-044). Interior now ≈ 194×134 with 3 mm walls. Manual XY layouts below based on the old 124×164 interior are **superseded** in dimension but the *z-stack-over-battery-tunnels resolve* (ADO §4.3) and the battery derivation (§2) are unchanged. Re-derive exact XY on the new interior before CNC.

> Convention used throughout: **X** = deck depth (front = 0, rear = 124/(130), mm), **Y** = deck width (left = 0, right = 164/(170)), **Z** = height above base floor (0 at inside of bottom plate). Naming per `info/terminology.md` where applicable.

---

## 1. External envelope verification (REQ-ENC-01)

| Dimension | Approved (REQ-ENC-01) | Design target | Interior (3 mm wall) | Verdict |
|---|---|---|---|---|
| Depth (X) | ≤ 130 | 130 | 124 | ✅ at limit |
| Width (Y) | ≤ 170 | 170 | 164 | ✅ at limit |
| Height (Z, closed) | ≤ 50 | 50 | 44 (= 50 − 3 mtl − 3 lid/glass budget ⇒ base interior ≈ 44) | ✅ (lid split shown in §3) |

Notes:
- The 130×170 plan is **at the envelope limit**; the 3 mm wall leaves only **124×164 mm** usable. No design margin exists for a 2 mm wider part; tolerances must be held with pass/fit sketches before CNC (RISK-002).
- Interior **height** is budgeted in §3 — the classic clamshell split (base + lid) does **not** leave 46 mm for the base; feasibility's "≈46 mm interior" was a volume-bookkeeping figure ($feasibility §1), not a z-stack budget. Practical usable base interior ≈ **41–44 mm** depending on lid/glass thickness (open decision, §8).

---

## 2. Battery footprint derivation (REQ-PWR-01/03, DEC-003/018)

### 2.1 Cell geometry (evidence: candidates.md §1, P50B datasheet)

- Molicel INR21700-P50B: **Ø max 21.55 mm × length max 70.15 mm**, 71 g, 18.0 Wh.
- Pack = **4S1P**, cells inline along the depth (X): 4 × 21.55 = **86.2 mm** of cell body.

### 2.2 Derived pack slab (corrected user estimate)

| Item | Value | Basis |
|---|---|---|
| Cells inline (X) | 86.2 mm | 4 × 21.55 |
| End housing: BMS PCB + terminal plate + contact sockets/pogo + label/bumper | ~14–19 mm | DEC-003 slab "105", feasibility §2 |
| **Pack footprint** | **≈ 105 × 70 × 24** (X×Y×Z) | user-specified slab, confirmed by cell math |
| Pack body (cells only, prior to end cap) | ≈ 90 × 72 × 24 | 86.2 + rails/grip; Y = 70.15 cell + ~2 housing |
| Two packs across width | **2 × ~72 = 144–150** + center divider/side rails (~6) | ≤ 164 interior ✅ (≤ 170 envelope with 2×3 walls) |
| Battery bay tile | ≈ 70 (Y) × 102–107 (X) × 27 (Z-in) each, 2 side-by-side | §5 |

**Key derivation:** pack "width" (70) is the **cell length** on edge; pack "depth" (105) is **4 cell diameters + end cap**. The user's "2×~84 cell-width across 170" reading ($feasibility §2: "2×70 + ~16 + 2×3 ≈ 168") is the same number expressed about 80 mm — cell length 70.15 → across-width total ≈ 2×70.15 + ~16 internals + 6 walls ≈ **162 < 170** ✅ (not 168; both fit).

Volume/energy: 2 × 4 × P50B = **144 Wh** at ~410 Wh/L pack class (feasibility §2) — REQ-PWR-01 satisfied. Mass: 8 × 71 g = **568 g** raw cells (§6).

---

## 3. Z-stack (flamingo: floor → top deck → lid)

Base interior 124 × 164 × ~44 (with 4 mm lid+glass allowance over deck; see §8).

### 3.1 Stack budget (bottom → top)

| # | Layer | Thickness (mm) | Notes |
|---|---|---|---|
| Z1 | Bottom plate (6061-T6) | 3 | machined floor, screw bosses, vent valleys |
| Z2 | Battery bay floor level + pack (front region) | pack 24 + guides 2 + slide 1 | cells z≈3–27 |
| Z3 | Air plenum / FPC / cable channel above pack | ~3 | under mid-deck |
| Z4.1 | **Daughterboard** (rear, vertical wall board) | board ~1.6 + components | z≈4–30 |
| Z4.2 | **Carrier** (mid-deck, aft over tunnels) | PCB 1.6 + SoM 9–13 + HS 5–7 | **board top ≈ z 31–33, SoM top ≈ z 41–46, HS top ≈ 47–53** (→ gated in §3.2) |
| Z4.3 | **Fan** 30×30×10 | 10 | beside SoM/heatsink, z≈33–43 |
| Z5 | Top deck (Al, keyboard/trackball/bay bezel) | 3 | underside ≈ z 49 |
| Z6 | Display lid stack (closed): panel + glass + driver/FPC cavity | ~4 (TBD glass) | closes at z 50 |

Sum: 3 + 27 + 3 + (13–19 HS top) + 3 + 4 ≈ **~50** boundary — **budget is at the limit**, see gating.

### 3.2 Constraint check (the binding math)

Required HS top: `= deck_underside − SoM_over_carrier − board − tunnel_ceiling`
`= 49 − (9…13) − 1.6 − 31 = 3.4 … 7.4 mm`

→ A **5–6 mm** finned heat-sink fits (DEC-017 5–7); **7 mm needs the SoM B2B at ≤ ~10 mm over-carrier and a 3 mm lid**, else a deck "hump" recess or drop to 4 mm (open decision §8). Measure actual Verdin+carrier stack height before locking ($candidates.md §2: module 6.0 mm + socket ~9–13 mm over carrier — **TBD measurement**).

### 3.3 Alternative (plan-split) z-reasons it fails — see §4.3

---

## 4. XY plan layouts (base, plan view, front at top)

### 4.1 Battery bays (front, DEC-018)

- Bay A: X 3–105,  Y 8–78
- Bay B: X 3–105,  Y 86–156
- Center divider + side rails: Y 78–86 channel + ~4 wall slivers — total bays ≈ 144–152 of 164 ✅
- Pack insertion: **from front**, slides aft on low-friction rails; rear contact block (leaf/pogo) mates with daughterboard front face at X ≈ 105. Latch at front bezel.

### 4.2 Carrier (mid-deck, aft) + daughterboard (rear, vertical)

- **Daughterboard** — vertical board against rear wall, X 112–124, Y 30–134, height ~4–30. Carries: USB-C PD (65 W), HDMI, USB3-A, USB2-OTG, RJ45, full-size SD, 3.5 mm AUX, magnetic pogo (REQ-UART-01), power-manager MCU, battery contact strips on its front face (X ≈ 105–112), LED strip. Connectors exit the rear wall (X=124/R=130).
- **Carrier** — horizontal mid-deck board, X 62–124, Y 24–140 (capacity: SoM 69.6×35 @ Y 60–130; M.2 2280 22×80 @ Y 60–140 laid along Y using the 80 mm width dimension; LTE mPCIe 51×30; WiFi 12×20-pad; PCIe/DSI/USB/SD splits), z ≈ 31–47 with heat-sink.
  - **SoM** (Toradex Verdin, 69.6×35×6) on B2B (260-pin 0.5 mm) → +9–13 over carrier, **TBD measured**.
  - **M.2 2280 NVMe M-key**: 22.0 × 80.0 mm, PCB 0.8 ±10%, single-side D2 ≤2.38 / double-sided D3 ≤3.7 (Wikipedia M.2, 2026-08-30). Mount: M2×0.4 + standoff ≈ 3 mm (hole Ø2.5, nominal 76 mm from connector datum — **confirm to SATA-IO spec**) + socket; keying/silkscreen blocking per DEC-033/A-027.
  - **LTE** Quectel EC25-EUX mPCIe 51×30×4.9 + nano-SIM (on carrier) + 2 antenna feeds.
  - **WiFi/BT** RTL8821CU USB pad module + aperture antenna at top-deck RF window.
  - **Fan** Sunon 30×30×10 at front of heat-sink blowing aft → rear vents (DEC-017/028).

### 4.3 Why the strict plan-view split (front battery / mid carrier / rear daughterboard) does not fit

| Zone | Min depth (X, mm) |
|---|---|
| Battery bay (pack 105 + finger space) | ≥ 107 |
| Carrier (35 SoM + M.2 socket zone + edges) | ≥ 45 |
| Daughterboard (vertical strip) | ≥ 12 |
| **Sum** vs interior 124 | 107+45+12 = **164 > 124** ✗ |

→ An **in-plane aft stack is infeasible** under REQ-ENC-01 (feasibility §2 flagged "constraint moves to z"; DEC-018 requested battery-front/rear-daughterboard in plane). **Resolution used here:** carrier sits on the **aft mid-deck over the battery tunnels** (z-stacked), preserving front hot-swap; daughterboard stays vertical-rear. This is the one arrangement that satisfies the envelope; it is a **deviation from the strict plan-view reading of DEC-018 and must be accepted by user (open decision §8)**.

---

## 5. Clearances, cables, connectors, bend radii

| Routing | Spec / allowance | Note |
|---|---|---|
| MIPI-DSI FPC to Vu8S (RISK-024) | 0.3–0.5 mm pitch FFC; **bend radius ≥ 5–6× thickness (≥ 1.5–2 mm)**, dynamic-hinge loop ≥ 10 mm in lid channel | custom FFC + ODROID-class connector, **TBD pinout** |
| USB3 / PCIe (on carrier, never across gap — RISK-015) | 2 mm strip-to-copper clearance min; 90 Ω diff to connector | carrier-local |
| Inter-board control FPC (carrier↔daughterboard) | 0.5 mm FFC, I2C/UART/PWM/LED/SIM only, short die ≤ 80 mm, bend ≥3 mm | low-speed only (RISK-015 mitigation) |
| Antenna coax (u.FL 1.13 mm) ×3 | bend radius ≥ 3 mm; keep ≥ 10 mm from heat-sink/fan metal | Wi-Fi→top deck, LTE→rear aperture |
| Battery interconnect (pack→daughterboard) | spring contacts, ≥ 3 mm separation between terminals; 14.4 V rail, 2–3 A expected | polarity-keyed, staggered-length pins (ground first) |
| PD 20 V (daughterboard) | ≥ 1 mm spacing to low-volt traces; connector-recessed | safety review REQ-PWR-05 |
| Fan/air duct | duct gap ≥ 2 mm walls; no pinch below 1.5 mm | noise/dust |
| Chassis-to-part keep-outs | ≥ 1.0 mm on all machined faces; ≥ 0.5 on pads/gaskets | RISK-002 margin |

---

## 6. Mass estimate and center of mass

### 6.1 Mass table (planning; feasibility §7 reconciled with DEC-033/035/038/041)

| Item | Mass (g) | X centroid (mm) |
|---|---|---|
| Battery cells 8×71 (P50B) @ bay A+B | 568 | 54 |
| Pack HW ×2 (BMS, plates, contacts, rails) | 60 | 54 |
| Carrier PCB 4–6L + SoM (Verdin 8GB) + B2B | 90–110 | 100 |
| Heat-sink 5–6 mm + Sunon fan | 40–70 | 100 |
| NVMe 2280 M.2 (user's, B-002) | 50–70 | 100 |
| WiFi/BT module + antenna | 15 | 105 |
| LTE EC25 + SIM + 2 antennas | 30–45 | 105 |
| Daughterboard + PD + connectors (REQ-IO/REQ-UART items) | 90 | 118 |
| Display Vu8S + glass/driver (closed) | 150–170 | 70 |
| Keyboard deck + membrane + trackball | 120–140 | 55 |
| Chassis (base + mid-deck + lid, 6061) | 400–550 | 65 |
| Screws/cables/misc | 100–130 | 70 |
| **Total** | **≈ 1,600–1,900** | — |

Using mid-range masses (total ≈ 1,865 g): Σ(m·x) = 128,767 g·mm ⇒ **X₍CoM, closed₎ = 69.0 mm = 53 % of base depth** ✅ within 45–55 % (feasibility §7, front-loaded battery target).

### 6.2 Tip-back check (lid open)

Hinge at X ≈ 125. Lid ≈ 160 g; when opened ~110° its centroid moves rearward toward the hinge (≈ +45 mm × 160 g ≈ +7,200 g·mm) ⇒ X₍CoM, open₎ ≈ 72.9 mm = **56 %, still well inside the footprint** (rear feet ≈ 120–127). With **both packs removed** (hot-swap event) and lid open: Σm ≈ 1,237 g, X₍CoM₎ ≈ 82 mm = 63 % — **still no tip-back** ✅. Y-CoM symmetric ≈ 85 (centered). Z-CoM ≈ 25–30 mm (low, favorable).

---

## 7. RF windows, vents, hinge zone

- **Wi-Fi/BT RF window** (RISK-013/021): top-deck aperture over carrier antenna at **X ≈ 80–95, Y ≈ 60–80**; dielectric (or slot) in deck; keep ≥ 10 mm of surrounding anodized aluminum intact as ground edge.
- **LTE RF window**: rear wall aperture at **X ≈ 124, Y ≈ 60–80, Z ≈ 30–44** opposite LTE antenna.
- **Vents**: rear X=124 slotted bank (fan exhaust) over HS/fan footprint; side vents Y=0/164 at X ≈ 70–114 (fan intake); small floor vents under battery cavities (cooling; ingress-considerate, add filter). Actual CFM/noise/pressure **TBD bench** (RISK-016).
- **Hinge zone**: rear top corners Y ≈ 10–40 / 130–160, z ≈ 40–50; hinge barrels inside 3 mm side-wall reliefs; FPC/antenna exits through a D- or R-slot in the rear wall center (Y ≈ 70–90), sealed by a foam gasket. Friction torque spec → `assembly-serviceability.md` §4.

---

## 8. Keep-out / bounding boxes (source-verified core dims)

| Component | Keep-out box (X×Y×Z mm) | Source |
|---|---|---|
| Battery pack ×2 | 105 × 70 × 24 each (bay 107 × 74 × 27) | P50B Ø21.55×70.15, candidates §1; feasibility §2 |
| Verdin i.MX8MP | 69.6 × 35.0 × 6.0 + 9–13 B2B (**TBD measured**) | candidates §2 |
| M.2 2280 NVMe (M-key) | 22 × 80 × ≤3.7 (D2 2.38) + standoff ≈3 | Wikipedia M.2, 2026-08-30 |
| Vu8S module w/ bracket | **202 × 153 × ~11 (TBD depth)** — **DOES NOT FIT lid interior 164×124 in either orientation** | hardkernel.com shop, 2026-08-30 |
| Vu8S active area (bare panel) | 172.224 × 107.64 | hardkernel.com shop, 2026-08-30 |
| Sunon fan | 30 × 30 × 10 | candidates §7 |
| EC25-EUX LTE | 51 × 30 × 4.9 (mPCIe) | candidates §5 |
| RTL8821CU pad | ~26 × 26 × 3 (TBD) | candidates §6 |
| Daughterboard (rear wall) | 12(d) × 104(w) × ~26(h) | layout §4.2 |

### 8.1 Display fit finding (drives an open decision)

- Vu8S **with bracket is 202 × 153 mm** — larger than the lid interior (164 × 124) in **both** orientations → **cannot be used as a bracketed module**.
- Bare active area 172.2 × 107.6 (16:10 landscape, portrait-native) → 172.2 > 164 (X-fit) and > 124 (Y-fit): **panel itself does not fit** under REQ-ENC-01 with 3 mm walls.
- **Consequence:** either (a) accept a **~7" class 16:10** panel (fits interior 164 × 124 with bezel), (b) enlarge the envelope (REQ-ENC-01 change), or (c) rebuild the lid with near-zero bezel and thin glass (172 mm needs lid ≥ ~176 inner → exterior > 180): **all three change baseline — open decision** (RISK-018/024, DEC-035 keep-out conflict).

---

## 9. Findings summary

1. Envelope 130×170×50 is achieved only by z-stacking the carrier over the battery tunnels (§4.3); strict plan-view DEC-018 split is out of fit.
2. Battery packs 2 × 105×70×24 fit across the 164 interior (≈150 + rails) and give 144 Wh.
3. Z-stack budget is **at the limit** — heat-sink 5–6 mm and SoM ≤ ~10 mm over carrier are gating (measure before CNC).
4. **Display does not fit**: Vu8S bracket 202×153 and even the bare 172×108 active area exceed the 164×124 lid → needs a user decision (§8.1).
5. Mass ≈ 1.63–1.9 kg; X₍CoM₎ = 53 % (closed), ≤ 63 % (both packs out, lid open) — no tip-back.
6. Carrier/daughterboard split keeps all high-speed routes on the carrier (RISK-015).

---

## 10. Open mechanical decisions needing user input

| # | Decision | Options | Blocks |
|---|---|---|---|
| M-1 | **Plan-view split vs z-stacked carrier** (DEC-018 interpretation) | accept mid-deck-over-bays (§4.2) | entire mechanical baseline |
| M-2 | **Display size/orientation** (Vu8S doesn't fit; §8.1) | 7" 16:10 vs envelope widen vs zero-bezel rebuild | lid anodizing + revenue |
| M-3 | Lid/glass thickness split (base interior 41 vs 44 mm) | 3–4 mm lid glass; affects HS 7 mm feasibility | z-stack, DEC-001 |
| M-4 | Heat-sink 5 vs 6 vs 7 mm | depends on measured SoM B2B height | thermal, DEC-017 |
| M-5 | Rear-wall I/O connector order + pogo exact magnet/contact spec | TBD, sourced Mill-Max 0964-class | daughterboard layout |
| M-6 | Rear vs side exhaust final allocation | bench-measure fan direction (RISK-016) | vent CNC |

All others in this doc are **TBD-unverified** until Phase-8 measurement of the actual parts.