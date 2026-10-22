# Lab 7: Multiplexers and Demultiplexers

**[Download the Lab 7 Report (PDF)](EE210L-Lab7-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Use a 74LS151 8:1 multiplexer to select one of eight data inputs.
- Use the enable input to disable a multiplexer output.
- Implement an arbitrary 3-variable logic function using a multiplexer as a lookup table.
- Build a demultiplexer from a decoder and watch it route a signal.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- Function generator and oscilloscope
- ICs: 74LS151 (8:1 MUX), 74LS138 (3-to-8 decoder)
- 3 LEDs and 3 resistors, 470 Ω, for §5.4

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

- `Y` (pin 5) is the output. `W` (pin 6) is the complement of `Y`, giving you free inversion.
- `E̅` (pin 7) is the **active-low** enable. Tie it to GND to turn the chip on. When it is
  HIGH, `Y` is forced LOW no matter what the data and select inputs are doing.
- `S0` is the least significant select bit and `S2` is the most significant. Note that they
  are **not** in pin order, so check §3.2 carefully when wiring.

This is a 16-pin package: **pin 16 is V_CC and pin 8 is GND**, not pins 14 and 7.

### 3.3 A Multiplexer Is a Lookup Table

Here is the useful trick. Feed the variables of a logic function into the select inputs and
wire each data input to a constant 1 or 0, whatever that row of the truth table requires.
The multiplexer then *is* the function. Any 3-variable function at all, with no gates and
no minimization.

To implement `F(A,B,C)`: wire `A → S2`, `B → S1`, `C → S0`, and tie `Dn` to +5 V for every
row `n` where `F = 1`, and to GND for every row where `F = 0`.

### 3.4 Demultiplexers

A demultiplexer is the reverse of a multiplexer: one data input is routed to one of many
outputs, chosen by the select lines. A decoder with an enable input *is* a demultiplexer:
feed your data into the enable pin and the address code picks which output it appears on.
You will do this with the 74LS138 in §5.4.

### 3.5 74LS138 Pinout

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
- The chip is enabled only when `E̅1 = 0` **and** `E̅2 = 0` **and** `E3 = 1`. If any of
  those three conditions fails, all eight outputs stay HIGH.
- The outputs are **active LOW**. The addressed output goes to 0 V; the other seven sit at
  about 3.4 V.
- `Y0` is on pin 15 and `Y7` is on pin 7, so the outputs run **backwards** relative to pin
  order. Wire carefully.
- 16-pin package: **pin 16 = V_CC, pin 8 = GND**.

## 4. Pre-Lab

1. Write the truth table for `F(A,B,C) = A'BC + AB'C' + AB` and list which of `D0`–`D7`
   must be tied HIGH to implement it on a 74LS151.
2. Sketch how you would wire the 74LS138 as a 1-to-8 demultiplexer.

## 5. Lab Work

### 5.1 Basic Multiplexer Operation

1. With the power off, place the 74LS151. Wire **pin 16 to +5 V** and **pin 8 to GND**.
2. Tie the enable `E̅` (pin 7) to GND so the chip is active.
3. Set up a distinctive fixed pattern on the eight data inputs: tie `D0`, `D2`, `D4`, `D6`
   to +5 V and `D1`, `D3`, `D5`, `D7` to GND.
4. Power on.
5. Step the select inputs `S2 S1 S0` through four combinations: `000`, `001`, `100`,
   `101`. For each, record the logic level at `Y` (pin 5) and at `W` (pin 6). Read the
   levels directly: anything near 0 V is a 0 and anything near 3.4 V is a 1.
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

### 5.4 Decoder as a Demultiplexer, with LEDs

This one you watch rather than measure. A slow square wave goes in, and the address
decides which LED it comes out of.

1. Power off. Place the 74LS138 (**pin 16 to +5 V, pin 8 to GND**).
2. Tie `E̅1` (pin 4) and `E̅2` (pin 5) to GND. These are active-low enables.
3. Put an LED on **three** of the outputs: `Y2` (pin 13), `Y5` (pin 10) and `Y7` (pin 7).
   The outputs are **active LOW**, so each LED goes from +5 V, through a **470 Ω**
   resistor, down to its output pin. The LED lights when that output is pulled LOW.
4. Wire the address inputs `A0` (pin 1), `A1` (pin 2), `A2` (pin 3) to the rails.
5. Set the function generator to a **square wave, 0 to 5 V, about 2 Hz**, and feed it into
   pin 6 (`E3`), the active-HIGH enable. This is your data input. Check the amplitude and
   offset on the scope before connecting it: a signal that swings negative will damage the
   chip.
6. Power on and set each of these three addresses in turn:

   | `A2 A1 A0` | Expected output |
   |---|---|
   | `0 1 0` | `Y2` |
   | `1 0 1` | `Y5` |
   | `1 1 1` | `Y7` |

7. For each address, record which LED blinks and what the other two do. **Exactly one
   should blink** at the square wave rate while the other two stay dark.

### 5.5 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report and hand it in at the end of the lab.
