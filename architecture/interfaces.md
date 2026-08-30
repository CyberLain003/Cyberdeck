# Interfaces and Bus Topology (Phase 3)

Status: **Topology, reconciled to Phase-6 decisions** — 2026-08-30. Lanes/roles listed; leading rows reflect the approved current state (DEC-033/044/046/052/058/062). Serves as the input to the carrier/daughterboard split (RISK-015).

## System interface map

| # | Signal | Interface | From → To | Through | Sharing / contention | Requirement | Status |
|---|---|---|---|---|---|---|---|
| I-01 | Storage | **PCIe Gen3 ×1 → M.2 2280 NVMe (M-key)** | SoM PCIe → M.2 2280 NVMe SSD (M-key, silkscreen-keyed) | Dedicated PCIe x1 | None (dedicated) | REQ-COMP-03 | **Approved (DEC-033/048)** — no SATA on Verdin |
| I-02 | Storage (alt) | ~~PCIe → SATA bridge~~ → **not applicable** | (superseded by DEC-033: direct NVMe; DEC-027 SATA-bridge plan dropped) | — | — | REQ-COMP-03 | **Superseded** (historical) |
| I-03 | HDMI out | HDMI 2.0 (1080p60+) | SoM HDMI TX → HDMI-A port (dedicated sink port) | Dedicated | None | REQ-IO-01 | Native (E-009); independent of DSI |
| I-04 | Display | **MIPI-DSI 4-lane** | SoM DSI → carrier FFC adapter → **Raystar RFU800G-AYH-MNN** (8", 800×1280, 1125 nit) | 4-lane | None | REQ-DISP-01/02 | **Approved (DEC-044)** — leading; OD on 200×140 lid ✓ (bench) |
| I-05 | USB3 host | USB 3.0 (5 Gbit) | SoM USB3 host → dedicated USB-C/A port | **Unshared** (no hub) to meet ≥1 Gbit/s effective | Keep unshared | REQ-IO-02 | Open |
| I-06 | USB2 OTG | USB 2.0 | SoM OTG-capable port → port | none | REQ-IO-03 | Open (OTG capable) |
| I-07 | Ethernet | **GbE (on-module PHY KSZ9131, ETH_1)** | SoM → integrated-magnetics RJ45 (no external RGMII PHY in v1) | Dedicated | REQ-IO-04 | **Resolved (DEC-014 superseded)** — native PHY |
| I-08 | SD slot | SD 3.0 (UHS-I, ≤104 Mbit) | SoM SDIO → full-size slot | Dedicated | REQ-IO-05 | Open |
| I-09 | USB-C PD | USB-C (power only) + PD 3.0 sink (~65 W) | external adapter → PD sink → charger | Dedicated 65 W path | REQ-IO-06 | Open (sink value/PD) |
| I-10 | Wi-Fi/BT | **On-module (Verdin "WB" variant)** | SoM on-module radio ↔ antenna coax (u.FL) → RF-transparent plastic lid-top (behind screen) | RF coax; no USB/SDIO lane | dedicated | REQ-RF-01 | **Approved (DEC-046)** — RTL8821CU module dropped |
| I-11 | LTE/5G | **USB 2.0 (via low-power hub)→ tiny-LGA module** | SoM ↔ tiny-LGA radio (~ESP32-WROOM footprint, ≤~30 mm; EC200U-EU/BG95/SIM7000-class) → nano-SIM + antennas | USB2 via USB2513-class hub | REQ-RF-02 | **Provisional (DEC-058)** — mPCIe EC25 superseded as leading; bands/price re-research |
| I-12 | Audio out | I2S | SoM → line-out codec → 3.5 mm AUX; BT audio separate | none | REQ-KB-02/REQ-ENC-03 | Open (no speaker amp) |
| I-13 | Keyboard | USB-HID matrix | SoM ↔ deck MCU (STM32G0-class, USB-HID) scanning membrane (6-row) + trackball (SPI) | - | REQ-KB-01 | Open (Option A deck MCU recommended, DEC-045) |
| I-14 | Power MCU link | I2C (SMBus-style) + UART | SoM ↔ power-manager MCU (daughterboard) | low-speed control + status; power sequencing GPIOs | REQ-PWR-03/04 | Open |
| I-15 | Fan | low-side PWM / voltage control | power MCU ↔ **Delta BFB0305HA-C blower (primary)**; Sunon HA30101V4 axial fallback | none; 2-wire, no tach (RISK-027) | (thermal) | **Resolved (DEC-062)** — left-back intake → right exhaust |
| I-16 | pogo UART | UART (level-shifted) | SoM UART → level-shift (3.3/5 V) + ESD/polyfuse → pogo magnetic contacts (signal+GND default) | dedicated; power pending safety review | REQ-UART-01/02 | Open (front-end design Phase 6) |
| I-17 | Battery contact | Power (14.4 V rail) | pack A/B → OR-ing → charger/rails | shared battery rail via ideal-diode | REQ-PWR-01/03 | Open (inrush/pre-charge) |
| I-18 | Power split rails | 12 V / 5 V / 3.3 V | battery rail → DC-DC | shared by loads | REQ-PWR - | Open (eff. ~90%) |

## Board split (carrier vs daughterboard)

| Signal group | Board | Notes |
|---|---|---|
| High-speed (HDMI, USB3, PCIe/NVMe, USB2, SDIO, DSI) | **Carrier only** — never crosses board gap | Keep impedance control local (RISK-015). Three flat planes (DEC-052): keyboard PCB (top) / carrier motherboard / rear daughterboard |
| Power rails + battery contacts + PD charger | **Daughterboard** | High-current path short |
| Control (I2C, UART, PWM, GPIO, LED) | Between boards (thin FPC) | Low-speed, cheap |
| Antenna feeds | Carrier → lid via U.FL coax (Wi-Fi/BT up-the-hinge; LTE 2×) | RF-transparent plastic lid-top behind screen (DEC-055) |

## Open interface questions (carried to Phase 6/7)
- Whether UART pogo must reach multiple SoM UARTs (mux) — target set for serial consoles MCUs servers.
- Exact trackball interface (SPI) per module; the deck MCU (USB-HID) owns matrix + trackball (DEC-045), so no raw SoM SPI path is needed.
- Fan PWM/tach: **Delta BFB0305HA-C is 2-wire with no tach** — low-side PWM/voltage control compatibility TBD (RISK-027); Sunon axial (fallback) also has no PWM/tach.
- PD sink required power class (65 W) — verify exact PD controllers that also feed charging of 4S at 65 W.