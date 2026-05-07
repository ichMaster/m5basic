# M5Basic Language Reference

Complete reference for M5Basic after all phases are implemented.
Target platform: M5Stack Core Basic (ESP32) + CardKB + LoRa Module.

---

## Program structure

A program is a sequence of numbered lines stored in memory. Lines execute in ascending numeric order unless redirected by GOTO, GOSUB, IF, FOR, or WHILE.

```basic
10 PRINT "Line ten"
20 PRINT "Line twenty"
30 END
```

**Line numbers:** 1 to 65535. Lines are always kept sorted by number.

**Multiple statements** on one line are separated by colon:
```basic
10 A = 5 : B = 10 : PRINT A + B
```

**Direct mode:** A statement typed without a line number executes immediately and is not stored:
```
> PRINT 2 + 2
4
> A = 42
> PRINT A
42
```

**Editing:**
- Type a line number followed by text to add or replace that line
- Type a line number alone to delete that line
- NEW erases the entire program
- LIST shows the program

---

## Data types

### Numbers

All numbers are IEEE 754 single-precision floating point. Integer arithmetic is exact up to 16777216. There is no separate integer type.

```basic
A = 42
B = 3.14159
C = -0.001
D = 1E6
```

### Strings

Strings hold up to 63 characters. String variable names end with `$`. String literals are enclosed in double quotes.

```basic
A$ = "Hello"
B$ = "World"
C$ = A$ + ", " + B$ + "!"
```

**Escape sequences in string literals:**
| Sequence | Character |
|----------|-----------|
| `\"` | Double quote |
| `\\` | Backslash |
| `\n` | Newline |
| `\t` | Tab |

### Arrays

Arrays are created with DIM before use. Indices run from 0 to the declared size inclusive (so `DIM A(10)` creates 11 elements: A(0) through A(10)).

```basic
DIM A(10)           REM 1D array, indices 0..10
DIM B(5, 5)         REM 2D array, indices 0..5 x 0..5
A(3) = 42
B(1, 2) = 99
```

Maximum 10 arrays. Maximum 100 elements per dimension.

---

## Variables

**Numeric variables:** Single uppercase letter A through Z. 26 total. Initialized to 0.

**String variables:** Single uppercase letter followed by $. A$ through Z$. 26 total. Initialized to empty string.

**Array variables:** Same single-letter names, accessed with parentheses. An array and a scalar can share a name (A and A() are separate).

**Reserved variables** -- set automatically by certain commands:

| Variable | Set by | Meaning |
|----------|--------|---------|
| H | WGET | HTTP response status code |
| E | ERECV | 1 if ESP-NOW data received, 0 otherwise |
| R$ | ERECV | Sender MAC address |
| L | LRECV | 1 if LoRa data received, 0 otherwise |
| R | LRECV | RSSI of last LoRa packet (dBm) |
| W | WSERV | 1 if web request pending, 0 otherwise |

These variables can also be used for other purposes; they are ordinary variables that certain commands happen to write to.

---

## Operators

### Arithmetic

| Operator | Description | Example |
|----------|-------------|---------|
| `+` | Addition (numbers) or concatenation (strings) | `3 + 4` yields 7, `"A" + "B"` yields `"AB"` |
| `-` | Subtraction or unary negation | `10 - 3` yields 7, `-X` |
| `*` | Multiplication | `6 * 7` yields 42 |
| `/` | Division | `22 / 7` yields 3.142857 |
| `MOD` | Integer modulus | `17 MOD 5` yields 2 |

### Comparison

All comparisons return 1 (true) or 0 (false).

| Operator | Description |
|----------|-------------|
| `=` | Equal (also used for assignment) |
| `<>` | Not equal (also `!=`) |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal |
| `>=` | Greater than or equal |

### Logical

| Operator | Description |
|----------|-------------|
| `AND` | True if both sides are non-zero |
| `OR` | True if either side is non-zero |
| `NOT` | True if operand is zero |

### Precedence (highest to lowest)

1. Parentheses `()`
2. `NOT`, unary `-`
3. `*`, `/`, `MOD`
4. `+`, `-`
5. `=`, `<>`, `<`, `>`, `<=`, `>=`
6. `AND`
7. `OR`

---

## Commands

### Program control

**RUN [filename$]**
Execute the program in memory. If a string argument is given, load that file from SD first then run.
```
RUN
RUN "game.bas"
```

**END**
Stop program execution. Returns to REPL prompt.

**NEW**
Erase the program from memory and clear all variables.

**LIST [from[-to]]**
Display program lines. Without arguments, show all lines.
```
LIST
LIST 100
LIST 100-200
```

**CLR**
Clear all variables (numeric set to 0, strings set to empty) without erasing the program.

---

### Output

**PRINT expression [; | , ] [expression ...] [;]**

Print to the active output (serial, and LCD if display is enabled).

- Expressions separated by `;` print with no space between them
- Expressions separated by `,` print with a tab between them
- A trailing `;` suppresses the newline at the end
- Without trailing `;`, a newline is printed after the last expression

```basic
PRINT "Hello"                     Hello
PRINT "A="; 42                    A=42
PRINT 1, 2, 3                    1    2    3
PRINT "No newline";              No newline(cursor stays)
PRINT                            (blank line)
```

### Input

**INPUT ["prompt" ;] variable**

Read a line from the input source (serial terminal, or CardKB when available). If a prompt string is given, it is displayed before waiting for input.

For a numeric variable, the entered text is converted to a number. For a string variable, the text is stored as-is.

```basic
INPUT "Name: "; N$
INPUT "Age: "; A
INPUT X                          (default prompt is "? ")
```

### Assignment

**[LET] variable = expression**

Assign a value to a variable. The keyword LET is optional.

```basic
LET A = 10
B = 20
C$ = "Hello"
A(3) = 99
```

### Comment

**REM text**

Everything after REM to the end of the line is ignored.

```basic
10 REM This is a comment
20 A = 5   : REM Inline comment after colon
```

---

### Control flow

**GOTO linenum**
Jump to the specified line number.
```basic
10 PRINT "Loop"
20 GOTO 10
```

**IF expression THEN statement [ELSE statement]**
Conditional execution. If the expression is non-zero, execute the THEN clause. Otherwise execute the ELSE clause (if present).
```basic
10 IF A > 0 THEN PRINT "Positive" ELSE PRINT "Non-positive"
```

**IF expression THEN linenum**
Conditional jump. Equivalent to `IF expression THEN GOTO linenum`.
```basic
10 IF X = 0 THEN 100
```

**FOR variable = start TO limit [STEP increment]**
**NEXT [variable]**
Counted loop. The variable steps from start to limit by increment (default 1). The body executes at least once if start meets the limit condition. NEXT increments and tests; if the limit is exceeded, execution continues after NEXT.
```basic
10 FOR I = 1 TO 10
20   PRINT I
30 NEXT I

40 FOR I = 10 TO 0 STEP -2
50   PRINT I
60 NEXT I
```

**WHILE expression**
**WEND**
Repeat the body while expression is non-zero. The condition is tested before each iteration; if initially false, the body does not execute.
```basic
10 X = 1
20 WHILE X <= 100
30   PRINT X
40   X = X * 2
50 WEND
```

**GOSUB linenum**
**RETURN**
Subroutine call. GOSUB saves the current position and jumps to linenum. RETURN resumes execution after the most recent GOSUB. Maximum nesting depth: 10.
```basic
10 GOSUB 100
20 GOSUB 100
30 END
100 PRINT "In subroutine"
110 RETURN
```

---

### Math functions

| Function | Description |
|----------|-------------|
| `ABS(x)` | Absolute value |
| `INT(x)` | Truncate to integer (toward zero) |
| `SQR(x)` | Square root |
| `RND(n)` | Random number. If n > 0: integer 0 to n-1. If n <= 0: float 0.0 to 1.0. |

---

### String functions

| Function | Returns | Description |
|----------|---------|-------------|
| `LEN(s$)` | Number | Length of string |
| `LEFT$(s$, n)` | String | First n characters |
| `RIGHT$(s$, n)` | String | Last n characters |
| `MID$(s$, pos, len)` | String | Substring starting at pos (1-based), len characters |
| `MID$(s$, pos)` | String | From pos to end of string |
| `STR$(n)` | String | Convert number to string |
| `VAL(s$)` | Number | Convert string to number (0 if not valid) |
| `CHR$(n)` | String | Character with ASCII code n |
| `ASC(s$)` | Number | ASCII code of first character |
| `UPPER$(s$)` | String | Convert to uppercase |
| `LOWER$(s$)` | String | Convert to lowercase |
| `INSTR(s$, find$)` | Number | Position of find$ in s$ (1-based), 0 if not found |

### String concatenation

The `+` operator concatenates strings:
```basic
C$ = "Hello" + ", " + "World"
```

---

### Arrays

**DIM variable(size)**
**DIM variable(rows, cols)**

Create an array. Elements are initialized to 0 (numeric) or empty string (string). Arrays must be dimensioned before use. Redimensioning the same name is an error.

```basic
DIM A(20)         REM A(0) through A(20) = 21 elements
DIM M(3, 4)       REM M(0,0) through M(3,4) = 20 elements
```

---

### Display

**CLS [color]**
Clear the screen. If color is given, fill with that RGB565 color.

**LOCATE row, col**
Move the text cursor. Row 0 is top, col 0 is left. Screen is 30 rows by 53 columns at default font.

**COLOR foreground [, background]**
Set the text color for subsequent PRINT and LOCATE output. Colors are 16-bit RGB565 values.

Common RGB565 colors:
| Name | Value |
|------|-------|
| Black | 0 |
| Red | 63488 |
| Green | 2016 |
| Blue | 31 |
| Yellow | 65504 |
| Cyan | 2047 |
| Magenta | 63519 |
| White | 65535 |

---

### Drawing

All drawing commands take RGB565 color as an optional last argument. If omitted, the current foreground color (set by COLOR) is used.

**PIXEL x, y [, color]**
Draw a single pixel.

**LINE x1, y1, x2, y2 [, color]**
Draw a line between two points.

**RECT x, y, width, height [, color]**
Draw a rectangle outline. x, y is the top-left corner.

**FILLRECT x, y, width, height [, color]**
Draw a filled rectangle.

**CIRCLE x, y, radius [, color]**
Draw a circle outline. x, y is the center.

**FILLCIRC x, y, radius [, color]**
Draw a filled circle.

Screen dimensions: 320 pixels wide (x: 0-319), 240 pixels tall (y: 0-239).

---

### Sound

**BEEP frequency, duration**
Play a tone. Frequency in Hz, duration in milliseconds.
```basic
BEEP 440, 500      REM A4 note for half a second
BEEP 1000, 100     REM short beep
```

---

### Timing and input

**DELAY milliseconds**
Pause execution. Maximum 60000 (one minute).

**MILLIS**
Returns the number of milliseconds since the device booted. Numeric value, usable in expressions.
```basic
T = MILLIS
```

**BTN(n)**
Returns 1 if button n is currently pressed, 0 otherwise. Button 1 = A (left), 2 = B (center), 3 = C (right).
```basic
IF BTN(1) = 1 THEN PRINT "A pressed"
```

**INKEY$**
Returns a one-character string containing the last key pressed (from CardKB or serial), or an empty string if no key was pressed. Non-blocking.
```basic
10 K$ = INKEY$
20 IF K$ <> "" THEN PRINT "Key: "; K$
```

---

### File system

All file operations use the `/basic/` directory on the SD card. Filenames that do not start with `/` are automatically prefixed with `/basic/`. LOAD automatically tries appending `.bas` if the file is not found.

**SAVE "filename"**
Save the current program to SD card.

**LOAD "filename"**
Load a program from SD card into memory. Replaces the current program.

**RUN "filename"**
Load from SD and immediately execute. Shortcut for LOAD + RUN.

**DIR**
List all files in `/basic/` with their sizes.

**DELETE "filename"**
Remove a file from the SD card.

**CAT "filename"**
Display the contents of a file on screen without loading it into program memory.

**Autorun:** If the file `/basic/autorun.bas` exists, it is automatically loaded and run when the device boots.

---

### Advanced file I/O

Three file handles (#1, #2, #3) are available for simultaneous file operations.

**OPEN "filename", handle, "mode"**
Open a file. Handle is 1, 2, or 3. Mode is "R" (read), "W" (write/create), or "A" (append).
```basic
OPEN "data.csv", 1, "W"
```

**CLOSE handle**
Close a file handle.

**FPRINT handle, expression**
Write a line to the file. Adds a newline at the end.
```basic
FPRINT 1, "Name,Score"
FPRINT 1, N$ + "," + STR$(S)
```

**FINPUT handle, variable**
Read one line from the file into a variable.
```basic
FINPUT 1, L$
FINPUT 1, N
```

---

### WiFi

**WIFI "ssid", "password"**
Connect to a WiFi network. Blocks until connected or timeout (10 seconds). Prints the assigned IP address on success.

**IP$**
String function returning the current local IP address (e.g. "192.168.1.42"). Returns "0.0.0.0" if not connected.

**WGET "url", result$**
Perform an HTTP GET request. The response body (truncated to 63 characters) is stored in result$. The HTTP status code is stored in variable H.
```basic
WGET "http://api.example.com/data", R$
PRINT "Status: "; H
PRINT "Body: "; R$
```

---

### Web server

**WSERV [port]**
Start a web server on the specified port (default 80). Requires WiFi to be connected first.

**WSTOP**
Stop the web server.

**WSEND statuscode, "content"**
Send an HTTP response to the pending request. Content-Type is always text/html.
```basic
WSEND 200, "<h1>Hello</h1>"
WSEND 404, "Not Found"
```

**Handling requests:** The variable W is set to 1 when an HTTP request is received and is waiting for a response. The program must poll W in a loop and call WSEND to respond.

```basic
10 WIFI "net", "pass"
20 WSERV 80
30 WHILE BTN(2) = 0
40   IF W = 1 THEN WSEND 200, "<p>OK</p>"
50   DELAY 50
60 WEND
70 WSTOP
```

---

### ESP-NOW

Peer-to-peer communication between ESP32 devices without a WiFi access point. Range approximately 200 meters line-of-sight. Latency under 1 millisecond. Maximum message size: 249 bytes.

**ENOW**
Initialize ESP-NOW. Prints the device's own MAC address. WiFi must be in STA mode (WIFI or ENOW sets this automatically).

**EPEER "AA:BB:CC:DD:EE:FF"**
Register a peer device by its MAC address. Must be called before sending to that peer.

**ESEND "AA:BB:CC:DD:EE:FF", message$**
Send a message to a registered peer.

**ERECV variable$**
Non-blocking receive. If a message has arrived, it is stored in the variable. Check variable E: 1 means data was received, 0 means no data. When data is received, R$ contains the sender's MAC address.

```basic
10 ENOW
20 WHILE BTN(2) = 0
30   ERECV M$
40   IF E = 1 THEN PRINT "From "; R$; ": "; M$
50   DELAY 100
60 WEND
```

---

### LoRa

Long-range radio communication using the SX1276 module. Range up to 15+ km depending on conditions. Low bandwidth (a few hundred bytes per second). Half-duplex: the radio switches between transmit and receive.

**LINIT [frequency]**
Initialize the LoRa radio. Frequency in MHz; default is 868.0 (EU standard). After init the radio enters receive mode.
```basic
LINIT           REM 868 MHz (default)
LINIT 433       REM 433 MHz
```

**LSEND message$**
Transmit a packet. The radio briefly switches to transmit mode, sends the data, then returns to receive mode.

**LRECV variable$**
Non-blocking receive. If a packet has arrived, it is stored in the variable. Check variable L: 1 means data received, 0 means no data. Variable R contains the RSSI (signal strength) of the last received packet in dBm (typically -30 to -120; closer to 0 is stronger).

**LFREQ frequency**
Change the radio frequency (MHz) without re-initializing.

```basic
10 LINIT 868
20 WHILE BTN(2) = 0
30   LRECV M$
40   IF L = 1 THEN PRINT M$; " ("; R; " dBm)"
50   DELAY 100
60 WEND
```

---

### Serial transfer

These commands work in conjunction with the PC-side tool `m5basic_upload.py` but can also be used manually.

**UPLOAD**
Enter serial receive mode. The interpreter waits for numbered BASIC lines via serial, terminated by `>>>END`. Lines are parsed and added to the program in memory.

**DOWNLOAD**
Send the current program over serial in a format that the PC-side tool can capture and save.

The serial transfer protocol also runs automatically in the background. The PC-side tool can upload programs at any time without typing UPLOAD first -- the interpreter detects the `>>>XFER` marker during idle REPL cycles.

---

## System commands summary

| Command | Description |
|---------|-------------|
| NEW | Erase program |
| LIST [from[-to]] | Show program |
| RUN ["file"] | Execute (optionally load first) |
| END | Stop execution |
| CLR | Clear variables |
| HELP | Show available commands |
| BtnB | Break (stop running program) |

---

## Limits summary

| Resource | Maximum |
|----------|---------|
| Program lines | 500 |
| Characters per line | 128 |
| Numeric variables | 26 (A-Z) |
| String variables | 26 (A$-Z$) |
| Maximum string length | 63 characters |
| Arrays | 10 |
| Array elements per dimension | 100 |
| GOSUB nesting | 10 |
| FOR nesting | 10 |
| WHILE nesting | 10 |
| Open file handles | 3 |
| Screen columns | 53 |
| Screen rows | 30 |
| Screen pixels | 320 x 240 |

---

## Error messages

| Message | Cause |
|---------|-------|
| SYNTAX ERROR IN EXPRESSION | Malformed expression or unexpected token |
| EXPECTED VARIABLE | Assignment target is not a variable |
| EXPECTED TOKEN n GOT m | Parser expected a specific token |
| LINE n NOT FOUND | GOTO or GOSUB target does not exist |
| DIVISION BY ZERO | Division or MOD by zero |
| NEXT WITHOUT FOR | NEXT without matching FOR |
| NEXT VARIABLE MISMATCH | NEXT variable does not match current FOR |
| FOR STACK OVERFLOW | FOR nested more than 10 deep |
| RETURN WITHOUT GOSUB | RETURN without matching GOSUB |
| GOSUB STACK OVERFLOW | GOSUB nested more than 10 deep |
| WEND WITHOUT WHILE | WEND without matching WHILE |
| ARRAY ALREADY DEFINED | DIM for a name that already has an array |
| INVALID ARRAY SIZE | DIM with zero, negative, or too-large size |
| TOO MANY ARRAYS | More than 10 arrays |
| OUT OF MEMORY | Cannot allocate array |
| PROGRAM TOO LARGE | More than 500 lines |
| NO PROGRAM | RUN with empty program |
| SD CARD NOT READY | SD card not inserted or failed to initialize |
| FILE NOT FOUND | LOAD, CAT, or OPEN for a non-existent file |
| CANNOT WRITE FILE | SD card full or write-protected |
| CANNOT OPEN FILE | File open failed |
| CANNOT DELETE | File delete failed |
| INVALID FILE HANDLE | Handle number not 1, 2, or 3 |
| FILE NOT OPEN | Read/write on a handle that is not open |
| WIFI TIMEOUT | Could not connect within 10 seconds |
| WIFI NOT CONNECTED | Network command before WIFI |
| HTTP ERROR n | WGET received non-200 status |
| NO WEB SERVER | WSEND without WSERV |
| ESP-NOW INIT FAILED | ENOW hardware failure |
| ESP-NOW NOT INIT | ESEND/ERECV before ENOW |
| INVALID MAC FORMAT | EPEER with malformed MAC address |
| ADD PEER FAILED | ESP-NOW peer table full or invalid |
| SEND FAILED | ESP-NOW or LoRa transmission error |
| LORA INIT FAILED | LINIT hardware failure |
| LORA NOT INIT | LSEND/LRECV before LINIT |
| LORA LIB NOT FOUND | LoRa library not installed |
| NOT AVAILABLE | Command not compiled in current build phase |
