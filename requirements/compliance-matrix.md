# Compliance Matrix

Status: **Initial (Phase 2)** — 2026-08-30.
Status states: `Pass`, `Partial`, `Fail`, `Unknown` only. This matrix is updated as evidence arrives (sourcing = Phase 4, measurements = Phase 8). Blank `Evidence/source` for design items is filled in later phases. TBD values are marked `TBD`, never inferred.

| ID | Category | Requirement | Type | Validation method | Status | Evidence/source | Confidence | Impact or gap | Next action |
|---|---|---|---|---|---|---|---|---|---|
| REQ-AUTH-01 | Authorized use | Passive/legal wireless surveying only; no unauthorized access/interference | Hard | Scope review | Pass | User confirmation 2026-08-30; TASK.md §Authorized Use | High | None | Legal/regulatory review Phase 8 |
| REQ-ENC-01 | Envelope | Closed ≤ 130×170×50 mm | Hard | CAD/geometry check | Pass | Approved change (40→50 mm) 2026-08-30; Phase 2 geometry analysis | Medium | Height relaxed by user; internal stack TBD | Mechanical stack Phase 6 |
| REQ-ENC-02 | Envelope | Clamshell form factor | Hard | Design review | Pass | User decision 2026-08-30 | Medium | Drives hinge/base layout | Mechanical design Phase 6 |
| REQ-ENC-03 | Envelope | No internal speaker/mic/camera | Hard | Design constraint | Partial | Policy set | Medium | Carrier audio = line out only; must exclude mic/cam | Electrical block diagram Phase 6 |
| REQ-COMP-01 | Compute | CPU ≥2 cores ≥1 GHz | Preference | Bench test | Unknown | — | High | Every SOM candidate ≥2 cores @ >1 GHz; mainstream | Select SOM Phase 3/4 |
| REQ-COMP-02 | Compute | RAM 8 GB pref / 4 GB min | Preference | Bench test | Unknown | — | Medium | Most SOMs ≥4 GB; 8 GB limits candidate set slightly | Select SOM Phase 3/4 |
| REQ-COMP-03 | Storage | M.2 2280 SATA SSD | Preference | Interface test | Unknown | — | Medium | Requires SOM native SATA or PCIe→SATA bridge (i.MX8MP native SATA is a candidate) | I/O topology Phase 6; candidate filter Phase 4 |
| REQ-COMP-04 | Software | Upstream/linux-firmware drivers only | Hard | Kernel check | Partial | Policy set | High | Restricts SOM choice to mainline-supported silicon | Per-SOM evidence Phase 4/7 |
| REQ-COMP-05 | Software | NixOS fully supported | Hard | NixOS install test | Partial | Test strategy TBD | Medium | Needs definition + evidence per SOM | NixOS plan Phase 7 |
| REQ-COMP-06 | Compute | Custom 4–6L carrier + SOM via B2B | Hard | Design review | Pass | User architectural decision 2026-08-30 | Medium | PCBWay 4–6L constraints drive placement/routing | Carrier design Phase 6 |
| REQ-DISP-01 | Display | 8" 4:3 ≥720p@30 | Preference | Panel data/measure | Unknown | — | Medium | Need sourced 4:3 8" panel ≤ ~2 W @ working brightness | Panel sourcing Phase 4 |
| REQ-DISP-02 | Display | Daylight-visible (~1200 nits target; ≥500 nits practical) | Preference | Panel data | Partial | 1200-nit 4:3 8" unobtainable as standard part → relaxed to preference | Medium | 1200 nits essentially custom; 500–800 nits realistic | Confirm panel choice Phase 4 |
| REQ-KB-01 | HMI | ≥5 rows, min 8 mm keys, US legends + ISO-DE Enter, trackball | Preference | Fit check | Partial | Geometry calc: 6 rows × ~10 mm pitch fits 130 mm; ~12 mm key width → netbook-class | Medium | Keys small but typable; trackbar strip placement TBD | Keyboard design Phase 6 |
| REQ-KB-02 | Audio | 3.5 mm AUX out + BT; no internal speaker | Hard | Block diagram | Partial | Policy set | Medium | Codec line-out + BT; no spk amps | Electrical design Phase 6 |
| REQ-IO-01 | I/O | HDMI ≥1080p30 | Preference | Test | Unknown | — | Medium | SOM native HDMI (i.MX8MP) or MIPI→HDMI bridge (extra cost) | Candidate filter Phase 4/6 |
| REQ-IO-02 | I/O | USB 3.x ≥1 Gbit/s effective | Preference | Test | Partial | USB 3 host dedicated to a port; effective ≥1 Gbit needs unshared 5 Gbit link | Medium | Hub/shared-lane must be avoided on this port | I/O topology Phase 6 |
| REQ-IO-03 | I/O | USB 2.0 OTG | Preference | Test | Unknown | — | Medium | Requires OTG-capable host port on SOM | Candidate filter Phase 4 |
| REQ-IO-04 | I/O | Ethernet ≥100 Mbit | Preference | Test | Unknown | — | High | Trivial via USB3 GbE (RTL8153-class) or native RGMII | I/O topology Phase 6 |
| REQ-IO-05 | I/O | Full-size SD slot | Preference | Test | Unknown | — | Medium | Carrier add-on, low risk | I/O topology Phase 6 |
| REQ-IO-06 | I/O | USB-C PD 65 W input (power only) | Preference | PD test | Partial | PD3.0 65 W chargers common; carrier must implement PD sink + ≥65 W current path | Medium | Sink controller + on-board charging architecture needed | Carrier power design Phase 6 |
| REQ-UART-01 | UART | Magnetic pogo UART, 3.3/5 V switchable, protected, signal+ground default | Hard | Review/test | Partial | Approved approach; front-end design pending | Medium | Level-shift + ESD + polyfuse + isolation required; not powering targets | UART design Phase 6 |
| REQ-UART-02 | UART | Connect varied targets via user's own cables | Hard | Test | Partial | Pogo chosen for ease; user fabricates cables | Medium | Pinout/footprint standardization needed | Pinout Phase 6 |
| REQ-RF-01 | Wireless | Wi-Fi+BT, ≥6 MB/s (~48 Mbit/s), PCB/ceramic antenna | Preference | Test | Unknown | — | High | Any 11n/ac module exceeds 6 MB/s; antenna + module integration to source | Module sourcing Phase 4 |
| REQ-RF-02 | Cellular | EU LTE/5G nano-SIM (Telekom bands) | Preference | Carrier test | Unknown | — | Medium | Quectel-class USB/mPCIe; bands 1/3/7/20/28; regulatory review needed | Modem sourcing Phase 4 |
| REQ-PWR-01 | Power | ≥120 Wh total, ≥60 Wh/pack | Hard | Calc | Pass | 2×4×21700: P45B 129.6 Wh, 50E/P50B ~141–144 Wh — all ≥120 Wh, per-pack ≥60 Wh | Medium | Capacity geometry fits envelope; cell sourcing + price TBD | Cell sourcing Phase 4 |
| REQ-PWR-02 | Power | ≥30 h runtime (20 term + 4 browse + 6 locked) | Preference | Workload test | Partial | Avg battery draw ≈ 3.7–3.9 Wh → ~3.2–3.4 W at rail (see feasibility); display power dominant | Medium | Pass only if display ≤ ~2 W @ working brightness and SoC idle ~1.5–2 W | Power budget verify Phase 6; workload test Phase 8 |
| REQ-PWR-03 | Power | True hot-swap (T480-style power bridge) | Hard | Review | Partial | Two parallel packs + OR-ing + pre-charge inrush mitigation; feasible topologically | Medium | Inrush/current-sharing/voltage-balance are the risks; circuit + safety review Phase 6 | Power-bridge design Phase 6 |
| REQ-PWR-04 | Power | Charge in-device via 65 W USB-C PD; external optional | Preference | Test | Pass | 65 W in-device charging feasible per charge-time analysis; external charging = optional per-pack ports | Medium | Thermal during charge limits (see feasibility) | Charge design + thermal Phase 6 |
| REQ-PWR-05 | Power | Safety-critical review gate | Hard | Process | Pass | Policy set; qualified review required before fabrication | High | Professional battery/electrical review is a gate, not a suggestion | Review booking Phase 8/9 |
| REQ-BUDG-01 | Budget | ≤ EUR 1,000 landed DE **baseline; may increase to facilitate the aluminum shell** | Preference | Cost model | Partial | First-order estimate €700–1,100 (incl. 15% contingency); chassis CNC labor now FREE (material + coating only, DEC-023); chassis no longer a top cost driver; risk shifts to SOM/LTE/keyboard | Medium | €1,000 target achievable; headroom for chassis material + anodizing | BOM + landed cost Phase 5 |
| REQ-ENC-02 | Envelope | Clamshell; **black matte anodized aluminum chassis; micro fan + 5–7 mm heatsink; front battery insertion; rear I/O/power daughterboard** | Hard | Design review | Partial | Architectural decisions 2026-08-30 (DEC-016/017/018); RF-opaque ⇒ antenna apertures; **CNC local at no labor cost (DEC-023)** | Medium | RF windows, material+coating cost, two-board bussing | Mechanical stack Phase 6 |

## Consistency note

- Statuses here are Phase-2 first-order views. No component is sourced yet; every `Unknown` requires evidence from Phase 4 research before any `Pass`.
- `Partial` items are design-understood but not yet implemented or measured.
- The 30 h runtime (`REQ-PWR-02`) and 1200-nit (`REQ-DISP-02`) statements include explicit assumptions — see `info/feasibility.md`.