# Lab 11: Counters and Memory on the FPGA

**[Download the Lab 11 Report Template (PDF)](EE210L-Lab11-Report-Template.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

The last lab of the course. You will put clocked logic on real hardware, deal with problems
that simulation hides, and infer a memory in Verilog.

## 1. Objectives

- Use the board's clock and divide it down to a visible rate.
- Run a counter and a ring counter on hardware.
- Debounce a push button and explain why it is necessary.
- Infer a ROM in Verilog and use a counter to address it.
- Infer a small RAM and read back what you wrote.
- Compare inferred memory against the chip's dedicated block RAM.

## 2. Equipment and Software

- PYNQ-Z2 board and micro-USB cable
- Computer with Vivado installed
- Your `counter4.v`, `shift4.v`, and ring counter from Lab 9

## 3. Background

### 3.1 The Board Clock

The PYNQ-Z2 supplies a **125 MHz** clock to the programmable logic on package pin **H16**:

```tcl
set_property -dict { PACKAGE_PIN H16 IOSTANDARD LVCMOS33 } [get_ports { clk }];
create_clock -add -name sys_clk_pin -period 8.00 -waveform {0 4} [get_ports { clk }];
```

The `create_clock` line matters as much as the pin assignment. It tells the timing engine
how fast the clock runs so it can check whether your logic settles in time. Without it,
Vivado has no timing target and will happily build a design that fails on hardware.

> **First thing to check in §5.1.** If your counter never appears to run, the clock is the
> first suspect, not your Verilog. Your instructor will confirm the clock source for your
> boards before you start.

### 3.2 Dividing the Clock

125 MHz is 125 million edges per second. A counter clocked directly at that rate cycles
through all 16 states in 128 nanoseconds — the LEDs would appear uniformly dim, not
counting.

The fix is a **clock divider**: a wide counter whose top bit toggles slowly.

```verilog
reg [26:0] divider = 27'd0;

always @(posedge clk) begin
    divider <= divider + 1'b1;
end

wire tick = (divider == 27'd0);   // one pulse every 2^27 clocks
```

At 125 MHz, a 27-bit counter overflows about every 1.07 seconds.

Note what this code does **not** do. It does not create a second clock. `tick` is a normal
signal used as a **clock enable**:

```verilog
always @(posedge clk) begin
    if (tick) count <= count + 1'b1;
end
```

Everything stays on the one 125 MHz clock, and `tick` decides which edges count. Generating
a real second clock by dividing in fabric — using `divider[26]` as another module's `clk` —
creates a clock the timing tools cannot analyze properly and is one of the classic
beginner mistakes in FPGA design. Use clock enables.

### 3.3 Switch Bounce

A mechanical switch does not close cleanly. The contacts physically bounce for a few
milliseconds, making and breaking the connection several times. In Lab 7 you clocked a
counter from a function generator and never saw this. Press a button as a clock instead and
a single press advances the counter an unpredictable number of steps.

A **debouncer** waits for the input to hold steady before accepting it:

```verilog
reg [19:0] count = 20'd0;
reg        sync0, sync1, stable;

always @(posedge clk) begin
    sync0 <= btn_raw;       // two flip-flops to synchronize the
    sync1 <= sync0;         // asynchronous button to our clock domain

    if (sync1 != stable) begin
        count <= count + 1'b1;
        if (count == 20'hFFFFF) begin
            stable <= sync1;
            count  <= 20'd0;
        end
    end else begin
        count <= 20'd0;
    end
end
```

The two-flip-flop chain at the top is a **synchronizer**. A button press is asynchronous to
your clock, so it can violate setup and hold time and drive a flip-flop metastable — the
condition described in Lab 6 §3.4. Two flip-flops in series give any metastable state a
full clock period to settle before the rest of the design sees it. Every asynchronous input
into a synchronous design needs this.

### 3.4 Inferring Memory

You do not instantiate memory on an FPGA by hand. You describe it, and the synthesizer
recognizes the pattern and maps it onto dedicated hardware.

A ROM is an array with constant contents:

```verilog
reg [3:0] rom [0:15];

initial begin
    rom[0]  = 4'h1;  rom[1]  = 4'h2;  rom[2]  = 4'h4;  rom[3]  = 4'h8;
    rom[4]  = 4'h8;  rom[5]  = 4'h4;  rom[6]  = 4'h2;  rom[7]  = 4'h1;
    rom[8]  = 4'h9;  rom[9]  = 4'h6;  rom[10] = 4'h9;  rom[11] = 4'h6;
    rom[12] = 4'hF;  rom[13] = 4'h0;  rom[14] = 4'hF;  rom[15] = 4'h0;
end

always @(posedge clk) begin
    dout <= rom[addr];
end
```

A RAM adds a write port:

```verilog
reg [3:0] ram [0:15];

always @(posedge clk) begin
    if (we) ram[addr] <= din;
    dout <= ram[addr];
end
```

The Zynq XC7Z020 contains 140 **block RAMs** of 36 kbit each. A 16×4 memory is far too small
to justify one, so the synthesizer will build it from **distributed RAM** — the LUTs
themselves, used as storage. You will see which one it chose in the utilization report.

Notice that reads are clocked in both examples. A registered read output is what allows the
synthesizer to use a real block RAM, since block RAM has a register built into its output
path. Writing an unclocked read (`assign dout = ram[addr];`) forces distributed RAM and can
silently cost you far more logic on a large memory.

## 4. Pre-Lab

1. Bring your working `counter4.v` and ring counter from Lab 9.
2. Calculate the divider width needed for roughly a 1 Hz tick from a 125 MHz clock. Show
   your work.
3. Write the top-level module for §5.2: a 4-bit counter on `led[3:0]`, clocked from `clk`
   with a ~1 Hz enable, reset from `btn[0]`.
4. Explain in one or two sentences why the two-flip-flop synchronizer in §3.3 is needed even
   though the debouncer already filters the input.
5. Choose 16 four-bit values for your ROM. Pick a pattern you will recognize on the LEDs.

## 5. Lab Work

### 5.1 Verify the Clock

1. Create a new RTL project targeting **xc7z020clg400-1**.
2. Write a minimal design: a 27-bit counter clocked from `clk`, with `divider[26]` driving
   `led[0]`.
3. Add the clock constraint from §3.1 plus the LED pin from
   [Lab 10 §3.3](../lab-10-first-fpga-design/README.md#33-constraints).
4. Build and program the board.
5. `LD0` should blink at roughly 1 Hz. Time ten blinks with a stopwatch and record the
   period.
6. Compute the actual clock frequency from your measurement and compare with 125 MHz.

**Do not continue until the LED blinks.** Everything below depends on a working clock.

### 5.2 Counter on Hardware

1. Add your `counter4.v` from Lab 9 to the project.
2. Build the top module from pre-lab §3: the divider drives a `tick` enable, the counter
   advances on `tick`, and `count[3:0]` drives `led[3:0]`. Wire `btn[0]` to reset.
3. Build, program, and watch it count 0 to 15 in binary on the LEDs.
4. Record how long a full 16-count cycle takes and confirm it agrees with your divider
   calculation.
5. Press `btn[0]` and confirm the count resets.
6. Record the utilization: LUTs and flip-flops used.

### 5.3 Ring Counter

1. Swap in your ring counter from Lab 9 §5.6, keeping the same `tick` enable.
2. Build and program. Confirm a single lit LED walks across `LD0`–`LD3` and wraps.
3. Note in your report how this display differs from the binary counter, and connect it back
   to what you saw on the 74LS194 in Lab 7.

### 5.4 Button as a Clock Enable — Without Debouncing

1. Modify your counter so it advances on a **raw** button press instead of the `tick`
   signal. Detect the press with a rising-edge detector:

   ```verilog
   reg btn_prev;
   always @(posedge clk) btn_prev <= btn[1];
   wire press = btn[1] & ~btn_prev;
   ```
2. Build, program, and press `btn[1]` ten times, counting carefully.
3. Record the count after ten presses. It will almost certainly not be 10.
4. Repeat the ten presses three more times and record each result.

### 5.5 Button as a Clock Enable — Debounced

1. Add the debouncer from §3.3 and take your edge detector from the debounced signal instead
   of the raw pin.
2. Rebuild and program.
3. Press `btn[1]` ten times and record the count. Repeat three more times.
4. Compare against §5.4 and note in your report what the debouncer fixed and how many extra
   counts the bouncing was producing.

### 5.6 ROM Addressed by the Counter

1. Add a 16×4 ROM as in §3.4, filled with the values you chose in pre-lab §5.
2. Address it with your 4-bit counter and drive `led[3:0]` from the ROM output instead of
   the counter output.
3. Build and program. Confirm the LEDs display your stored pattern in sequence rather than
   a binary count.
4. Record the utilization again. Check the synthesis log to see whether the ROM was built
   from LUTs or from a block RAM, and record which.

### 5.7 RAM Write and Read Back

1. Replace the ROM with the 16×4 RAM from §3.4.
2. Wire it up so that:
   - The counter provides the address.
   - `sw[1:0]` provide the low two bits of write data, with the upper two bits tied to the
     counter's own low bits so each location gets a distinguishable value.
   - The debounced `btn[1]` asserts `we` for exactly one clock cycle.
   - `sw[0]` selects between write mode and read mode.
3. Build and program.
4. Step the counter through several addresses, writing a different value at each.
5. Switch to read mode and step through the same addresses, recording what comes back on
   the LEDs.
6. Confirm each location returns the value you wrote.
7. Record the utilization and whether the tool used distributed RAM or block RAM.

### 5.8 Force Block RAM

1. Add the synthesis attribute `(* ram_style = "block" *)` immediately before your RAM
   declaration.
2. Rebuild and record the utilization. Compare the LUT count and the block RAM count with
   §5.7.
3. Note in your report which implementation is a better fit for a 16×4 memory and why the
   tool chose what it did by default.

### 5.9 Shut Down

Close the hardware target in Vivado before unplugging the board.

## 6. Report

Fill in the report template linked at the top of this page and submit it on Canvas. Include
your source code, all measured counts, and every utilization figure.
