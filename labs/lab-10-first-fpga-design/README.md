# Lab 10: First FPGA Design

**[Download the Lab 10 Report (PDF)](EE210L-Lab10-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

This is your first lab on the FPGA board. The designs are ones you have already built and
already simulated, so everything new in this session is the **tool flow**: constraints,
synthesis, implementation, bitstream, and programming.

## 1. Objectives

- Create a Vivado project targeting the PYNQ-Z2 board.
- Write an XDC constraints file that maps module ports to physical pins.
- Run synthesis and implementation and read the resource utilization report.
- Generate a bitstream and program the FPGA over USB-JTAG.
- Verify a combinational design on real hardware.

## 2. Equipment and Software

- PYNQ-Z2 board (Xilinx Zynq XC7Z020) and micro-USB cable
- Computer with Vivado installed
- Your `majority.v` and `compare2.v` from Lab 8

## 3. Background

### 3.1 What an FPGA Actually Is

The 74LS chips in Labs 1–7 each contained one fixed function. An FPGA contains a large array
of **lookup tables (LUTs)**, flip-flops, and programmable interconnect. A LUT is a tiny
memory: feed it the input combination as an address and it returns the stored output bit.
Six inputs, one output, and 64 bits of storage is enough to implement *any* 6-input Boolean
function just by changing what is stored.

That is what a bitstream is: the contents of every LUT and the setting of every
interconnect switch. Programming an FPGA does not run a program on it. It rewires it.

Because a LUT implements any function of its inputs at the same cost, minimizing a function
by hand no longer saves you anything below the LUT boundary. The K-map work from Lab 2 still
matters for understanding, and it still matters at scale, but the synthesizer does that job
now.

### 3.2 The Board

The PYNQ-Z2 is built around a Zynq XC7Z020, which contains both an ARM processor system
(PS) and FPGA fabric (PL). This course uses **only the PL**. You will not create a block
design or instantiate the processor. Plain Verilog and a constraints file are enough.

User I/O connected to the PL:

| Item | Count | Notes |
|---|---|---|
| Slide switches | 2 | `SW0`, `SW1` |
| Push buttons | 4 | `BTN0`–`BTN3`, high when pressed |
| Green LEDs | 4 | `LD0`–`LD3` |
| RGB LEDs | 2 | `LD4`, `LD5`, three pins each |

Two switches is not many. Designs in this course use the four push buttons as additional
inputs, which is why every design here fits in six inputs or fewer.

### 3.3 Constraints

Vivado has no idea which port of your module goes to which pin on the chip. An **XDC**
(Xilinx Design Constraints) file tells it. Every line names a port and gives it a package
pin and an I/O standard:

```tcl
set_property -dict { PACKAGE_PIN M20 IOSTANDARD LVCMOS33 } [get_ports { sw[0] }];
```

The pin assignments for the PYNQ-Z2 user I/O are:

```tcl
## Slide switches
set_property -dict { PACKAGE_PIN M20 IOSTANDARD LVCMOS33 } [get_ports { sw[0] }];
set_property -dict { PACKAGE_PIN M19 IOSTANDARD LVCMOS33 } [get_ports { sw[1] }];

## Push buttons
set_property -dict { PACKAGE_PIN D19 IOSTANDARD LVCMOS33 } [get_ports { btn[0] }];
set_property -dict { PACKAGE_PIN D20 IOSTANDARD LVCMOS33 } [get_ports { btn[1] }];
set_property -dict { PACKAGE_PIN L20 IOSTANDARD LVCMOS33 } [get_ports { btn[2] }];
set_property -dict { PACKAGE_PIN L19 IOSTANDARD LVCMOS33 } [get_ports { btn[3] }];

## Green LEDs
set_property -dict { PACKAGE_PIN R14 IOSTANDARD LVCMOS33 } [get_ports { led[0] }];
set_property -dict { PACKAGE_PIN P14 IOSTANDARD LVCMOS33 } [get_ports { led[1] }];
set_property -dict { PACKAGE_PIN N16 IOSTANDARD LVCMOS33 } [get_ports { led[2] }];
set_property -dict { PACKAGE_PIN M14 IOSTANDARD LVCMOS33 } [get_ports { led[3] }];

## RGB LED LD4
set_property -dict { PACKAGE_PIN N15 IOSTANDARD LVCMOS33 } [get_ports { led4_r }];
set_property -dict { PACKAGE_PIN G17 IOSTANDARD LVCMOS33 } [get_ports { led4_g }];
set_property -dict { PACKAGE_PIN L15 IOSTANDARD LVCMOS33 } [get_ports { led4_b }];
```

The port names in the brackets must match your Verilog **exactly**, including the bit
indices. A mismatch produces an error at implementation, not at synthesis, so it can appear
several minutes after you made the mistake.

`LVCMOS33` means 3.3 V CMOS signaling. Unlike the 5 V TTL levels of the 74LS parts, FPGA
I/O runs at 3.3 V. Never connect a 5 V signal to an FPGA pin.

### 3.4 The Flow

| Step | What it does | Typical output |
|---|---|---|
| Synthesis | Turns Verilog into a netlist of LUTs and flip-flops | Utilization estimate |
| Implementation | Places those onto real fabric sites and routes the wires | Timing report |
| Bitstream | Produces the `.bit` file that configures the chip | `top.bit` |
| Program | Loads the bitstream over JTAG | Board runs your design |

Synthesis errors are language errors. Implementation errors are usually constraint errors.
Knowing which stage failed tells you where to look.

## 4. Pre-Lab

1. Bring your working `majority.v` and `compare2.v` from Lab 8.
2. Write the top-level module you will use in §5.2. It must have ports named to match the
   XDC in §3.3 (`sw[1:0]`, `btn[3:0]`, `led[3:0]`), and it should
   wire the majority voter's
   three inputs to `sw[0]`, `sw[1]`, and `btn[0]`, with the result on `led[0]`.
3. Write the full XDC file you will need for that design. You only need constraint lines
   for the ports your top module actually uses; extra lines for unused ports cause errors.
4. Explain in one sentence why minimizing a function with a K-map saves nothing inside a
   single LUT.

## 5. Lab Work

### 5.1 Board Setup

1. Check the board jumpers with your instructor before applying power. The boot mode jumper
   should be set to **JTAG** and the power source jumper to **USB** for this lab.
2. Connect the micro-USB cable to the PROG/UART port and to your computer.
3. Confirm the board powers up.

### 5.2 Majority Voter on Hardware

1. Create a new RTL project in Vivado. Set the part to **xc7z020clg400-1**.
2. Add `majority.v` from Lab 8 as a design source.
3. Add your top-level module from pre-lab §2 and set it as the top module.
4. Add a constraints source, name it `pynq_z2.xdc`, and enter the lines your design needs.
5. Run synthesis. Fix any errors before continuing.
6. Open the **Report Utilization** view and record how many LUTs and flip-flops your design
   used, and what percentage of the chip that is.
7. Run implementation, then generate the bitstream.
8. Open the Hardware Manager, connect to the target, and program the device.
9. Test all eight input combinations using the two switches and `BTN0`. Record the LED state
   for each and confirm it matches the truth table you measured on the breadboard in Lab 2.

### 5.3 Resource Utilization

Record from the utilization report:

- LUTs used, and the percentage of the 53,200 available
- Flip-flops used
- Bonded IOBs used

Then answer in your report: the majority voter took three gates on the breadboard. How many
LUTs did it take here, and why is that number so small compared with the size of the chip?

### 5.4 Two-Bit Comparator on Hardware

1. Add `compare2.v` from Lab 8 to the project and change your top module to instantiate it
   instead.
2. Map `a[1:0]` to the two slide switches and `b[1:0]` to `BTN0` and `BTN1`. Map the three
   outputs to `led[0]` (`gt`), `led[1]` (`eq`), and `led[2]` (`lt`).
3. Update the XDC for the ports you are now using.
4. Rebuild and reprogram.
5. Test all 16 combinations of `a` and `b`. Record the three LED states for each.
6. Confirm exactly one LED is lit in every case.

### 5.5 Break a Constraint on Purpose

1. Change one pin assignment in your XDC to a different valid pin. For example, move
   `led[0]` from `R14` to `P14`.
2. Rebuild the bitstream and reprogram.
3. Record what the board does now. Note in your report that the Verilog never changed, and
   what that tells you about the role of the constraints file.
4. Restore the correct pin.

### 5.6 An Unconstrained Port

1. Delete the constraint line for one port that your design uses.
2. Run implementation and record the exact error or critical warning Vivado produces.
3. Restore the line.

Recognizing this message will save you time every future lab.

### 5.7 Shut Down

Close the hardware target in Vivado before unplugging the board.

## 6. Report

Fill in the report and hand it in at the end of the lab. Include
your top module, your XDC file, the utilization numbers, and the error text from §5.6.
