# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

M5Basic is an educational BASIC interpreter for M5Stack Core Basic (ESP32). It is implemented in C++ (Arduino framework) with PlatformIO as the build system. The interpreter is built in phases — from serial-only MVP to a standalone device with display, keyboard, SD card, WiFi, ESP-NOW, and LoRa.

All specifications live in `specifications/`. No source code has been implemented yet.

## Build System

PlatformIO with multiple build environments. Each environment enables a different feature set via `#define` flags.

```bash
pio run -e phase0           # Serial + embedded program
pio run -e phase0-repl      # Serial REPL only (no embedded program)
pio run -e phase1           # + LCD display mirror
pio run -e phase2           # + CardKB + SD + graphics
pio run -e full             # All phases including WiFi/ESP-NOW/LoRa

pio run -e <env> -t upload  # Build and flash to M5Stack
pio device monitor          # Open serial terminal (115200 baud)
```

### Build Flag Matrix

| Environment | SERIAL_ONLY | DISPLAY | CARDKB | FILESYSTEM | EMBEDDED | PHASE2 | PHASE3 | PHASE4-8 |
|-------------|:-----------:|:-------:|:------:|:----------:|:--------:|:------:|:------:|:--------:|
| phase0      | x | | | | x | x | | |
| phase0-repl | x | | | | | x | | |
| phase1      | x | x | | | | x | x | |
| phase2      | | x | x | x | | x | x | x (4,5) |
| full        | | x | x | x | | x | x | all |

## Architecture

**Single class, many files.** `BasicInterpreter` is declared in `interpreter.h` but implemented across multiple `.cpp` files selected by `build_src_filter` and `#ifdef` guards. Only one I/O implementation is active per build.

### Source File Responsibilities

| File | Role |
|------|------|
| `main.cpp` | Entry point (setup/loop) |
| `lexer.cpp` | Tokenization, keyword lookup (~80 keywords) |
| `interpreter_core.cpp` | Expression evaluator (recursive descent), statement dispatcher, variable/program storage, full-hardware I/O |
| `serial_io.cpp` | Serial-mode I/O, LCD mirror, embedded loader — active when `M5BASIC_SERIAL_ONLY` defined |
| `repl.cpp` | REPL loop, RUN/NEW/LIST/HELP commands |
| `phase1_files.cpp` | SD card ops, serial transfer protocol, autorun |
| `phase2_control.cpp` | IF/FOR/WHILE/GOSUB |
| `phase3_display.cpp` | Graphics primitives, BEEP, BTN, DELAY, MILLIS, INKEY$ |
| `phase4_5_arrays_files.cpp` | DIM arrays, OPEN/CLOSE/FPRINT/FINPUT |
| `phase6_wifi.cpp` | WIFI, WGET, WSERV/WSTOP/WSEND |
| `phase7_8_espnow_lora.cpp` | ESP-NOW and LoRa commands |

### I/O Abstraction

Two implementations of the same console methods (`consolePrint`, `consoleReadLine`, etc.):
- `serial_io.cpp` — when `M5BASIC_SERIAL_ONLY` is defined (serial output, optional LCD mirror via `M5BASIC_DISPLAY`)
- `interpreter_core.cpp` — when not defined (LCD + CardKB primary, serial fallback)

### Execution Pipeline

Source line → Lexer (single-pass tokenizer) → Token stream → Statement executor (switch dispatch) → Expression evaluator (recursive descent: evalExpr → evalLogic → evalComparison → evalAddSub → evalMulDiv → evalUnary → evalAtom)

## PC-Side Tools

```bash
python tools/bas2h.py program.bas                    # Convert .bas to C header for firmware embedding
python tools/bas2h.py program.bas -o include/embedded_program.h

python tools/m5basic_upload.py upload game.bas        # Upload to RAM via serial
python tools/m5basic_upload.py upload game.bas --save  # Upload to RAM + save to SD
python tools/m5basic_upload.py upload game.bas --run   # Upload + run
python tools/m5basic_upload.py download backup.bas     # Download from device
python tools/m5basic_upload.py list                    # List SD files

bash tools/deploy_to_sd.sh                            # Batch-copy examples to SD
```

Requires `pyserial` (`pip install pyserial`).

## Key Constraints

- **26 numeric variables (A-Z), 26 string variables (A$-Z$)**, max 63-char strings, 500 program lines, 128 chars/line
- **No exceptions** — errors call `consoleError(msg)`, set `_running = false`, return to REPL
- **No threading** — single-threaded Arduino loop; ESP-NOW/LoRa callbacks set flags polled in main loop
- **LoRa is optional** — guarded by `__has_include(<LoRa.h>)`; stubs if library absent
- **Phase 2 (control flow) is always included from Phase 0** — without IF/FOR/WHILE/GOSUB the language is too limited
- **ESP32 memory budget** — ~270-330 KB SRAM free after framework; program storage uses up to 65 KB

## Phase Dependencies

Phases 2-6 all depend only on Phase 1 (core interpreter) and are independent of each other. Recommended order: 1 → 2 → 3 → 4 → 5 → 6, but any phase can be developed after Phase 1.
