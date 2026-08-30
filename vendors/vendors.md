# Vendors (Phase 4)

Status: 2026-08-30. EU-first sourcing; marketplace flagged. All prices snapshots.

| Vendor | Region | Role | Categories | Confidence | Notes |
|---|---|---|---|---|---|
| **NKON.nl** | NL | Retail (cells) | 21700 cells (P45B/50E/P50B) | High (storefront, in-stock flags) | Good cells pricing/tiers; VAT shown incl. |
| **akkuteile.de** | DE | Retail (cells) | 21700 cells | High | DE VAT incl. |
| **Mouser / TME / Farnell / DigiKey / Arrow / RS / Distrelec** | EU/US | Authorized distribution | SoM, PD sink, fan, charger IC, bridge chips, connectors | High (primary datasheets) | EU EUR pages sometimes bot-block — requote |
| **Octopart/TrustedParts** | — | Price/stock aggregator | Cross-vendor pricing today | High for pulls | Used for fan/PD-sink/SoM stock |
| **Toradex** (docs.toradex.com) | CH/DE | SoM vendor | Verdin i.MX 8M Plus, design docs, power data | High (primary) | Carrier design guide + power-consumption pages |
| **Quectel** | CN/global | Modem vendor | EC25-EUX/EG25-G | High (datasheets) | RED pre-qualified modules |
| **SIMCom** | CN | Modem vendor | SIM7600E-H | High | |
| **Realtek (OEM modules)** | CN | Wi-Fi modules | RTL8821CU/8822CS modules | Med | via marketplace sellers |
| **AliExpress DE** | CN→DE | Marketplace | Pogo magnetic connectors, Wi-Fi modules, ADNS-9800, generic 25-30mm fans, HDMI→DSI boards, 21700-affiliates | **Low–Med** (seller-stated) | Confirm drawings/ratings before design-in; prices highly variable |
| **Panelook / Youritech / lcddisplay.co / display-lcd.com** | CN/global | Display trading/OEM | Panels + driver boards (HE080IA-01E, YD691MIPI-V1, HOTHMI) | Med–High (datasheets verified) | Quote-only; B2B |
| **Hardkernel** | KR | SBC/panel DIY | Vu8S DSI kit | High (official store, $39) | Brightness/power unstated |
| **Waveshare** | CN | Panel kits | 8" DSI LCD | High (official) | $74.99 |
| **Local CNC (cnc mill + anodizing shop)** | DE | Fabrication | Aluminum chassis machining + black matte anodizing | High (user-provided) | CNC labor free — material + coating only |

## Regional & import notes
- EU distributor preferred for IT imports; VAT + small duty (~0–5%) assumptions to be confirmed in Phase 5 landed-cost model.
- Marketplace (AliExpress) prices exclude VAT/shipping; add ~19% VAT + freight in landed model.
- Long-lead / quote-only items: panels (quote), Verdin (stock-limited), LTE (Unikey stock 2). Recommend engineering-validation samples before volume purchases (Phase 9).