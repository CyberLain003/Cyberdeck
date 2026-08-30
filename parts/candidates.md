# Parts — Candidates (Phase 4)

Status: **Phase-4/5 sourced research, reconciled to Phase-6 decisions** — 2026-08-30. Every row has source URL + access date. **Candidate = has evidence; not yet "Recommended"** (strict rule: Recommended requires dimensions + interfaces + Linux/NixOS + price/availability + envelope/power/thermal/budget impact — full verdict lands after budget model + acceptance planning). Prices are **snapshots**, stale within weeks.

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
| **SATA** | **NOT available on any pin** → **M.2 2280 NVMe direct on PCIe Gen3 ×1** (DEC-033; no bridge, see §3) |
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

## 3. Storage (M.2 2280 NVMe — REQ-COMP-03, user-approved change from SATA)

- **Verdin has no SATA ⇒ M.2 2280 NVMe on PCIe Gen3 ×1**, direct (no bridge chip — DEC-027 superseded by DEC-033).
- **Keying/silkscreen blocking (user requirement):** socket **M-key only**, silkscreen stating "NVMe M.2 2280 ONLY", physical keying prevents a B/B+M SATA drive from being inserted. Only the correct drive fits.
- PCIe Gen3 ×1 ≈ 985 MB/s theoretical → low/mid-range NVMe is ideal; prefer DRAM-less low-power models.
- Exact NVMe model + price: **TBD** (Phase-5 BOM / Phase-7 freeze). Many 2280s available (~€40–90). Budget row in §ref.

## 4. Display (chosen: Raystar RFU800G-AYH-MNN — brightest 1125 nit, DEC-044)

### Selected: Raystar RFU800G-AYH-MNN
- 800×1280 (5:8 portrait, 16:10 landscape), 189 ppi, **MIPI-DSI 4-lane** (native to Verdin), **1125 cd/m²**.
- OD **115.74 × 184.93 × 4.75 mm** (module incl. backlight), active 107.64×172.22 mm.
- Backlight: ~4.32 W max @1125 nit; **~2.2 W @ ~50% PWM (≈560 nit)** — fits the ~2.5 W display budget at working brightness.
- Price quote-only (~$35–60, stale, E-014). Panel OD drives the new envelope (DEC-043).

### Upgrade/alternatives (if price/ratio matter more)
| Panel | Res/ppi | Ratio | I/F | Brightness | Notes |
|---|---|---|---|---|---|
| **HE080IA-01E** | 1024×768 / 160 | 4:3 | DSI 4-lane | bare cell — needs backlight | true 4:3; BL power TBD |
| HOTHMI HTM-H080D14-LVDS-A01R | 1024×768 / 160 | 4:3 | LVDS | 1000 nit | needs LVDS driver |
| Hardkernel Vu8S | 800×1280 / 189 | 5:8 | DSI kit | ~300–400 nit (TBD) | cheapest ($39), brightness unstated |

## 5. LTE modem — tiny-LGA Cat-1 (cheapest, private use — DEC-026/058)

**Leading form factor (DEC-058):** small **LGA radio module (~ESP32-WROOM footprint, ≤~30 mm)**, direct-solder on the carrier — replaces the mPCIe EC25-EUX as the leading pick. nano-SIM on the carrier.

| Part | Class | Bands (EU, target) | I/F | Size | Linux | Price/avail | Conf |
|---|---|---|---|---|---|---|---|
| **Quectel EC200U-EU** (leading candidate) | Cat-1 | **B1/3/7/8/20** (B28 TBD) | USB2 HS (QMI/ECM → ModemManager) | LGA ≈ 29×32 (TBD) | mainline (cdc-ecm/qmi) | **TBD re-quote (DEC-058)** | Med |
| SIMCom SIM7000-class | Cat-M1/NB-IoT (verify Cat-1 alt) | B1/3/8/20 (verify) | USB2/UART | LGA ≈ 31×31 (TBD) | generic `option`/cdc | TBD | Med |
| Quectel BG95-class | Cat-M1/NB-IoT | B1/3/8/20 (verify) | USB2/UART | LGA ≈ 28×31 (TBD) | generic | TBD | Med |

> EU bands B1/3/7/8/20 + nano-SIM required (REQ-RF-02); **exact part, bands, power, and landed price are pending re-research (DEC-058)** → Phase-5 BOM. SIM7000/BG95 are Cat-M1/NB-IoT class — verify a Cat-1 alternative (EC200U) if speed/Cat-1 is required.

**Alternative (historical — superseded as leading by DEC-058):**

| Part | Bands (EU) | I/F | Size | Linux | Price/avail | Conf |
|---|---|---|---|---|---|---|
| **Quectel EC25-EUX** (alt) | B1/3/7/8/20/28A | USB2.0 HS (QMI → ModemManager) | mPCIe 51×30×4.9 | **qmi_wwan mainline** | **€31.51** (mPCIe, Avnet/Unikey) | High |
| SIMCom SIM7600E-H | B1/3/5/7/8/20 (no 28) | USB+UART+PCM (option/PPP) | LGA 30×30×2.9 | generic `option` | **€32.76** (TME etc.) | High |
| Quectel EG25-G | global, incl EU | QMI | LGA/mPCIe | QMI | €55–64 (disty); **EOL/harvest pressure — recheck** | Med |

> EC25-EUX stays the **alternative** if the tiny-LGA re-research (DEC-058) cannot find a Cat-1 part meeting B1/3/7/8/20 at target cost: cheapest historical pick, most complete EU bands incl B28, best Linux story. Module RED pre-qualified; final device RED integration responsibility stays (Phase 8).

## 6. Wi-Fi + BT (REQ-RF-01) — provided by the SoM (DEC-046)

- Verdin i.MX 8M Plus **"WB" variant** = Wireless + Bluetooth **on-module** (user-confirmed "there is a wifi module on the som"). No separate RTL8821CU module needed.
- Carrier routes **antenna coax (U.FL) from the SoM to an RF window** in the aluminum top deck/rear.
- Throughput: on-module Wi-Fi (802.11ac-class) ≫ 6 MB/s target; mainline drivers (brcmfmac/rtl8xxx/mwifiex depending on module).
- Verify exact RF module in the WB SoM variant + driver at Phase-4/7 check (TBD).

## 7. Micro fan (thermal — DEC-017)

| Part | Size | V | Airflow | Noise | Power | Price/avail |
|---|---|---|---|---|---|---|
| **Delta BFB0305HA-C (blower)** ✅ | 30×30×10 blower | 5 V | 1.45 CFM, **0.285 inH₂O** | 29 dBA | 0.65 W | €7.69–7.77 |
| Sunon HA30101V4-1000U-A99 | 30×30×10 axial | 5 V | 3.5 CFM | 15.1 dBA | 0.3 W | €4.63 |

**Primary = Delta BFB0305HA-C blower** (DEC-062 ducted path; `hardware/thermal.md` §2): left-back intake → fan (in-plane left of the heatsink) → heatsink fins L→R → right-side exhaust. **Sunon axial is the fallback** for a low-restriction arrangement (open fin field, where 3.5 CFM/15.1 dBA matter). ⚠ Delta -C is 2-wire, **no tach**; low-side PWM / voltage control compatibility **TBD (RISK-027)**. Verify CFM/noise/PWM in Phase 6/8.

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

- **ADNS-9800** laser sensor, SPI, up to 8000 cpi, **€1.25** (AliExpress `1005006588379741`). Read by the deck USB-HID MCU (DEC-045).
- ⚠ **Ball-size caveat:** ADNS-style sensors are designed around 34–38 mm pool-ball curvature; a **15–21 mm ball is atypical and may give geometric error** — bench-test before locking the deck cutout (RISK-020, DEC-050).

## 11. Deck keyboard MCU (USB-HID, DEC-045)

- **Recommended: STM32G0-class** (Cortex-M0+, USB Full-Speed device, very low power) e.g. **STM32G030/G031/G041**; scans membrane matrix + trackball (SPI), exposes USB-HID keyboard/mouse. ~€2–4, hand-solderable (TSSOP/QFN).
- Alt: **RP2040** (easy, common, ~€1–2 core module) but higher idle power — acceptable fallback.
- Runs TinyUSB/LUFA (Phase 7 firmware). Zero host driver + works in U-Boot.

## 12. USB 2.0 hub (DEC-049)

- **Low-power USB2513-class** (3-port USB2 hub) on carrier for: deck MCU (HID), LTE modem (QMI), spare. USB3 dedicated port stays **unshared** (REQ-IO-02). Mouser/DigiKey, ~€3–6. Mainline driver built-in (standard hub).