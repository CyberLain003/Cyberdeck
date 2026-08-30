# System Overview and Block Diagram (Phase 3)

Status: **Architecture, reconciled to Phase-6 decisions** — 2026-08-30. Leading rows below reflect the approved current state (DEC-033/044/046/052/058/061/062). Dimensions/bus values per the top-level diagrams; detailed wiring in `electrical/block-diagram.md`.

## Functional Partitioning

The cyberdeck's base is a **three-plane flat stack** (DEC-052) plus the display lid — keyboard PCB (top) / carrier motherboard (mid) / rear daughterboard (bottom, low-freq I/O + power):

| Zone | Contents | Interfaces |
|---|---|---|
| **Display lid** | **Raystar RFU800G-AYH-MNN** 8" panel (800×1280 DSI 4-lane, 1125 nit) + backlight + touch; **plastic bezel + RF-transparent plastic lid-top** behind screen for antennas | Hinge flex: DSI + BL PWM + I2C + power; Wi-Fi/BT coax (EDGE-03); LTE coax (2×) |
| **Deck — keyboard PCB (top plane)** | Custom 6-row membrane + trackball (below space bar, between G-H-B) + **deck USB-HID MCU (STM32G0-class)**; keyboard top edge at **10% depth**, ends at **40% from the bottom** (DEC-061); palm-rest = front 40% | To carrier: USB2 (deck HID) |
| **Front of base** | Two hot-swappable 4×21700 battery bays on **drill-style rails in the middle chassis spine**, front-insertion, hot-swap (OR-ing) | Battery contacts → rear daughterboard power rails |
| **Rear of base — daughterboard (bottom plane)** | Power concentration + ALL low-frequency ports: USB-C PD sink, power-manager MCU, battery contact blocks, fan driver; DB holds power mgmt + pogo UART front-end | To carrier (power, I2C, UART, control), to packs, to chassis connectors |
| **Mid plane — carrier motherboard** | SoM (via B2B), M.2 2280 **NVMe** (M-key), **on-module Wi-Fi/BT (SoM "WB") antenna feed**, tiny-LGA **LTE** module + nano-SIM, USB2 hub, heatsink 5–7 mm + 30 mm fan | To daughterboard (power + low-speed), to lid (display + antennas), to deck (HID) |

## Block Diagram (ASCII overview)

```
 ┌──────────────────────────── DISPLAY LID ────────────────────────────┐
 │  [ Raystar 8" 800×1280, DSI 4-lane, 1125 nit ] (no driver board)   │
 │  antennas (RF-transparent plastic lid-top, DEC-055):                │
 │     Wi-Fi/BT coax (u.FL, from SoM)  ·  LTE coax (2×)                │
 └─────────────────────────────────────────────────────────────────────┘
             │  hinge: DSI flex + BL PWM + Wi-Fi/BT coax + LTE coax
 ┌───────────┴──────────────────────── BASE ───────────────────────────┐
 │  FLAT PLANE STACK (DEC-052, top→bottom): deck / carrier / daughter  │
 │                                                                     │
 │  TOP PLANE — KEYBOARD PCB: 6-row membrane + trackball (below space) │
 │      keyboard 10% from top → ends 40% from bottom; palm rest =      │
 │      front 40%; deck USB-HID MCU (STM32G0)                          │
 │                                                                     │
 │  ┌─────────── battery bay A ─────────┐  ┌───── battery bay B ────┐  │
 │  │  4×21700 pack, front-insert       │  │  4×21700 pack          │  │
 │  │  (drill-style rails in mid spine) │  │                        │  │
 │  └────────────────────────────┬─(contact)────────────────────────┘  │
 │                (front)  ◄═══ packs slide in ═══► (rear contact)     │
 │  ┌──────────────────────────────────────┐  ┌─────────────────────┐  │
 │  │  CARRIER MOTHERBOARD (mid plane)     │  │ REAR DAUGHTERBOARD  │  │
 │  │  SOM ─ B2B ─ (+heatsink 5–7 mm +     │  │ (bottom plane):     │  │
 │  │     30 mm Delta blower, right palm-  │  │  USB-C PD 65W sink   │  │
 │  │     rest zone, air L-back→R-exhaust) │  │  HDMI / USB3 / USB2  │  │
 │  │  M.2 2280 NVMe (M-key)               │  │  ETH / SD / AUX      │  │
 │  │  on-module Wi-Fi/BT (soM WB) feed    │  │  pogo UART front-end │  │
 │  │  LTE tiny-LGA (EC200U-class)+nanoSIM │  │  power-manager MCU   │  │
 │  │  USB2 hub (USB2513)                  │  │  battery contacts A/B│  │
 │  └──────────────────────────────────────┘  │  fan driver, LEDs    │  │
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
| SOM | SSD | **PCIe Gen3 ×1 → NVMe** | M.2 2280 M-key, silkscreen "NVMe 2280 ONLY" (DEC-033/048; no SATA) |
| SOM | Display | **MIPI-DSI 4-lane → Raystar RFU800G** | 8", 800×1280, 1125 nit (DEC-044); HDMI-out is a separate sink port |
| SOM | USB3 | USB 3.0 host (dedicated port) | keep unshared for ≥1 Gbit/s effective |
| SOM | USB2 OTG | USB 2.0 | OTG-capable port |
| SOM / on-module | ETH | **GBE on-module PHY (KSZ9131, ETH_1)** | ≥1 Gbit; no external PHY in v1 |
| SOM | WiFi/BT | **On-module (Verdin "WB")** → antenna coax → plastic lid-top | ≥6 MB/s throughput target (DEC-046) |
| SOM | LTE | **USB2 (USB2513 hub) → tiny-LGA module (EC200U-class)** | nano-SIM on carrier; Telekom bands B1/3/7/8/20 (DEC-058) |
| SOM | Audio | I2S → line-out codec + BT | no internal speaker |
| SOM | Keyboard | **USB2 → deck USB-HID MCU (STM32G0)** | custom membrane + trackball (SPI→HID) |
| SOM | Power MCU | I2C/UART + GPIO | charge/fan/LED/level-switch control |
| SOM | pogo UART | UART → level-shift/protection | 3.3 V/5 V switchable, signal+GND default |

## Z-Stack (first-cut, to refine in Phase 6)

Base bottom-up (total 50 mm, flat planes):
1. Bottom aluminum wall ≈ 1.5 mm
2. Battery bays (cells 24 mm incl. pack wall) on mid-chassis drill rails — front region
3. **Flat planes:** keyboard PCB (top) / carrier motherboard / rear daughterboard; heatsink 5–7 mm + 30 mm fan in the right palm-rest zone, **air path left-back intake → fan → heatsink fins L→R → right-side exhaust** (DEC-062)
4. Keyboard deck / palm rest (top surface)

Lid: panel + driver + front glass + hinge.

> First-cut stack is conceptually consistent (needs ~4 mm gaps, hinge clearances — Phase 6). Battery CoM is now **front-heavy** (front insertion) — mitigated by rear power/IO weight; typing keeps front loaded. Fine for a laptop-like device.

## Open architecture items carried to Phase 6/7

- Final z-stack, hinge location, antenna placement on the plastic lid-top (DEC-055).
- **LTE tiny-LGA final part** — Cat-1 bands/power/price re-research (DEC-058).
- Hinge vendor + torque spec (~0.6–1.0 N·m/side ThinkPad-class, DEC-054).
- Fan PWM/tach wiring (Delta blower 2-wire, RISK-027).
- Trackball small-ball bench gate (RISK-020); keyboard 10%/40% + palm-rest cutout final.