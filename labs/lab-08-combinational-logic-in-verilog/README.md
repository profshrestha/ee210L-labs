# Lab 8: Combinational Logic in Verilog

**[Download the Lab 8 Report Template (PDF)](EE210L-Lab8-Report-Template.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

This lab is **simulation only**. You will not use the FPGA board. The goal is to learn the
language and the simulator without also fighting the hardware flow — that comes in Lab 10.

## 1. Objectives

- Write a Verilog module with a correct port list.
- Describe combinational logic with continuous assignment and with `always @(*)`.
- Write a testbench that drives a design and checks its output automatically.
- Read a simulation waveform and use it to find a bug.
- Recognize the difference between describing hardware and writing a program.

## 2. Equipment and Software

- Computer with Vivado installed (version specified by your instructor)
- No hardware

## 3. Background

### 3.1 Verilog Describes Hardware

This is the idea that trips up everyone who has written software before. Verilog is not a
list of instructions that execute in order. It is a **description of a circuit**. When you
write

```verilog
assign y = a & b;
```

you are not saying "compute `a AND b` and store it in `y`." You are saying "there is an AND
gate, its inputs are `a` and `b`, and its output is permanently wired to `y`." That gate
exists for as long as the circuit is powered, and `y` responds to any change on `a` or `b`
immediately and continuously.

Everything you write in this lab describes gates like the ones you wired by hand in Labs
1–5.

### 3.2 Module Structure

A module is the Verilog equivalent of a chip: a named block with a defined set of pins.

```verilog
module majority (
    input  wire a,
    input  wire b,
    input  wire c,
    output wire m
);
    assign m = (a & b) | (b & c) | (a & c);
endmodule
```

- `input` and `output` declare the direction of each port.
- `wire` is a connection that carries a value from a driver — it cannot store anything.
- `assign` creates a **continuous assignment**: the left side always tracks the right side.

### 3.3 Vectors

A multi-bit signal is declared with a range:

```verilog
input  wire [3:0] a;    // 4 bits, a[3] is the most significant
output wire [3:0] sum;
```

`a[3:0]` means bit 3 down to bit 0. Concatenation uses braces: `{carry, sum}` makes a 5-bit
value from a 1-bit and a 4-bit signal, which is exactly how you capture an adder's carry
out along with its sum.

### 3.4 `always @(*)` and `reg`

The other way to describe combinational logic is a procedural block:

```verilog
module compare2 (
    input  wire [1:0] a,
    input  wire [1:0] b,
    output reg        gt,
    output reg        eq,
    output reg        lt
);
    always @(*) begin
        gt = 1'b0;
        eq = 1'b0;
        lt = 1'b0;
        if (a > b)      gt = 1'b1;
        else if (a == b) eq = 1'b1;
        else             lt = 1'b1;
    end
endmodule
```

Two things to notice:

- A signal assigned inside an `always` block must be declared `reg`. **This does not mean it
  becomes a register.** It is a quirk of the language, not a statement about the hardware.
  This block still synthesizes to pure combinational logic.
- `@(*)` means "re-evaluate whenever any input changes." The three default assignments at
  the top matter: if some path through the block left an output unassigned, the synthesizer
  would have to remember its old value, and it would infer a **latch** to do that. Assigning
  every output at the top of the block guarantees that never happens.

Accidental latches are the single most common bug in student combinational code. Assign
defaults, always.

### 3.5 Testbenches

A testbench is a module with no ports. It instantiates your design, drives its inputs, and
checks its outputs.

```verilog
`timescale 1ns / 1ps

module majority_tb;
    reg  a, b, c;
    wire m;
    integer errors = 0;

    majority dut (.a(a), .b(b), .c(c), .m(m));

    task check (input expected);
        begin
            if (m !== expected) begin
                $display("FAIL: a=%b b=%b c=%b  got m=%b expected %b", a, b, c, m, expected);
                errors = errors + 1;
            end
        end
    endtask

    initial begin
        a = 0; b = 0; c = 0; #10 check(0);
        a = 0; b = 0; c = 1; #10 check(0);
        a = 0; b = 1; c = 0; #10 check(0);
        a = 0; b = 1; c = 1; #10 check(1);
        a = 1; b = 0; c = 0; #10 check(0);
        a = 1; b = 0; c = 1; #10 check(1);
        a = 1; b = 1; c = 0; #10 check(1);
        a = 1; b = 1; c = 1; #10 check(1);

        if (errors == 0) $display("PASS: all cases correct");
        else             $display("%0d case(s) failed", errors);
        $finish;
    end
endmodule
```

- `dut` stands for device under test. The `.a(a)` syntax connects testbench signal `a` to
  port `a` of the module — always connect ports by name, never by position.
- `#10` waits 10 time units so the output settles before you check it.
- `!==` compares including the `x` (unknown) and `z` (high-impedance) states, which `!=`
  does not. Use `!==` in testbenches or a design outputting `x` will silently pass.
- A testbench that prints PASS or FAIL is called **self-checking**. Reading waveforms by eye
  does not scale — even here it is 8 cases, and a 4-bit adder has 512.

## 4. Pre-Lab

1. Write the Verilog for a 2-input XOR gate using continuous assignment.
2. Write the same thing using `always @(*)`. Note which signals must be `reg`.
3. Write out, on paper, what would be wrong with this block and what hardware it would
   infer:

   ```verilog
   always @(*) begin
       if (sel) y = a;
   end
   ```
4. Sketch the truth table for a 2-bit magnitude comparator (4 input bits, 3 outputs). You
   will implement it in §5.3.

## 5. Lab Work

### 5.1 Project Setup and First Simulation

1. Create a new RTL project in Vivado. When asked, do **not** specify sources yet, and pick
   any part — this project is simulation only, so the part does not matter.
2. Add a design source named `majority.v` and enter the module from §3.2.
3. Add a simulation source named `majority_tb.v` and enter the testbench from §3.5.
4. Run behavioral simulation. Confirm the Tcl console prints `PASS: all cases correct`.
5. Capture the waveform window showing all eight input combinations and the resulting `m`.

### 5.2 Break It on Purpose

1. Change the `majority` module so it computes `(a & b) | (b & c)` — dropping the `a & c`
   term.
2. Re-run the simulation. Record exactly what the testbench prints and which case or cases
   fail.
3. Find the failing case on the waveform and note how you would have spotted it there.
4. Restore the correct expression and confirm it passes again.

This is the point of a self-checking testbench: it told you precisely which row was wrong
without you reading anything.

### 5.3 Two-Bit Comparator

1. Create `compare2.v` using the `always @(*)` style from §3.4.
2. Write a self-checking testbench that exercises **all 16** combinations of `a[1:0]` and
   `b[1:0]`. Use nested loops rather than 16 hand-written cases:

   ```verilog
   integer i, j;
   initial begin
       for (i = 0; i < 4; i = i + 1)
           for (j = 0; j < 4; j = j + 1) begin
               a = i[1:0]; b = j[1:0];
               #10;
               // check gt, eq, lt against i and j here
           end
   end
   ```
3. Run it and confirm all 16 cases pass. Capture the waveform.
4. Confirm that exactly one of `gt`, `eq`, `lt` is high in every case. Add that as a check
   in your testbench — it catches a whole class of bugs that checking the three outputs
   individually would miss.

### 5.4 Inferred Latch

1. Create a module containing the faulty block from pre-lab §3.
2. Run **synthesis** on it (not just simulation). Open the synthesis log and search for the
   word `latch`.
3. Record the warning message verbatim in your report.
4. Fix the block by assigning a default, re-synthesize, and confirm the warning is gone.

Vivado tells you when you have inferred a latch. Learning to read that warning now will
save you hours later.

### 5.5 Four-Bit Adder

1. Write `adder4.v` with inputs `a[3:0]`, `b[3:0]`, `cin`, and outputs `sum[3:0]`, `cout`.
   Use a single continuous assignment with concatenation:

   ```verilog
   assign {cout, sum} = a + b + cin;
   ```
2. Write a testbench that checks all 512 combinations of `a`, `b`, and `cin` using nested
   loops. Compare against the expected value computed in the testbench itself.
3. Run it and confirm it passes.
4. Note in your report how long this took to write compared with wiring the 74LS283 in
   Lab 5, and how many cases you verified compared with the four you measured by hand.

## 6. Report

Fill in the report template linked at the top of this page and submit it on Canvas. Include
your source code, waveform captures, and the latch warning text.
