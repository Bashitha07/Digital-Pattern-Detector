# Digital Pattern Detector (`0011`)

**IE3004 – Digital Systems Design Assignment**  
**Sri Lanka Institute of Information Technology (SLIIT)**  
**B.Sc. (Hons) in Computer Systems Engineering**  
**Group 02**

## Project Overview

This project presents the hardware design, simulation, and PCB layout of a synchronous digital pattern detection system built with 74HC-series CMOS logic ICs.

The system:

1. Generates a pseudo-random 4-bit sequence using a linear feedback shift register (LFSR).
2. Loads each 4-bit word into a parallel-in, serial-out (PISO) shift register.
3. Transmits the word least-significant bit first.
4. Detects the overlapping binary pattern `0011` using a four-state Mealy finite-state machine (FSM).

The design was simulated in Proteus Design Suite and captured as a two-layer, through-hole PCB in KiCad.

## System Architecture

```text
+-------------------+       P3..P0       +-------------------+       X = Q0       +-------------------+
| 4-bit LFSR        | ------------------> | 4-bit PISO       | ------------------> | Mealy FSM         |
| pseudo-random     |                     | shift register   |                     | `0011` detector  |
+-------------------+                     +-------------------+                     +-------------------+
        ^                                          ^                                      |
        |                                          |                                      v
   [SW1 seed]                                 [LOAD control]                         Z (output)
                                             Common rising-edge CLK
```

All three subsystems use the same rising-edge clock.

## 1. 4-Bit LFSR

A maximal-length 4-bit LFSR cycles through all `2^4 - 1 = 15` non-zero states before repeating. The all-zero state is excluded because it would lock the register at zero.

The active-low asynchronous initialization input, controlled by push-button `SW1` and a `10 kΩ` pull-up resistor, loads the seed word:

```text
G3 G2 G1 G0 = 0001
```

The next-state equations are:

```text
Q3+ = Q2
Q2+ = Q1
Q1+ = Q0
Q0+ = Q3 XOR Q2
```

## 2. 4-Bit PISO Shift Register

The PISO register captures the LFSR output in parallel when `LOAD = 0`. When `LOAD = 1`, it shifts one bit toward `Q0` on each rising clock edge.

For `L = LOAD`, the D-input equations are:

```text
D3 = P3 · L'
D2 = Q3 · L + P2 · L'
D1 = Q2 · L + P1 · L'
D0 = Q1 · L + P0 · L'
```

The serial output is:

```text
X = Q0
```

Because transmission is least-significant bit first, a parallel word of `1100` produces the serial sequence:

```text
0 → 0 → 1 → 1
```

This sequence is the target pattern detected by the FSM.

## 3. Overlapping Mealy FSM Detector

The detector identifies `0011` using a four-state overlapping Mealy FSM. A Mealy machine generates its output from both the current state and the current input, allowing the detection pulse to occur during the clock cycle in which the final `1` arrives.

### State Encoding

| State | Encoding | Meaning |
| :--- | :---: | :--- |
| A | `00` | No useful prefix matched |
| B | `01` | Prefix `0` matched |
| C | `11` | Prefix `00` matched |
| D | `10` | Prefix `001` matched |

### Excitation and Output Equations

```text
D1 = Q0 · X' + Q1 · Q0
D0 = X'
Z  = Q1 · Q0' · X
```

The output `Z` is HIGH when the FSM is in state D and the current serial input `X` is `1`, indicating that `0011` has been detected.

### Self-Synchronization

Two consecutive `1` inputs drive the FSM to state A (`00`) from any state. This provides a practical way to place the detector in a known state without a dedicated FSM reset line.

## Integrated Circuit Allocation

| IC | Function | Packages Used | Utilization |
| :--- | :--- | :---: | :--- |
| `74HC74` | Dual D-type positive-edge flip-flop | 5 | 10 of 10 flip-flops |
| `74HC08` | Quad 2-input AND gate | 3 | 11 of 12 gates |
| `74HC32` | Quad 2-input OR gate | 1 | 4 of 4 gates |
| `74HC04` | Hex inverter | 1 | 3 of 6 inverters |
| `74HC86` | Quad 2-input XOR gate | 1 | 1 of 4 gates |

### Additional Components

- `1 × 10 kΩ` pull-up resistor
- `9 × 330 Ω` LED current-limiting resistors
- `9 ×` indicator LEDs (`D1–D9`)
- `1 ×` tactile push-button (`SW1`)
- `2 ×` 2-pin header connectors (`J1`, `J2`)

## Repository Contents

| Directory/File | Description |
| :--- | :--- |
| [`KiCad Files/`](KiCad%20Files/) | KiCad schematic, PCB layout, and project design files |
| [`Proteus/`](Proteus/) | Proteus simulation project and generated simulation files |
| [`Photos/`](Photos/) | PCB renders, board-layer views, schematics, and waveform captures |
| [`Report/`](Report/) | Technical reports and assignment documentation |
| [`README.md`](README.md) | Project overview and design documentation |

## Testing and Verification

### T1 – Seed Initialization

Stop the clock, press and release `SW1`, and verify that the LFSR is initialized to `0001`.

### T2 – LFSR Period

Apply 15 clock pulses and verify that all 15 non-zero states occur before the sequence returns to `0001`.

### T5 – FSM Synchronization

Apply two consecutive serial `1` inputs and verify that the FSM reaches state A (`00`).

### T6/T7 – Pattern Detection

1. Set `LOAD = 0` and apply one rising clock edge to load the selected LFSR word into the PISO register.
2. Set `LOAD = 1` and apply four rising clock edges.
3. For the loaded word `1100`, verify the serial sequence `0 → 0 → 1 → 1`.
4. Verify that `Z` pulses HIGH during the fourth transmitted bit, when the FSM detects the complete `0011` pattern.

## Design Files

- **KiCad:** [`KiCad Files/KiCad_Y3S1.kicad_sch`](KiCad%20Files/KiCad_Y3S1.kicad_sch) and [`KiCad Files/KiCad_Y3S1.kicad_pcb`](KiCad%20Files/KiCad_Y3S1.kicad_pcb)
- **Proteus:** [`Proteus/DSD_Final_Schematic.pdsprj`](Proteus/DSD_Final_Schematic.pdsprj)
- **Reports:** [`Report/`](Report/)

## Group Members

| Student ID | Name |
| :--- | :--- |
| `IT24103485` | Sellapperuma B.A. |
| `IT24100888` | L P Jayasinghe |
| `IT24610799` | Kalpage M.L.P. |
