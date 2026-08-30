# Power Tree — Phase 6 Electrical Design

**Status:** Phase-6 planning design. **NOT released for fabrication.** All part values, prices and efficiency figures are first-order engineering estimates to be confirmed in the **schematic review gate** and the **Phase-8 bench** (REQ-PWR-05). Prices are snapshots (stale within weeks) unless marked otherwise.
**Date:** 2026-08-30.
**Nomenclature:** `VBAT_SYS` = shared system/battery rail (OR-ed pack output + NVDC charger node). Units: V, A, W, Wh, Ω, µF.

---

## 1. Topology Overview

```
USB-C receptacle
   │  USB-PD (20V / 3.25A, 65W, E-marked 5A C-C cable)
   ▼
 STUSB4500  (sink, fixed PDOs, NVM)
   │ VBUS_EN / POWER_OK          ─► PWR-MCU GPIO
   ▼
[P-FET VBUS load switch + eFuse/ILIM  (schematic-review gate G-SINK)]
   ▼ VBUS 20V
 BQ25713RSNR  (NVDC 1-4S buck-boost charger, I2C)
   │                       │
   │  SYS (VBAT_SYS)        │  BAT (battery node, also VBAT_SYS by NVDC)
   ▼                       │
 VBAT_SYS rail             │
   ├─► pre-charge FET+R   ◄┘   (per pack X_A, 100 Ω)
   ├─► charge-steer FET-A  ◄──── pack A slot  (charges pack A)
   ├─► charge-steer FET-B  ◄──── pack B slot
   ├─► LM5050-1 OR FET-A ◄────── pack A slot   (discharge only)
   ├─► LM5050-1 OR FET-B ◄────── pack B slot   (discharge only)
   └─► split-rail converters
```

- **Pack internal (per `hardware/battery-hot-swap.md`):** 4S1P Molicel INR21700-P50B (14.4 V nom, 10.0–16.8 V window), BMS = TI BQ76920-class AFE (cell monitor, balance, OV/UV/OCD/SCD, temp, coulomb counter) with optional S-8261/S-8252-class analog secondary protector. Pack terminals: **P+ / P− / NTC / ID**.
- **Discharge OR-ing:** one ideal-diode FET controller (LM5050-1, SOT-23-6, 5–75 V) per pack, external N-FET (see §4). Direction strictly pack → `VBAT_SYS`; reverse (back-feed) blocked by design (RISK-004 mitigation).
- **Charge steering:** each pack slot has a **back-to-back FET pair** from `VBAT_SYS` to the pack node, gated by the PWR-MCU (`EN_CHG_A/B`) for per-pack charge priority/taper. Back-to-back = block in both OFF directions (no back-feed, no self-discharge through the switch).
- **Pre-charge:** each pack slot has a low-current pre-charge FET + resistor (≈100 Ω, 1 W) from `VBAT_SYS` to the gain node, closed by the MCU **before** either main switch enables → bounds inrush and parallel-pack circulating current (RISK-004/RISK-008).
- **Per-pack current/voltage sense:** INA260 (integrated 2 mΩ shunt, I2C, ±15 A, 0–36 V common mode) per slot, reports net bidir charge/discharge current, bus voltage, power to the PWR-MCU.
- **Charger (BQ25713RSNR):** NVDC buck-boost, input 20 V USB-PD, output battery node. NVDC = system rail tracks battery voltage, **instant-on with 0 packs** when adapter present; battery supplements on peak load; designed for 1–4S Li-ion, charge V up to 19.2 V, charge I up to 8.13 A.

### Design decisions in this topology
1. **Ideal diode per source (LM5050-1) rather than a shared power MUX (TPS2121).** TPS2121 (2.7–22 V, 4.5 A max, integrated 56 mΩ switches) is a strong *integrated* alternative, but (a) its 4.5 A limit is marginal when one pack must carry charge (≤4.9 A worst) + peak load, and (b) reverse-bidirectional conduction while a channel is *on* complicates guaranteed no-back-feed proof. The discrete LM5050 approach is slower to assemble but gives independent, provable uni-directional blocks and higher current headroom. TPS2121 retained as an alternate for review (G-ORMUX). *LTC4417 (3-input priority PowerPath, 5.5–38 V) was evaluated as an alternative; page-level figures could not be verified today — TBD.*
2. **Charge path separated from discharge path.** An ideal-diode OR controller cannot charge a pack (it blocks reverse current), so charge flows through the dedicated back-to-back charge-steering FETs; discharge flows only through the OR FETs. This is the standards-compliant "power bridge" structure (T480-class behaviour, DEC-004).
3. **Charger battery node == VBAT_SYS** (NVDC). Simplifies rails; charging one or both packs is a function of which charge-steer switch is closed, not of re-wiring the rail.

---

## 2. Rail Voltage Definitions and Budget

### 2.1 Rail map

| Rail | Nominal | Range | Sourced from | Primary loads | Notes |
|---|---|---|---|---|---|
| VBUS | 20 V | 5/9/20 V PDO (target 20) | USB-C PD adapter | Charger input | 65 W contract |
| `VBAT_SYS` | 14.4 V | 13.2–16.8 V (pack) / up to ~16.8 V on adapter | OR-ed packs + charger | split-rail converters, charger BAT node | NVDC: system ≤ battery CV |
| 12 V | 12 V | ±3% | LMR33630 DDA | Display backlight + panel driver, 12 V aux | Light load |
| 5 V | 5 V | ±3% | LMR33630 DDA | Wi-Fi/BT, LTE, fan, USB VBUS, USB bus-peripherals | — |
| 3.3 V | 3.3 V | ±3% | LMR33640 DDA | SoM (Verdin i.MX8MP), NVMe SSD, ETH PHY, SD, audio, trackball, logic | Heaviest rail |
| 3.3V_ALW | 3.3 V | ±3% | LMR33620 DDA (always-on from VBAT_SYS) | PWR-MCU, INA260, LED, wake logic | Always-on when any power present |

### 2.2 Per-state load budget

Planning targets (battery-side conversion assumed η_buck = 0.90 per rail, quiescent fixed 0.3 W). These reconcile with the feasibility totals (3.8 / 6.6 / 8.7 / 1.2 W) within a small modelling delta (feasibility pooled converter loss) — **final authority = Phase-8 measurement.**

**State definitions (REQ-PWR-02 workload):**
- **Idle/Terminal** — terminal + idle SoM, display on @ working brightness.
- **Browse** — web browsing + SoM active.
- **Radio** — survey mode, Wi-Fi/BT active + LTE data.
- **Peak burst** — short SoC boost spike (ms–s).
- **Suspend** — locked / suspend (display off).
- **Charge+use** — adapter attached while browsing/radio.

| State | 12 V load (W/A) | 5 V load (W/A) | 3.3 V load (W/A) | Quiescent (W) | **Battery-side total (W)** @η=0.90 | Feasibility target (W) |
|---|---|---|---|---|---|---|
| Idle / Terminal | 1.8 / 0.15 | 0.2 / 0.04 | 1.5 / 0.46 | 0.3 | **≈4.2** | 3.8 |
| Browse | 2.5 / 0.21 | 0.6 / 0.12 | 3.2 / 0.97 | 0.5 | **≈7.5** | 6.6 |
| Radio | 2.5 / 0.21 | 2.9 / 0.58 | 3.2 / 0.97 | 0.6 | **≈10.1** | 8.7 |
| Peak burst | 2.5 / 0.21 | 0.9 / 0.18 | **5.5 / 1.67** | 0.5 | **≈10.4 (avg) / 12–18 raw** | 15–18 |
| Suspend | 0.3 / 0.03 | 0 | 0.8 / 0.24 | 0.2 | **≈1.4** | 1.2 |
| Charge + use | as Browse | as Browse | as Browse | — | system fed by adapter; battery side = charging power | 65 W input |

Notes:
- **3.3 V rail in Peak burst:** SoM (Verdin) mean max ~5 W but spec'd short spikes to 10–15 W ⇒ sized LMR33640 (4 A = 13.2 W) with a note: sustained >4 A → thermal throttle via PROCHOT. Peak-burst current 1.67 A is the *mean*; burst to 3.8–4.5 A accepted by converter transient (presented as 10.4 W average, short 15–18 W spikes).
- 5 V rail: Wi-Fi 0.6 W, LTE 2.0 W (1.5–2.5), fan 0.3 W. USB bus-powering external peripherals adds up to ~1.5 A (7.5 W) — treated as transient, reduces charge headroom (IINDPM governs).
- 12 V rail: display backlight ≤2.5 W working brightness is the budget driver (REQ-PWR-02 sensitivity: **+0.5 W display ≈ −2.4 h**).

### 2.3 Battery-rail current (bounds the OR FETs, fuses, conductors)

| Quantity | Formula (units) | Worst-case value | Comment |
|---|---|---|---|
| Charge current into pack(s) | I_chg = P_in·η_chg / V_pack | 65 × 0.92 / 14.4 = **4.15 A** (3.56 A @16.8 V) | both packs in parallel when charging simultaneously |
| Discharge current | I_dis = P_load / V_pack_min | 18 / 13.2 = **1.36 A** | peak burst |
| Single-pack worst path | I_path ≤ I_chg + I_dis | ≈ **4.9 A** | one pack carrying charge+load |
| OR FET / fuse / conductor rating | ≥ 2 × worst path | **≥ 8 A design, 12 A deck main fuse** | margin per §8 |

---

## 3. DC–DC Converter Selection

Family choice: **TI LMR3361x/3362x/33630/33640 SIMPLE-SWITCHER synchronous bucks** (3.8–36 V in, light-load PFM, >95 % peak efficiency, `POWER_GOOD`, HSOIC-8 4.9×6.0 mm = user-solderable; pin-compatible across the 1/2/3/4 A family so one layout serves all rails) [LMR33630, LMR33640 product pages, accessed 2026-08-30].

Selected parts:

| Rail | Part | Package | Vin (V) | Iout (A) | η peak | Indic. price (€, snapshot, TBD re-quote) | Availability (TBD at order) | Notes |
|---|---|---|---|---|---|---|---|---|
| 12 V | **LMR33630DDA** | HSOIC-8 | 3.8–36 | 3 | >95 % | ~1.30–1.90 | Mouser/Farnell stock varies | 12.9:1 extreme duty OK; backlight light load → PFM |
| 5 V | **LMR33630DDA** | HSOIC-8 | 3.8–36 | 3 | >95 % | ~1.30–1.90 | as above | USB + RF + fan; 3 A covers surge |
| 3.3 V | **LMR33640DDA** | HSOIC-8 | 3.8–36 | **4** | >95 % | ~1.60–2.20 | as above | 4.4:1 duty; 13.2 W capacity covers SoM+SSD+logic |
| 3.3V_ALW | **LMR33620DDA** | HSOIC-8 | 3.8–36 | 2 | >95 % | ~1.20–1.80 | as above | always-on; 24 µA Iq; enables MCU wake |
| Discharge OR | **LM5050-1** (×2) | SOT-23-6 | 5–75 | n/a (gate) | — | ~0.80–1.30 | stock varies | 400 µA Iq, 50 ns reverse block, 100 V transients |
| OR external FETs (×2) | e.g. 40–60 V N-FET, RDS(on) ≤5 mΩ, ≥20 A | DFN/PowerPAK | — | 20 | — | ~0.50–1.20 (TBD) | — | exact part **G-FETSEL** |
| Current sense | **INA260AIDGSR** (×2) | TSSOP-16 | 0–36 CM | ±15 | — | ~1.40–2.00 | stock varies | 2 mΩ integrated shunt, I2C, 16-bit |
| Charger | **BQ25713RSNR** | WQFN-32 (4×4) | 3.5–24 | 8.13 (max) | ~97 (buck-boost 1:1) | **€3.30** net (BOM B-009) | in stock (BOM) | — |
| PD sink | **STUSB4500QTR** | QFN-24 | VBUS | — | — | **€0.99** net (BOM B-010) | production (BOM) | — |

Cascade efficiency vs target (≥90 % per the approved constraint):
```
η_total (battery → load) = η_OR ≈1.00 × η_charger(bypass/buck-boost) × η_buck
  NVDC battery→system:   ~0.97–0.99 (system tracks battery)
  buck 14.4→12/5/3.3:    ~0.90–0.93 (LMR336x at operating points)
  ⇒ overall ≈ 0.87–0.92
```
The approved target "≥90 % conversion" is interpreted per-stage; the **composite battery→3.3 V is ≈ 87–91 %** and is the correct number for runtime math (§6).

---

## 4. OR-ing, Pre-charge and Hot-Swap Bridge

### 4.1 Discharge OR (LM5050-1)
- One controller + one N-FET per pack. Controller regulates FET V_DS to act as an ideal diode (low forward drop when pack is the highest source); 50 ns response to **any reverse current** → FET off (back-feed excluded) [LM5050-1 product page, accessed 2026-08-30].
- FET headroom: V_DSS ≥ 40 V (pack max 16.8 + transients), I_D ≥ 20 A, R_DS(on) ≤ 5 mΩ → P_loss = I²R at 4.9 A worst ≈ 0.12 W (no heatsink needed). **Exact FET part + gate charge vs LM5050 charge-pump drive capability = review gate G-FETSEL.**
- **No battery present / adapter present:** OR FET off; rail fed by charger via NVDC → system runs (0-pack operation).
- Deck bulk capacitance on VBAT_SYS ≈ 1000 µF (ceramic + electrolytic mix) — **TBD value in review** (noise + hold-up, §7.3).

### 4.2 Pre-charge / insertion sequence (both packs powered, T480-style)
1. Pack slides in; **GND mates first, then ID/NTC sense, then P+ (staged pins)** → MCU sees slot occupied + reads ID (max ~1 ms).
2. MCU measures V_pack via INA260 bus-voltage register. If |V_pack − V_VBAT_SYS| > ~200 mV → **pre-charge**:
   - close `PRE_CHG_A` FET → pack charges `VBAT_SYS` bulk via **R_PRE = 100 Ω / 1 W**.
   - τ = R_PRE × C_bulk = 100 × 0.001 = **100 ms**; allow ≈2–3 τ (200–300 ms) until |ΔV| < ~150 mV.
   - pre-charge peak current = ΔV_max / R_PRE ≈ (16.8 − 13.2)/100 = **36 mA** (benign).
3. When ΔV settled → MCU enables discharge OR (LM5050 engages on forward current by itself) and/or charge-steer FET. Residual circulating current between paralleled packs ≤ ΔV/(R_int_A + R_int_B + R_sw) ≈ 0.15/(0.03) ≈ **5 A peak, settling in seconds** — acceptable and bounded; pre-charge prevents the uncapped 10–20 A slam of un-gated parallel insertion.
4. If |V_pack − V_rail| > 700 mV (deep differential) → **hold off full connection**, charge pack alone via pre-charge/charger up to match, or reject (out-of-window) and set fault LED. **Policy = review gate G-SEQ.**

### 4.3 Charge steering
- `EN_CHG_A` / `EN_CHG_B` gated back-to-back FET pairs from VBAT_SYS → pack node. Used for **sequential / priority charging** and to isolate a full or faulted pack.
- Back-to-back ensures zero leakage path pack→rail when off (supports "one charging, other idle, no back-feed").

---

## 5. USB-C PD Sink (STUSB4500) Notes

Fixed-PDO sink, no host firmware needed at runtime (NVM-programmed once via the ST USB-I2C tool). **[STUSB4500 product page: https://www.st.com/en/power-management/stusb4500.html; page did not load in this session (timeout). Specs below per Phase-4 evidence (candidates.md E-025) + app-note knowledge; validate against the ST datasheet PDF (https://www.st.com/resource/en/datasheet/stusb4500.pdf) in the review gate — TBD items marked G-SINK.]**

| Item | Design statement |
|---|---|
| PDO table (NVM) | PDO1 = 5 V/3 A (15 W), PDO2 = 9 V/3 A (27 W), PDO3 = **20 V/3.25 A (65 W)**. Sink requests highest → 20 V |
| Cable | **E-marked 5 A USB-C C-C cable required** for full 65 W (20 V × 3.25 A > 3 A cable class) — fixed constraint |
| CC pins | CC1/CC2 → STUSB4500; 5.1 kΩ Rd-to-GND (sink baseline) for attach |
| VBUS power path | Receptacle VBUS → **external P-FET load switch** (≥30 V, ≥6 A; controlled by STUSB4500 `POWER_OK`/GPIO) → charger VBUS. Internal switch (if present in the IC) is **not** sized for 3.25 A continuous — **verify in G-SINK** |
| IINDPM | Hardware ILIM resistor on BQ25713 + charger input-current register set ≈3.1–3.25 A (20 V PDO); `ILIM` pin R is a fail-safe independent of I2C |
| Dead battery / 0 packs | STUSB4500 self-powers from VBUS (internal LDO) on attach and negotiates regardless of battery SoC ⇒ system boots from adapter with **0 packs** (NVDC instant-on, BQ25713 "Instant-on with no battery") |
| Optional Host link | I2C to PWR-MCU optional (runtime PD status/GPIO); default = autonomous NVM operation. `POWER_OK/CONNECT` flags → MCU for charge-enable + LED |
| ESD/OVP on VBUS | fixed-OVP ±20% on VBUS, + TVS (§8). |

---

## 6. Charger (BQ25713RSNR) Configuration Notes

Verified from the TI product datasheet summary [BQ25713 product page, accessed 2026-08-30]:

| Parameter | Setting | Basis |
|---|---|---|
| Input range | 20 V USB-PD (3.5–24 V OK; abs 30 V) | datasheet |
| Topology | NVDC buck-boost 1–4S; seamless buck/boost/buck-boost | datasheet |
| Charge voltage (4S) | **16.8 V (4.2 V/cell)**; ±0.5 % regulation | datasheet; Molicel P50B CV = 4.2 V |
| Charge current | Combined CC **≈ 3.0–3.25 A** (register; LSB 512 mA) → **≈1.5–1.6 A/pack** (0.3 C @5 Ah) | conservative, cool; input-limited by IINDPM anyway |
| Input current limit | IINDPM ≈ **3.1–3.25 A** (reg 0x03/ILIM pin); ICO (Input Current Optimizer) enabled for weak adapters | datasheet feature |
| Switching freq / inductor | 800 kHz / 2.2 µH (efficiency) or 1.2 MHz / 1.0 µH (size) — **G-CHG** | datasheet |
| Monitoring | Integrated ADC: VBUS/IBAT/ISYS/Power (±2 % I, ±4 % P) → PWR-MCU telemetry | datasheet |
| Thermal | IC thermal regulation reduces charge current; **PROCHOT** output → SoM throttle; PWR-MCU uses BQ76920 NTC + BQ25713 die temp for charge taper | datasheet + RISK-016 |
| Safety | Input/system/battery OVP, input/FET/inductor OCP, thermal shutdown, signed-down IEC 62368-1 CB (datasheet) | datasheet |
| Host | I2C/SMBus — target slave, PWR-MCU writes ChargeCurrent/ChargeVoltage, reads ADC/protection status | — |
| OTG | Disabled (no port power-back needed) | — |

Charge-time math (see `hardware/battery-hot-swap.md` §6): 0–100 % both packs in parallel **≈ 2.7–3.2 h**, sequential **≈ 3.1–3.6 h** at 65 W input, 0–80 % ≈ 2.0–2.4 h.

---

## 7. Power Sequencing and PWR-MCU Control Model

### 7.1 Power-good sequence (boot)
1. Any source wake: pack (OR) or adapter (NVDC instant-on) → `VBAT_SYS` rises; `3.3V_ALW` up → PWR-MCU alive.
2. MCU checks: slot occupancy/ID, pack voltages, temps, OR status, adapter present. Sets LEDs.
3. MCU enables **12 V → PG → 5 V → PG → 3.3 V → PG → SoM PWR_EN** (Verdin `PWR_CTRL`), with 10 ms staggers (converter soft-start + inrush).
4. SoM boots, takes over I2C/UART control link; MCU remains power-supervisor (watchdog).
5. Charging allowed only after both (a) adapter present and (b) pack BMS OV/UV/temp checks pass.

### 7.2 PWR-MCU → peripherals map (rear daughterboard, DEC-018)

| Function | Peripheral/interface | Notes |
|---|---|---|
| Charge priority/taper | BQ25713 I2C (ChargeCurrent/ChargeVoltage/status) + `EN_CHG_A/B` | sequential or parallel policy §→ battery-hot-swap §7 |
| Battery sense | INA260 ×2 (I2C) per slot; BQ76920 ×2 (I2C) inside packs | voltage, I, P, coulomb count, temp |
| OR-ing sequencing | `PRE_CHG_A/B`, `EN_DSG_A/B`, `EN_CHG_A/B` (GPIO) + LM5050 status | §4.2 sequence |
| Fan PWM | Fan PWM + tach (Sunon HA30101V4) | thermal from BQ25713 die + NTC |
| LED | RGB/status LED GPIO | on/charge/fault |
| Power button | GPIO wake; suspend/resume handshake with SoM | — |
| SoM link | I2C + UART (I-14) | telemetry + commands |
| Fault handling | PG deassert / charger fault / BMS fault → shutdown rails in reverse order; fault latch + LED | — |

### 7.3 Problems to hold up through sequencing / hot-swap
- Hold-up on removal: load 4 W ÷ 14.4 V = 0.28 A; 1000 µF allows ΔV=0.3 V for t = C·ΔV/I = 0.001·0.3/0.28 ≈ **1.1 ms** — far above LM5050 switch-over (sub-µs–µs). Sufficient.
- Sequencing protects against rail brownout on insertion: pre-charge first, main after ΔV settled (§4.2).

---

## 8. Fusing and Protection

| Element | Location | Rating/selection (TBD = review) | Fails against |
|---|---|---|---|
| Slim per-pack fuse | P+ in pack or deck bay contact | ≥8 A design; interrupt ≥50 A at 16.8 V; I²t small **G-FUSE** | shorted cell/module, bay short |
| Deck main fuse | VBAT_SYS input | 12 A automotive/panel | combined charge+load short |
| TVS `VBAT_SYS` | on rail | SMBJ18A-class (standoff ~18 V, clamp ≤ 29 V < converter 36 V abs) **G-TVS** | hot-plug ringing, charger overshoot |
| TVS VBUS | after load switch | standoff ~22–24 V, clamp <30 V **G-SINK** | adapter overshoot/ESD |
| Polarity | pack connector + OR FET | keyed connector + LM5050 FET blocks reverse (body-diode orientation) | reversed insertion |
| Overcurrent | charger | BQ25713 input/charge OCP; OR FET no OCP (relies on pack fuse + BMS OCD/SCD) | adapter short, pack overload |
| eFuse (optional) | VBUS, 20 V | TPS2594x or discrete `ILIM` fail-safe | adapter current runaway — **G-SINK** |
| Connector staged pins | pack↔bay | GND first, ID/NTC next, P+ last-mate (long) | inrush/ESD without pre-charge |
| Cell-level | inside pack | BQ76920 OV/UV/OCD/SCD + optional secondary protector (see battery-hot-swap.md §3) | cell fault within pack |

---

## 9. Formulas, Units, Margins (recap)

- Battery draw: `I_batt = Σ(P_rail / η) / V_batt`  (A = W/V). Unity check.
- Cascade efficiency: `η_total = η_OR · η_charger · η_buck ≈ 0.87–0.92`.
- Runtime: `t = E_cells · SOH · η_total / P_avg`. P50B: `144 Wh · 0.90 (SOH+reserve) · 0.90 = 116.6 Wh usable`.
  - Workload: `20h·3.8 + 4h·6.6 + 6h·1.2 = 109.6 Wh` → **31.9 h nominal vs 30 h target (+6.4 %)**; avg draw 3.65 W < 4 W constraint ✓.
  - Sensitivity: +0.5 W display ≈ −2.4 h.
- Charge time: `t_chg = E_batt / (P_in · η_chg − P_load)` (CC) + CV taper term; see battery-hot-swap.md §6.
- Pre-charge: `τ = R_PRE · C_bulk`, `I_pre = ΔV / R_PRE`, `ΔV allow < 150–200 mV`.

---

## 10. Open Items for Schematic Review Gate (Phase-6 contiguous)

| Gate | Item | Status |
|---|---|---|
| G-SINK | STUSB4500 PDO NVM programming + external VBUS P-FET vs internal switch rating; D+/D− wiring; eFuse | **Open — ST datasheet not parsed this session** |
| G-CHG | BQ25713 inductor value, ILIM resistor, sense-resistor & brew sense; register map base config | Open |
| G-FETSEL | OR N-FET exact part (V_DSS, R_DS(on), gate-charge vs LM5050 charge-pump); charge-steer back-to-back FET pair | Open |
| G-SEQ | Insertion policy: acceptable ΔV, pre-charge time, fault/withhold thresholds | Open |
| G-FUSE / G-TVS | Fuse part + fuseholders; exact TVS values; connector pin staging | Open |
| G-MUX | TPS2121 integrated alternative (4.5 A margin study) | Open |
| G-HOLDUP | VBAT_SYS bulk capacitance value (noise vs hold-up) | Open |
| G-ALW | 3.3V_ALW definition: MCU idle current budget (<2 mA) | Open |
| G-MECH | Pack connector part (pogo/leaf-spring), staged pins, latch — cross-ref mechanical docs | Open (Phase-6 mech) |
| **G-SAFETY** | **Independent professional battery engineer review (REQ-PWR-05) before any fabrication** | **Mandatory** |

---

## 11. References (all accessed 2026-08-30)

| # | Source | URL |
|---|---|---|
| R1 | TI BQ25713 product page (NVDC buck-boost, 1–4S, 3.5–24 V, 8.13 A max, WQFN-32) | https://www.ti.com/product/BQ25713 |
| R2 | TI BQ25713 datasheet PDF (Rev. C) | https://www.ti.com/lit/gpn/BQ25713 |
| R3 | TI LM5050-1 product page (OR-ing FET controller, 5–75 V, 400 µA Iq, 50 ns reverse block, SOT-23-6) | https://www.ti.com/product/LM5050-1 |
| R4 | TI TPS2121 product page (dual power mux, 2.7–22 V, 4.5 A, reverse block, inrush) | https://www.ti.com/product/TPS2121 |
| R5 | TI BQ76920 product page (3–5S AFE: OV/UV/OCD/SCD, balance FETs, CHG/DSG drivers, coulomb counter, 3×NTC; BQ76920 = 3–5S) | https://www.ti.com/product/BQ76920 |
| R6 | TI BQ76940 product page (same family, 9–15S — listed for completeness, NOT used) | https://www.ti.com/product/BQ76940 |
| R7 | TI INA260 product page (I2C monitor, 2 mΩ int. shunt, ±15 A, 0–36 V CM, 16-bit) | https://www.ti.com/product/INA260 |
| R8 | TI LMR33630 product page (36 V, 3 A, HSOIC-8, >95 %, 24 µA Iq, PG) | https://www.ti.com/product/LMR33630 |
| R9 | TI LMR33640 product page (36 V, 4 A, HSOIC-8, pin-compatible family) | https://www.ti.com/product/LMR33640 |
| R10 | ST STUSB4500 product page (PD3.0 sink, 3 fixed PDOs) — **page did not load (timeout); specs TBD** | https://www.st.com/en/power-management/stusb4500.html |
| R11 | ST STUSB4500 datasheet PDF — **binary/timeout; specs TBD** | https://www.st.com/resource/en/datasheet/stusb4500.pdf |
| R12 | Analog Devices LTC4417 product page — **page did not load (timeout); specs TBD** | https://www.analog.com/en/products/ltc4417.html |
| R13 | Project-internal | `parts/candidates.md` §9 (E-025), `bom/bom.md` B-009/B-010, `decisions/decision-log.md` DEC-004/012/038, `risks/risk-register.md` RISK-004/008/016/023 |