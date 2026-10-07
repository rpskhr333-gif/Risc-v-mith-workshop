NASSCOM RISC-V MYTH Workshop — Complete Day 1 Notes

Welcome to the comprehensive, structured notes for Day 1 of the NASSCOM RISC-V Microprocessor for You in Thirty Hours (MYTH) Workshop, conducted by VSD (VLSI System Design).

---

MASTER COURSE OVERVIEW & LEARNING ROADMAP

The MYTH workshop offers a practical, bottom-up approach to computer architecture—tracing the complete path from high-level software application design down to digital hardware logic and CPU pipelining.

Software Application (C/C++)
│
Assembly Language
│
Instruction Set Architecture (ISA)
│
Digital Logic (TL-Verilog)
│
CPU Datapath Design
│
5-Stage Pipelined RISC-V CPU

5-Day Workshop Agenda

* Day 1: RISC-V ISA, GNU Compiler Toolchain, and Number Systems
* Day 2: Application Binary Interface (ABI) and Verification Flow
* Day 3: Digital Logic Design using TL-Verilog and Makerchip IDE
* Day 4: Basic RISC-V CPU Microarchitecture Construction
* Day 5: Complete Pipelined RISC-V CPU with Hazard Handling

---

MODULE 1: WORKSHOP INTRODUCTION & OBJECTIVES

Key Goals

* Master the core concepts of the open-source RISC-V ISA.
* Understand the full software-to-hardware compilation and execution pipeline.
* Design and simulate digital logic using TL-Verilog and Makerchip IDE.
* Construct and verify a custom 5-stage pipelined RISC-V CPU core.

Core Toolchain Summary

* GNU Compiler Toolchain: Cross-compilation framework (riscv64-unknown-elf-gcc).
* Spike Simulator: The official RISC-V ISA golden reference simulator.
* TL-Verilog & Makerchip IDE: Hardware description language and browser-based design platform featuring timing abstraction and pipeline visualization.
* Icarus Verilog: Simulation engine for Verilog logic verification.

---

MODULE 2: FROM APPLICATIONS TO HARDWARE EXECUTION

This section details how high-level program code is translated into low-level electrical execution inside CPU hardware.

The Software-to-Hardware Execution Stack

```
   Applications (C, C++, Python)
                │
    Operating System (Linux, macOS, Windows)
                │
      Compiler (GCC, LLVM)
                │
 Assembly Language (ISA Instructions)
                │

```

Instruction Set Architecture (ISA Interface)
│
Hardware Execution (ALU, Registers, Logic Gates)

1. Applications: High-level programs containing logic, data structures, and algorithms.
2. Operating System: Manages system resources, task scheduling, memory mapping, and device drivers.
3. Compiler: Parses source code, optimizes execution paths, and translates it into hardware-specific assembly (e.g., GCC).
4. Assembly Language: Human-readable symbolic representation of binary machine instructions (e.g., add x5, x1, x2).
5. Instruction Set Architecture (ISA): The standard interface/contract defining register sets, instruction encodings, and addressing modes.
6. Hardware: Digital logic (ALUs, control units, multiplexers, flip-flops) executing binary instructions.

Why Choose RISC-V?

* Open Source: Completely royalty-free without proprietary licensing constraints.
* Modular Architecture: Core Base Integer ISA (RV32I / RV64I) extended via modular extensions (e.g., M = Multiplication, A = Atomic, F = Floating point, C = Compressed).
* Industry Standard: Broad adoption across global semiconductor design, research, and production.

---

MODULE 3: C PROGRAMMING EXAMPLE (SUM 1 TO N)

To analyze software-to-hardware translation, a simple C program computing the sum of numbers from 1 to N is used as a baseline example.

Mathematical Definition
Sum = 1 + 2 + 3 + ... + N

C Source Code (sum1ton.c)

#include <stdio.h>

int main() {
int i, sum = 0, n = 5;
for (i = 1; i <= n; i++) {
sum = sum + i;
}
printf("Sum from 1 to %d is %d\n", n, sum);
return 0;
}

Execution Trace Table (N = 5)

| Iteration | Loop Variable (i) | Running Total (sum) |
| --- | --- | --- |
| Initial | 1 | 0 |
| 1 | 1 | 1 |
| 2 | 2 | 3 |
| 3 | 3 | 6 |
| 4 | 4 | 10 |
| 5 | 5 | 15 |

---

MODULE 4: COMPILATION AND DISASSEMBLY FLOW

Compiling with RISC-V GCC
Cross-compile the baseline C code for the 64-bit RISC-V base integer target:

riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c

Compilation Command Flags Breakdown

| Flag | Description |
| --- | --- |
| -O1 | Applies basic code optimizations (balances size and speed). |
| -mabi=lp64 | Sets the 64-bit ABI (long integers and pointers are 64-bit). |
| -march=rv64i | Target architecture set to RV64I (64-bit Base Integer ISA). |
| -o | Specifies output binary file name. |

Disassembling Executables
Convert compiled binary machine code back into inspectable assembly code:

riscv64-unknown-elf-objdump -d sum1ton.o | less

C to Assembly Translation Mapping

* Loop Initialization: Loading initial loop counts and bounds into registers (addi).
* Accumulation Logic: Arithmetic register-to-register additions (add / addw).
* Loop Control & Branches: Conditional branching operations (bne, blt, j).

---

MODULE 5: SPIKE SIMULATOR AND INTERACTIVE DEBUGGING

What is Spike?
Spike is the official golden reference ISA simulator for RISC-V. It allows functional instruction-level verification before target hardware implementation.

Running Binaries in Spike

spike pk sum1ton.o

Note: pk (Proxy Kernel) is a minimal kernel handling basic system calls (such as printf) made by the standard library.

Interactive Debug Mode
Step through instructions one by one to inspect processor internal states:

spike -d pk sum1ton.o

Essential Spike Debugger Commands

| Command | Action |
| --- | --- |
| reg 0 <reg_name> | Print contents of a specific register (e.g., reg 0 a0). |
| pc | Output current Program Counter address. |
| until pc  | Execute continuously until Program Counter hits target. |
| [Enter] / Step | Single-step execution (run next single instruction). |
| q | Terminate debug session. |

---

MODULE 6: UNSIGNED NUMBER SYSTEMS

Digital processors process all data using binary digits (bits), where each bit stores either 0 or 1.

Binary Positional Notation
An N-bit binary number represents summed powers of 2:
Decimal Value = Sum of (bit_k * 2^k) for k from 0 to N-1

4-Bit Unsigned Place Weight Matrix

| Bit Position | 2^3 | 2^2 | 2^1 | 2^0 |
| --- | --- | --- | --- | --- |
| Weight | 8 | 4 | 2 | 1 |

Conversion Example: 1011 (binary) -> Decimal
Value = (1 * 8) + (0 * 4) + (1 * 2) + (1 * 1) = 8 + 0 + 2 + 1 = 11

Unsigned Number Ranges

* Minimum Value: 0
* Maximum Value: (2^N) - 1

| Bit Width (N) | Minimum | Maximum |
| --- | --- | --- |
| 4-bit | 0 | 15 |
| 8-bit | 0 | 255 |
| 16-bit | 0 | 65,535 |
| 32-bit | 0 | 4,294,967,295 |
| 64-bit | 0 | 18,446,744,073,709,551,615 (~1.84 x 10^19) |

---

MODULE 7: SIGNED NUMBERS & 2'S COMPLEMENT

Sign Bit (MSB)
For signed binary numbers, the Most Significant Bit (MSB) defines sign polarity:

* MSB = 0: Positive Number
* MSB = 1: Negative Number

2's Complement Arithmetic
Modern hardware uses 2's complement representation because subtraction can be performed using standard binary addition logic without dedicated subtraction circuits.

Converting Positive Values to Negative (2's Complement):

1. Write down the positive magnitude in binary.
2. Invert all bits (0 -> 1, 1 -> 0).
3. Add 1 to the Least Significant Bit (LSB).

Conversion Example (+5 to -5 in 4-bit):

1. +5  = 0101
2. Invert bits: 1010
3. Add 1: 1010 + 1 = 1011
Result: -5 = 1011

Signed Number Ranges

* Minimum Value: -(2^(N-1))
* Maximum Value: (2^(N-1)) - 1

| Bit Width (N) | Minimum | Maximum |
| --- | --- | --- |
| 4-bit | -8 | +7 |
| 8-bit | -128 | +127 |
| 16-bit | -32,768 | +32,767 |
| 32-bit | -2,147,483,648 | +2,147,483,647 |
| 64-bit | -9,223,372,036,854,775,808 | +9,223,372,036,854,775,807 |

---

MODULE 8: LAB ANALYSIS — ARITHMETIC OVERFLOW & DATA TYPES

Understanding Arithmetic Overflow
Overflow occurs when an arithmetic calculation yields a value outside the range that the target data type can store, resulting in bit truncation and signed value distortion.

Lab Study: Data Type Overflow Analysis

Faulty Implementation (Explicit Truncation Error):

long long int max = (long long int) (int) (pow(2, 63) - 1);
long long int min = (long long int) (int) (pow(2, 63) * -1);

Why it fails:
Explicitly casting the result to (int) forces a 64-bit calculation down into a standard 32-bit integer boundary. The upper 32 bits are truncated, causing overflow and corrupting the expected output:

* Expected Max Value: 9223372036854775807
* Corrupted Output: -1 or -2147483648

Corrected C Code:

#include <stdio.h>
#include <math.h>

int main() {
long long int max = (long long int) (pow(2, 63) - 1);
long long int min = (long long int) (pow(2, 63) * -1);

```
printf("Highest 64-bit signed integer value: %lld\n", max);
printf("Lowest 64-bit signed integer value: %lld\n", min);

return 0;

```

}

Correct Terminal Output:

Highest 64-bit signed integer value: 9223372036854775807
Lowest 64-bit signed integer value: -9223372036854775808

---

SUMMARY CHECKLIST

1. Hardware-Software Interface: High-level C code is translated by GCC compilers into assembly instructions defined by the RISC-V ISA, which run on physical CPU hardware.
2. Simulation & Inspection Tools: Software execution on RISC-V targets can be modeled and debugged instruction-by-instruction using spike and pk.
3. Data Representation: Understanding hardware representation limits (signed/unsigned ranges and 2's complement logic) is critical for preventing runtime overflow issues in both software and ALU design.