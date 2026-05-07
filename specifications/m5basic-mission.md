# Missions

Deliver a phased, educational BASIC interpreter for M5Stack Core Basic that starts as a serial-only embedded program runner and progressively adds display, keyboard, file system, and wireless communication capabilities (WiFi, ESP-NOW, LoRa).

## Vision

### Serial-first bootstrap
The first usable version requires only an M5Stack and a USB cable. No display output, no keyboard, no SD card. The BASIC program is compiled into the firmware as a C string array. The user writes `.bas` files on a PC, runs `bas2h.py` to embed the program, flashes the firmware, and opens a serial terminal. The program runs immediately on boot, then a REPL prompt appears for interactive experimentation.

### Progressive hardware adoption
Each phase adds exactly one hardware layer on top of what already works. Display mirrors serial output. CardKB adds physical input. SD card enables persistence. WiFi enables networking. ESP-NOW enables peer-to-peer. LoRa enables long-range radio. No phase breaks what the previous phase delivered. No phase requires hardware that isn't explicitly listed.

### One program at a time
In Phase 0, there is exactly one BASIC program on the device -- the one embedded in firmware. This is deliberate. It eliminates the need for file systems, transfer protocols, and storage management at the start. A single program in flash is the simplest possible delivery mechanism for running code on a microcontroller.

### Small language, real hardware
The BASIC dialect is intentionally minimal: 26 numeric variables, 26 string variables, 500 lines, 128 characters per line. These are constraints, not limitations. A small language is learnable in a day, fits comfortably in ESP32 memory, and forces programs to be short and readable. But the programs control real hardware: drawing to a physical screen, playing sounds through a physical speaker, communicating over real radio links.

### Serial transfer as escape hatch
Even in Phase 0, a serial transfer protocol allows uploading programs from a PC without reflashing. The Python tool `m5basic_upload.py` sends `.bas` files through a simple text protocol. This bridges the gap between "one embedded program" and "full file system" -- the user can iterate without recompiling.

## Goals

1. **Run BASIC on M5Stack with zero peripherals** -- Phase 0 requires only the M5Stack board and a USB cable
2. **Embed programs in firmware** -- write BASIC on a PC, bake it into flash, run on boot
3. **Add hardware layers incrementally** -- display, keyboard, SD card, WiFi, ESP-NOW, LoRa as separate phases
4. **Provide a serial REPL from Phase 0** -- type BASIC commands in a serial terminal, see results immediately
5. **Support serial program transfer** -- upload/download programs via USB without reflashing, starting from Phase 0
6. **Enable wireless experimentation in BASIC** -- ESP-NOW peer-to-peer and LoRa long-range communication as BASIC commands
7. **Include learning examples per phase** -- each phase ships with 6-10 example programs demonstrating its features

## Target Users

| User | Role in workflow |
|------|-----------------|
| **Embedded beginner** | Learns programming on a physical device; types BASIC instead of writing C++ |
| **Maker / prototyper** | Tests hardware setups (sensors, radios, displays) with 10-line scripts instead of full compile cycles |
| **Robotics experimenter** | Programs mesh network nodes with BASIC; uploads new behavior via serial without reflashing |
| **Retro computing hobbyist** | Builds a self-contained programmable handheld with CardKB, SD card, and LCD |
| **Educator** | Uses M5Basic as a teaching platform where students see immediate results on real hardware |

## Key Features

| ID | Feature | Description | Priority |
|----|---------|-------------|----------|
| mb-1 | Serial REPL | Interactive BASIC prompt via USB serial at 115200 baud | Must have |
| mb-2 | Embedded program loader | `bas2h.py` converts `.bas` to C header; program runs on boot | Must have |
| mb-3 | Core BASIC interpreter | Lexer, expression evaluator, PRINT/LET/INPUT/REM/END | Must have |
| mb-4 | Control flow | IF/THEN/ELSE, GOTO, FOR/NEXT, WHILE/WEND, GOSUB/RETURN | Must have |
| mb-5 | Serial transfer protocol | Upload/download programs via `>>>XFER` protocol over USB | Must have |
| mb-6 | LCD text console | 53x30 character grid mirroring serial output to 320x240 TFT | Must have |
| mb-7 | Graphics primitives | PIXEL, LINE, RECT, CIRCLE, FILLRECT, FILLCIRC with RGB565 colors | Must have |
| mb-8 | Sound output | BEEP command with frequency and duration | Should have |
| mb-9 | String functions | LEFT$, RIGHT$, MID$, STR$, VAL, CHR$, ASC, LEN, INSTR, UPPER$, LOWER$ | Must have |
| mb-10 | Arrays | DIM for 1D and 2D arrays with numeric data | Must have |
| mb-11 | CardKB input | I2C keyboard polling with serial fallback | Must have |
| mb-12 | SD card file system | SAVE, LOAD, DIR, DELETE, CAT; autorun.bas on boot | Must have |
| mb-13 | Advanced file I/O | OPEN/CLOSE/FPRINT/FINPUT with 3 file handles | Should have |
| mb-14 | WiFi client | WIFI connect, IP$, WGET for HTTP GET | Should have |
| mb-15 | Web server | WSERV/WSEND/WSTOP for serving HTML from BASIC | Should have |
| mb-16 | ESP-NOW | ENOW/EPEER/ESEND/ERECV for peer-to-peer communication | Should have |
| mb-17 | LoRa radio | LINIT/LSEND/LRECV/LFREQ for long-range communication | Nice to have |
| mb-18 | PC upload tool | `m5basic_upload.py` Python CLI for serial upload/download/list | Must have |
| mb-19 | SD deploy tool | `deploy_to_sd.sh` for batch-copying examples to SD card | Nice to have |
| mb-20 | Autorun | `/basic/autorun.bas` loads and runs on boot | Should have |

## Success Criteria

1. Phase 0 firmware with embedded "guess the number" game flashes and runs on M5Stack -- serial terminal shows prompts, accepts input, game works
2. `m5basic_upload.py upload game.bas` sends a program over USB and it executes on the device without reflashing
3. Phase 1 LCD mirror shows the same text output as serial terminal simultaneously
4. A self-contained M5Stack with CardKB and SD card can edit, save, load, and run BASIC programs without a PC
5. Two M5Stacks running ESP-NOW chat programs can exchange messages in real time
6. A LoRa beacon program on one M5Stack is received by a LoRa receiver program on another at 100m+ distance
7. Each phase has 6+ working example programs that demonstrate its features
