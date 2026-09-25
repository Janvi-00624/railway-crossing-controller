## Railway Crossing Controller — Verilog Project

## Project Description
- Developed a railway crossing gate controller using **Verilog HDL** to simulate the operation of a real-world safety system.
- The project focused on understanding how **digital logic and finite state machines (FSMs)** can be used to model sequential control processes.
- Started with a basic gate-control design and later enhanced it with additional states and timing logic to create a more realistic and responsive controller.
- The project provided practical experience with **sequential logic, FSM-based design, timing behavior, and digital circuit simulation**.


## What the Controller Does
The controller monitors train detection sensors and controls:
- Railway gate (open/close)
- Signal indication

The upgraded version introduces multiple states to safely handle:
- Train arrival
- Crossing phase
- Safety delay before reopening

This prevents unsafe gate behavior and mimics real-world timing.

## Design Approach
The controller is implemented using a Finite State Machine (FSM).
The main states include:
- Idle: no train detected, gate open
- Approach: train arrival detected
- Passing: train crossing
- Safety wait: delay before reopening 
- Clear: system resets to idle
The FSM structure made it easier to organize logic and transitions.

## Tools Used
- Verilog HDL
- Icarus Verilog (simulation)
- GTKWave (waveform viewer)
- VS Code

## how to run simulation
```
iverilog -o railway.out adaptive_railway.v adaptive_tb.v
vvp railway.out
gtkwave railway.vcd
```
## What I Learned
- Basics of FSM-based design
- Sequential logic behavior
- Simulation workflow
- Debugging digital circuits
- Structuring HDL projects

## Author
Janvi
