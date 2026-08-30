# Assumptions Log

| Assumption ID | Assumption | Basis/source | Confidence | Affected requirement IDs | Impact if wrong | Validation action | Owner | Status/date |
|---|---|---|---|---|---|---|---|---|
| A-001 | Envelope height relaxed 40→50 mm; closed 130×170×50 mm | User approval 2026-08-30 | High | REQ-ENC-01 | More room for battery/stack | Geometry check Phase 6 | Lain | Accepted 2026-08-30 |
| A-002 | Clamshell form factor | User decision 2026-08-30 | High | REQ-ENC-02 | Mechanical layout | Design Phase 6 | Lain | Accepted |
| A-003 | Packs = 4× 21700 in series per pack (4S1P, 14.4 V); 2 packs | User geometry + feasibility §2 | Medium | REQ-PWR-01/03 | Voltage/current topology; could be 2S2P | Electrical Phase 6 | Lain | Provisional |
| A-004 | Pack slab ≈ 105×70×24 mm; two side-by-side ≈ 16 cm < 17 cm width | User estimate + calculation | Medium | REQ-ENC-01/REQ-PWR-01 | Interior layout | Mechanical Phase 6 | Lain | Provisional |
| A-005 | Cell classes: P45B 16.2 Wh, 50E 17.7 Wh, P50B-class ~18 Wh @3.6 V | Datasheet-typical figures, to be sourced in Phase 4 | Medium | REQ-PWR-01/02 | Runtime/capacity numbers move ±5 % | Cell sourcing Phase 4 | Lain | Provisional |
| A-006 | Battery→rail conversion efficiency 90%; aging+reserve 10% | Common practice for 4S BMS + multi-rail DC-DC | Medium | REQ-PWR-02 | Runtime margin shifts | Measured Phase 8 | Lain | Provisional |
| A-007 | Workload power: suspend 1.2 W, terminal 3.8 W, browse 6.6 W (battery side) | First-order planning targets | Low | REQ-PWR-02 | 30 h is the tightest requirement | Power test Phase 8 | Lain | Provisional |
| A-008 | Display ≤ 2.5 W at working brightness, idle-capable SoM ~1.5–2 W | Needed to close 30 h maths | Low | REQ-PWR-02 | Runtime target fails; trade space invoked | Sourcing Phase 4 | Lain | Provisional |
| A-009 | Interior walls ≈ 3 mm | Design assumption | Medium | REQ-ENC-01 | Volume budget shifts | Mechanical Phase 6 | Lain | Provisional |
| A-010 | Carrier PCB 4–6 layers on PCBWay; SOM via B2B only (no fine BGA by user) | User constraint REQ-COMP-06 | High | REQ-COMP-06 | Narrower SOM set | Placement Phase 6 | Lain | Accepted |
| A-011 | FX 1 EUR ≈ 1 USD; German VAT 19% | Planning basis | Medium | REQ-BUDG-01 | Budget shifts | Phase 5 quotes | Lain | Provisional |
| A-012 | Duty + brokerage small (~0–5%) on IT components into DE | Typical; to be confirmed | Medium | REQ-BUDG-01 | Budget shifts | Phase 5 | Lain | Provisional |
| A-013 | 21700 cell OD ≈ 21.5 mm with wrap/label; length ≈ 70 mm | Standard packaging allowance | Medium | REQ-PWR-01 | Pack slab size ± a few mm | Cell sourcing Phase 4 | Lain | Provisional |
| A-014 | Wi-Fi/BT throughput target ≥6 MB/s ≈ 48 Mbit/s (user clarification) | User clarification 2026-08-30 | High | REQ-RF-01 | Interpretation locked | Test Phase 8 | Lain | Accepted |
| A-015 | No internal speaker/mic/camera (REQ-ENC-03/REQ-KB-02) | User requirement | High | REQ-ENC-03, REQ-KB-02 | Audio = line out + BT only | Block diagram Phase 6 | Lain | Accepted |