# Cyberdeck

A compact, fully functional custom cyberdeck — researched, planned, and documented as an evidence-driven engineering project.

## Project Objective and Authorized Use

Plan a compact, fully functional custom cyberdeck authorized for:

- **Authorized and legal wardriving and wireless surveying only** (passive, lawful surveying and logging on systems/networks where the user has authorization or observation is legally permitted). Excludes unauthorized access, disruption, credential interception, exploitation, concealment, and evasion.
- Terminal work and web browsing.
- Music playback via 3.5 mm AUX or Bluetooth — **no internal speaker**.
- Note-taking and programming.
- Authorized hardware/security work using a UART terminal.

## Requirements Baseline (Summary)

| Area | Requirement |
|---|---|
| Compute | ≥2 cores, ≥1 GHz CPU; 4–8 GB DDR4+ (8 GB mandatory status under clarification) |
| Storage | One M.2 2280 **SATA** SSD (B-key or B+M-key — not NVMe M-key) |
| Display | 8" 4:3, ≥720p @ 30 Hz, ≥1200 nits |
| Keyboard | Integrated trackball; US labels, ISO/German-style Enter key |
| I/O | HDMI ≥1080p@30, USB 3.x (≥1 Gbit/s effective), USB 2.0 OTG, 3.5 mm AUX, USB-C PD (65 W), Ethernet ≥100 Mbit/s, full-size SD slot, magnetic pogo-pin UART (switchable 3.3 V/5 V), internal Wi-Fi+BT, EU LTE/5G modem + nano-SIM |
| Power | ≥120 Wh battery, ≥30 h runtime, swappable/hot-swappable packs |
| Envelope | ≤13 cm × 17 cm × 4 cm (≈884 cm³) |
| Linux/NixOS | Full upstream/`linux-firmware` driver support; NixOS fully supported |

## Budget

- **Starting budget:** EUR 1,000 landed in Germany (total delivered-to-Germany project budget, unless exclusions are clarified).

## Critical Feasibility Flags

- Fitting ≥120 Wh into the **884 cm³** envelope is a major packing, thermal, and safety constraint.
- 30 h at 120 Wh implies an average draw of ≤**4 W before conversion losses** — allowable device load is lower after losses and reserve margins.
- All numerical requirements are validation criteria tracked in a compliance matrix (status: `Pass` / `Partial` / `Fail` / `Unknown`).

## Workflow

Staged, gate-reviewed workflow, each phase requiring approval before advancing:

1. Discovery and clarification
2. Feasibility
3. Architecture and options
4. Sourced component research
5. BOM and cost
6. Electrical and mechanical plans
7. NixOS and software
8. Validation and risk
9. Procurement and build roadmap

Comprehensive project detail, full requirements baseline, evidence sources, compliance tracking, and planning assets are maintained in `TASK.md` and the project documentation tree.

## Repository Layout

```
cyberdeck-project/
├── README.md
├── requirements/   (baseline, compliance matrix)
├── architecture/   (system, options, interfaces)
├── hardware/       (compute, display, storage, networking, cellular, keyboard, UART, power, battery, thermal, PCB)
├── software/       (NixOS, firmware/drivers, applications)
├── parts/          (candidates, alternatives)
├── vendors/
├── bom/            (BOM, landed cost Germany)
├── mechanical/     (envelope, assembly)
├── electrical/     (block diagram, protection, charging/runtime)
├── tests/          (acceptance plan, results)
├── risks/
├── decisions/
├── tools/
└── info/           (open questions, assumptions, sources, terminology, session handoff)
```

## Scope and Governance

- **No purchases** are made by this project; procurement is recommended for approval and staged/gated.
- Battery, BMS, hot-swap, charging, and USB-C PD subsystems are treated as **safety-critical**; detailed battery construction must be reviewed by a qualified electrical/battery safety engineer before fabrication.
- Wireless activity is limited to authorized and legal passive surveying; no unauthorized access or interference.
- All claims are evidence- and source-backed (URL + access date + confidence + stale-price warning where applicable). Unverifiable values are marked `TBD`.