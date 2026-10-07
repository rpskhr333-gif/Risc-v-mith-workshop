NASSCOM RISC-V MYTH Workshop — Complete Day 4 Notes

Welcome to the comprehensive, structured notes for Day 4 of the NASSCOM RISC-V Microprocessor for You in Thirty Hours (MYTH) Workshop, conducted by VSD (VLSI System Design).

---

MASTER DAY 4 OVERVIEW & OBJECTIVES

Day 4 transitions from fundamental digital circuit design into building a single-cycle RISC-V CPU core microarchitecture using TL-Verilog in the Makerchip IDE.

Key Learning Goals

* Construct a single-cycle RV32I-compliant CPU core datapath step-by-step.
* Implement Instruction Fetch logic including Program Counter (PC) and Instruction Memory.
* Design Instruction Decode logic to extract opcodes, registers (rd, rs1, rs2), and immediate values.
* Build a 32-entry Register File supporting simultaneous dual-read and single-write ports.
* Implement Arithmetic Logic Unit (ALU) operations and Branch execution logic.

---

MODULE 1: SINGLE-CYCLE CPU ARCHITECTURE OVERVIEW

In a single-cycle CPU microarchitecture, every instruction completes its entire execution cycle—from fetch to decode, register read, ALU execution, memory access, and register writeback—within a single clock period.

Core Hardware Blocks

Program Counter (PC) ──► Instruction Memory ──► Instruction Decoder
│
┌────────────────────────────────────────────────────┴───────────────────────────────────┐
│                                                                                        │
▼                                                                                        ▼
Register File (Read rs1, rs2) ──────────────────────────────────────────────────► Arithmetic Logic Unit (ALU)
▲                                                                                        │
│                                                                                        │
└─────────────────────────────── Writeback (rd) ─────────────────────────────────────────┘

Execution Stages Overview

1. Fetch: Read instruction from Instruction Memory at the address specified by PC.
2. Decode: Parse instruction fields (opcode, funct3, funct7, rd, rs1, rs2, immediates).
3. Register Read: Read values from source registers rs1 and rs2.
4. Execute (ALU): Perform arithmetic/logical operation or calculate branch/jump condition.
5. Register Writeback: Write result back to destination register rd.

---

MODULE 2: INSTRUCTION FETCH LOGIC & PROGRAM COUNTER

Program Counter (PC) Logic
The Program Counter holds the memory address of the instruction being executed. In sequential execution, the PC increments by 4 bytes (32 bits) every clock cycle.

TL-Verilog Implementation

\TLV
@0
// Next PC selection: Branch target or sequential increment (+4)
$pc[31:0] = >>1$reset ? 32'b0 :
>>1$taken_branch ? >>1$br_tgt_pc :                   (>>1$pc + 32'd4);

```
  // Fetch 32-bit instruction from Instruction Memory
  $imem_rd_en = !$reset;
  $imem_rd_addr[28:0] =$pc[30:2]; // Word-aligned address

```

---

MODULE 3: INSTRUCTION DECODE LOGIC & IMMEDIATE DECODING

Instruction Type Classification
RV32I instructions fall into distinct formats based on operand encoding:

* R-Type: Register-to-register arithmetic (add, sub, sll, slt, xor, srl, or, and)
* I-Type: Immediate arithmetic and loads (addi, slti, xori, ori, andi, lw)
* S-Type: Store instructions (sw, sb, sh)
* B-Type: Conditional branches (beq, bne, blt, bge, bltu, bgeu)
* U-Type: Upper immediates (lui, auipc)
* J-Type: Unconditional jumps (jal)

Field Extraction

$opcode[6:0] =$instr[6:0];
$rd[4:0]     =$instr[11:7];
$funct3[2:0] =$instr;
$rs1[4:0]    =$instr;
$rs2[4:0]    =$instr;
$funct7[6:0] =$instr;

Instruction Type Decoding Logic

$is_u_instr = $opcode == 7'b0110111 \vert{}\vert{}$opcode == 7'b0010111;
$is_j_instr =$opcode == 7'b1101111;
$is_b_instr =$opcode == 7'b1100011;
$is_s_instr = $opcode == 7'b0100011 \vert{}\vert{}$opcode == 7'b0100011;
$is_i_instr =$opcode == 7'b0000011 || $opcode == 7'b0010011 \vert{}\vert{}$opcode == 7'b1100111;
$is_r_instr =$opcode == 7'b0110011;

Immediate Value Decoding
Immediates are extracted and sign-extended based on instruction format:

$imm[31:0] =$is_i_instr ? {{21{$instr[31]}},$instr} :
$is_s_instr ? {{21{$instr[31]}}, $instr,$instr[11:7]} :
$is_b_instr ? {{20{$instr[31]}}, $instr[7],$instr, $instr[11:8], 1'b0} :$is_u_instr ? {$instr, 12'b0} :$is_j_instr ? {{12{$instr[31]}},$instr, $instr[20],$instr, 1'b0} :
32'b0;

---

MODULE 4: REGISTER FILE READ AND WRITEBACK LOGIC

Register File Specifications

* Contains 32 general-purpose registers (r0 to r31), each 32 bits wide.
* Register r0 is hardwired to zero (writes to r0 are ignored).
* Dual Read Ports: Reads source register data (rf_rd_data1, rf_rd_data2) simultaneously.
* Single Write Port: Writes result (rf_wr_data) to target register (rd) when write-enable is active.

Register File Interface Logic

// Read Enable Control
$rf_rd_en1 =$rs1_valid;
$rf_rd_index1[4:0] =$rs1;

$rf_rd_en2 =$rs2_valid;
$rf_rd_index2[4:0] =$rs2;

// Write Enable Control
$rf_wr_en =$rd_valid && ($rd != 5'b0) &&$valid;
$rf_wr_index[4:0] =$rd;
$rf_wr_data[31:0] =$result;

---

MODULE 5: ARITHMETIC LOGIC UNIT (ALU) & BRANCH EXECUTION

ALU Execution Unit
The ALU performs register-register and register-immediate computations based on instruction decoding:

$result[31:0] = $is_addi ? ($src1_value + $imm) :$is_add  ? ($src1_value +$src2_value) :
$is_sub  ? ($src1_value - $src2_value) :$is_and  ? ($src1_value & $src2_value) :
$is_or   ? ($src1_value | $src2_value) :$is_xor  ? ($src1_value ^ $src2_value) :
32'b0;

Branch Logic Unit
Branch instructions compare operands and modify PC control flow conditionally:

$taken_branch =$is_b_instr && (
($funct3 == 3'b000 && $src1_value ==$src2_value) || // BEQ
($funct3 == 3'b001 && $src1_value !=$src2_value) || // BNE
($funct3 == 3'b100 && ($src1_value <$src2_value)) || // BLT
($funct3 == 3'b101 && ($src1_value >=$src2_value))   // BGE
);

$br_tgt_pc[31:0] = $pc +$imm;

---

MODULE 6: COMPLETE DAY 4 SINGLE-CYCLE CPU TL-VERILOG CODE

\m4_TLV_version 1d: tl-x
\SV
module top(input wire clk, input wire reset, output wire [31:0] passed);
\TLV
|cpu
@0
$pc[31:0] = >>1$reset ? 32'b0 :
>>1$taken_branch ? >>1$br_tgt_pc :                      (>>1$pc + 32'd4);
@1
$imem_rd_en = !$reset;
$imem_rd_addr[28:0] =$pc[30:2];
$instr[31:0] =$imem_rd_data[31:0];

```
     // Decode instruction fields
     $opcode[6:0] =$instr[6:0];
     $rd[4:0]     =$instr[11:7];
     $funct3[2:0] =$instr;
     $rs1[4:0]    =$instr;
     $rs2[4:0]    =$instr;
     
     // Instruction classification
     $is_i_instr =$opcode == 7'b0010011;
     $is_r_instr =$opcode == 7'b0110011;
     $is_b_instr =$opcode == 7'b1100011;
     
     // Immediate decoding
     $imm[31:0] = $is_i_instr ? {{21{$instr[31]}}, $instr} :$is_b_instr ? {{20{$instr[31]}},$instr[7], $instr,$instr[11:8], 1'b0} :
                  32'b0;
                  
     // ALU and Branch execution
     $src1_value[31:0] = $rf_rd_data1[31:0];$src2_value[31:0] = $rf_rd_data2[31:0];$result[31:0] = $is_addi ? ($src1_value + $imm) :$is_add  ? ($src1_value +$src2_value) : 32'b0;
                     
     $taken_branch = $is_b_instr && ($src1_value == $src2_value);$br_tgt_pc[31:0] = $pc +$imm;

```

\SV
endmodule

---

SUMMARY CHECKLIST

1. Fetch Unit: Increments Program Counter sequentially (+4) or branches to target PC based on control signals.
2. Decode Unit: Extracts register specifiers, opcodes, function codes, and generates sign-extended immediates.
3. Register File: Features dual-read ports for source operands and a single write port with register r0 write-inhibit protection.
4. ALU & Branch Logic: Computes target results and determines branch conditions within a single-cycle execution path.