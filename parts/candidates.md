# Parts — Candidates (Phase 4)

Status: **Phase-4 sourced research** — 2026-08-30. Every row has source URL + access date. **Candidate = has evidence; not yet "Recommended"** (strict rule: Recommended requires dimensions + interfaces + Linux/NixOS + price/availability + envelope/power/thermal/budget impact — full verdict lands after budget model Phase 5 + acceptance planning). Prices are **snapshots**, stale within weeks.

## 1. Battery cells (8× 21700, 2× 4S1P packs)

All three meet ≥120 Wh total and ≥60 Wh/pack. Datasheet values from manufacturer PDFs (High confidence).

| Cell | Nominal | Wh/cell | Wh/pack (4S) | Wh total | Dia×H max | Weight | Wh/L · Wh/kg | Price EUR (NKON/DE) | Conf |
|---|---|---|---|---|---|---|---|---|---|
| **Molicel INR21700-P45B** | 4500 mAh @3.6 V | 16.2 | 64.8 | 129.6 | 21.55×70.15 | 70 g | 643 · 242 | €4.55 (NKON) / €6.50 (akkuteile.de) | High |
| **Samsung INR21700-50E** | 4900 mAh min @3.6 V | 17.6 | 70.6 | 141.2 | 21.25×70.80 | 69.5 g | n/a | €3.45 (NKON) | High |
| **Molicel INR21700-P50B** | 5000 mAh @3.6 V | 18.0 | 72.0 | 144.0 | 21.55×70.15 | 71 g | 714 · 260 | €6.75 (NKON) / €8.90 (akkuteile.de) | High |

Notes:
- **50E nominal is 4900 mAh min, NOT "4910"** (reseller figure). P50B is best-documented ~5 Ah option and cheapest per Wh.
- Design holder for ≥70.8 mm height, 21.6 mm diameter (max dims across sources).
- P45B at NKON is a **mixed batch** listing — batch traceability caveat for a custom pack.
- ≥60 Wh/pack guaranteed: P45B 64.8 / 50E 70.6 / P50B 72.0 → **REQ-PWR-01 met with all three**.

Sources: moliceel.com datasheets (P45B pdf `INR21700P45B_1.2…80109.pdf`; P50B pdf `…80122.pdf`), Samsung SDI cell-spec v1.1 mirror (dkwireless/WPG), NKON.nl + akkuteile.de listings. Access 2026-08-30.

## 2. Compute (SoM)

### Leading: Toradex Verdin i.MX 8M Plus (8 GB) — **strongest hobby-carrier fit**
Evidence (Toradex datasheet Rev 1.8 dated 2026-06-22, docs.toradex.com/116795; power page developer.toradex.com):

| Attribute | Evidence |
|---|---|
| CPU/RAM | 4× Cortex-A53 @1.6 GHz (8GB IT SKU), +M7; **8 GB LPDDR4 inline-ECC (PN 0070)**; 32 GB eMMC |
| Native interfaces | **USB 3.1 Gen1 host; USB 2.0 OTG; MIPI-DSI quad-lane (up to 1080p60); HDMI 2.0 TX; dual GbE (1× on-module PHY + 1× RGMII); PCIe Gen3 ×1 (Reserved)** |
| **SATA** | **NOT available on any pin** → SSD must use PCIe→SATA bridge (see §3) |
| B2B connector | **DDR4 SODIMM epage, 260-pin, 0.5 mm**; TE 2309409-2 socket; Toradex "Direct Breakout™" → diff pairs routable on 4-layer |
| Dimensions | 69.6×35.0×6.0 mm module; ~9–13 mm over carrier with socket |
| Power | vendor rating 1.5–6.3 W; **measured idle headless 1.44–1.7 W, idle+scren 1.7–2.5 W, STR 0.10–0.18 W, max ~5 W mean** |
| Linux | **mainline** (imx8mp-verdin.dtsi ≥v6.6 + U-Boot upstream + etnaviv GPU); **no published NixOS image** (soft gap to close in Phase 7) |
| Price (stale) | **volatile €250–950** (Mouser list ~$927 vs Arrow ~$293; EU pages bot-blocked today) |

### Alternatives (research summary)
| SoM | Pros | Cons | Verdict |
|---|---|---|---|
| RPi CM5 | Cheap (~$67–200), fully mainline, 2×HDMI+2×DSI+GbE+USB3 | **0.4 mm B2B ≈ near-BGA routing** (fails user constraint); no SATA; idle ~2 W | Conditional (routing risk high) |
| RK3588 SoM (BPI-RK3588) | most perf/€, 2×GbE | weak mainline (NPU/VPU/panfrost partial); no EU franchised distro; idle TBD | Conditional (runtime/mainline risk) |

## 3. SSD path (M.2 2280 SATA — REQ-COMP-03)

- Verdin lacks SATA ⇒ **PCIe Gen3 ×1 → SATA bridge** on carrier. **JMB582 / JMS582-class** (ASMedia/JMicron) — TQFP, user-routable. **Part + price not yet quoted (TBD, Phase 4 cont.)**.
- M.2 2280 SATA SSD: not sourced this pass (many options ~€55–90; see Phase 5).
- Note: i.MX8M **Plus SoC has native SATA** but Verdin does not route it; other i.MX8MP SoMs (see alternatives) may expose SATA — to research.

## 4. Display (8" 4:3-class; ≥100 ppi; MIPI-DSI accepted — DEC-024)

| Panel | Res / ppi | Ratio | I/F | Brightness | Backlight power | OD×thick | Price/avail | Conf |
|---|---|---|---|---|---|---|---|---|
| **HE080IA-01E** (Innolux/HannStar) | 1024×768 / **160 ppi** | **4:3** | **MIPI 4-lane** | bare cell (0 cd/m²) — **backlight not included** | **TBD** (matched BLU est ~3–4.5 W @ ≥500 nit — **unverified**) | 171.1×132.6×1.07 mm | Panelook/Youritech/Alibaba, ~$20–35 (stale, quote-only) | Secondary (specs consistent across 2+ mirrors) |
| **Raystar RFU800G-AYH-MNN** | 800×1280 / 189 ppi | 5:8 port | MIPI 4-lane | **1125 nits** | ~4.32 W max / ~2.2 W @50% PWM | 115.7×184.9×4.75 | quote-only (~$35–60 stale) | Primary |
| **HOTHMI HTM-H080D14-LVDS-A01R** | 1024×768 / 160 ppi | 4:3 | **LVDS** | **1000 nits** | ~3 W (implied) | 183×~7.1 (w/ RTP) | quote-only (~$45–70 stale) | Primary |
| **Hardkernel Vu8S** | 800×1280 / 189 ppi | 5:8 port | DSI kit | not published (~300–400 nit class) | TBD (5 V PWM) | 202×153 w/ bracket | **$39 in stock** | Primary (brightness TBD) |
| Waveshare 8" DSI (C) | 1280×800 / 189 ppi | 16:10 | DSI | not published | TBD | board-mount | $74.99 official | Primary (brightness TBD) |

- **HDMI→DSI bridge board** (if SoM HDMI-only, not needed for Verdin which has native DSI): **YD691MIPI-V1** (RGB/XGA/SVGA→DSI 2/4-lane; quote-only ~$25–45); chip-level Lontium LT8918. Not required for leading compute.
- **Runtime-critical:** panel backlight is the dominant power term (~2.5 W allowance). True-4:3 DSI (HE080IA-01E) needs a matched backlight (power TBD) → **display choice is the open decision under gate**.

## 5. LTE modem (cheapest, private use — DEC-026)

| Part | Bands (EU) | I/F | Size | Linux | Price/avail | Conf |
|---|---|---|---|---|---|---|
| **Quectel EC25-EUX** ✅ | B1/3/7/8/20/28A | USB2.0 HS (QMI → ModemManager) | mPCIe 51×30×4.9 (or LGA) | **qmi_wwan mainline** | **€31.51** (mPCIe, Avnet/Unikey) | High |
| SIMCom SIM7600E-H | B1/3/5/7/8/20 (no 28) | USB+UART+PCM (option/PPP) | LGA 30×30×2.9 | generic `option` | **€32.76** (TME etc.) | High |
| Quectel EG25-G | global, incl EU | QMI | LGA/mPCIe | QMI | €55–64 (disty); **EOL/harvest pressure — recheck** | Med |

**EC25-EUX wins**: cheapest, most complete EU bands incl B28, best Linux story. Module RED pre-qualified; final device RED integration responsibility stays (Phase 8).

## 6. Wi-Fi + BT (≥6 MB/s ≈ 48 Mbit/s — REQ-RF-01)

| Part | I/F | Bands | Driver (mainline) | Antenna | Price/avail |
|---|---|---|---|---|---|
| **RTL8821CU** (pad module BL-M8821CU1 / dongle) ✅ | USB 2.0 | 2.4/5 GHz 802.11ac 1×1, 433 Mbps | **rtw88_usb (≥6.2)** + btusb | onboard PCB antenna variant (or MHF→aperture) | €3.79 dongle / €6.85 pad (AliExpress DE) |
| RTL8822CS (BL-M8822CS1) | SDIO 3.0 | ac 2×2 866 Mbps | **rtw88_sdio (≥6.4)** | needs MHF external | €7.00 (AliExpress DE) |

- Any candidate vastly exceeds 6 MB/s. Leading = **RTL8821CU USB module**, PCB antenna relocated to an RF window in the aluminum deck (ceramic-antenna variant unverified — plan u.FL→aperture antenna). Firmware in `linux-firmware`.

## 7. Micro fan (thermal — DEC-017)

| Part | Size | V | Airflow | Noise | Power | Price/avail |
|---|---|---|---|---|---|---|
| **Sunon HA30101V4-1000U-A99 (5 V)** ✅ | 30×30×10 axial | 5 V | 3.5 CFM | 15.1 dBA | 0.3 W | €4.63 |
| Delta BFB0305HA-C | 30×30×10 blower | 5 V | 1.45 CFM, 0.285 inH₂O | 29 dBA | 0.65 W | €7.69–7.77 |

Leading = **Sunon axial (quiet, low power)** pushing air across a 5–7 mm finned heatsink on the SoM, ducted to rear/side vents. Delta blower fallback if static pressure needed. Verify CFM/noise/PWM in Phase 6/8.

## 8. Magnetic pogo UART connector (REQ-UART-01)

- AliExpress magnetic 2.54 mm-pitch pogo sets (4–6 pin, vendor-stated 2 A): **€3.77/set** (`1005005122336187`, 375 sold). ⚠ Low confidence — seller-stated only; no datasheet; retention/plating/spring travel **must be verified** or replaced with branded spring pins (Mill-Max 0964 + magnets) in Phase 6.
- Electrical front-end (level-shift 3.3/5 V + ESD + polyfuse, signal+GND default) is design work → `hardware/uart.md` in Phase 6.

## 9. USB-C PD sink + charge (REQ-IO-06 / REQ-PWR-04)

| Part | Role | Spec | Price/avail |
|---|---|---|---|
| **STUSB4500** ✅ | standalone PD3.0 **sink**, 3 fixed PDOs (to 20 V), POWER_OK, no MCU | dead-battery, auto-run | **€0.99** (Arrow/Farnell) |
| onsemi FUSB302 | Type-C SNK/SRC/DRP, **PPS** (needs host fw) | for max-efficiency later | €0.49 |
| CH224K | cheap PD3 sink/trigger, resistor-set | alternative | ~€0.5–1 (TBD price) |

- 20 V/65 W → battery via **4S buck-boost charger IC** (e.g., TI **BQ25713 / BQ25703A**, 1–4S) — part-level design Phase 6. Full 65 W needs an **E-marked 5 A C-C cable** (STUSB4500 fixed-PDO → likely 20 V/3.25 A only if E-marked).
- **Leading = STUSB4500 + BQ25713-class charger** (provisional).

## 10. Trackball sensor (REQ-KB-01)

- **ADNS-9800** laser sensor, SPI, up to 8000 cpi, **€1.25** (AliExpress `1005006588379741`).
- ⚠ **Ball-size caveat:** ADNS-style sensors are designed around 34–38 mm pool-ball curvature; a **15–21 mm ball is atypical and may give geometric error** — verify with a test mount before locking the mechanical design (open item, Phase 6). User wants trackball *below space, between G/H/B* (DEC-025) → small recessed ball in aluminum deck is the plan, but sensor suitability for small ball is unresolved.