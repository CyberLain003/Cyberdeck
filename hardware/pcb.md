# Phase-6 Carrier PCB Design Document

Status: **Draft for review — 2026-08-30.** Produces the board-level requirements for REQ-COMP-06 (custom 4–6L carrier, user-routed, no BGA) and the Phase-6 PCB gate. Every Toradex-sourced value cites the source URL + access date. Anything not verified is `TBD`.

Applies fixed constraints (approved, do not change): SoM = **Toradex Verdin i.MX 8M Plus 8 GB** via **0.5 mm 260-pin DDR4-SODIMM socket**; **all SoM I/O is 1.8 V** unless noted (SDIO is switchable 1.8/3.3 V, Ethernet MDI analog, USB/HDMI/DSI/PCIe are protocol PHY). Carrier is user-routed on **4 or 6 layers**, no user-BGA, high-speed stays on carrier; a **rear daughterboard carries power + low-speed + battery contacts**.

---

## 1 Sources (all accessed 2026-08-30)

| Ref | Document | URL | Used for |
|---|---|---|---|
| [S1] | Verdin Carrier Board Design Guide (Verdin CBDG) | https://docs.toradex.com/108140-verdin-carrier-board-design-guide.pdf | Socket/stacking-heights §4.2, land pattern §4.5, standoffs §4.3/§4.5, power §3.1, interface schematics §2.2–2.9 |
| [S2] | Verdin Family Specification Rev 1.5 (2026-04-30) | https://docs.toradex.com/109262-verdin-family-specification.pdf | Module dims §4.2, Direct Breakout §3.5, connector/stacking §4.3, interface grouping |
| [S3] | Toradex Layout Design Guide | https://docs.toradex.com/102492-layout-design-guide.pdf | Stack-ups §3, impedance §4, length matching §6.7, return path §6.8, per-interface rules §7 |
| [S4] | Verdin i.MX 8M Plus Datasheet Rev 1.8 | https://docs.toradex.com/116795 (login-gated; claims per E-009 as quoted in `info/sources.md`) | Pin table (secondary/verification): PCIe/DSI/HDMI/USB/SD assignment |
| [S5] | M.2 specification summary (Wikipedia, citing PCI-SIG M.2 / TE NGFF) | https://en.wikipedia.org/wiki/M.2 | M.2 keying, 75-position/67-pin edge, 0.5 mm pitch, 2280 = 22 mm × 80 mm |
| [S6] | Octopart → TE.com listing for 2309409-2 | https://octopart.com/search?q=2309409-2 | B2B socket price/stock (€1.655 @1, stock 13,376) |
| [S7] | `bom/bom.md`, `parts/candidates.md`, `decisions/decision-log.md`, `architecture/interfaces.md` | local repo | Approved part BOM (B-xxx), power split, board split |

> Note: the Verdin CBDG's *layout* guidance points at [S3] for "trace impedance and length matching" (§1.1). §7 of [S3] is written against Apalis/Colibri modules but the tabulated rules are interface-type-based (PCIe Gen3, USB 3.0, HDMI, DSI D-PHY, GbE, SDIO) and are applied unchanged to Verdin. Values under "user target" in §2/§3 combine industry norms (PCI-SIG/MIPI/HDMI) with the approved constraint wording; Toradex's own targets are given alongside.

---

## 2 PCB Layer Stack Recommendation (4 vs 6 layers)

Constraint: user hand-routes, wants controlled-impedance differentials on 4 or 6 layers, no user-BGA, and high-speed confined to the carrier. Both stacks below are Toradex-endorsed 1.6 mm examples ([S3] §3) adapted so **both outer high-speed layers reference a solid GND plane** (Toradex's printed 6-layer example places PWR on L2; we recommend swapping PWR/GND so L2 and L5 are GND).

### 2.1 Recommended stack (default: 6 layers, 1.6 mm, 1 oz outer / 0.5 oz inner)

| Layer | Function | Notes |
|---|---|---|
| L1 | **HS microstrip** — USB3, PCIe/REFCLK, HDMI, DSI, GbE MDI | ref GND L2 |
| L2 | **GND plane** (solid) | return path for L1 |
| L3 | **PWR planes** — 3.3V / 5V / 12V splits + VCC | splits stitched |
| L4 | **Low-speed** signal (I2C, UART, PWM, GPIO, SD data if kept short) | no controlled impedance needed |
| L5 | **GND plane** (solid) | return path for L6 |
| L6 | **HS microstrip** — balance of diff pairs, SD/MMC, RGMII | ref GND L5 |

Inner prepreg ~112 µm (1.6 mm total) per [S3] Fig.2. L2/L5 both solid GND avoids the "power-plane reference + stitching caps" trap on L1/L6 entirely ([S3] §3.2/§6.8).

### 2.2 Fallback (4 layers) — only if 6-layer cost forces a trade

| Layer | Function | Notes |
|---|---|---|
| L1 | HS microstrip (primary) | ref GND L2 |
| L2 | GND plane | solid |
| L3 | PWR plane (+ GND fills) | splits at rail boundaries, stitching caps |
| L4 | HS microstrip / short low-speed | ref L3 (uses L3 as reference → stitch caps required, [S3] §3.1) |

On 4L, **all critical diff pairs must route on L1** (GND-referenced); L4 is used only for non-critical low-speed, or pairs whose return can accept a stitched PWR reference. This roughly halves usable routing area and raises via discipline importance.

### 2.3 4 vs 6 layer trade-off table

| Aspect | 4-layer | 6-layer | Impact / decision |
|---|---|---|---|
| HS signal layers | 1 (L1) solid-GND referenced only | 2 (L1, L6) | 6L roughly doubles HS routing channels → critical for a 15+ pair board |
| Reference planes | L3 is PWR → stitching caps on every L4 HS run ([S3] §6.8) | L2/L5 GND → minimal stitching | 4L risk: forgotten stitch caps = EMI/SI issues |
| Plane quality | PWR+GND share split L3 | dedicated PWR L3, GND L2/L5 | 6L cleaner for 3.3V/5V/12V splits under M.2 + SoM |
| Routing of SDIO/RGMII + 8+ diff-pair groups | congested | comfortable | user-routable without agonising bottlenecks (RISK-007 mitigation) |
| Cost (PCBWay walk, 100×130 mm) | ~€8–18 | ~€18–35 | TBD quote (B-005); 6L overhead acceptable vs re-spin risk |
| Fab lead time | − | +1–2 days | minor |
| **Recommendation** | — | **Default 6L** | 4L is an acceptable fallback only if nearly all HS stays on L1 |

---

## 3 Controlled-Impedance Targets (explicit)

Formulas used ([S3] §4, §6.4, §6.7):

- Differential microstrip (approximate; real value via Polar / PCBWay stack-up tool): target Z_diff with a **W : S ratio from the fab stack**, always prioritizing **differential over single-ended** impedance ([S3] §4).
- Propagation: `v ≈ c / √εr ≈ 150 µm/ps` on FR-4 ([S3] §6.7). Length-to-time: `ΔL[µm] ≈ 150 × Δt[ps]`.
- Stub limit: `l_max(stub) ≪ λ/10 = v/(10·f_max)`; PCIe Gen3 example: ≈ **3.5 mm** ([S3] §6.4 → do not exceed stubs >3.5 mm on either PCIe or USB3.0).

Consolidated targets (fabrication impedance tolerance ±10%, all ± on microstrip outer layers):

| Interface | User/industry target | Toradex target [S3] §7 | Carrier max trace | Intra-pair skew | Inter-pair / clk skew | Notes |
|---|---|---|---|---|---|---|
| **USB 3.1 Gen1 (5 G)** – USB_2 SS RX/TX | 90 Ω diff, 50 Ω SE | 90 Ω±15% diff, 50 Ω±15% SE (Table 9) | <200 mm | <1 ps ≈ 150 µm | TX↔RX <1.6 ns ≈ 240 mm | AC-coupling 100 nF (module-side TX; device-side RX for connector). ≤2 vias TX / ≤2 vias RX (conn.) [S3] §7.4.1.2 |
| **PCIe Gen3 ×1 → M.2** | 85 Ω diff, 50 Ω SE | 90 Ω±15% diff, 50 Ω±15% SE (Table 5) | device-down <300 mm | <1 ps ≈ 150 µm | clk↔data <1.6 ns ≈ 240 mm | PCI-SIG nominal 85 Ω; 90 Ω within ±15% tolerance → target **87 Ω centre**, OK both. No AC caps on carrier (M.2 card has them). ≤2 vias TX / ≤4 RX device-down [S3] §7.1 |
| **HDMI 2.0 TMDS** | 100 Ω diff (HDMI spec), 50 Ω SE | 90 Ω±15% diff (Table 12) | <250 mm | <5 ps ≈ 750 µm | clk↔data <150 ps ≈ 22 mm | Toradex routes 90 Ω; ±15% covers 100 Ω. Minimise vias; DDC/HPD level-shift 1.8 V↔5 V; CEC 3.3 V shift [S1] §2.5 |
| **MIPI-DSI 4-lane** | 100 Ω diff (D-PHY), 85–100 Ω accepted | 90 Ω±15% diff, 50 Ω SE (Table 17) | <200 mm (to panel-FPC) | <1 ps ≈ 150 µm | clk↔lanes <10 ps ≈ **1.5 mm** | Clock is NOT embedded → tightest skew budget. ≤ few vias. 0.5 mm FFC transition counts toward the 200 mm [S3] §7.12 |
| **RGMII (spare MAC, ETH_2)** | 50 Ω SE | (RGMII not tabled in [S3]; Verdin defines 1.8 V RGMII [S1] §2.3.2) | <100 mm, short | n/a (SE) | group matching: TXC↔TXD[3:0] and RXC↔RXD[3:0] within **≤1–2 mm**; clock DDR @125 MHz | 50 Ω SE; place PHY close; exact i.MX8MP RGMII budget TBD |
| **GbE MDI (on-module PHY KSZ9131, ETH_1)** | 100 Ω diff + xfmr | 95 Ω±15% diff, 55 Ω SE (Table 7) | module→magnet <~100 mm; magnet→jack PWB minimal | <1.6 ps ≈ 250 µm | pair↔pair <330 ps ≈ 50 mm; pair↔pair spacing >450 µm; **>7.5 mm from HS signals, >2.5 mm from low-speed** | RJ45 w/ integrated magnetics preferred; no center tap needed (voltage mode) [S1] §2.3.1 |
| **SD/MMC/SDIO (4-bit)** | 50 Ω SE | 50 Ω±15% SE (Table 15) | <100 mm | — | clk↔data <20 ps ≈ 3 mm at SDR104 (SD/MMC clk ≤ 50 MHz HS → <15 mm) | no pull-ups on carrier (module has them); card powered 3.3 V [S1] §2.8 |
| **USB2 host & OTG** | 90 Ω diff | 90 Ω±15% diff, 50 Ω SE (Table 8) | <200 mm | <7.5 ps ≈ 1.1 mm | — | No AC caps; no polarity swap [S1] §2.4 |
| **I2C_1/_2/_3** | 50 Ω SE nominal | Table 16 | <200 mm (Fast) | — | — | pull-ups on carrier 1.8→10 kΩ per load [S1] §2.9 |

> Build-time note: target the *single-ended* column with the fab's Polar/impedance report as final authority, not this sheet.
Trace W : S for the two stacks (Toradex guidance, [S3] §4 Table 2/3): 

| Stack | 50 Ω SE W1 | 90 Ω diff W1: S1 | 95 Ω diff (ETH) W1: S1 | 100 Ω diff (DSI/HDMI fallback) W1 : S1 |
|---|---|---|---|----|
| 4L (112 µm prepreg) | 180 µm | 180 : 190 µm | 150 : 165 µm | 150 : 200 µm |
| 6L (112 µm prepreg) | 180 µm | 180 : 200 µm | 150 : 165 µm | 150 : 200 µm |

**Setting your fab:** spec `controlled impedance — 1 oz outer, 112 µm prepreg L1–L2 / L5–L6, tol ±10%`; ask the fab to re-verify W : S for their material and issue an impedance coupon/verification (TBD at quote stage).

---

## 4 B2B Socket: TE 2309409-2, Land Pattern and Keep-Outs

### 4.1 Connector selection
- Module = 260-pin DDR4-SODIMM edge (0.5 mm pitch, JEDEC edge geometry) [S1] §4.2, [S2] §4.3.
- **Recommended socket: TE Connectivity 2309409-2** – connector height 5.2 mm, board↔module 2.62 mm; specifically recommended for cooling-capable products [S1] §4.2. Price €1.655 @1, stock 13,376 (TE.com via Octopart [S6], 2026-08-30). Alternative: Amphenol-sourced DDR4-SODIMM sockets (4 mm to 9.2 mm heights [S1] §4.2). **Amphenol alt exact PN for this stack TBD; rule: 5.2 mm height, verify land pattern with the maker.**
- Do **not** use a 4 mm socket (1.52 mm b2b is too small) or reverse-angle sockets (unsuitable with cooling) [S1] §4.2.

### 4.2 Stacking-height / component constraint table [S1] §4.2
| Connector height | b2b distance | Max component height (under module) | Max component (next to connector) | Verdict |
|---|---|---|---|---|
| 4.0 mm | 1.52 mm | — | — | Not suitable |
| **5.2 mm** | 2.62 mm | 0 mm | 0.8 mm | **Recommended** |
| 8.0 mm | 5.42 mm | 2.8 mm | 3.6 mm | alternative |
| 9.2 mm | 6.62 mm | 4.0 mm | 4.8 mm | alternative |

### 4.3 Land pattern (TE 2309409-2 optimised) [S1] §4.5 (Fig.114)
- Footprint envelope **31.0 × 68.8 mm** (module S1–S4 boss ring to ring).
- Standoffs: 
  - **S1/S2** (long edge away from socket): 2.5 mm tall M2×0.4, boss Ø5.30 mm, located at the far long edge (≈4.60 mm / 3.50 mm insets to module edge; Ø3.00 copper pad optional), for module fixation + heatsink mount.
  - **S3/S4** (at the SODIMM clip ends): Ø1.60/Ø1.10 mm pads; standoffs need **7.0 mm** height for the Verdin Industrial Heatsink [S1] §4.4.1.
  - Either no standoffs, only S1/S2, or all four are acceptable [S1] §4.5.
- **No-component zone**: under entire module (only ≤0.8 mm parts within ~18 mm band next to the connector). Clip is not rigid → do not squeeze any components under the module [S1] §4.2.
- The socket pins (pitch 0.5 mm, 260 pins) break out on two 0.5-mm rows at the long edge; leaves/shadow of the module = the "Direct Breakout" inter-space groups (DSI, HDMI, USB, PCIe, SD, Ether) fan out **without crossing** adjacent groups [S2] Fig.3-4. Keep each group's pairs localised at the pin row exit.

### 4.4 Keep-out / void rules at the socket area
| Zone | Rule |
|---|---|
| Under-module (full 69.6×35 mm) | No components above 0.8 mm; blank; no vias unless needed |
| Next to connector (module short side) | ≤0.8 mm decoupling/series parts allowed in ~18 mm band [S1] Fig.112 |
| S1–S4 standoff bosses | keep ≤Ø5.30 mm clearance around; add mounting holes per land pattern |
| Heatsink zone | module-centered 5–7 mm tall space, duct zone (see §9) |
| L2/L5 ground voiding | keep solid under the socket tail pins; route diff-pair breakouts tight (≤15 mm) |

---

## 5 High-Speed Routing Strategy per Interface

General discipline [S3] §6 (all to be re-verified at build):
- 135° bends only, never 90°; serpentine segments ≥1.5×W apart, adjacent serpentines ≥4×W [S3] §6.2.
- Stub lengths ≤ 3.5 mm on PCIe/USB3 lanes (λ/10 @ 4 GHz) [S3] §6.4.
- Diff pairs: both traces on the same layer; nothing between pair traces (no components/vias); 0402 coupling caps placed symmetrically; same via count on both legs; symmetric breakout; match per-segment (connector/magnetics/AC-coupling-cap/via delimit pairs); compensation close to the mismatch origin, max 15 mm async skew [S3] §6.6/§6.7.
- Route same-interface pairs on the same layer [S3] §6.7.

| Interface | Path | Strategy |
|---|---|---|
| **USB3 host (dedicated)** | USB_2_SSTX 169/171; SSRX 175/177; D± 181/183; EN 185; OC 187 [S1] §2.4.1 → rear Type-A (or C) | Route SS pairs L1/L6, GND-referenced; keep USB2 D± together with the SS pairs toward the same connector; no AC caps on carrier for the connector case (module TX-cap on module, device RX-cap in cable/plug) [S1] §2.4.3.2.3; ESD TVS on all lines; 5 V power path via switch (TPS2066-class), ≥150 µF output [S1] §2.4.3.2.3; OC# back to pin 187 |
| **USB2 OTG-able** | USB_1 D 165/163, ID pin, VBUS sense, EN/OC | Very short run to Type-C. D± both sides of C connector tie together (no mux needed for USB2.0-only) [S1] §2.4.2; TUSB321-class CC→ID logic if C-DRD role needed (C-receptacle or Micro-AB recovery; TBD by connector choice) |
| **PCIe Gen3 ×1 → M.2 M-key 2280** | PCIE_1_CLK ± 228/226; RX 234/232; TX 240/238; RESET 244; WAKE 252 [S1] §2.2.1 | REFCLK pair keeps ≤150 µm intra-pair skew; 100 MHz ref clk, one load only (M.2 card); reset level-shift 1.8→3.3 V (M.2 PERST# level) per [S1] §2.2.2.1; NO hot-plug supported (ignore M.2 presence detect); 3.3 V only supply (no 12 V needed on M.2 — good for battery) |
| **HDMI out** | TXC 69/67; TXD0 75/73; TXD1 81/79; TXD2 87/85; CEC 63; HPD 61; DDC I2C_3 57/59 [S1] §2.5.1 | All 4 pairs from the B2B onto L1 near-parallel, clk↔data ≤22 mm skew; level-shift DDC/HPD 1.8→5 V domain (PCA9306-class), CEC 3.3 V; ESD small Cap-TVS; HDMI 5 V 50 mA rail gated via switch [S1] §2.5.2 |
| **DSI 4-lane → Raystar RFU800G via small FPC adapter** | CLK 37/35; D0 49/47; D1 43/41; D2 31/29; D3 25/23; I2C_2_DSI 53/55; PWM_3 (backlight) 19; GPIO 17/21 [S1] §2.6.1 | 4 data + clk pairs → 0.5 mm FFC / mini FPC land at edge; keep total (carrier + flex) ≤200 mm; clk↔lane ≤1.5 mm; backlight PWM + BL-enable + DSI I2C 1.8 V (level-shift if panel I/O is 3.3 V) [S1] §2.6; **exact Raystar RFU800G FPC connector pitch/pin-count TBD (RISK-024) — source the panel first, then lock the adapter** |
| **GbE MDI (on-module PHY, 1st)** | MDI0 225/227, MDI1 233/231, MDI2 239/241, MDI3 245/247; LED1 235, LED2 237 | conn→magnetics ≤100 mm in 95 Ω pairs; RJ45 w/ integrated magnetic preferred; no center-tap/transformer power (voltage-mode PHY) [S1] §2.3.1; if discrete mag, isolated mag-GND island ≥2 mm [S3] §7.3 |
| **RGMII spare MAC (ETH_2)** | RXC 197, RX_CTL 199, RXD3..0 207/205/203/201, TXC 213, TX_CTL 211, TXD3..0 215/217/219/221, MDC 193, MDIO 191, INT 189 [S1] §2.3.2.1 | 1.8 V I/O. 50 Ω SE, MDIO pull-up 1.8 V; PHY strapping must not collide with the on-module PHY address (default address 00 111) → choose a different strap; **KSZ9131RNX (1.8 V capable) preferred [S1] §2.3.2.2**; group match TXC↔TXD and RXC↔RXD ≤1–2 mm; connector-side magnetics same as §5. Not populated in v1 (spare MAC documented) |
| **SD/MMC/SDIO (full-size)** | D0..3 80/82/70/72, CMD 74, CLK 78, CD 84, PWR_EN 76 [S1] §2.8.1 | 50 Ω SE, clk↔data ≤~15 mm (HS) and design for SDR104 (≤3 mm); no pull-ups (on module); VCC3.3 only via MIC94073 switch (SD_1_PWR_EN 1.8/3.3 V tolerant); ESD optional [S1] §2.8.2 |
| **On-module Wi-Fi/BT (SoM "WB")** + **USB2 LTE (tiny-LGA)** | Wi-Fi/BT = antenna coax (u.FL) from SoM; LTE = USB2 D± on a connector-adjacent pad area [S1] §2.2.2.2 (USB pattern) | Wi-Fi/BT needs **no discrete module and no USB data lane** — only U.FL coax to the lid (DEC-046). LTE is USB2 — route 90 Ω diff, short; **LTE + deck-HID share one USB2513-class 2.0 hub upstream of SoM USB_2 (DEC-049, see §13)**; nano-SIM holder on the carrier near the tiny-LGA radio; LGA module direct-soldered (~ESP32-WROOM footprint, ≤~30 mm, DEC-058), no mPCIe slot |

---

## 6 Power Distribution on the Carrier

### 6.1 Rail split points
Carrier receives **12 V / 5 V / 3.3 V + GND** from the daughterboard power tree (fixed constraint, I-18 / power split). Inside the carrier:

| Net | From | Primary loads | Copper/entry strategy |
|---|---|---|---|
| **VCC (module 5 V feed)** | 5 V rail, always-on (STR–Module OFF keeps VCC on [S1] §3.1) | SoM VCC pins 251/253/255/257/259 (3.135–5.5 V; SoM ≤8.25 W sustained / 12.5 W peak form-factor [S1] §3.1; this SKU ~1.5–6.3 W) | star from entry to module pins; ≥1 via/A rule |
| 3.3 V | daughterboard | M.2 NVMe (≤3 A), codec, LTE tiny-LGA (3.3 V), SD VCC3.3, level-shift VCCB, RJ-45 LED pull | separate plane L3 split, ≥4 A |
| 5 V | daughterboard | USB VBUS (900 mA×2 via switches), fan (5 V), HDMI 5 V(50 mA) | plane/wide trace ≥2 A |
| 12 V | daughterboard | **reserve** – currently no mandatory load (M.2 needs only 3.3 V; no full PCIe slot) | keep light; C-plate ~100 µF |
| 1.8 V (PWR_1V8_MOCI pin 214) | **module output**, 250 mA [S1] §3.1.1 | codec/control logic, level-shift VCCA, DSI I2C | short to the parts; not derived from carrier rail |
| GND | daughterboard, star-point | all | full L2/L5 planes; single-point audio/analog-GND merge near codec [S3] §6.9 |

### 6.2 Input filtering and decoupling
- At each rail entry (FFC/power header): **ferrite bead (≥3 A, 120 Ω@100 MHz class) + 10 µF + 100 nF** per rail ([S1] reference schem convention, VRM-side bypass).
- Bulk: **≥100 µF** on 5 V entry (USB VBUS rule [S1] §2.4.3.2.3), **≥47–100 µF** on 3.3 V entry (M.2).
- Per-IC decoupling: 100 nF + 10 nF at every supply pin; array near each interface block (M.2, PHY-magnetics, codec, modem, Wi-Fi).
- SoM VCC pins: 1 × 100 nF + closest 10 µF within the allowed 0.8 mm "next to connector" band ([S1] §4.2/§4.5); series net is star, not daisy.
- Avoid backfeeding: UART/I2C/HDMI CEC must not power an un-powered domain; use open-drain + level-shifters with OE gated by CTRL_SLEEP_MOCI# for high-side parts ([S1] §3.5.5).
- Power-sequencing: CTRL_PWR_EN_MOCI stays high in sleep for always-on carrier parts; CTRL_SLEEP_MOCI# controls sleep-able blocks (LTE/M.2/fan can sleep; on-module Wi-Fi/BT sleeps with the SoM) ([S1] §3.1.2, Table 41 + [S3] §6.8 stitching).

---

## 7 Placement Block Layout (carrier top view; left = front of base, right = rear connector bay)

```
  ←— front of base (battery bays / keyboards edge) — (no connectors)
  ┌───────────────────────────────────────────────────────────────┐
  │ L3 PWR plane stitch caps along rail necks        │ intake ◄───│
  │                                          [FAN]  │ (L-back)   │
  │  [SoM on-module Wi-Fi/BT]            ╔══════════════════╗ →R  │
  │   u.FL coax → lid-top               ║  B2B + Verdin   ║ exh  │
  │                       (right palm  ║  (centered)      ║ DUCT │
  │                      rest zone)    ║  + HS 5–7 mm     ║ fins │
  │                                    ╚══════════════════╝ →R side
  │   [LTE tiny-LGA (EC200U-class) + nano-SIM]   [M.2 M-key 2280]│
  │      (direct-solder; SIM near radio)           (edge, service)│
  │  [CODEC I2S + line-out]   [RGMII spare PHY KSZ9131 – NC v1]  │
  │  [USB2 hub USB2513 (DEC-049)]                                │
  └────────────────────────────────────────────────────────────────┘
          Carrier rear edge (this face is the connector bay):
   [HDMI out] [USB3 A] [USB2 OTG C] [RJ45 w/ mag] [3.5 AUX] [SD full]
          (bay continued on daughterboard below/behind: [USBC PD] [pogo]…)
```

Rules:
- **SoM centered** so its heatsink sits over the CPU with the fan duct unobstructed; fan axis along the fins of the heatsink; SoM edges get S1/S2 (2.5 mm) + S3/S4 (7.0 mm) stand-offs — heatsink (B-014) picks up those bosses [S1] §4.4.
- **M.2 near an edge** (opens service access, not under heatsink duct); 3.3 V bulk next to it; keep-out for double-sided cards; screw post at 2280 hole (and provide 2242/2260 scoring posts as optional).
- **LTE tiny-LGA + SIM**: direct-solder LGA radio (~ESP32-WROOM footprint, ≤~30 mm, DEC-058) near the nano-SIM holder (short UIM/USB traces); no mPCIe slot. USB2-only; keep ≥25 mm from SoM/radios.
- **Wi-Fi/BT (on-module, SoM "WB")**: radio on the SoM; antenna coax (u.FL) up the hinge to the **RF-transparent plastic lid-top** behind the screen (DEC-046/055) — no discrete module on the carrier, no RF aperture in the aluminum deck.
- **Codec** near the 3.5 mm AUX jack, analog-GND island, output stage out of the fan duct.
- **Ports (HDMI/USB/RJ45/AUX/SD)** at the carrier rear edge; **connectors with big pads get GND-fill keep-out under pads** ([S3] §6.5) and edge copper ≥0.5 mm clearance.
- **Fan** (**Delta BFB0305HA-C blower**, primary) on 5 V in the duct — inlet from the left-back intake, blowing **left→right through the heatsink fins** to the right-side exhaust (DEC-062); control = low-side PWM/voltage, **TBD (RISK-027; Delta -C is 2-wire, no tach)**; Sunon HA30101V4 axial retained as fallback.

---

## 8 Connector Locations (rear conn bay) + Front-Edge Battery Notes

Carrier rear edge (main, out the chassis rear):

| # | Connector | Type | Signal(s) | Notes |
|---|---|---|---|---|
| E1 | HDMI 2.0 out | standard HDMI Type-A | HDMI_1 pairs + 5 V/DDC/HPD/CEC | rear, screen/HDMI host reach |
| E2 | USB 3.2 G1 host (≥1 Gbit/s effective) | Type-A (or C-DP with mux) | USB_2 SS+HS, 900 mA switch, OC# | **dedicated/unshared SS lane** — REQ-IO-02 |
| E3 | USB 2.0 OTG | Type-C (or Micro-A B) | USB_1 + ID/VBUS | REQ-IO-03; C needs CC→ID logic (TUSB321-class) |
| E4 | GbE | RJ45 with integrated magnetics | ETH_1 MDI (on-module KSZ9131) | LED1/2 to jack LEDs; 1000Base-T |
| E5 | AUX 3.5 mm | stereo TRS jack | I2S codec line-out | no speaker amp (REQ-KB-02) |
| E6 | SD slot | full-size push-pull | SD_1 bus + PWR_EN/cd + 3.3 V switch/LDO | REQ-IO-05 |
| E7 | LTE antennas (2× U.FL) + Wi-Fi/BT coax (1× u.FL) | U.FL | up the hinge to plastic lid-top (RF-transparent, DEC-055) | RISK-013 |

Daughterboard (behind/under the carrier rear edge, its OWN rear face):

| DB-E1 | USB-C PD sink 65 W (power only) | USB-C receptacle, E-marked cable note | STUSB4500 + BQ25713 | REQ-IO-06 / PWR-04 |
| DB-E2 | Magnetic pogo-UART (switchable 3.3/5 V) | pogo set | level-shift+ESD+polyfuse front-end here | REQ-UART-01/02 (see `hw/uart.md`) |
| DB-E3 | Power/LED/buttons | tact + LEDs | power button, battery LEDs, charger status |
| DB-E4 | Battery contact blocks A & B | sprung contacts | front-edge of daughterboard receives packs |

**Front-edge battery contact notes (daughterboard, NOT carrier):** packs slide in from the base front; their contacts land at the **front edge of the daughterboard** (the front battery bays meet the daughterboard at its front edge). Requirements inherited from REQ-PWR-01/03/04/05 + DEC-004/012:
- OR-ing (ideal-diode FETs) with per-pack voltage/current sense + pre-charge on insertion (RISK-004 mitigation).
- Contact block sized 4-pin per pack (B+/B−/T/NTC class), sprung contacts; **contact style, pin pitch and current rating TBD** (target ≥10 A pulse, ≤5 A steady per pack).
- Daughterboard keeps the pack-rail path short (<30 mm), copper ≥ 4 A/mm cross-section → plane spread or bus-bar; exact width per Phase-6 electrical review.
- Safety-critical gate: this block is **not** carrier design; `battery-hot-swap.md` + professional design review required before fab (REQ-PWR-05).

---

## 9 Thermal Construct (heatsink + fan path)

- Heatsink (B-014): **5–7 mm finned (al) block** on top of SoM (module is 6.0 mm tall; heatsink base on BGA/SoC area), kept under the ~35 mm module area; mounted to stand-offs S1/S2/S3/S4 (2.5 / 7.0 mm heights [S1] §4.4) — a sink that screws onto those bosses solves retention + keeps parallel.
- Fan (B-013): **primary = Delta BFB0305HA-C blower** (30×30×10, 5 V, 0.65 W, 29 dBA, 1.45 CFM, 0.285 inH₂O); **Sunon HA30101V4 axial (0.30 W, 15.1 dBA) is the fallback** (`hardware/thermal.md` §2). Ducted air path per DEC-062: **left-back intake louvres → fan (in-plane, left of the heatsink) → heatsink fins L→R → right-side exhaust**. 5 V control from the power-MCU, profiled by the SoC temperature sensor read over I2C; low-side PWM/voltage TBD (RISK-027 — Delta -C is 2-wire, no tach).
- Air gap path must not short-circuit (fan outlet close to the heatsink fin mouths; intake louvre plenum upstream); keep M.2/antenna coax out of the direct duct.
- z-budget: heatsink 7 mm + module 6 mm (socket 5.2 + 2.62 b2b = module top ≈8 mm over carrier) + 0.5 clearance ≈ **~16 mm** from carrier → fits 50 mm stack (base 30 mm electronics plane + deck) with margin [S1] §4.1/4.2.
- SoM peak form-factor 8.25 W sustained / 12.5 W peak [S1] §3.1 is the thermal design point; our 8 GB SKU measured ~1.5–6.3 W (E-009), so 5–7 mm + the Delta blower's 0.285 inH₂O static pressure is margin but validate on bench (RISK-016).

---

## 10 DFM Notes (user hand-routed; PCBWay-friendly; B-005)

| Item | Rule |
|---|---|
| Board outline | 100 × 150 mm nominal (fits 160×~120 base zone); route-cuts 45°; ≥0.5 mm copper-to-outline edge spacing; add 4× M2/M3 mounting + stand-off holes (S1–S4 bosses on SoM land pattern) |
| Traces | min 0.15 mm (6 mil) signal, 0.2 mm (8 mil) power; min spacing 0.15 mm — wider on diff pairs per §3 |
| Vias (PTH) | ≥0.3 mm drill / ≥0.6 mm pad, annular ≥0.15 mm; **no blind/buried**; ≤2–4 vias per HS pair ([S3] §7); teardrops on every via+pad connection (esp. power/module) |
| Teardrops | enable globally; fan out to pads 45° |
| Edge clearances / rounded traces | copper-to-board-edge ≥0.5 mm; keep HS 1–2 mm off plane edges; stitch GND periphery every 5 mm |
| Solder mask | weed-free pads (no mask slivers); mask clear 0.1 mm; avoid solder mask between tight differential pairs |
| Silk | **“NVMe M.2 2280 ONLY”** on the M.2 (DEC-033/REQ-COMP-03) + polarity/lane-1 markers, “2×USB”, “DC in”, etc. |
| Impedance | only outer-layer microstrip; fab impedance coupon + report; tolerance ±10% |
| Assembly | SMD pads default; split planes for 6L; no via-in-pad (place vias beside pads) |
| Panel | 1 + 5 coupon copy array or panel with rails; edge plated half-holes none |
| File set | Gerber X2 (RS-274-X), drill Excellon, ODB++ / KiCad or Altium export — PCBWay-friendly |

---

## 11 Carrier BOM (new lines: C-xxx; referenced B-xxx from `bom/bom.md`)

Prices are snapshots (stale within weeks). Access date 2026-08-30 for all evidence rows; `TBD` = not yet sourced.

| ID | Part | Package | Unit | Qty | EUR | Vendor / avail. | Access | Notes |
|---|---|---|--:--:|---|--:--:|---|---|---|
| B2B socket (SoM) | TE 2309409-2 (recommended) | SMD (DDR4-SODIMM, 5.2 mm) | 1 | €1.655 @1 / €1.472 @100 | TE.com via Octopart [S6] | 2026-08-30 | stock 13,376; Amphenol alt TBD |
| M.2 | M-key 2280 socket (NGFF) | SMD, 0.5 mm, M-key (75-position) | 1 | TBD | TBD | — | standard M.2, M-key only; supplier-confirm key-block geometry (see §13 open) |
| LTE radio (tiny-LGA) | Quectel EC200U-EU-class LGA (~ESP32-WROOM footprint, ≤~30 mm) | SMD | 1 | TBD | TBD | — | Cat-1; re-research bands/power/price (DEC-058) |
| nano-SIM holder | nano-SIM 6-pad push-push | SMD | 1 | TBD | TBD | — | UIM/USB to tiny-LGA radio |
| RJ45 w/ mag | 1000Base-T jack w/ LED | THT | 1 | TBD | TBD | — | integrated magnetics (voltage-mode PHY) [S1] §2.3.1 |
| RGMII spare PHY | KSZ9131RNX (or KSZ9031) | QFN-48 | NC in v1 | TBD | TBD | — | spare MAC documented; pop for 2nd port |
| Codec (line-out) | TI TLV320AIC3104-class, mainline drivers | TQFN/SOIC | 1 | ~€2–3 (TBD) | TBD | — | I2S + I2C control, line-out only |
| USB3 Type-A host | USB 3.2 Gen 1 Type-A (or Type-C + SS mux) | THT | 1 | TBD | TBD | — | ESD + 900 mA power switch |
| USB2 OTG Type-C | Type-C receptacle (or Micro-AB) | SMD/THT | 1 | TBD | TBD | — | TUSB321-class if C |
| SD slot | full-size push-pull | THT | 1 | TBD | TBD | — | + MIC94073 PWR switch |
| Level-shifters | PCA9306 / FXMA2102 1.8↔3.3/5 V | SO-8 / SOT-6 | ~4 | TBD | TBD | — | HDMI DDC/HPD, CEC, PCIe-RESET 1.8→3.3, I2C |
| ESD | USB3-capable TVS (e.g., TPD2EUSB30) + HDMI RCLAMP + LAN | various | ~18 | TBD | TBD | — | on every exposed port |
| Power switches | TI TPS2066-class (USB), MIC94073 (SD) | SO-8 | ~3 | TBD | TBD | — | OC# to SoM pins |
| Bulk/decoupling | 0402/0805: 100 nF, 10 µF; 1210/3520 100 µF 10 V | 0402–3520 | ~180 | ~€3–6 (TBD) | TBD | — | per rail + SI-method |
| Ferrite | 120 Ω@100 MHz, 3 A | 0603 | ~5 | TBD | TBD | — | rail entry |
| DSI FPC adapter | 0.5 mm FFC → Raystar RFU800G | custom flex | 1 | TBD | TBD | — | RISK-024 open; lock after Raystar panel |
| USB2 hub (if needed) | SMSC/MC USB2513-class | QFN48 | 1 (opt) | ~€1.5–3 | TBD | — | only if module-specific USB2 hosts run short (§10) |
| Carrier PCB | 6L user design | 1.6 mm, controlled-Z | 1 | ~€20–35 (CN walk, 6L) | PCBWay | TBD quote | B-005 + B-006 assembly backed in |
| Heatsink/fan/SoM/etc | see B-001/B-013/B-014/B-011/B-012 | — | — | — | — | — | unchanged from `bom/bom.md` |

> Keep the BOM strictly gerber-aligned: every C-row gets a final PN + footprint + rev in `bom/bom.md` before the fab run (B-006).

---

## 12 Board Split: Carrier ↔ Daughterboard (FPC-crossing signal/rail list)

Board gap per DEC-018/RISK-015: high-speed **never crosses**; only low-speed control + power.

### 12.1 Crosses from carrier → daughterboard

| Group | Signals | Level | Notes |
|---|---|---|---|
| Power MCU link | I2C_1 (SCL/SDA pin 14/12) | 1.8 V, pull-ups carrier-side | SoM→MCU (REQ-PWR-03/04) |
| Pogo UART control | UART_1 or UART_2 (RX/TX 1.8 V) + EN + level-mode | 1.8 V | front-end (level-shift/ESD/polyfuse) lives on daughterboard near pogo (REQ-UART-01) — see `hw/uart.md` |
| Fan control | low-side PWM / voltage; tach N/A (2-wire Delta -C) | 5 V | to fan drive (DB; RISK-027, DEC-062) |
| Power sequencing monitor | CTRL_FORCE_OFF, CTRL_SLEEP_MOCI#, CTRL_PWR_EN | 1.8 V | DB can honor sleep-able rail gating [S1] §3.1.2 |
| Buttons/LEDs | PWR_BTN_MICO#, CTRL_RESET_MICO# (in), user LEDs + battery LED | 1.8 V OD | power state to MCU |
| Wake | CTRL_WAKE1_MICO# | 1.8 V | MCU-derived wake into SoM [S1] Table 41 |
| Status | BAT_present, charge-status | 3.3 V | power-tree status → SoM via GPIO or I2C |

### 12.2 Crosses from daughterboard → carrier

| Group | Rails/signals | Rating | Notes |
|---|---|---|---|
| Power | **12 V, 5 V, 3.3 V (separate), GND** | 5 V ≥4 A, 3.3 V ≥4 A, 12 V ≤2 A | DC-DC converters (power tree) on DB; carrier just sinks. High-current rails mated with ≥8 parallel pins each at ≥0.5 A/pin, or a 2.54 mm power-header rated 5 A/pin (exact part TBD). |
| Ground | GND × N (return) | return | star return, single-point merge |
| Control (option) | MCU→carrier peripherals (Wi-Fi/PHY reset-enable) | 3.3 V | sleep-able rail gating |

### 12.3 PHYSICALLY NOT crossing (must never leave carrier)
HDMI, USB3 SS, USB2 on USB_1/USB_2, PCIe/ref-clk to M.2, SD/MMC, DSI+backlight-PWM, GbE MDI, RGMII (if used), audio digital (I2S can cross as it is low-speed but keep on carrier for simplicity).

### 12.4 Interconnect hardware
- **FPC for signals**: 0.5 mm FFC, ≥26–30 pins, 1.8 V / 3.3 V signals only (I2C/UART/PWM/GPIO/Wake/Reset).
- **FPC for power**: `TBD` — either a second 40-pos 0.5 mm FFC with 8×3.3 V + 8×5 V + 12×GND pins (≈0.5 A/pin each → derated), **or** a single 20-pos 2.54 mm pin-header (5 A/pin). Recommend a separate **power harness (2.54 mm or Mini-Fit 2-row)** so signal FFC and power can be routed independently in the base. **Rated part + wire gauge TBD at the Phase-6 electrical review (REQ-PWR-05 safety gate).**
- Keep the board gap short (<50 mm); both boards share a chassis common ground; match the FFC ground-pin count to the §3 star-return rule.

---

## 13 Open design items (carried to Phase-6 electrical/mechanical reviews)

| Item | Impact | Recommended resolution |
|---|---|---|
| **M.2 hardware-block vs B+M SATA** — M-key-only socket blocks B-key-only cards, but **B+M-key SATA 2280 cards carry both notches and physically fit an M-socket** (their design intent, [S5]); keying alone does not exclude them | REQ-COMP-03 wanted "only NVMe physically fits" | Keep M-key socket + silkscreen **"NVMe M.2 2280 ONLY"** (per DEC-033) as the mechanical marker; add a **chassis/PCB boss interlock** (final-geometry study) OR accept functional lock-out (no SATA on the PCIe×1 lane) and document. **Decision required with user.** |
| USB2 hosts for LTE (tiny-LGA) + deck-HID | 2 USB2 devices; only USB_1(OTG)/USB_2(host) on standard pins | One **USB2513-class USB2 hub** (DEC-049) upstreams both into SoM USB_2 host (SS lane stays unshared → ≥1 Gbit/s effective still met); verify **Verdin i.MX8MP module-specific extra USB2** (`TBD` in S4 datasheet §module-specific) as an alternative |
| Raystar RFU800G FPC connector pitch/pin-count | RISK-024 | Source the Raystar panel first, then lock the 0.5 mm FFC adapter + its keep-out at the carrier edge |
| M.2 socket exact PN + key-geometry data | blocking detail | Ask TE/JAE/Amphenol for the M-key 2280 socket land pattern + mechanical spec; keep the M-only variant |
| Exact receptacle choices (Type-C vs A; OTG C vs Micro-AB) | mux/ID-logic count | fixed at schematic review (Type-C receptacles need TUSB321-class CC logic for USB2; TUSB546/1042-class SS mux for USB3 orientation) |
| Codec final (line-out) + drivers | REQ-KB-02 | pick a mainline (kernel ≥ 4.19) part, e.g., TI TLV320AIC3104-class; bench output levels |
| RGMII length budget per i.MX8MP | spare MAC | get the official i.MX8MP hardware design checklist value; KSZ9131 fits [S1] §2.3.2.2 |
| Power interconnect rated part + gauge | heat/voltage | Phase-6 electrical review (REQ-PWR-05 gate, safety-critical) |
| 4-layer fallback go/no-go | cost | only when near-full HS routing fits L1; else stay 6L |

---

## 14 Summary

- **6-layer default** stack with GND L2/L5, HS on L1/L6 (4L fallback limited to L1-only HS). Controlled impedance: USB3 90 Ω, PCIe/HDMI target ~87–90 Ω; the ±15% tolerance covers 85/100 Ω, DSI ~90–100 Ω (fine), GbE 95 Ω diff, 50 Ω SE everywhere else — with explicit W : S from the fab.
- **B2B**: TE 2309409-2 (5.2 mm, €1.655 @1, stock) + S1–S4 stand-offs; never populate under the module (except ≤0.8 mm band).
- High-speed: short, GND-referenced, per-interface skew budgets; all on carrier only; rear connector bay carries HDMI/USB3/USB2-OTG/RJ45/AUX/SD; daughterboard keeps PD, pogo-UART front-end, power rails + battery contacts at its front edge.
- Power: 12/5/3.3 V entered + star-gated; VCC from always-on 5 V; SoM pins 251–259; per-rail ferrite + bulk caps; no user-BGA anywhere — every carrier part is a THT/SMD footprint the user can place.
- Thermal: 5–7 mm finned sink on SoM bosses in the right palm-rest zone + **Delta blower primary (Sunon axial fallback)** ducted left→right (left-back intake → right-side exhaust, DEC-062), within the 50 mm stack.
- DFM: teardrops, ≥6-mil rules, PTH vias ≥0.3/0.6, ≥0.5 mm edge clearance, silkscreen “NVMe M.2 2280 ONLY”.
- Board split freezes RISK-015: only I2C/UART/PWM/GPIO/buttons/wake + the three power rails + GND cross the gap.

Ready for the Phase-6 electrical/mechanical review gate. Open items in §13 (esp. the M.2 blocking reality-check and USB2 host count) need a user decision before schematic lock.