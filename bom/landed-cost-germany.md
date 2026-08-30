# Landed Cost — Germany (Phase 5)

Status: **Draft** — 2026-08-30. Every figure is a **snapshot**, re-verify before ordering. This is the delivery-to-Germany total cost model, per TASK.md rules.

## Method & assumptions
- FX: **1 USD = 0.92 EUR** (planning rate; stale-price/caveated).
- **DE VAT 19%** on all purchases for a private individual; AliExpress/Amazon shows VAT (IOSS) at checkout.
- **Import duty**: electronics (HS 8471 computers/parts, 8528 displays) ≈ **0%** EU common external tariff; no quota. Flag: 21700 cells might classify HS 8507 (batteries) — duty & REACH note in §4.
- **Shipping**: EU mostly free/low; KR/US → DE ≈ US$10–15 + DHL handling ~€5–7.
- **Brokerage**: EU channels none; courier customs-handling ~€5–7 on overseas.
- **Chassis CNC labor = €0** (local shop). Chassis cost = raw material + consumables + anodizing + hinges.
- Contingency **15%** (rate & market volatility in 2026 for NAND/LME aluminum).

## Scenario A — "Confirmed build" (user decisions: 8GB, LTE, cells DE, NVMe owned, PCB user-designed)

| # | Item | Qty | Landed EUR |
|---|---|---|---|
| B-001 | Verdin i.MX 8M Plus 8GB (00701100) | 1 | 489 |
| B-002 | NVMe — **user-owned** | 0 | 0 |
| B-004 | Raystar RFU800G-AYH-MNN (1125 nit, DSI) + ship + VAT + handling | 1 | 65 |
| B-005/6 | Carrier PCB (user design, online fab; assembly user/SMT-opt) | 1 | 130 |
| B-007 | Molicel P50B ×8 (akkuteile.de, DE) | 8 | 76 |
| B-008 | 2× BMS/pack PCBs + wiring + contacts | 2 | 70 |
| B-009 | BQ25713RSNR charger | 1 | 4 |
| B-010 | STUSB4500 sink | 1 | 1.2 |
| B-011 | Quectel EC25-EUX LTE | 1 | 37 |
| B-012 | RTL8821CU Wi-Fi module | 1 | 9 |
| B-013/14 | Sunon fan + heatsink | 1+1 | 11 |
| B-015 | Keyboard membrane (custom) | 1 | 45 |
| B-016 | ADNS-9800 trackball sensor | 1 | 3 |
| B-017/18 | Pogo + UART front-end | 1 | 11 |
| B-019 | USB-C conn + power path parts | 1 | 24 |
| B-020/21 | Chassis Al material + anodizing | 1 | 165 |
| B-022 | Hinges | 2 | 9 |
| B-023 | SIM slot + antennas | 1 | 14 |
| B-024 | Audio/ETH/SD/misc | 1 | 42 |
| | **Hardware subtotal** | | **≈ 1,198** |
| B-025 | Test/burn-in (Verdin eval bring-up, bench) | 1 | 120 |
| | **Prototype all-in (hw + test)** | | **≈ 1,318** |
| | **Contingency 15%** | | **≈ 198** |
| | **Total** | | **≈ 1,515** |

Notes: cells line €76 = 8×€8.90 + ~€5 DE ship + rounding. NVMe owned → removed. PCB user-designed keeps fab+optional-SMT line. 8GB + LTE confirmed.

## Scenario B — "Budget-lean" (not selected; kept for reference)
- 4GB Verdin (−€140), 256GB SSD (omitted — user owns NVMe anyway), defer LTE (−€37), no anodize premium.
- Roughly **≈ €1,150–1,250** all-in — but user chose 8GB + LTE, so this path is superseded.

## Cost drivers (for the gate)
| Item | € |
|---|---|
| SoM (8GB) | 489 |
| Chassis (material + anodize) | 165 |
| Carrier + assembly | 130 |
| Cells + BMS/packs | 146 |
| Test/burn-in | 120 |
| Keyboard + trackball | 48 |
| Display | 62 |
| Remaining (LTE/WiFi/fan/charge/UART/etc.) | ~318 |
| **Hardware** | ≈ 1,198 |
| **All-in with 15% contingency** | ≈ **1,515** |

## §4 Import/safety caveats (unchanged — verify in Phase 8/9 unless already done)
1. **21700 cells**: UN38.3 + ADR 3480 air-transport restrictions, REACH/ROHS, and duty code HS 8507 (not 0%). DE-inland cells (akkuteile.de) reduce cross-border concern but transport rules still apply.
2. **Verdin list €410** (8GB, B-001) — batch-negotiate at qty ≥10 (~5–15%).
3. **Anodizing €100–180** — get 2–3 local eloxal quotes.
4. Battery pack build + hot-swap circuit requires professional review before fabrication (REQ-PWR-05).