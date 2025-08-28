

claus# Product Requirements Document (PRD) – VW T6.1 CAN Scanner

## 1. Product Overview
The VW T6.1 CAN Scanner is an **ESP32‑based hardware and firmware solution** designed to monitor, decode, and (optionally) control signals on the **Comfort‑CAN bus** of the Volkswagen T6.1 California (2024). It integrates seamlessly with **Home Assistant**, providing users with real‑time visibility of vehicle systems (lighting, fridge, auxiliary power) and optional safe control functions (e.g., LED brightness).

---

## 2. Goals & Objectives
- Enable **safe CAN bus monitoring** from a non‑intrusive interface.
- Provide a **local Web UI** for scanning, filtering, and logging CAN messages.
- Support **export formats** (CSV, JSONL, PCAP) for external analysis.
- Auto‑discover and expose decoded CAN signals as **entities in Home Assistant**.
- Support optional **control functions** (guarded with a TX Arm safety model).
- Keep the solution **low‑cost, easy to install**, and extensible with community‑shared CAN ID maps.

---

## 3. Target Users & Use Cases
- **Camper Owners**: View and control lighting, fridge status, and auxiliary functions from Home Assistant.
- **DIY Enthusiasts & Hackers**: Reverse engineer CAN messages using Web UI and logging tools.
- **Fleet Managers / Service**: Monitor vehicle state and logs for diagnostics.
- **Developers**: Extend via ESPHome rules and community‑shared CAN ID mappings.

---

## 4. Functional Requirements

### A) Hardware
- ESP32 board with onboard CAN transceiver and 12 V regulator (e.g., **LILYGO T‑CAN485**).
- Harness with MQS connectors, inline fuse, twisted pair for CAN H/L.
- ABS enclosure with strain relief, surge protection (TVS diode).

### B) Firmware (ESPHome‑based)
1. **CAN Interface**
   - Mode: LISTEN_ONLY (default), NORMAL (guarded).
   - Bitrate: 500 kbps.
   - Capture all standard CAN frames.
2. **Data Handling**
   - ISR → ring buffer → dispatcher.
   - Filters: ID/mask, DLC, change‑only, rate limit.
   - Logging: rotating buffers with export (CSV, PCAP, JSONL).
3. **Web UI**
   - Live CAN message table.
   - Filters/bookmarks.
   - Stats dashboard (frames/s, errors).
   - Log management (download, delete).
4. **Home Assistant Integration**
   - Native ESPHome API (preferred) with auto discovery.
   - Entities: sensors, binary_sensors, switches, numbers, lights.
   - Guarded control via TX Arm switch.
   - Optional MQTT discovery.
5. **OTA Updates**
   - Update firmware via ESPHome API.
   - Configurable YAML settings.

### C) Control Path
- Home Assistant control entities map to CAN TX only when **TX Arm** is enabled.
- Example: Camper LED brightness slider → CAN payload → update entity state from RX.

---

## 5. Non‑Functional Requirements
- **Performance**: Handle ≥ 2000 frames/s without data loss.
- **Latency**: < 60 ms from CAN RX to Web UI; < 100 ms to HA entity update.
- **Reliability**: Safe LISTEN_ONLY default, watchdog, graceful error handling.
- **Security**: WPA2 Wi‑Fi, optional Web UI auth.
- **Durability**: Automotive‑grade harness, power surge protection.

---

## 6. Constraints & Assumptions
- Operates only on **Comfort‑CAN bus** (not drivetrain CAN).
- Community will contribute mappings of CAN IDs (e.g., lights, fridge).
- Limited flash storage: rotating logs ~10 MB max unless SD card extension.
- Default installation at **gateway connector in driver footwell**.

---

## 7. Success Metrics
- Web UI shows live CAN traffic within 5 s of boot.
- Users can export logs in at least 2 formats.
- HA entities auto‑discovered without manual config.
- Safe TX model prevents any bus errors when disabled.
- First community‑contributed CAN ID map published within 3 months.

---

## 8. Risks & Mitigations
- **Bus disruption risk** → Default LISTEN_ONLY, TX guarded.
- **Flash wear** → Rotating logs with limits.
- **User mis‑wiring** → Provide clear wiring guide, inline fuse.
- **Legal/compliance** → Document for research & hobbyist use only.

---

## 9. Future Enhancements
- SD card support for extended logging.
- BLE provisioning and mobile companion app.
- Richer HA entities (covers, HVAC, window state).
- AI‑assisted CAN ID decoding.
- Community‑maintained rulesets for VW camper features.

