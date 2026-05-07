# Architecture

## System Overview

The system is a BASIC interpreter running on M5Stack Core Basic (ESP32). It reads BASIC source lines from memory, tokenizes, parses, and executes them. I/O is routed through an abstraction layer that targets either USB serial (Phase 0) or LCD + CardKB (Phase 4+). Programs enter the device via firmware embedding, serial transfer protocol, or SD card.

```
+------------------------------------------------------------------+
|  M5Stack Core Basic                                              |
|                                                                  |
|  +------------------+    +-----------------------------------+   |
|  | Hardware         |    | M5Basic Interpreter               |   |
|  |                  |    |                                   |   |
|  | ESP32 MCU        |    |  BasicInterpreter class           |   |
|  | 320x240 LCD      |<-->|    +-- I/O layer                  |   |
|  | 3 buttons        |    |    +-- Lexer                      |   |
|  | SD card slot     |    |    +-- Expression evaluator       |   |
|  | USB-C serial     |    |    +-- Statement executor         |   |
|  | I2C (CardKB)     |    |    +-- Program storage            |   |
|  | SPI (LoRa)       |    |    +-- Variable storage           |   |
|  | WiFi radio       |    |    +-- Phase modules              |   |
|  +------------------+    +-----------------------------------+   |
+------------------------------------------------------------------+
         |
         | USB serial (115200 baud)
         v
+------------------+
|  PC              |
|  Serial terminal |
|  m5basic_upload  |
|  bas2h.py        |
+------------------+
```

## Components

### 1. Lexer (mb-3)

Converts source text into a token stream. Single-pass, no lookahead beyond one token.

- **Input:** Raw text line (char array, max 128 chars)
- **Output:** Stream of Token structs (type, numVal, strVal, ident)
- **Implementation:** `tokens.h` (token enum, keyword table, Lexer class), `lexer.cpp` (tokenization logic)

Tokenization rules: skip whitespace; digit starts a number (atof); `"` starts a string literal (with escape sequences); letter starts an identifier or keyword (case-insensitive lookup in keyword table, ~80 entries); otherwise single/double-character operator.

### 2. Expression Evaluator (mb-3)

Recursive descent parser for numeric and string expressions.

- **Input:** Token stream from Lexer
- **Output:** float (numeric) or char array (string)
- **Implementation:** `interpreter_core.cpp` (evalExpr chain, evalStrExpr)

Call chain: evalExpr -> evalLogic (AND/OR) -> evalComparison (=/<>/</>) -> evalAddSub (+/-) -> evalMulDiv (*/\\/MOD) -> evalUnary (NOT/-) -> evalAtom (number/variable/function/parenthesized).

String expressions evaluated separately by evalStrExpr. PRINT determines which to call based on current token type.

### 3. Statement Executor (mb-3, mb-4)

Switch dispatch on token type. Each case calls a handler method.

- **Input:** Current token from Lexer
- **Output:** Side effects (console output, variable assignment, control flow change)
- **Implementation:** `interpreter_core.cpp` (execStatement, execPrint, execLet, execInput, execRem), phase module files for extended commands

Statements not available in the current build phase are guarded by `#ifdef` and either absent from the switch or implemented as stubs printing "NOT AVAILABLE".

### 4. Program Storage (mb-3)

Array of 500 BasicLine structs, sorted by line number.

- **Input:** addLine(lineNum, text), deleteLine(lineNum)
- **Output:** findLineIndex(lineNum) returns array index or -1
- **Implementation:** `interpreter_core.cpp`

Lines kept sorted by lineNum. Insertion shifts elements down. Deletion shifts up. GOTO/GOSUB do linear scan (under 1ms for 500 lines on ESP32).

### 5. I/O Layer (mb-1, mb-6, mb-11)

Abstraction over serial, LCD, and CardKB. Two implementations selected at compile time.

- **serial_io.cpp** — compiled when `M5BASIC_SERIAL_ONLY` defined. All output via `Serial.print`. Input via `Serial.read`. Optional LCD mirror (`M5BASIC_DISPLAY`). Optional CardKB polling (`M5BASIC_CARDKB`).
- **interpreter_core.cpp** — compiled when `M5BASIC_SERIAL_ONLY` not defined. LCD text grid (53x30 at 6x8 font) with scroll. CardKB as primary input, serial as fallback.

Methods: consolePrint, consolePrintLn, consolePrintNum, consoleError, consoleClear, consoleNewLine, consoleReadKey, consoleReadLine, consoleScroll.

### 6. REPL Loop (mb-1, mb-5)

Read-eval-print loop and program runner.

- **Input:** User input from I/O layer
- **Output:** Stored program lines or immediate execution results
- **Implementation:** `repl.cpp`

Logic: read line; if starts with digit, store as program line (or delete if digit only); otherwise tokenize and execute as direct command. Special commands: RUN, NEW, LIST, HELP, SAVE, LOAD, DIR, UPLOAD, DOWNLOAD. Serial transfer protocol polled every REPL cycle via checkSerial().

### 7. Embedded Program Loader (mb-2)

Loads a BASIC program from a const array in flash memory at boot.

- **Input:** `embedded_program.h` (generated by `bas2h.py`)
- **Output:** Program lines added to program storage, then runProgram() called
- **Implementation:** `serial_io.cpp` (loadEmbeddedProgram), active only when `M5BASIC_EMBEDDED_PROGRAM` defined

### 8. Serial Transfer Protocol (mb-5, mb-18)

Upload/download programs between PC and M5Stack over USB serial.

- **Input:** `>>>XFER`, `>>>FETCH`, `>>>LIST` markers from serial
- **Output:** `<<<READY`, `<<<OK`, `<<<LINE`, `<<<EOF`, `<<<ERR` responses
- **Implementation:** `phase1_files.cpp` (when `M5BASIC_FILESYSTEM`), `serial_io.cpp` stubs (when not)

Polled in two places: checkSerial() at top of REPL loop, and inside consoleReadLine() during input wait. Allows uploads at any time without user typing UPLOAD.

### 9. File System (mb-12, mb-13)

SD card operations for program persistence.

- **Input:** SAVE/LOAD/DIR/DELETE/CAT commands with filenames
- **Output:** Files in `/basic/` directory on SD card
- **Implementation:** `phase1_files.cpp` (basic ops), `phase4_5_arrays_files.cpp` (OPEN/CLOSE/FPRINT/FINPUT)

Autorun: if `/basic/autorun.bas` exists, loaded and run at boot.

### 10. Phase Modules (mb-4, mb-7 through mb-17)

Conditionally compiled source files, each adding a group of BASIC commands.

| Module | File | Build flag | Commands added |
|--------|------|-----------|----------------|
| Control flow | `phase2_control.cpp` | `M5BASIC_PHASE2` | IF, GOTO, FOR, WHILE, GOSUB, RETURN |
| Graphics/sound | `phase3_display.cpp` | `M5BASIC_PHASE3` | CLS, LOCATE, COLOR, PIXEL, LINE, RECT, CIRCLE, BEEP, DELAY, BTN, MILLIS, INKEY$ |
| Arrays | `phase4_5_arrays_files.cpp` | `M5BASIC_PHASE4` | DIM |
| Advanced file I/O | `phase4_5_arrays_files.cpp` | `M5BASIC_PHASE5` | OPEN, CLOSE, FPRINT, FINPUT |
| WiFi/web | `phase6_wifi.cpp` | `M5BASIC_PHASE6` | WIFI, WGET, WSERV, WSTOP, WSEND, IP$ |
| ESP-NOW | `phase7_8_espnow_lora.cpp` | `M5BASIC_PHASE7` | ENOW, EPEER, ESEND, ERECV |
| LoRa | `phase7_8_espnow_lora.cpp` | `M5BASIC_PHASE8` | LINIT, LSEND, LRECV, LFREQ |

## Data Flow

### Program execution pipeline

```
Source text (line from program storage)
       |
       v
   Lexer (tokenize)
       |
       v
   Token stream
       |
       v
   Statement executor (switch dispatch)
       |
       +-- Expression evaluator (for PRINT, LET, IF, etc.)
       +-- I/O layer (for PRINT, INPUT)
       +-- Control flow stacks (for FOR, GOSUB, WHILE)
       +-- Variable storage (for LET, INPUT)
       +-- Hardware calls (for PIXEL, BEEP, WIFI, LSEND, etc.)
```

### Program delivery paths

```
Path 1: Embedded in firmware (Phase 0)
  PC: .bas file --> bas2h.py --> embedded_program.h --> pio build --> flash
  M5Stack: boot --> loadEmbeddedProgram() --> runProgram() --> REPL

Path 2: Serial transfer (Phase 0+)
  PC: .bas file --> m5basic_upload.py --> >>>XFER protocol --> serial
  M5Stack: checkSerial() --> serialReceiveProgram() --> program in RAM --> RUN

Path 3: SD card (Phase 4+)
  PC: .bas file --> copy to SD /basic/ --> insert SD in M5Stack
  M5Stack: LOAD "file.bas" --> program in RAM --> RUN

Path 4: Autorun (Phase 4+)
  SD: /basic/autorun.bas exists
  M5Stack: boot --> checkAutorun() --> loadFile() --> runProgram() --> REPL
```

## Solution File Structure

```
m5basic/
|-- platformio.ini                 Build environments and flags
|
|-- include/
|   |-- config.h                   Constants, limits, pin definitions, protocol markers
|   |-- tokens.h                   Token enum, Token struct, Keyword table, Lexer class
|   |-- interpreter.h              BasicInterpreter class (all methods, all state)
|   |-- embedded_program.h         Generated: BASIC program as C array (Phase 0)
|
|-- src/
|   |-- main.cpp                   Entry point: setup() -> begin(), loop() -> loop()
|   |-- lexer.cpp                  Tokenization, keyword lookup
|   |-- interpreter_core.cpp       Core: constructor, full-hardware I/O, program/variable storage,
|   |                              expression evaluator, statement dispatcher, PRINT/LET/INPUT/REM
|   |-- serial_io.cpp              Serial-mode I/O, LCD mirror, embedded loader, file system stubs
|   |-- repl.cpp                   REPL loop, runProgram(), direct commands, HELP
|   |-- phase1_files.cpp           SD card ops, serial transfer, autorun (when FILESYSTEM)
|   |-- phase2_control.cpp         IF/FOR/WHILE/GOSUB (when PHASE2)
|   |-- phase3_display.cpp         Graphics, sound (when PHASE3)
|   |-- phase4_5_arrays_files.cpp  DIM (when PHASE4), OPEN/CLOSE/FPRINT/FINPUT (when PHASE5)
|   |-- phase6_wifi.cpp            WiFi, HTTP, web server (when PHASE6)
|   |-- phase7_8_espnow_lora.cpp   ESP-NOW (when PHASE7), LoRa (when PHASE8)
|
|-- programs/                      BASIC programs for firmware embedding
|   |-- test_minimal.bas           Smoke test: PRINT, variables, RND
|   |-- demo.bas                   Fibonacci + dice: FOR, INPUT, GOSUB
|   |-- guess_game.bas             Interactive game: WHILE, IF, INPUT
|
|-- examples/                      Learning examples per phase (8 directories)
|   |-- phase0/ .. phase7/         Each contains examples.bas with multiple programs
|
|-- tools/
|   |-- bas2h.py                   .bas to C header converter
|   |-- m5basic_upload.py          Serial upload/download CLI tool
|   |-- deploy_to_sd.sh            Batch SD card deployment
|
|-- docs/                          Specification documents
    |-- m5basic-mission.md
    |-- m5basic-architecture.md
    |-- m5basic-roadmap.md
    |-- m5basic-spec.md
    |-- m5basic-lang-ref.md
```

### Build flag matrix

| Environment | SERIAL_ONLY | DISPLAY | CARDKB | FILESYSTEM | EMBEDDED | PHASE2 | PHASE3 | PHASE4 | PHASE5 | PHASE6 | PHASE7 | PHASE8 |
|-------------|:-----------:|:-------:|:------:|:----------:|:--------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
| phase0 | x | | | | x | x | | | | | | |
| phase0-repl | x | | | | | x | | | | | | |
| phase1 | x | x | | | | x | x | | | | | |
| phase2 | | x | x | x | | x | x | x | x | | | |
| full | | x | x | x | | x | x | x | x | x | x | x |

## Memory Layout

| Resource | Size | Notes |
|----------|------|-------|
| Program storage | 65 KB max | 500 lines x 130 bytes |
| Numeric variables | 104 bytes | float[26] (A-Z) |
| String variables | 1,664 bytes | char[26][64] (A$-Z$) |
| Arrays | Up to 40 KB | Heap-allocated, max 10 arrays, 100 elements/dim |
| Control stacks | ~160 bytes | GOSUB[10] + FOR[10] + WHILE[10] |
| ESP32 SRAM total | 520 KB | After framework ~270-330 KB free |

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Language | C++ (Arduino) | Interpreter implementation |
| Build system | PlatformIO | Multi-environment builds, dependency management |
| Platform | espressif32 6.5.0 | ESP32 toolchain |
| Hardware library | M5Stack 0.4.6 | LCD, buttons, speaker, SD, I2C abstraction |
| WiFi | ESP32 WiFi (built-in) | STA mode for network access |
| HTTP client | HTTPClient (Arduino) | WGET implementation |
| HTTP server | WebServer (Arduino) | WSERV implementation |
| ESP-NOW | esp_now (ESP-IDF) | Peer-to-peer communication |
| LoRa | LoRa library (SX1276) | Long-range radio (optional module) |
| PC tools | Python 3 + pyserial | bas2h.py, m5basic_upload.py |

## Integration Points

| Integration | Protocol | Direction | Purpose |
|------------|----------|-----------|---------|
| PC serial terminal | USB serial 115200 | Bidirectional | REPL I/O, program transfer |
| SD card | SPI / FAT32 | Read/Write | Program persistence, data files |
| CardKB | I2C (0x5F) | Input | Physical keyboard |
| WiFi networks | 802.11 b/g/n | Outbound | HTTP client, web server |
| ESP-NOW peers | ESP-NOW (WiFi PHY) | Bidirectional | Peer-to-peer messaging |
| LoRa nodes | SX1276 SPI (868/433 MHz) | Bidirectional | Long-range radio packets |

## Error Handling

Errors call consoleError(msg) which prints the message, sets `_running = false`, and returns. The run loop sees `_running == false` and stops. No exception support (Arduino limitation). No error recovery within a running program -- any error stops execution and returns to REPL.

Runtime checks: division by zero, stack overflow/underflow (FOR/GOSUB/WHILE), undefined line (GOTO/GOSUB), file errors, network errors, parse errors.

## Constraints

1. **Phase 0 requires only USB** -- no other hardware, no SD card, no WiFi
2. **One class, many files** -- BasicInterpreter is declared in interpreter.h but implemented across multiple .cpp files selected by build_src_filter and #ifdef guards
3. **Two I/O implementations** -- serial_io.cpp and interpreter_core.cpp provide the same console methods; only one is active per build, selected by M5BASIC_SERIAL_ONLY
4. **Phase 2 (control flow) is included from Phase 0** -- without IF/FOR/WHILE/GOSUB the language is too limited for interesting programs
5. **LoRa library is optional** -- `__has_include(<LoRa.h>)` check; stubs if not present
6. **No threading** -- single-threaded Arduino loop; ESP-NOW and LoRa callbacks set flags that are polled in the main loop
7. **String length cap 63 chars** -- affects WGET response truncation and string concatenation limits
