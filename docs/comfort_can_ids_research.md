# VW T6/T6.1 Comfort‑CAN: Research Notes (community sources)

- Scope: Gather public hints for VW Transporter (T5/T6/T6.1) Comfort‑CAN message IDs and signals to inform bench/vehicle validation. Treat all items below as candidates until verified on your vehicle.
- Comfort‑CAN speed: 100 kbit/s (multiple VW forum sources for T5/T6 family).

Sources consulted (public, non‑paywalled)
- TX‑Board (T5/T6 forum): “CAN Bus im Volkswagen” — explains Comfort‑CAN (100 kbit/s), typical modules (cluster, Climatronic, window lifters, roof unit, heater), VW TP1.6 on T5, and gateway details. URL: http://tx-board.de/threads/can-bus-im-volkswagen.20100/
- TX‑Board: “CAN‑IDs” — community thread with specific IDs discussed; examples include 0x371 (door bits) and 0x571 (battery voltage scaling example), plus TP base channels. URL: http://tx-board.de/threads/can-ids.50640/
- TX‑Board: “T5.1 CAN‑Antrieb – ID 0x420” — shows 0x420 used for outside and oil temps on drivetrain CAN (not Comfort). Useful to avoid false positives. URL: http://tx-board.de/threads/t5-1-can-antrieb-id-0x420.158829/
- T6Forum: “CAN bus or LIN bus” — confirms door/window/lock status is on CAN (convenience/comfort) while door modules may speak LIN to BCM; CAN sleeps after inactivity. URL: http://www.t6forum.com/threads/can-bus-or-lin-bus.47218/

Candidate Comfort‑CAN IDs (to validate)
- 0x371 — Door/lock bits (body domain)
  - Context: Reported as “four doors bit‑coded” on VAG platforms in TX‑Board CAN‑IDs thread. Expect individual bits for sliding door(s), rear hatch, lock state. Exact bit map may differ by platform/year.
  - Action: Log frames while toggling each door and central lock; diff per‑bit changes at 100 kbit/s.

- 0x571 — Battery voltage (example scaling)
  - Context: TX‑Board post describes byte‑level pattern where a byte value (e.g. 0x19 → 25 dec) scaled as 25/2 = 12.5 V. Treat as main/comfort domain battery supply indication (not BMS SOC).
  - Action: Record raw bytes at rest and with charger/load; confirm “raw/2 V” scaling, endian, and byte index on T6/T6.1.

- 0x3xx range — Body/lighting/interior
  - Context: Common VAG convention places body/lighting around 0x300–0x3FF. Threads and codebases often mention interior/ambient lighting near 0x3B0/0x3B2 on other VW platforms. Not yet found T6‑specific public confirmation.
  - Action: With interior lights/ambient lighting controls, observe 0x3xx activity, especially 0x3B0/0x3B2; correlate brightness/color bytes.

- 0x47x/0x6C0 — Park distance control (PDC) candidates
  - Context: PDC messages commonly seen around these IDs in VAG communities (model dependent). We did not find an open T6 thread with explicit mapping. Treat as hypothesis only.
  - Action: Engage reverse/parking assist; capture deltas and check 8‑bit distances by sensor position.

- TP channels (diagnostic/transport protocol context)
  - Context from TX‑Board snippets: 0x600 base for TP on some platforms; derived functional IDs like 0x0699/0x06B9 in examples. Not used for live sensor values but useful when filtering out diag chatter.

What we did not find publicly (T6‑specific)
- Open, T6/T6.1‑specific posts with exact Comfort‑CAN byte/bit maps for: pop‑top roof, water levels, awning, ambient lighting. These topics appear in forums but concrete IDs/bit maps are often kept to VIP sections or private logs.

Recommended validation plan (bench/vehicle)
- Bit rate: Switch to 100 kbit/s when tapping Comfort‑CAN. The current `esphome/t6_can_scanner.yaml` defaults to 500 kbit/s (drivetrain‑like). Adjust before logging Comfort‑CAN.
- Baseline/diff: With `esphome logs`, capture a quiet baseline, then toggle one function at a time: each door/hatch, interior lights, ambient color, locks, PDC, heater, windows.
- Focused filters: Start broad (0x300–0x3FF) then hone to stable candidates (e.g., 0x371, 0x3B0/0x3B2). Save raw frames with timestamps for later diffing.
- Scaling heuristics: Try typical VAG scales: temperature `(raw * 0.5) - 40`, percent `raw * 100 / 255`, voltage `raw / 2` or `raw / 10`, distances `raw * 10` mm.
- Sleep behavior: Expect Comfort‑CAN to sleep after inactivity. Unlocking or a button press should wake it; plan captures accordingly.

Next steps for this repo
- If you want, we can add a “comfort_can_100kbps” variant or substitution in `t6_can_scanner.yaml` to quickly swap bit rates and add a raw‑frame logger focused on 0x300–0x3FF.
- Once you validate new IDs/bytes, we’ll add precise entries to `decoder_rules/vw_t6_comfort.yaml` with byte/bit offsets, scaling, units, and descriptions, keeping TX off by default.

Notes
- Many VAG IDs are shared across platforms but differ in details (length, endian, byte index). Treat any cross‑model reference as a lead, not truth.
- Several T6/T6.1 posts with deeper mappings appear to be behind VIP/member areas; links above are public summaries we could access.

