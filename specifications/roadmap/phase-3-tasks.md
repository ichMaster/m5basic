# Phase 3 — CardKB, SD Card, File System Tasks

## 1. CardKB Input

### 1.1 I2C keyboard polling (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Initialize I2C and poll CardKB at address 0x5F in `begin()` when `M5BASIC_CARDKB` defined | -- |
| 2 | Implement `consoleReadKey()` — poll CardKB via `Wire.requestFrom(0x5F, 1)`, return keycode or -1 | -- |
| 3 | Implement dual input in `consoleReadLine()` — accept characters from both CardKB and serial simultaneously | -- |
| 4 | Update `INKEY$` to read from CardKB when available, serial as fallback | -- |
| 5 | Map CardKB special keys: Enter, Backspace, arrow keys | -- |

### 1.2 Button B break (`src/repl.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Poll `M5.BtnB` during `runProgram()` loop — if pressed, set `_running = false` | -- |
| 2 | Print `? BREAK` when button B stops a running program | -- |
| 3 | Poll button B inside `DELAY` command so long delays are interruptible | -- |

---

## 2. SD Card File System

### 2.1 SD card initialization (`src/phase1_files.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `initSD()` — call `SD.begin()`, check card presence, print status | -- |
| 2 | Create `/basic/` directory on SD if it doesn't exist | -- |
| 3 | Call `initSD()` in `begin()` when `M5BASIC_FILESYSTEM` defined | -- |

### 2.2 Basic file commands (`src/phase1_files.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `SAVE "filename"` — write all program lines to `/basic/filename` on SD, auto-add `.bas` extension | -- |
| 2 | Implement `LOAD "filename"` — read file from SD, parse lines, call `addLine()`, auto-try `.bas` extension | -- |
| 3 | Implement `RUN "filename"` — LOAD then RUN in one command | -- |
| 4 | Implement `DIR` — list files in `/basic/` with sizes | -- |
| 5 | Implement `DELETE "filename"` — remove file from SD | -- |
| 6 | Implement `CAT "filename"` — display file contents on screen without loading into program memory | -- |
| 7 | Auto-prefix filenames that don't start with `/` with `/basic/` | -- |

### 2.3 Autorun (`src/phase1_files.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `checkAutorun()` — check if `/basic/autorun.bas` exists on SD | -- |
| 2 | If autorun exists: call `loadFile("autorun.bas")` then `runProgram()` | -- |
| 3 | Call `checkAutorun()` in `begin()` after SD init, before REPL | -- |

---

## 3. Advanced File I/O

### 3.1 File handle system (`src/phase4_5_arrays_files.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Define file handle state: 3 handles, each with `File` object, open flag, mode (R/W/A) | -- |
| 2 | Implement `OPEN "filename", #n, "mode"` — validate handle 1-3, open file in specified mode | -- |
| 3 | Implement `CLOSE #n` — close file handle, clear state | -- |
| 4 | Implement `FPRINT #n, expression` — evaluate expression, write line to file with newline | -- |
| 5 | Implement `FINPUT #n, variable` — read one line from file into numeric or string variable | -- |
| 6 | Error handling: invalid handle, file not open, read past EOF, write failure | -- |

---

## 4. Serial Transfer with SD

### 4.1 SD-aware transfer protocol (`src/phase1_files.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Update `serialReceiveProgram()` — if filename provided in `>>>XFER`, also save to SD after loading into RAM | -- |
| 2 | Update `serialSendProgram()` — if filename in `>>>FETCH`, read from SD file instead of RAM | -- |
| 3 | Implement `serialListFiles()` — list files in `/basic/` as `<<<LINE filename size` entries | -- |

---

## 5. SD Deploy Tool

### 5.1 Batch deployment (`tools/deploy_to_sd.sh`)

| # | Task | Issue |
|---|------|-------|
| 1 | Accept SD card mount point as argument | -- |
| 2 | Create `/basic/` directory on SD if missing | -- |
| 3 | Copy all `.bas` files from `examples/phase0/` through `examples/phase7/` to SD | -- |
| 4 | Print summary of files copied | -- |

---

## 6. Example Programs

### 6.1 File system examples (`examples/phase4/`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create data logger — OPEN, FPRINT CSV data with MILLIS timestamps, CLOSE | -- |
| 2 | Create config manager — SAVE/LOAD settings to file, read back on start | -- |
| 3 | Create high score system — read/write scores with FPRINT/FINPUT | -- |
| 4 | Create simple file browser — DIR, CAT, user selects file to LOAD and RUN | -- |
| 5 | Create autorun.bas example — startup menu that offers to run different programs | -- |
