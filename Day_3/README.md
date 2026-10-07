NASSCOM RISC-V MYTH Workshop — Complete Day 3 Notes

Welcome to the comprehensive, structured notes for Day 3 of the NASSCOM RISC-V Microprocessor for You in Thirty Hours (MYTH) Workshop, conducted by VSD (VLSI System Design).

---

MASTER DAY 3 OVERVIEW & OBJECTIVES

Day 3 shifts the focus from software, ABI, and compiler flows to digital logic design using Transaction-Level Verilog (TL-Verilog) and the browser-based Makerchip IDE.

Key Learning Goals

* Transition from traditional RTL abstraction to Transaction-Level Verilog (TL-Verilog).
* Master combinational logic, sequential logic, and pipelining structures in TL-Verilog.
* Learn Makerchip IDE features (VIZ visualization, waveform viewer, and logic diagrams).
* Implement advanced TL-Verilog concepts: validity, hierarchical structures, and state machines.
* Build foundational hardware components for the RISC-V CPU core.

---

MODULE 1: INTRODUCTION TO TL-VERILOG & MAKERCHIP IDE

What is TL-Verilog?
Transaction-Level Verilog (TL-Verilog) is an extension to SystemVerilog that introduces timing abstraction. It decouples design logic from timing pipelines, making hardware design simpler, faster, and less error-prone.

Advantages of TL-Verilog over SystemVerilog/Verilog

* Reduces code size significantly (often up to 50% fewer lines of code).
* Automatic generation of pipeline stages, flip-flops, and clocking logic.
* Eliminates redundant signal declarations and manual bus slicing.
* Inherent support for transaction-level pipelining and validity logic.

Makerchip IDE Overview
Makerchip is a browser-based integrated development environment for TL-Verilog design.

* Block Diagram View: Generates visual schematic representations of design logic.
* Waveform Viewer: Displays signal transitions over clock cycles.
* VIZ (Visualizer): Allows custom graphical visualization of core architecture execution.

---

MODULE 2: COMBINATIONAL LOGIC IN TL-VERILOG

Combinational logic outputs depend strictly on current input values, without memory storage or clock dependencies.

Basic Operators

* Logical: && (AND), || (OR), ! (NOT)
* Bitwise: & (AND), | (OR), ^ (XOR), ~ (NOT)
* Arithmetic: +, -, *, /
* Comparison: ==, !=, >, <, >=, <=
* Ternary Multiplexer: assign_out = sel ? in1 : in0;

Combinational Circuits in TL-Verilog

1. Inverter / Gate Logic
\m4_TLV_version 1d: tl-x
\SV
module top(input wire in, output wire out);
\TLV
$out = !$in;
\SV
endmodule
2. Multiplexer (2-to-1 MUX)
$out =$sel ? $in1 :$in0;
3. Simple Arithmetic Logic Unit (ALU) Slice
$alu_out[31:0] = ($op == 2'b00) ? ($src1 +$src2) :
($op == 2'b01) ? ($src1 -$src2) :
($op == 2 me10) ? ($src1 & $src2) :
($src1 \vert{}$src2);

---

MODULE 3: SEQUENTIAL LOGIC & STATE ELEMENTS

Sequential logic elements store data across clock cycles using flip-flops and registers.

Clock and Reset Handling
In TL-Verilog, clock ($clk) and reset ($reset) signals are implicitly handled by the compiler framework, removing the need for verbose always @(posedge clk) blocks.

Sequential Design Examples

1. Simple Register / Delay Element
Signal value from the previous clock cycle is accessed using the >>1 operator.
$out = $reset ? 1'b0 : >>1$in;
2. Free-Running Binary Counter
$cnt[7:0] = $reset ? 8'b0 : (>>1$cnt + 8'b1);
3. Fibonacci Series Generator
$num[31:0] =$reset ? 31'b1 : (>>1$num + >>2$num);

---

MODULE 4: PIPELINED LOGIC IN TL-VERILOG

Pipelining breaks complex combinational logic into smaller processing stages separated by registers, increasing clock frequency and instruction throughput.

Pipeline Syntax in TL-Verilog
Pipelines are defined using the |pipe_name scope, and stages are declared using @stage_number.

Pipelined Computation Example

3-Stage Computation Pipeline:
Stage 0 (@0): Input capture
Stage 1 (@1): Compute intermediate product
Stage 2 (@2): Accumulate final sum

\TLV
|calc
@0
$val1[31:0] =$in1[31:0];
$val2[31:0] =$in2[31:0];
@1
$prod[31:0] = $val1 * $val2;
@2
$out[31:0]  =$reset ? 31'b0 : (>>1$out +$prod);

Automatic Retiming
If a signal from stage 0 (@0) is referenced in stage 2 (@2), TL-Verilog automatically infers and creates the necessary intermediate pipeline staging registers.

---

MODULE 5: VALIDITY CONCEPT & CYCLE-LEVEL LOGIC

What is Validity?
Validity defines when data within a pipeline stage contains meaningful information. It simplifies clock-gating, hazard handling, and power optimization.

TL-Verilog Validity Syntax
Validity conditions are attached to pipeline scopes using the ?valid_signal operator.

Validity Example: Pipelined Calculator with Valid Input

\TLV
|calc
@1
$valid = $reset ? 1'b0 : >>1$valid_in;
?$valid
@1
$sum[31:0]  = $val1 +$val2;
$diff[31:0] = $val1 -$val2;
@2
$out[31:0]  =$sel ? $diff :$sum;

Benefits of Validity

* Disables downstream calculations when $valid == 0.
* Automatic clock-gating generation for power savings.
* Simplifies instruction-validity tracking in processor pipelines.

---

MODULE 6: LAB WORKSHOP — BUILDING A PIPELINED CALCULATOR

As the culminating exercise for Day 3, a multi-operation pipelined calculator is constructed in Makerchip.

Pipelined Calculator Features

* Supports Addition, Subtraction, Multiplication, and Division.
* Includes memory recall/accumulator register ($acc).
* Operates inside a 2-stage pipeline scope with reset logic.

TL-Verilog Calculator Source Code

\m4_TLV_version 1d: tl-x
\SV
module top(input wire clk, input wire reset, output wire [31:0] out);
\TLV
|calc
@1
$reset = *reset;
$valid = $reset ? 1'b0 : >>1$valid;
$val1[31:0] = >>1$out[31:0];$val2[31:0] = $rand_val2[3:0]; // Random test input$sum[31:0]  = $val1 +$val2;
$diff[31:0] = $val1 -$val2;
$prod[31:0] = $val1 * $val2;
$quot[31:0] =$val2 != 0 ? ($val1 / $val2) : 32'b0;
@2
$out[31:0] =$reset ? 32'b0 :
($op[1:0] == 2'b00) ?$sum :
($op[1:0] == 2'b01) ?$diff :
($op[1:0] == 2'b10) ? $prod :$quot;
\SV
endmodule

---

SUMMARY CHECKLIST

1. TL-Verilog Abstraction: Removes timing clutter and explicit clock/reset wires, allowing designers to focus on high-level pipeline structures.
2. Sequential State Access: Accesses past cycle state values cleanly using the >>N operator.
3. Pipeline Automation: Manages intermediate staging registers across @stage boundaries automatically.
4. Validity Constructs: Provides built-in support for conditional pipeline execution, power gating, and transaction control.