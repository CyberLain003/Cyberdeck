# First-Order Feasibility (Phase 2)

Date: 2026-08-30. Status: **Provisional planning numbers only — no sourced component data yet**. All values are estimates with stated assumptions; nothing here is a guarantee. Component-level verification lands in Phase 4 (sourcing) and Phase 8 (measurement).

> `/info/feasibility.md` is an adaptation of the TASK.md skeleton tree: Phase 2 explicitly requires first-order feasibility work, and the skeleton had no dedicated file for it. No required coverage is lost; final verification lives in `electric`/`hardware` docs.

## 0. Assumption Summary (details in `info/assumptions.md`)

- Envelope: **130 × 170 × 50 mm** closed, interior ≈ 124 × 164 × 46 mm (3 mm walls).
- Cells: **21700**, Φ21.0–21.5 mm × 70 mm (nom. 65 mm + protection/wrap allowance), 3.6 V nominal.
- Packs: 2, each **4S1P** (4 × 21700 in series → 14.4 V nominal) = 4 cells/pack. (*See note 5.1 — 2S2P alternative.*)
- Cell classes considered: Molicel P45B (4500 mAh, 16.2 Wh), Samsung INR21700-50E (4910 mAh, 17.7 Wh), Molicel P50B-class (≈5000 mAh, ≈18.0 Wh).
- DC-DC/BMS conversion efficiency **90%** battery→rails; **aging + reserve margin 10%**.
- Workload (REQ-PWR-02): 20 h terminal/idle + 4 h web browsing + 6 h locked-idle = 30 h.

---

## 1. Envelope and Volume Budget

| Item | Estimate | Basis |
|---|---|---|
| Total closed volume | **1105 cm³** | 13 × 17 × 5 cm |
| Interior volume (≈3 mm walls) | ~934 cm³ | (12.4 × 16.4 × 4.6) |
| Battery subsystem (2 × pack slab 105×70×24 mm) | **353 cm³ (32% of envelope)** | 4 cells inline + PCB + connector + wall |
| Raw cells (8 × 24.3 cm³) | 194 cm³ (17.6%) | π/4·(21)²·70 mm each |
| Mainboard (est. 100×60×1.6 mm + components) | ~20–30 cm³ | carrier + SOM |
| Display + lid stack (8" panel + glass + backlight) | ~40–55 cm³ | panel ~2 mm + glass/lid |
| Keyboard + base decks | ~60–80 cm³ | membrane keys + decks |
| Remaining for gaps/interconnects/thermal pathways | remainder | Phase 6 |

**Finding:** Battery fits without dominating. The constraint moves to **z-height stack** (display+mainboard+keyboard+battery in 50 mm) and the **3–4 cm width sliver** beside the two packs. Not a blocker at box level.

---

## 2. Battery Geometry, Energy, Mass

Per pack (4 × 21700, inline):
- Slab: ≈ **105 × 70 × 24 mm** (4×21.5 + ~16 mm PCB/connector/wall, width 70 mm, height 24 mm).
- Two packs side-by-side across the width: **2×70 + internals (~16 mm) + walls (2×3) ≈ 16 cm < 17 cm** ✅ (matches user estimate ~16 cm).

| Configuration | Cell | Wh/cell | Wh/pack | Wh total | Pack mass (est.) | Total battery mass |
|---|---|---|---|---|---|---|
| 2× 4× P45B | 4500 mAh @3.6 V | 16.2 | 64.8 | **129.6** | ~290–300 g | ~580–600 g |
| 2× 4× 50E | 4910 mAh @3.6 V | 17.7 | 70.6 | **141.2** | ~305 g | ~610 g |
| 2× 4× P50B-class | ~5000 mAh | ~18.0 | ~72.0 | **~144** | ~300 g | ~600 g |

- ≥120 Wh met in all ×3, per-pack ≥60 Wh met in all ×3. ✅ (REQ-PWR-01 Pass at design level.)
- Pack-level energy density ≈ **360–410 Wh/L**, ~**220–240 Wh/kg** — plausible for 4S1P 21700 packs w/ BMS.
- Voltage **14.4 V nominal** (13.2–16.8 V range): low losses, modest currents (4.5 A full charge, ~0.3 A average load).

**5.1 Topology note (open for Phase 6):** 4S1P gives 14.4 V; 2S2P gives 7.4 V and doubles per-pack current. 4S preferred if any rail ≥12 V is needed (display backlight often needs 5–20 V; separate boost is fine). Retain as a decision item.

---

## 3. Per-State Power Budget (battery side, nominal)

Values are planning targets, not measurements. Display at "working" brightness (~150–300 nit effective), not full 1200-nit.

| State | Display | SoC/SOM | WiFi/BT | LTE | Converter+other | **Total (battery Wh)** |
|---|---|---|---|---|---|---|
| Locked / suspend (6 h) | off (or 0.3) | 0.8 | off | off | 0.3 | **~1.2 W** |
| Terminal / idle (20 h) | 1.8 | 1.5 | 0.2 | off | 0.3 | **~3.8 W** |
| Browsing (4 h) | 2.5 | 3.0 | 0.6 | 0 | 0.5 | **~6.6 W** |
| Radio-active (survey, peak) | 2.5 | 3.0 | 0.6 | 1.5–2.5 | 0.6 | **~8.7 W** |
| Charging + load (transient) | 2.5 | 3.0 | 0.6 | 0–2.5 | charging loss | **input 65 W / ~10–16 W internal** |
| Peak burst | — | 10–15 (boost) | — | — | 1 | **~15–18 W short** |

**Key:** the ~4 W average is dominated by the **display** (≈ half) and the **SoC idle**. Display choice is the single highest-leverage decision for runtime.

---

## 4. Runtime vs. Workload (REQ-PWR-02)

Energy consumed at the rail (system) per week-day workload:

- 20 h + 4 h + 6 h ⇒ **20×3.8 + 4×6.6 + 6×1.2 = 76 + 26.4 + 7.2 = 109.6 Wh (battery side)**.

Available vs required:

| Config | Cells Wh | ×90% conversion | ×90% aging/reserve | Margin over 109.6 Wh | Implied avg battery draw |
|---|---|---|---|---|---|
| 2× P45B (129.6) | 129.6 | 116.6 | 104.9 | **−4.3% (fails w/ margin)** | 4.3 W |
| 2× 50E (141.2) | 141.2 | 127.1 | 114.4 | **+4.4%** | 4.0 W |
| 2× P50B-class (144) | 144.0 | 129.6 | 116.6 | **+6.4%** | 3.9 W |

**Reading:**
- The workload is **just about feasible** with 50E/P50B-class cells **only if** the display stays ≤ ~2.5 W at working brightness and the SoC idles ≤ ~1.5–2 W. That points to an ARM-class SOM.
- **The 30 h target is the tightest requirement in the project.** Any display that draws >3 W at working brightness pushes it past the margin. If it cannot be met, the approved trade space is: (a) lower display brightness/panel, (b) accept ~24–26 h, or (c) larger cells. No changes made without user approval.
- Sensitivity: **+0.5 W display ≈ −2.4 h** over 30 h at 141 Wh.

---

## 5. Charge Time (65 W USB-C PD, REQ-PWR-06/REQ-PWR-04)

- Ideal: 141.2 Wh / 60 W (after ~7% charger+path loss) ≈ **2.4 h**.
- Realistic to full (CC/CV taper, thermal, ~90% bank efficiency, light simultaneous use): **~2.8–3.4 h** for both packs.
- Charging **while running** (system ~4–7 W): net ~50–55 W to battery → **~3.0–3.5 h** to full.
- External per-pack charging (optional): 72 Wh at ~20 W (5 V/3 A) ≈ **3.6–4.5 h**; faster only if a PD-capable cradle exists. Slower, as user expected.
- Early-prototype milestone ≈ **0–80% in ~2.2–2.5 h** (fast-charge window).

Per-pack and total charging currents are within 4S1P BMS practice; PD sink controller + charge path detail is Phase 6.

---

## 6. Thermal (active: micro fan + 5–7 mm heatsink on SOM; aluminum chassis)

- **User constraints (DEC-017/016/018):** micro fan + 5–7 mm heatsink over the SoM; black matte anodized aluminum chassis (structure-as-heatsink); ventilation via rear/side vents. Local CNC is free ⇒ apertures/cutouts are cheap (material + coating only).
- Surface area ≈ **0.074 m²** (box model). Matte-anodized aluminum adds conduction/spread + emissivity (~0.85–0.9).
- **Active continuous dissipation budget ≈ 10–14 W** (vs 6–8 W sealed/passive). Fan ~0.25–0.5 W continuous, PWM by power-manager MCU.

| State | Internal dissipation | vs. active budget |
|---|---|---|
| Terminal/idle | ~3.8 W | ✅ comfortable |
| Browsing | ~6.6 W | ✅ comfortable |
| Radio-active | ~8.7 W | ✅ within |
| Charging + use (65 W) | ~11–16 W | ⚠️ at/over ceiling — fan full + charge taper (RISK-016) |

**Finding:** active cooling + aluminum chassis largely resolves the previous sealed-thermal blocker; charge-while-use still needs fan-at-max + charge taper on temperature. Verify CFM/noise/power of the 25–30 mm fan in Phase 6/8.

---

## 7. Mass and Center of Mass (first-order)

| Subsystem | Est. mass |
|---|---|
| Battery cells (8×70 g) | 560 g |
| BMS/connectors/wiring (2 packs) | 60 g |
| Mainboard + SOM | 150–200 g |
| Heatsink (5–7 mm) + micro fan | 40–70 g |
| Display (8" panel + glass) | 150–170 g |
| Keyboard + trackball | 120–140 g |
| M.2 SATA SSD | 50–70 g |
| LTE modem + antennas | 30–40 g |
| Chassis (Al, CNC, base+lid+deck) | 400–550 g |
| Screws, cables, PD/charge HW | 100–130 g |
| **Total** | **≈ 1.6–1.9 kg** |

CoM note (front battery insertion, DEC-018): packs sit in the **front** of the base, so the base is now relatively **front-loaded**; the rear I/O/power daughterboard offsets it. Expect CoM ≈ **45–55% of base depth** from front — neutral-to-slightly-forward, acceptable for laptops with a rear hinge. Verify with CAD in Phase 6; no tip-back risk either way.

---

## 8. I/O & Bus Feasibility (first-order topology)

| Requirement | First-order path | Risk |
|---|---|---|
| REQ-COMP-03 M.2 2280 **SATA** | SOM w/ native SATA (e.g., i.MX8M Plus SoM) or PCIe x1 → SATA bridge chip (JMB/JMS) | Medium — SATA-capable SOM narrows candidate set; bridge needs 4L-compatible routing |
| REQ-IO-01 HDMI ≥1080p30 | SOM native HDMI (i.MX8MP has HDMI TX) or MIPI-DSI→HDMI bridge | Low-Medium; native HDMI is cleaner |
| REQ-IO-02 USB 3.x ≥1 Gbit/s effective | Dedicated USB 3 host port, no hub sharing on that port; 4–6L impedance-controlled traces | Medium — 5 Gbit routing on 4L is doable if short/controlled |
| REQ-IO-03 USB 2.0 OTG | OTG-capable host port | Low |
| REQ-IO-04 Eth ≥100 M | USB3 GbE (RTL8153-class) or native RGMII | Low |
| REQ-IO-05 Full-size SD | Carrier slot | Low |
| REQ-IO-06 USB-C PD 65 W | PD sink controller + high-current path | Medium; power path + PD negotiation |
| REQ-UART-01 pogo 3.3/5 V | Level shift + ESD + polyfuse + isolation; signal/gnd default | Medium; safety-focused design |
| REQ-RF-01 Wi-Fi/BT ≥6 MB/s | Any 802.11n/ac BT module + PCB/ceramic antenna | Low (throughput trivially exceeded) |
| REQ-RF-02 LTE/5G nano-SIM | USB/mPCIe Quectel-class + 2 antennas | Medium regulatory/cert |

**Carrier 4–6L note:** USB 3, PCIe/SATA, MIPI/HDMI differential pairs are all routable in 4–6 layers at PCBWay if layer stack + controlled impedance are planned and trace lengths are modest. Fine-pitch/BGA direct routing is excluded by user constraint (REQ-COMP-06) — SOM must land on a B2B connector with routable pitch. This is a **hard filter on SOM selection**.

---

## 9. Budget (first-order top-down, EUR, landed DE incl. VAT ≈ 19%)

Ranges are planning envelopes; **no quotes yet (Phase 4/5)**. FX assumption 1 EUR = 1 USD.

| Category | Est. range |
|---|---|
| SOM (compute + RAM) | 90–170 |
| Carrier PCB + assembly | 40–100 |
| M.2 SATA SSD | 55–90 |
| Display + driver/bridge | 80–130 |
| Keyboard + trackball | 30–80 |
| Battery cells (8) | 40–60 |
| Pack BMS/PCBs/connectors | 40–70 |
| PD charger + sink + charge path | 30–50 |
| Wi-Fi/BT module + antennas | 15–35 |
| LTE modem + antennas | 60–120 |
| UART pogo + protection | 15–30 |
| Heatsink (5–7 mm) + micro fan | 20–40 |
| Chassis **material + coating only** (CNC labor free, DEC-023) + purchased hinges | 60–150 |
| Misc (SD slot, HDMI/LAN/USB, screws, cables, test) | 50–100 |
| **Subtotal** | **≈ 660–1,045** |
| **Contingency 15%** | **~100–155** |
| **Total landed (incl. VAT, ship, conting.)** | **≈ 735–1,150** |

**Finding:** €1,000 remains a strong baseline and is **feasible** because the aluminium chassis is no longer a labor-cost driver (CNC free — only material/coating/hinges). Budget flexibility is approved for the shell (DEC-022). Remaining drivers: SOM, LTE, keyboard, panels. No change to the €1,000 baseline without approval, but headroom exists.

---

## 10. Findings, Hard Blockers, Trade Space

### Hard blockers (must be resolved in later phases)
1. **SOM must idle ≤ ~1.5–2 W and support long-term 4 W-average workloads** — likely ARM-class. x86 is expected to fail REQ-PWR-02. (Phase 3/4)
2. **Display ≤ ~2.5 W at working brightness** while ≥500-nits daylight-visible and 8" 4:3 — needs a low-power panel + driver. Highest-leverage part. (Phase 4)
3. **SATA-capable SOM or PCIe→SATA bridge** for REQ-COMP-03 within the B2B/6-layer constraint. (Phase 3/6)
4. **Hot-swap power bridge with pre-charge/inrush + safety review** (REQ-PWR-03, REQ-PWR-05). (Phase 6)

### Trade space (no requirements changed, offered for approval)
- Runtime vs. display/power vs. cell capacity (P45B / 50E / P50B).
- ~~Chassis material: aluminium vs printed/plastic~~ → **resolved: black matte anodized aluminum, CNC free locally (DEC-016/023)**.
- LTE as a later add-on to protect budget.
- Mechanical keyboard vs. membrane/low-cost keys.

### Provisional verdict
The concept is **feasible in principle**, the binder being battery+runtime+thermal, all of which resolve to **display power and SoC idle power**. Budget has flexibility for the aluminum shell (CNC free). Next: **Phase 3 — Architecture and options** (SOM families, display path, power topology, hot-swap approach, keyboard/trackball options, chassis strategy).