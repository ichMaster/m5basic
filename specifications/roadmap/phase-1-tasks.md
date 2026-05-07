# Phase 1 — Core Interpreter (Serial Only) Tasks

Each subphase produces a working BASIC interpreter that compiles, flashes, and runs on hardware.

---

## Phase 1A — Minimal Working Interpreter

**Goal:** A BASIC interpreter you can interact with over serial. Supports PRINT, LET, REM, END, numeric expressions with arithmetic, program storage, and REPL.

**Deliverable:** Connect via serial terminal, type `10 PRINT "Hello"` / `20 A = 5` / `30 PRINT A + 1` / `RUN` — it works.

### 1A.1 PlatformIO project scaffold

| # | Task | Issue |
|---|------|-------|
| 1 | Create `platformio.ini` with 5 build environments: `phase0`, `phase0-repl`, `phase1`, `phase2`, `full` | M5B-001 |
| 2 | Define build flags per environment: `M5BASIC_SERIAL_ONLY`, `M5BASIC_DISPLAY`, `M5BASIC_CARDKB`, `M5BASIC_FILESYSTEM`, `M5BASIC_EMBEDDED_PROGRAM`, `M5BASIC_PHASE2`–`M5BASIC_PHASE8` | M5B-001 |
| 3 | Configure `build_src_filter` per environment to include only relevant `.cpp` files | M5B-001 |
| 4 | Set platform `espressif32@6.5.0`, board `m5stack-core-esp32`, framework `arduino` | M5B-001 |
| 5 | Add library dependencies: `M5Stack@0.4.6`, `LoRa` (optional) | M5B-001 |
| 6 | Create directory structure: `include/`, `src/`, `programs/`, `examples/`, `tools/`, `docs/` | M5B-001 |

### 1A.2 Configuration header (`include/config.h`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define program limits: `MAX_LINES` (500), `MAX_LINE_LEN` (128), `MAX_STRING_LEN` (63), `MAX_VARS` (26) | M5B-001 |
| 2 | Define array limits: `MAX_ARRAYS` (10), `MAX_ARRAY_DIM` (100) | M5B-001 |
| 3 | Define stack limits: `MAX_GOSUB_DEPTH` (10), `MAX_FOR_DEPTH` (10), `MAX_WHILE_DEPTH` (10) | M5B-001 |
| 4 | Define serial settings: `SERIAL_BAUD` (115200) | M5B-001 |
| 5 | Define serial transfer protocol markers: `>>>XFER`, `>>>FETCH`, `>>>LIST`, `>>>END`, `<<<READY`, `<<<OK`, `<<<LINE`, `<<<EOF`, `<<<ERR` | M5B-001 |
| 6 | Define display constants: `SCREEN_W` (320), `SCREEN_H` (240), `FONT_W` (6), `FONT_H` (8), `COLS` (53), `ROWS` (30) | M5B-001 |
| 7 | Define hardware pin assignments: CardKB I2C address (0x5F), LoRa SPI pins, button pins | M5B-001 |

### 1A.3 Interpreter class declaration (`include/interpreter.h`)

| # | Task | Issue |
|---|------|-------|
| 1 | Declare `BasicInterpreter` class with all public methods: `begin()`, `loop()`, `runProgram()`, `addLine()`, `deleteLine()` | M5B-001 |
| 2 | Declare REPL methods: `replLoop()`, `directCommand()`, `listProgram()`, `newProgram()` | M5B-001 |
| 3 | Declare I/O methods: `consolePrint()`, `consolePrintLn()`, `consolePrintNum()`, `consoleError()`, `consoleClear()`, `consoleNewLine()`, `consoleReadKey()`, `consoleReadLine()`, `consoleScroll()` | M5B-001 |
| 4 | Declare expression evaluator methods: `evalExpr()`, `evalLogic()`, `evalComparison()`, `evalAddSub()`, `evalMulDiv()`, `evalUnary()`, `evalAtom()`, `evalStrExpr()` | M5B-001 |
| 5 | Declare statement executor methods: `execStatement()`, `execPrint()`, `execLet()`, `execInput()`, `execRem()` | M5B-001 |
| 6 | Declare phase-guarded method stubs for control flow, graphics, files, WiFi, ESP-NOW, LoRa | M5B-001 |
| 7 | Declare private state: `BasicLine _program[MAX_LINES]`, `float _numVars[26]`, `char _strVars[26][MAX_STRING_LEN+1]`, `int _lineCount`, `int _pc`, `bool _running` | M5B-001 |
| 8 | Declare control flow stacks: `GosubEntry _gosubStack[]`, `ForEntry _forStack[]`, `WhileEntry _whileStack[]` with depth counters | M5B-001 |

### 1A.4 Entry point (`src/main.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create global `BasicInterpreter` instance | M5B-001 |
| 2 | `setup()` calls `interpreter.begin()` | M5B-001 |
| 3 | `loop()` calls `interpreter.loop()` | M5B-001 |

### 1A.5 Token definitions (`include/tokens.h`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define `TokenType` enum covering: `TOK_NUMBER`, `TOK_STRING`, `TOK_IDENT`, `TOK_EOF`, operators (`+`, `-`, `*`, `/`, `=`, `<>`, `<`, `>`, `<=`, `>=`), parentheses, comma, colon, semicolon | M5B-002 |
| 2 | Define keyword tokens (~80 entries): `TOK_PRINT`, `TOK_LET`, `TOK_INPUT`, `TOK_REM`, `TOK_END`, `TOK_IF`, `TOK_THEN`, `TOK_ELSE`, `TOK_GOTO`, `TOK_FOR`, `TOK_TO`, `TOK_STEP`, `TOK_NEXT`, `TOK_WHILE`, `TOK_WEND`, `TOK_GOSUB`, `TOK_RETURN`, `TOK_AND`, `TOK_OR`, `TOK_NOT`, `TOK_MOD`, etc. | M5B-002 |
| 3 | Define function tokens: `TOK_ABS`, `TOK_INT`, `TOK_SQR`, `TOK_RND`, `TOK_LEN`, `TOK_LEFT$`, `TOK_RIGHT$`, `TOK_MID$`, `TOK_STR$`, `TOK_VAL`, `TOK_CHR$`, `TOK_ASC`, `TOK_UPPER$`, `TOK_LOWER$`, `TOK_INSTR` | M5B-002 |
| 4 | Define display/hardware tokens (guarded): `TOK_CLS`, `TOK_LOCATE`, `TOK_COLOR`, `TOK_PIXEL`, `TOK_LINE_CMD`, `TOK_RECT`, `TOK_FILLRECT`, `TOK_CIRCLE`, `TOK_FILLCIRC`, `TOK_BEEP`, `TOK_DELAY`, `TOK_BTN`, `TOK_MILLIS`, `TOK_INKEY$` | M5B-002 |
| 5 | Define file/network tokens (guarded): `TOK_SAVE`, `TOK_LOAD`, `TOK_DIR`, `TOK_DELETE`, `TOK_CAT`, `TOK_OPEN`, `TOK_CLOSE`, `TOK_FPRINT`, `TOK_FINPUT`, `TOK_WIFI`, `TOK_WGET`, `TOK_WSERV`, `TOK_WSTOP`, `TOK_WSEND`, `TOK_ENOW`, `TOK_EPEER`, `TOK_ESEND`, `TOK_ERECV`, `TOK_LINIT`, `TOK_LSEND`, `TOK_LRECV`, `TOK_LFREQ` | M5B-002 |
| 6 | Define `Token` struct: `TokenType type`, `float numVal`, `char strVal[MAX_STRING_LEN+1]`, `char ident[16]` | M5B-002 |
| 7 | Define `Keyword` struct and static keyword table: `{ "PRINT", TOK_PRINT }`, ... | M5B-002 |
| 8 | Declare `Lexer` class: `init(const char* line)`, `nextToken()`, `peekToken()`, `currentToken()` | M5B-002 |

### 1A.6 Tokenizer implementation (`src/lexer.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement whitespace skipping | M5B-002 |
| 2 | Implement number tokenization: digit sequence → `atof()` → `TOK_NUMBER` with `numVal` | M5B-002 |
| 3 | Implement string literal tokenization: `"` delimited with escape sequences (`\"`, `\\`, `\n`, `\t`) → `TOK_STRING` with `strVal` | M5B-002 |
| 4 | Implement identifier/keyword tokenization: letter sequence → case-insensitive keyword table lookup → `TOK_xxx` or `TOK_IDENT` with `ident` | M5B-002 |
| 5 | Implement operator tokenization: single-char (`+`, `-`, `*`, `/`, `=`, `<`, `>`, `(`, `)`, `,`, `:`, `;`) and double-char (`<=`, `>=`, `<>`, `!=`) | M5B-002 |
| 6 | Implement `nextToken()` main dispatch loop | M5B-002 |
| 7 | Implement `peekToken()` — tokenize without advancing position | M5B-002 |

### 1A.7 Basic numeric expression evaluator (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `evalExpr()` — entry point, delegates to `evalLogic()` | M5B-003 |
| 2 | Implement `evalLogic()` — handles `AND`, `OR` | M5B-003 |
| 3 | Implement `evalComparison()` — handles `=`, `<>`, `<`, `>`, `<=`, `>=` | M5B-003 |
| 4 | Implement `evalAddSub()` — handles `+`, `-` | M5B-003 |
| 5 | Implement `evalMulDiv()` — handles `*`, `/`, `MOD` | M5B-003 |
| 6 | Implement `evalUnary()` — handles `NOT`, unary `-` | M5B-003 |
| 7 | Implement `evalAtom()` — handles: numeric literal, numeric variable (A-Z), parenthesized expression | M5B-003 |

Note: Math functions (ABS, INT, SQR, RND) and string functions (LEN, VAL, ASC, INSTR) are added to `evalAtom()` in Phase 1C.

### 1A.8 Core statements (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `execStatement()` — switch dispatch on current token type, delegate to handler methods | M5B-004 |
| 2 | Implement `execPrint()` — evaluate and print expressions, handle `;` (no space) and `,` (tab) separators, trailing `;` suppresses newline | M5B-004 |
| 3 | Implement `execLet()` — optional LET keyword, assign numeric or string expression to variable | M5B-004 |
| 4 | Implement `execRem()` — skip to end of line | M5B-004 |
| 5 | Implement `END` — set `_running = false` | M5B-004 |

Note: INPUT, CLR, and colon separator are added in Phase 1B.

### 1A.9 Program storage (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define `BasicLine` struct: `uint16_t lineNum`, `char text[MAX_LINE_LEN+1]` | M5B-004 |
| 2 | Implement `addLine(lineNum, text)` — insert into sorted `_program` array, shift elements down | M5B-004 |
| 3 | Implement `deleteLine(lineNum)` — remove from array, shift elements up | M5B-004 |
| 4 | Implement `findLineIndex(lineNum)` — linear scan, return index or -1 | M5B-004 |
| 5 | Implement `runProgram()` — set `_pc = 0`, `_running = true`, loop: tokenize current line, execute, advance `_pc`, stop when `_pc >= _lineCount` or `!_running` | M5B-004 |

### 1A.10 Variable storage (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `getNumVar(name)` — map single letter A-Z to `_numVars[0..25]` | M5B-004 |
| 2 | Implement `setNumVar(name, value)` | M5B-004 |
| 3 | Implement `getStrVar(name)` — map single letter to `_strVars[0..25]` | M5B-004 |
| 4 | Implement `setStrVar(name, value)` — copy with length cap at `MAX_STRING_LEN` | M5B-004 |
| 5 | Implement constructor: zero all variables, set `_lineCount = 0`, `_running = false` | M5B-004 |

### 1A.11 Serial I/O implementation (`src/serial_io.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `consolePrint(const char*)` — `Serial.print()` | M5B-005 |
| 2 | Implement `consolePrintLn(const char*)` — `Serial.println()` | M5B-005 |
| 3 | Implement `consolePrintNum(float)` — format and print number (strip trailing zeros for integers) | M5B-005 |
| 4 | Implement `consoleError(const char*)` — print `? ` prefix, set `_running = false` | M5B-005 |
| 5 | Implement `consoleClear()` — print escape codes to clear terminal | M5B-005 |
| 6 | Implement `consoleReadLine(char*, int maxLen)` — read from Serial character by character, handle backspace, echo characters, return on newline | M5B-005 |
| 7 | Implement `consoleReadKey()` — non-blocking `Serial.read()` | M5B-005 |
| 8 | Implement `begin()` — call `M5.begin()`, `Serial.begin(115200)`, print startup banner | M5B-005 |

### 1A.12 REPL loop (`src/repl.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `replLoop()` — print `> ` prompt, read line, dispatch | M5B-005 |
| 2 | Implement line number detection — if line starts with digit, parse line number and text | M5B-005 |
| 3 | If line number with text: call `addLine(lineNum, text)` | M5B-005 |
| 4 | If line number alone: call `deleteLine(lineNum)` | M5B-005 |
| 5 | If no line number: tokenize and execute as direct command | M5B-005 |
| 6 | Implement `RUN` command — call `runProgram()` | M5B-005 |
| 7 | Implement `NEW` command — clear all program lines and variables | M5B-005 |
| 8 | Implement `LIST [from[-to]]` command — display program lines with optional range | M5B-005 |
| 9 | Implement `HELP` command — print available commands for current build phase | M5B-005 |
| 10 | Implement `loop()` — call `replLoop()` | M5B-005 |

---

## Phase 1B — Control Flow and INPUT

**Goal:** Interactive programs with loops, conditionals, subroutines, and user input. Multiple statements per line.

**Deliverable:** Write and run a number guessing game with WHILE loop, IF/THEN, and INPUT. `FOR I = 1 TO 10 : PRINT I : NEXT I` works.

### 1B.1 INPUT statement, CLR, and colon separator (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `execInput()` — optional prompt string, read line from I/O layer, convert to number or store as string | M5B-006 |
| 2 | Implement `CLR` — zero all numeric variables, empty all string variables, free arrays | M5B-006 |
| 3 | Implement colon statement separator — after executing one statement, if next token is `:`, advance and execute next statement | M5B-006 |

### 1B.2 Branching (`src/phase2_control.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `IF expr THEN statement [ELSE statement]` — evaluate expression, execute THEN or ELSE clause | M5B-007 |
| 2 | Implement `IF expr THEN linenum` — conditional GOTO | M5B-007 |
| 3 | Implement `GOTO linenum` — find line by number, set `_pc` to its index | M5B-007 |

### 1B.3 Loops (`src/phase2_control.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define `ForEntry` struct: variable index, limit, step, line index of FOR statement | M5B-007 |
| 2 | Implement `FOR var = start TO limit [STEP inc]` — push `ForEntry` onto `_forStack`, error on overflow | M5B-007 |
| 3 | Implement `NEXT [var]` — increment variable, test against limit (respecting step sign), loop back or pop and continue | M5B-007 |
| 4 | Define `WhileEntry` struct: line index of WHILE statement | M5B-007 |
| 5 | Implement `WHILE expr` — evaluate expression, if false skip to matching WEND, else push entry | M5B-007 |
| 6 | Implement `WEND` — jump back to matching WHILE for re-evaluation | M5B-007 |

### 1B.4 Subroutines (`src/phase2_control.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define `GosubEntry` struct: return line index | M5B-007 |
| 2 | Implement `GOSUB linenum` — push return address onto `_gosubStack`, jump to target, error on overflow | M5B-007 |
| 3 | Implement `RETURN` — pop `_gosubStack`, set `_pc` to return address, error on underflow | M5B-007 |

---

## Phase 1C — Functions, Strings, and Arrays

**Goal:** Full BASIC language with math/string built-in functions, string expressions, and dimensioned arrays.

**Deliverable:** `PRINT LEFT$("Hello", 3)` outputs `Hel`. `DIM A(10) : FOR I = 0 TO 10 : A(I) = I * I : NEXT I` works.

### 1C.1 Math functions (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Add math functions to `evalAtom()`: `ABS()`, `INT()`, `SQR()`, `RND()` | M5B-008 |
| 2 | `RND(n)` — if n > 0: random integer 0 to n-1; if n <= 0: random float 0.0 to 1.0 | M5B-008 |

### 1C.2 String expression evaluator (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `evalStrExpr()` — handles string literal, string variable (A$-Z$), string concatenation with `+` | M5B-008 |
| 2 | Implement `STR$(n)` — number to string conversion | M5B-008 |
| 3 | Implement `CHR$(n)` — ASCII code to character | M5B-008 |
| 4 | Implement `LEFT$(s$, n)`, `RIGHT$(s$, n)`, `MID$(s$, pos, len)` | M5B-008 |
| 5 | Implement `UPPER$(s$)`, `LOWER$(s$)` | M5B-008 |
| 6 | Implement `INKEY$` — last key pressed (serial polling) | M5B-008 |
| 7 | Implement numeric string functions in `evalAtom()`: `LEN(s$)`, `VAL(s$)`, `ASC(s$)`, `INSTR(s$, find$)` | M5B-008 |

### 1C.3 String functions integration (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Integrate string function tokens into expression evaluator `evalStrExpr()` dispatch | M5B-009 |
| 2 | Integrate numeric string function tokens into `evalAtom()` dispatch | M5B-009 |
| 3 | Verify string concatenation truncates at `MAX_STRING_LEN` (63 chars) without buffer overflow | M5B-009 |

### 1C.4 Arrays (`src/phase4_5_arrays_files.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define `ArrayDef` struct: name (char), dimensions (1 or 2), sizes, heap-allocated float array | M5B-010 |
| 2 | Implement `DIM var(size)` — allocate 1D array, initialize to 0, add to `_arrays[]`, error on overflow or redefinition | M5B-010 |
| 3 | Implement `DIM var(rows, cols)` — allocate 2D array, same checks | M5B-010 |
| 4 | Implement array access in `evalAtom()` — detect `IDENT(` pattern, evaluate index expression(s), bounds check, return value | M5B-010 |
| 5 | Implement array assignment in `execLet()` — detect `IDENT(` pattern on left side, evaluate index, store value | M5B-010 |
| 6 | Implement `CLR` array cleanup — free all heap-allocated arrays | M5B-010 |

---

## Phase 1D — Tools, Examples, and Delivery

**Goal:** Complete toolchain for embedding programs in firmware and uploading over serial without reflashing. Verified on hardware.

**Deliverable:** `python tools/bas2h.py programs/guess_game.bas && pio run -e phase0 -t upload` — game runs on boot. `python tools/m5basic_upload.py upload programs/demo.bas --run` — uploads and runs without reflashing.

### 1D.1 bas2h.py tool (`tools/bas2h.py`)

| # | Task | Issue |
|---|------|-------|
| 1 | Parse `.bas` file: extract line number and text from each line | M5B-011 |
| 2 | Generate C header with `EmbeddedLine` struct array: `{lineNum, "text"}` per line, null terminator | M5B-011 |
| 3 | Handle string escaping: double quotes in BASIC source → escaped in C string | M5B-011 |
| 4 | Support `-o` flag for output path (default: `include/embedded_program.h`) | M5B-011 |
| 5 | Support `--name` flag for array name (default: `EMBEDDED_PROGRAM`) | M5B-011 |
| 6 | Add argument parsing with `argparse` and usage help | M5B-011 |

### 1D.2 Embedded loader (`src/serial_io.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define `EmbeddedLine` struct: `uint16_t lineNum`, `const char* text` | M5B-011 |
| 2 | Conditionally include `embedded_program.h` when `M5BASIC_EMBEDDED_PROGRAM` defined | M5B-011 |
| 3 | Implement `loadEmbeddedProgram()` — iterate `EMBEDDED_PROGRAM[]` array, call `addLine()` for each entry | M5B-011 |
| 4 | Call `loadEmbeddedProgram()` then `runProgram()` in `begin()` when `M5BASIC_EMBEDDED_PROGRAM` defined | M5B-011 |

### 1D.3 Serial transfer protocol handler (`src/serial_io.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `checkSerial()` — peek at serial buffer, detect `>>>XFER`, `>>>FETCH`, `>>>LIST` markers | M5B-012 |
| 2 | Implement `serialReceiveProgram()` — respond `<<<READY`, receive numbered lines until `>>>END`, respond `<<<OK N` with line count | M5B-012 |
| 3 | Implement `serialSendProgram()` — send each program line as `<<<LINE linenum text`, end with `<<<EOF` | M5B-012 |
| 4 | Implement `serialListFiles()` — stub in Phase 1 (no SD card), respond `<<<ERR NO SD CARD` or empty `<<<EOF` | M5B-012 |
| 5 | Implement 5-second timeout between lines during upload — terminate on timeout | M5B-012 |
| 6 | Poll `checkSerial()` at top of REPL loop and inside `consoleReadLine()` during input wait | M5B-012 |

### 1D.4 PC upload tool (`tools/m5basic_upload.py`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `upload` command — open serial port, send `>>>XFER filename\n`, wait for `<<<READY`, send lines, send `>>>END`, wait for `<<<OK` | M5B-012 |
| 2 | Implement `download` command — send `>>>FETCH filename\n`, receive `<<<LINE` entries until `<<<EOF`, write to file | M5B-012 |
| 3 | Implement `list` command — send `>>>LIST\n`, receive and display `<<<LINE` entries | M5B-012 |
| 4 | Implement `--save` flag — include filename in `>>>XFER` to save on SD | M5B-012 |
| 5 | Implement `--run` flag — after upload, send `RUN\n` to device | M5B-012 |
| 6 | Implement `--port` flag for serial port selection (auto-detect as default) | M5B-012 |
| 7 | Implement batch upload: `upload *.bas --save` sends multiple files | M5B-012 |
| 8 | Implement `install-examples` command — upload all files from `examples/` | M5B-012 |
| 9 | Add argument parsing with `argparse`, usage help, error handling for serial connection failures | M5B-012 |

### 1D.5 Example programs (`programs/`, `examples/phase0/`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create `programs/test_minimal.bas` — smoke test: PRINT, variables, RND | M5B-013 |
| 2 | Create `programs/demo.bas` — Fibonacci + dice: FOR, INPUT, GOSUB | M5B-013 |
| 3 | Create `programs/guess_game.bas` — interactive game: WHILE, IF, INPUT | M5B-013 |
| 4 | Create `examples/phase0/examples.bas` — 8 serial-only examples: hello, math, fibonacci, guess game, multiplication table, prime checker, menu with subroutines, text adventure | M5B-013 |

### 1D.6 Build verification

| # | Task | Issue |
|---|------|-------|
| 1 | Verify `pio run -e phase0` compiles without errors | M5B-013 |
| 2 | Verify `pio run -e phase0-repl` compiles without errors | M5B-013 |
| 3 | Verify `python tools/bas2h.py programs/guess_game.bas` generates valid header | M5B-013 |
| 4 | Verify `pio run -e phase0 -t upload && pio device monitor` — game runs on boot, REPL prompt appears | M5B-013 |
| 5 | Verify `python tools/m5basic_upload.py upload programs/demo.bas` uploads successfully without reflashing | M5B-013 |
| 6 | Verify all 8 Phase 0 example programs execute correctly via serial REPL | M5B-013 |
