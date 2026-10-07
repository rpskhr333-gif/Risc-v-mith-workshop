NASSCOM RISC-V MYTH Workshop — Complete 5-Day Master Notes
Welcome to the consolidated master notes for all 5 days of the NASSCOM RISC-V Microprocessor for You in Thirty Hours (MYTH) Workshop, conducted by VSD (VLSI System Design).
WORKSHOP OVERVIEW & LEARNING ROADMAP
| Phase | Days | Core Focus | Tools Used |
|---|---|---|---|
| Software Abstraction | Day 1 & Day 2 | RISC-V ISA, C-to-Assembly Compilation, ABI, System Calls | GCC, Spike Simulator, PK (Proxy Kernel), Objdump |
| Digital Design Foundations | Day 3 | Transaction-Level Verilog (TL-Verilog), Combinational/Sequential Circuits, Pipelining | Makerchip IDE, SandPiper Compiler, VIZ |
| Microarchitecture | Day 4 & Day 5 | Single-Cycle & 5-Stage Pipelined RV32I Core, Hazard Handling, Forwarding | Makerchip IDE, TL-Verilog |
DAY 1: RISC-V ISA & C-TO-ASSEMBLY COMPILATION FLOW
Key Concepts
 * Software to Hardware Interface: C/C++ Code \rightarrow Assembly Language \rightarrow Machine Code (Binary) \rightarrow Hardware Execution.
 * GNU Toolchain Flow: riscv64-unknown-elf-gcc compiles C programs into RISC-V ELF executables.
 * RISC-V ISA Variants: RV32I (32-bit base integer, 32 registers), RV64I (64-bit base integer), with extensions: M (Multiply/Divide), A (Atomic), F (Single-Precision Float), D (Double-Precision Float), C (Compressed).
Key Commands & Workflows
# Compile C code targeting RV64I architecture with ABI lp64
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1num.o sum1num.c

# Disassemble binary to inspect assembly instructions
riscv64-unknown-elf-objdump -d sum1num.o | less

# Simulate execution using Spike ISA simulator with Proxy Kernel (PK)
spike pk sum1num.o

DAY 2: APPLICATION BINARY INTERFACE (ABI) & FUNCTION CALLS
Key Concepts
 * Register Conventions (32 Registers):
   * x0 / zero: Hardwired zero
   * x1 / ra: Return address
   * x2 / sp: Stack pointer
   * x8-x9, x18-x27 / s0-s11: Saved registers (preserved across calls)
   * x10-x17 / a0-a7: Function arguments / return values
   * x5-x7, x28-x31 / t0-t6: Temporary registers
 * ABI Role: Defines rules for how applications interact with the OS/hardware and how functions pass arguments/return values via registers instead of stack memory.
C & RISC-V Assembly Mapping Example
// C Function
int custom_add(int a, int b) {
    return a + b;
}

# RISC-V Assembly (a0 = a, a1 = b)
custom_add:
    add a0, a0, a1    # a0 = a0 + a1
    ret               # Return to caller (jalr x0, 0(ra))

DAY 3: DIGITAL LOGIC DESIGN USING TL-VERILOG & MAKERCHIP
Key Concepts
 * TL-Verilog Abstraction: Replaces verbose SystemVerilog code with timing-abstracted constructs, eliminating manual clock/reset declarations and reducing line count by up to 50%.
 * Sequential Delay Operator (>>N): Easily accesses signal values from N clock cycles ago (e.g., >>1$val).
 * Pipelining Scope (|pipe & @stage): Automatically infers intermediate pipeline registers across pipeline stages.
TL-Verilog Code Examples
1. Free-Running Counter
$cnt[7:0] = $reset ? 8'b0 : (>>1$cnt + 8'b1);

2. Pipelined Calculator with Validity
\m4_TLV_version 1d: tl-x
\SV
   module top(input wire clk, input wire reset, output wire [31:0] out);
\TLV
   |calc
      @1
         $reset = *reset;
         $val1[31:0] = >>1$out[31:0];
         $val2[31:0] = $rand_val2[3:0];
         
         $sum[31:0]  = $val1 + $val2;
         $diff[31:0] = $val1 - $val2;
         $prod[31:0] = $val1 * $val2;
      @2
         $out[31:0] = $reset ? 32'b0 :
                      ($op[1:0] == 2'b00) ? $sum :
                      ($op[1:0] == 2'b01) ? $diff : $prod;
\SV
   endmodule

DAY 4: BUILDING A SINGLE-CYCLE RISC-V CPU CORE
Microarchitecture Overview
Every instruction completes its execution (Fetch, Decode, Read, Execute, Writeback) in one clock cycle.
[PC] ──► [Instruction Memory] ──► [Decode] ──► [Register File] ──► [ALU] ──► [Writeback]

Core Subsystems
 * Instruction Fetch: Increments PC sequentially (PC + 4) or updates PC on branches.
 * Instruction Decode: Extracts opcode, rd, rs1, rs2, funct3, funct7, and generates sign-extended immediates (I-type, S-type, B-type, U-type, J-type).
 * Register File: 32 registers \times 32 bits, supporting dual-read ports and one write-port (with x0 write-inhibit).
 * ALU & Branch Unit: Computes mathematical results and evaluates conditional branch statements (BEQ, BNE, BLT, BGE).
DAY 5: 5-STAGE PIPELINED CPU WITH HAZARD RESOLUTION
5-Stage Architecture
 * Fetch (@1 IF): Fetch instruction from memory using PC.
 * Decode (@2 ID): Decode instruction and read source registers.
 * Execute (@3 EX): Perform ALU operations and evaluate branch condition/target.
 * Memory (@4 MEM): Perform Data Memory Load (lw) or Store (sw) accesses.
 * Writeback (@5 WB): Write final output back to destination register rd.
Hazard Resolution
 * Data Hazard (RAW - Read-After-Write): Solved via Register Forwarding from EX/MEM (@4) and MEM/WB (@5) stages directly into the Execute stage operands without stalling.
 * Control Hazard (Branch Penalty): Solved by flushing/invalidating the fetched instructions in Stage 1 & 2 when a branch is taken ($valid flag gating) and redirecting PC to the target address.
Master Day 5 Top-Level TL-Verilog Code
\m4_TLV_version 1d: tl-x
\SV
   module top(input wire clk, input wire reset, output wire [31:0] passed);
\TLV
   |cpu
      @0
         $pc[31:0] = >>1$reset ? 32'b0 :
                     >>2$taken_branch ? >>2$br_tgt_pc :
                     >>2$is_jal ? >>2$br_tgt_pc :
                     >>2$is_jalr ? >>2$jalr_target :
                     (>>1$pc + 32'd4);
      @1
         $imem_rd_en = !$reset;
         $imem_rd_addr[28:0] = $pc[30:2];
         $instr[31:0] = $imem_rd_data[31:0];
         $valid = $reset ? 1'b0 : !(>>1$valid_taken_branch || >>2$valid_taken_branch);
         
      @2
         $opcode[6:0] = $instr[6:0];
         $rd[4:0]     = $instr[11:7];
         $funct3[2:0] = $instr[14:12];
         $rs1[4:0]    = $instr[19:15];
         $rs2[4:0]    = $instr[24:20];
         
         $is_i_instr = $opcode == 7'b0010011 || $opcode == 7'b0000011 || $opcode == 7'b1100111;
         $is_r_instr = $opcode == 7'b0110011;
         $is_s_instr = $opcode == 7'b0100011;
         $is_b_instr = $opcode == 7'b1100011;
         $is_jal     = $opcode == 7'b1101111;
         $is_jalr    = $opcode == 7'b1100111;
         $is_lw      = $opcode == 7'b0000011;
         $is_sw      = $opcode == 7'b0100011;

      @3
         // Forwarding Logic for RAW Data Hazards
         $src1_value[31:0] = (>>1$rf_wr_en && (>>1$rd == $rs1) && (>>1$rd != 5'b0)) ? >>1$result :
                             (>>2$rf_wr_en && (>>2$rd == $rs1) && (>>2$rd != 5'b0)) ? >>2$result :
                             $rf_rd_data1;
                             
         $src2_value[31:0] = (>>1$rf_wr_en && (>>1$rd == $rs2) && (>>1$rd != 5'b0)) ? >>1$result :
                             (>>2$rf_wr_en && (>>2$rd == $rs2) && (>>2$rd != 5'b0)) ? >>2$result :
                             $rf_rd_data2;

         // ALU Execution
         $alu_result[31:0] = $is_addi ? ($src1_value + $imm) :
                             $is_add  ? ($src1_value + $src2_value) :
                             $is_sub  ? ($src1_value - $src2_value) :
                             ($is_lw || $is_sw) ? ($src1_value + $imm) : 32'b0;

         $taken_branch = $is_b_instr && (
                            ($funct3 == 3'b000 && $src1_value == $src2_value) ||
                            ($funct3 == 3'b001 && $src1_value != $src2_value)
                         );
         $valid_taken_branch = $valid && $taken_branch;
         $br_tgt_pc[31:0] = $pc + $imm;
         $jalr_target[31:0] = $src1_value + $imm;

      @4
         $dmem_rd_en = $is_lw && $valid;
         $dmem_wr_en = $is_sw && $valid;
         $dmem_addr[28:0] = $alu_result[30:2];
         $dmem_wr_data[31:0] = $src2_value;

      @5
         $result[31:0] = $is_lw ? $dmem_rd_data :
                         ($is_jal || $is_jalr) ? ($pc + 32'd4) :
                         $alu_result;
                         
         $rf_wr_en = ($rd != 5'b0) && $valid && ($is_r_instr || $is_i_instr || $is_lw || $is_jal || $is_jalr);
         $rf_wr_index[4:0] = $rd;
         $rf_wr_data[31:0]  = $result;
\SV
   endmodule