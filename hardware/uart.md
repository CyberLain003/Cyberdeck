# Phase 6 — UART Port Protection Design (Magnetic Pogo, Switchable 3.3/5 V)

Status: **First-cut electrical design for review** — 2026-08-30.
Author: P6 design (document owner: Lain).
Requirement mapping: **REQ-UART-01**, **REQ-UART-02**; interface **I-16** (`architecture/interfaces.md`); parts base **B-017/B-018** (`bom/bom.md`); sourcing evidence per `info/sources.md` strict rule.

> **SAFETY GATE (blocking):** Every block below marked **SAFETY GATE** is NOT construction-ready. It requires (1) a dedicated power-path design review by a qualified electrical/battery safety engineer and (2) user confirmation before any PCB work. This document specifies the recommended gating, not a release.

---

## 1. Objective and scope

Design the external-facing debug UART front-end so that:
1. A user may attach **their own cables** (magnetic pogo, 2.54 mm pitch) to console ports on MCUs, serial consoles, and server COM ports without wiring risk.
2. Logic level is **switchable 3.3 V / 5 V** on the target side (REQ-UART-01).
3. **Default = signal + ground only.** No power is presented on any pogo pin unless the user explicitly enables a dedicated, fused, MCU-gated, and reviewed power path (REQ-UART-01 safety clause).
4. The deck survives the classic failure matrix: reversed cables, shorts, hot/energized targets, unpowered targets, ESD.
5. Signal integrity supports console baud rates (115 200 baud typical; up to ~1 Mbit/s target).

Out of scope (Phase 6 later or Phase 7+): SoM-side UART wiring on the carrier, baud/flow-control policy in NixOS, and whether the port muxes multiple SoM UARTs (open question `OQ` in `interfaces.md`).

---

## 2. Topology

```
 SoM UART (3.3 V)                      DECK FRONT-END (daughterboard rear I/O)              TARGET
┌──────────────┐      ┌──────────────────────────────────────────────────┐        ┌─────────────┐
│ UART_TXD  ───┼──────▶│ series R ─▶ ESD clamp ─▶ level shift ─▶ ESD ──────────▶│ pogo        │
│ UART_RXD  ◀──┼───────│ series R ◀─ ESD clamp ◀─ level shift ◀─ ESD ──────────│ 2.54 mm      │
│ (3.3 Cmos)  │      │                                                  │        │ magnetic set │
└──────────────┘      │ VCCA = 3V3_SoM rail (always on)                  │        │ (TX/RX/GND/ │
                      │ VCCB = 3V3 or 5V rail  ◀── MCU load-switch +     │        │  +VCCO alt)  │
                      │        polyfuse  ── **SAFETY GATE**              │        └─────────────┘
                      │ OE  ◀── power-MCU GPIO (default LOW = Hi-Z)      │
                      │ DET ◀── cable-present sense (mag/Hall or pin)    │
                      └──────────────────────────────────────────────────┘
```

Design intent (why this shape):
- **SoM side is fixed 3.3 V** (i.MX8MP-class UART I/O). Only the **target-facing side (port B)** switches 3.3/5 V.
- The **OE pin** (output-enable) on the level translator is the *master disable*: default low → all pogo-facing outputs Hi-Z → the port is electrically inert until the power MCU enables it.
- All external-facing pins pass through **series resistors** (short/current limit) and an **ESD array to GND** before touching the translator.

---

## 3. Connector and pinout (PROPOSAL — TBD pending user confirmation)

Physical envelope: magnetic pogo set in the **rear I/O zone** of the daughterboard edge (`system-overview.md`, DEC-018). Default user cable = **3 conductors (TX, RX, GND)**. Six-pin head proposed for future capability; the spare pins are **not required** to make a basic cable.

| Pin | Deck signal | Direction (deck↔target) | Default state | Note |
|---|---|---|---|---|
| 1 | **GND** | ⇄ | Connected | Reference; mates **first / breaks last** (longer pogo travel if possible — specify in mechanical doc) |
| 2 | **TX** (deck → target RX) | → | Hi-Z until enable | Series R + ESD |
| 3 | **RX** (target TX → deck) | ← | Hi-Z until enable | Series R + ESD |
| 4 | **+VCCO** (gated power-out) | → | **OPEN / no power** | **SAFETY GATE** — see §8. Absent from default 3-wire cable |
| 5 | SPARE / NC (future: 2nd UART, RTS/CTS, or flow control) | — | NC | Reserved; **must stay unconnected in this design** |
| 6 | **DET** / cable-presence (optional) | — | pulled-low sense | Shorts to GND in a *compliant* user cable; alt. = Hall/reed switch under magnet. Optional in v1 |

> **PROPOSAL STATUS:** Pin map is a proposal, not a release. User must confirm: (a) 4-pin minimal (TX/RX/GND) vs 6-pin; (b) whether a DET/presence pin is wanted or cable-presence shall be sensed off the GND-pinto-magnet geometry; (c) the +VCCO pin **exists but stays disabled by default** (recommended) vs omitted entirely.

Cable guidance for user-built cables (documented in the build guide, not here):
- Core default cable: **3-wire, TX–RX crossed, GND common**, 26–28 AWG, length ≤ ~0.5 m for ≤1 Mbit/s.
- Reversal of TX/RX is electrically harmless (verified in §7 matrix) but logged; a reversed cable simply sees no traffic.

---

## 4. Level shifting — switchable 3.3/5 V

### 4.1 Recommended IC: **TI TXS0108EPWR (primary)** / **TXB0108PWR (alternative)**

| Property | TXS0108E (primary recommendation) | TXB0108 (alternative) |
|---|---|---|
| Part (hand-solderable pkg) | **TXS0108EPWR** — TSSOP-20, 0.65 mm pitch (6.5×6.4 mm) | **TXB0108PWR** — TSSOP-20, 0.65 mm pitch |
| Topology | 8-bit, bi-directional, **auto-direction** | 8-bit, bi-directional, **auto-direction** |
| Data rate | **110 Mbps push-pull**, 1.2 Mbps open-drain | 100 Mbps push-pull (no open-drain mode) |
| VCCA (SoM side) | 1.4–3.6 V (fix = 3.3 V) | 1.2–3.6 V (fix = 3.3 V) |
| VCCB (target side) | 1.65–5.5 V (fix = 3.3 V or 5 V) | 1.65–5.5 V |
| Direction control | None needed; OE for Hi-Z | OE; **VCC-isolation: if either VCC = GND, all I/O are Hi-Z** |
| I/O ESD | B-port **±8 kV contact IEC 61000-4-2** | B-port **±15 kV HBM** |
| Partial power-down (Ioff) | No | **Yes** (inputs remain Hi-Z when deck rail off) |
| IEC/body rating | Latches ≥100 mA JESD78 class II | Latches ≥100 mA |
| Source (verified) | TI product page + Rev. L datasheet `<https://www.ti.com/product/TXS0108E>` (also `https://www.ti.com/lit/gpn/TXS0108E`) | TI product page + Rev. L datasheet `<https://www.ti.com/product/TXB0108>` |
| Access date | 2026-08-30 | 2026-08-30 |
| Price & availability | **TBD** (distributor pages bot-blocked from this tool today; typical street ~€1–2 at qty in the TXS family — confirm at Phase-5/9 ordering) | **TBD** (same; ~€1–2 typical) |

**Selection rationale:**
- **UART is push-pull and direction is fixed per pin.** The TXS's auto-direction sensing is field-proven in hobby UART/SPI/I2C front-ends and removes the direction-control decision entirely. B-port ±8 kV inherent ESD is a useful belt-and-suspenders layer *behind* the external ESD array.
- **TXB0108 is the safer choice for the "deck asleep, target hot" case** because of its explicit **VCC-isolation** and Ioff features (verified in its feature list): if VCCA (deck rail) is off, the pogo-facing I/O goes Hi-Z, so a hot target cannot back-feed a 5 V logic level into an unpowered deck. If the power MCU's gate design cannot *guarantee* sequencing, **prefer TXB0108**. Both are drop-in TSSOP-20.
- These are 8-bit parts (8?1 UART uses 2). The surplus channels can be left floating (input side terminated) or reserved for a 2nd UART / RTS/CTS in the 6-pin variant — part-count stays at 1 IC.

> **TISO1800 (TI):** requested as a candidate; **could not be verified** — both `https://www.ti.com/product/TISO1800` (404) and TI search return nothing. Reasonable reading: the intended part is a TI *digital isolator* (e.g., **ISO7721** dual-channel, or ISO6742) for the galvanic-isolation study. **TBD until user confirms which part family was meant**; digital isolation is treated as optional §8.3.

### 4.2 VCCB selection (3.3 V / 5 V switch)

| Element | Recommendation | Notes |
|---|---|---|
| Source rails | Reuse **3V3 rail** and **5V rail** from the split-rail DC-DC (I-18) | Both already exist; no new converter needed |
| Selector | **MCU-controlled load switch** (e.g., tiny eFuse with fault flag — part TBD: TPS2041/2051-class or AP228x-class P-FET switch) | VCCB ≥ up to 5.5 V max on TXS/TXB B-port — the switch must be rated ≥5.5 V |
| Controller | Power-manager MCU (daughterboard) GPIO: `VCCO_SEL` (3.3/5), `PWR_OE` (enable/disable); see §7 | Part integral to daughterboard MCU assignment |
| Sequencing | **VCCA (3V3 SoM rail) must be present before VCCB is asserted** (VCCA ≤ VCCB spec) | Power MCU holds `PWR_OE` low until rails detected (`POWER_OK` from DC-DC) |
| Pull-ups | On the A port, the TXS family integrates pull-ups/open-drain support; for a **push-pull UART** no external pull-ups are required. Keep 0 Ω series default; add 10 kΩ pull-up only if a target uses open-drain TX (rare) | Terminal-pullups TBD at board bring-up |

### 4.3 Direction control

- **None needed**: TXS0108E/TXB0108 are auto-direction per-pin.
- The only "direction" control that matters is **OE (master disable)** → driven low by power-MCU in idle/off, pulled low via a weak pulldown on the PCB so the port defaults disabled even if the MCU is unprogrammed or asleep. Pull-down value per TI: "tie OE to GND through a pull-down; value depends on driver current capability" → use **10 kΩ** (safe for the ~µA-level OE input).

### 4.4 Fully discrete FET option — when to use

Classic 2-transistor auto-direction cell (SparkFun-style, one per line):
`BSS138` (N-ch, Vovres ≥30 V) + 2× **10 kΩ** pull-ups per line → 2 lines = 2× FET + 4× resistors; ~€0.20 total.

| Criterion | Discrete (BSS138×2) | TXS0108E/TXB (IC) |
|---|---|---|
| Part cost | ~€0.20 | ~€1–2 |
| Speed | ~2 Mbps class (edge-limited by 10 kΩ pull-ups) — **fine for ≤115 200 baud**, marginal >1 Mbps | 100+ Mbps |
| ESD / robustness | None inherent — MUST have external ESD array (loads the port head) | Inherent ±8 kV B-port + external array belts-and-suspenders |
| Hot-target / back-feed | FET body diode can back-feed in some configs — needs care | TXB's VCC-isolation is superior |
| Comms integrity | Edge rates poor on long cables | Strong drive, edge accelerators |
| User solderability | SOT-23 single FETs — very easy | TSSOP-20, 0.65 mm — moderate |

**When to use discrete:** (a) strict per-unit cost target, (b) baud never above 115 200 on short (<20 cm) cables, (c) user prefers a pure-SOT23 BOM, (d) second-source availability. **Primary recommendation remains the IC** for robustness; the discrete cell is the documented fallback, not the baseline.

---

## 5. Protection

### 5.1 ESD array (external, at the connector)

| Property | **TPD4E05U06 (primary; TI verified)** | USBLC6-2SC6 (ST; datasheet this tool could not load today — **TBD**) |
|---|---|---|
| Channels | 4 (one device protects TX+RX + spares) | 2-line low-cap array (2 data + rail pins) |
| IEC 61000-4-2 | **±12 kV contact / ±15 kV air** (level 4) | ~±8–15 kV class (per ST datasheet) |
| I/O clamp / Vrwm | 5.5 V working, 6.5 V min breakdown, ~10 V clamp @ 16 A 8/20 µs, 0.5 pF | LV-class, low capacitance (intended for USB lines) |
| Package | **USON-10 (DQA), 2.5×1.0 mm** — tight but hand-solderable with flux; SMT-assist at PCBWay also fine (B-006) | **SOT-23-6** (~3 mm) — very easy hand-solder |
| Typical | USB/HDMI-class, works perfectly as UART ESD clamp to GND | Works fine for UART; widely stocked |
| Source verified | TI product page + Rev. O datasheet `<https://www.ti.com/product/TPD4E05U06>` (access 2026-08-30) | ST `<https://www.st.com/en/protection-devices/usblc6-2sc6.html>` — unreachable from this tool today |
| Price | **TBD** (typical ~€0.2–0.5 street) | **TBD** (typical ~€0.2–0.4 street) |

Placement: **directly at the pogo terminals**, before (in series path of) the series resistors? No — *after* the series resistor closest to the connector so the resistor also limits ESD strike current into the clamp. Order from connector inward: **pogo pad → series R → ESD array to GND → translator B-port.** (The ESD array's cathode rail, if used, ties to VCCB through a small ferrite/resistor — or simply tie the common to GND for a GND-clamp-only topology, which is the safer choice for a multi-voltage port.)

### 5.2 Series resistors — short protection & current limit

Two series resistors (baseline), each in the external path: **330 Ω on deck-TX** and **330 Ω on deck-RX**.

Design derivation (worst-case short to GND at connector):
```
I_short(max) = V_target / R_series  = 5.0 V / 330 Ω ≈ 15 mA
```
- 15 mA is far below any level-shifter/SoM rating and below body-diode/ESD damage thresholds — a dead short of any pin (TX/RX/GND/+VCCO mishandled) cannot smoke a trace or the translator.
- Load clamp math (edge integrity vs loaded line capacitance, target baud):
```
t_rise ≈ R_series × C_line ;  require t_rise ≤ 0.1 × bit_period
C_line ≈ 50 pF (0.5 m cable) + ~5 pF I/O ≈ 55 pF
bit_period(115200) = 8.68 µs  →  t_rise ≤ 868 ns
R_allowed = 868 ns / 55 pF ≈ 15.8 kΩ max   (so 330 Ω is massively comfortable)
bit_period(1 Mbit/s) = 1.00 µs → t_rise ≤ 100 ns → R_allowed ≈ 1.8 kΩ  (still OK)
```
- **330 Ω** gives 5 V/330 Ω ≈ 15 mA fault current; **230–330 Ω (E24: 270/330) is the stated sweet spot** for console rates. If later the port must run ≥3 Mbit/s, drop to **47–100 Ω** and accept the higher short current (fine — translator pins handle >100 mA per JESD78) — decision stored as a test item, not now.
- Placement: R goes **between connector and ESD clamp** (per §5.1) so the resistor also serves as the ESD strike-current choke.

### 5.3 TVS / reverse-voltage handling

- The **ESD array is the TVS** for impulse (IEC 61000-4-2 ±12 kV). A separate bulk TVS is unnecessary at these speeds/capacitances.
- **Reverse-polarity cable** (user wires +5V of target rail into a UART pin, or swaps GND): handled by:
  1. **Series R**: 5 V into 330 Ω to the clamp = safe current, and 5 V ≤ Vrwm 5.5 V of the clamp → the clamp stays off; no stress.
  2. **Clamp-to-GND topology** (recommended, no VCCB rail tie): a reversed connection clamps against the GND rail only. No capacitive charge path back toward VCCB.
  3. **TXB0108 VCC-isolation** for the hot/unpowered case (§4.1).
- Explicitly **NOT** exposed on any pin: deck battery rail (14.4 V), 12 V rail. Only 3.3/5 V logic and (behind the safety gate) 3.3/5 V VCCO appear at the port.

### 5.4 Polyfuse (PPTC) recommendation

| Where | Recommendation | Rationale |
|---|---|---|
| **+VCCO power-out pin (SAFETY GATE)** | Bourns **MF-MSMF050-2** (0805): **0.5 A hold / 1.0 A trip, 13.2 V max**, R≈0.7–1.5 Ω. Or Littelfuse 1206L075-class (0.75 A hold) for extra current to USB-powered targets. **Part selection to be confirmed at ordering; price ~€0.2–0.4 TBD** | A target dead-shorting +VCCO must not exceed the load switch's rating; PPTC auto-recover after the cable is removed |
| Signal lines (TX/RX) | **No PPTC.** Use the series R (5.2) instead | A PPTC on a *logic* line is a poor fit: it adds 0.7–1.5 Ω series (harmless at these rates, but needless), and its latch is driven by *current*, which UART lines rarely reach meaningfully. The series-R already bounds fault current to ~15 mA — below any damage current. Adding a PPTC raises part count for zero benefit |

**Why the UART should not expose VCC by default (design rule):**
1. An always-live power pin on a magnetic connector is **the #1 cause of blown serial adapters**: users connect 5 V targets to 3.3 V systems or short power to GND when the connector rotates.
2. It violates the approved baseline ("safe default = signal + ground only").
3. It forces every user-built cable to become a *power* cable with its own current/voltage spec — complexity, liability, and failure modes (a reversed or wrong-spec cable can power a target from the wrong rail).
Default = **3-wire signal+GND port**. VCCO exists only as an explicitly enabled, fused, MCU-gated option after review (§8).

---

## 6. Behavior matrix

First column = connection scenario. "MCU" = power-manager MCU; `PWR_OE` = level-shift VCCB/OE enable; `VCCO` = target-power gate (§8).

| Scenario | What happens | Safe? | Expected result | MCU handling | LED (deck) |
|---|---|---|---|---|---|
| **3.3 V target, correct cable** | VCCO_SEL=3.3, PWR_OE=high → VCCB=3.3 V, OE on | ✅ | Bidirectional console works at 3.3 V | Assert rails in order VCCA→VCCB; set VCCO=3.3 | Green (linked) |
| **5 V target, correct cable** | VCCO_SEL=5 , PWR_OE=high → VCCB=5 V (≤5.5 ok) | ✅ | Console works at 5 V; deck SoM side stays 3.3 V | Set VCCO=5 before PWR_OE | Amber (5 V mode) |
| **Cable reversed (TX↔RX)** | Both are in-series-R + ESD paths; level translator sees no traffic either way | ✅ benign | No data (target talks into deck's TX path = fine; no contention since both driven by their own TX lines) | Keep enabled; log "no traffic" | Green; no error |
| **Logic levels reversed if VCCB mis-set** (5 V cable on 3.3 target or vice-versa) | If VCCB=5 but target is 3.3: B-port drives 5 V high onto a 3.3 V input → overshoot **through the 330 Ω** into a 3.3 V target pin (~5 mA) | ⚠ tolerable but must be flagged | Overshoot to a 3.3 V-only pin could exceed abs-max on *some* cheap MCUs | **MCU should demand the user confirm rail before enable** (supervisory GPIO or serial challenge); log loud | Amber + blink (V/mismatch hint) |
| **Short: TX→GND, RX→GND, TX↔RX** | Series R limits to ~15 mA; clamp holds | ✅ | Nothing burns; translator keeps operating | Detect via "no idle line" in SoM?/MCU CRC; keep OE low until user resolves | Red (fault detect) |
| **Short: +VCCO→GND (only if VCCO enabled)** | PPTC trips (0.5 A hold); load-switch fault flag (if eFuse used) | ✅ **gate-protected** | PPTC stays hot-trip until unplugged; auto-reset on removal | MCU reads FAULT, latches PWR_OE off, logs | Red |
| **Target-energized-hot** (target running, deck off/asleep) | VCCA=0 → **TXB: VCC-isolation → pogo I/O Hi-Z**; TXS (no Ioff): series R + clamp keeps damage away, but back-feed current via TXS I/O exists — mitigations: TXB priority, or R+VCCB sequenced | ✅ (TXB) / ⚠ (TXS: needs the R+clamp, safe at ≤15 mA) | No data until deck enables | PWR_OE stays low while deck asleep (default); rails not sequenced | Off |
| **Unpowered target** (deck on, target dead) | Target floats at 3.3/5 via nothing — TX line idle-high, RX sees nothing | ✅ | Single-ended console needs target GND common; unpowered MCU = no traffic, no current | Keep port idle; no VCCO | Green idle |
| **ESD / cable rub, hot-plugging** | ESD array clamps; series R limits | ✅ | No latch, no damage | No action (hardware) | — |
| **Wrong power on any pin** (user taps 12 V/14.4 V rail) | 12 V > clamp Vrwm 5.5 V → clamp conducts hard; series R 330 Ω → I≈36 mA into clamp; **12 V may exceed the translator's B-port abs-max even when clamped** | ❌ **unsafe if high rails are routable to the port** | Damage possible → **hard requirement: no rail ≥6 V may be routed anywhere near the port traces / foot of the ESD array** (keep-out on carrier/daughterboard layout) | N/A (layout rule, not software) | — |

### 6.1 Power-MCU supervision checklist (this is a *control* plan, not safety release)

1. **Rail presence first**: hold `PWR_OE` low until `3V3` and the selected `5V (or 3V3)` rail are stable (`POWER_OK`).
2. **Seq order**: VCCA (SoM 3.3) → VCCB (selected rail) → OE. Reverse at teardown: OE → VCCB → done.
3. **Supervisory GPIO**: sense cable presence via DET (or GND continuity on the innermost pogo pin); sense load-switch FAULT; sense port current (0.1 Ω shunt + MCU ADC, optional) — trips `PWR_OE` if shunt I > 30 mA for 100 ms (indicates a failed external short the PPTC/ESD didn't catch).
4. **Level confirmation**: before asserting, MCU requests user confirmation (1-wire handshake key, or a short "rail select" menu over the port itself) to reduce 5 V-into-3.3 V cases (§6 row).
5. **LED**: Green = enabled & idle/active; Amber = 5 V selected; Red = latched fault (requires re-enable, not auto-retry); Off = port disabled.
6. **Default on boot**: `PWR_OE=0`, `VCCO_SEL=3V3`, `VCCO_GATE=off` — the port boots electrically inert.

---

## 7. Powering the target — explicit gate (NOT construction-ready)

> ### **SAFETY GATE — REQUIRED PROCESS**
> The +VCCO option (Pin 4) is **off by design and unavailable until all of the following are true**:
> 1. Dedicated, fused, **user-switched** power path designed (load switch + PPTC + shunt-sense), reviewed by a qualified electrical/battery safety engineer (mirrors REQ-PWR-05 discipline).
> 2. **MCU + physical approval**: `VCCO_GATE` enable requires both a physical user action (e.g., chassis-accessible toggle / long-press combo) *and* firmware consent; never firmware-alone; never default-on.
> 3. Layout keep-out enforced: no battery/12 V rail nets within the UART front-end zone (§6 last row).
> 4. User sign-off on this document section + decision-log entry.

**Why the gate exists:** this is the difference between a *console cable* and a *power source*. Once the deck can drive 5 V (or 3.3 V) from its own battery into an arbitrary external gadget, three new incident classes appear: (a) a 3.3 V-only gadget fed 5 V, (b) a target shorting the deck's rail back into the BMS/hot-swap domain, (c) an unapproved adapter connecting deck battery power to a device that re-feeds it. Until (1)–(4) are satisfied, **the +VCCO pin shall be a no-mount / DNP placeholder**, and the shipped default remains 3-wire signal+GND.

**Recommended release parameters (once the gate passes):**
- Rail: 5 V or 3.3 V, switch-selected on the same VCCO_SEL as logic (matches target logic level).
- Protection chain, connector-inward: **Pin4 → PPTC (0.5 A) → load-switch (≤5.5 V rating, FAULT) → shunt (0.1 Ω) → rail** and a **bidirectional clamp at the pin** (same TPD4E05U06 channel) so a dead-short during hot-mate is clamped before the PPTC trips.
- Max specified target current: **≤500 mA** (PPTC hold), derated to **≤400 mA** design max for margin.
- Default cable must NOT wire Pin 4 unless the cable is explicitly labeled "POWER". Include a **keyed/profile detail** in the mechanical doc so a power-less cable cannot accidentally populate the power path (physically shorter pin-4 pogo, or detent).

### 7.1 Optional galvanic isolation (only if user wants it) — **TBD, do not build**
- If the user's threat model includes **large ground-potential differences / ground loops** on hot server COM ports, a **digital isolator** (TI **ISO7721** dual-channel, or ISO6742) on TX/RX plus a floating field supply (small isolated DC–DC) is the textbook approach. That isolated field-side supply is *power* — and therefore **joins the same SAFETY GATE as +VCCO**. Until confirmed, **design assumes a shared GND is acceptable** (99% of console use). Marked **TBD / not construction-ready**.

---

## 8. Weather and mechanical notes

- **Seal:** magnetic pogo heads are **not** waterproof. Provision: form-fit recessed port pocket behind a captive gasket + spring-loaded magnetic cover (or a simple silicone flap) in the rear I/O zone. Mate face should be below the chassis surface by ~0.5 mm so the latch face seals against the cable puck, not the chassis lip. (Detail owned by `mechanical/` docs.)
- **Retention:** magnets in the cable puck + steel/ferrite targets (or matching magnets) in the deck head; verify pull-off force 1.5–3 kg so the deck can host a hanging cable (panel-mount puck) without the port sagging, but releases before the user pulls the chassis.
- **Pogo plungers:** specify ≥1 mm travel, ≥10 000-cycle class; prefer **gold-plated** plunger/barrel; wiper wipe on each mate keeps the oxide off (that's the *point* of pogo vs plain pads for weather/oxidation).
- **Mechanical pointer:** see `mechanical/port-io-zone.md` ** (not yet created — TBD) for cutout, gasket, magnet layout, spring-pin keep-out; the electrical doc assumes: 2.54 mm pitch, heat-set/press-fit sockets acceptable, all six pads on 1×6 footprint, connector faces rear edge. Update this pointer when the mechanical doc lands.
- AliExpress pogo set (B-017, `1005005122336187`, €3.77) remains **low-confidence** (no datasheet). Phase-6 mechanical task: **verify travel/plating/force on a sample or substitute branded spring pins (Mill-Max 09xx-class) + separate magnets** before committing the electrical layout.

---

## 9. Test plan → requirements mapping

| Test | Method | Pass criterion | Requirement |
|---|---|---|---|
| T-01 3.3 V loopback | Pogo → benchtop 3.3 V UART loopback board; `stty 115200 raw -echo` + `cat > /dev/ttyUARTx` pulse | Round-trip bitstream exact, CRC-8 clean, BER 0 over 10⁶ bytes | REQ-UART-01 (3.3 V path) |
| T-02 5 V loopback | Same at 5 V rail target board | BER 0 over 10⁶ bytes | REQ-UART-01 (5 V path) |
| T-03 5 V into 3.3 V logic (mismatch) | Confirm MCU warns; confirm no damage after 1 h | Overshoot ≤3.6 V at 3.3 V target pin, no latch-up | REQ-UART-01 (protection) |
| T-04 Reverse cable | Swap TX/RX; run T-01 | No damage, no data, port stays functional after | REQ-UART-01 (robustness) |
| T-05 Dead shorts | Short TX/RX/GND combos each 30 s | No smoke, translator + rail intact; resume normal after clear | REQ-UART-01/02 |
| T-06 ESD gun | IEC 61000-4-2 ±8 kV, 10 strikes each pin into GND | No functional loss; clamp stays | REQ-UART-01 (protection) |
| T-07 Hot-target | Energized 5 V target, deck asleep, then wake | No back-feed current >1 mA (shunt); TXB Hi-Z verified | REQ-UART-01/02 |
| T-08 Hot-plug cycling | 500 mate/unmate cycles at 115200 stream | Zero glitch events that reset the deck (power or MCU); BER 0 | REQ-UART-02 (ease of use) |
| T-09 Unpowered target | Target rail off, deck on | No crash, no current beyond µA | REQ-UART-01 |
| T-10 +VCCO gate (only post-review) | Enable via MCU+physical; short VCCO to GND | PPTC trips, FAULT latches, no rail disturbance on deck | REQ-UART-01 (power gate) |
| T-11 1 Mbit/s integrity | 921600 baud loopback with 0.5 m cable | BER 0; edge τ ≤ 0.1 bit | REQ-UART-02 (server/console speeds) |

All T0x results recorded in `tests/` with date/rig; failures open a risk-register row (RISK-xxx) and revisit this doc.

---

## 10. Summary and open questions

**Summary of this design:**
- 6-pin magnetic pogo **proposal** (default = 3-wire TX/RX/GND); +VCCO pin reserved **disabled by default (SAFETY GATE)**.
- Level shift: **TXS0108EPWR** (primary) / **TXB0108PWR** (preferred if hot-target priority) auto-direction, VCCA 3.3 V fixed, VCCB switched 3.3/5 V by power-MCU via load switch; OE default-low keeps port inert. Discrete BSS138 cell documented as fallback.
- Protection: series 330 Ω ×2 + TPD4E05U06 (±12 kV IEC, GND-clamp) at the head; **no PPTC on signal lines**; PPTC (MF-MSMF050 0.5 A) only on the gated +VCCO path.
- Power-manager MCU owns enable/seq/supervise/LED; port boots dead.
- Test plan maps T-01…T-11 to REQ-UART-01/02.

**Open questions for the user (reply required before Phase-6 PCB work):**
1. **Pin complement:** 4-pin (TX/RX/GND) vs 6-pin (adds +VCCO, SPARE, DET)? Confirm the proposal or override. **(Pin-map is TBD until this.**
2. **TXS vs TXB:** do you prioritize cheaper/more-common TXS0108E, or the hot-target-safe (Ioff/VCC-isolation) TXB0108? Both fit.
3. **Target power:** keep +VCCO as an explicitly gated, DNP-until-reviewed option (recommended), or omit the pin entirely from v1 (strictest possible "signal+GND only")?
4. **Isolation:** was `TISO1800` a real part you meant, or is digital isolation (ISO7721-class) on the table at all? Assume shared-GND unless you say otherwise.
5. **Cable presence:** DET pin or magnet/Hall sensing — which for v1?
6. **Mechanical pointer (mechanical/port-io-zone.md)** — confirm it will be authored alongside this doc.

---

### References (URL + access date; per strict evidence rule)
- TI TXS0108E product + datasheet Rev. L — `https://www.ti.com/product/TXS0108E` / `https://www.ti.com/lit/gpn/TXS0108E` — accessed 2026-08-30.
- TI TXB0108 product + datasheet Rev. L — `https://www.ti.com/product/TXB0108` / `https://www.ti.com/lit/gpn/TXB0108` — accessed 2026-08-30.
- TI TPD4E05U06 product + datasheet Rev. O — `https://www.ti.com/product/TPD4E05U06` / `https://www.ti.com/lit/gpn/TPD4E05U06` — accessed 2026-08-30.
- TI "Isolating UART Signals" AN SLLA217 (for §7.1) — `https://www.ti.com/lit/pdf/sllt217` — accessed 2026-08-30.
- ST USBLC6-2SC6 — `https://www.st.com/en/protection-devices/usblc6-2sc6.html` — **TBD (page unreachable from this tool 2026-08-30; datasheet values unverified, marked TBD)**.
- Pogo magnetic set (B-017) — AliExpress `https://www.aliexpress.com/item/1005005122336187.html` — **low confidence; datasheet TBD; verify in Phase 6 mechanical**.
- All distributor price/stock (Mouser/DigiKey/TME/Arrow) — **bot-blocked from this tool 2026-08-30; prices TBD at confirmed ordering**.