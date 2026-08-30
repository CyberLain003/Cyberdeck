# Alternatives (Phase 4)

Status: 2026-08-30. To be re-warmed if lead leading option fails in Phase 4/5 research or Phase 6 fit checks.

## Compute (SoM)
| Alt | When to use | Gap vs leading |
|---|---|---|
| **RPi CM5** | If Verdin EU price ≥ ~€600; if NixOS evidence for Verdin stays absent | 0.4 mm B2B (routing risk); no SATA (needs PCIe bridge anyway); idle ~2 W (still feasible); **cheapest + best OS ecosystem** |
| **Other i.MX8M Plus SoMs** (research: SolidRun/Architek-class ones exposing **native SATA** on a 0.5-mm B2B) | If we can find native-SATA + 0.5-mm-pitch + mainline at lower cost than Verdin | would remove PCIe→SATA bridge; verify price/availability |
| **Rockchip RK3588 SoM** | Only if runtime target is relaxed (idle 3–5 W) and mainline acceptable | higher idle; weak NPU/VPU mainline; no EU distro |
| NXP official i.MX8MP Evo-whatever / eval — not portable | — | eval boards not B2B to custom carrier |

## SSD (REQ-COMP-03 — resolved to NVMe)
| Alt | When | Gap |
|---|---|---|
| **NVMe M.2 2280 (M-key, PCIe Gen3 ×1)** — leading (DEC-033) | Default | none — drops bridge chip entirely |
| M.2 SATA B+M-key (original REQ-COMP-03 wording) | Only if driver/NVMe issues | would reintroduce PCIe→SATA bridge; conflicts with keying-block choice — superseded |

## Display
| Alt | When | Gap |
|---|---|---|
| **Vu8S (5:8, $39, official)** — leading (DEC-035) | Default (cheapest) | 5:8 not 4:3; brightness TBD |
| **HE080IA-01E + separate backlight** (true 4:3 DSI) | If 4:3 required | backlight sourcing + power unknown |
| **HOTHMI 1000-nit LVDS** | Buy brightness at 4:3; accept LVDS + LVDS driver | extra driver board; ~3 W BL |
| **Raystar 1125-nit DSI (5:8 portrait)** | brightness + DSI when ratio not prioritized | _not 4:3_ — violates ratio preference |

→ Display decision is an open gate (runtime power is the lead constraint).

## Battery
| Alt | When | Gap |
|---|---|---|
| P45B | if budget-short (cheapest cells at NKON list? no — 50E cheaper/Wh) | lower margin at 30 h |
| **Samsung 50E** | Best Wh/€ at €3.45 | 70.8 mm length → holder 71 mm |
| **P50B (leading default)** | Best runtime margin + best-documented | €6.75/cell (€54/8) |

## LTE
| Alt | When | Gap |
|---|---|---|
| SIM7600E-H (€32.76, no B28) | if EC25-EUX stock unavailable | fewer bands; option-driver |
| EG25-G | Pi-ecosystem compatibility | EOL risk; more expensive |

## Wi-Fi/BT
| Alt | When | Gap |
|---|---|---|
| RTL8822CS SDIO | if need 2×2 MIMO | SDIO slot + external antenna needed |
| Any 802.11n USB dongle w/ PCB antenna | budget fallback | less tidy integration |

## Fan
| Alt | When | Gap |
|---|---|---|
| Delta BFB0305HA-C blower | if static pressure needed against heatsink | louder (29 dBA), 0.65 W |

## Marked TBD / to resolve in Phase 4 continuation
- NVMe M.2 2280 exact model + price (low-power, DRAM-less preferred).
- Vu8S measured brightness + backlight power (RISK-018).
- Verdin 8GB EU/US price + lead time (RISK-017) — CN/US import ok (DEC-036).
- Charger IC (BQ25713) exact part + price.
- Trackball ball-size test (ADNS small-ball viability).
- Pogo connector mechanical proof or branded equivalent.