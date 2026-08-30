# System Overview and Block Diagram (Phase 3)

Status: **Architecture draft for approval** — 2026-08-30. Leading choices below are **Provisional** (per strict evidence rule) until Phase-4 sourcing provides dimensions, power, interface and NixOS evidence. Dimensions/bus values flagged in each row.

## Functional Partitioning

The cyberdeck is split into two physical zones in the base (per user DEC-018) plus the display lid:

| Zone | Contents | Interfaces |
|---|---|---|
| **Display lid** | 8" XGA 4:3 panel (≥500–800 nit), display driver board, glass/frame; antenna windows for Wi-Fi/BT/LTE | Hinge flex: display signal, antenna feeds |
| **Front of base** | Two hot-swappable 4S battery bays (packs slide in from front edge) | Battery contacts → rear daughterboard power rails |
| **Rear of base — daughterboard** | I/O + power concentration: USB-C PD sink, HDMI/USB3/USB2/ETH/SD/AUX/pogo-UART connectors, power manager MCU, battery contact blocks, fan driver | To carrier (power, I2C, UART, control), to packs, to chassis connectors |
| **Mid/rear of base — main carrier** | SOM (via B2B), M.2 SATA SSD, Wi-Fi/BT module + ceramic/PCB antenna, LTE module + antenna(s), split-rail DC-DC, heatsink + micro fan | To daughterboard (power + low-speed), to lid (display), antennas |

## Block Diagram (ASCII overview)

```
 ┌──────────────────────────── DISPLAY LID ────────────────────────────┐
 │  [ 8" XGA 4:3 panel (eDP/LVDS) ]──[ driver board ]                 │
 │  antennas: WiFi/BT ceramic (top window)  ·  LTE (rear window)      │
 └─────────────────────────────────────────────────────────────────────┘
             │  hinge: display signal flex + antenna feeds (RF windows)
 ┌───────────┴──────────────────────── BASE ───────────────────────────┐
 │  TOP DECK: keyboard (≥5–6 rows, ~12 mm pitch) + trackball (≤20 mm)  │
 │                                                                     │
 │  ┌─────────── battery bay A ─────────┐  ┌───── battery bay B ────┐  │
 │  │  4S1P 21700 pack (front-insert)   │  │  4S1P 21700 pack       │  │
 │  └────────────────────────────┬─(plug)└─────────────────────────┘   │
 │                (front)  ◄═══ packs slide in ═══► (rear contact)     │
 │  ┌──────────────────────────────────────┐  ┌─────────────────────┐  │
 │  │       MAIN CARRIER (mid/rear)        │  │  DAUGHTERBOARD (rear)│  │
 │  │  SOM ─ B2B ─ (+heatsink 5–7 mm +     │  │  USB-C PD 65W sink   │  │
 │  │     micro fan)                       │  │  HDMI / USB3 / USB2-OTG│ │
 │  │  M.2 SATA 2280 SSD                   │  │  ETH / SD / AUX       │  │
 │  │  WiFi/BT module + ceramic antenna    │  │  pogo UART            │  │
 │  │  LTE module + antenna(s)             │  │  power-manager MCU    │  │
 │  │  rail DC-DC (5 V/3.3 V/12 V)         │  │  battery contact A/B  │  │
 │  └──────────────────────────────────────┘  │  fan driver, LEDs     │  │
 │                                            └─────────────────────┘  │
 └─────────────────────────────────────────────────────────────────────┘
```

## Power Architecture (topology-level)

- **Two 4S packs** (≈14.4 V) on a shared battery rail via **OR-ing (ideal-diode FETs) + pre-charge** — true hot-swap (ThinkPad Power Bridge style).
- **Single USB-C PD sink (65 W)** is the charge adapter. Charger feeds the pack management circuit; system rail runs from pack rail.
- **Power-manager MCU** (on daughterboard): battery voltage/current sense per pack, OR-ing/pre-charge sequencing, charge priority/taper, fan profile, LED state, power button, and UART pogo level-switch control. Communicates with the SOM over I2C/UART.
- Split-relic: battery rail → 12 V / 5 V / 3.3 V rails (efficiency ~90% assumed, verify).

## Interfaces (summary; detail in `interfaces.md`)

| From | To | Bus | Notes |
|---|---|---|---|
| SOM | SSD | **SATA 3.0** | native SATA (i.MX8MP-class) or PCIe→SATA bridge (CM-class) |
| SOM | Display | HDMI or DSI | leading: HDMI→eDP driver board |
| SOM | USB3 | USB 3.0 host (dedicated port) | keep unshared for ≥1 Gbit/s effective |
| SOM | USB2 OTG | USB 2.0 | OTG-capable port |
| SOM / bridge | ETH | RGMII or USB3→GBE RTL8153-class | ≥100 Mbit required, 1 Gbit preferred |
| SOM | WiFi/BT | SDIO/USB/UART + ceramic antenna | ≥6 MB/s throughput target |
| SOM | LTE | USB or mPCIe | nano-SIM on carrier; Telekom bands |
| SOM | Audio | I2S → line-out codec + BT | no internal speaker |
| SOM | Keyboard | I2C matrix (or USB module) | custom membrane + trackball |
| SOM | Power MCU | I2C/UART + GPIO | charge/fan/LED/level-switch control |
| SOM | pogo UART | UART → level-shift/protection | 3.3 V/5 V switchable, signal+GND default |

## Z-Stack (first-cut, to refine in Phase 6)

Base bottom-up (total 50 mm):
1. Bottom aluminum wall ≈ 1.5 mm
2. Battery bays (cells 24 mm incl. pack wall) — front region
3. Carrier + daughterboard plane + heatsink 5–7 mm + fan ≈ 30 mm path, vents to rear/side
4. Keyboard deck (top surface)

Lid: panel + driver + front glass + hinge.

> First-cut stack is conceptually consistent (needs ~4 mm gaps, hinge clearances — Phase 6). Battery CoM is now **front-heavy** (front insertion) — mitigated by rear power/IO weight; typing keeps front loaded. Fine for a laptop-like device.

## Open architecture items carried to Phase 6

- Final z-stack, hinge location, antenna aperture placement.
- Whether all high-speed lines stay on the carrier (vs crossing to daughterboard) — DEC/RISK-015.
- Trackball module choice; keyboard row/pitch final; function-row decision.
- 4S vs 2S pack confirmation.