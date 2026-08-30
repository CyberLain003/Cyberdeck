# Keyboard and Trackball (Phase 6)

Status: **Phase-6 design reference — draft** — 2026-08-30.
References: REQ-KB-01, DEC-021 (custom membrane), DEC-025 (trackball between G-H-B), RISK-010 (depth/rows), RISK-014 (fit/cost), RISK-020 (small-ball sensor caveat), E-026 (ADNS-9800).
Related Phase-6 docs: `hardware/pcb.md` (§8 deck/ports, §13 USB2 hub), `electrical/block-diagram.md` (EDGE-11).
Strict evidence rule: geometry/pricing quotes are targets until quoted; `TBD` = unverified.

| Symbol | Convention |
|---|---|
| E-xxx | evidence refs in `info/sources.md` |
| TBD | to verify |
| 🔲 | open item |

---

## 1. Requirements recap

- **6-row** membrane, **US legends + ISO/German-style Enter (hybrid)** — REQ-KB-01 / DEC-021.
- Min key **~8 mm**, pitch **~12 mm netbook-class** (row step 9.5–10 mm, key width 11–12 mm) — RISK-010/014.
- **Integrated trackball** mounted **below the space bar, between G-H-B**, small **15–21 mm ball** — DEC-025, RISK-020.
- Aluminum top-deck cutouts (A-016/A-022).
- **140 mm base depth budget** (envelope 200×140×50, DEC-043): keyboard + trackball + palm rest must fit. **Keyboard top edge at 10% depth, bottom edge 40% from the bottom; palm rest = front 40%; SoM + 5–7 mm heatsink + 30 mm fan in the RIGHT palm-rest zone** (DEC-053/061).

---

## 2. Keyboard architecture — layout map

### 2.1 6-row matrix (14 columns × 6 rows)

Standard 14-column scan matrix; **ISO-DE Enter** is the 2U-tall L-key at the far right, spanned over rows R3/R4 at column C13. US legends for the alphas; `#~` key sits left of Enter (German-ISO habit), `\|` key at R2/C13 (US placement). Function row optional half-row (see 2.3).

```
          C0    C1    C2    C3    C4    C5    C6    C7    C8    C9    C10   C11   C12    C13
 R0   Esc   F1    F2    F3    F4    F5    F6    F7    F8    F9    F10   F11   F12    PrtSc    ← full-height F-row (½-height variant saves ~5 mm)
 R1   `~    1!    2@    3#    4$    5%    6^    7&    8*    9(    0)    -_    =+     Bksp(2U)
 R2   Tab   Q     W     E     R     T     Y     U     I     O     P     [{    ]}     \|
 R3   Caps  A     S     D     F     G     H     J     K     L     ;:    ':    #~     [ISO-E ┐]
 R4   LShf  Z     X     C     V     B     N     M     ,<    .>    /?    RShf  —       [ISO-E ┘]
 R5   LCtl  LWin  LAlt  [Space, 5U>]              (over C4–C8)         RAlt  RWin    Menu
```

- **Node usage:** R0=14 (Esc + F1–F12 + PrtSc), R1=14, R2=14, R3=13 (C13 is the top half of ISO Enter — one key, single node), R4=14 (incl. ISO Enter node at C13), R5=7 (Space = single 5U-wide node). Total ≈ **76 keys with the F-row**, **62 without it**.
- **ISO-DE Enter:** physically 1.25U wide × 2U tall, occupying R3C13+R4C13 as **one** scan node (assigned R4C13; R3C13 left unswitched/ghosted). German-ISO muscle-memory: `#~` remains left of it; all other legends US.
- **Wide keys:** Bksp 2U, Tab 1.5U, Caps 1.75U, LShift 2.25U, RShift 2.75U, Space 5U. Each wide key = one node regardless of width.

### 2.2 Pitch / geometry (netbook class)

| Dimension | Target | Note |
|---|---|---|
| Row step (vertical) | **9.5–10.0 mm** | 6 rows ≈ 57–60 mm stack |
| Key width (1U) | **11–12 mm** | min key body ≥ 8 mm (REQ-KB-01) |
| Keyboard block width | ≈ 12 × 12 mm ≈ **144–156 mm** | fits inside 200 mm base (interior 194, excl. chassis walls) |
| Keyboard block depth | 6 × 10 ≈ **60 mm** | + function-row half: ~64 mm |
| Trackball zone | below space bar, between G-H-B | ≈ 14 mm deep well (see §4) |
| Palm rest | **front 40% ≈ 56 mm** | DEC-061; SoM + thermal tower in the **right** palm-rest zone (DEC-053) |
| **Keyboard placement band** | **10% → 40%-from-bottom ≈ 70 mm** | DEC-061: keyboard block (60–64 mm) fits the band with margin |

### 2.3 Function row decision

- Default: **full-height F-row** (Esc + F1–F12 + PrtSc, 14 keys) if depth permits (RISK-010; total ≈64 mm keyboard block). A **½-height F-row variant** saves ~5 mm depth when the palm rest or hinge budget is tight.
- 🔲 **Open:** include or drop; user preference (OQ-005). Keep R0 in the mask either way (matrix supports it); final keying decided at mechanical sign-off.

---

## 3. Matrix scan — interface to the SoM

Two options (constraint allows both). **Option A recommended** (zero host-driver work, mainline-clean, works in U-Boot).

### Option A — Deck USB-HID controller (recommended)

```
Membrane 6×14 (diodes) ──► MCU @ deck     ADNS-9800 ──(SPI)──► MCU
                                 MCU ──(USB 2.0 device)──► SoM root USB2 (internal header)
```
- Small MCU — **STM32G0-class recommended (DEC-045)** — lowest-power USB-HID that works (M0+, USB FS); RP2040 / ATmega32U4 are acceptable alts. Scans the matrix (1 kHz, debounce in FW) + reads the trackball sensor (SPI, via HID mouse endpoint) → **composite HID (keyboard + mouse)**.
- Host sees a normal `usbhid` keyboard+mouse — **no kernel driver, no `linux-firmware` dependency**, works in U-Boot/bootloader (useful for recovery).
- 🔲 Firmware: small custom HID report descriptor; open source in-repo (Phase 7). No proprietary blobs.
- **Cost:** MCU board ~€4 + 1 USB-A/C internal header (part of carrier BOM).

### Option B — I2C GPIO expanders (MCP23017-class)

```
Membrane 6×14 ──► 2× MCP23017 (I2C, addr 0x20/0x21) ──(I2C 400 kHz)──► SoM I2C bus
```
- 20 GPIO lines needed (6 rows + 14 cols) → 2× MCP23017 (32 lines; spare pins for LED / F-key / matrix interrupt / etc.). 
- Scanning in kernel/userspace via mainline `gpiochip` + a `matrix-keypad`-style platform glue (small DT/board file work). Report rate limited by I2C (≈ 2× bytes/row × 6 rows @ 400 kHz ≈ sub-ms; fine for typing).
- **Cons:** needs a kernel-side matrix driver override (Ph7 work); keyboard dead until kernel up (no boot-time input). Trackball follows its own SPI path (see §4) — split host interfaces.
- **Cost:** 2× MCP23017 ≈ €1–2 total; software cost higher.

> **Recommendation:** Option A. It keeps all matrix+trackball complexity out of the Linux stack, satisfies REQ-COMP-04 (mainline all the way), and unblocks U-Boot keyboard nav. Option B remains a fallback if a deck MCU is undesirable.

### Backlight (optional)

- Per-key/zone LED underlay on the membrane, driven from the deck MCU (Option A) or spare expander pins (Option B) via a small constant-current driver; PWM dimming.
- 🔲 Optional — no requirement; cost impact ≈ €5–15 + firmware; decide at budget review. Default: none.

---

## 4. Trackball — ADNS-9800 + small 15–21 mm ball

### 4.1 Sensor set + geometry

| Part | Role | Notes |
|---|---|---|
| **ADNS-9800** | SPI navigation sensor (laser-class, up to 8000 cpi) | E-026, €1.25 |
| Lens (per sensor datasheet, e.g., ADNS lens) | fixed elevation optics | required working distance ~ datasheet |
| Mini ball **15–21 mm Ø** | rolling surface | polished steel or high-contrast matte ball; retention via socket |
| IR/Laser illumination | integrated in ADNS-9800 (laser VCSEL) | no external LED needed normally (check variant) |
| Low-drop voltage for VCSEL | LDO if 5 V rail drops | datasheet |

Wiring (SPI from deck MCU in Option A, or from SoM SPI in Option B):

```
ADNS-9800     ── SCK  ──►  host SPI-SCK
              ── CS_N ──►  SPI-CS (dedicated)
              ── SDIO ──►  SPI MISO/MOSI  (per datasheet variant: 4-wire or 3-wire SDIO; 🔲 confirm)
              ── NRESET ─►  GPIO reset
              ── VCC/GND
```

### 4.2 Placement (DEC-025)

- **Well below the space bar, between G-H-B** (row 4, under the G/H/B cluster — the palm-rest side of the space bar).
- Ball well: pocket Ø ≈ (ball Ø + 0.5–1 mm) in the aluminum top deck; ball center ≈ 0.5–0.7×Ø below deck surface so the top 1–2 mm of the sphere protrudes for finger/thumb access. Sensor + lens mounting hole at the bottom of the well, optical axis toward the ball surface at datasheet elevation.
- Depth budget: recessed well ≈ 12–15 mm under the deck — fits the 140 mm depth stack (RISK-010), clears the carrier below.

### 4.3 Small-ball caveat (RISK-020) — MUST TEST

- ADNS-class sensors are designed for **34–38 mm pool-ball curvature**. A **15–21 mm ball** has ~2× smaller surface radius → the imaging window spans a strongly curved, sloped surface → **focus blur, geometric scale error, and a dead zone** at the lens edge.
- **Action (gate before mechanical lock):** build a bench rig — sensor + lens + candidate ball (18–21 mm first) on a test jig; verify trackability, CPI linearity, and click-consistency over ±30° rotation. If >~10 % error or unusable dead-zone → fallbacks: (a) 22–25 mm ball in a slightly larger well, (b) alternate sensor geometry, (c) trackpoint-node option as last resort.
- 🔲 **Test deliverable:** rotation error %, CPI map, usable band; feed decision.

### 4.4 Firmware / driver note

- **No mainline kernel driver** for ADNS-9800 exists today the repo knows of. In Option A (deck MCU) the sensor is handled in MCU firmware → HID mouse — **no host driver**. In Option B the SoM would need SPI + userspace daemon (spidev → uinput) or a small kernel input driver → more Phase-7 work.
- Recommend Option A specifically for this reason (single firmware blobs remain in-repo open source).

---

## 5. Aluminum top-deck cutouts (handoff to `mechanical/`)

| Cutout | Spec (target) |
|---|---|
| Keyboard opening | ≈ 156 × 64 mm (+F-row: ~160 × 68) wide/tall opening; 12 mm pitch block |
| Key well depth | membrane + 3–4 mm standoff under deck; ~8 mm keycap travel available |
| Trackball well | Ø 15–22 mm pocket below G-H-B; sensor/lens aperture at bottom |
| Palm rest | **front 40% flat zone (≈56 mm)**; SoM + thermal tower under the **right** half (DEC-053/061) |
| RF windows | remain per RISK-013 (antennas) — do not cover with key section |

---

## 6. Cost estimate

| Item | € | Source |
|---|---|---|
| Membrane PCB + overlay + dome (custom, DE/CN quote) | **30–60** | B-015 |
| Deck MCU (**STM32G0-class**, DEC-045; RP2040 alt) — Option A | **3–6** | BOM estimate |
| ADNS-9800 sensor | ~1.25 | B-016 |
| Lens kit | ~2 | `TBD` |
| Ball 15–21 mm + socket | ~5–10 | `TBD` |
| 2× MCP23017 (Option B only) | ~1–2 | BOM estimate |
| Backlight (optional) | 5–15 | `TBD` (default none) |
| **Keyboard+trackball total** | **≈ 40–90** | — |

---

## 7. Open items

1. 🔲 **Function-row decision** (R0): full/½/dropped (OQ-005).
2. 🔲 **Small-ball test** (4.3) — ball Ø + sensor acceptance; gate for §4.2 well geometry.
3. 🔲 Matrix scan route A vs B final (recommend A; needs user-visible cost point).
4. 🔲 Membrane supplier quote + legend artwork (US + `#~` ISO-Enter hybrid).
5. 🔲 ADNS-9800 exact serial-pin variant (3/4-wire SDIO) + lens part.
6. 🔲 Backlight on/off decision (default none).