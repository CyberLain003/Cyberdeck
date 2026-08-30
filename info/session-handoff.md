# Session Handoff

Updated: 2026-08-30. Purpose: allow any future session to resume with full context.

## Current status
- **Phase 2 (Feasibility) approved and completed.** Repo (branch `master`) at hash with baseline, compliance matrix, feasibility, decisions, risks, info assets.
- **Go-ahead received** from user for Phase 2 after Phase-1 Q&A (12 questions answered).
- Next gate: **Phase 3 — Architecture and options** (SOM family, display path, power/hot-swap topology, keyboard, chassis). Await user approval to begin Phase 3.

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

## Immediate next actions (Phase 3)
1. Compare SOM families (i.MX8MP-class vs PCIe-based) on: idle power, native SATA/HDMI/USB3/OTG, B2B pitch routable at 6L, mainline/NixOS support.
2. Confirm display interface (native HDMI vs DSI) to narrow panels.
3. Sketch power topology (4S power-bridge, charge path) and hot-swap inrush plan (high level).
4. Present architecture matrix for approval.

## Open questions to resolve early
- OQ-001 SOM family; OQ-008 display interface; OQ-007 LTE mandatory vs optional; OQ-005 keyboard/trackball details; OQ-004 pack topology (4S vs 2S).

## Unresolved / flagged
- 1200-nit original target not met in practice → reclassified Preference with ≥500 nit practical target (user aware + agreed in Q&A).
- 30 h runtime requires display ≤ ~2.5 W and SoC idle ≤ ~1.5–2 W — needs validation with real parts in Phase 4/8.
- 65 W charge-while-use thermal issue — mitigated via vents/charge-taper in Phase 6.