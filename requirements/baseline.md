# Requirements Baseline

Status: **Approved for Phase 2 (Feasibility)** — 2026-08-30.
Change process: this is the living baseline. Requirements are only changed with explicit user approval and a decision-log entry (see `decisions/decision-log.md`). Original TASK.md targets are preserved; user clarifications and approved changes are marked.

## Authorized Use and Scope (locked)

The cyberdeck is authorized for **passive, lawful** wireless surveying and wardriving on systems/networks the user is authorized to assess; terminal work, web browsing, note-taking and programming; music via 3.5 mm AUX or Bluetooth (no internal speaker); and authorized hardware/security work via a UART terminal. It excludes unauthorized access, disruption, credential interception, exploitation, concealment, or evasion.

## Approved Envelope and Form Factor (user-approved changes)

| Item | Original target (TASK.md) | Approved baseline | Status |
|---|---|---|---|
| Closed external dimensions | 13 × 17 × 4 cm (884 cm³) | **~20 × 14 × 5–5.5 cm (~1400–1540 cm³)** — approved: 8" panel + 3/3/3/10 mm bezels (2026-08-30) | Changed, approved |
| Form factor | (not specified) | **Clamshell, normal laptop style** | Defined by user |
| Internal Wi-Fi "6MB" | ambiguous | **≥6 MB/s (~48 Mbit/s) throughput** with PCB/ceramic antenna | Clarified |

## Requirements List

IDs are stable and used across all project assets. `Type` follows TASK.md discipline (Hard / Preference / Example / Unclear). Per user statement "everything is malleable", numeric targets below original spec are graded **Preference** unless the user explicitly made them hard, but the original target text is preserved verbatim in the requirement.

### Authorized Use & Envelope
| ID | Requirement | Type |
|---|---|---|
| REQ-AUTH-01 | Passive/legal wireless surveying only; no unauthorized access, disruption, interception, exploitation, concealment, evasion. | Hard (safety/legal, non-negotiable) |
| REQ-ENC-01 | Closed envelope ≤ ~200 × 140 × 50 mm (≤55 with 7 mm heatsink); 3 mm top/left/right bezel + 10 mm bottom on the lid for the 8" panel. | Hard (approved change from 130×170×50) |
| REQ-ENC-02 | Clamshell (landscape palmtop/laptop, 200 mm width); **three horizontal flat boards** (keyboard PCB top, carrier motherboard, rear daughterboard) via FPC/J-connectors; **keyboard at regular-laptop position (10% from top, ends 40% from bottom)**, soM + thermal tower on right palm-rest; premium ThinkPad-class hinges (one-finger open, carry by screen); **rainproof for hours with keyboard drainage**; plastic bezel + plastic lid-top behind screen for antennas, metal edges/rims; **cooling air: left-back intake channels → low-pressure draw → 30 mm fan → 5–7 mm heatsink → right exhaust**; drill-rail hot-swap batteries; **minimal mechanical changes permitted for fit** | Hard |
| REQ-ENC-03 | No internal speaker, microphone, or camera. | Hard |

### Compute, Memory, Storage
| ID | Requirement | Type |
|---|---|---|
| REQ-COMP-01 | CPU ≥ 2 cores, ≥ 1 GHz. | Preference |
| REQ-COMP-02 | Memory 4–8 GB DDR4 or better; 8 GB preferred, 4 GB acceptable. | Preference |
| REQ-COMP-03 | One M.2 2280 **NVMe** SSD (M-key; **user-approved change** from SATA). Socket keyed **M-only** + silkscreen so only the matching NVMe 2280 physically fits ("hardware blocking", user 2026-08-30). | Preference (changed) |
| REQ-COMP-04 | Linux compatibility: all firmware/drivers upstream or in `linux-firmware`; no proprietary out-of-tree drivers. | Hard |
| REQ-COMP-05 | NixOS fully supported (defined = OS runs reproducibly; kernel, firmware, Wi-Fi/BT, modem, audio, suspend/resume work with a test strategy). | Hard |
| REQ-COMP-06 | Compute = custom carrier PCB (4–6 layer, PCBWay-practical) hosting an off-the-shelf SOM/computer-on-module via B2B connector. | Hard (user architectural decision) |

### Display & Human Interface
| ID | Requirement | Type |
|---|---|---|
| REQ-DISP-01 | Display ~8", MIPI-DSI 800×1280 (5:8), ≥ 100 ppi (**Raystar RFU800G-AYH-MNN, 189 ppi**); 4:3 original target preserved as preference. | Preference (panel chosen) |
| REQ-DISP-02 | Brightness ~1200 nits original target; **chosen panel = Raystar RFU800G-AYH-MNN @ 1125 nits** (brightest in list, ~daylight-visible at working PWM). | Preference (exceeds 500–800 practical) |
| REQ-DISP-03 | **Lid-open sensor (Hall) + ambient-light sensor pointing away from the screen for automatic brightness.** | Preference |
| REQ-KB-01 | Keyboard ≥ 5 rows (+ up to ½ row of function keys), min ~8 mm per key, US legends with ISO/German-style Enter, integrated trackball **placed between G-H-B (below space bar)**; scanned by a **low-power USB-HID deck MCU** (STM32G0-class). | Preference |
| REQ-KB-02 | Audio out only: 3.5 mm AUX + Bluetooth (no internal speaker). | Hard (paired with REQ-ENC-03) |

### I/O
| ID | Requirement | Type |
|---|---|---|
| REQ-IO-01 | ≥ 1 HDMI output ≥ 1080p30. | Preference |
| REQ-IO-02 | ≥ 1 USB 3.x port, ≥ 1 Gbit/s **effective** throughput. | Preference |
| REQ-IO-03 | ≥ 1 USB 2.0 OTG port. | Preference |
| REQ-IO-04 | Ethernet ≥ 100 Mbit/s (1 Gbit preferred). | Preference |
| REQ-IO-05 | One full-size SD card slot. | Preference |
| REQ-IO-06 | Dedicated USB-C PD power input, 65 W capability (power only). | Preference |
| REQ-UART-01 | Magnetic pogo-pin UART (6-pin) with switchable 3.3 V / 5 V logic levels, with electrical protection and level shifting; **software-controlled bits (files/symlinks): `send_power` (0=off), `logic_level` (0=3.3 V), `power_level` = VCCO select (0=3.3 V / 1=5 V); gated +VCCO up to 5 W @ 5 V**; safe default = signal + ground only until power behavior is designed and reviewed. | Hard |
| REQ-UART-02 | UART serves varied targets: server terminal COM ports, microcontrollers, serial consoles (user's own cables). Pogo for ease of use/weather resistance. | Hard (usage) |

### Wireless
| ID | Requirement | Type |
|---|---|---|
| REQ-RF-01 | Internal Wi-Fi + Bluetooth from the **SoM (Verdin "WB" variant, on-module Wi-Fi/BT)**, throughput ≥ 6 MB/s (~48 Mbit/s); antenna coax to an RF window in the aluminum deck. | Preference |
| REQ-RF-02 | European LTE/5G modem, nano-SIM, **tiny LGA radio module (~ESP32-WROOM footprint, ≤~30 mm)**; Telekom-class bands; cheapest option, private use only. | Preference |

### Power, Battery, Runtime
| ID | Requirement | Type |
|---|---|---|
| REQ-PWR-01 | Total battery ≥ 120 Wh; implemented as 2 packs, each ≥ 60 Wh guaranteed (up to ~72 Wh/pack with 5000 mAh cells). | Hard |
| REQ-PWR-02 | Runtime ≥ 30 h for defined workload: 20 h terminal/idle + 4 h browsing + 6 h locked-idle. | Preference (malleable) |
| REQ-PWR-03 | True hot-swap between packs (T480-style power bridge); both packs removable while powered. | Hard |
| REQ-PWR-04 | Charging in-device via the single 65 W USB-C PD port; external per-pack charging optional (slower). | Preference |
| REQ-PWR-05 | Battery, BMS, hot-swap, charging, and USB-C PD treated as safety-critical; design must be reviewed by a qualified electrical/battery safety engineer before fabrication. | Hard (process) |

### Budget
| ID | Requirement | Type |
|---|---|---|
| REQ-BUDG-01 | Total landed budget ≤ EUR 1,000 delivered-to-Germany (VAT, shipping, duty, fees, contingency included) as **planning baseline; may increase to facilitate the aluminum chassis** (approved). **Imports from CN/US allowed; shipping/VAT/duty must be priced in.** Chassis CNC local at no labor cost. | Preference (strong target, flex approved) |

## Workload Definition (REQ-PWR-02 detail)

30 h minimum = **20 h terminal/idle + 4 h web browsing + 6 h locked/suspended**, representing an average work day plus margin. Display at a "typical working" brightness (not full 1200-nits peak) for the active portions. This is the validation workload for runtime.

## Derived Constraints (from user-approved architecture)

- Battery geometry: 2 packs, each 4× 21700 inline → pack slab ≈ 105 × 70 × 24 mm, two side-by-side across the 17 cm width (≈16 cm incl. internals and walls).
- Batteries hot-swappable with the system powered (power-bridge architecture).
- Carrier PCB: 4–6 layers, no ultra-fine pitch (user CANNOT route sub-0.4 mm / BGA; SOM must expose its carrier via a B2B connector with routable pitch).