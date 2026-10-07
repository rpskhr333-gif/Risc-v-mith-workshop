NASSCOM RISC-V MYTH Workshop — Complete Day 2 Notes

Welcome to the comprehensive, structured notes for Day 2 of the NASSCOM RISC-V Microprocessor for You in Thirty Hours (MYTH) Workshop, conducted by VSD (VLSI System Design).

---

MASTER DAY 2 OVERVIEW & OBJECTIVES

Day 2 bridges the gap between software execution and processor register hardware by exploring the Application Binary Interface (ABI), register usage conventions, assembly language fundamentals, and memory allocation structures.

Key Learning Goals

* Understand the role of ABI as the contract between application code, OS, and processor hardware.
* Master RISC-V register allocation, naming conventions, and caller/callee save responsibilities.
* Trace memory layout: Text, Data, Heap, and Stack segments.
* Interface C programs directly with hand-written RISC-V assembly functions.
* Perform load, store, and stack pointer operations.

---

MODULE 1: INTRODUCTION TO APPLICATION BINARY INTERFACE (ABI)

What is an ABI?
An Application Binary Interface (ABI) defines the low-level binary contract between application software and the underlying hardware platform (or operating system).

API vs. ABI Difference

* Application Programming Interface (API): Source-code level contract (functions, data types, header files defined in C/C++).
* Application Binary Interface (ABI): Binary/hardware-level contract (register usage, calling conventions, stack layout, data alignment, system call interface).

Why ABI Matters

* Ensures compiled object code from different compilers or libraries can link together seamlessly.
* Standardizes how parameters are passed into functions via CPU registers.
* Ensures system calls interact properly with the operating system kernel.

---

MODULE 2: RISC-V REGISTER ARCHITECTURE & ABI NAMES

RISC-V base integer architectures (RV32I / RV64I) feature 32 general-purpose registers (x0 to x31). To establish software consistency, each register is assigned a specific ABI name and dedicated operational rule.

RISC-V General-Purpose Register Map (x0 - x31)

| Register | ABI Name | Description | Saver Rule |
| --- | --- | --- | --- |
| x0 | zero | Hardwired Zero | N/A |
| x1 | ra | Return Address | Caller |
| x2 | sp | Stack Pointer | Callee |
| x3 | gp | Global Pointer | N/A |
| x4 | tp | Thread Pointer | N/A |
| x5 - x7 | t0 - t2 | Temporary Registers | Caller |
| x8 | s0 / fp | Saved Register / Frame Pointer | Callee |
| x9 | s1 | Saved Register | Callee |
| x10 - x11 | a0 - a1 | Function Arguments / Return Values | Caller |
| x12 - x17 | a2 - a7 | Function Arguments | Caller |
| x18 - x27 | s2 - s11 | Saved Registers | Callee |
| x28 - x31 | t3 - t6 | Temporary Registers | Caller |

Key Register Designations

* x0 (zero): Always reads 0; writes to it are disregarded by hardware.
* x1 (ra): Stores return address when entering functions via jal / jalr instructions.
* x2 (sp): Points to the top of the runtime stack (grows downwards toward lower memory addresses).
* x10 - x17 (a0 - a7): Used to pass up to 8 arguments into functions. x10 (a0) and x11 (a1) hold function return values.

---

MODULE 3: CALLER VS. CALLEE SAVED REGISTERS

When Function A (Caller) invokes Function B (Callee), processor registers must be managed to prevent register data corruption.

Caller-Saved Registers (t0-t6, a0-a7, ra)

* Saved by the calling function onto the stack before calling a nested function if the values are needed after the call.
* The called function is free to overwrite these registers without preserving them.

Callee-Saved Registers (s0-s11, sp)

* Preserved by the called function. If Function B needs to use registers s0 through s11, it must push their original values onto the stack upon entry and restore them before returning.

---

MODULE 4: MEMORY MAP & RUNTIME STACK MANAGEMENT

Processor Memory Layout
When a program runs, its virtual address space is organized into specific segments:

High Memory Address
┌─────────────────────────┐
│          Stack          │  ↓ Grows downwards (decrements sp)
├─────────────────────────┤
│            │            │
│            ▼            │
│            ▲            │
│            │            │
├─────────────────────────┤
│          Heap           │  ↑ Grows upwards (dynamic memory, malloc)
├─────────────────────────┤
│      BSS Segment        │  Uninitialized global/static variables
├─────────────────────────┤
│      Data Segment       │  Initialized global/static variables
├─────────────────────────┤
│      Text Segment       │  Binary machine instructions (read-only)
└─────────────────────────┘
Low Memory Address

Stack Operations

* Stack pointer (sp / x2) must stay aligned to 16-byte boundaries in RISC-V ABI.
* Pushing onto stack: Decrement sp, then store data (sd / sw).
* Popping from stack: Load data (ld / lw), then increment sp.

Example Assembly Stack Allocation (RV64):

# Allocate 16 bytes on stack

addi sp, sp, -16

# Store return address (ra) and saved register (s0)

sd ra, 8(sp)
sd s0, 0(sp)

# ... [Function Execution Body] ...

# Restore registers

ld s0, 0(sp)
ld ra, 8(sp)

# Deallocate stack frame

addi sp, sp, 16
ret

---

MODULE 5: C AND ASSEMBLY INTERFACING (HANDS-ON LAB)

Combining C source files with standalone RISC-V assembly functions allows performance-critical routines to run with direct hardware control.

Case Study: Load Array Sum via External Assembly Routine

1. Main C Source File (main.c)

#include <stdio.h>

// External assembly function declaration
extern long long int load(long long int* array, int count);

int main() {
long long int array[5] = {10, 20, 30, 40, 50};
long long int result = load(array, 5);

```
printf("Total sum returned from Assembly: %lld\n", result);
return 0;

```

}

2. RISC-V Assembly Source File (load.S)

.global load
.type load, @function

# Argument Mapping per ABI:

# a0 (x10) = pointer to base array address

# a1 (x11) = count parameter (5)

load:
addi a2, zero, 0      # a2 = running accumulator sum (set to 0)
addi a3, zero, 0      # a3 = loop index i (set to 0)

loop:
bge a3, a1, done      # if index i >= count, jump to done

```
# Load 64-bit integer from memory address [base + offset]
# Memory offset = i * 8 bytes (since each long long int is 8 bytes)
slli a4, a3, 3        # a4 = i * 8 (shift left logical by 3)
add a5, a0, a4        # a5 = base_address + offset
ld a6, 0(a5)          # load double-word (64-bit) from address a5 into a6

add a2, a2, a6        # sum = sum + array[i]
addi a3, a3, 1        # i = i + 1
j loop                # jump back to loop head

```

done:
add a0, a2, zero      # Move total sum into return register a0 per ABI
ret                   # Return to calling program (jalr zero, ra, 0)

3. Compilation Command

riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o custom_load.o main.c load.S
spike pk custom_load.o

---

MODULE 6: CORE RISC-V INSTRUCTION SET SUMMARY (DAY 2 LABS)

Data Transfer Instructions

* ld rd, offset(rs1): Load Doubleword (64-bit value from memory into rd).
* lw rd, offset(rs1): Load Word (32-bit sign-extended value into rd).
* sd rs2, offset(rs1): Store Doubleword (64-bit value from rs2 to memory).
* sw rs2, offset(rs1): Store Word (32-bit value from rs2 to memory).

Arithmetic & Logical Instructions

* add rd, rs1, rs2: rd = rs1 + rs2
* addi rd, rs1, imm: rd = rs1 + immediate_value
* slli rd, rs1, shamt: Shift Left Logical Immediate (multiplication by 2^shamt).

Control & Jump Instructions

* bge rs1, rs2, label: Branch to label if rs1 >= rs2.
* bne rs1, rs2, label: Branch to label if rs1 != rs2.
* ret: Return from subroutine (expands to jalr x0, x1, 0).

---

SUMMARY CHECKLIST

1. Application Binary Interface (ABI): Defines binary contracts including register naming, calling conventions, and stack management.
2. Register Utilization: Function parameters are passed via a0-a7; return values are returned in a0-a1; callers/callees manage stack preservation for temporary vs. saved registers.
3. C-Assembly Interfacing: Allows C applications to invoke assembly routines directly using standard ABI parameter registers (a0, a1, etc.).
4. Memory Management: Array processing relies on load/store instructions (ld, sd) combined with address offset generation (slli) to access RAM locations.