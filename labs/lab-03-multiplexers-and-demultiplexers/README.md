# Lab 3: Multiplexers and Demultiplexers

**[Download the Lab 3 Report Template (PDF)](EE210L-Lab3-Report-Template.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Use a 74LS151 8:1 multiplexer to select one of eight data inputs.
- Use the enable input to disable a multiplexer output.
- Implement an arbitrary 3-variable logic function using a multiplexer as a lookup table.
- Implement a 4-variable function on an 8:1 multiplexer using input residues.
- Build a demultiplexer from a decoder.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- Function generator and oscilloscope
- ICs: 74LS151 (8:1 MUX), 74LS138 (3-to-8 decoder), 74LS04 (NOT)

## 3. Background

### 3.1 What a Multiplexer Does

A multiplexer is a digitally controlled selector switch. An 8:1 MUX has eight data inputs
`D0`–`D7`, three select inputs `S2 S1 S0`, and one output `Y`. The select inputs form a
3-bit binary number that picks which data input reaches the output:

```
Y = D[ S2 S1 S0 ]
```

If `S2 S1 S0 = 101` (decimal 5), then `Y = D5`, and the other seven inputs are ignored.

### 3.2 74LS151 Pinout

```
                       74LS151 (8:1 MUX)
                        ┌───────∪───────┐
                 D3  1 ─┤               ├─ 16  V_CC
                 D2  2 ─┤               ├─ 15  D4
                 D1  3 ─┤               ├─ 14  D5
                 D0  4 ─┤               ├─ 13  D6
                  Y  5 ─┤               ├─ 12  D7
                  W  6 ─┤               ├─ 11  S0
                 E̅  7 ─┤               ├─ 10  S1
                GND  8 ─┤               ├─  9  S2
                        └───────────────┘
```

- `Y` (pin 5) is the output. `W` (pin 6) is the complement of `Y` — free inversion.
- `E̅` (pin 7) is the **active-low** enable. Tie it to GND to turn the chip on. When it is
  HIGH, `Y` is forced LOW no matter what the data and select inputs are doing.
- `S0` is the least significant select bit and `S2` is the most significant. Note that they
  are **not** in pin order — check §3.2 carefully when wiring.

This is a 16-pin package: **pin 16 is V_CC and pin 8 is GND**, not pins 14 and 7.

### 3.3 A Multiplexer Is a Lookup Table

Here is the useful trick. Feed the variables of a logic function into the select inputs and
wire each data input to a constant 1 or 0 — whatever that row of the truth table requires.
The multiplexer then *is* the function. Any 3-variable function at all, with no gates and
no minimization.

To implement `F(A,B,C)`: wire `A → S2`, `B → S1`, `C → S0`, and tie `Dn` to +5 V for every
row `n` where `F = 1`, and to GND for every row where `F = 0`.

### 3.4 Four Variables on an Eight-Input MUX

You can go one variable further. Put three of the variables on the select lines and let the
fourth appear on the data inputs. Group the truth table into pairs of rows that differ only
in the fourth variable `D`. Each pair collapses to one data input, which must be wired to
one of four things:

| Both rows of the pair | Wire `Dn` to |
|---|---|
| F = 0 and F = 0 | GND |
| F = 1 and F = 1 | +5 V |
| F follows D | `D` |
| F is the opposite of D | `D'` (through an inverter) |

This is called the **residue** method, and it is how a single 8:1 MUX handles any
4-variable function.

### 3.5 Demultiplexers

A demultiplexer is the reverse of a multiplexer: one data input is routed to one of many
outputs, chosen by the select lines. A decoder with an enable input *is* a demultiplexer —
feed your data into the enable pin and the select code picks which output it appears on.
You will do this with the 74LS138 in §5.5. Its pinout is in
[Lab 4, §3.2](../lab-04-decoders-and-encoders/README.md#32-74ls138-pinout).

## 4. Pre-Lab

1. Write the truth table for `F(A,B,C) = A'BC + AB'C' + AB` and list which of `D0`–`D7`
   must be tied HIGH to implement it on a 74LS151.
2. For the 4-variable function `G(A,B,C,D) = Σm(0, 2, 5, 7, 8, 10, 13, 15)`, build the
   8-row residue table described in §3.4 with `A → S2`, `B → S1`, `C → S0`. For each of the
   eight data inputs, state whether it must be tied to 0, 1, `D`, or `D'`.
3. Sketch how you would wire the 74LS138 as a 1-to-8 demultiplexer.

## 5. Lab Work

### 5.1 Basic Multiplexer Operation

1. With the power off, place the 74LS151. Wire **pin 16 to +5 V** and **pin 8 to GND**.
2. Tie the enable `E̅` (pin 7) to GND so the chip is active.
3. Set up a distinctive fixed pattern on the eight data inputs: tie `D0`, `D2`, `D4`, `D6`
   to +5 V and `D1`, `D3`, `D5`, `D7` to GND.
4. Have the circuit checked, then power on.
5. Step the select inputs `S2 S1 S0` through all eight combinations. For each, record the
   measured voltage at `Y` (pin 5) and at `W` (pin 6), and the logic level of each.
6. Confirm that `Y` reproduces the pattern you wired and that `W` is always its complement.

### 5.2 The Enable Input

With the select inputs set to a combination that gives `Y = 1`, take `E̅` (pin 7) HIGH.
Record the output. Return `E̅` to GND and confirm the output comes back. Note in your report
what the enable does and why a shared bus needs one.

### 5.3 A 3-Variable Function on the MUX

1. Power off. Rewire the eight data inputs according to your pre-lab answer for
   `F(A,B,C) = A'BC + AB'C' + AB`.
2. Wire `A → S2` (pin 9), `B → S1` (pin 10), `C → S0` (pin 11).
3. Power on and step through all eight input combinations. Record the measured output and
   logic level for each.
4. Confirm the circuit matches your pre-lab truth table. Note how many logic gates you used
   to build this function.

### 5.4 A 4-Variable Function on the MUX

1. Power off. Rewire the data inputs according to your residue table from pre-lab §2. Two
   of them need `D'`, so wire `D` through an inverter on the 74LS04 and take the inverted
   signal to those pins.
2. Wire `A → S2`, `B → S1`, `C → S0`, and bring `D` in on the data inputs as your table
   requires.
3. Power on and step through all 16 combinations of `A B C D`. Record the measured output
   and logic level for each.
4. Compare against the minterm list for `G`.

### 5.5 Decoder as a Demultiplexer

1. Power off. Place the 74LS138 (**pin 16 to +5 V, pin 8 to GND**).
2. Tie `E̅1` (pin 4) and `E̅2` (pin 5) to GND. These are active-low enables.
3. Pin 6 (`E3`) is the active-HIGH enable — this is your **data input**.
4. Wire the address inputs `A0` (pin 1), `A1` (pin 2), `A2` (pin 3) to the rails.
5. Power on. Set the address to `010` and toggle the data input on pin 6 between GND and
   +5 V. Record the voltage on every output `Y0`–`Y7` for both data values.
6. Repeat for address `101`.
7. Note in your report which output responded to the data and what the other seven did.
   The 74LS138 outputs are **active LOW** — the selected output goes to 0, not 1.

### 5.6 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report template linked at the top of this page and submit it on Canvas.
