# Lab 6: Adders and Comparators

**[Download the Lab 6 Report (PDF)](EE210L-Lab6-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Use a 74LS283 4-bit adder to add binary numbers.
- Observe unsigned overflow using the carry out.
- Convert the adder into a subtractor using two's complement.
- Use a 74LS85 magnitude comparator and cascade its inputs.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- ICs: 74LS283 (4-bit adder), 74LS85 (4-bit comparator), 74LS86 (XOR)

## 3. Background

### 3.1 Half Adder and Full Adder

A **half adder** adds two bits and produces a sum and a carry:

```
S = A ⊕ B
C = A · B
```

It is called "half" because it has nowhere to accept a carry coming in from a lower bit
position, which makes it useless for anything but the least significant bit.

A **full adder** adds three bits (`A`, `B`, and a carry in) and produces a sum and a
carry out:

```
S    = A ⊕ B ⊕ C_in
C_out = A·B + C_in·(A ⊕ B)
```

Chain four full adders, each one's carry out feeding the next one's carry in, and you have
a 4-bit adder. The 74LS283 produces the same four sum bits and the same carry out, by the
faster route in §3.2.

### 3.2 Ripple Carry

In a chained adder the carry has to propagate from the least significant bit all the way
to the most significant before the answer is final. Each stage costs a couple of gate
delays, so a 4-bit adder waits on four of them and a 64-bit one on sixteen times as many.
This is the **ripple carry** problem, and defeating it is why real processors use
carry-lookahead adders, which work out every carry straight from the inputs instead of
waiting for the stage below. The 74LS283 is built that way inside, which is why one chip
beats four discrete full adders wired in a chain.

### 3.3 74LS283 Pinout

```
                   74LS283 (4-bit full adder)
                        ┌───────∪───────┐
                 Σ2  1 ─┤               ├─ 16  V_CC
                 B2  2 ─┤               ├─ 15  B3
                 A2  3 ─┤               ├─ 14  A3
                 Σ1  4 ─┤               ├─ 13  Σ3
                 A1  5 ─┤               ├─ 12  A4
                 B1  6 ─┤               ├─ 11  B4
                 C0  7 ─┤               ├─ 10  Σ4
                GND  8 ─┤               ├─  9  C4
                        └───────────────┘
```

- `A1`/`B1`/`Σ1` are the **least** significant bit; `A4`/`B4`/`Σ4` are the most significant.
- `C0` (pin 7) is the carry in. Tie it to GND for plain addition.
- `C4` (pin 9) is the carry out.
- 16-pin package: **pin 16 = V_CC, pin 8 = GND**.

The pin ordering on this chip is scattered, and bits are not grouped together. Build a wiring
table before you touch the breadboard.

### 3.4 Subtraction by Two's Complement

To compute `A − B`, add `A` to the two's complement of `B`:

```
A − B = A + (B' + 1)
```

You get `B'` by running each bit of `B` through an XOR gate with a control line held HIGH
(an XOR with one input HIGH is an inverter, and with one input LOW it passes the signal
through unchanged). You get the `+1` for free by setting the carry in `C0` to 1.

So the same chip does both operations, controlled by a single line:

| Control | XOR inputs | C0 | Operation |
|---|---|---|---|
| 0 | pass B through | 0 | A + B |
| 1 | invert B | 1 | A − B |

This is how a real ALU handles addition and subtraction with one adder.

### 3.5 Magnitude Comparator

The 74LS85 compares two 4-bit numbers and asserts one of three outputs: `A > B`, `A = B`,
or `A < B`. It also has three **cascade inputs** with the same names. Those exist so that
comparators can be chained to compare wider numbers: the cascade inputs of the
more-significant chip resolve ties from the less-significant one.

For a standalone 4-bit comparison, the cascade inputs must be set to
`A>B = 0`, `A<B = 0`, `A=B = 1`. Getting this wrong is the usual reason a comparator gives
nonsense for equal inputs.

### 3.6 74LS85 Pinout

```
               74LS85 (4-bit magnitude comparator)
                        ┌───────∪───────┐
                 B3  1 ─┤               ├─ 16  V_CC
         A<B (casc)  2 ─┤               ├─ 15  A3
         A=B (casc)  3 ─┤               ├─ 14  B2
         A>B (casc)  4 ─┤               ├─ 13  A2
         A>B (out)   5 ─┤               ├─ 12  A1
         A=B (out)   6 ─┤               ├─ 11  B1
         A<B (out)   7 ─┤               ├─ 10  A0
                GND  8 ─┤               ├─  9  B0
                        └───────────────┘
```

`A0`/`B0` are the least significant bits. Pins 2, 3, 4 are **inputs** (cascade); pins 5, 6,
7 are the **outputs**. Check each pin number against this diagram as you wire it.

## 4. Pre-Lab

1. Write the truth tables for a half adder and a full adder.
2. Work out `9 + 6` and `9 + 11` in 4-bit binary by hand. For each, state the 4-bit sum and
   the carry out, and say whether the 4-bit result alone is correct.
3. Work out `12 − 5` using the two's complement method of §3.4. Show `B'`, the carry in,
   and the final sum.
4. State the cascade input values needed for a standalone 74LS85 comparison.

## 5. Lab Work

### 5.1 Four-Bit Addition

1. Power off. Place the 74LS283 (**pin 16 to +5 V, pin 8 to GND**) and tie `C0` (pin 7) to
   GND.
2. Wire the eight data inputs to the rails. Build the wiring table first, because the pin order is
   not sequential.
3. Power on. Enter `A = 9 (1001)` and `B = 6 (0110)`. Record `Σ4 Σ3 Σ2 Σ1` and `C4`.
4. Repeat for `A = 9`, `B = 11 (1011)`.
5. For both cases, state whether the 4-bit sum alone is the correct answer, and how `C4`
   tells you when it is not.

### 5.2 Adder/Subtractor

1. Power off. Insert four XOR gates from the 74LS86 between your `B` input switches and the
   `B` inputs of the 74LS283. Tie the second input of all four XOR gates to a single
   control line, and tie that same control line to `C0` (pin 7).
2. Power on. With the control line LOW, verify the circuit still adds: enter `A = 7`,
   `B = 3` and confirm you read 10.
3. Take the control line HIGH. With the same inputs, the circuit should now compute
   `7 − 3 = 4`. Record the result.
4. Compute `12 − 5`. Record the 4-bit output and `C4`.
5. Note in your report what `C4` means during subtraction. It is no longer an overflow
   flag, it tells you whether the result came out non-negative.

### 5.3 Magnitude Comparator

1. Power off. Place the 74LS85 (**pin 16 to +5 V, pin 8 to GND**).
2. Set the cascade inputs for standalone operation: pin 4 (`A>B` in) to GND, pin 2 (`A<B`
   in) to GND, pin 3 (`A=B` in) to +5 V.
3. Wire the `A` and `B` inputs to the rails.
4. Power on and test at least six input pairs, including one where `A > B`, one where
   `A < B`, one where `A = B`, and one pair that differs only in the least significant bit.
   Record all three outputs for each pair.

### 5.4 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report and hand it in at the end of the lab.
