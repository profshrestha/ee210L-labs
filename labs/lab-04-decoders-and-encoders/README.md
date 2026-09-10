# Lab 4: Decoders and Encoders

**[Download the Lab 4 Report Template (PDF)](EE210L-Lab4-Report-Template.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Use a 74LS138 3-to-8 decoder and verify its active-low outputs.
- Use the three enable inputs to expand and control a decoder.
- Implement a logic function as a sum of decoder outputs.
- Use a 74LS148 8-to-3 priority encoder and observe priority resolution.
- Explain what the `GS` and `EO` outputs are for.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- ICs: 74LS138 (3-to-8 decoder), 74LS148 (8-to-3 priority encoder), 74LS00 (NAND), 74LS04 (NOT)

## 3. Background

### 3.1 Decoders

A decoder takes an n-bit address and activates exactly one of its 2ⁿ outputs. The 74LS138
takes a 3-bit address `A2 A1 A0` and drives one of `Y0`–`Y7`.

The important detail: **the 74LS138 outputs are active LOW**. The selected output goes to
0 V and all seven others sit at about 3.4 V. This is the opposite of what most students
expect, and it is deliberate. Active-low outputs were cheaper and faster to build in
bipolar logic, and they chain naturally into active-low enables on other chips.

### 3.2 74LS138 Pinout

```
                     74LS138 (3-to-8 decoder)
                        ┌───────∪───────┐
                 A0  1 ─┤               ├─ 16  V_CC
                 A1  2 ─┤               ├─ 15  Y0
                 A2  3 ─┤               ├─ 14  Y1
                E̅1  4 ─┤               ├─ 13  Y2
                E̅2  5 ─┤               ├─ 12  Y3
                 E3  6 ─┤               ├─ 11  Y4
                 Y7  7 ─┤               ├─ 10  Y5
                GND  8 ─┤               ├─  9  Y6
                        └───────────────┘
```

- `A0` is the least significant address bit.
- The chip is enabled only when `E̅1 = 0` **and** `E̅2 = 0` **and** `E3 = 1`. If any of those
  three conditions fails, all outputs stay HIGH.
- `Y0` is on pin 15 and `Y7` is on pin 7, so the outputs run **backwards** relative to pin
  order. Wire carefully.
- 16-pin package: **pin 16 = V_CC, pin 8 = GND**.

### 3.3 Decoders Generate Minterms

Each decoder output corresponds to exactly one minterm of the address variables. `Y5` is
low precisely when `A2 A1 A0 = 101`, which is the minterm `A2·A1'·A0`.

So any function of three variables is just the OR of the decoder outputs for its minterms.
But since the outputs are active low, you do not OR them; you **NAND** them. By De
Morgan's law:

```
(Y3' · Y5' · Y6')'  =  Y3 + Y5 + Y6
```

A NAND gate fed by the active-low outputs for minterms 3, 5, and 6 produces an active-high
`F = Σm(3,5,6)`. One decoder plus one NAND gate implements any 3-variable function with up
to four minterms.

### 3.4 Priority Encoders

An encoder is the reverse of a decoder: it takes 2ⁿ input lines and outputs the binary
index of the active one. The obvious problem is what happens when two inputs are active at
once. A **priority** encoder resolves this by always reporting the highest-numbered active
input and ignoring the rest.

This is exactly how interrupt controllers work. Several devices can request service
simultaneously, and the encoder reports the most urgent one.

### 3.5 74LS148 Pinout

```
                   74LS148 (8-to-3 priority encoder)
                        ┌───────∪───────┐
                 I̅4  1 ─┤               ├─ 16  V_CC
                 I̅5  2 ─┤               ├─ 15  E̅O
                 I̅6  3 ─┤               ├─ 14  G̅S
                 I̅7  4 ─┤               ├─ 13  I̅3
                 E̅I  5 ─┤               ├─ 12  I̅2
                 A̅2  6 ─┤               ├─ 11  I̅1
                 A̅1  7 ─┤               ├─ 10  I̅0
                GND  8 ─┤               ├─  9  A̅0
                        └───────────────┘
```

Everything on this chip is **active low**: the inputs, the address outputs, and the status
outputs. An input is "requesting" when it is at 0 V, and the address outputs are the
complement of the binary index.

- `E̅I` (pin 5) enables the chip when LOW.
- `G̅S` (pin 14) goes LOW when at least one input is requesting. It tells you the address
  outputs are meaningful, which matters because address `000` on the outputs is ambiguous
  otherwise.
- `E̅O` (pin 15) goes LOW when the chip is enabled and **no** input is requesting. It is
  meant to drive the `E̅I` of a lower-priority encoder so that two chips cascade into a
  16-input priority encoder.

## 4. Pre-Lab

1. Write the full truth table for the 74LS138: all eight address combinations against all
   eight outputs, using the active-low convention.
2. For `F(A2,A1,A0) = Σm(1, 2, 4, 7)`, state which decoder outputs feed the NAND gate.
3. For the 74LS148, write the output code `A̅2 A̅1 A̅0` you expect when inputs `I̅2` and `I̅5`
   are both LOW at the same time. State which one wins and why.
4. Predict the state of `G̅S` and `E̅O` when no input is requesting.

## 5. Lab Work

### 5.1 Decoder Truth Table

1. With the power off, place the 74LS138. Wire **pin 16 to +5 V** and **pin 8 to GND**.
2. Enable the chip: `E̅1` (pin 4) to GND, `E̅2` (pin 5) to GND, `E3` (pin 6) to +5 V.
3. Wire the address inputs `A0`, `A1`, `A2` to the rails.
4. Power on.
5. Step the address through all eight combinations. For each, find which output is LOW and
   record its measured voltage, plus the voltage of one output that stayed HIGH.
6. Confirm the pattern matches your pre-lab truth table.

### 5.2 Enable Behavior

With the address set to `011`:

1. Record the state of `Y3` with the chip enabled.
2. Take `E̅1` (pin 4) HIGH. Record `Y3` again.
3. Return `E̅1` to GND, then take `E3` (pin 6) LOW. Record `Y3` again.
4. Note in your report what happens to *all* outputs when the chip is disabled by either
   method, and why a decoder needs three separate enable pins.

### 5.3 A Function from Decoder Outputs

1. Power off. Keep the decoder wired and enabled.
2. Connect the decoder outputs for the minterms of `F = Σm(1, 2, 4, 7)` to the inputs of a
   NAND gate on the 74LS00. You have four minterms and the 74LS00 gates have two inputs
   each, so you will need to combine them, with two NANDs feeding a third stage. Work out the
   arrangement and check it against De Morgan's law before you wire it.
3. Power on and step through all eight address combinations. Record the measured output and
   logic level for each.
4. Confirm the result equals `F`.

### 5.4 Priority Encoder Basics

1. Power off. Place the 74LS148 (**pin 16 to +5 V, pin 8 to GND**).
2. Tie `E̅I` (pin 5) to GND to enable the chip.
3. Tie **all eight** inputs `I̅0`–`I̅7` to +5 V, which means "no request".
4. Power on. Record `A̅2 A̅1 A̅0`, `G̅S`, and `E̅O` for the no-request case.
5. One at a time, take a single input LOW and record the output code, `G̅S`, and `E̅O`.
   Do this for all eight inputs. Remember to return the previous input HIGH first.
6. Convert each output code to the decimal index it represents. The outputs are active low,
   so invert them before converting.

### 5.5 Priority Resolution

1. Take `I̅2` LOW and record the outputs.
2. With `I̅2` still LOW, also take `I̅5` LOW. Record the outputs again.
3. Now also take `I̅7` LOW and record once more.
4. Release `I̅7` and confirm the outputs fall back to reporting input 5.
5. Note in your report which input controls the output when several are active, and why
   this behavior is what an interrupt controller needs.

### 5.6 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report template linked at the top of this page and submit it on Canvas.
