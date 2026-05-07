# Phase 4 — WiFi and Web Server Tasks

## 1. WiFi Client

### 1.1 WiFi connection (`src/phase6_wifi.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `WIFI "ssid", "password"` — call `WiFi.begin()` in STA mode, block until connected or 10-second timeout | -- |
| 2 | Print IP address on successful connection | -- |
| 3 | Implement `IP$` string function — return `WiFi.localIP().toString()`, "0.0.0.0" if not connected; integrate into `evalStrExpr()` | -- |
| 4 | Error handling: `? WIFI TIMEOUT` after 10 seconds, `? WIFI NOT CONNECTED` for network commands before WIFI | -- |

### 1.2 HTTP client (`src/phase6_wifi.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `WGET "url", result$` — perform HTTP GET using `HTTPClient`, store response body (truncated to 63 chars) in string variable | -- |
| 2 | Store HTTP status code in variable H after WGET | -- |
| 3 | Handle redirect, timeout, and connection errors | -- |
| 4 | Check WiFi connected before WGET — error if not | -- |

---

## 2. Web Server

### 2.1 HTTP server (`src/phase6_wifi.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `WSERV [port]` — create `WebServer` instance on specified port (default 80), register catch-all handler | -- |
| 2 | Implement `WSTOP` — stop and destroy web server instance | -- |
| 3 | Implement `WSEND statuscode, "content"` — send HTTP response with Content-Type `text/html` to pending request | -- |
| 4 | Implement request flag: set variable W = 1 when request received, W = 0 after WSEND | -- |
| 5 | Call `server.handleClient()` in interpreter loop during DELAY and WHILE polling | -- |
| 6 | Check WiFi connected before WSERV — error if not | -- |

---

## 3. Example Programs

### 3.1 WiFi examples (`examples/phase5/`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create web dashboard — WIFI connect, WSERV, serve HTML with auto-refresh showing sensor-like data | -- |
| 2 | Create REST endpoint — WSERV serving JSON responses | -- |
| 3 | Create HTTP client demo — WGET to public API, display result | -- |
| 4 | Create IoT data poster — WGET to POST endpoint (simulated with GET), periodic send | -- |
| 5 | Create network scanner — WIFI connect, print IP$, serve status page | -- |
