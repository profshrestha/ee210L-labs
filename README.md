# EE 210L: Digital Circuits and Logic Design Lab

Lab instructions and report templates for EE 210L.

Check Canvas for deliverables, deadlines, and grading rubric.

## Labs

The release order is the one in the `main` branch README, and the `#` column below follows it.
Folder names still carry the number each lab had when it was drafted, so a folder is renamed at
the point it is released. Labs 1 through 6 are already out and live on `main`.

| # | Lab | Draft folder | Key Parts |
|---|-----|---|-----------|
| 7 | [Multiplexers and Demultiplexers](labs/lab-03-multiplexers-and-demultiplexers/README.md) | `lab-03-...` | 74LS151, 74LS138 |
| 8 | [Decoders and Encoders](labs/lab-04-decoders-and-encoders/README.md) | `lab-04-...` | 74LS138, 74LS148 |
| 9 | [Latches and Flip-Flops](labs/lab-06-latches-and-flip-flops/README.md) | `lab-06-...` | 74LS279, 74LS74, 74LS73 |
| 10 | [Shift Registers and Counters](labs/lab-07-shift-registers-and-counters/README.md) | `lab-07-...` | 74LS194, 74LS163, 74LS90 |
| 11 | [Counters and Memory on the FPGA](labs/lab-11-counters-and-memory-on-fpga/README.md) | `lab-11-...` | PYNQ-Z2 |

Each lab's report PDF is rendered from HTML in [`report-src/`](report-src/). A released lab
takes three files to `main`: the lab page, the report PDF, and that HTML. Until then all three
stay here.

### Drafted but not in the release plan

These predate the current sequence and are held in case the plan changes.

| Lab | Draft folder |
|-----|---|
| [Boolean Simplification and K-Maps](labs/lab-02-boolean-simplification-and-k-maps/README.md) | superseded by Lab 2 and Lab 4 on `main` |
| [Combinational Logic in Verilog](labs/lab-08-combinational-logic-in-verilog/README.md) | `lab-08-...` |
| [Sequential Logic in Verilog](labs/lab-09-sequential-logic-in-verilog/README.md) | `lab-09-...` |
| [First FPGA Design](labs/lab-10-first-fpga-design/README.md) | `lab-10-...` |

## Lab Equipment

Each station has a digital multimeter (DMM), DC power supply, oscilloscope, function generator, and a computer with a Verilog toolchain.

Labs 8 and 9 are simulation only. Labs 10 and 11 use the **PYNQ-Z2** board (Xilinx Zynq XC7Z020, part `xc7z020clg400-1`) with Vivado.

## Parts Kit

| Part | Function |
|------|----------|
| 74LS00 | Quad 2-input NAND gate |
| 74LS02 | Quad 2-input NOR gate |
| 74LS04 | Hex inverter (NOT) |
| 74LS08 | Quad 2-input AND gate |
| 74LS32 | Quad 2-input OR gate |
| 74LS86 | Quad 2-input XOR gate |
| 74LS73 | Dual JK flip-flop |
| 74LS74 | Dual D flip-flop |
| 74LS85 | 4-bit magnitude comparator |
| 74LS90 | Decade counter |
| 74LS138 | 3-to-8 decoder |
| 74LS148 | 8-to-3 priority encoder |
| 74LS151 | 8:1 multiplexer |
| 74LS163 | Synchronous 4-bit counter |
| 74LS194 | Universal shift register |
| 74LS279 | Quad SR latch |
| 74LS283 | 4-bit full adder |

## Before Every Lab

1. Read the lab instructions and work through any pre-lab calculations **before** you arrive.
2. Download the lab report template from the lab folder.
3. Bring your parts kit.
