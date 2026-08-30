# Session Handoff

Updated: 2026-08-30. Purpose: allow any future session to resume with full context.

## Current status
- **Phase 6 (Electrical + Mechanical) complete** — electrical (block-diagram, pcb, power-tree, battery hot-swap, thermal, uart) **and** mechanical **flat-board rework done**.
- Envelope **200 × 140 × 50 mm** (≤55 w/ 7 mm heatsink); bezels 3/3/3/10 (T/L/R/B); lid interior 194 × 134.
- **Display = Raystar RFU800G-AYH-MNN** (8", 800×1280, DSI 4-lane, 1125 nit, OD 115.74×184.93×4.75 — fits lid window ≈ 188×121).
- **On-module Wi-Fi/BT** (Verdin "WB", antenna coax → RF-transparent plastic lid-top).
- **Tiny-LGA LTE (EC200U/BG95/SIM7000-class) pending re-research + BOM** (DEC-058).
- **Air path: Delta BFB0305HA-C blower** primary — left-back intake → heatsink fins L→R → right-side exhaust (DEC-062).
- Repo `master`.

| Area | State |
|---|---|
| Electrical (Phase 6) | **complete** (block-diagram, pcb, power-tree, battery hot-swap, thermal, uart) |
| Mechanical (Phase 6) | **complete** — flat-board rework; envelope 200×140×50 |
| Display | Raystar RFU800G (DEC-044) — OD/power bench pending |
| Wi-Fi/BT | SoM on-module (WB, DEC-046) — antenna feed to plastic lid-top |
| LTE | tiny-LGA Cat-1 (EC200U-EU/BG95/SIM7000) — **re-research pending (DEC-058)** |
| Thermal | **Delta blower primary**; Sunon axial fallback (DEC-062) |
| Storage | M.2 2280 **NVMe** (M-key, silkscreen-keyed) — DEC-033/048 |
| Next | **Phase 7 — NixOS (planned for tomorrow)** |

## Key decisions since last handoff (DEC-052..063, full log in decisions/decision-log.md)

| ID | Decision |
|---|---|
| DEC-052 | Three horizontal flat boards: keyboard PCB (top) / carrier motherboard / rear daughterboard; FPC interconnect |
| DEC-053 | SoM + thermal tower (5–7 mm HS + 30 mm fan) in the right palm-rest |
| DEC-054 | ThinkPad-class friction hinges (~0.6–1.0 N·m/side) |
| DEC-055 | Plastic bezel + RF-transparent plastic lid-top behind screen; metal edges/rims |
| DEC-056 | Rainproof for hours + keyboard drainage |
| DEC-057 | Hall lid sensor + ALS pointing away from screen |
| DEC-058 | LTE = tiny LGA module (~ESP32-WROOM footprint, ≤~30 mm), EC200U-EU/BG95/SIM7000-class — Cat-1; mPCIe EC25 superseded as leading |
| DEC-059 | UART VCCO 3.3/5 V, ≤5 W@5 V, software bit-files |
| DEC-060 | Drill-battery rails in mid chassis spine; front-insert; hot-swap (OR-ing) |
| DEC-061 | Keyboard 10% from top → ends 40% from bottom; palm rest = front 40% |
| DEC-062 | Air path: left-back intake → 30 mm fan → heatsink fins L→R → right-side exhaust |
| DEC-063 | Landscape palmtop/laptop; 200 mm = width; minimal mechanical changes permitted for fit |

## Open items (from current-state docs)
- Tiny-LGA LTE **final part** (bands B1/3/7/8/20, power, landed price) → BOM (DEC-058).
- B2B socket stack height final (TE 2309409-2 5.2 mm recommended vs Amphenol alt).
- Heatsink **5 vs 7 mm** (thermal margin).
- Hinge vendor + torque spec.
- Fan **PWM/tach** wiring — Delta -C is 2-wire, low-side PWM TBD (RISK-027).
- Panel **OD/power bench** (Raystar; DEC-044) + re-quote price.
- Small-ball trackball bench gate (RISK-020); function-row decision (OQ-005).

## Constraints & hard gates
- Authorized/legal passive surveying only (REQ-AUTH-01, non-negotiable).
- Safety-critical battery/power design must be professionally reviewed before fabrication (REQ-PWR-05).
- No purchases; procurement recommendations only for approval.
- Strict evidence rule: no "Recommended" without dimensions/interfaces/Linux-NixOS support/price/availability/envelope-effort evidence; TBD otherwise.

## Feasibility summary (info/feasibility.md)
- Concept feasible; binder = runtime (4 W avg class) + budget (€700–1,100, contingency 15%) + sealed-thermal (addressed by the ducted blower path, DEC-062).
- Hard blockers for later phases: low-idle ARM SOM (solved: Verdin idle 1.44–1.7 W); display ≤ ~2.5 W @ working brightness (solved: Raystar ≈2.2 W @ ~560 nit); hot-swap pre-charge + safety review (designed, review gate stands).

## Immediate next actions (Phase 7 — NixOS, planned tomorrow)
1. **Phase 7 NixOS:** image/build plan for Verdin i.MX 8M Plus — mainline U-Boot + extlinux/ext4 (no config.txt, DEC-034); key: kernel, firmware, Wi-Fi/BT, modem (ModemManager), audio, suspend/resume + test strategy (REQ-COMP-05).
2. Parallel research: tiny-LGA LTE **final part** (Cat-1 bands/power/price, DEC-058) → update BOM.
3. Bench gates to schedule: Raystar OD/power; fan PWM (RISK-027); small-ball trackball (RISK-020).

## Key evidence (in info/sources.md; access 2026-08-30)
- **Compute:** Verdin i.MX 8M Plus 8 GB — idle 1.44–1.7 W, DSI/HDMI/USB3/OTG/dual-GbE/PCIe-Gen3×1, **no SATA → NVMe direct (DEC-033)**, mainline U-Boot/extlinux (E-009..011, DEC-034).
- **Display:** Raystar RFU800G 1125-nit DSI (E-014, DEC-044); Vu8S ~300–400-nit (historical, E-016/030); HE080IA 4:3 DSI (E-013); HOTHMI LVDS 1000-nit (E-015).
- **Thermal:** Delta BFB0305HA-C blower 1.45 CFM/0.285 inH₂O vs Sunon HA30101V4 3.5 CFM axial (E-022/023/033/034).
- **LTE:** EC25-EUX mPCIe €31.51 (E-018 — **historical alt** after DEC-058); tiny-LGA Cat-1 re-research pending.
- Unchanged: pogo (E-024), PD STUSB4500 (E-025), trackball ADNS-9800 + small-ball caveat (E-026).

## Open questions to resolve early
- OQ-001/004/008: **resolved** — Verdin; 4S1P packs; Raystar DSI.
- OQ-005: keyboard function-row + trackball small-ball finalization.
- OQ-007: LTE — now tiny-LGA Cat-1 re-research (DEC-058) → BOM.

## Unresolved / flagged
- Display + SoM power validated on real parts in Phase 8 (RISK-001/018).
- 65 W charge-while-use thermal — handled via fan-max + charge-taper (RISK-016), validated on bench.
- Fan control (Delta 2-wire PWM/Tach) needs a confirmed driver scheme before layout (RISK-027).