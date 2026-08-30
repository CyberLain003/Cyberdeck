# Session Handoff

Updated: 2026-08-30. Purpose: allow any future session to resume with full context.

## Current status
- **Phase 2 (Feasibility) complete; Phase 3 (Architecture) complete; Phase 4 (Sourced component research) in progress — first evidence pull done (2026-08-30).**
- Repo `master` — Phase 4 candidates/alternatives/vendors/sources written, not yet committed.
- Next gate: **Phase 5 BOM & cost** after Phase-4 continuation (SSD/bridge, panel+backlight, Verdin quoted price, SSD model).

## Key decisions so far (see decisions/decision-log.md)
1. Envelope 13×17×5 cm (height 4→5 cm, user approved).
2. Clamshell laptop form factor.
3. Battery = 2 packs × 4×21700 (each ≥60 Wh; 50E/P50B-class favored for runtime margin), true hot-swap.
4. In-device 65 W USB-C PD charging; external charging optional.
5. Workload = 20 h terminal + 4 h browse + 6 h locked.
6. Wi-Fi/BT throughput ≥6 MB/s (~48 Mbit/s) w/ PCB/ceramic antenna.
7. RAM 8 GB pref / 4 GB min.
8. All numeric targets are Preferences (user: "everything is malleable") — original text retained in baseline.
9. Custom 4–6L carrier hosting SOM via B2B; user cannot route BGA.

## Constraints & hard gates
- Authorized/legal passive surveying only (REQ-AUTH-01, non-negotiable).
- Safety-critical battery/power design must be professionally reviewed before fabrication (REQ-PWR-05).
- No purchases; procurement recommendations only for approval.
- Strict evidence rule: no "Recommended" without dimensions/interfaces/Linux-NixOS support/price/availability/envelope-effort evidence; TBD otherwise.

## Feasibility summary (info/feasibility.md)
- Concept feasible in principle; binder = runtime (4 W avg class) + sealed-thermal (6–8 W passive) + budget (€700–1,100, contingency 15%).
- Hard blockers for later phases: low-idle ARM SOM; display ≤ ~2.5 W @ working brightness, ≥500 nit 8" 4:3; SATA-capable SOM (or bridge); hot-swap pre-charge + safety review.

## Immediate next actions (Phase 4 continuation → Phase 5)
1. Phase-4 continue: quote JMB582/JMS582 SATA bridge (+M.2 SATA SSD model), panel/backlight decision (RISK-018), Verdin 8GB EU quote (RISK-017), charger IC (BQ25713) part.
2. Then Phase 5: full BOM + landed-Germany cost model.
3. Then gate to Phase 6 (electrical/mechanical).

## Key evidence just gathered (Phase 4, 2026-08-30, all in info/sources.md)
- Cells sourced: P45B/50E/P50B datasheets + EU prices (E-004..008).
- Verdin i.MX 8M Plus confirmed: 0.5 mm SODIMM B2B, idle 1.44–1.7 W, native DSI/HDMI/USB3/OTG/dual-GbE/PCIe-Gen3; **no SATA** (E-009..011).
- Display: HE080IA-01E (4:3 DSI, 160 ppi, no BL), HOTHMI LVDS 1000-nit, Vu8S/Raystar DSI 5:8 (E-013..017).
- LTE: EC25-EUX €31.51 EU bands incl B28 (E-018); Wi-Fi RTL8821CU mainline (E-020/021); fan Sunon/Delta (E-022/023); pogo (E-024); PD sink STUSB4500 (E-025); trackball ADNS-9800 (E-026), small-ball caveat.

## Open questions to resolve early
- OQ-001 SOM family; OQ-008 display interface; OQ-007 LTE mandatory vs optional; OQ-005 keyboard/trackball details; OQ-004 pack topology (4S vs 2S).

## Unresolved / flagged
- 1200-nit original target not met in practice → reclassified Preference with ≥500 nit practical target (user aware + agreed in Q&A).
- 30 h runtime requires display ≤ ~2.5 W and SoC idle ≤ ~1.5–2 W — needs validation with real parts in Phase 4/8.
- 65 W charge-while-use thermal issue — mitigated via vents/charge-taper in Phase 6.