# Roadmap

## Phase 1 -- Core Interpreter (Serial Only, CLI)

**Goal:** Run BASIC programs on M5Stack via serial terminal. Includes embedded program loader, serial REPL, serial transfer protocol, and control flow.

**Final milestone:** `python tools/bas2h.py programs/guess_game.bas && pio run -e phase0 -t upload && pio device monitor` -- game runs on boot, then REPL prompt accepts PRINT/FOR/IF commands. `python tools/m5basic_upload.py upload programs/demo.bas` uploads a different program without reflashing.

**Dependencies:** None -- this is the foundation.

### Phase 1A -- Minimal Working Interpreter

**Milestone:** Connect via serial terminal, type `10 PRINT "Hello"` / `20 A = 5` / `30 PRINT A + 1` / `RUN` -- it works.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| -- | PlatformIO project setup | platformio.ini with phase0, phase0-repl, phase1, phase2, full environments | Not started |
| -- | Header files | config.h, tokens.h (~80 keywords), interpreter.h (class declaration) | Not started |
| mb-3 | Lexer | Single-pass tokenizer, keyword table (~80 entries), Lexer class | Not started |
| mb-3 | Basic expression evaluator | Recursive descent: arithmetic, comparisons, logic, variables, parentheses | Not started |
| mb-3 | Core statements | PRINT, LET, REM, END. Program storage (500 lines). Variable storage (A-Z, A$-Z$) | Not started |
| mb-1 | Serial I/O and REPL | consolePrint/ReadLine, REPL loop, RUN/NEW/LIST/HELP commands | Not started |

### Phase 1B -- Control Flow and INPUT

**Milestone:** Write and run a number guessing game: WHILE loop, IF/THEN, INPUT, GOSUB/RETURN.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-3 | INPUT, CLR, colon separator | User input, variable reset, multiple statements per line | Not started |
| mb-4 | Control flow | IF/THEN/ELSE, GOTO, FOR/TO/STEP/NEXT, WHILE/WEND, GOSUB/RETURN | Not started |

### Phase 1C -- Functions, Strings, and Arrays

**Milestone:** `PRINT LEFT$("Hello", 3)` outputs `Hel`. `DIM A(10) : FOR I = 0 TO 10 : A(I) = I * I : NEXT I` works.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-3 | Math functions | ABS, INT, SQR, RND in expression evaluator | Not started |
| mb-9 | String functions | LEN, LEFT$, RIGHT$, MID$, STR$, VAL, CHR$, ASC, UPPER$, LOWER$, INSTR | Not started |
| mb-10 | Arrays | DIM for 1D and 2D numeric arrays | Not started |

### Phase 1D -- Tools and Delivery

**Milestone:** Embed programs in firmware, upload over serial without reflashing, 8 example programs verified on hardware.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-2 | Embedded program loader | bas2h.py tool, loadEmbeddedProgram(), auto-run on boot | Not started |
| mb-5 | Serial transfer protocol | >>>XFER upload, >>>FETCH download, >>>LIST files. Polled during REPL idle | Not started |
| mb-18 | PC upload tool | m5basic_upload.py: upload, download, list, install-examples commands | Not started |
| -- | Example programs | 8 serial-only examples: hello, math, fibonacci, guess game, primes, menu, adventure | Not started |

---

## Phase 2 -- Display and Graphics

**Goal:** LCD screen shows interpreter output. Graphics primitives allow visual programs. Serial remains the input source.

**Milestone:** `pio run -e phase1 -t upload` -- PRINT output appears on both serial and 320x240 LCD simultaneously. Upload bouncing ball program -- animation plays on screen, press key in serial to stop.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-6 | LCD text console | 53x30 character grid, 6x8 font, auto-scroll, CLS, LOCATE, COLOR | Not started |
| mb-7 | Graphics primitives | PIXEL, LINE, RECT, FILLRECT, CIRCLE, FILLCIRC with RGB565 color argument | Not started |
| mb-8 | Sound output | BEEP freq, duration_ms via M5Stack speaker | Not started |
| -- | Hardware input functions | BTN(n) for 3 buttons, MILLIS for uptime, DELAY for pause, INKEY$ for key polling | Not started |
| -- | Display examples | Bouncing ball animation, bar chart, color text demo, piano keyboard, timer | Not started |

**Dependencies:** Phase 1 complete.

---

## Phase 3 -- CardKB, SD Card, File System

**Goal:** Self-contained device. CardKB for physical input, SD card for program persistence. No PC needed after initial flash.

**Milestone:** Attach CardKB. Type `10 PRINT "HELLO"` on physical keyboard, `SAVE "test.bas"`, `NEW`, `LOAD "test.bas"`, `RUN` -- works. `DIR` lists files. Reboot with `autorun.bas` on SD -- program runs automatically.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-11 | CardKB input | I2C polling at 0x5F, dual input (CardKB + serial both active) | Not started |
| -- | BtnB BREAK | Hardware button B stops running programs | Not started |
| mb-12 | SD card file system | initSD, SAVE, LOAD, RUN "file", DIR, DELETE, CAT | Not started |
| mb-20 | Autorun | Load and run /basic/autorun.bas at boot if present | Not started |
| mb-13 | Advanced file I/O | OPEN/CLOSE/FPRINT/FINPUT with 3 file handles (R/W/A modes) | Not started |
| mb-5 | Serial transfer with SD | >>>XFER saves to SD card, >>>FETCH reads from SD | Not started |
| mb-19 | SD deploy tool | deploy_to_sd.sh for batch-copying example files | Not started |
| -- | File system examples | Data logger (CSV), config manager, high score system, file browser | Not started |

**Dependencies:** Phase 1 complete. CardKB and microSD card physically available.

---

## Phase 4 -- WiFi and Web Server

**Goal:** Network connectivity. Programs fetch data from the internet and serve web pages.

**Milestone:** `WIFI "ssid", "pass"` connects and prints IP. `WGET "http://httpbin.org/ip", R$` stores response. Start web server, visit IP in browser -- see HTML page served by BASIC program.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-14 | WiFi client | WIFI "ssid", "pass" with 10s timeout. IP$ function. WiFi STA mode | Not started |
| mb-14 | HTTP GET | WGET "url", var$ with response body and H variable for status code | Not started |
| mb-15 | Web server | WSERV [port], WSTOP, WSEND code "html". W variable for pending request | Not started |
| -- | WiFi examples | Dashboard with auto-refresh, REST JSON endpoint, IoT data poster, temperature monitor | Not started |

**Dependencies:** Phase 1 complete. WiFi network available.

---

## Phase 5 -- ESP-NOW

**Goal:** Peer-to-peer communication between M5Stack devices without WiFi infrastructure.

**Milestone:** Two M5Stacks: device A sends "HELLO" via ESEND, device B prints "HELLO" with sender MAC via ERECV. Chat example: type on device A serial, text appears on device B screen.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-16 | ESP-NOW init | ENOW command, print own MAC, register recv/send callbacks | Not started |
| mb-16 | Peer management | EPEER "AA:BB:CC:DD:EE:FF" to register peers | Not started |
| mb-16 | Send/receive | ESEND "MAC", msg$. ERECV var$ (non-blocking). E flag, R$ sender MAC | Not started |
| -- | ESP-NOW examples | Chat, sensor relay, remote control (controller + receiver) | Not started |

**Dependencies:** Phase 1 complete. Two or more M5Stack devices.

---

## Phase 6 -- LoRa

**Goal:** Long-range radio communication for field deployments. Requires M5Stack LoRa Module (SX1276).

**Milestone:** Device A sends beacon every 5 seconds via LSEND. Device B receives via LRECV and prints message with RSSI. Range test: packets received at 100m+ distance.

| ID | Feature | Scope | Status |
|----|---------|-------|--------|
| mb-17 | LoRa init | LINIT [freq_mhz], default 868 MHz. SPI setup for SX1276 | Not started |
| mb-17 | Send/receive | LSEND msg$. LRECV var$ (non-blocking). L flag, R for RSSI | Not started |
| mb-17 | Frequency control | LFREQ freq_mhz to change frequency at runtime | Not started |
| -- | LoRa examples | Beacon, range tester with RSSI stats, LoRa-WiFi gateway (combines Phase 4 + 6) | Not started |

**Dependencies:** Phase 1 complete. M5Stack LoRa Module physically available.

---

## Phase Summary

| Phase | Focus | Features | Depends on |
|-------|-------|----------|------------|
| **1A** | Minimal working interpreter -- lexer, basic expressions, PRINT/LET, serial REPL | mb-1, mb-3 | -- |
| **1B** | Control flow and INPUT -- IF/FOR/WHILE/GOSUB, INPUT, colon separator | mb-3, mb-4 | 1A |
| **1C** | Functions, strings, arrays -- math/string functions, DIM arrays | mb-9, mb-10 | 1A |
| **1D** | Tools and delivery -- embedded loader, serial transfer, upload tool, examples | mb-2, mb-5, mb-18 | 1A |
| **2** | Display -- LCD text console, graphics, sound, button/timer input | mb-6, mb-7, mb-8 | Phase 1 |
| **3** | Standalone -- CardKB input, SD card file system, autorun, advanced file I/O | mb-11, mb-12, mb-13, mb-19, mb-20 | Phase 1 |
| **4** | Network -- WiFi client, HTTP GET, web server | mb-14, mb-15 | Phase 1 |
| **5** | Peer-to-peer -- ESP-NOW init, peer management, send/receive | mb-16 | Phase 1 |
| **6** | Long-range -- LoRa init, send/receive, frequency control | mb-17 | Phase 1 |

**Note:** Phases 2-6 all depend only on Phase 1 and are independent of each other. They can be developed and tested in any order. The recommended sequence is 1 -> 2 -> 3 -> 4 -> 5 -> 6 because display (Phase 2) makes debugging easier, and file system (Phase 3) makes iteration faster. But a developer working on LoRa could jump from Phase 1 directly to Phase 6.
