# Alternatives (Phase 4)

Status: 2026-08-30. To be re-warmed if lead leading option fails in Phase 4/5 research or Phase 6 fit checks.

## Compute (SoM)
| Alt | When to use | Gap vs leading |
|---|---|---|
| **RPi CM5** | If Verdin EU price ≥ ~€600; if NixOS evidence for Verdin stays absent | 0.4 mm B2B (routing risk); no SATA (needs PCIe bridge anyway); idle ~2 W (still feasible); **cheapest + best OS ecosystem** |
| **Other i.MX8M Plus SoMs** (research: SolidRun/Architek-class ones exposing **native SATA** on a 0.5-mm B2B) | If we can find native-SATA + 0.5-mm-pitch + mainline at lower cost than Verdin | would remove PCIe→SATA bridge; verify price/availability |
| **Rockchip RK3588 SoM** | Only if runtime target is relaxed (idle 3–5 W) and mainline acceptable | higher idle; weak NPU/VPU mainline; no EU distro |
| NXP official i.MX8MP Evo-whatever / eval — not portable | — | eval boards not B2B to custom carrier |

## SSD (REQ-COMP-03)
| Alt | When | Gap |
|---|---|---|
| PCIe→SATA via **JMicron JMS582 / ASMedia ASM1062** (TQFP) | Default (leading) | adds one chip + routing; cheap (~€5–10) |
| **NVMe M.2** (drop SATA requirement, user-approved malleability) | If SATA unavailable/bridge fails | changes REQ-COMP-03 wording (Preference — allowed w/ approval) |

## Display
| Alt | When | Gap |
|---|---|---|
| **HE080IA-01E + separate backlight** (true 4:3 DSI) | Prefer 4:3 + DSI; backlight power TBD | backlight sourcing + power unknown (~3–4.5 W est) |
| **HOTHMI 1000-nit LVDS** | Buy brightness at 4:3; accept LVDS + LVDS driver | extra driver board; ~3 W BL |
| **Raystar 1125-nit DSI (5:8 portrait)** | Brightness + DSI + brightness > daily | _not 4:3_ — violates ratio preference |
| **Vu8S / Waveshare 8" DSI (off-the-shelf kit)** | Lowest risk of getting a working panel fast; official $39 | not 4:3; brightness unstated |

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
- JMicron/ASMedia SATA-bridge exact part, price, footprint.
- M.2 2280 SATA SSD exact model + price.
- Backlight unit/power for HE080IA-01E.
- Trackball ball-size test (ADNS small-ball viability).
- Pogo connector mechanical proof or branded equivalent.
- Verdin 8GB EU price + lead time (authoritative quote).