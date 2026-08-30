# Battery Pack & Hot-Swap Design — Phase 6

**Status:** Phase-6 planning design. **NOT released for fabrication.** Safety-critical subsystem (REQ-PWR-05): every value here requires confirmation by a **qualified battery/safety engineer** before fabrication and before UN38.3/ADR transport. Prices/pages marked TBD were not verifiable today (2026-08-30).
**Date:** 2026-08-30.

---

## 1. Scope and Requirements Mapping

| Requirement | Covered by |
|---|---|
| REQ-PWR-01 (≥120 Wh, 2 × pack ≥60 Wh) | §2 — 2×4S1P × P50B = **144 Wh** (72 Wh/pack) |
| REQ-PWR-02 (30 h runtime) | `power-tree.md` §9 — 31.9 h nominal |
| REQ-PWR-03 (true T480-style hot-swap) | §4/§5 behaviour matrix |
| REQ-PWR-04 (in-device 65 W USB-C PD charge; external optional) | `power-tree.md` §5/§6, §7 charging behaviour |
| REQ-PWR-05 (safety review before fabrication) | §9 gates |
| RISK-004 / RISK-008 / RISK-016 / RISK-023 | §4–§9 |

---

## 2. Pack Mechanical + Electrical Design

### 2.1 4S1P physical layout
- Cells: **4 × Molicel INR21700-P50B** in series (4S1P), inline (end-to-end) along the 105 mm pack axis (matches DEC-003 geometry: slab ≈ **105 × 70 × 24 mm**). Datasheet max dims: Ø21.55 × 70.15 mm; holder sized for 21.6 mm / 70.8 mm see `parts/candidates.md`.
- Construction: cells in a plastic/Li-ion-safe carrier (insulated cells, tab welding on nickel strips — **spot weld, never solder** on cell terminals), BMS PCB at the cap end, NTCs on cells, Kapton/fiberglass insulation over welds. Custom-pack professional build indicated (RISK-008).
- Cell spec baseline (datasheet values per `parts/candidates.md`, High confidence; **page-level pin numbers TBD at Phase-8**):

| Param | Value | Unit | Source |
|---|---|---|---|
| Nominal voltage | 3.6 (4S → 14.4) | V | datasheet 1.0 |
| Capacity | 5000 | mAh | datasheet |
| Energy / cell / pack | 18.0 / **72.0** | Wh | datasheet |
| Max charge voltage | 4.2 ±0.05 (4S → **16.8**) | V | datasheet |
| Min discharge voltage | 2.5 (4S → **10.0**) | V | datasheet |
| Max continuous discharge current | **TBD** (power-grade cell; ≥15 A expected) | A | unverified today — **G-CELL** |
| Max charge current / recommended | **TBD** (recommend ≥0.5 C = 2.5 A/cell; deck charges ≤1.6 A/pack) | A | unverified — **G-CELL** |
| AC impedance | **TBD** (~10–16 mΩ class) | mΩ | unverified — **G-CELL** |

### 2.2 Pack voltage window

| Level | Value (4S) | Value (per cell) | Comment |
|---|---|---|---|
| Overcharge hard limit (BMS OV) | **16.8–17.2** | 4.2–4.30 | BQ76920 configurable; charger CV 16.8 |
| Full charge | 16.8 | 4.20 | charger CV |
| Nominal | 14.4 | 3.60 | — |
| Load disconnect (BMS UV) | **12.0** | 3.00 | protects cell <2.5 V; sets practical floor above datasheet min |
| Absolute cell floor (datasheet) | 10.0 | 2.50 | below this = damage; BMS must cut earlier |

Operational deck window therefore **13.2–16.8 V** (load), protection 12.0 (UV) / 16.8+ (OV).

### 2.3 BMS / protection requirements

| Function | Requirement | Implementation | Notes |
|---|---|---|---|
| Overcharge protection | OV per cell, detect + interrupt charge | BQ76920 (per-cell 0.78% V ADC) + charger CV 16.8 + charge-steer off | dual-layer (charger + BMS) |
| Overdischarge protection | UV per cell, load disconnect | BQ76920 DSG FET control at ~3.0 V/cell | |
| Overcurrent / short | charge & discharge | BQ76920 OCD/SCD with sense FET/external shunt; pack fuse | |
| Cell balance | passive 4-cell | BQ76920 integrated balance FETs, MCU-driven during CV | only used in CC/CV window |
| Temperature | max 3 × 10k NTC (103AT) | BQ76920 TS inputs + deck-side | charge limit temp (typically 0–45 °C), discharge limit — per Molicel spec **G-CELL** |
| Coulomb counting | fuel gauge | BQ76920 internal current ADC | updated via I2C |
| Communication | pack → deck/MCU | I2C (BQ76920) via pack **ID/comm** pin | optional; connector adds a pin |
| Secondary protection (recommended) | independent OV/UV | S-8261-class per-cell or S-8254/S-8252-class 4S analog protector **G-BMS2** | belt-&-braces redundancy for custom builds |

Circuit choice: **BQ76920** (TSSOP-20, 3–5S, 40 µA op-current, `SHIP` mode) as primary AFE for full telemetry + balance [TI product page, accessed 2026-08-30]; S-8261/S-8252-class analog protector as **optional independent secondary** (page-level params TBD — see §11). BQ76940 (9–15S) is **not** suitable (too many cells) — listed for completeness.

### 2.4 Contact / pin design (front insertion)
- **Slot/connector:** front-facing bay, staged pins **GND (first), ID/NTC (mid), P+ (last-mate / first-break longest pair)**. Power handled by two parallel pins each for P+/P− (current 4–5 A worst, ≥10 A combined rating). Pogo or leaf-spring contacts; rated for ≥500 insert/removal cycles.
- Pinout: `P+ | P+ | ID | NTC | P− | P−` (6-way, 5 distinct signals). ID = resistor divider per slot to auto-detect pack in A vs B.
- Keying + polarization so reversed insertion is physically impossible; TVS + LM5050 FET body-diode orientation as electrical backstop (`power-tree.md` §8).
- **Connector exact part: TBD — mechanical doc gate G-MECH** (either branded spring-pins + magnets or pogo module; 2-pin 2 A seller-pogo sets are NOT adequate for 5 A).

---

## 3. Hot-Swap Behaviour Matrix (REQ-PWR-03)

Conventions: `OR-A` = pack A discharge ideal diode (LM5050-1 + FET); `CS-A` = pack A charge-steer switch; `PRE-A` = pack A pre-charge FET+R.

| # | Scenario | Rail state | Current paths | Controller action | Upset? | Notes |
|---|---|---|---|---|---|---|
| 1 | **2 packs installed, normal op** | Held ≈ higher pack V | load shared by V/I of OR diodes | none (auto) | none | Or-ing shares load by pack voltage; equalization bounded by R_int |
| 2 | **Remove A while powered** | Held by B (no rail drop) | load B→rail | OR-A off on reverse sense (50 ns); MCU opens CS-A if charging A; charger retargets to B | none; hold-up cap covers (<1 ms) | ThinkPad T480-class behaviour |
| 3 | **Insert A while powered** | Held by B | pre-charge A→rail (≤36 mA) → then OR-A → rail | §4.2 sequence (PRE-A → compare ΔV → OR-A/CS-A) | <1 % dip during pre-charge; no inrush | pre-charge mandatory |
| 4 | **Both charging** | Adapter fed | charger→VBAT_SYS→CS-A+CS-B→both packs; parallel equalization | both CS closed; charge current register-limited (~1.5–1.6 A/pack); charger CV 16.8 tapers both | equalization current bounded because pre-charged + charger current-limited | default parallel mode |
| 5 | **One charging, one discharging** | Adapter + battery | power: adapter→system; A: VBAT_SYS→CS-A (charge); B: B→OR-B→rail (supplement on peak) | CS-A on, CS-B off, OR-B auto | none (flows independent) | no cross-conduction: OR blocks rail→pack; CS blocks pack↔pack |
| 6 | **Back-feed exclusion** | — | A must never drive B | per-pack barriers: OR (discharge-only, reverse blocked) + CS (back-to-back, OFF = no path) | none | verified by construction + bench (Phase-8) |
| 7 | **0 packs + adapter** | Held by charger (NVDC instant-on) | adapter→charger→system | charging disabled (no pack) | none | boot works with 0 packs |
| 8 | **0 packs, no adapter** | off | — | system off; MCU powered from ALW off | — | RTC/coin-cell fallback **TBD G-MECH** |
| 9 | **Remove A while A charging** | Held by B | B discharges load; A charge path opened by MCU before OR-A shuts | MCU: CS-A off → OR-A off; charger current re-divides to B | minimal (LM5050 + hold-up) | order matters: gated switch opens first, then diode disconnects |
| 10 | **Insert with deeply mismatched V (ΔV> ~0.7 V)** | held | pre-charge only | MCU withholds full connection; charge lone pack to match or fault LED | none | policy in G-SEQ |
| 11 | **Inrush / hot-plug transient** | rail glitch < pre-charge bound | — | PRE sequence + TVS + fuses + staged pins | <1 % | measured Phase-8 |

**Guarantee (primary):** rail continuity during any pack add/remove is provided by (a) the *other* pack via OR-ing, (b) NVDC charger instant-on when adapter present, and (c) `VBAT_SYS` bulk cap → together they bridge the sub-ms switch-over window.

---

## 4. Removal / Insertion Sequence (operational)

**Insert (slot A new, hot):**
1. Slide in until keyed; GND+ID+NTC mate; MCU sees occupancy + ID (~1 ms).
2. Read `V_A` (INA260 bus-V). Compare to `V_rail`.
3. |ΔV| ≤ 0.2 V → skip pre-charge, enable OR-A (+ optionally CS-A). |ΔV| ≤ 0.7 V → `PRE-A` 200–300 ms, re-check, then enable. |ΔV| > 0.7 V → withhold, charge-to-match, or fault.
4. Report pack online + LED.

**Remove (slot A live):**
1. Optional graceful path: MCU signals, transfers out of A, opens CS-A, then OR-A self-disconnects on reverse sense.
2. Hard pull at any instant: OR-A blocks within 50 ns; B + bulk cap sustain rail; CS-A already isolating if not charging A.

---

## 5. Charging Behaviour (single vs both packs)

| Mode | Config | Current steering | When |
|---|---|---|---|
| **Parallel** (default) | CS-A+CS-B on | one shared CC=3.0–3.25 A, 16.8 V CV, both packs equalize in parallel | battery low, both installed, normal |
| **Sequential / priority** | MCU alternates CS | charge low-SOC pack first to 80 %, then other, then top-up both at reduced taper current | any | 
| **Charge + use** | adapter feeds system first (NVDC), remainder splits to pack(s) | IINDPM governs; charge rate auto-reduces on load spikes | adapter plugged while working |
| **Taper** | CV at 16.8 V; current falls | termination at ~5 % C-rate (register, e.g. 0.25 A combined); recharge threshold set | end-of-charge |
| **Thermal taper** | BQ76920 NTC + BQ25713 die temp | MCU lowers ChargeCurrent if temp above threshold (RISK-016) | hot charge |
| **External pack charging** (optional, REQ-PWR-04) | dock/cradle, ≤20 W (5 V/3 A) | slower (~3.6–4.5 h/pack per feasibility §5) | travel spare packs |

Back-feed exclusion integrated with charging: when CS-A is on and CS-B is off, and OR-B blocks rail→B, current **cannot** loop A→rail→B (each path is uni-directional or open). ✓

---

## 6. Charge-Time Calculation (65 W USB-C PD → 144 Wh)

Inputs: P_in = 65 W; η_chg (charger + input path) ≈ 0.92 ⇒ **P_batt,bus ≈ 60 W**; system draw while charging ≈ 4–7 W (working) — from NVDC, the adapter covers it and reduces battery charge power to ≈ **53–56 W**; cells Molicel P50B, E = 144 Wh (both packs), CV taper ≈ +15–20 % on CC-only time; actual taper measured Phase-8.

**Parallel (both packs):**
```
t = E / (P_batt − P_load)  [CC phase, s→h]
  = 144 / 55 ≈ 2.6 h  (CC to ≈85 %)
  + taper ≈ +0.4–0.6 h
  ⇒ 0–100 % ≈ 2.7–3.2 h     0–80 % ≈ 2.0–2.4 h
```
**Sequential:** each pack = 72 Wh; when charging one pack the charger still CC-limits ≈ 3.2 A → 16.8 V ≈ 54 W:
```
  72 / 54 ≈ 1.3 h CC + taper ≈ ≈ 1.4–1.6 h/pack
  ⇒ total ≈ 3.1–3.6 h (slightly longer than parallel; gives priority/health benefits)
```
**While fully loaded (radio + charge):** P_load ≈ 8.7 W ⇒ P_batt ≈ 60 − 8.7/η ≈ 50 W ⇒ ≈ **3.0–3.6 h**. Consistent with feasibility (§5) and RISK-016 thermal: full fan + possible charge taper above temperature threshold.

Notes: charge perf-index lower bounds — realistic full-charge time with degradation & light simultaneous use is **2.8–3.4 h** (feasibility) which matches this model.

---

## 7. Failure / Degradation Behaviour

| Fault | Detection | Response |
|---|---|---|
| Cell overvoltage | BQ76920 OV + charger CV | charge interrupt; LED + UART alert |
| Cell undervoltage | BQ76920 UV | load disconnect (DSG); ship mode to stop self-discharge creep |
| Short / overcurrent | OCD/SCD + pack fuse | immediate disconnect; fuse one-shot |
| Cell temp out-of-window | NTC ×3 | charge/load taper or stop (per Molicel spec G-CELL) |
| Tepid/worn cell (SOH) | coulomb counter vs expected | fuel-gauge readout; charge taper unchanged (0.3 C is gentle) |
| Parallel packs drifting | per-slot INA260 + balance | equalize at CV; if >0.3 V persistent → prioritize/isolate |
| Back-feed event | reverse sense on OR / CS | single-point block per §3 row 6 |

---

## 8. Mechanical Notes / Latch (pointer)

- Pack bay at **front** of base (DEC-018), 2 slots side-by-side across 17 cm width, each ~105×70×24 mm + clearance for contacts (~8–12 mm depth allowance front), latched against the front face.
- Latch mechanism: spring-loaded barrel latch + keyed ramp; eject push-POP from front; **CAD + FEA + prototype in Phase-6 mechanical** (`hardware/` index — create `mechanical/` doc for the bay draft, **G-MECH**).
- Keep ≥4 mm airflow gap around packs during charging (thermal budget, RISK-016); packs ride on rails, isolated from aluminium directly where practical (thermal + wear).

---

## 9. Safety Review Gates (REQ-PWR-05 — Mandatory)

| Gate | Item | Required before |
|---|---|---|
| **G-CELL** | Molicel P50B datasheet page-level pin values (charge/discharge current & temp limits, impedance, cell dimensional end-caps/tab) — **all currently TBD** | cell purchase for fabrication |
| **G-BMS2** | secondary protector choice (S-8261/S-8252/S-8254-class) + thresholds | BMS layout |
| **G-MECH** | connector/pogo part, staged pins, latch, bay draft | fabrication |
| **G-SEQ** | insertion policy (ΔV thresholds, timings, withhold policy) | fabrication |
| **G-FUSE** | lock fuse ratings on cut-sheet | fabrication |
| **G-UN38.3** | cells need UN38.3 test report; transport per IATA/ADR **Pi 965/P903 (IRP)** | shipping packs |
| **G-ADR/REACH** | ADR 2025/2027 class 9 lithium classification; REACH/RoHS declarations (RISK-023) | import/transport |
| **G-SAFETY** | **Independent qualified battery engineer sign-off of pack layout, BMS, hot-swap bridge, charge path, fusing** | **fabrication start** |
| G-PROTO | bench validation: inrush scope, ΔV matrix (§3), thermal charge taper, disconnect/reconnect biking | Phase-8 |

Best practice: two independent protection layers per pack (primary AFE with programmable thresholds + secondary analog protector), certified cells, spot-welded tabs, insulated weld joints, and a **lab test plan** covering rows 1–11 of §3 before any field use.

---

## 10. Formulas + Units (explicit)

- `I_chg = P_in · η_chg / V_pack` — A = W·% / V (65·0.92/16.8 = **3.56 A** max, 4.15 A @14.4 V for both packs).
- Per-pack CC current (parallel) = I_chg/2 ≈ **1.6 A** = 0.32 C @5 Ah (gentle).
- Pre-charge: `τ = R_PRE·C_bulk`; `I_pre = ΔV/R_PRE`; `ΔV_allowed = 150–200 mV`.
- Equalization current after connection: `I_eq ≤ ΔV / (R_int,A + R_int,B + R_sw)` — bounded by pre-charge + charger current limit.
- Charge time: `t = E/(P_chg − P_load)` CC + taper ~1.15–1.20 (h, from Wh/W). 
- Runtime link: `t_run = 144·0.90·0.90 / P_avg` → 31.9 h @ 3.65 W avg (§power-tree).

---

## 11. References (all accessed 2026-08-30)

| # | Source | URL |
|---|---|---|
| R1 | TI BQ76920 product page (3–5S AFE, balance, CHG/DSG FET drivers, OV/UV/OCD/SCD, coulomb, 3×NTC) | https://www.ti.com/product/BQ76920 |
| R2 | TI BQ76940 product page (9–15S — NOT used for 4S; family reference) | https://www.ti.com/product/BQ76940 |
| R3 | Molicel INR21700-P50B datasheet (1.0), referenced values per `parts/candidates.md` §1 — **page-level pins TBD (G-CELL)** | datasheet: `INR21700P50B_…80122.pdf` (molienergy.com, via candidates.md) |
| R4 | TI LM5050-1 (OR-ing controller) | https://www.ti.com/product/LM5050-1 |
| R5 | TI INA260 (per-slot sense) | https://www.ti.com/product/INA260 |
| R6 | ABLIC S-8261/S-8252-class battery protection ICs — **product pages 404/timeout this session; parameters TBD (G-BMS2)** | https://www.ablic.com |
| R7 | Project-internal | `parts/candidates.md`, `bom/bom.md` B-007/B-008, `decisions/decision-log.md` DEC-003/004/012/018/038/041, `risks/risk-register.md` RISK-004/008/016/023, `info/feasibility.md` §2/§5 |