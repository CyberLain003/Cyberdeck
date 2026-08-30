# Cyberdeck

A compact, fully functional custom cyberdeck — designed, engineered and documented as an open, evidence-driven build. Target: a rugged, rainproof, ThinkPad-class palmtop/laptop with a bright daylight-visible screen, 30 h of battery life, and true hot-swap power.

Everything here is a **living engineering record**: requirements, feasibility math, sourced component evidence, BOM + landed-Germany cost model, electrical/mechanical designs, and risk tracking. No purchases are made by this repo — procurement recommendations are staged and gated.

---

## The machine in one line

**≈ 200 × 140 × 50 mm clamshell** · 8" 800×1280 @ 1125 nits MIPI-DSI display · Verdin i.MX 8M Plus (8 GB) on a custom carrier · dual 4S 21700 packs (≥120 Wh) with drill-style **hot-swap** battery rails · 30 h runtime target · NixOS · passive-legal wireless surveying workhorse.

## Key specs (current design state)

| Area | Spec |
|---|---|
| Compute | Toradex Verdin i.MX 8M Plus, 8 GB LPDDR4, 4× Cortex-A53 @ 1.6 GHz — mainline Linux/U-Boot, **NixOS-friendly (no `config.txt`)** |
| Display | **Raystar RFU800G-AYH-MNN** — 8" 800×1280, MIPI-DSI 4-lane, **1125 nits** (daylight-visible), 189 ppi |
| Storage | M.2 2280 NVMe (M-key, PCIe Gen3 ×1) — silkscreen-keyed so only the right drive fits |
| Battery | **2× 4S 21700** (Molicel P50B), ~144 Wh total, drill-style **hot-swap rails** (ThinkPad power-bridge behavior) |
| Runtime | ≥ 30 h target: 20 h terminal/idle + 4 h browsing + 6 h locked (work-day + margin) |
| Cooling | Ducted air path: **left-back intake → 30 mm fan → 5–7 mm heatsink → right-side exhaust** |
| I/O | HDMI, USB 3.x (unshared, ≥1 Gbit/s), USB 2.0 OTG, GbE, full-size SD, 3.5 mm AUX out, USB-C PD 65 W, **magnetic pogo UART** (3.3/5 V, software-controlled, protected) |
| Wireless | On-module Wi-Fi + BT (antenna through RF-transparent lid) · **EU LTE** (tiny LGA radio, nano-SIM) |
| Keyboard | Custom 6-row membrane, US legends + ISO/German Enter, integrated **trackball** below the space bar |
| Chassis | Black matte anodized **aluminum** (CNC), plastic bezel/lid-top for antennas, **ThinkPad-class hinges**, **rainproof with keyboard drainage** (T480-style) |
| Sensors | Lid-open Hall sensor + ambient-light sensor (away from screen) for auto brightness |
| OS | NixOS (plan in progress; mainline kernel/firmware only — no proprietary out-of-tree drivers) |
| Audio | 3.5 mm AUX out + Bluetooth only — **no internal speaker** (by design) |

## Design highlights

- **True hot-swap:** both 4S packs live on a shared rail via ideal-diode OR-ing; slide a pack out, slide another in — power never drops.
- **30 h runtime math is done:** at ~3.7 W average, 144 Wh clears the workload with ~+6% margin at the rail; screen and idle power are the levers to watch.
- **Brightness where it counts:** the 1125-nit panel at ~50 % PWM ≈ 560 nit keeps daylight visibility without eating the runtime budget (~2.2 W).
- **Safety first:** battery/BMS/hot-swap/PD are treated as safety-critical and require professional review before fabrication. The UART port is *signal + ground only* by default; target power is gated, fused and software-controlled (`send_power` / `logic_level` / `power_level`).
- **Authorized use only:** wireless surveying is strictly passive and lawful — no unauthorized access or interference. Legal/regulatory review (RED/CE, privacy, battery transport) is a planned phase.

## Status

Staged engineering workflow (each phase gated by explicit approval):

1. ✅ Discovery & clarification
2. ✅ Feasibility
3. ✅ Architecture & options
4. ✅ Sourced component research (cells, SoM, panel, modules — dated evidence)
5. ✅ BOM & landed-Germany cost (~€1,200 hardware, ≈€1.5k all-in incl. test + contingency)
6. 🚧 Electrical & mechanical plans (drafts: power tree, hot-swap, carrier, UART, thermal, keyboard, envelope — **some in active revision**)
7. ⬜ NixOS & software
8. ⬜ Validation & risk
9. ⬜ Procurement & build roadmap

## Repository layout

```
├── README.md
├── TASK.md              # the original engineering brief / definition of done
├── requirements/        # baseline + compliance matrix (Pass/Partial/Fail/Unknown)
├── architecture/        # system overview, options, interfaces, block diagram
├── hardware/            # compute, display, keyboard, UART, power tree, battery/hot-swap, thermal, PCB
├── mechanical/          # envelope/z-stack + assembly/serviceability
├── parts/               # candidates + alternatives (sourced, dated, priced)
├── vendors/
├── bom/                 # full BOM + landed cost (Germany)
├── electrical/          # block diagram, protection, charging/runtime
├── tests/               # acceptance plan + results (Phase 8)
├── risks/  decisions/  tools/
└── info/                # assumptions, open questions, sources, terminology, session handoff
```

## Governance & sourcing notes

- **Evidence-driven:** every claim carries a source URL, access date, confidence and a stale-price warning where it matters. Unverifiable values are `TBD`, never guessed.
- **Budget:** €1,000 planning baseline; flexibility approved for the aluminum shell. Imports from EU/CN/US are priced into the landed model (VAT/duty/shipping).
- **No purchases by this repo:** procurement is staged, re-quoted at order time, and every price snapshot is dated.
- Battery pack construction, the hot-swap bridge and charging electronics require a qualified electrical/battery safety engineer review before fabrication.

## License / legal

Engineering documentation for personal, authorized use. All wireless activity limited to lawful, passive observation on networks you are authorized to assess. Battery transport (UN38.3/ADR) and radio compliance (RED/CE) rules apply — details tracked in the plan.