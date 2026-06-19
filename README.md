# Verifying a Synchronous FIFO and Analyzing Boundary Conditions

This repository contains a functional verification environment for a parameterized Synchronous FIFO (16-depth, 8-bit width). The objective of this project was to construct a directed Verilog testbench from scratch to validate internal tracking logic, pointer increments, and flag status transitions under boundary stress conditions.

## Project Architecture

The verified module is a classic pointer-based Synchronous FIFO memory array operating on a single clock domain.

* **Data Width:** 8 bits (`din[7:0]`, `dout[7:0]`)
* **FIFO Depth:** 16 entries
* **Status Flags:** `full` and `empty`
* **Internal Trackers:** 4-bit write pointer (`wr_ptr`), 4-bit read pointer (`rd_ptr`), and a 5-bit element counter (`count[4:0]`)

## Verification Strategy

The verification environment focuses heavily on verifying hardware corner cases rather than basic functional data paths. The testbench drives specific stimulus loops to evaluate the following boundary conditions:

### 1. Burst Write & Overflow Protection
Continuous data sequences (values 1 through 16) are driven into the memory array. The testbench verifies that upon reaching maximum capacity (`count == 16`), the `full` flag activates on the correct clock edge. The stimulus intentionally holds the write enable (`wr_en`) signal high past full capacity to verify that internal logic successfully gates off further modifications, preventing memory corruption and pointer runaway.

### 2. Burst Read & Underflow Protection
Following the write phase, back-to-back burst read requests are asserted until the memory is completely drained. The environment verifies that when `count == 0`, the `empty` flag asserts immediately. The testbench drives extra read clock cycles while empty to confirm that the read logic ignores the invalid transactions and holds the last valid data pattern instead of outputting garbage or undefined states.

## Simulation Waveform Progression

The simulation follows a structured linear pipeline to observe state transition latencies:

1. **System Reset:** Asserting `rst` high clears the data buses, resets pointers to zero, and forces the `empty` flag high.
2. **Sequential Load:** Driving successive data blocks until the buffer fills completely up to value 16.
3. **Overflow Gate Window:** Sustaining write operations at maximum capacity to confirm data blocking.
4. **Sequential Unload:** Reading back elements sequentially to observe proper flag de-assertion.
5. **Underflow Gate Window:** Sustaining read operations at zero capacity to confirm logic stability.

## Repository Directory Structure

```text
├── rtl/
│   └── fifo.v          # Synchronous FIFO RTL Design
├── bench/
│   └── fifo_tb.v       # Directed Verilog Testbench
└── docs/
    └── waveforms/      # Simulation Waveform Captures
