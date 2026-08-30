# Hardware — Thermal (Phase 6): Ducted left→right forced air path

Status: **Phase-6 thermal design draft — 2026-08-30.** Governed by **DEC-062 / A-040** (air path) and the freedom grant **DEC-063 / A-041** (minimal physical changes permitted to make the path fit inside 200×140×50). Uses the mechanical coordinate convention: **X = depth (front 0 … rear 140)**, **Y = width (left 0 … right 200)**, **Z = height (0 at floor)**. All figures marked **TBD** are unverified and must be confirmed on the Phase-8 thermal bench (RISK-016/027/028/029/030). No vendor P–Q curves are trusted blindly — this document derives an operating-point model and marks vendor-curve inputs **TBD**.

> Supersession note: the earlier "fan on top of the heatsink blowing down" construct (`mechanical/envelope-and-stack.md` §3.3, open item M-1) is **superseded by DEC-062/A-040**: the 30 mm fan is **in-plane to the left of the heatsink**, air flows **left→right (+Y)** through the fin channels, intaken at the left-back, exhausted at the right side. The z-stack impact is favourable (fan no longer stacks above the HS) and is captured in §6.

---

## 1. Air path and ducting (DEC-062, A-040)

### 1.1 Path statement

```
LEFT-BACK intake louvres (+)           RIGHT-side exhaust louvres (−)
  (Y≈0 wall + rear-corner, X≈105–137) ──► (Y=200 wall, X≈108–137, + weep)
        │                                        ▲
        ▼                                        │
 intake plenum (~8 mm deep)            exhaust plenum (~6–8 mm, labyrinth + weep)
        │                                        ▲
        ▼                                        │
30 mm fan (in-plane, draws LOW PRESSURE ──blows──► finned heatsink channels
at its inlet; outlet slot / face feeds the HS)   (base 1.5 mm + fins 3.5–5.5 mm;
        │                                        fins RUN +Y = left→right)
```

- The fan creates a **low-pressure zone** at its inlet: the intake louvres are upstream of the fan and the plenum feeds the fan inlet with outside air pulled through the left-back apertures.
- The fan outlet **pressurizes** the duct and forces air through the fin channels of the heatsink (fins parallel to flow, +Y), discharging into a small exhaust plenum and out the right-side louvres.
- The heatsink sits on the SoM in the right palm-rest column (mechanical §4.2: X≈108–137, Y≈120–180). The fan sits in-plane **left** of the HS so its outlet is collinear with the fin channels.
- Everything upstream must never be smaller than the fan inlet, everything downstream never smaller than the HS flow area, otherwise the duct becomes a nozzle and the operating point collapses.

### 1.2 Intake louvre sizing (free area for ~3.5 CFM at low ΔP)

Design flow for the louvre sizing is the **fan free-air rating** (3.5 CFM), so the intake never throttles the fan; the system operating flow is lower (§2).

| Qty | Symbol | Value | Formula |
|---|---|---|---|
| Design flow | Q | 3.5 CFM = **0.001652 m³/s** | 1 CFM = 0.00047195 m³/s |
| Target louvre face velocity | v | ≤ 2.5 m/s (keeps ΔP_grille < ~6 Pa) | empirical |
| **Required free area** | A_req | ≥ **Q / v max = 0.00066 m² ≈ 660 mm²** | A = Q/v |
| Louvres/vane open fraction | η_open | 0.55–0.65 (vane thickness, labyrinth, mesh, drain) | geometry |
| **Gross aperture** | A_gross | ≥ 660 / 0.60 ≈ **1100 mm²** | A_req / η_open |
| Suggested pattern | | 6 slots × ~1.8 mm × ~50 mm long, or 1 aperture 32 × 35 mm with internal labyrinth vanes | — |

ΔP_grille (minor-loss model, Engineering ToolBox in-duct loss K-factors, E-036):
```
ΔP = K · ½ ρ v²      K ≈ 1.2–1.4 (louvre + mesh + labyrinth + insect grid)
  = 0.5 × 1.3 × 1.2 × 6.25 ≈ 4.9 Pa ≈ 0.02 inH₂O      @ 2.5 m/s
```

This is deliberately < 5 % of the blower's rated static pressure (0.285 inH₂O ≈ 71 Pa) — the intake is **not** a system bottleneck by design.

Keep net slot area **≥ 0.5–0.7 × fan face (30×30 ≈ 900 mm²) → ≥ 500–600 mm² effective**, consistent with `mechanical/envelope-and-stack.md` §7.2. Slot width ≤ 1.8 mm (amp-protection), open area only where a wall is behind (no short-circuit into an internal cavity).

### 1.3 Intake plenum

- Volume between the louvre wall and the fan inlet, ≈ **40 × 30 × 8 mm (≈ 9 cm³)**, allowing the inlet velocity to settle and equalize over the fan face.
- Fan-to-wall standoff **≥ 3 mm** (noise + flow recirculation, consistent with mechanical §8 "grill-to-fan gap ≥ 3 mm").
- Plenum walls **≥ 2 mm** machined into the chassis/spine; no pinch below 1.5 mm anywhere.
- Routing to the fan given the geometry is a **mechanical coordination item** (§6.7): the mid-chassis spine (Y 92–102, X 3–137) currently separates the left-back corner from the right palm-rest column. Resolutions: a **spine pass-through duct** (milled slot ≥ 150 mm² in the spine at the rear, X≈110–137) or routing **over the spine top** (z 33–38 free volume above the spine plate). Either is a permitted "minimal physical change" (DEC-063/A-041).

### 1.4 Fan placement relative to heatsink

- Fan axis collinear with the fin channels (**+Y**), inlet faces the intake plenum, outlet faces the HS inlet face.
- **Axial (Sunon HA30101V4-1000U-A99):** both faces 30×30; outlet face 2–3 mm from the HS fin mouths; the fan funnels directly into the ~17 fin channels (contraction K≈0.5, E-036).
- **Blower (Delta BFB0305HA-C, recommended §2):** inlet = large face (faces intake plenum), **outlet = narrow peripheral slot (~30×5 mm ≈ 150 mm²)** which is near-ideal for feeding a fin channel stack (laptop-style). The slot is already shaped to blow across compact fin rows; align the slot with channel mouths, ≤ 2 mm gap.
- Seal the fan→HS interface with closed-cell foam surround so air cannot bypass the fins (bypass is the #1 airflow kill in small ducted systems; measured leak check in §5).

### 1.5 Fin orientation

- Fins run **parallel to +Y** (left→right), NOT along X. Channel length = HS extent in Y (38 mm); width across flow = HS extent in X (32 mm). Fins are continuous straight channels (no interrupted/staggered fins in a 38 mm run — pressure budget is tight).
- Duct walls (top shroud over fins + side rails) force all delivered flow through the channels; shroud gap over fin tips ≤ 0.5 mm (this gap is the dominant bypass if loose).

### 1.6 Exhaust louvre sizing + drain/weep

| Qty | Value |
|---|---|
| Design exhaust flow | 1.00 CFM (blower full free-air ceiling) = 0.000472 m³/s |
| Free area @ ≤ 2.5 m/s | ≥ 0.00019 m² = **≥ 190 mm²**; spec **≥ 350–400 mm²** (asymmetric convenience + hygiene) |
| Gross aperture | ≈ 950 mm² (e.g., 38 × 18 mm), open ≈ 0.42–0.55 after labyrinth |
| Location | Right wall (Y = 200), **X ≈ 108–137**, in front of the hinge barrels; coordinate with high-speed port wall split (mechanical M-3) |
| Vane direction | Vanes sloped **down-outward**; labyrinth oil-can profile so water cannot jet straight in (DEC-056, A-034) |
| Drain/weep | Low-point **weep slot to the exterior at the right bottom edge** (≈ 3 × 15 mm, one-way flap); any rain caught in the exhaust plenum exits before reaching the HS |

Intake side gets the same treatment: **downward/backward-facing intake louvres**, internal drip tray (2° slope), low-point weep to the left-bottom edge, plus a replaceable filter pocket (1 mm open-cell foam) upstream of the fan (dust, RISK-029). Both vent plena drain before the fan/HS — consistent with mechanical §7.2.

### 1.7 Ducting summary (dimensions)

| Element | Spec |
|---|---|
| Intake louvre free area | ≥ 660 mm² (600–700) @ ≤ 2.5 m/s |
| Intake gross aperture | ≈ 1100 mm², vane slots ≤ 1.8 mm |
| Intake plenum | ≈ 40 × 30 × 8 mm, fan standoff ≥ 3 mm |
| Spine / routing duct | ≥ 150 mm² cross-section, walls ≥ 2 mm, no pinch < 1.5 mm |
| Fan→HS gap | ≤ 2 mm + foam seal surround |
| HS flow area | 17 ch × 1.4 mm × 3.5 mm ≈ **83 mm²** (§3) |
| Exhaust plenum | ≈ 6–8 mm between HS outlet face and right wall |
| Exhaust louvre free area | ≥ 350–400 mm², gross ≈ 950 mm² |
| Drip/weep | both plena, 3 × 15 mm flap weep to bottom edges |

---

## 2. Fan sizing and operating point

### 2.1 Sourced fan data (candidates §7, E-022/E-023/E-033/E-034, access 2026-08-30)

| Part | Type | V | Free-air | Max static pressure | Noise | Power | PWM/tach |
|---|---|---|---|---|---|---|---|
| **Sunon HA30101V4-1000U-A99** | 30×30×10 axial | 5 V | **3.5 CFM** | **TBD 5 V**; 12 V sibling HA30101V3-1000U-A99 (7000 RPM) documented at **0.10 inH₂O** (E-033). 5 V assumed ≤ ~0.08–0.10 inH₂O | 15.1 dBA | **0.30 W** | −A99 auto-restart; no PWM/tach (low-side PWM or voltage control needed) |
| **Delta BFB0305HA-C** | 30×30×10 blower | 5 V | **1.45 CFM** | **0.285 inH₂O (71 Pa)** (E-022/E-034) | 29 dBA | **0.65 W** | 2-wire; no tach (-C config). Low-side PWM / voltage control **TBD compatibility** (RISK-027) |

Vendor P–Q curves: Delta publishes curve anchors in the datasheet; Sunon does not publish a P–Q curve for the 5 V part (only free-air + max static of the 12 V sibling). **Both curves are treated as inputs to a model, marked TBD, verified on bench (§5).**

### 2.2 Duct + heatsink system curve (Hagen–Poiseuille laminar fin model)

Fin channels (HS-5 geometry §3): 17 channels, gap 1.4 mm, height 3.5 mm, length 38 mm, Dh = 2.0 mm, flow area A_f = 83.3 mm². Laminar parallel-plate friction `f = 96/Re` gives:

```
ΔP_fin(Q) = (96 ν L)/(Dh²) · v · ½ρ   =  (48 ρ ν L)/(Dh² · A_f) · Q
          ≡ C · Q,  C ≈ 98,500 Pa·s/m³   (linear, laminar)
Grilles (minor-loss, E-036):  ΔP_grille = K · ½ρv²  (K≈1.2–1.3)   → ≤ ~0.02 inH₂O each, negligible
```

| Q | v_channel | Re | ΔP_fin | + 2 grilles |
|---|---|---|---|---|
| 1.45 CFM | 8.2 m/s | 1095 | 0.271 inH₂O | ~0.30 inH₂O |
| 1.00 CFM | 5.7 m/s | 756 | 0.187 | ~0.21 |
| **0.76 CFM (blower op. pt.)** | **4.3 m/s** | **574** | **0.142** | **~0.16** |
| 0.50 CFM | 2.8 m/s | 378 | 0.093 | ~0.11 |
| 0.40 CFM | 2.3 m/s | 302 | 0.075 | ~0.09 |

**Typical fin stack pressure loss** for 5 mm-high, 1.4 mm-gap aluminium fins at these velocities is **0.08–0.27 inH₂O**, dominated by laminar friction; entry/exit losses add a few % (E-036). This is the conservative pressure-drop assumption used below.

### 2.3 Operating points (linear-curve fan model; TBD vendor curve)

Fan curve approximated linear between free-air Q and max static `Q = Q_m · (1 − ΔP/P_max)`; intersect with system curve `ΔP = C·Q`:

| Fan | Assumed max static | **Operating Q** | **Operating ΔP** |
|---|---|---|---|
| **Delta BFB0305HA-C (blower)** | 0.285 inH₂O (datasheet) | **≈ 0.74–0.80 CFM** | **≈ 0.14–0.16 inH₂O (35–40 Pa)** |
| Sunon HA30101V4 (axial) | 0.10 inH₂O (12 V sibling, optimistic for 5 V) | ≈ 0.46 CFM | ≈ 0.087 inH₂O |
| Sunon axial, conservative | 0.08 inH₂O | ≈ 0.38 CFM | ≈ 0.075 inH₂O |

**Reading:**
- The axial's 3.5 CFM free-air rating is **irrelevant inside this duct** — the channel pressure loss chokes it down to ~0.4–0.5 CFM, i.e. **~12–14 % of free air**. The duct is a *high-pressure-loss path*; axial fans stall on such systems.
- The blower has **1.9–2.0× the working static pressure** and delivers **~0.75 CFM ≈ 1.6–1.9× the axial's real flow** in the same duct. Air-carrying capacity (below, §2.5) is the decisive metric.

### 2.4 Recommendation

> **Primary: Delta BFB0305HA-C blower (DEC-028 revisit; supersedes Sunon-as-leading for the DEC-062 ducted path).** The Sunon HA30101V4-1000U-A99 is retained only as a fallback for a **low-restriction** arrangement (e.g., if the heatsink is replaced by an open fin field with no duct, where its 3.5 CFM free-air and 15.1 dBA matter). Costs of the blower: +0.35 W fan power, +14 dBA (29 vs 15.1), fan-mounting tolerance for the side-slot outlet. Both shift DEC-028 → new decision requested.

### 2.5 Efficiency / heat to dissipate at operating CFM

Blower at 0.75 CFM ≈ 0.000354 m³/s @ ~37 Pa:
```
fluid power  P_f = Q · ΔP = 0.000354 × 37 ≈ 0.013 W
motor in     P_in = 0.65 W                                  →  blower efficiency ≈ 2 %  (typical micro-blower 2–8 %; TBD curve)
```
Fan waste (~0.65 W) is internal heat and must be added to the thermal budget at the fan's location (radiated/convected into the duct air, exits right). Net heatsink-air heat to dissipate at operating CFM = **state dissipation (SoM share) + 0.65 W (blower) / + 0.30 W (axial)**.

Air mass flow and carrying capacity (this is what really matters):

| Fan @ op. pt. | ṁ | Air-carried with ΔT_air ≤ 20 K | ≤ 25 K |
|---|---|---|---|
| Blower 0.75 CFM | 0.42–0.43 g/s | **8.7 W** | **10.8 W** |
| Axial 0.40–0.46 CFM | 0.23–0.26 g/s | 5.1–5.3 W | 6.4–6.6 W |

`P_carried = ṁ · cp · ΔT_air; ṁ = ρ·Q`. The blower just covers the **radio 8.7 W** state at ΔT_air ≈ 20 K; the axial does not. Charge+use (§4) relies on taper + splitting heat to the chassis, not on pushing all 11–16 W through the fins.

---

## 3. Heatsink design (left→right fins)

### 3.1 Geometry (within DEC-017 5–7 mm)

| Parameter | HS-5 (baseline) | HS-7 (7 mm, if M-2/≤55 mm ceiling) |
|---|---|---|
| Footprint | 32 (X, across flow) × 38 (Y, along flow) mm | same |
| Base | 1.5 mm (6061-T6, into fan duct) | 1.5 mm |
| Fin height | 3.5 mm | 5.5 mm |
| **Total height** | **5.0 mm** | **7.0 mm** |
| Fin pitch / thickness / gap | 1.8 / 0.4 / **1.4 mm** | 1.8 / 0.4 / 1.4 mm |
| Fin count | 17 | 17 |
| Fin length (channel) | 38 mm (along +Y) | 38 mm |
| Material / finish | 6061-T6, black matte anodize (ε≈0.88, E-035) | same |
| Flow area A_f | 83.3 mm² | 131 mm² |
| Wetted surface | ≈ **56 cm²** | ≈ **82 cm²** |

Fins continuous, running +Y; denser pitch (≤1.2 mm gap) rejected — it collapses the blower operating point (ΔP_fin ≈ ∝ Q with 1/gap³ in laminar parallel plates). Fin efficiency η_fin ≈ 0.99 (5 mm) / 0.98 (7 mm) at h≈55–65 W/m²·K.

### 3.2 Rθ estimate (laminar forced convection + spreading)

```
Rθ_HS ≈ R_base_cond (≈0) + R_cv + R_spread(+TIM/pkg, 1.3–2.3 K/W TBD)
R_cv = 1 / (h · A_wet · η_fin)
h(laminar) ≈ Nu · k_air / Dh,  Nu ≈ 4–6 (developing), k_air 0.026:  h ≈ 52–78 W/m²·K
    → R_cv ≈ 2.4 (HS-7) … 4.4 (HS-5) K/W
⇒  Rθ_HS ≈ 5–7 K/W (HS-5)  /  ≈ 3.8–5 K/W (HS-7) ... conservative mid-range Rθ ≈ 6.2 / 4.3 K/W
```
Fan-off (no forced channel flow): Rθ_pas ≈ **9–12 K/W** (natural convection on the micro fin field + radiation coupling to the black chassis, ε≈0.88) — usable only for ≤ ~3 W SoM share (idle).

### 3.3 ΔT SoM → ambient per state (25 °C ambient; P = SoM-share-of-state, see §4 for shares)

| State | Total P | SoM share | Fan ON (blower, Rθ≈6 K/W HS-5) | Fan/duct OFF (Rθ≈11 K/W) | Right-wall exhaust air | Skin note |
|---|---|---|---|---|---|---|
| Idle 3.8 W | 3.8 | ~1.7 W | ΔT ≈ 10 K → HS ≈ 35 °C | ΔT ≈ 19 → 44 °C | +4 K | ✅ ≤45 target, even passive |
| Browse 6.6 W | 6.6 | ~3.0 W | ΔT ≈ 18 → **43 °C** | ΔT ≈ 33 → 58 °C | +7 K | ✅ forced; passive ✗ |
| Radio 8.7 W | 8.7 | ~5.0 W | ΔT ≈ 30 → **52–55 °C** (borderline vs 45–50 skin) | ✗ 80 °C | +12–15 K | ⚠ needs HS-7 + 100 % fan + spread to deck |
| Charge+use 11–16 W | 11–16 | ~5 W (chip) + ~3–6 W (charger/conv on DB->chassis) | ΔT ≈ 30 (HS) + charger zone spread on chassis | ✗ | +25–37 K *if all on air* | ⚠ only with taper (§4.3) + HS-7; taper keeps 16 W from being sustained |

Formulas: `T = T_amb + Rθ · P_som`; exhaust air `ΔT_air = P_air / (ṁ·cp)` with blower ṁ ≈ 0.43 g/s.

**Worst-case caveat (why the small HS cannot be a single heat sink):** if *all* of the state power were forced through the 32×38×5 mm HS, browse=6.6 W → ΔT 41 K (66 °C), radio 8.7 W → 54 K, charge 16 W → ~100 °C — all failing the 45–50 °C skin target. The design therefore **splits heat** (§4): the finned HS takes only the **SoM share** (≤ ~5 W continuous), while the **black anodized chassis box (~0.074 m² exterior, ε≈0.88, E-035)** takes the display/backlight, converter, charger, LTE remainder by conduction + natural convection + radiation. This split, the blower's air-carrying capacity, and charge taper together close the budget.

### 3.4 Surface-skin targets

- Palm-rest deck over HS + right exhaust wall: **≤ 45 °C preferred / ≤ 50 °C hard** at 25 °C ambient, per-user. Couple HS base to the deck through a **conductive gap pad / machined boss** so local hot spots spread before reaching skin; deck underside over the HS gets a 2 mm boss or TIM.
- Battery packs (front): keep ≥ 4 mm air gap; packs ride rails (mechanical A-038) — pack NTC governs charge taper (§4.3).

---

## 4. Thermal budgets, fan profile, charge-while-use taper

### 4.1 Per-state budget (internal dissipation split, battery-side numbers from feasibility §3)

| State | Total internal | → SoM HS (finned) | → Chassis box (conv + rad, ε 0.88) + lid | Fan phase (blower) |
|---|---|---|---|---|
| Locked/idle 3.8 W | 3.8 | ~1.7 W (SoC idle+screen) | ~1.8 W (backlight idle, lid) + conv | **Off** (below 38 °C) |
| Browse 6.6 W | 6.6 | ~3.0 W (SoC) | ~2.5 W (BL) + ~0.8 W conv | 40–60 % |
| Radio 8.7 W | 8.7 | ~5.0 W (SoC + radio) | ~2.5 W (BL) + ~1.2 W (LTE/conv) | 75–100 % |
| Charge+use 11–16 W | 11–16 | ~5 W (SoC) * | ~5–9 W (charger/PD/conv on DB) + BL | **100 %** + taper (BQ25713) |
| Peak burst | ≤ ~18 W short | ≤ ~8–12 W on HS (turbo) | rest | **100 %**, PROCHOT / boost guard |

\* With taper active, overall falls to ~11–13 W sustained; the 16 W transient is allowed (thermal mass) but not sustained.

Heat to dissipate per state at operating CFM = **state power + 0.65 W (blower motor)**, per §2.5.

### 4.2 Fan RPM/PWM profile (power-MCU ramp, RISK-016)

Fan controller = power-manager MCU (STM32G0) on the rear daughterboard; inputs = BQ25713 die temp (I²C), BQ76920 pack NTC, HS-base NTC (added sense), fan tach (if available; RISK-027), optional SoM thermal diode. Blower is 5 V, 2-wire: **low-side 20–25 kHz PWM** or 3.3–5 V analog; speed-vs-duty linearity **TBD** (RISK-027).

| HS-base temp (NTC) | PWM (duty) | RPM est. | Action |
|---|---|---|---|
| < 38 °C | 0 % | off | Silent below idle requirement |
| ≥ 38 °C | 25 % | ~stall-avoid floor | ramp-in |
| ≥ 42 °C | 50 % | mid | browse |
| ≥ 48 °C | 75 % | high | radio |
| ≥ 55 °C | **100 %** | max | + PROCHOT (SoM throttle); then §4.3 taper |
| Hysteresis | 3 K | | anti-flutter |

Profile rules:
- **Startup:** force 100 % for ~0.4 s (break blower static friction), then settle to setpoint; slew ≤ ±5 %/s (acoustic + inrush).
- **Tach watch:** if no tach pulses within 2 s of a "run" command → fan-fault: force max thermal margin, warn, and **reduce charge current** (§4.3) so the system survives fan-off until service (RISK-027 fallback).
- **Fail-safe:** if fan PWM signal lost → watchdog asserts full-on (fails to full speed, not off).

### 4.3 Charge-while-use taper coordination (RISK-016, BQ25713)

BQ25713RSNR has **on-die thermal regulation** (auto-reduces charge current at die-temperature threshold) and a **PROCHOT output** (SoM throttle). MCU policy layers on top:

| Priority | Trigger | Action |
|---|---|---|
| 1 | Pack NTC (BQ76920) charge window 0–45 °C | Hard charger limit/cut per Molicel G-cell spec (already in pack design) |
| 2 | HS-base NTC ≥ 55 °C AND fan at 100 % | MCU lowers BQ25713 ChargeCurrent via I2C: 4.0 → 3.25 → 2.0 → 1.0 A (0.5–1 min steps), targets T_HS < 52 °C |
| 3 | BQ25713 die thermal regulation | IC-level current fold-back (independent of MCU) |
| 4 | SoM PROCHOT / kernel throttle | CPU governor clamps bursts; reduces SoM share |
| 5 | Skin ≥ 50 °C (exhaust wall NTC) | Reduce ChargeCurrent one step more; if still ≥ 50 → alert user (accept heat, or pause charge) |

Effect: 65 W charge+use enters **no sustained 16 W window**; sustained internal ≈ 11–13 W with ΔT_air (blower) ≈ 25–30 K and right-wall skin ≤ ~50 °C. This is the RISK-016 mitigation made concrete: fan-max + taper-on-temp, both automatic.

---

## 5. Verification (Phase 8 thermal bench, RISK-016/027)

### 5.1 Test points (K-type / T-type 0.5 mm thermocouples, + IR)

| # | Point | Purpose |
|---|---|---|
| T1 | SoM SoC top / HS base center under die | die-representative (via Rθ) |
| T2 | HS base edge (non-die) | spread asymmetry |
| T3 | Fin outlet (fin #8, discharge face) | channel ΔT + bypass check |
| T4 | Fan inlet air | intake air temp |
| T5 | Exhaust louvre discharge air (right wall) | exit ΔT_air |
| T6 | Palm-rest deck skin over HS | skin target |
| T7 | Right wall skin at exhaust | skin target |
| T8 | Rear daughterboard charger zone (BQ25713 + converter) | charge-state heat |
| T9 | Pack NTC (front band) + bay ambient | taper trigger |
| A1 | Ambient, ≥ 0.5 m away | reference |

### 5.2 Pressure / flow measurement

| Instrument | Spec |
|---|---|
| Differential manometer | ±0.01 inH₂O resolution; taps: intake plenum, fan inlet, HS inlet, HS outlet, exhaust plenum (ΔP across louvres, across fins) |
| Velocity | hot-wire anemometer 0.2–20 m/s at exhaust louvre; integrate across slots → Q |
| Fan speed | tach (add a sense magnet/3rd wire if 2-wire part lacks it, or optical) |
| RPM/PWM sweep | measure Q, ΔP, noise, temp at 0/25/50/75/100 % duty per state |

Also: **bypass smoke test** (cured aerosol) around fan→HS surround and shroud gap; fix duct leaks until smoke only exits right louvre. Then record P–Q curve model vs measured (validates §2.3 — vendor curves TBD).

### 5.3 Risk table (additions to risk-register.md)

| Risk | Item | Mitigation | Review |
|---|---|---|---|
| RISK-027 | Delta -C 2-wire: no tach/PWM pin → speed control + fan-fault detection path **TBD** | low-side PWM + external tach (Hall/magnet), or swap to a Delta/Sunon variant with 3rd wire; fan-fault → charge taper | Phase 6/8 |
| RISK-028 | Blower noise 29 dBA (vs 15.1 axial) at palm rest | ducts damp fan noise; PWM slew; right-side discharge; bench acoustic target ≤ ~32 dBA @1 m sustained, TBD user tolerance | Phase 8 |
| RISK-029 | Dust / debris clogging 1.4 mm fin channels | replaceable 1 mm filter pocket upstream; slot ≤ 1.8 mm; service = pull filter (mechanical serviceability) | Phase 8 |
| RISK-030 | Vendor P–Q curves unpublished (Sunon 5 V) / operating-point model unverified | §5.2 bench measurement replaces model; blower chosen partly for guaranteed higher static | Phase 8 |
| RISK-016 (existing) | Sustained 65 W charge+use | resolved-as-designed here: blower max + taper + heat split + HS-7 option | Phase 8 |

### 5.4 Acceptance criteria

| Criterion | Pass |
|---|---|
| Skin (deck over HS, right exhaust wall) ≤ 45 °C pref / ≤ 50 °C hard | at 25 °C amb, states: idle, browse, radio (fan profile §4.2) |
| Skin during charge+use | ≤ 50 °C with fan 100 % + taper active (§4.3); transient 16 W allowed |
| SoM junction (via T1 + Rθ cal, or on-die sensor) | ≤ 85 °C sustained, ≤ 105 °C transient |
| Exhaust air ΔT | ≤ +20 K @ radio, ≤ +30 K @ charge+use (taper) |
| Pack NTC during charging | within Molicel 0–45 °C window (charge) at all times |
| Fan-fault behaviour | charge tapers + warning; system survives fan-off at browse state |
| Duct integrity | ≥ 90 % of fan flow exits right louvre (bypass smoke / anemometry) |
| Acoustics | ≤ ~32 dBA @ 1 m sustained (user tolerance TBD, RISK-028) |

---

## 6. Minimal physical changes required (DEC-063 / A-041 freedom to fit 200×140×50)

1. **Fan position — in-plane, left of HS** (was "on top blowing down", mechanical M-1). Decouples fan from the HS z-height; the z-stack closes at 50 mm nominal for HS-5 (fan at z 25–35 beside the HS, both under the deck).
2. **Right-wall exhaust louvre**: enlarge to **≈ 38 × 18 mm gross (~950 mm², ~400 mm² free)** at Y=200, X≈108–137 (was implicit palm-rest grill exhaust); must be carved in the machined right wall — free CNC (DEC-023), coating only.
3. **Left-back intake aperture + plenum**: ≈ 32 × 35 mm gross (~1100 mm²) on the left-back/rear corner, with an ~8 mm internal plenum sculpted into the rear-left chassis volume (X 108–137, Y 0–~30) — new machined pocket.
4. **Duct depth/cross-section**: intake-routing duct and the fan→HS insert need ≥ 150 mm² cross-section, walls ≥ 2 mm (§1.7). This is the single most space-costly item (the rear-left zone is otherwise empty above Bay A's end at X 108, so low cost).
5. **Spine pass-through (likely)**: a **milled slot through the mid-spine (Y 92–102) at X≈108–137** (or over-spine routing above z 33) to carry intake air from the left zone into the palm-rest column — **flagged mechanical coordination item, TBD routing** (§1.3). Alternatively intake louvres could be placed on the left wall at X 96–137 to feed the fan more directly, accepting a shorter outside-air path.
6. **Heatsink**: 32 × 38 mm, 5 mm nominal (HS-5) / **7 mm (HS-7) if the SoM B2B stack measures tall or the ≤ 55 mm ceiling is approved** (mechanical M-2). Base boss + TIM to SoM; conductive coupling pad to deck for skin spread (§3.4).
7. **Weep/drain geometry**: two weep slots (3 × 15 mm) to the bottom edges (left intake, right exhaust) + drip trays — added to mechanical §7 part list.
8. **No envelope growth needed at HS-5**; HS-7 may raise the palm-rest deck to ≤ 55 mm (already approved ceiling, DEC-043/M-6).

---

## References (access 2026-08-30; vendor curves TBD on bench)

| ID | Source | Claim | URL |
|---|---|---|---|
| E-022 | Octopart (Delta), BFB0305HA-C datasheet | Blower 30×30×10, 5 V, 1.45 CFM free-air, 0.285 inH₂O max static, 29 dBA, 0.65 W | https://www.delta-fan.com/Download/Spec/BFB0305HA-C.pdf |
| E-023 | Octopart (Sunon) HA30101V4-1000U-A99 listing | Axial 30×30×10, 5 V, 3.5 CFM, 15.1 dBA, 0.3 W | https://www.sunon.com/Product/ |
| E-033 | Mouser / sunonusa spec: HA30101V3-1000U-A99 (12 V sibling) | 3.5 CFM, **0.10 inH₂O static**, 7000 RPM, 15.1 dBA, 0.3 W — proxy for 5 V static pressure (TBD) | https://www.mouser.com/en/ProductDetail/Sunon/HA30101V3-1000U-A99 / https://www.sunonusa.com/ |
| E-034 | delta-fan spec PDF + E-022 | Blower P–Q anchors: (0.285 inH₂O, 0 CFM) ↔ (0, 1.45 CFM); curve linear model | https://www.delta-fan.com/Download/Spec/BFB0305HA-C.pdf |
| E-035 | Engineering ToolBox emissivity table | **Anodize black ε ≈ 0.88**, Aluminium anodized 0.77 @ ~300 K | https://www.engineeringtoolbox.com/emissivity-coefficients-d_447.html |
| E-036 | Engineering ToolBox minor-loss / duct loss refs | Minor-loss (K) model for louvres, contractions, expansions; laminar duct friction | https://www.engineeringtoolbox.com/minor-pressure-loss-ducts-pipes-d_624.html |

Formula basis: laminar parallel-plate friction `f = 96/Re` (Hagen–Poiseuille), Darcy–Weisbach minor losses, forced-convection `Nu ≈ 4–6` developing laminar channel correlation, Stefan–Boltzmann radiation with ε = 0.88. **All are first-principles estimates pending the §5 bench; numbers carry ±30–50 % engineering uncertainty (TBD).**

Open items carried: DEC-028 fan recommendation update (axial → blower); mechanical M-1/M-2/M-6 re-derive; RISK-016/027/028/029/030 gate at Phase 8.