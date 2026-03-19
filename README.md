# UVM ALU Agent – Vivado XSim
A UVM-based verification environment for an 8-bit ALU, simulated using Vivado XSim (v2025.2). Implements a full UVM agent with driver, monitor, scoreboard, directed and random sequences.


# Project Structure
rtl/alu.sv          ->  8-bit ALU DUT (registered handshake) \
tb/alu_if.sv        ->  SystemVerilog interface (modports: dut, drv, mon) \
tb/alu_pkg.sv       ->  UVM package (all classes in compile order) \
top/alu_tb_top.sv   ->  Top level testbench (clock, reset, run_test) \
sim/run.tcl         ->  Compile + simulate script 


# ALU Specification
    Opcode        Operation        Description 
   0    |       ADD      |   result = a + b \
   1           SUB         result = a - b \
   2           AND         result = a & b \
   3           OR          result = a | b \
   4           XOR         result = a ^ b \
   5           SHL         result = a << (b & 7) \
   6           SHR         result = a >> (b & 7) 

8-bit unsigned operands, overflow wraps \
1-cycle registered latency with valid/ready handshake


# UVM Components
     File                                 Role 
`alu_seq_item           Transaction (a, b, op, exp_result, act_result) \
`alu_directed_seq       Fixed corner-case stimulus \
`alu_rand_seq           Randomized stimulus (length controlled by `ntxn`) \
`alu_driver             Drives DUT pins, respects in_valid/in_ready \
`alu_monitor            Samples output handshake, publishes to scoreboard \
`alu_scoreboard         Pure reference model, compares expected vs actual \
`alu_agent              Bundles sequencer + driver + monitor \
`alu_env                Connects agent → scoreboard via analysis port \
`alu_smoke_test         Runs directed sequence \
`alu_rand_test          Runs random sequence 


# How to Compile
exec xvlog -sv -L uvm --timescale 1ns/1ps rtl/alu.sv tb/alu_if.sv tb/alu_pkg.sv top/alu_tb_top.sv \
exec xelab -L uvm -debug typical -top alu_tb_top -snapshot alu_tb


# How to Run
Smoke test (directed):
exec xsim alu_tb -R \
  -testplusarg UVM_TESTNAME=alu_smoke_test \
  -testplusarg UVM_VERBOSITY=UVM_MEDIUM \
  -sv_seed 12345 \
  -log results/smoke_run.log

Random test:
exec xsim alu_tb -R \
  -testplusarg UVM_TESTNAME=alu_rand_test \
  -testplusarg UVM_VERBOSITY=UVM_LOW \
  -testplusarg ntxn=200 \
  -sv_seed 999 \
  -log results/rand_run.log

# Tools
- Vivado / XSim v2025.2 \
- UVM 1.2 (built into XSim) \
- SystemVerilog \


## ALU Specification

| Opcode | Operation | Description |
|--------|-----------|-------------|
| 0 | ADD | result = a + b |
| 1 | SUB | result = a - b |
| 2 | AND | result = a & b |
| 3 | OR  | result = a \| b |
| 4 | XOR | result = a ^ b |
| 5 | SHL | result = a << (b & 7) |
| 6 | SHR | result = a >> (b & 7) |

- 8-bit unsigned operands, overflow wraps (modulo 2⁸)
- 1-cycle registered latency with valid/ready handshake

---

## UVM Components

| File | Role |
|------|------|
| `alu_seq_item` | Transaction (a, b, op, exp_result, act_result) |
| `alu_directed_seq` | Fixed corner-case stimulus |
| `alu_rand_seq` | Randomized stimulus (length controlled by `ntxn`) |
| `alu_driver` | Drives DUT pins, respects in_valid/in_ready |
| `alu_monitor` | Samples output handshake, publishes to scoreboard |
| `alu_scoreboard` | Pure reference model, compares expected vs actual |
| `alu_agent` | Bundles sequencer + driver + monitor |
| `alu_env` | Connects agent → scoreboard via analysis port |
| `alu_smoke_test` | Runs directed sequence |
| `alu_rand_test` | Runs random sequence |

