# Final System Block Diagram (Phase 6)

Status: **Phase-6 reference for Phase 7 (software) and Phase 8 (acceptance)** — 2026-08-30.
This document is the authoritative wiring map: every SoM interface is numbered and traced to its module, with bus, speed, sharing and board location. `TBD` items are flagged.

Inputs: `architecture/interfaces.md` (I-xx), `architecture/system-overview.md`, DEC-014/015/017/018/025/030/033/034/038/043/044/046/049/052/053/058/061/062, RISK-015, RISK-021.

| Symbol | Meaning |
|---|---|
| EDGE-nn | numbered interface in the diagram below |
| Carrier / DB | main carrier PCB / rear daughterboard PCB |
| TBD | unverified: price, pin, part, or pending decision |

---

## 1. Block diagram (ASCII)

```
┌──────────────────────────────── DISPLAY LID ─────────────────────────────────┐
│            [ Raystar 8" 800×1280 DSI + backlight + touch ] (1125 nit)         │
└───────────────────────────────────────────────────────────────────────┬──────┘
        lid FFC: DSI 4-lane + touch I2C + BL PWM + power   EDGE-02      │
        Wi-Fi/BT antenna coax (u.FL, on-module SoM radio) up           │
        through hinge into RF-transparent plastic lid-top    EDGE-03   │
┌───────────────────────────────────────────────────────────────────────┴──────┐
│  FLAT PLANE STACK (DEC-052, top → bottom): ① keyboard PCB (deck) ② carrier   │
│  motherboard (SoM + M.2 + LTE + Wi-Fi feed) ③ rear daughterboard (all low-    │
│  freq ports + power mgmt + battery contacts).  Interconnect via FPC/J-conn.   │
│   ┌─ CARRIER MOTHERBOARD (mid plane) ─────────────────────────────────────┐   │
│   │                                                                       │   │
│   │  ┌──────────────────┐   EDGE-01 PCIe x1   ┌────────────────────────┐ │   │
│   │  │  Verdin SoM (WB) │────────────────────►│ M.2 2280 NVMe (M-key)   │ │   │
│   │  │ i.MX8MP 8GB (B2B)│─EDGE-04 HDMI ──────►│ HDMI-A port            │ │   │
│   │  │ on-module        │─EDGE-05 USB3 ──────►│ USB3-A port            │ │   │
│   │  │ Wi-Fi/BT (WB)    │─EDGE-06 USB2 ──────►│ USB2-C/A OTG           │ │   │
│   │  │ antenna→EDGE-03  │─EDGE-08 SDIO ──────►│ SD slot                │ │   │
│   │  │                  │─EDGE-07 GbE(ETH_1)─►│ RJ45 (on-module PHY)    │ │   │
│   │  │                  │─EDGE-09 USB(hub)───►│ LTE tiny-LGA + nano-SIM │ │   │
│   │  │                  │─EDGE-10 I2S/I2C ───►│ AIC3204 codec → AUX     │ │   │
│   │  │                  │─EDGE-11 USB(hub)───►│ deck USB-HID ctrl       │ │   │
│   │  │                  │─EDGE-12 UART ──────►│ (to DB pogo front-end)  │ │   │
│   │  │                  │─EDGE-13 ↔ FPC ─────►│ (to power-manager MCU)  │ │   │
│   │  └──────────────────┘                      │               ◄EDGE-17 │ │   │
│   │   air path (DEC-062): left-back intake louvres → 30 mm fan (Delta     │   │
│   │   BFB0305HA-C blower, primary) → 5–7 mm heatsink fins L→R → RIGHT-    │   │
│   │   side exhaust; SoM + heatsink + fan in the RIGHT palm-rest zone       │   │
│   └────────┼───────────────────────────────────┴───── EDGE-16 gap ───────┘   │
│            │   signal FFC (I2C/UART/PWM/GPIO/buttons/wake)                    │
│            │   + power harness (12/5/3.3V + GND)  (hardware/pcb.md §8/§12)    │
│            ▼                                                                  │
│   ┌─ REAR DAUGHTERBOARD (bottom plane — low-freq I/O + power) ──────────┐    │
│   │  USB-C PD sink (STUSB4500) ─► charger BQ25713 ─► 4S rail (EDGE-14)   │    │
│   │        │                          (OR-ing)  ├──► pack A / pack B     │    │
│   │        └── 20 V / 65 W in (USB-C, rear, power only)                   │    │
│   │  power-manager MCU: sense, sequencing, taper, fan, LED, power btn,    │    │
│   │   UART level-switch + pogo UART front-end (DB-E2)                     │    │
│   │  DC-DC split: 12 V / 5 V / 3.3 V (from 4S rail, ≈90% eff)             │    │
│   │  battery contact blocks at DB FRONT edge — drill-rail packs slide in   │    │
│   │  on mid-chassis rails, hot-swap via OR-ing (DEC-060)                   │    │
│   └───────────────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────────────────┘
```

*(Mermaid equivalent in Appendix A.)*

---

## 2. Interface table (numbered edges → bus, speed, sharing, board)

| EDGE | I-xx (interfaces.md) | Signal | From → To | Bus / speed | Sharing / contention | Board | Requirement | Status |
|---|---|---|---|---|---|---|---|---|
| 01 | I-01/02 | NVMe | SoM PCIe Gen3 ×1 → M.2 2280 **NVMe** (M-key, DEC-033) | **PCIe Gen3 ×1 (~985 MB/s)** | **dedicated** (unshared) | carrier | REQ-COMP-03 | Approved; keying+silkscreen block |
| 02 | I-04 | Display | SoM DSI 4-lane → carrier FFC adapter → **Raystar RFU800G** (lid) | **MIPI-DSI 4-lane**, 800×1280@60 | dedicated | carrier→lid (hinge FFC ≤150 mm) | REQ-DISP-01/02 | Approved (DEC-044); adapter/pinout TBD (RISK-024); **OD/power bench (DEC-044)** |
| 02b | I-04 | Touch + BL | SoM I2C + PWM (backlight) + power → Raystar RFU800G | I2C 400 kHz; PWM ~20 kHz | low-speed part of EDGE-02 FFC | carrier→lid | — | chipsets TBD |
| 03 | I-10 | Wi-Fi/BT | **SoM on-module Wi-Fi/BT (Verdin "WB" variant)** → antenna coax (u.FL) → plastic lid-top | RF coax (u.FL); no USB data lane | dedicated coax feed (no USB root contention) | carrier→lid (through hinge) | REQ-RF-01 | Approved (DEC-046); antenna placement on RF-transparent plastic lid-top (DEC-055) |
| 04 | I-03 | HDMI out | SoM HDMI 2.0 TX → HDMI-A port | HDMI 2.0 (up to 4k30/1080p60) | dedicated | carrier (rear edge) | REQ-IO-01 | native (E-009); independent of DSI (§3.5 display.md) |
| 05 | I-05 | USB3 host | SoM USB3.1 Gen1 → USB3-A port | **USB 3.0 (5 Gbit/s)** | **unshared** (REQ ≥1 Gbit/s effective) | carrier (rear edge) | REQ-IO-02 | fixed unshared |
| 06 | I-06 | USB2 OTG | SoM USB2 OTG-capable → USB2-C/A port | USB 2.0 | dedicated OTG | carrier (rear edge) | REQ-IO-03 | Type-C needs CC/ID logic (TUSB321-class) |
| 07 | I-07 | Ethernet | **SoM on-module GbE PHY (KSZ9131, ETH_1)** → integrated-magnetics **RJ45** | GbE (1000BASE-T) | dedicated | carrier (rear edge) | REQ-IO-04 | **supersedes DEC-014 USB3→GbE plan** (native PHY, no hub, USB3 stays free) — no external RGMII PHY in v1 (spare ETH_2 documented, NC) |
| 08 | I-08 | SD slot | SoM SDIO → full-size SD slot | SD 3.0 UHS-I (≤104 MB/s) | dedicated | carrier (rear edge) | REQ-IO-05 | — |
| 09* | I-11 | LTE | SoM USB2 (via USB2513-class hub) → **tiny-LGA LTE (EC200U-EU-class)** + nano-SIM + antennas | USB 2.0 HS (QMI → ModemManager) | shares hub with EDGE-11 (USB3 unshared → ≥1 Gbit/s holds) | carrier (mid plane) | REQ-RF-02 | Provisional DEC-058; Cat-1 bands/price re-research (TBD §6.8) |
| 10 | I-12 | Audio out | SoM I2S + I2C → **TLV320AIC3204** codec (or AIC3104-class; both mainline) → 3.5 mm **AUX** (stereo line out) | **I2S** (16/24-bit ≤192 kHz) + I2C control | dedicated; BT audio runs on-module over EDGE-03 radio | carrier (rear edge, E5) | REQ-KB-02 / REQ-ENC-03 | no internal speaker/mic/cam (A-015); mainline ASoC `tlv320aic32x4` / `aic3x` 🔲 final part |
| 11* | I-13 | Keyboard/trackball | SoM USB2 → **deck USB-HID controller** (matrix 6×14 + ADNS-9800 SPI → composite HID KB+mouse) | USB 2.0 (1 kHz report) | shares USB2 hub with EDGE-09 (USB2513-class; pcb.md §13), USB3 unshared | carrier → deck | REQ-KB-01 | Option A recommended (see keyboard-trackball.md §3); Option B=I2C MCP23017 fallback |
| 12 | I-16 | pogo UART | SoM UART (1.8 V) + control crosses FPC → **level-shift 3.3/5 V + ESD + polyfuse on daughterboard** → magnetic **pogo** (DB-E2) | UART up to ~1 Mbit | dedicated; safe default signal+GND | **daughterboard** (front-end), UART crosses via EDGE-16 | REQ-UART-01/02 | front-end design → hardware/uart.md 🔲 |
| 13 | I-14 | Power-MCU link | SoM I2C (SMBus-style) + UART + power GPIOs ↔ power-manager MCU | I2C 400 kHz + UART + GPIO | low-speed control/status | **crosses EDGE-16** | REQ-PWR-03/04 | part of EDGE-16 |
| 14 | I-17 | Battery/power | PD sink → charger → 4S rail → **OR-ing → packs A/B** | 20 V/65 W PD → 14.4 V rail | dedicated 65 W path | daughterboard | REQ-PWR-01/03/04/05 | STUSB4500+BQ25713 (DEC-031); **safety review gate** |
| 15 | I-15 | Fan | power MCU → **Delta BFB0305HA-C blower (primary)**; Sunon HA30101V4 axial fallback | low-side PWM / voltage control (2-wire, no tach — **TBD RISK-027**) | dedicated | daughterboard → carrier fan (in-plane left of heatsink) | (thermal) | DEC-017/062; see `hardware/thermal.md` §2 |
| 16 | I-18 | Cross-board | carrier ↔ daughterboard power + control | **signal FFC** (0.5 mm, ≥26–30p: I2C/UART/PWM/GPIO/buttons/wake, 1.8/3.3 V) + **power harness** (12/5/3.3 V + GND, ≥rated pins; 2.54 mm or Mini-Fit) | shared low-speed | board gap <50 mm | REQ-PWR - | high-speed never crosses (RISK-015); rated power interconnect TBD at Phase-6 electrical review (REQ-PWR-05 gate) |
| 17 | I-18 | Split rails | 4S rail → 12 V / 5 V / 3.3 V | DC-DC (~90%) | shared by loads | daughterboard | REQ-PWR - | 5 V feeds SoM (VDD_IN) + panel BL + USB/WiFi/deck |

> * EDGE-09 (LTE) + EDGE-11 (keyboard/trackball) are the two **USB2** consumers — served via the **low-power USB2513-class hub** (DEC-049) upstream of SoM USB_2 (host) (pcb.md §13); USB_1 stays OTG (EDGE-06). EDGE-03 (Wi-Fi/BT) is the **on-module radio's antenna coax** — no USB data lane (DEC-046). Audio (EDGE-10) uses I2S/I2C, not USB. The **external USB3 edge (05) and USB2-OTG edge (06) stay unshared** (REQ-IO-02); ≥1 Gbit/s effective unaffected.

---

## 3. Board split (carrier vs daughterboard) + cross-board gap

Detailed in `hardware/pcb.md` §8 and §12 (connector labels E1–E7 / DB-E1–E4; exact crossing list). Summary:

| Signal group | Board | Connector side | Why |
|---|---|---|---|
| **High-speed** (PCIe, DSI, HDMI, USB3, USB2, SDIO, GbE MDI) | **Carrier only** (never crosses to DB) | carrier rear edge: HDMI / USB3-A / USB2-OTG / RJ45 / SD / AUX | impedance control (RISK-007), short runs (RISK-015) |
| **Power + battery + PD + pogo front-end** | **Daughterboard** | rear DB edge: USB-C PD (power-only), pogo (DB-E2); battery contacts at DB **front** edge | high-current path short; battery bays cross the front |
| **Control** (I2C, UART, PWM, GPIO, buttons, wake) | Between boards — **signal FFC (EDGE-13/16)** | — | low-speed, cheap (RISK-015), list in pcb.md §12.1 |
| **Power rails** (12/5/3.3 V + GND) | Between boards — **power harness** (separate from signal FFC) | rated pin count TBD | high current; independence for routing (pcb.md §12.4) |
| Antenna feeds | Carrier → **plastic lid-top (RF-transparent, behind screen)** via U.FL coax (EDGE-03 Wi-Fi/BT; LTE 2×) | — | aluminum chassis is RF-opaque (RISK-013/021); DEC-055 |
| Lid signals | Carrier → lid FFC | through hinge (EDGE-02) | DSI + I2C + BL PWM + power (+ Wi-Fi/BT coax) |

- ✅ **DEC-018 reconciliation (consistency check):** `hardware/pcb.md` implements the recommended split — rear connectors (HDMI/USB/SD/RJ45/AUX) on the **carrier** rear edge; the **daughterboard** holds USB-C PD (power-only), pogo-UART front-end, power rails + battery contacts (front edge) + power-MCU. This is the Phase-6 approved direction; the decision-log entry may be updated to reflect "I/O connectors on carrier rear edge, DB = power/battery/control zone" (see open items).

---

## 4. Power zones (daughterboard side) and heat

```
USB-C PD 20V/65W ─► STUSB4500 ─► BQ25713 (4S buck-boost charger)
                                        │
                      battery rail  14.4 V  (OR-ing ideal-diode + pre-charge)
                         ▲     │     ▲
                  pack A / pack B    │ (hot-swap, EDGE-14)
                         │
           ┌─────────────┼──────────────┐
           ▼             ▼              ▼
         12 V rail     5 V rail      3.3 V rail   (≈90% eff)
         (reserved)    SoM VDD_IN · panel BL · USB · deck ctrl · fan
                            3.3 V: codec · LTE I/O · SD I/O · misc
```

- **Runtime math driver:** idle SoM 1.44–1.7 W (E-010) + panel ≤2.5 W @ working brightness (display.md §4) + fan ≤0.65 W (Delta blower max; Sunon axial fallback 0.3 W) + peripherals → target 4 W-class average (A-007) for 30 h. Panel/fan figures go through the Phase-6 bench before acceptance.
- **Safety:** battery/BMS/hot-swap/PD = safety-critical (REQ-PWR-05, RISK-008/023) → professional review gate before fabrication. DC-DC converting 14.4 V to 5/3.3 on daughterboard; high current short paths.
- **Heat:** air path per DEC-062 — **left-back intake louvres → 30 mm fan (Delta BFB0305HA-C blower primary, Sunon axial fallback) → 5–7 mm heatsink fins L→R → right-side exhaust** (`hardware/thermal.md`). Fan (EDGE-15) powered/controlled by the power MCU (low-side PWM TBD — RISK-027; Delta -C has no tach). SoM + heatsink + fan sit in the right palm-rest zone (DEC-053/061). Sustained 65 W charge-while-use → charge-taper on temp (RISK-016).

---

## 5. Notes on power delivery to the SoM

- Verdin VDD_IN is 5 V (carrier-supplied via B2B) — the **5 V rail** (EDGE-17) feeds the SoM; do **not** power discrete SoM rails from 3.3 V. Confirm exact rails in the Verdin datasheet power section (E-009) before layout.

---

## 6. TBD / open items (consolidated)

1. **Raystar OD/power bench** (display.md §6.1/§8) — confirm OD 115.74×184.93 mm fits the 194×134 mm lid interior (window 3/3/3/10) and backlight power at working brightness before CNC + acceptance (DEC-043/044).
2. **DEC-018 wording update** — decision-log note: rear connectors on **carrier** (pcb.md §8), DB = power/battery/pogo/control zone (implemented; log entry to reflect).
3. EDGE-07: resolved to **on-module KSZ9131 GbE PHY → RJ45** (pcb.md E4) — DEC-014 USB3→GbE plan superseded; spare ETH_2 RGMII documented-NC.
4. EDGE-10: codec final (AIC3204 vs AIC3104-class) + mainline ASoC confirm; AUX jack variant (stereo TRS).
5. EDGE-11: Option A deck USB-HID controller vs Option B I2C expanders; trackball small-ball test (RISK-020).
6. EDGE-02: J7 pinout / Verdin pinout mapping + BL topology (display.md V-2/V-3).
7. EDGE-14: BQ25713 exact part + charge-while-use thermal profile.
8. EDGE-09/11: **USB2513-class hub placement** + tiny-LGA LTE final part (Cat-1 bands/price re-research, DEC-058; candidates in `parts/candidates.md` §5) (pcb.md §13).
9. Cross-board gap: signal-FFC + power-harness rated parts + pinout freeze at Phase-6 electrical review (REQ-PWR-05 gate).

---

## Appendix A — Mermaid equivalent

```mermaid
flowchart TB
  subgraph Lid["DISPLAY LID"]
    P["Raystar 8" panel+BL+touch (DSI 4-lane, 1125 nit)"]
  end
  subgraph Base["BASE — 3 flat planes (DEC-052)"]
    subgraph Carrier["CARRIER MOTHERBOARD (mid plane)"]
      SOM["Verdin i.MX8MP 8GB (B2B, on-module Wi-Fi/BT)"]
      NVME["M.2 2280 NVMe (M-key)"]
      SS["USB3-A · USB2-OTG · HDMI · RJ45(GbE) · SD · AUX"]
      WIFI["on-module Wi-Fi/BT → antenna coax (EDGE-03)"]
      LTE["LTE tiny-LGA (EC200U-class) + nano-SIM"]
      AUD["AIC3204 codec → 3.5mm AUX"]
      KB["deck USB-HID (keyboard+trackball)"]
      FAN["Delta blower → HS fins L→R (left-back intake → right exhaust)"]
    end
    subgraph DB["REAR DAUGHTERBOARD (bottom plane)"]
      PD["USB-C PD sink STUSB4500"]
      CHG["Charger BQ25713"]
      OR["OR-ing + pre-charge"]
      PWRMCU["power-manager MCU"]
      RAILS["12V / 5V / 3.3V DC-DC"]
      BATS["battery bays A/B (4×21700, drill-rail mid-chassis)"]
      POGO["pogo UART front-end + mag. pogo"]
    end
  end
  SOM -- "EDGE-01 PCIe x1" --> NVME
  SOM -- "EDGE-02 DSI+BL+I2C" --> P
  SOM --> SS
  SOM -- "EDGE-03 antenna coax" --> WIFI
  SOM -- "EDGE-09 USB (hub)" --> LTE
  SOM -- "EDGE-10 I2S/I2C" --> AUD
  SOM -- "EDGE-11 USB (hub)" --> KB
  SOM -- "EDGE-13/16 (I2C/UART/GPIO)" --> PWRMCU
  PWRMCU -- "EDGE-15 fan control (PWM TBD)" --> FAN
  PD --> CHG --> OR --> BATS
  OR --> RAILS
  RAILS -- "EDGE-16/17 power harness" --> SOM
  PWRMCU --> RAILS
  POGO -- "EDGE-12/16 UART + control" --> PWRMCU
  POGO -- "EDGE-12 UART" --> SOM
```

---

*Reference for Phase 7 (DT binding: DSI panel node, pwm-backlight, codec, HID, modem/ModemManager, power-MCU protocol) and Phase 8 (acceptance: brightness/power draw, 30 h runtime, all I/O edges, safety review of EDGE-14).*