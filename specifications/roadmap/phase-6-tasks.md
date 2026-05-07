# Phase 6 — LoRa Tasks

## 1. LoRa Core

### 1.1 Initialization (`src/phase7_8_espnow_lora.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `LINIT [freq_mhz]` — call `LoRa.begin(freq * 1E6)`, default 868 MHz, enter receive mode | -- |
| 2 | Guard with `__has_include(<LoRa.h>)` — if library not installed, print `? LORA LIB NOT FOUND` | -- |
| 3 | Configure SPI pins for M5Stack LoRa Module (SX1276) | -- |
| 4 | Error handling: `? LORA INIT FAILED` on hardware failure, `? LORA NOT INIT` if using commands before LINIT | -- |

### 1.2 Frequency control (`src/phase7_8_espnow_lora.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `LFREQ freq_mhz` — call `LoRa.setFrequency(freq * 1E6)` to change frequency at runtime | -- |
| 2 | Validate frequency range (common ISM bands: 433, 868, 915 MHz) | -- |

---

## 2. LoRa Communication

### 2.1 Send and receive (`src/phase7_8_espnow_lora.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `LSEND message$` — call `LoRa.beginPacket()`, `LoRa.print()`, `LoRa.endPacket()`, return to receive mode | -- |
| 2 | Implement `LRECV var$` — non-blocking: call `LoRa.parsePacket()`, if data available read into string variable, set L = 1 and R = RSSI (`LoRa.packetRssi()`); if no data, set L = 0 | -- |
| 3 | Truncate received packets to `MAX_STRING_LEN` (63 chars) | -- |
| 4 | Error handling: `? SEND FAILED` on transmission error | -- |

---

## 3. Example Programs

### 3.1 LoRa examples (`examples/phase7/`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create LoRa beacon — periodic LSEND with device ID and counter, DELAY between sends | -- |
| 2 | Create LoRa receiver — LRECV loop, print message with RSSI value | -- |
| 3 | Create range tester — LSEND ping, LRECV response, display RSSI stats (min/max/avg) | -- |
| 4 | Create LoRa-WiFi gateway — LINIT + WIFI connect + WSERV, forward LoRa messages to web page | -- |
| 5 | Create LoRa chat — alternating LSEND/LRECV with INPUT for messages, half-duplex | -- |
