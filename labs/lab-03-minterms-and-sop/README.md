# Lab 3: Minterms and SOP Implementation

**[Download the Lab 3 Report (PDF)](EE210L-Lab3-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Write the minterms of a function directly from its truth table.
- Turn a written specification into a truth table, a minterm list, and a canonical
  sum-of-products expression.
- Build a circuit from an SOP expression and confirm it against every row of the truth table.
- Measure the cost of the canonical form against the simplified form, in gates and in packages.

## 2. Equipment and Parts

- Breadboard and jumper wires
- DC power supply
- Digital multimeter (DMM)
- ICs: 74LS08 (AND), 74LS32 (OR), 74LS04 (NOT), 74LS86 (XOR)

## 3. Background

### 3.1 Minterms

A **minterm** is a product term that contains **every** input variable exactly once, either
plain or complemented. For three variables there are eight of them, and each one is true for
exactly one row of the truth table.

Number the minterms by reading the input row as a binary number, with the leftmost variable as
the most significant bit:

| Row | A | B | C | Minterm | Name |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | `A′B′C′` | `m₀` |
| 1 | 0 | 0 | 1 | `A′B′C`  | `m₁` |
| 2 | 0 | 1 | 0 | `A′BC′`  | `m₂` |
| 3 | 0 | 1 | 1 | `A′BC`   | `m₃` |
| 4 | 1 | 0 | 0 | `AB′C′`  | `m₄` |
| 5 | 1 | 0 | 1 | `AB′C`   | `m₅` |
| 6 | 1 | 1 | 0 | `ABC′`   | `m₆` |
| 7 | 1 | 1 | 1 | `ABC`    | `m₇` |

A variable appears plain where that row has a 1 and complemented where it has a 0. That is the
whole rule.

### 3.2 From a Truth Table to a Circuit

Every function is the OR of the minterms of the rows where its output is 1. That sum is the
**canonical sum-of-products** form, written compactly as `Σm( … )`.

The procedure never varies:

1. Write the truth table.
2. Mark every row where the output is 1.
3. Write the minterm for each marked row.
4. OR them together.

This always works, on any function, without cleverness. It is the reason a truth table is a
complete specification of a circuit, not just a description of one.

### 3.3 Why the Canonical Form Is Expensive

The canonical form is correct, but it is rarely what you build. Each minterm of an `n`-variable
function needs an `n`-input AND gate, one OR gate wide enough to take all of them, and an
inverter for every complemented variable. A function with six minterms of three variables costs
about twenty 2-input gates. Simplified with the laws from Lab 2, the same function may cost
three.

In this lab you will build the canonical form once, at small scale, so you can see the procedure
work. After that you will simplify first and build second.

### 3.4 Building Wide Gates from 2-Input Gates

Your chips are all 2-input gates, so a wider gate has to be built from them. Cascade them:

- A 3-input AND `X·Y·Z` is two 2-input ANDs: feed `X` and `Y` into the first, then its output
  and `Z` into the second.
- A 4-input OR is three 2-input ORs, and so on.

An `n`-input gate costs `n − 1` two-input gates. **Count them that way in §5.5.**

### 3.5 Recording Logic Levels

As in Lab 2, record only the logic level, 0 or 1. Read the output with the DMM, black lead on
ground, red lead on the output pin, and convert using the bands from Lab 1 §3.1. A reading in
the undefined band between 0.8 V and 2.0 V means something is wired wrong, most often a missing
power or ground pin.

### 3.6 Pinouts

```
      74LS08 (AND)  ·  74LS32 (OR)  ·  74LS86 (XOR)
              (all three share this layout)
                    ┌───────∪───────┐
             1A  1 ─┤               ├─ 14  V_CC
             1B  2 ─┤               ├─ 13  4B
             1Y  3 ─┤               ├─ 12  4A
             2A  4 ─┤               ├─ 11  4Y
             2B  5 ─┤               ├─ 10  3B
             2Y  6 ─┤               ├─  9  3A
            GND  7 ─┤               ├─  8  3Y
                    └───────────────┘
```

```
                  74LS04 (Hex Inverter)
                    ┌───────∪───────┐
             1A  1 ─┤               ├─ 14  V_CC
             1Y  2 ─┤               ├─ 13  6A
             2A  3 ─┤               ├─ 12  6Y
             2Y  4 ─┤               ├─ 11  5A
             3A  5 ─┤               ├─ 10  5Y
             3Y  6 ─┤               ├─  9  4A
            GND  7 ─┤               ├─  8  4Y
                    └───────────────┘
```

On every 14-pin chip here, **pin 7 is GND and pin 14 is V_CC**, and both must be connected.

## 4. Pre-Lab

**Bring your ICs, breadboard, DMM, and parts kit to lab.**

Complete the pre-lab before you arrive. There is nothing to submit: show me your work at the
start of lab. It must be **hand-written and hand-drawn** in a physical notebook.

You will work with three functions.

**Function G** — two variables: `G(A,B) = Σm(1, 2)`

**Function M** — the majority voter. `M(A,B,C)` is 1 when **two or more** of its three inputs
are 1, and 0 otherwise. This is how redundant systems vote: three sensors report, and the
majority wins.

**Function F** — three variables: `F(A,B,C) = Σm(0, 1, 2, 3, 4, 5)`

1. For **G**, write the full truth table and the canonical SOP expression. Draw the circuit,
   built from minterms exactly as written, and label every chip and pin number.
2. For **M**, write the truth table from the specification above. List the minterms in `Σm( … )`
   form, write the canonical SOP, then simplify it using the laws from Lab 2. Name the law at
   each step. Draw the **simplified** circuit with chips and pins labelled.
3. For **F**, write the truth table, then the canonical SOP, then simplify it. Name the law at
   each step. Draw the **simplified** circuit with chips and pins labelled.
4. Predict the output for every row of the tables in §5.2, §5.3, and §5.4.
5. Count the gates for the canonical and the simplified version of both **M** and **F**, using
   the rule in §3.4. You will enter these in §5.5.

> Function **F** simplifies further than you may expect. If one of the three variables is still
> in your answer that ought not to be, check your work before you wire anything.

## 5. Lab Work

### 5.1 Power Supply Setup

1. With the supply **off**, set it to 5.0 V and set the current limit to about 250 mA.
2. Turn the supply on, measure the output with the DMM, and record the actual voltage. Turn the
   supply back off.
3. Connect the supply to the breadboard power rails: +5 V to the red rail, ground to the blue
   rail.

> **Build every circuit with the power supply off.** Turn it on only after you have checked your
> wiring.

### 5.2 Function G: Building Straight from the Minterms

Here you build the canonical form exactly as written, with no simplification at all.

1. Power off. Place the 74LS08, 74LS32, and 74LS04. Wire **pin 14 to +5 V and pin 7 to GND on
   each chip.**
2. Build your circuit for `G` from the pre-lab: one AND gate per minterm, an OR gate to combine
   them, and an inverter for each complemented input.
3. Bring `A` and `B` out to two jumper wires.
4. Power on and record `G` for all four input combinations.

   | # | A | B |
   |---|---|---|
   | 1 | 0 | 0 |
   | 2 | 0 | 1 |
   | 3 | 1 | 0 |
   | 4 | 1 | 1 |

5. **Now compare against a chip that already does this.** Place the 74LS86, power it, and drive
   one of its gates with the same `A` and `B`. Record its output for the same four rows.
6. The two columns should agree on every row. Note in your report which single gate you just
   built out of five.

### 5.3 Function M: The Majority Voter

1. Power off. Take down circuit G and build the **simplified** majority voter from your pre-lab.
2. Bring `A`, `B`, and `C` out to three jumper wires.
3. Power on and record `M` for all eight combinations.

   | # | A | B | C |
   |---|---|---|---|
   | 1 | 0 | 0 | 0 |
   | 2 | 0 | 0 | 1 |
   | 3 | 0 | 1 | 0 |
   | 4 | 0 | 1 | 1 |
   | 5 | 1 | 0 | 0 |
   | 6 | 1 | 0 | 1 |
   | 7 | 1 | 1 | 0 |
   | 8 | 1 | 1 | 1 |

4. Check every row against the specification in §4, not just against your own prediction. The
   output must be 1 exactly when two or three inputs are 1.

### 5.4 Function F: A Variable That Disappears

1. Power off. Take down the majority voter and build the **simplified** `F` from your pre-lab.
2. Power on and record `F` for all eight combinations, in the same order as §5.3.
3. **Then test the claim directly.** Set `A` and `B` to 0 and toggle `C` back and forth several
   times, watching the output. Repeat with `A = 0, B = 1`, and again with `A = 1, B = 1`.
4. Record in your report what `C` does to the output, and explain how six minterms that all
   mention `C` produced a function that does not depend on it at all.

### 5.5 Cost of the Canonical Form

Using the gate counts from your pre-lab (§3.4 rule: an `n`-input gate costs `n − 1` two-input
gates), fill in the comparison table in the report for both `M` and `F`.

You built the canonical form only for `G`, the smallest of the three. Note in your report how
many gates the canonical form of `F` would have taken, and how many packages that is.

### 5.6 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report and hand it in at the end of the lab.

Your report must include the truth tables and measured outputs for all three functions, your
simplified expressions for `M` and `F`, and the canonical-versus-simplified gate counts from
§5.5.
