# Lab 9: Sequential Logic in Verilog

**[Download the Lab 9 Report (PDF)](EE210L-Lab9-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

This lab is **simulation only**. The designs you build here are the ones you will put on the
FPGA board in Lab 11, so get them right now.

## 1. Objectives

- Describe flip-flops and registers with `always @(posedge clk)`.
- Use non-blocking assignments correctly and explain why blocking assignments break
  sequential logic.
- Write a testbench that generates a clock and applies a reset.
- Build a counter and a shift register in Verilog.
- Distinguish synchronous from asynchronous reset in both code and waveform.

## 2. Equipment and Software

- Computer with Vivado installed
- No hardware

## 3. Background

### 3.1 A Flip-Flop in Verilog

```verilog
module dff (
    input  wire clk,
    input  wire d,
    output reg  q
);
    always @(posedge clk) begin
        q <= d;
    end
endmodule
```

`always @(posedge clk)` means "on every rising edge of `clk`, do this." That single line is
what tells the synthesizer to build a flip-flop rather than a wire. Here `q` genuinely is a
register, unlike the `reg` declarations in Lab 8, which were combinational despite the
keyword.

### 3.2 Blocking vs Non-Blocking

This is the most important rule in the whole lab.

| Operator | Name | Use it for |
|---|---|---|
| `=` | blocking | combinational logic, inside `always @(*)` |
| `<=` | non-blocking | sequential logic, inside `always @(posedge clk)` |

**Blocking** (`=`) executes immediately and in order, like a line of software. The next
statement sees the new value.

**Non-blocking** (`<=`) evaluates every right-hand side first, then updates every left-hand
side simultaneously at the end of the time step. That is exactly what real flip-flops do:
they all sample their inputs at the same clock edge, and none of them sees another's new
output until after the edge.

Consider a two-stage shift register:

```verilog
always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;      // non-blocking: q2 gets the OLD q1
end
```

This builds two flip-flops in a chain, which is what you want. Now with blocking
assignments:

```verilog
always @(posedge clk) begin
    q1 = d;
    q2 = q1;       // blocking: q1 was ALREADY updated, so q2 gets the NEW value
end
```

Here `q2` receives `d` on the same edge, so both flip-flops hold the same value and the
shift register collapses into one stage. The code looks nearly identical. The hardware is
completely different. You will demonstrate this in §5.3.

### 3.3 Reset

An **asynchronous** reset acts the moment it is asserted, like the `CLR̅` pin you used on the
74LS74 in Lab 6:

```verilog
always @(posedge clk or posedge rst) begin
    if (rst) q <= 1'b0;
    else     q <= d;
end
```

A **synchronous** reset only takes effect at the next clock edge, like the `CLR̅` on the
74LS163 in Lab 7:

```verilog
always @(posedge clk) begin
    if (rst) q <= 1'b0;
    else     q <= d;
end
```

The difference is entirely in the sensitivity list. Note that the asynchronous version
lists `rst` as an edge; that is the only situation where a signal other than the clock
belongs in a `posedge` list.

Synchronous reset is generally preferred inside FPGAs, because it does not create a second
timing path into every flip-flop and it cannot glitch a register on a noisy reset line.

### 3.4 Generating a Clock in a Testbench

```verilog
reg clk = 1'b0;
always #5 clk = ~clk;      // toggles every 5 ns -> 10 ns period -> 100 MHz
```

Note this uses a blocking assignment, and that is correct, because the clock generator is
testbench code, not hardware being synthesized.

A typical reset sequence at the start of a testbench:

```verilog
initial begin
    rst = 1'b1;
    repeat (2) @(posedge clk);   // hold reset for two clock edges
    rst = 1'b0;
end
```

### 3.5 Counters in Verilog

```verilog
module counter4 (
    input  wire       clk,
    input  wire       rst,
    input  wire       en,
    output reg  [3:0] count
);
    always @(posedge clk) begin
        if (rst)      count <= 4'd0;
        else if (en)  count <= count + 1'b1;
    end
endmodule
```

`count <= count + 1'b1` reads the current value of the register and writes back the
incremented value on the next edge. The `else if (en)` with no final `else` is correct here
and does **not** infer a latch. Inside a clocked block, "no assignment" simply means the
flip-flop holds, which is what an enable is supposed to do. Latch inference is only a
concern in combinational blocks.

## 4. Pre-Lab

1. Write the Verilog for a D flip-flop with an asynchronous active-high reset.
2. Write a 4-bit shift register that shifts right, with a serial input `sin`.
3. Predict, on paper, the value of `q1` and `q2` after three clock edges for both versions
   of the code in §3.2, starting from `q1 = q2 = 0` and holding `d = 1`.
4. State which reset style, synchronous or asynchronous, matches the 74LS163 you used in
   Lab 7, and which matches the 74LS74.

## 5. Lab Work

### 5.1 D Flip-Flop

1. Create an RTL project in Vivado, simulation only.
2. Enter the `dff` module from §3.1.
3. Write a testbench with a 100 MHz clock as in §3.4. Drive `d` through a sequence of at
   least six values, changing it between clock edges rather than on them.
4. Run behavioral simulation and capture the waveform.
5. On the waveform, confirm that `q` changes only at rising clock edges and never in
   between. Mark one edge in your capture where `d` had changed earlier but `q` waited.

### 5.2 Synchronous vs Asynchronous Reset

1. Create two modules: `dff_sync_rst` and `dff_async_rst`, using the two styles from §3.3.
2. Instantiate **both** in a single testbench, driving them from the same `clk`, `d`, and
   `rst`.
3. Assert `rst` in the middle of a clock period, not aligned with an edge, and hold it
   for less than one full clock cycle.
4. Capture the waveform showing both outputs. Record which one responded immediately and
   which one waited for the next edge.
5. Now assert `rst` as a very short pulse that begins and ends between two clock edges.
   Record what each version does. Note in your report which reset style can miss a short
   pulse entirely, and why that could be either a bug or a feature.

### 5.3 Blocking vs Non-Blocking

1. Create `shift_nonblocking.v` with the two-stage shift register from §3.2 using `<=`.
2. Create `shift_blocking.v`, identical but using `=`.
3. Instantiate both in one testbench with a shared clock. Hold `d = 1` and let at least
   four clock edges pass.
4. Capture the waveform showing `q1` and `q2` from both modules.
5. Record the values after each edge for both versions and compare against your pre-lab
   prediction.
6. Run synthesis on both. Open the schematic view for each and record how many flip-flops
   each one produced.

The waveform shows you the symptom. The schematic shows you the cause.

### 5.4 Four-Bit Counter

1. Create `counter4.v` from §3.5.
2. Write a testbench that resets the counter, enables it, and lets it run for at least 20
   clock edges so it wraps from 15 back to 0.
3. Add a self-checking section: keep an expected count in the testbench and compare it with
   the design's output after every edge.
4. Confirm the counter wraps correctly and that `en = 0` holds the value.
5. Capture the waveform showing the wrap from 1111 to 0000.

### 5.5 Shift Register with Modes

1. Extend your shift register into `shift4.v` with a 2-bit mode input matching the 74LS194
   from Lab 7:

   | `mode` | Behavior |
   |---|---|
   | `2'b00` | hold |
   | `2'b01` | shift right, `sin` enters at the MSB |
   | `2'b10` | shift left, `sin` enters at the LSB |
   | `2'b11` | parallel load from `din[3:0]` |

2. Use a `case` statement inside `always @(posedge clk)`.
3. Write a testbench that exercises all four modes in sequence: load `4'b1011`, hold for two
   edges, shift right four times, reload, then shift left four times.
4. Capture the waveform and confirm the behavior matches what you measured on the 74LS194.

### 5.6 Ring Counter

1. Build a 4-bit ring counter by feeding the LSB of your shift register back into `sin`,
   loading `4'b1000` to start.
2. Simulate it for at least eight clock edges and confirm the pattern circulates with a
   period of four, matching Lab 7 §5.6.
3. Capture the waveform. You will put this design on the board in Lab 11.

## 6. Report

Fill in the report linked at the top of this page and submit it on Canvas. Include
your source code, all waveform captures, and the flip-flop counts from §5.3.
