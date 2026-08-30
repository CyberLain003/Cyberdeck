# Open Questions

Priority: P1 blocker for phase progress, P2 design-shaping, P3 nice-to-have.

| Question ID | Priority | Question | Why it matters | Affected requirement IDs | Options/current interpretation | Needed from | Blocking phase | Status / date |
|---|---|---|---|---|---|---|---|---|
| OQ-001 | P1 | Compute: is Verdin i.MX 8M Plus confirmed given price volatility + no SATA? | cost + storage path | REQ-COMP-03/06, REQ-BUDG-01 | Verdin + PCIe→SATA bridge (DEC-019/027) unless price blows (RISK-017) | Phase 5 quote | Phase 5 | Answer: leading confirmed w/ bridge; price TBD |
| OQ-002 | P1 | Display: confirm Vu8S (5:8, $39, brightness TBD) vs 4:3 upgrade path | runtime + ratio trade | REQ-DISP-01/02, REQ-PWR-02 | **Vu8S leading (cheapest, DEC-035/A-028)**; HE080IA-01E 4:3 as upgrade; HOTHMI 1000-nit when brightness matters over ratio | User + bench | Phase 4 cont | Leading chosen; bench to confirm brightness/power |
| OQ-003 | P1 | Final cell: P45B / 50E / P50B? | runtime margin + price | REQ-PWR-01/02 | P50B (best docs+margin, €6.75) default | Phase 5 | Phase 5 | data ready (E-004/005/006) |
| OQ-004 | P2 | Pack topology: 4S1P (14.4 V) vs 2S2P (7.4 V)? | Currents, rail design, BMS | REQ-PWR-01/03 | 4S preferred unless a rail needs it otherwise | Electrical Phase 6 | Phase 6 | Open |
| OQ-005 | P2 | Keyboard: exact rows (5 vs 5.5 vs 6 incl. func), key pitch, trackball placement (thumb-cluster vs below-trackpad)? | Fit + build method + cost | REQ-KB-01 | 6 rows × ~10 mm pitch; trackball TBD | User | Phase 6 | Open |
| OQ-006 | P2 | Is Ethernet required simultaneously-dedicated, or occasionally usable? | Adds a magnet + PHY on carrier | REQ-IO-04 | Likely occasional; design as bridge chip | User | Phase 6 | Open |
| OQ-007 | P2 | LTE: mandatory in final build or optional add-on (budget/space)? | Budget + antennas + certifications | REQ-RF-02, REQ-BUDG-01 | Optional add-on recommended | User | Phase 4 | Open |
| OQ-008 | P2 | Display interface: native HDMI vs MIPI-DSI direct? | SOM candidate + driver circuitry | REQ-DISP-01 | Native HDMI simpler; DSI lower power | Research Phase 3 | Phase 3 | Open |
| OQ-009 | P3 | External per-pack charging connectors (pack-side USB-C vs proprietary)? | More pack HW + safety | REQ-PWR-04 | Optional; pack-side USB-C if used | User | Phase 6 | Open |
| OQ-010 | P3 | Target runtime validation: full 30 h bench test acceptable, or shortened model? | Test time/cost | REQ-PWR-02 | Model + spot checks, then one 30 h run | User | Phase 8 | Open |
| OQ-011 | P3 | Storage partition/NixOS layout preference (imperative config, flakes, encrypted root?) | NixOS plan shape | REQ-COMP-05 | Open | User | Phase 7 | Open |
| OQ-012 | P3 | Trackball: integrated optical module vs spare-part; any ergonomic priority? | Keyboard design | REQ-KB-01 | ADNS-9800 + small ball (RISK-020 caveat) | Research resolved? | Phase 6 | ADNS under test |
| OQ-013 | P1 | NixOS on Verdin — no published image; how to guarantee? | REQ-COMP-05 hard | REQ-COMP-05 | Custom NixOS config from mainline U-Boot/kernel; gen image via nix generate (Phase 7) | User + research | Phase 7 | Open — plan in Phase 7 |