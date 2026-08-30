# Terminology

| Term | Meaning in this project |
|---|---|
| B2B connector | Board-to-board connector joining the SOM to the carrier PCB (chosen for routable pitch; BGA not user-routable) |
| SOM / SoM | System-on-Module (compute + RAM) hosted on the carrier via B2B |
| Carrier | User's 4–6 layer PCB that hosts the SOM and all I/O |
| 21700 | Cylindrical Li-ion cell, Φ21 × 65–70 mm, ~5000 mAh-class used here |
| 4S1P / 2S2P | Series/parallel cell arrangement (4 series × 1 parallel = 14.4 V; 2S2P = 7.4 V) |
| Hot-swap / power bridge | Both packs live on a common rail via OR-ing; inserting/removing a pack does not interrupt power (cf. ThinkPad Power Bridge, T480) |
| PD (USB-C) | USB Power Delivery, 65 W sink capability on the charge input |
| OR-ing | Diode/FET arrangement letting either supply power while blocking back-feed between packs |
| pogo pin | Spring-loaded magnetic connector for the UART access port |
| PCB antenna | Printed / ceramic antenna for Wi-Fi/BT; no external antenna elements |
| Watthour (Wh) | Energy; pack = sum of cell Volts×Ah; total ≥ 120 Wh |
| Workload | Defined 30 h usage mix (20 term + 4 browse + 6 locked) used to validate runtime |
| Landed cost | Delivered price incl. shipping, VAT, duty/customs, brokerage |
| Effective throughput | Real USB 3 throughput after protocol/hub overhead (≥1 Gbit/s REQ-IO-02) |
| REQ-xxxxx | Stable requirement ID (see requirements/baseline.md) |
| Strictive unknown | Value required by the strict evidence rule; marked TBD until primary evidence exists |