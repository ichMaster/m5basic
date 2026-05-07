# Phase 1 — Issues

## Issues Summary Table

| # | ID | Title | Size | Sub-phase | Dependencies |
|---|---|---|---|---|---|
| 1 | M5B-001 | PlatformIO project scaffold and foundation | S | 1A — Minimal Working Interpreter | -- |
| 2 | M5B-002 | Lexer and token system | M | 1A — Minimal Working Interpreter | M5B-001 |
| 3 | M5B-003 | Basic numeric expression evaluator | S | 1A — Minimal Working Interpreter | M5B-002 |
| 4 | M5B-004 | Core statements and program/variable storage | M | 1A — Minimal Working Interpreter | M5B-003 |
| 5 | M5B-005 | Serial I/O and REPL | M | 1A — Minimal Working Interpreter | M5B-004 |
| 6 | M5B-006 | INPUT statement, CLR, and colon separator | S | 1B — Control Flow and INPUT | M5B-005 |
| 7 | M5B-007 | Control flow (IF/GOTO/FOR/WHILE/GOSUB) | M | 1B — Control Flow and INPUT | M5B-006 |
| 8 | M5B-008 | Math functions and string expression evaluator | M | 1C — Functions, Strings, Arrays | M5B-005 |
| 9 | M5B-009 | String functions integration | S | 1C — Functions, Strings, Arrays | M5B-008 |
| 10 | M5B-010 | Arrays | S | 1C — Functions, Strings, Arrays | M5B-005 |
| 11 | M5B-011 | Embedded program loader and bas2h.py | S | 1D — Tools and Delivery | M5B-005 |
| 12 | M5B-012 | Serial transfer protocol and m5basic_upload.py | M | 1D — Tools and Delivery | M5B-005 |
| 13 | M5B-013 | Example programs and build verification | S | 1D — Tools and Delivery | M5B-011, M5B-012 |

**Size legend:** S = 1–2 days, M = 3–5 days

---

## Sub-phase Summary

| Sub-phase | Focus | Issues | ~Days | Deliverable |
|-----------|-------|--------|-------|-------------|
| **1A** | Minimal Working Interpreter | M5B-001 – M5B-005 | 8–12 | PRINT/LET/REM/END work over serial REPL |
| **1B** | Control Flow and INPUT | M5B-006, M5B-007 | 4–7 | Loops, conditionals, subroutines, user input |
| **1C** | Functions, Strings, and Arrays | M5B-008 – M5B-010 | 5–8 | Full BASIC language |
| **1D** | Tools and Delivery | M5B-011 – M5B-013 | 5–7 | Embedded loader, serial upload, examples |

---

## Dependency Tree

```
Phase 1A                     Phase 1B          Phase 1C          Phase 1D
────────                     ────────          ────────          ────────

M5B-001 (scaffold)
    |
M5B-002 (lexer)
    |
M5B-003 (eval-basic)
    |
M5B-004 (stmts+storage)
    |
M5B-005 (serial+REPL)
    |
    +──────────> M5B-006 (INPUT/CLR/colon)
    |                |
    |            M5B-007 (control flow)
    |
    +──────────> M5B-008 (math+strings)
    |                |
    |            M5B-009 (string integration)
    |
    +──────────> M5B-010 (arrays)
    |
    +──────────> M5B-011 (embedded loader)
    |                |
    +──────────> M5B-012 (transfer protocol)
                     |
                 M5B-013 (examples+verify)
```

**Parallelization hints:**

- Phases 1B, 1C, and 1D can all start in parallel after 1A is complete
- Within 1C: M5B-008/009 and M5B-010 can run in parallel
- Within 1D: M5B-011 and M5B-012 can run in parallel

---

## Phase 1A — Minimal Working Interpreter

### M5B-001 — PlatformIO project scaffold and foundation

**Description:**
Set up the PlatformIO project skeleton with all build environments, configuration header, interpreter class declaration, and entry point. This is the base that all other issues build on.

**What needs to be done:**
- Create `platformio.ini` with 5 build environments (`phase0`, `phase0-repl`, `phase1`, `phase2`, `full`), each with appropriate `build_flags` and `build_src_filter`
- Set platform `espressif32@6.5.0`, board `m5stack-core-esp32`, framework `arduino`
- Add library dependency `M5Stack@0.4.6`, optional `LoRa`
- Create directory structure: `include/`, `src/`, `programs/`, `examples/phase0/` through `examples/phase7/`, `tools/`
- Create `include/config.h` with all constants: program limits (`MAX_LINES=500`, `MAX_LINE_LEN=128`, `MAX_STRING_LEN=63`), array limits, stack limits, serial settings (`115200`), display constants (`53x30`), protocol markers, hardware pins
- Create `include/interpreter.h` with `BasicInterpreter` class declaration: all public methods, private state (program array, variables, control flow stacks), phase-guarded method stubs
- Create `src/main.cpp` entry point: global `BasicInterpreter`, `setup()` calls `begin()`, `loop()` calls `loop()`
- Create empty stub `.cpp` files for all source modules so all environments compile

**Dependencies:** None

**Expected result:**
All 5 PlatformIO environments compile successfully (with stub implementations). Directory structure matches the architecture specification.

**Acceptance criteria:**
- [ ] `pio run -e phase0` compiles without errors
- [ ] `pio run -e phase0-repl` compiles without errors
- [ ] `pio run -e full` compiles without errors
- [ ] `include/config.h` defines all constants from the architecture spec
- [ ] `include/interpreter.h` declares the complete `BasicInterpreter` class
- [ ] `include/tokens.h` exists as a stub header
- [ ] All source files listed in the architecture spec exist (even if empty stubs)

---

### M5B-002 — Lexer and token system

**Description:**
Implement the single-pass tokenizer that converts BASIC source text into a token stream. The lexer is consumed by the expression evaluator and statement executor. Includes the full keyword table (~80 entries) and the `Lexer` class.

**What needs to be done:**

**Token definitions (`include/tokens.h`):**
- Define `TokenType` enum: all operators, keywords (~80), built-in functions, special tokens (`TOK_EOF`, `TOK_NUMBER`, `TOK_STRING`, `TOK_IDENT`)
- Define `Token` struct: `type`, `numVal`, `strVal[MAX_STRING_LEN+1]`, `ident[16]`
- Define `Keyword` struct and static keyword lookup table mapping strings to token types (case-insensitive)
- Declare `Lexer` class: `init(const char* line)`, `nextToken()`, `peekToken()`, `currentToken()`

**Tokenizer (`src/lexer.cpp`):**
- Implement number tokenization: digit sequences → `atof()` → `TOK_NUMBER`
- Implement string literal tokenization: `"`-delimited, escape sequences (`\"`, `\\`, `\n`, `\t`)
- Implement identifier/keyword tokenization: letter start → accumulate → case-insensitive keyword table lookup → keyword token or `TOK_IDENT`
- Implement operator tokenization: single-char and double-char (`<=`, `>=`, `<>`, `!=`)
- Implement `nextToken()` main dispatch, `peekToken()` without advancing, `currentToken()`

**Dependencies:** M5B-001

**Expected result:**
Any BASIC source line can be tokenized into a correct stream of tokens. Keywords are case-insensitive. The lexer handles all edge cases: empty lines, multiple operators, string escapes.

**Acceptance criteria:**
- [ ] `PRINT "Hello"` tokenizes to `[TOK_PRINT, TOK_STRING("Hello")]`
- [ ] `A = 3.14 + B` tokenizes to `[TOK_IDENT(A), TOK_EQ, TOK_NUMBER(3.14), TOK_PLUS, TOK_IDENT(B)]`
- [ ] `IF X >= 10 THEN GOTO 100` tokenizes correctly with `TOK_GTE`
- [ ] Case insensitive: `print`, `Print`, `PRINT` all produce `TOK_PRINT`
- [ ] String escapes: `"He said \"hi\""` produces `TOK_STRING(He said "hi")`
- [ ] Keyword table contains ~80 entries covering all phases
- [ ] `peekToken()` returns next token without advancing

---

### M5B-003 — Basic numeric expression evaluator

**Description:**
Implement the recursive descent parser for numeric expressions — the core computation engine. This phase covers arithmetic, comparisons, logical operators, variables, and parentheses. Math functions and string functions are added in Phase 1C (M5B-008).

**What needs to be done:**

**Numeric expression evaluator (`src/interpreter_core.cpp`):**
- `evalExpr()` — entry point, delegates to `evalLogic()`
- `evalLogic()` — `AND`, `OR` operators
- `evalComparison()` — `=`, `<>`, `<`, `>`, `<=`, `>=` (return 1.0 or 0.0)
- `evalAddSub()` — `+`, `-`
- `evalMulDiv()` — `*`, `/`, `MOD` (with division-by-zero check)
- `evalUnary()` — `NOT`, unary `-`
- `evalAtom()` — numeric literal, numeric variable (A-Z), parenthesized expression
- Stub `evalStrExpr()` — returns empty string (implemented in M5B-008)

**Dependencies:** M5B-002

**Expected result:**
Expressions like `(A + 3) * 2`, `X > 0 AND Y < 100`, `NOT 0` evaluate correctly according to precedence rules.

**Acceptance criteria:**
- [ ] `3 + 4 * 2` evaluates to 11 (not 14) — precedence correct
- [ ] `NOT 0` evaluates to 1, `NOT 5` evaluates to 0
- [ ] `10 MOD 3` evaluates to 1
- [ ] Division by zero calls `consoleError()` and stops execution
- [ ] `(2 + 3) * 4` evaluates to 20
- [ ] `A = 5`, then `A + 1` evaluates to 6
- [ ] Unary minus: `-3 + 5` evaluates to 2

---

### M5B-004 — Core statements and program/variable storage

**Description:**
Implement the statement executor (switch dispatch), four core statements (PRINT, LET, REM, END), program storage (sorted array of 500 lines), variable storage (26 numeric + 26 string), and the program runner. INPUT, CLR, and colon separator are deferred to Phase 1B (M5B-006).

**What needs to be done:**

**Statement executor (`src/interpreter_core.cpp`):**
- `execStatement()` — switch on current token type, delegate to handler
- `execPrint()` — evaluate and print numeric/string expressions, handle `;` (no space), `,` (tab), trailing `;` (no newline), bare PRINT (blank line)
- `execLet()` — optional LET keyword, detect variable type (numeric/string), evaluate RHS, assign
- `execRem()` — skip to end of line
- `END` — set `_running = false`

**Program storage:**
- `BasicLine` struct: `uint16_t lineNum`, `char text[MAX_LINE_LEN+1]`
- `addLine()` — sorted insert into `_program[]` array, replace if line number exists
- `deleteLine()` — remove and shift, decrement `_lineCount`
- `findLineIndex()` — linear scan by line number
- `runProgram()` — reset stacks, set `_pc = 0`, `_running = true`, loop: tokenize `_program[_pc].text`, execute, advance `_pc`

**Variable storage:**
- `float _numVars[26]` — A-Z mapped to indices 0-25
- `char _strVars[26][MAX_STRING_LEN+1]` — A$-Z$
- Get/set methods for both types

**Dependencies:** M5B-003

**Expected result:**
Programs can be stored line by line, run in order, and execute PRINT/LET statements. Variable state persists across statements.

**Acceptance criteria:**
- [ ] `10 PRINT "Hello"` / `20 END` / `RUN` prints "Hello" and returns to prompt
- [ ] `PRINT 2+3; " "; "hi"` outputs `5 hi` (no newline with trailing `;`)
- [ ] `PRINT 1, 2, 3` outputs `1    2    3` (tab separation)
- [ ] `A = 5 : PRINT A` — colon not yet supported, should handle gracefully
- [ ] Adding line 15 between existing lines 10 and 20 inserts correctly
- [ ] Typing `15` alone deletes line 15
- [ ] `_program[]` stays sorted by line number after all operations
- [ ] `NEW` clears all lines and variables

---

### M5B-005 — Serial I/O and REPL

**Description:**
Implement the serial-mode I/O layer (all console methods using `Serial.print`/`Serial.read`) and the REPL loop that accepts user input, stores program lines, and dispatches direct commands. This completes Phase 1A — the interpreter becomes interactive on hardware.

**What needs to be done:**

**Serial I/O (`src/serial_io.cpp`):**
- `consolePrint(const char*)` — `Serial.print()`
- `consolePrintLn(const char*)` — `Serial.println()`
- `consolePrintNum(float)` — print number, strip trailing zeros for integer values
- `consoleError(const char*)` — print `? ` + message, set `_running = false`
- `consoleClear()` — send ANSI clear escape codes
- `consoleReadLine(buf, maxLen)` — read from Serial char by char, echo back, handle backspace, return on `\n` or `\r`
- `consoleReadKey()` — non-blocking `Serial.read()`, return -1 if no data
- `begin()` — `M5.begin()`, `Serial.begin(115200)`, print startup banner with version and phase info

**REPL loop (`src/repl.cpp`):**
- `replLoop()` — print `> ` prompt, call `consoleReadLine()`, dispatch:
  - Line starts with digit → parse line number + text → `addLine()` or `deleteLine()`
  - Otherwise → tokenize and execute as direct command
- `RUN` — call `runProgram()`, print "Ready." after completion
- `NEW` — call `newProgram()` (clear all lines, variables, arrays)
- `LIST [from[-to]]` — display lines in range
- `HELP` — print command list for current build phase
- `loop()` — call `replLoop()`

**Dependencies:** M5B-004

**Expected result:**
Connect via serial terminal, type BASIC commands interactively, store and run programs. The full interactive development experience works over USB serial.

**Acceptance criteria:**
- [ ] On boot, serial terminal shows banner: `M5Basic vX.X` and `> ` prompt
- [ ] Typing `PRINT 2+2` at prompt outputs `4`
- [ ] Typing `10 PRINT "Hello"` stores the line (no output)
- [ ] `LIST` shows stored lines with line numbers
- [ ] `RUN` executes stored program
- [ ] `NEW` clears program and shows `> ` prompt
- [ ] Typing `10` alone deletes line 10
- [ ] `HELP` prints available commands
- [ ] Backspace works during line editing
- [ ] Numbers print without trailing zeros: `PRINT 5` shows `5` not `5.000000`
- [ ] Error messages start with `? ` prefix
- [ ] After program ends (END or last line), `Ready.` is printed and prompt returns

---

## Phase 1B — Control Flow and INPUT

### M5B-006 — INPUT statement, CLR, and colon separator

**Description:**
Add the INPUT statement for user interaction, CLR for variable reset, and the colon separator for multiple statements per line. These complete the core statement set needed for interactive programs.

**What needs to be done:**

**Statements (`src/interpreter_core.cpp`):**
- `execInput()` — optional `"prompt";` display, read line from I/O, convert to number or store as string
- `CLR` — zero all numeric variables, empty all string variables, free arrays
- Colon separator — after executing one statement, if next token is `:`, advance and execute next

**Dependencies:** M5B-005

**Expected result:**
Users can write programs that accept input, use multiple statements per line, and reset variables.

**Acceptance criteria:**
- [ ] `INPUT "Name: "; N$` displays prompt and stores input in N$
- [ ] `INPUT A` reads a number from serial
- [ ] `A = 5 : B = A * 2 : PRINT B` outputs `10` (colon separator)
- [ ] `CLR` zeros all variables — `PRINT A` shows `0` after CLR
- [ ] Multiple colons: `A = 1 : B = 2 : C = 3 : PRINT A + B + C` outputs `6`

---

### M5B-007 — Control flow (IF/GOTO/FOR/WHILE/GOSUB)

**Description:**
Implement IF/THEN/ELSE, GOTO, FOR/NEXT, WHILE/WEND, and GOSUB/RETURN. Compiled under `M5BASIC_PHASE2` flag but included from Phase 0 because the language is too limited without it. Control flow uses stack-based tracking for loops and subroutines.

**What needs to be done:**

**Branching (`src/phase2_control.cpp`):**
- `IF expr THEN statement [ELSE statement]` — evaluate expression, execute appropriate clause
- `IF expr THEN linenum` — conditional GOTO (syntactic sugar)
- `GOTO linenum` — find line by `findLineIndex()`, set `_pc`, error if line not found

**FOR/NEXT loops:**
- `ForEntry` struct: variable index, limit, step, line index
- `FOR var = start TO limit [STEP inc]` — push entry, default step = 1
- `NEXT [var]` — increment, compare with limit (positive step: var > limit → exit; negative step: var < limit → exit), loop back or pop
- Stack overflow error at depth 10

**WHILE/WEND loops:**
- `WhileEntry` struct: line index of WHILE
- `WHILE expr` — if expression false, scan forward to matching WEND and skip; if true, push entry and execute body
- `WEND` — pop entry, jump back to WHILE line for re-evaluation
- Handle nested WHILE/WEND correctly when scanning forward

**Subroutines:**
- `GosubEntry` struct: return line index
- `GOSUB linenum` — push current `_pc` + 1 onto stack, jump to target
- `RETURN` — pop return address, set `_pc`
- Stack overflow/underflow errors

**Dependencies:** M5B-006

**Expected result:**
Programs with loops, conditionals, and subroutines execute correctly. Nesting works. Stack overflow is reported cleanly. This completes Phase 1B — interactive programs with full control flow.

**Acceptance criteria:**
- [ ] `IF 1 THEN PRINT "yes" ELSE PRINT "no"` prints "yes"
- [ ] `IF 0 THEN PRINT "yes" ELSE PRINT "no"` prints "no"
- [ ] `FOR I = 1 TO 5 : PRINT I; " "; : NEXT I` prints `1 2 3 4 5`
- [ ] `FOR I = 10 TO 0 STEP -2` counts down correctly
- [ ] Nested FOR loops work (at least 3 deep)
- [ ] `WHILE X < 10 : X = X + 1 : WEND` loops 10 times
- [ ] `GOSUB 100` / `RETURN` correctly resumes after GOSUB
- [ ] `GOTO` to non-existent line prints `? LINE N NOT FOUND`
- [ ] 11 nested GOSUBs prints `? GOSUB STACK OVERFLOW`
- [ ] `RETURN` without GOSUB prints `? RETURN WITHOUT GOSUB`
- [ ] `NEXT` without FOR prints `? NEXT WITHOUT FOR`

---

## Phase 1C — Functions, Strings, and Arrays

### M5B-008 — Math functions and string expression evaluator

**Description:**
Add math built-in functions to the numeric evaluator and implement the full string expression evaluator with all string functions. This extends the expression engine from basic arithmetic (Phase 1A) to the complete BASIC function set.

**What needs to be done:**

**Math functions (`src/interpreter_core.cpp`):**
- Add to `evalAtom()`: `ABS()`, `INT()`, `SQR()`, `RND()`
- `RND(n)` — if n > 0: random integer 0 to n-1; if n <= 0: random float 0.0 to 1.0

**String expression evaluator (`src/interpreter_core.cpp`):**
- `evalStrExpr()` — string literal, string variable (A$-Z$), string concatenation (`+`), string functions
- `STR$(n)`, `CHR$(n)`, `LEFT$(s$, n)`, `RIGHT$(s$, n)`, `MID$(s$, pos [, len])`, `UPPER$(s$)`, `LOWER$(s$)`, `INKEY$`
- Numeric string functions in `evalAtom()`: `LEN(s$)`, `VAL(s$)`, `ASC(s$)`, `INSTR(s$, find$)`
- String concatenation truncates at `MAX_STRING_LEN` (63) without buffer overflow

**Dependencies:** M5B-005

**Expected result:**
Expressions like `ABS(-5)`, `RND(6)`, `LEFT$("Hello", 3)`, `LEN(A$)` all evaluate correctly.

**Acceptance criteria:**
- [ ] `ABS(-5)` returns 5, `INT(3.7)` returns 3, `SQR(9)` returns 3
- [ ] `RND(6)` returns values in range 0–5
- [ ] `LEFT$("Hello", 3)` returns `"Hel"`
- [ ] `MID$("Hello", 2, 3)` returns `"ell"` (1-based position)
- [ ] `LEN("test")` returns 4
- [ ] `"Hello" + " " + "World"` returns `"Hello World"`
- [ ] String concatenation exceeding 63 chars truncates without crash
- [ ] `STR$(42)` returns `"42"`, `VAL("3.14")` returns 3.14
- [ ] `CHR$(65)` returns `"A"`, `ASC("A")` returns 65
- [ ] `UPPER$("hello")` returns `"HELLO"`, `LOWER$("HELLO")` returns `"hello"`
- [ ] `INSTR("Hello World", "World")` returns 7

---

### M5B-009 — String functions integration

**Description:**
Wire the string function implementations (from M5B-008) into the statement executor and verify they work end-to-end in programs — in PRINT, LET, IF comparisons, and INPUT.

**What needs to be done:**
- Ensure all string function tokens dispatch correctly from `evalStrExpr()` and `evalAtom()`
- Verify `PRINT LEFT$("Hello", 3)` works from REPL and from stored programs
- Verify `A$ = UPPER$(B$)` assignment works
- Verify `INSTR("Hello World", "World")` returns 7 (1-based)
- Verify string concatenation truncation at 63 chars without crash
- Verify `MID$` with 2 args (no length) returns rest of string

**Dependencies:** M5B-008

**Expected result:**
All 11 string functions and string concatenation work correctly in expressions, PRINT statements, and assignments.

**Acceptance criteria:**
- [ ] `PRINT LEN("Hello")` outputs `5`
- [ ] `PRINT LEFT$("Hello", 3)` outputs `Hel`
- [ ] `PRINT RIGHT$("Hello", 3)` outputs `llo`
- [ ] `PRINT MID$("Hello", 2, 3)` outputs `ell`
- [ ] `PRINT MID$("Hello", 4)` outputs `lo`
- [ ] `PRINT STR$(42)` outputs `42`
- [ ] `PRINT VAL("3.14")` outputs `3.14`
- [ ] `PRINT CHR$(65)` outputs `A`
- [ ] `PRINT ASC("A")` outputs `65`
- [ ] `PRINT UPPER$("hello")` outputs `HELLO`
- [ ] `PRINT LOWER$("HELLO")` outputs `hello`
- [ ] `PRINT INSTR("Hello World", "World")` outputs `7`
- [ ] `PRINT INSTR("Hello", "xyz")` outputs `0`

---

### M5B-010 — Arrays

**Description:**
Implement the DIM statement for 1D and 2D numeric arrays, array access in expressions, and array assignment. Arrays are heap-allocated and tracked in a fixed-size registry. This completes Phase 1C — the full BASIC language.

**What needs to be done:**

**Array system (`src/phase4_5_arrays_files.cpp`):**
- `ArrayDef` struct: name (single char), dimension count (1 or 2), size per dimension, `float*` data pointer
- `DIM var(size)` — allocate `(size+1)` floats (0 to size inclusive), zero-fill, register in `_arrays[]`
- `DIM var(rows, cols)` — allocate `(rows+1) * (cols+1)` floats, zero-fill
- Array access in `evalAtom()` — when `TOK_IDENT` followed by `(`, look up array, evaluate 1 or 2 index expressions, bounds check, return `data[index]`
- Array assignment in `execLet()` — detect array pattern on LHS, evaluate indices, store value
- Errors: `ARRAY ALREADY DEFINED`, `INVALID ARRAY SIZE`, `TOO MANY ARRAYS` (>10), `OUT OF MEMORY` (malloc fail), array index out of bounds
- `CLR` frees all arrays and resets registry

**Dependencies:** M5B-005

**Expected result:**
Programs can dimension, write to, and read from 1D and 2D arrays. Arrays are independent of scalar variables with the same name.

**Acceptance criteria:**
- [ ] `DIM A(10) : A(5) = 42 : PRINT A(5)` outputs `42`
- [ ] `DIM M(3, 4) : M(1, 2) = 99 : PRINT M(1, 2)` outputs `99`
- [ ] `DIM A(10)` creates 11 elements (0 through 10 inclusive)
- [ ] `A = 5 : DIM A(10) : PRINT A` outputs `5` (scalar and array are separate)
- [ ] Accessing undimensioned array prints error
- [ ] `DIM A(10) : DIM A(5)` prints `? ARRAY ALREADY DEFINED`
- [ ] 11th `DIM` call prints `? TOO MANY ARRAYS`
- [ ] `A(101)` on a `DIM A(100)` prints index out of bounds error
- [ ] `CLR` frees all arrays — `DIM A(10)` works again after CLR

---

## Phase 1D — Tools, Examples, and Delivery

### M5B-011 — Embedded program loader and bas2h.py

**Description:**
Implement the Python tool that converts `.bas` files to C headers for firmware embedding, and the device-side loader that reads the embedded program into interpreter memory at boot.

**What needs to be done:**

**`tools/bas2h.py`:**
- Parse `.bas` file line by line: extract line number (digits at start) and remaining text
- Generate `include/embedded_program.h` with `EmbeddedLine` struct array
- Escape double quotes in BASIC strings for C: `"` → `\"`
- Handle `argparse` CLI: positional `.bas` path, `-o` output path, `--name` array name
- Skip blank lines and comment-only lines in input

**Embedded loader (`src/serial_io.cpp`):**
- Define `EmbeddedLine` struct: `uint16_t lineNum`, `const char* text`
- When `M5BASIC_EMBEDDED_PROGRAM` defined: `#include "embedded_program.h"`
- `loadEmbeddedProgram()` — iterate `EMBEDDED_PROGRAM[]` until null terminator, call `addLine()` for each
- In `begin()` when `M5BASIC_EMBEDDED_PROGRAM` defined: call `loadEmbeddedProgram()`, then `runProgram()`
- After embedded program finishes, drop into REPL

**Dependencies:** M5B-005

**Expected result:**
`python tools/bas2h.py programs/guess_game.bas && pio run -e phase0 -t upload && pio device monitor` — game runs immediately on boot, then REPL prompt appears.

**Acceptance criteria:**
- [ ] `python tools/bas2h.py programs/test_minimal.bas` generates valid C header
- [ ] Generated header contains correct line numbers and escaped strings
- [ ] `-o path` flag writes to specified output file
- [ ] `--name GAME` flag changes array name to `GAME`
- [ ] `phase0` environment compiles and runs embedded program on boot
- [ ] After embedded program reaches END, REPL prompt `> ` appears
- [ ] `phase0-repl` environment skips embedded program, goes straight to REPL

---

### M5B-012 — Serial transfer protocol and m5basic_upload.py

**Description:**
Implement the bidirectional serial transfer protocol that allows uploading and downloading BASIC programs between a PC and M5Stack over USB without reflashing. Includes both the device-side handler and the Python CLI tool.

**What needs to be done:**

**Device-side protocol handler (`src/serial_io.cpp`):**
- `checkSerial()` — non-blocking peek at serial buffer for `>>>XFER`, `>>>FETCH`, `>>>LIST` markers
- `serialReceiveProgram()` — respond `<<<READY`, receive numbered lines until `>>>END`, add to program memory, respond `<<<OK N`
- `serialSendProgram()` — send each stored line as `<<<LINE linenum text\n`, end with `<<<EOF\n`
- `serialListFiles()` — stub in Phase 1 (no SD), respond `<<<EOF` or `<<<ERR NO SD`
- 5-second timeout between lines during upload
- Poll `checkSerial()` in REPL loop and during `consoleReadLine()` wait

**PC-side tool (`tools/m5basic_upload.py`):**
- Serial port auto-detection (scan common ports) and `--port` flag
- `upload` command: send `>>>XFER [filename]\n`, wait `<<<READY`, send lines, send `>>>END\n`, wait `<<<OK`
- `download` command: send `>>>FETCH [filename]\n`, receive `<<<LINE` entries until `<<<EOF`, write to file
- `list` command: send `>>>LIST\n`, display `<<<LINE` entries
- `--save` flag: include filename so device saves to SD (when available)
- `--run` flag: after upload, send `RUN\n` to start program
- Batch upload: `upload *.bas --save`
- `install-examples` command: upload all files from `examples/`
- `argparse` CLI with usage help
- Error handling: connection timeout, `<<<ERR` responses, serial port not found

**Dependencies:** M5B-005

**Expected result:**
`python tools/m5basic_upload.py upload programs/demo.bas --run` sends a program over USB and it immediately starts executing on the M5Stack without reflashing.

**Acceptance criteria:**
- [ ] Upload: `m5basic_upload.py upload test.bas` sends program, device responds `<<<OK N`
- [ ] Download: `m5basic_upload.py download backup.bas` saves current program to PC file
- [ ] List: `m5basic_upload.py list` shows files (or "no SD" in Phase 1)
- [ ] `--run` flag makes program execute immediately after upload
- [ ] `--port /dev/ttyUSB0` flag overrides auto-detection
- [ ] Upload during REPL idle works (no need to type UPLOAD first)
- [ ] Upload during `INPUT` wait works (protocol detected inside `consoleReadLine()`)
- [ ] Timeout after 5 seconds of no data during upload terminates cleanly
- [ ] Batch upload of 3+ files works
- [ ] `<<<ERR` responses are displayed as errors on PC side

---

### M5B-013 — Example programs and build verification

**Description:**
Create the BASIC example programs for Phase 0 and verify the complete build-flash-run workflow end-to-end. This completes Phase 1 — every feature is verified on hardware.

**What needs to be done:**

**Example programs:**
- `programs/test_minimal.bas` — smoke test: PRINT, LET, RND, basic expressions
- `programs/demo.bas` — Fibonacci + dice: FOR/NEXT, INPUT, GOSUB/RETURN
- `programs/guess_game.bas` — interactive number guessing: WHILE/WEND, IF/THEN, INPUT, RND
- `examples/phase0/examples.bas` — 8 examples in one file (separated by comments):
  1. Hello World
  2. Input and math
  3. Fibonacci sequence
  4. Guess the number game
  5. Multiplication table
  6. Prime checker
  7. Menu with subroutines
  8. Text adventure

**Build verification:**
- `pio run -e phase0` compiles cleanly
- `pio run -e phase0-repl` compiles cleanly
- `pio run -e phase1` compiles cleanly (display stubs)
- `pio run -e phase2` compiles cleanly (full stubs)
- `pio run -e full` compiles cleanly (all stubs)
- `python tools/bas2h.py programs/guess_game.bas` generates valid header
- Flash `phase0` and verify game runs on boot via serial monitor
- Flash `phase0-repl` and verify REPL prompt appears
- Upload demo.bas via `m5basic_upload.py` and verify execution
- Run all 8 Phase 0 examples through the REPL

**Dependencies:** M5B-011, M5B-012

**Expected result:**
A complete, working Phase 1 interpreter: programs run on boot or via serial upload, REPL is interactive, all control flow and string functions work, examples demonstrate the language capabilities.

**Acceptance criteria:**
- [ ] All 5 PlatformIO environments compile without errors
- [ ] `programs/guess_game.bas` runs correctly on hardware via embedded loader
- [ ] `programs/demo.bas` uploads and runs correctly via serial transfer
- [ ] All 8 Phase 0 examples produce expected output
- [ ] FOR/NEXT, WHILE/WEND, IF/THEN/ELSE, GOSUB/RETURN all work in example programs
- [ ] String functions work in examples (if used)
- [ ] DIM/array access works in examples (if used)
- [ ] No crashes or memory leaks during extended REPL sessions

---

## Phase 1 scope notes

**Total effort:** ~3–4 weeks for a single developer.

**Critical path:** M5B-001 → M5B-002 → M5B-003 → M5B-004 → M5B-005 → M5B-006 → M5B-007

**Sub-phase execution order:**
1. **Phase 1A** (8–12 days) — scaffold, lexer, basic expressions, core statements, serial I/O, REPL. **Working interpreter on hardware.**
2. **Phase 1B** (4–7 days) — INPUT, colon separator, IF/GOTO/FOR/WHILE/GOSUB. **Interactive programs with control flow.**
3. **Phase 1C** (5–8 days) — math functions, string evaluator, string functions, arrays. **Full BASIC language.**
4. **Phase 1D** (5–7 days) — bas2h.py, embedded loader, serial transfer, upload tool, examples. **Complete delivery pipeline.**

**Parallel tracks after Phase 1A:**
- 1B, 1C, and 1D can all start after 1A is complete
- Within 1C: M5B-008/009 and M5B-010 can run in parallel
- Within 1D: M5B-011 and M5B-012 can run in parallel, M5B-013 follows both

**Companion documents:**
- `phase-1-tasks.md` — detailed task tables per sub-phase
- `m5basic-architecture.md` — system architecture and file structure
- `m5basic-lang-ref.md` — complete BASIC language reference
- `m5basic-roadmap.md` — phase overview and feature list
