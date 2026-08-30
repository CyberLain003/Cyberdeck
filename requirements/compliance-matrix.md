# Compliance Matrix

Status: **Initial (Phase 2)** — 2026-08-30.
Status states: `Pass`, `Partial`, `Fail`, `Unknown` only. This matrix is updated as evidence arrives (sourcing = Phase 4, measurements = Phase 8). Blank `Evidence/source` for design items is filled in later phases. TBD values are marked `TBD`, never inferred.

| ID | Category | Requirement | Type | Validation method | Status | Evidence/source | Confidence | Impact or gap | Next action |
|---|---|---|---|---|---|---|---|---|---|
| REQ-AUTH-01 | Authorized use | Passive/legal wireless surveying only; no unauthorized access/interference | Hard | Scope review | Pass | User confirmation 2026-08-30; TASK.md §Authorized Use | High | None | Legal/regulatory review Phase 8 |
| REQ-ENC-01 | Envelope | Closed ≤ ~200×140×50 (≤55 w/ 7mm HS); bezels 3/3/3/10 | Hard | CAD/geometry check | Pass | Approved 2026-08-30 (DEC-043): 8" Raystar panel OD 184.93 → needs ~201×138 (calc) ≈ 200×140×50; panel fits | Med | Panel OD to confirm on bench before CNC (TBD) | Mechanical re-derive + bench verify |
| REQ-DISP-01 | Display | 8" MIPI-DSI 800×1280, ≥100 ppi (Raystar, 189 ppi) | Preference | Panel data/measure | Partial | Raystar RFU800G-AYH-MNN sourced (E-014): OD 115.74×184.93×4.75, DSI 4-lane, 1125 nit (DEC-044) | High | OD drives envelope (DEC-043); price quote-only | Bench verify OD + init |
| REQ-DISP-02 | Display | Daylight-visible; 1125-nit panel (exceeds 500–800 target) | Preference | Panel data | Partial | 1125 nit @max; ~2.2 W @ ~50% PWM (≈560 nit) fits ~2.5 W display budget (E-014) | Med | Runtime OK if driven ~50% PWM | Bench measure at working brightness |
| REQ-KB-01 | HMI | ≥5 rows, ≥8 mm keys, US + ISO-DE Enter, trackball below-space; **USB-HID deck MCU (STM32G0)** | Preference | Fit check | Partial | 6-row membrane ~12 mm pitch; deck MCU (DEC-045); trackball small-ball bench-gate (DEC-050) | Med | Trackball geometry test before cutout | Deck firmware Phase 7 |
| REQ-KB-02 | Audio | 3.5 mm AUX out + BT; no internal speaker | Hard | Block diagram | Partial | Policy set | Medium | Codec line-out + BT; no spk amps | Electrical design Phase 6 |
| REQ-IO-01 | I/O | HDMI ≥1080p30 | Preference | Test | Partial | Verdin native HDMI 2.0 TX (E-009) — exceeds 1080p30 | High | Native — no bridge | Cable+bridge test Phase 8 |
| REQ-IO-02 | I/O | USB 3.x ≥1 Gbit/s effective | Preference | Test | Partial | Verdin native USB 3.1 Gen1 host (E-009); keep port unshared (no hub) | High | Do not hub this port | I/O topology Phase 6 |
| REQ-IO-03 | I/O | USB 2.0 OTG | Preference | Test | Partial | Verdin native USB 2.0 OTG (E-009) | High | Dual-role port on custom carrier | I/O topology Phase 6 |
| REQ-IO-04 | I/O | Ethernet ≥100 Mbit | Preference | Test | Partial | Verdin dual GbE (RGMII or on-module PHY, E-009) — exceeds requirement | High | Native GMAC2 via RGMII | I/O topology Phase 6 |
| REQ-IO-05 | I/O | Full-size SD slot | Preference | Test | Unknown | carrier add-on (SDIO) | Med | low risk | I/O topology Phase 6 |
| REQ-ENC-02 | Envelope | Clamshell; **black matte anodized aluminum chassis (metal edges/rims; plastic bezel + plastic lid-top behind screen for antennas); micro fan + 5–7 mm heatsink; front battery insertion; three vertical boards (motherboard/daughterboard/keyboard PCB) w/ SoM on right palm rest; ThinkPad-class hinges; rainproof w/ keyboard drainage** | Hard | Design review | Partial | DEC-016/017/018/052/053/054/055/056/023; CNC local no labor cost | Medium | Mechanical re-derive for vertical stack + palm-rest SoM + plastic lid-top + rainproof | Mechanical re-derive Phase 6 |

## Consistency note

- Statuses here are Phase-2 first-order views. No component is sourced yet; every `Unknown` requires evidence from Phase 4 research before any `Pass`.
- `Partial` items are design-understood but not yet implemented or measured.
- The 30 h runtime (`REQ-PWR-02`) and 1200-nit (`REQ-DISP-02`) statements include explicit assumptions — see `info/feasibility.md`.