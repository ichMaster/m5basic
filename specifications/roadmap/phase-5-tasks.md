# Phase 5 — ESP-NOW Tasks

## 1. ESP-NOW Core

### 1.1 Initialization (`src/phase7_8_espnow_lora.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `ENOW` — call `esp_now_init()`, set WiFi to STA mode if not already, print own MAC address | -- |
| 2 | Register ESP-NOW send callback — set success/failure flag for error reporting | -- |
| 3 | Register ESP-NOW receive callback — copy received data and sender MAC to buffer, set flag | -- |
| 4 | Error handling: `? ESP-NOW INIT FAILED` on hardware failure, `? ESP-NOW NOT INIT` if using commands before ENOW | -- |

### 1.2 Peer management (`src/phase7_8_espnow_lora.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `EPEER "AA:BB:CC:DD:EE:FF"` — parse MAC string, call `esp_now_add_peer()` | -- |
| 2 | Validate MAC format — error `? INVALID MAC FORMAT` on malformed string | -- |
| 3 | Error handling: `? ADD PEER FAILED` if peer table full or invalid | -- |

---

## 2. ESP-NOW Communication

### 2.1 Send and receive (`src/phase7_8_espnow_lora.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `ESEND "MAC", message$` — evaluate MAC and message expressions, call `esp_now_send()` | -- |
| 2 | Implement `ERECV var$` — non-blocking: check receive buffer flag, if data available copy to string variable, set E = 1 and R$ = sender MAC; if no data, set E = 0 | -- |
| 3 | Error handling: `? SEND FAILED` on transmission error | -- |
| 4 | Clear receive buffer after ERECV reads it | -- |

---

## 3. Example Programs

### 3.1 ESP-NOW examples (`examples/phase6/`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create chat program (sender side) — INPUT message, ESEND to peer, loop | -- |
| 2 | Create chat program (receiver side) — ERECV loop, print received messages with sender MAC | -- |
| 3 | Create sensor relay — periodic ESEND with simulated sensor data | -- |
| 4 | Create remote control (controller) — BTN press sends command strings | -- |
| 5 | Create remote control (receiver) — ERECV commands, execute actions (BEEP, LED, etc.) | -- |
