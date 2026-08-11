# Railway Crossing Gate Controller — Verilog HDL

A safety-critical railway crossing gate controller implemented in Verilog HDL using a Finite State Machine. Built in two versions — a basic 3-state controller and an upgraded 5-state adaptive controller with parameterized safety timing.

## What It Does

Monitors two train detection sensors and controls:
- **Gate** — closes when a train is detected, reopens only after a safety delay
- **Road signal** — red during train crossing, green when clear

The safety wait state is the most important design decision — it prevents the gate from reopening immediately after the train clears the sensor, modelling how real railway crossings behave.

## FSM Design

```
                  sensor_A=1
   ┌──────────────────────────────────┐
   │                                  ▼
 IDLE ──────────────────────────► APPROACH
   ▲                                  │
   │                                  │ (next cycle)
   │                                  ▼
 CLEAR ◄──── SAFETY_WAIT ◄─── PASSING
            (timer ≥ N cycles)  (sensor_B=1)
```

| State | Gate | Signal | Description |
|---|---|---|---|
| IDLE | Open | Green | No train detected |
| APPROACH | Closed | Red | Train arriving |
| PASSING | Closed | Red | Train crossing |
| SAFETY_WAIT | Closed | Red | Timed delay before reopening |
| CLEAR | Open | Green | Returning to idle |

**Moore machine** — outputs depend only on current state, not inputs. This makes gate behaviour glitch-free and predictable, which is essential for safety-critical hardware.

## Two Versions

### Basic (`railway.v`) — 3 states
Simple controller with IDLE → TRAIN_APPROACH → TRAIN_PASS states.
Single sensor input, no safety delay.

### Adaptive (`upgraded_railway.v`) — 5 states
Upgraded controller with dual sensors and parameterized safety timing.

```verilog
module adaptive_railway #(
    parameter SAFETY_CYCLES = 5   // adjustable safety delay
)(
    input  wire clk, reset,
    input  wire sensor_A,   // train approaching
    input  wire sensor_B,   // train cleared crossing
    output reg  gate,       // 1 = gate closed
    output reg  signal      // 1 = road signal red
);
```

## File Structure

```
railway_controller/
├── rtl/
│   ├── railway.v            — basic 3-state FSM
│   └── upgraded_railway.v   — adaptive 5-state FSM
├── tb/
│   ├── railway_tb.v         — basic testbench
│   └── upgraded_tb.v        — upgraded testbench
├── synth/
│   └── synth_report.txt     — Yosys synthesis results
├── docs/
│   └── waveform.png         — GTKWave simulation output
└── sim/                     — VCD waveform files
```

## Simulation

```
# Basic version
iverilog -o sim/railway.out rtl/railway.v tb/railway_tb.v
vvp sim/railway.out
gtkwave sim/railway.vcd

# Upgraded version
iverilog -o sim/upgraded.out rtl/upgraded_railway.v tb/upgraded_tb.v
vvp sim/upgraded.out
gtkwave sim/upgraded.vcd
```

## Synthesis (Yosys)

Synthesized with Yosys 0.33 — generic technology-independent gate mapping.

| Version | Cells | Flip-flops | Notes |
|---|---|---|---|
| Basic (3-state) | 24 | 2 | Simple FSM |
| Adaptive (5-state) | 56 | 7 | Includes 4-bit timer |

**Adaptive version cell breakdown:**
```
AND : 10    OR  : 21    NOT : 12
MUX :  3    XOR :  3    DFF :  7
```

Full report: `synth/synth_report.txt`

## Design Decisions

**Why Moore over Mealy?** Moore machine outputs depend only on the current state, not on inputs. This means the gate can never flicker due to a brief noise pulse on the sensor — output changes only happen at clock edges. Safety-critical hardware almost always uses Moore machines.

**Why a separate SAFETY_WAIT state?** In a real railway crossing, even after the train clears the exit sensor, the gate must wait to ensure the train has fully passed. Hardcoding this as a parameterized timer (`SAFETY_CYCLES`) means the delay can be adjusted for different crossing lengths without changing the FSM logic.

**Why default output assignments?** Without `gate = 0; signal = 0;` at the top of the output always block, the synthesis tool infers a latch for unhandled states. Latches are level-sensitive and cause timing issues in real silicon. The fixed version eliminates this by always providing a defined output.

## Tools Used

- Verilog HDL
- Icarus Verilog (simulation)
- GTKWave (waveform viewer)
- Yosys 0.33 (synthesis)
- VS Code

## Author

Janvi Papola — ECE, IGDTUW, 2nd Year
