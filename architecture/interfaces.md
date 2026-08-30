# Interfaces and Bus Topology (Phase 3)

Status: **First-cut topology** — 2026-08-30. Lanes/roles listed; verify per chosen SoM in Phase 4. Serves as the input to the carrier/daughterboard split (RISK-015).

## System interface map

| # | Signal | Interface | From → To | Through | Sharing / contention | Requirement | Status |
|---|---|---|---|---|---|---|---|
| I-01 | Storage | **SATA 3.0 (6 Gbit)** | SoM (native SATA) → M.2 2280 SATA SSD | Dedicated SATA x1 | None (dedicated) | REQ-COMP-03 | Open (verify native SATA) |
| I-02 | Storage (alt) | PCIe x1/x2 → SATA bridge | SoM → bridge chip (JMB/JMS) → SSD | PCIe lane shared only if bridge | Only if CM-family | REQ-COMP-03 | Fallback |
| I-03 | Display | HDMI 1.4 (1080p60) | SoM HDMI TX → driver board → panel (eDP/LVDS) | Dedicated | None | REQ-IO-01/REQ-DISP-01 | Open (native HDMI) |
| I-04 | Display (alt) | MIPI-DSI | SoM DSI → panel | 4-lane | None | REQ-DISP-01 | Investigate |
| I-05 | USB3 host | USB 3.0 (5 Gbit) | SoM USB3 host → dedicated USB-C/A port | **Unshared** (no hub) to meet ≥1 Gbit/s effective | Keep unshared | REQ-IO-02 | Open |
| I-06 | USB2 OTG | USB 2.0 | SoM OTG-capable port → port | none | REQ-IO-03 | Open (OTG capable) |
| I-07 | Ethernet | GbE (RGMII) or USB3→GBE (RTL8153-class) | SoM or bridge → RJ45 | Dedicated (or USB3, then hub) | REQ-IO-04 | Open (prefer RTL8153 dongle-class) |
| I-08 | SD slot | SD 3.0 (UHS-I, ≤104 Mbit) | SoM SDIO → full-size slot | Dedicated | REQ-IO-05 | Open |
| I-09 | USB-C PD | USB-C (power only) + PD 3.0 sink (~65 W) | external adapter → PD sink → charger | Dedicated 65 W path | REQ-IO-06 | Open (sink value/PD) |
| I-10 | Wi-Fi/BT | SDIO or USB/UART + BT | SoM ↔ module (e.g., RTL8822/88x-class or AMG-compatible) → ceramic/PCB antenna (top window) | one of SDIO/USB | REQ-RF-01 | Open (≥6 MB/s target; module choice) |
| I-11 | LTE/5G | USB or mPCIe | SoM ↔ module → nano-SIM + antenna(s) (rear window) | USB (or mPCIe logical) | REQ-RF-02 | Open (bands/brand) |
| I-12 | Audio out | I2S | SoM → line-out codec → 3.5 mm AUX; BT audio separate | none | REQ-KB-02/REQ-ENC-03 | Open (no speaker amp) |
| I-13 | Keyboard | I2C/USB matrix | SoM ↔ membrane (6-row) + trackball (SPI/I2C? verify module) | - | REQ-KB-01 | Open |
| I-14 | Power MCU link | I2C (SMBus-style) + UART | SoM ↔ power-manager MCU (daughterboard) | low-speed control + status; power sequencing GPIOs | REQ-PWR-03/04 | Open |
| I-15 | Fan | PWM + tach | power MCU ↔ 25–30 mm fan | none | (thermal) | Open |
| I-16 | pogo UART | UART (level-shifted) | SoM UART → level-shift (3.3/5 V) + ESD/polyfuse → pogo magnetic contacts (signal+GND default) | dedicated; power pending safety review | REQ-UART-01/02 | Open (front-end design Phase 6) |
| I-17 | Battery contact | Power (14.4 V rail) | pack A/B → OR-ing → charger/rails | shared battery rail via ideal-diode | REQ-PWR-01/03 | Open (inrush/pre-charge) |
| I-18 | Power split rails | 12 V / 5 V / 3.3 V | battery rail → DC-DC | shared by loads | REQ-PWR - | Open (eff. ~90%) |

## Board split (carrier vs daughterboard)

| Signal group | Board | Notes |
|---|---|---|
| High-speed (HDMI, USB3, SATA/PCIe, USB2, SDIO, DSI) | **Carrier only** — never crosses board gap | Keep impedance control local (RISK-015) |
| Power rails + battery contacts + PD charger | **Daughterboard** | High-current path short |
| Control (I2C, UART, PWM, GPIO, LED) | Between boards (thin FPC) | Low-speed, cheap |
| Antenna feeds | Carrier → lid/ top window via coax (U.FL→aperture) | RF windows in aluminum |

## Open interface questions (carried to Phase 6)
- Whether UART pogo must reach multiple SoM UARTs (mux) — target set for serial consoles MCUs servers.
- Exact trackball interface (SPI/I2C/USB) per module; some are USB — then it joins a hub (contention noted).
- Fan form: blower vs axial within 5–7 mm height.
- PD sink required power class (65 W) — verify exact PD controllers that also feed charging of 4S at 65 W.