NASSCOM RISC-V MYTH Workshop — Complete Day 5 Notes

Welcome to the comprehensive, structured notes for Day 5 of the NASSCOM RISC-V Microprocessor for You in Thirty Hours (MYTH) Workshop, conducted by VSD (VLSI System Design).

---

MASTER DAY 5 OVERVIEW & OBJECTIVES

Day 5 completes the MYTH workshop by converting the single-cycle core into a fully functional 5-stage pipelined RISC-V CPU. It addresses hazard detection, register forwarding, branch misprediction penalty handling, load/store memory integration, and core verification.

Key Learning Goals

* Convert single-cycle datapath logic into a 5-stage CPU pipeline structure.
* Identify and resolve Read-After-Write (RAW) data hazards using register forwarding.
* Implement control hazard logic (branch redirect penalty handling).
* Integrate Data Memory for load (`lw`) and store (`sw`) operations.
* Support jump instructions (`jal`, `jalr`) and complete end-to-end CPU simulation/verification.

---

MODULE 1: 5-STAGE CPU PIPELINE ARCHITECTURE

Pipelining increases CPU clock frequency and instruction throughput by dividing execution into discrete processing stages separated by pipeline registers.

The 5 Pipeline Stages

Stage 1: Fetch (IF)   ──► Fetch instruction from memory using Program Counter (PC).
Stage 2: Decode (ID)  ──► Decode instruction fields, read source registers (rs1, rs2).
Stage 3: Execute (EX) ──► Compute ALU operations, calculate branch condition/target address.
Stage 4: Memory (MEM) ──► Perform Data Memory load/store accesses.
Stage 5: Writeback (WB)─► Write final result back to destination register (rd).

TL-Verilog Pipeline Structure
In TL-Verilog, pipeline stages are represented cleanly using stage scopes (`@1` through `@5` under `|cpu`).

---

MODULE 2: PIPELINE DATA HAZARDS & REGISTER FORWARDING

What is a Data Hazard (RAW)?
A Read-After-Write (RAW) hazard occurs when an instruction in an earlier pipeline stage depends on a result produced by an older instruction that has not yet completed its Writeback stage.

Example Hazard Scenario:
Instruction 1 (@3 EX): add x5, x1, x2  (x5 updated at Writeback / @5)
Instruction 2 (@2 ID): add x6, x5, x3  (Reads x5 at Decode / @2 -> STALE VALUE!)

Register Forwarding Logic
Rather than stalling the pipeline, intermediate results are forwarded directly from downstream pipeline stages (EX/MEM/WB) back to the Execution stage operands.

Forwarding Rules for Source Operand 1 (rs1):

* Forward from Stage 4 (MEM) if source register `rs1` matches Stage 4 destination `rd`.
* Forward from Stage 5 (WB) if source register `rs1` matches Stage 5 destination `rd`.
* Otherwise, use value read from Register File.

TL-Verilog Forwarding Syntax:

$src1_value[31:0] = (>>1$rf_wr_en && (>>1$rd ==$rs1)) ? >>1$result : // Forward from EX/MEM (@4)                     (>>2$rf_wr_en && (>>2$rd ==$rs1)) ? >>2$result : // Forward from MEM/WB (@5)$rf_rd_data1;

$src2_value[31:0] = (>>1$rf_wr_en && (>>1$rd ==$rs2)) ? >>1$result :                     (>>2$rf_wr_en && (>>2$rd ==$rs2)) ? >>2$result :$rf_rd_data2;

---

MODULE 3: CONTROL HAZARDS & BRANCH REDIRECT HANDLING

What is a Control Hazard?
Branch outcomes are evaluated in Stage 3 (Execute). By the time a taken branch is resolved, subsequent instructions have already been fetched in Stage 1 and Stage 2, introducing invalid instructions into the pipeline.

Branch Penalty & Pipeline Flush
When a branch is taken:

1. Redirect Program Counter (PC) to branch target address.
2. Invalidate/flush instructions currently fetched in Stage 1 and Stage 2 using validity flags (`$valid`).

TL-Verilog Branch Redirect Logic:

@0
$pc[31:0] = >>1$reset ? 32'b0 :
>>2$taken_branch ? >>2$br_tgt_pc : // PC redirect from EX stage (@3)                (>>1$pc + 32'd4);

@1
// Invalidate instruction if preceding branch was taken
$valid =$reset ? 1'b0 :
!(>>1$valid_taken_branch \vert{}\vert{} >>2$valid_taken_branch);

---

MODULE 4: DATA MEMORY INTEGRATION (LOAD & STORE)

Data Memory Interface
Data memory operations occur in Stage 4 (Memory Stage):

* Load Word (`lw`): Reads 32-bit data from memory address calculated by ALU (`src1 + imm`).
* Store Word (`sw`): Writes 32-bit data from `src2` into memory address calculated by ALU.

Memory Control Signals (Stage 4):

$dmem_rd_en = $is_lw &&$valid;
$dmem_wr_en = $is_sw &&$valid;
$dmem_addr[28:0] =$result[30:2]; // Word-aligned memory address
$dmem_wr_data[31:0] =$src2_value;

Writeback Result Selection (Stage 5):

$result[31:0] =$is_lw ? $dmem_rd_data :$alu_result;

---

MODULE 5: JUMP INSTRUCTIONS & COMPLETE CPU INTEGRATION

Supporting Unconditional Jumps

* `jal` (Jump and Link): Target address = `PC + imm`. Writes `PC + 4` into `rd`.
* `jalr` (Jump and Link Register): Target address = `src1 + imm`. Writes `PC + 4` into `rd`.

Jump Execution Logic:

$is_jump = $is_jal \vert{}\vert{}$is_jalr;
$jalr_target[31:0] = $src1_value +$imm;
$br_tgt_pc[31:0]   = $is_jalr ? $jalr_target : ($pc +$imm);

---

MODULE 6: COMPLETE DAY 5 PIPELINED RISC-V CPU TL-VERILOG CODE

\m4_TLV_version 1d: tl-x
\SV
module top(input wire clk, input wire reset, output wire [31:0] passed);
\TLV
|cpu
@0
$pc[31:0] = >>1$reset ? 32'b0 :
>>2$taken_branch ? >>2$br_tgt_pc :                      >>2$is_jal ? >>2$br_tgt_pc :                      >>2$is_jalr ? >>2$jalr_target :                      (>>1$pc + 32'd4);
@1
$imem_rd_en = !$reset;
$imem_rd_addr[28:0] =$pc[30:2];
$instr[31:0] =$imem_rd_data[31:0];
$valid =$reset ? 1'b0 : !(>>1$valid_taken_branch \vert{}\vert{} >>2$valid_taken_branch);

```
  @2
     $opcode[6:0] =$instr[6:0];
     $rd[4:0]     =$instr[11:7];
     $funct3[2:0] =$instr;
     $rs1[4:0]    =$instr;
     $rs2[4:0]    =$instr;
     
     // Instruction Decoding Flags
     $is_i_instr =$opcode == 7'b0010011 || $opcode == 7'b0000011 \vert{}\vert{}$opcode == 7'b1100111;
     $is_r_instr =$opcode == 7'b0110011;
     $is_s_instr =$opcode == 7'b0100011;
     $is_b_instr =$opcode == 7'b1100011;
     $is_jal     =$opcode == 7'b1101111;
     $is_jalr    =$opcode == 7'b1100111;
     $is_lw      =$opcode == 7'b0000011;
     $is_sw      =$opcode == 7'b0100011;

  @3
     // Register Forwarding Logic
     $src1_value[31:0] = (>>1$rf_wr_en && (>>1$rd ==$rs1) && (>>1$rd != 5'b0)) ? >>1$result :
                         (>>2$rf_wr_en && (>>2$rd == $rs1) && (>>2$rd != 5'b0)) ? >>2$result :$rf_rd_data1;
                         
     $src2_value[31:0] = (>>1$rf_wr_en && (>>1$rd ==$rs2) && (>>1$rd != 5'b0)) ? >>1$result :
                         (>>2$rf_wr_en && (>>2$rd == $rs2) && (>>2$rd != 5'b0)) ? >>2$result :$rf_rd_data2;

     // ALU Operation
     $alu_result[31:0] =$is_addi ? ($src1_value +$imm) :
                         $is_add  ? ($src1_value + $src2_value) :$is_sub  ? ($src1_value -$src2_value) :
                         ($is_lw \vert{}\vert{}$is_sw) ? ($src1_value +$imm) : 32'b0;

     $taken_branch =$is_b_instr && (
                        ($funct3 == 3'b000 && $src1_value ==$src2_value) || // BEQ
                        ($funct3 == 3'b001 && $src1_value !=$src2_value)    // BNE
                     );
     $valid_taken_branch = $valid &&$taken_branch;
     $br_tgt_pc[31:0] = $pc +$imm;
     $jalr_target[31:0] = $src1_value +$imm;

  @4
     $dmem_rd_en = $is_lw &&$valid;
     $dmem_wr_en = $is_sw &&$valid;
     $dmem_addr[28:0] =$alu_result[30:2];
     $dmem_wr_data[31:0] =$src2_value;

  @5
     $result[31:0] = $is_lw ? $dmem_rd_data :
                     ($is_jal \vert{}\vert{}$is_jalr) ? ($pc + 32'd4) :$alu_result;
                     
     $rf_wr_en = ($rd != 5'b0) && $valid && ($is_r_instr || $is_i_instr \vert{}\vert{}$is_lw || $is_jal \vert{}\vert{}$is_jalr);
     $rf_wr_index[4:0] =$rd;
     $rf_wr_data[31:0]  =$result;

```

\SV
endmodule

---

SUMMARY CHECKLIST

1. 5-Stage Pipelining: Pipelined execution across Fetch (IF), Decode (ID), Execute (EX), Memory (MEM), and Writeback (WB) stages improves timing performance.
2. Data Hazard Handling: Solved Read-After-Write (RAW) data hazards by implementing forwarding paths from downstream stages (EX/MEM and MEM/WB).
3. Control Hazard Resolution: Resolved branch prediction penalties by flushing invalid pipeline stages when branches are taken.
4. Complete CPU Core: Fully integrates memory access (load/store), jump instructions, forwarding networks, and register writebacks in a verified RISC-V core.