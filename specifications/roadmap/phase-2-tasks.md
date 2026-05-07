# Phase 2 — Display and Graphics Tasks

## 1. LCD Text Console

### 1.1 Text console implementation (`src/interpreter_core.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Initialize M5Stack LCD: orientation, font size 6x8, black background | -- |
| 2 | Implement LCD `consolePrint()` — write characters to 53x30 text grid, track cursor position (row, col) | -- |
| 3 | Implement LCD `consolePrintLn()` — print text then advance to next row | -- |
| 4 | Implement LCD `consoleNewLine()` — advance cursor row, reset col to 0 | -- |
| 5 | Implement LCD `consoleScroll()` — when cursor exceeds row 29, scroll all text up by one row, clear bottom row | -- |
| 6 | Implement LCD `consoleClear()` — fill screen with color, reset cursor to (0, 0) | -- |
| 7 | Implement LCD `consoleReadLine()` — read from CardKB (if available) or serial, echo to LCD and serial, handle backspace on LCD | -- |
| 8 | Maintain dual output: all LCD console methods also call `Serial.print()` so serial terminal mirrors display | -- |

### 1.2 Display mirror for serial mode (`src/serial_io.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | When `M5BASIC_DISPLAY` defined, add LCD mirror to serial `consolePrint()` — write to LCD in addition to serial | -- |
| 2 | Track LCD cursor position in serial mode for `LOCATE` support | -- |
| 3 | Implement `consoleClear()` for serial+display mode — clear both serial terminal and LCD | -- |

---

## 2. Display Commands

### 2.1 Text display commands (`src/phase3_display.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `CLS [color]` — clear LCD screen, optional RGB565 fill color, reset cursor | -- |
| 2 | Implement `LOCATE row, col` — move text cursor to specified position (0-based) | -- |
| 3 | Implement `COLOR fg [, bg]` — set foreground (and optional background) text color for subsequent PRINT | -- |

---

## 3. Graphics Primitives

### 3.1 Drawing commands (`src/phase3_display.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `PIXEL x, y [, color]` — draw single pixel, default to current foreground color | -- |
| 2 | Implement `LINE x1, y1, x2, y2 [, color]` — draw line between two points | -- |
| 3 | Implement `RECT x, y, w, h [, color]` — draw rectangle outline | -- |
| 4 | Implement `FILLRECT x, y, w, h [, color]` — draw filled rectangle | -- |
| 5 | Implement `CIRCLE x, y, r [, color]` — draw circle outline | -- |
| 6 | Implement `FILLCIRC x, y, r [, color]` — draw filled circle | -- |
| 7 | Parse optional color argument for all drawing commands — if present use it, otherwise use current foreground | -- |

---

## 4. Sound

### 4.1 Speaker output (`src/phase3_display.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `BEEP freq, duration_ms` — play tone via `M5.Speaker` | -- |
| 2 | Validate frequency (reasonable range) and duration (max 60000ms) | -- |

---

## 5. Hardware Input and Timing

### 5.1 Button and timing functions (`src/phase3_display.cpp`)

| # | Task | Issue |
|---|------|-------|
| 1 | Implement `BTN(n)` — return 1 if button pressed (1=A, 2=B, 3=C), 0 otherwise; integrate into `evalAtom()` | -- |
| 2 | Implement `MILLIS` — return `millis()` as float; integrate into `evalAtom()` | -- |
| 3 | Implement `DELAY ms` — call `delay()`, cap at 60000ms | -- |
| 4 | Implement `INKEY$` hardware extension — poll CardKB (if `M5BASIC_CARDKB`) in addition to serial | -- |

---

## 6. Example Programs

### 6.1 Display examples (`examples/phase1/`, `examples/phase2/`)

| # | Task | Issue |
|---|------|-------|
| 1 | Create color text demo — CLS, COLOR, LOCATE, PRINT with multiple colors | -- |
| 2 | Create bouncing ball animation — FILLCIRC, DELAY, INKEY$ loop | -- |
| 3 | Create bar chart — FILLRECT with multiple colors, LOCATE for labels | -- |
| 4 | Create piano keyboard — BEEP with different frequencies on button press | -- |
| 5 | Create stopwatch — MILLIS, BTN, CLS, LOCATE for time display | -- |
| 6 | Create drawing program — BTN for color selection, pixel art grid | -- |
