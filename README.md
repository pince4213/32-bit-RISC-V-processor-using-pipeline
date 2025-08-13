# RISC-V 32-Bit Processor README

## Overview

This project is a 32-bit RISC-V processor designed using Verilog. It is a pipelined processor, which means it processes instructions in five steps to make it faster. The processor can run basic RISC-V instructions like addition, subtraction, shifting, branching, and memory operations. It’s make to handle instructions stored in a file called `instruction.mem` and includes features to avoid problems like data conflicts during execution.

The main file, `RISC32_bit.v`, connects smaller modules that handle different tasks, such as fetching instructions, decoding them, performing calculations, accessing memory, and writing results back to registers. This README explains how the processor works, how to simulate it, and what the provided instructions do, all in simple terms.

## What is a RISC-V Processor?

RISC-V is a type of computer architecture that uses simple instructions to tell the processor what to do. Our processor is a 32-bit version, meaning it works with 32-bit numbers and instructions. It is designed to:

- Fetch instructions from memory.
- Decode what each instruction means.
- Execute calculations like adding numbers or jumping to other instruction.
- Access memory to read or write data.
- Save results back to registers (small storage units inside the processor).

## Processor Structure

The processor uses a **5-stage pipeline** to process instructions faster by working on multiple instructions at once. The five stages are:

1. **Fetch**: Grabs the next instruction from memory using the Program Counter (PC).
2. **Decode**: Figures out what the instruction does and prepares data.
3. **Execute**: Does the math or logic (e.g., addition, comparison).
4. **Memory**: Reads or writes data to memory if needed.
5. **Write-Back**: Saves the result to a register.

The processor includes **hazard handling** to avoid mistakes when instructions depend on each other, like waiting for a result before using it.

### Key Components

The processor is made up of several smaller modules. Here what the main parts:

- **Address Generator (RI5)**: Updates the PC to point to the next instruction or a branch/jump address.
- **Instruction Memory (RI10)**: Stores and provides instructions from `instruction.mem`.
- **PCPlus4 (RI11)**: Adds 4 to the PC (since each instruction is 4 bytes).
- **Instruction Fetch (RI9)**: Breaks down instructions into parts (like register numbers or immediate values).
- **Register File (RI13)**: A storage bank with 32 registers (x0 to x31) to hold data.
- **Sign Extend (RI23)**: Extends small numbers (immediates) to 32 bits.
- **Controller (RI6)**: Decides what each instruction should do like add, store, branch.
- **ALU (RI14)**: Performs math and logic like add, subtract, XOR, shift.
- **Data Memory (RI7)**: Stores and retrieves data during load (`lw`) or store (`sw`) instructions.
- **Hazard Unit (RI8)**: Prevents errors by forwarding data or pausing the pipeline.
- **Pipeline Registers (RI1 to RI4)**: Pass data between stages (Fetch to Decode, Decode to Execute, etc.).
- **Multiplexers (RI15, RI16, RI21, RI22)**: Choose the right data (register value or immediate) for calculations or results.

The `RISC32_bit.v` file connects all these parts to form the complete processor.

## Provided Instructions

The processor runs instructions from `instruction.mem`, which contains:

```
01450513  // addi x10, x10, 20  (adds 20 to register x10)
00a58593  // addi x11, x11, 10  (adds 10 to x11)
00060613  // addi x12, x12, 0   (no change to x12)
00068693  // addi x13, x13, 0   (no change to x13)
00170713  // addi x14, x14, 1   (adds 1 to x14)
01f78793  // addi x15, x15, 31  (adds 31 to x15)
02d61c63  // bne x12, x13, 56   (if x12 ≠ x13, jump to PC+56; here, no jump since x12 = x13 = 0)
40b508b3  // sub x10, x10, x11  (subtracts x11 from x10)
00f8d8b3  // srl x17, x17, x15  (shifts x17 right by x15)
00e8c8b3  // xor x17, x17, x14  (XORs x17 with x14)
00d89a63  // bne x17, x13, 20   (if x17 ≠ x13, jump to PC+20; jumps to PC=60)
00a68833  // add x16, x13, x10  (skipped due to branch)
00b68533  // add x10, x13, x11  (skipped)
010685b3  // add x11, x13, x16  (skipped)
fed680e3  // beq x13, x13, -20  (skipped)
00d59663  // bne x11, x13, 12   (skipped)
00160613  // addi x12, x12, 1   (skipped)
fcd68ae3  // beq x13, x12, -52  (skipped)
40b50533  // sub x10, x10, x11  (skipped)
fcd686e3  // beq x13, x12, -20  (skipped)
00a688b3  // add x17, x13, x10  (adds x13 and x10, stores in x17)
01142023  // sw x17, 0(x8)      (stores x17 to memory at address in x8)
```

### What These Instructions Do

Starting with all registers at 0 (except x28=6, x22=4, x18=6 due to `Register_File` settings):

 1. **PC=0**: Sets x10 = 20.
 2. **PC=4**: Sets x11 = 10.
 3. **PC=8**: Sets x12 = 0.
 4. **PC=12**: Sets x13 = 0.
 5. **PC=16**: Sets x14 = 1.
 6. **PC=20**: Sets x15 = 31.
 7. **PC=24**: Checks if x12 ≠ x13. Since x12 = x13 = 0, continues to PC=28.
 8. **PC=28**: Subtracts x11 (10) from x10 (20), so x10 = 10.
 9. **PC=32**: Shifts x17 (0) right by x15 (31), so x17 = 0.
10. **PC=36**: XORs x17 (0) with x14 (1), so x17 = 1.
11. **PC=40**: Checks if x17 ≠ x13. Since x17 = 1 and x13 = 0, jumps to PC=60.
12. **PC=60**: Adds x13 (0) and x10 (10), so x17 = 10.
13. **PC=64**: Stores x17 (10) to memory at address x8 (6, hard-coded), so `Data_Mem[6] = 10`.

**Final State**:

- Registers: x10 = 10, x11 = 10, x12 = 0, x13 = 0, x14 = 1, x15 = 31, x17 = 10, x28 = 6, x22 = 4, x18 = 6.
- Memory: `Data_Mem[6] = 10`.

## How to Simulate the Processor

To see how the processor runs these instructions, you need to simulate it using a tool like ModelSim or QuestaSim. Here’s how:

### 1. Prepare Files

- **Verilog Files**: Use `RISC32_bit.v` (main module) and submodules (RI1.v to RI20.v).
- **Instruction Memory**: Save the instructions in a file called `instruction.mem`:

  ```
  01450513
  00a58593
  00060613
  00068693
  00170713
  01f78793
  02d61c63
  40b508b3
  00f8d8b3
  00e8c8b3
  00d89a63
  00a68833
  00b68533
  010685b3
  fed680e3
  00d59663
  00160613
  fcd68ae3
  40b50533
  fcd686e3
  00a688b3
  01142023
  ```
- **Testbench**: Create a file `tb_RISC32_bit.v` to drive the processor and monitor signals (provided below).

### 2. Testbench

The testbench sets up the clock, reset, and logs signals to a file (`simulation_output.txt`). Here’s a simple version:

```verilog
`timescale 1ns/1ps

module tb_RISC32_bit;
    reg clk, rst;
    wire [31:0] checkx1, checkx2, checkx3, checkx4, checkx5, checkx6, DM0, instruction;
    wire [31:0] PCF, PCPlus4F, instrD, PCD, PCPlus4D, SrcAE, ResultW, RD1, RD2, ImmExtD;
    wire [31:0] RD1E, RD2E, PCE, ImmExtE, PCPlus4E, PCTargetE, SrcBE, ALUResult;
    wire [31:0] ALUResultM, ALUResultW, WriteDataM, PCPlus4M, PCPlus4W, ReadData, ReadDataW, WriteDataE;
    wire [4:0] A1, A2, RdD, RdW, RdE, RdM, Rs1E, Rs2E, Rs1D, Rs2D, ALUControlD, ALUControlE;
    wire [6:0] OP, funct77;
    wire [2:0] funct3, funct3E, ImmSrcD;
    wire [1:0] ResultSrcD, ResultSrcE, ResultSrcM, ResultSrcW, ForwardAE, ForwardBE;
    wire [24:0] Imm;
    wire funct7, WE3, RegWriteW, RegWriteD, MemWriteD, JumpD, BranchD, ALUSrcD;
    wire ZeroE, RegWriteE, MemWriteE, JumpE, BranchE, ALUSrcE, PCSrcE, CarryOut, RegWriteM, MemWriteM;
    wire StallF, StallD, FlushE, FlushD;

    RISC32_bit uut (
        .clk(clk), .rst(rst), .checkx1(checkx1), .checkx2(checkx2), .checkx3(checkx3),
        .checkx4(checkx4), .checkx5(checkx5), .checkx6(checkx6), .DM0(DM0), .instruction(instruction),
        .PCF(PCF), .PCPlus4F(PCPlus4F), .instrD(instrD), .PCD(PCD), .PCPlus4D(PCPlus4D),
        .SrcAE(SrcAE), .A1(A1), .A2(A2), .RdD(RdD), .RdW(RdW), .RdE(RdE), .RdM(RdM),
        .Rs1E(Rs1E), .Rs2E(Rs2E), .Rs1D(Rs1D), .Rs2D(Rs2D), .OP(OP),
        .funct3(funct3), .funct3E(funct3E), .funct7(funct7), .WE3(WE3),
        .RegWriteW(RegWriteW), .RegWriteD(RegWriteD), .MemWriteD(MemWriteD),
        .JumpD(JumpD), .BranchD(BranchD), .ALUSrcD(ALUSrcD), .ZeroE(ZeroE),
        .RegWriteE(RegWriteE), .MemWriteE(MemWriteE), .JumpE(JumpE), .BranchE(BranchE),
        .ALUSrcE(ALUSrcE), .PCSrcE(PCSrcE), .Imm(Imm), .funct77(funct77),
        .ResultW(ResultW), .RD1(RD1), .RD2(RD2), .ImmExtD(ImmExtD), .ImmSrcD(ImmSrcD),
        .ResultSrcD(ResultSrcD), .ResultSrcE(ResultSrcE), .ResultSrcM(ResultSrcM),
        .ResultSrcW(ResultSrcW), .ALUControlD(ALUControlD), .ALUControlE(ALUControlE),
        .RD1E(RD1E), .RD2E(RD2E), .PCE(PCE), .ImmExtE(ImmExtE), .PCPlus4E(PCPlus4E),
        .PCTargetE(PCTargetE), .SrcBE(SrcBE), .ALUResult(ALUResult), .ALUResultM(ALUResultM),
        .ALUResultW(ALUResultW), .WriteDataM(WriteDataM), .PCPlus4M(PCPlus4M),
        .PCPlus4W(PCPlus4W), .CarryOut(CarryOut), .RegWriteM(RegWriteM),
        .MemWriteM(MemWriteM), .ReadData(ReadData), .ReadDataW(ReadDataW),
        .WriteDataE(WriteDataE), .ForwardAE(ForwardAE), .ForwardBE(ForwardBE),
        .StallF(StallF), .StallD(StallD), .FlushE(FlushE), .FlushD(FlushD)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0; rst = 1;
        #10 rst = 0;
        #500 $finish;
    }

    integer file;
    initial begin
        file = $fopen("simulation_output.txt", "w");
        $fmonitor(file, "Time=%0t rst=%b PCF=%h instruction=%h checkx1=%h checkx2=%h checkx3=%h checkx4=%h checkx5=%h checkx6=%h DM0=%h ALUResult=%h ResultW=%h",
                  $time, rst, PCF, instruction, checkx1, checkx2, checkx3, checkx4, checkx5, checkx6, DM0, ALUResult, ResultW);
        #500;
        $fdisplay(file, "\nFinal Register File Contents:");
        for (int i = 0; i < 32; i = i + 1)
            $fdisplay(file, "x%0d: %h", i, uut.i_rf.Registers[i]);
        $fdisplay(file, "\nData Memory[0]: %h", uut.i_dm.Data_Mem[0]);
        $fdisplay(file, "Data Memory[6]: %h", uut.i_dm.Data_Mem[6]);
        $fclose(file);
    end
endmodule
```

### 3. Simulation Steps (Using ModelSim/QuestaSim)

1. **Create a Project**:
   - Open ModelSim/QuestaSim.
   - Go to `File > New > Project`.
   - Add all Verilog files (`RISC32_bit.v`, `tb_RISC32_bit.v`, RI1.v to RI23.v, except RI20.v).
2. **Compile**:
   - Select all files and click `Compile > Compile All`.
   - Ensure `instruction.mem` is in the project directory.
3. **Start Simulation**:
   - Select `tb_RISC32_bit` and click `Simulate > Start Simulation`.
4. **Add Signals**:
   - In the “Objects” window, find signals under `sim:/tb_RISC32_bit/uut/`.
   - Add all signals (e.g., `PCF`, `checkx1` to `checkx6`, `instruction`, `ALUResult`) using:

     ```tcl
     add wave sim:/tb_RISC32_bit/uut/*
     ```
5. **Run Simulation**:
   - Type `run 500 ns` in the console or use the GUI “Run” button.
6. **View Results**:
   - **Waveforms**: Look at the waveform window to see how signals like `PCF` (Program Counter), `checkx5` (x10), and `checkx6` (x11) change over time.
   - **Text Output**: Open `simulation_output.txt` to see logged values.
   - **Registers and Memory**: Check `sim:/tb_RISC32_bit/uut/i_rf/Registers` and `sim:/tb_RISC32_bit/uut/i_dm/Data_Mem` in the memory viewer.

### Expected Results

After running the simulation for 500 ns:

- **Registers**:
  - x10 (`checkx5`): 10 (after `sub x10, x10, x11`).
  - x11 (`checkx6`): 10.
  - x12: 0.
  - x13: 0.
  - x14: 1.
  - x15: 31.
  - x17: 10 (after `add x17, x13, x10`).
  - x18 (`checkx3`): 6 (hard-coded).
  - x22: 4 (hard-coded).
  - x28: 6 (hard-coded).
- **Memory**:
  - `Data_Mem[6]`: 10 (from `sw x17, 0(x8)`).
  - `DM0`: Undefined (`X`) unless `Data_Memory` is modified to assign `DM0 = Data_Mem[0]`.
- **Program Counter (**`PCF`**)**:
  - 0, 4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 60, 64 (jumps to 60 due to `bne`).
- **Key Signals**:
  - `instruction`: Shows the current instruction (e.g., `32'h01450513` at PC=0).
  - `ALUResult`: Shows calculation results (e.g., `32'h0000000a` for `sub x10, x10, x11`).
  - `ResultW`: Final result written to registers.
  - `ZeroE`: 1 for `bne x12, x13` (equal), 0 for `bne x17, x13` (not equal).
  - `PCSrcE`: 1 during the branch at PC=40.

### Notes and Tips

- **DM0 Issue**: The `DM0` signal is undefined (`X`) because it’s not set in `Data_Memory`. To fix, add `DM0 = Data_Mem[0];` in `Data_Memory`’s `always` block.
- **Hard-Coded Registers**: The `Register_File` sets x28=6, x22=4, x18=6, which affects `checkx3` and the store address (x8=6). Comment out these lines in `RI13.v` for normal behavior:

  ```verilog
  // Registers[28] = 32'd6;
  // Registers[22] = 32'd4;
  // Registers[18] = 32'd6;
  ```
- **Simulation Time**: 500 ns covers the executed instructions. Increase if you want to test loops (e.g., `beq` instructions).
- **Visualization**: To see a graph of signals like `PCF` or `checkx5`, check `simulation_output.txt` and share the data for a chart.


## Next Steps

- Run the simulation and check simulation waveforms.
- If you see unexpected results, share the log file or waveform screenshots.
- To test more instructions or loops, extend the simulation time or modify `instruction.mem`.
- For a visual graph of signals, share simulation data, and I can create a chart.

This processor is a great way to learn about RISC-V and pipelined designs.
