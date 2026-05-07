# Requirements

BASIC interpreter for M5Stack Core Basic.
Phased implementation from serial-only MVP to full standalone device with wireless communication.

---

## 1. Goals

Educational BASIC interpreter that:

- Runs on M5Stack Core Basic (ESP32, 320x240 LCD, 3 buttons, SD slot)
- Starts as serial-only (Phase 0) -- no display, no keyboard, no SD needed
- First version: one BASIC program embedded in firmware alongside interpreter
- Progressively adds hardware: display, CardKB, SD card, WiFi, ESP-NOW, LoRa
- Each phase is self-contained and usable
- Each phase has learning examples

---

## 2. Hardware platform

| Component | Details |
|-----------|---------|
| MCU | ESP32 (dual-core, 240 MHz, 520KB SRAM) |
| Board | M5Stack Core Basic |
| Display | 320x240 IPS TFT (ILI9342C) |
| Buttons | 3 hardware (A, B, C) |
| SD slot | microSD, SPI |
| Keyboard | CardKB (I2C, address 0x5F) -- attached later |
| LoRa | M5Stack LoRa Module 868MHz (SX1276) -- optional |
| USB | USB-C, CP2104 serial bridge |
| Framework | Arduino via PlatformIO |

---

## 3. Phases

### Phase 0 -- Serial only, embedded program

**Hardware required:** M5Stack + USB cable. Nothing else.

**I/O:** All input/output through USB serial (115200 baud). PC terminal (PlatformIO monitor, PuTTY, minicom) is the user interface.

**Program delivery:** One BASIC program compiled into firmware as a C string array. Tool `bas2h.py` converts `.bas` file to a C header. Program runs automatically on boot, then interpreter enters REPL mode via serial.

**Workflow:**
```
1. Write program.bas in any text editor
2. python tools/bas2h.py program.bas
3. pio run -e phase0 -t upload
4. pio device monitor
5. Program runs, then REPL prompt appears
```

**Alternative workflow (REPL only):**
```
1. pio run -e phase0-repl -t upload
2. pio device monitor
3. Type BASIC lines directly in terminal
```

**Serial transfer protocol:** Even in Phase 0, programs can be uploaded via serial without re-flashing. Python tool `m5basic_upload.py` sends `.bas` files through a simple text protocol. M5Stack receives lines into RAM.

**Language subset:**
- PRINT, LET (optional), INPUT, REM, END
- Variables: A-Z (numeric), A$-Z$ (string)
- Math: +, -, *, /, MOD, parentheses
- Comparison: =, <>, <, >, <=, >=
- Logic: AND, OR, NOT
- Functions: ABS(), INT(), RND(), SQR()
- IF/THEN/ELSE, GOTO, FOR/TO/STEP/NEXT, WHILE/WEND, GOSUB/RETURN

Control flow is included from Phase 0 because without it programs are too limited to be interesting.

**What is NOT in Phase 0:**
- No display output (LCD stays dark or shows static splash)
- No CardKB
- No SD card operations
- No file save/load
- No graphics commands

---

### Phase 1 -- Display mirror

**Added hardware:** None new (uses existing LCD).

**Change:** Serial output is mirrored to the LCD screen. Serial remains the primary input. The display shows a text console (53 columns x 30 rows at 6x8 font).

**New commands:**
- CLS [color] -- clear screen
- LOCATE row, col -- position text cursor
- COLOR fg [, bg] -- text color (RGB565)

**No graphics yet** -- just text on screen. This validates the display driver and text rendering before adding drawing commands.

---

### Phase 2 -- Graphics and sound

**Added hardware:** None new.

**New commands:**
- PIXEL x, y [, color]
- LINE x1, y1, x2, y2 [, color]
- RECT x, y, w, h [, color]
- FILLRECT x, y, w, h [, color]
- CIRCLE x, y, r [, color]
- FILLCIRC x, y, r [, color]
- BEEP freq, duration_ms
- DELAY ms
- BTN(n) -- button state (1=A, 2=B, 3=C), returns 0 or 1
- MILLIS -- uptime in milliseconds
- INKEY$ -- last key pressed (from serial or CardKB when available)

**RGB565 color constants:**
Black=0, Red=63488, Green=2016, Blue=31, Yellow=65504, Cyan=2047, White=65535.

---

### Phase 3 -- Strings and arrays

**New commands:**
- DIM var(size) -- 1D array (0 to size inclusive)
- DIM var(rows, cols) -- 2D array
- Array access: A(i) or A(i, j)
- LEN(s$) -- string length
- LEFT$(s$, n), RIGHT$(s$, n), MID$(s$, pos, len)
- STR$(n) -- number to string
- VAL(s$) -- string to number
- CHR$(n) -- ASCII code to character
- ASC(s$) -- character to ASCII code
- UPPER$(s$), LOWER$(s$)
- INSTR(haystack$, needle$) -- substring search, returns position (1-based) or 0
- String concatenation with +

---

### Phase 4 -- CardKB and file system

**Added hardware:** CardKB (I2C keyboard module).

**Changes:**
- Input can come from CardKB OR serial (both active simultaneously)
- INKEY$ reads from CardKB when attached
- Button B on M5Stack = BREAK (stop running program)

**File system commands (SD card):**
- SAVE "filename" -- save current program to SD /basic/ directory
- LOAD "filename" -- load program from SD (auto-adds .bas extension)
- RUN "filename" -- load and immediately run
- DIR -- list files in /basic/
- DELETE "filename" -- remove file
- CAT "filename" -- display file contents on screen

**Autorun:** If `/basic/autorun.bas` exists on SD, it loads and runs at boot.

**Advanced file I/O:**
- OPEN "filename", #n, "mode" -- open file (mode: R/W/A, handles 1-3)
- CLOSE #n
- FPRINT #n, expression -- write line to file
- FINPUT #n, variable -- read line from file

---

### Phase 5 -- WiFi and web server

**New commands:**
- WIFI "ssid", "password" -- connect to WiFi network
- IP$ -- string function returning local IP address
- WGET "url", result$ -- HTTP GET, stores response in string variable. HTTP status code stored in variable H.
- WSERV [port] -- start web server (default port 80)
- WSTOP -- stop web server
- WSEND code, "content" -- send HTTP response to pending request

**Web server pattern:**
```basic
10 WIFI "MyNetwork", "pass123"
20 WSERV 80
30 PRINT "http://"; IP$
40 WHILE BTN(2) = 0
50   REM poll for requests
60   IF W = 1 THEN GOSUB 200
70   DELAY 100
80 WEND
90 WSTOP : END
200 P$ = "<h1>Hello from BASIC</h1>"
210 WSEND 200, P$
220 RETURN
```

Variable W is set to 1 when a web request is pending.

---

### Phase 6 -- ESP-NOW

Peer-to-peer communication between ESP32 devices. No WiFi access point needed. Range approximately 200m line-of-sight, latency under 1ms.

**New commands:**
- ENOW -- initialize ESP-NOW, prints own MAC address
- EPEER "AA:BB:CC:DD:EE:FF" -- add peer by MAC address
- ESEND "MAC", message$ -- send message to peer
- ERECV var$ -- non-blocking receive

**After ERECV:** Variable E = 1 if data received (0 otherwise). Variable R$ = sender's MAC address.

---

### Phase 7 -- LoRa

Long-range radio communication (up to 15+ km). Requires M5Stack LoRa Module (SX1276).

**New commands:**
- LINIT [freq_mhz] -- initialize LoRa (default 868 MHz)
- LSEND message$ -- transmit packet
- LRECV var$ -- non-blocking receive
- LFREQ freq_mhz -- change frequency

**After LRECV:** Variable L = 1 if data received. Variable R = RSSI in dBm.

**Frequency notes:** EU standard is 868 MHz. Ukrainian Meshtastic community uses 433 MHz.

---

## 4. Language specification

### Program structure

- Programs consist of numbered lines (1-65535)
- Lines execute in numeric order
- Multiple statements per line separated by colon (:)
- Direct mode: commands without line numbers execute immediately
- Line entered with only a number deletes that line

### Data types

- **Numeric:** floating point (stored as C float)
- **String:** up to 63 characters, variable names end with $
- **Arrays:** 1D or 2D, created with DIM, 0-based indexing

### Variables

- Numeric: single letter A-Z (26 variables)
- String: single letter + $: A$-Z$ (26 variables)
- Arrays: same letters, accessed with parentheses: A(i), B(i,j)
- Some variables have special meaning after certain operations:
  - H = HTTP status code (after WGET)
  - E = ESP-NOW receive flag (after ERECV)
  - R$ = sender MAC (after ERECV)
  - L = LoRa receive flag (after LRECV)
  - R = RSSI (after LRECV)
  - W = web request pending flag

### Expression precedence (high to low)

1. NOT, unary minus
2. *, /, MOD
3. +, -
4. =, <>, <, >, <=, >=
5. AND
6. OR

### Limits

| Resource | Limit |
|----------|-------|
| Program lines | 500 |
| Line length | 128 characters |
| Numeric variables | 26 (A-Z) |
| String variables | 26 (A$-Z$) |
| String length | 63 characters |
| Arrays | 10 |
| Array size | 100 elements per dimension |
| GOSUB depth | 10 |
| FOR nesting | 10 |
| File handles | 3 (#1 - #3) |

---

## 5. Serial transfer protocol

Allows uploading/downloading programs between PC and M5Stack via USB serial at any time, even while REPL is waiting for input.

### Upload (PC to M5Stack)

```
PC sends:    >>>XFER filename.bas\n
M5 responds: <<<READY\n
PC sends:    10 PRINT "Hello"\n
PC sends:    20 END\n
PC sends:    >>>END\n
M5 responds: <<<OK 2\n
```

If filename is provided, M5Stack saves to SD card (if available). Lines are parsed by line number and stored in interpreter memory.

### Download (M5Stack to PC)

```
PC sends:    >>>FETCH filename.bas\n
M5 responds: <<<LINE 10 PRINT "Hello"\n
M5 responds: <<<LINE 20 END\n
M5 responds: <<<EOF\n
```

### List files

```
PC sends:    >>>LIST\n
M5 responds: <<<LINE hello.bas 45\n
M5 responds: <<<LINE game.bas 128\n
M5 responds: <<<EOF\n
```

### Error

```
M5 responds: <<<ERR message\n
```

### Timeouts

5 seconds between lines during upload. If no data received within timeout, upload terminates with whatever lines were received.

---

## 6. Embedded program format (Phase 0)

Tool `bas2h.py` converts a `.bas` file to a C header:

**Input (program.bas):**
```basic
10 PRINT "Hello"
20 END
```

**Output (embedded_program.h):**
```c
static const EmbeddedLine EMBEDDED_PROGRAM[] = {
    {10, "PRINT \"Hello\""},
    {20, "END"},
    {0, nullptr}
};
```

At boot, the interpreter loads these lines into its program array and calls `runProgram()`. After the embedded program finishes (END or last line), the REPL prompt appears.

---

## 7. Build environments

| Environment | Command | What it does |
|-------------|---------|--------------|
| phase0 | `pio run -e phase0` | Serial + embedded program |
| phase0-repl | `pio run -e phase0-repl` | Serial REPL only |
| phase1 | `pio run -e phase1` | + LCD display mirror |
| phase2 | `pio run -e phase2` | + CardKB + SD + graphics |
| full | `pio run -e full` | All phases including WiFi/ESP-NOW/LoRa |

Build flags control compilation:

| Flag | Purpose |
|------|---------|
| M5BASIC_SERIAL_ONLY | Serial I/O (no full LCD console) |
| M5BASIC_DISPLAY | Mirror output to LCD |
| M5BASIC_CARDKB | Enable CardKB I2C input |
| M5BASIC_FILESYSTEM | Enable SD card operations |
| M5BASIC_EMBEDDED_PROGRAM | Load program from flash at boot |
| M5BASIC_PHASE2 | Control flow (IF/FOR/WHILE/GOSUB) |
| M5BASIC_PHASE3 | Graphics and sound |
| M5BASIC_PHASE4 | Strings and arrays |
| M5BASIC_PHASE5 | Advanced file I/O (OPEN/CLOSE) |
| M5BASIC_PHASE6 | WiFi and web server |
| M5BASIC_PHASE7 | ESP-NOW |
| M5BASIC_PHASE8 | LoRa |

---

## 8. Architecture

```
main.cpp                Entry point (setup/loop)
                        |
                        v
            +--[ BasicInterpreter ]--+
            |                        |
      I/O layer                  Core engine
   (serial_io.cpp or             (interpreter_core.cpp)
    interpreter_core.cpp)        |
            |                    +-- Lexer (tokens.h, lexer.cpp)
            |                    +-- Expression evaluator
            v                    +-- Statement executor
    Serial / LCD / CardKB        +-- Variable storage
                                 +-- Program storage (500 lines)
                                 |
                        Phase modules:
                        +-- phase2_control.cpp     (IF/FOR/WHILE)
                        +-- phase3_display.cpp     (graphics/sound)
                        +-- phase4_5_arrays_files.cpp (DIM/OPEN)
                        +-- phase1_files.cpp       (SAVE/LOAD/SD)
                        +-- phase6_wifi.cpp        (WiFi/HTTP)
                        +-- phase7_8_espnow_lora.cpp
```

**I/O abstraction:** When `M5BASIC_SERIAL_ONLY` is defined, `serial_io.cpp` provides `consolePrint`, `consoleReadLine`, etc. using Serial. When not defined, `interpreter_core.cpp` provides them using LCD + CardKB. The `M5BASIC_DISPLAY` flag enables LCD mirror in serial mode.

**Lexer:** Single-pass tokenizer. Keywords are looked up in a static table. Tokens include type, numeric value, string value, and identifier name.

**Parser:** Recursive descent for expressions. Statement dispatch via switch on token type.

**Program storage:** Array of 500 `BasicLine` structs, each holding a line number (uint16_t) and text (128 chars). Lines are kept sorted by number. GOTO/GOSUB do linear search by line number.

---

## 9. Tools

| Tool | Purpose |
|------|---------|
| `tools/bas2h.py` | Convert .bas to C header for embedding in firmware |
| `tools/m5basic_upload.py` | Upload/download programs via USB serial |
| `tools/deploy_to_sd.sh` | Batch-copy .bas files to SD card |

### bas2h.py

```
python tools/bas2h.py programs/game.bas
python tools/bas2h.py programs/game.bas -o include/embedded_program.h
python tools/bas2h.py programs/game.bas --name GAME
```

### m5basic_upload.py

Requires `pyserial` (`pip install pyserial`).

```
python tools/m5basic_upload.py upload game.bas           # to RAM
python tools/m5basic_upload.py upload game.bas --save     # to RAM + SD
python tools/m5basic_upload.py upload game.bas --run      # to RAM + run
python tools/m5basic_upload.py upload *.bas --save        # batch
python tools/m5basic_upload.py download backup.bas        # from M5 to PC
python tools/m5basic_upload.py list                       # list SD files
python tools/m5basic_upload.py install-examples           # all examples
```

---

## 10. Examples per phase

### Phase 0 examples

**0.1 -- Hello World**
```basic
10 PRINT "Hello from M5Basic!"
20 END
```

**0.2 -- Input and math**
```basic
10 INPUT "Your name: "; N$
20 PRINT "Hello, "; N$
30 INPUT "A number: "; X
40 PRINT X; " squared = "; X * X
50 END
```

**0.3 -- Fibonacci**
```basic
10 A = 0 : B = 1
20 FOR I = 1 TO 15
30   PRINT A; " ";
40   C = A + B : A = B : B = C
50 NEXT I
60 PRINT ""
70 END
```

**0.4 -- Guess the number**
```basic
10 S = RND(100) + 1
20 T = 0
30 WHILE G <> S
40   INPUT "Guess 1-100: "; G
50   T = T + 1
60   IF G < S THEN PRINT "Too low!"
70   IF G > S THEN PRINT "Too high!"
80 WEND
90 PRINT "Got it in "; T; " tries!"
100 END
```

**0.5 -- Multiplication table**
```basic
10 FOR I = 1 TO 9
20   FOR J = 1 TO 9
30     P = I * J
40     IF P < 10 THEN PRINT " ";
50     PRINT P; " ";
60   NEXT J
70   PRINT ""
80 NEXT I
90 END
```

**0.6 -- Prime checker**
```basic
10 INPUT "Number: "; N
20 IF N < 2 THEN PRINT "Not prime" : END
30 P = 1
40 FOR I = 2 TO SQR(N)
50   IF N MOD I = 0 THEN P = 0
60 NEXT I
70 IF P THEN PRINT N; " is prime"
80 IF NOT P THEN PRINT N; " is NOT prime"
90 END
```

**0.7 -- Menu with subroutines**
```basic
10 PRINT "1) Dice  2) Coin  3) Quit"
20 INPUT "Choice: "; C
30 IF C = 1 THEN GOSUB 100
40 IF C = 2 THEN GOSUB 200
50 IF C = 3 THEN END
60 GOTO 10
100 PRINT "Dice: "; RND(6) + 1
110 RETURN
200 IF RND(2) = 0 THEN PRINT "Heads"
210 IF RND(2) = 1 THEN PRINT "Tails"
220 RETURN
```

**0.8 -- Text adventure**
```basic
10 H = 100 : G = 0
20 PRINT "HP:"; H; " Gold:"; G
30 PRINT "1=North 2=East 3=Rest"
40 INPUT "> "; D
50 IF D = 1 THEN GOSUB 200
60 IF D = 2 THEN GOSUB 300
70 IF D = 3 THEN H = H + 10 : PRINT "Rested."
80 IF H <= 0 THEN PRINT "You died!" : END
90 GOTO 20
200 G = G + RND(30) + 5
210 PRINT "Found "; G; " gold!"
220 RETURN
300 D = RND(20) + 5
310 PRINT "Monster hits for "; D
320 H = H - D
330 RETURN
```

### Phase 1 examples (display)

**1.1 -- Color text**
```basic
10 CLS
20 COLOR 63488
30 LOCATE 2, 5
40 PRINT "RED TEXT"
50 COLOR 2016
60 LOCATE 4, 5
70 PRINT "GREEN TEXT"
80 END
```

### Phase 2 examples (graphics)

**2.1 -- Bouncing ball**
```basic
10 CLS
20 X = 160 : Y = 120 : DX = 3 : DY = 2
30 WHILE INKEY$ = ""
40   FILLCIRC X, Y, 8, 0
50   X = X + DX : Y = Y + DY
60   IF X<8 OR X>312 THEN DX = 0-DX
70   IF Y<8 OR Y>232 THEN DY = 0-DY
80   FILLCIRC X, Y, 8, 2016
90   DELAY 20
100 WEND
110 END
```

**2.2 -- Bar chart**
```basic
10 CLS
20 FILLRECT 30,  180, 50, 50,  63488
30 FILLRECT 100, 130, 50, 100, 2016
40 FILLRECT 170, 160, 50, 70,  31
50 FILLRECT 240, 100, 50, 130, 65504
60 LOCATE 28, 5
70 PRINT "Q1  Q2  Q3  Q4"
80 END
```

### Phase 3 examples (strings/arrays)

**3.1 -- Bubble sort**
```basic
10 DIM A(9)
20 FOR I = 0 TO 9 : A(I) = RND(100) : NEXT I
30 FOR I = 0 TO 8
40   FOR J = 0 TO 8 - I
50     IF A(J) > A(J+1) THEN T=A(J) : A(J)=A(J+1) : A(J+1)=T
60   NEXT J
70 NEXT I
80 FOR I = 0 TO 9 : PRINT A(I); " "; : NEXT I
90 END
```

**3.2 -- Caesar cipher**
```basic
10 INPUT "Text: "; T$
20 INPUT "Shift: "; S
30 R$ = ""
40 FOR I = 1 TO LEN(T$)
50   A = ASC(MID$(T$, I, 1))
60   IF A>=65 AND A<=90 THEN A = ((A-65+S) MOD 26) + 65
70   R$ = R$ + CHR$(A)
80 NEXT I
90 PRINT "Result: "; R$
100 END
```

### Phase 4 examples (files)

**4.1 -- Data logger**
```basic
10 OPEN "log.csv", 1, "W"
20 FPRINT 1, "time,value"
30 FOR I = 1 TO 10
40   V = RND(100)
50   FPRINT 1, STR$(MILLIS) + "," + STR$(V)
60   PRINT "Logged: "; V
70   DELAY 1000
80 NEXT I
90 CLOSE 1
100 PRINT "Saved to log.csv"
110 END
```

### Phase 5 examples (WiFi)

**5.1 -- Web dashboard**
```basic
10 WIFI "SSID", "password"
20 WSERV 80
30 PRINT "http://"; IP$
40 WHILE BTN(2) = 0
50   IF W = 1 THEN GOSUB 100
60   DELAY 100
70 WEND
80 WSTOP : END
100 T = 20 + RND(10)
110 P$ = "<h1>Temp: " + STR$(T) + "C</h1>"
120 WSEND 200, P$
130 RETURN
```

### Phase 6 examples (ESP-NOW)

**6.1 -- Sensor network (sender)**
```basic
10 ENOW
20 EPEER "AA:BB:CC:DD:EE:FF"
30 WHILE BTN(2) = 0
40   T = 20 + RND(10)
50   ESEND "AA:BB:CC:DD:EE:FF", "T:" + STR$(T)
60   PRINT "Sent T="; T
70   DELAY 5000
80 WEND
90 END
```

### Phase 7 examples (LoRa)

**7.1 -- LoRa-WiFi gateway**
```basic
10 WIFI "SSID", "pass"
20 LINIT 868
30 WSERV 80
40 D$ = "(waiting)"
50 WHILE BTN(2) = 0
60   LRECV M$
70   IF L = 1 THEN D$ = M$
80   IF W = 1 THEN GOSUB 200
90   DELAY 50
100 WEND
110 WSTOP : END
200 P$ = "<h1>LoRa: " + D$ + "</h1>"
210 WSEND 200, P$
220 RETURN
```

---

## 11. Error handling

Errors stop program execution and print a message prefixed with `?`:

```
? SYNTAX ERROR IN EXPRESSION
? LINE 150 NOT FOUND
? DIVISION BY ZERO
? NEXT WITHOUT FOR
? GOSUB STACK OVERFLOW
? OUT OF MEMORY
? SD CARD NOT READY
? WIFI TIMEOUT
? FILE NOT FOUND
```

In Phase 0 (serial), errors print to serial. When display is active, errors show in red.

---

## 12. Future extensions (out of scope)

Ideas for after all 8 phases are complete:

- DATA/READ statements
- DEF FN user-defined functions
- Sprite engine for games
- I2C/SPI sensor access from BASIC
- Bluetooth BLE
- MQTT for IoT
- JSON parser for REST APIs
- Full-screen text editor on LCD
- Multi-variable names (not just single letters)
