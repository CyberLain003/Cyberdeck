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

## Scenario A — "Recommended build" (current leading choices)
| # | Item | Qty | Landed EUR |
|---|---|---|---|
| B-001 | Verdin i.MX 8M Plus Quad 8GB (00701100) | 1 | 489 |
| B-002 | Kingston NV3 500GB NVMe | 1 | 107 |
| B-004 | Hardkernel Vu8S + ship + VAT + handling | 1 | 62 |
| B-005/6 | Carrier PCB + assembly (prototype qty, conventional parts) | 1 | 130 |
| B-007 | Molicel P50B ×8 + ship | 8 | 61 |
| B-008 | 2× BMS/pack PCBs + wiring + contacts | 2 | 70 |
| B-009 | BQ25713RSNR charger | 1 | 4 |
| B-010 | STUSB4500 sink | 1 | 1.2 |
| B-011 | LC25-EUX LTE | 1 | 37 |
| B-012 | RTL8821CU Wi-Fi module | 1 | 9 |
| B-013 | Sunon fan + B-014 heatsink | 1+1 | 11 |
| B-015 | Keyboard membrane (custom) | 1 | 45 |
| B-016 | ADNS-9800 trackball sensor | 1 | 3 |
| B-017/18 | Pogo + UART front-end | 1 | 11 |
| B-019 | USB-C conn + power path parts | 1 | 24 |
| B-020/21 | Chassis Al material + anodizing | 1 | 165 |
| B-022 | Hinges | 2 | 9 |
| B-023 | SIM slot + antennas | 1 | 14 |
| B-024 | Audio/ETH/SD/misc | 1 | 42 |
| | **Subtotal (hardware)** | | **≈ 1,294** |
| B-025 | Test/burn-in (eval kit, panel) | 1 | 160 |
| | **Prototype all-in (hw + test)** | | **≈ 1,454** |
| | **Contingency 15%** | | **215** |
| | **Total (recommended, incl. test + contingency)** | | **≈ 1,650–1,700** |

> Note: B-025 (eval kit + test bench) is a real cost to validate before buying volume; it can be reduced by using the Verdin's own carrier-less bring-up (soldering the B2B onto a dev carrier) — trim to ~€100 if budget-bound.

## Scenario B — Budget-lean (drop extras)
- Swap B-001 → **Verdin 4GB (00631102)**: saves ~€140.
- B-002 → KingSpec 256GB: saves ~€59.
- Keep LTE + Wi-Fi + fan + aluminium. (No other meaningful cuts without breaking REQ.)
| | | ≈ 1,240 all-in incl. contingency |
|---|---|---|

## Scenario A vs B figures — for the gate
| Item | A (recommended) | B (lean) |
|---|---|---|
| Verdin 8GB/4GB | €489 | €350 |
| NV3/KingSpec | €107 | €47 |
| Display | €62 | €62 |
| Carrier+asm | €130 | €130 |
| Cells+BMS/packs | €131 | €131 |
| Charge/PD | €5 | €5 |
| LTE+WiFi+fan+heat | €57 | €57 |
| Keyboard+trackball | €48 | €48 |
| Pogo+UART | €11 | €11 |
| USB-C/power | €24 | €24 |
| Chassis (mat+anodize) | €165 | €165 |
| Hinges | €9 | €9 |
| Antenna/SIM | €14 | €14 |
| Audio/ETH/misc | €42 | €42 |
| **hw subtotal** | **≈1,294** | **≈1,094** |
| test/burn-in | €160 | €95 |
| contingency 15% | €215 | €180 |
| **total** | **≈1,650–1,700** | **≈1,240–1,300** |

## Decision impacts
- **€1,000 planning baseline is exceeded in both scenarios** once everything is included (prototype + test). The original budget cannot cover a first build with today's Verdin pricing (€410 list).
- Budget flexibility was approved for the aluminium shell, but the biggest single line is now the **SoM (~€350–490)**. Options to respect €1,000 (hardware-only, no test contingency):
  - take **4GB Verdin** (€294) + KingSpec SSD → hw subtotal ≈ **€1,050**; plus contingency pushes past €1,200 still.
  - Defer LTE (~€37) and/or trim chassis anodize, or reuse donor laptop (battery/BMS, hinges).
  - The **only way under €1,000 hardware-only** is 4GB Verdin + lean SSD and dropping LTE; otherwise the SoC price is the floor.

## §4 Import/safety caveats (to confirm in Phase 8/9)
1. **21700 cells (HS 8507)**: EU import duty is not 0% for batteries — DE may charge ~0–2.7% (8507.60.00) + **REACH/ROHS** and **UN38.3 + transport (ADR 3480)** for lithium cells by air; suppliers must be certified. Must verify before ordering cells from NL/elsewhere. (Stale-price + regulatory caveat — legal specialist review if commercial.)
2. **Verdin list price (00701100) €410** is negotiated-down in batch — a quote at qty 10+ yields ~5–15% less; not assumed here.
3. **Anodizing quote range €100–180** for ~2.7 kg chassis — get 2–3 eloxal quotes (local).
4. Battery pack build + hot-swap circuit **requires professional electrical/battery safety review before fabrication** (REQ-PWR-05).