# Lab 5: NAND and NOR as Universal Gates

**[Download the Lab 5 Report (PDF)](EE210L-Lab5-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Express NOT, AND, and OR using NAND gates alone, and again using NOR gates alone.
- Convert a two-level sum-of-products circuit into an all-NAND circuit with the same structure.
- Build the all-NAND versions of `W` and `P` and confirm they produce the Lab 4 results.
- Compare gate count, package count, and chip variety between the two-chip-type and the
  one-chip-type versions.

## 2. Equipment and Parts

- Breadboard and jumper wires
- DC power supply
- Digital multimeter (DMM)
- ICs: 74LS00 (NAND)

Your Lab 4 results for `W` and `P` are needed for comparison. Bring them.

## 3. Background

### 3.1 Universal Gates

Labs 2 through 4 produced expressions in AND, OR, and NOT, and Lab 4 built them from three
different chips: a 74LS08, a 74LS32, and a 74LS04. Three packages for three gates.

A **universal gate** is one from which every other logic function can be built. NAND is universal,
and so is NOR. Either one alone is enough for any circuit you can write, which means a design can
be built from a single chip type.

That matters for two reasons. In a real CMOS process NAND is the cheapest gate to fabricate, so
large designs are mapped onto NAND whether or not they were written that way. And a board built
from one part number is cheaper to stock, assemble, and test than the same board built from three.

### 3.2 The Three Functions from NAND

A NAND gate with both inputs tied together is an inverter:

```
NAND(a, a) = (a·a)′ = a′
```

AND is a NAND followed by that inverter:

```
AND(a, b) = (NAND(a, b))′
```

OR comes from De Morgan. Invert both inputs, then NAND them:

```
NAND(a′, b′) = (a′·b′)′ = a + b
```

| Function | Built from NAND | Gates |
|---|---|---|
| NOT a | `NAND(a, a)` | 1 |
| a · b | `NAND( NAND(a,b), NAND(a,b) )` | 2 |
| a + b | `NAND( NAND(a,a), NAND(b,b) )` | 3 |

### 3.3 The Three Functions from NOR

NOR is the mirror image. Tying its inputs together gives an inverter, inverting its output gives
OR, and inverting both inputs gives AND:

| Function | Built from NOR | Gates |
|---|---|---|
| NOT a | `NOR(a, a)` | 1 |
| a + b | `NOR( NOR(a,b), NOR(a,b) )` | 2 |
| a · b | `NOR( NOR(a,a), NOR(b,b) )` | 3 |

Compare the two tables. NAND produces AND in two gates and OR in three; NOR does the opposite.
That asymmetry decides which gate suits which form, which is the subject of §3.4.

### 3.4 Converting SOP to NAND Directly

Replacing every gate with its NAND equivalent from §3.2 works, but it is wasteful. For a
sum-of-products expression there is a conversion that costs nothing at all.

Take `F = ac + d`. Written as AND-OR it is one AND gate feeding one OR gate. Now replace **both**
levels with NAND:

```
F = NAND( NAND(a, c),  NAND(d, d) )
```

Check it. The first-level NAND gives `(ac)′`. The `d` term has only one literal, so a NAND with
both inputs tied gives `d′`. The second-level NAND then gives

```
( (ac)′ · d′ )′ = ac + d
```

which is `F`. **The structure did not change.** Every AND became a NAND, the OR became a NAND, and
the result is the same function. This is the rule:

> A two-level AND-OR circuit becomes a two-level NAND-NAND circuit with the same wiring. Any
> product term consisting of a single literal needs that literal inverted, which one more NAND
> provides.

The same conversion applies to NOR gates and the dual form, which this course does not use. That
is why this lab builds with NAND and uses NOR only to show that it could.

### 3.5 Counting the Cost

The conversion is not free in gates, only in structure. `F = ac + d` as AND-OR is 2 gates; as
all-NAND it is 3. What falls is the number of packages and the number of distinct part numbers,
and those are what a board is actually billed for.

A 74LS00 holds four NAND gates. Keep the running total in gates and in packages separately, as in
Lab 4 §3.4, and remember that a package is wasted the moment you need its fifth gate.

### 3.6 Recording Logic Levels

As in Labs 1 through 4, record only the logic level, 0 or 1. Read the output with the DMM, black
lead on ground, red lead on the output pin, and convert using the bands from Lab 1 §3.1. A reading
in the undefined band between 0.8 V and 2.0 V means something is wired wrong, most often a missing
power or ground pin.

### 3.7 Pinout

```
                  74LS00 (Quad 2-input NAND)
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

**Pin 7 is GND and pin 14 is V_CC**, and both must be connected.

## 4. Pre-Lab

**Bring your ICs, breadboard, DMM, parts kit, and your Lab 4 pre-lab to lab.**

Complete the pre-lab before you arrive. It must be **hand-written and hand-drawn**. Scan or
photograph your work and **submit it on Canvas before lab begins.** Bring the original with you to
work from during lab.

Logic diagrams in this pre-lab are **logic diagrams only**: gate symbols, inputs, and output. You
do not need chip or pin numbers. You will assign those at the breadboard.

`W` and `P` are the same two functions as Lab 4. Start from the expressions you already have.

**1. Function W as all-NAND.** Carry over your minimum SOP for `W` from Lab 4.

- Write the expression.
- Apply the §3.4 conversion and write `W` as a nest of NAND operations.
- Draw the logic diagram.
- Count the NAND gates and the 74LS00 packages it needs.

**2. Function P as all-NAND.** Carry over the form of `P` you built in Lab 4 §5.3.

- Write the expression.
- Apply the §3.4 conversion and write `P` as a nest of NAND operations.
- Draw the logic diagram.
- Count the NAND gates and the 74LS00 packages it needs.

**3.** Predict the output for every row of the tables in §5.2 and §5.3.

> Your `P` needs one inverted literal. Two of Lab 4's three minimal forms need one inverter and
> the third needs two, so the form you chose decides whether `P` fits in a single 74LS00. Count
> before you assume it does.

## 5. Lab Work

### 5.1 Power Supply Setup

1. With the supply **off**, set it to 5.0 V and set the current limit to about 250 mA.
2. Turn the supply on, measure the output with the DMM, and record the actual voltage. Turn the
   supply back off.
3. Connect the supply to the breadboard power rails: +5 V to the red rail, ground to the blue
   rail.

> **Build every circuit with the power supply off.** Turn it on only after you have checked your
> wiring.

### 5.2 Function W, Built from NAND Only

1. Power off. Place the 74LS00. Wire **pin 14 to +5 V and pin 7 to GND.**
2. Build `W` from your pre-lab §1 diagram.
3. Bring `a`, `b`, `c`, and `d` out to four jumper wires.
4. Power on and record `W` for the twelve rows below, the same twelve as Lab 4 §5.2.

   | # | a | b | c | d |
   |---|---|---|---|---|
   | 1 | 0 | 0 | 0 | 0 |
   | 2 | 0 | 0 | 1 | 0 |
   | 3 | 0 | 0 | 1 | 1 |
   | 4 | 0 | 1 | 0 | 0 |
   | 5 | 0 | 1 | 0 | 1 |
   | 6 | 0 | 1 | 1 | 0 |
   | 7 | 0 | 1 | 1 | 1 |
   | 8 | 1 | 0 | 0 | 0 |
   | 9 | 1 | 0 | 1 | 0 |
   | 10 | 1 | 0 | 1 | 1 |
   | 11 | 1 | 1 | 0 | 1 |
   | 12 | 1 | 1 | 1 | 0 |

5. Copy your Lab 4 measured column for `W` alongside. All twelve rows must agree. Two circuits
   with different gates, different chips, and different gate counts implement one function.

### 5.3 Function P, Built from NAND Only

1. Power off. Take down `W`.
2. Build `P` from your pre-lab §2 diagram on the same 74LS00.
3. Power on and record `P` for the eight digits that can occur, the same eight as Lab 4 §5.3.

   | # | Digit | a | b | c | d |
   |---|---|---|---|---|---|
   | 1 | 0 |  |  |  |  |
   | 2 | 1 |  |  |  |  |
   | 3 | 2 |  |  |  |  |
   | 4 | 3 |  |  |  |  |
   | 5 | 4 |  |  |  |  |
   | 6 | 5 |  |  |  |  |
   | 7 | 6 |  |  |  |  |
   | 8 | 7 |  |  |  |  |

   Fill in `a`, `b`, `c`, and `d` from your Lab 4 table before you apply any inputs.

4. Copy your Lab 4 measured column for `P` alongside and confirm all eight rows agree.
5. Record how many of the four gates on the 74LS00 you used, and how many chips the Lab 4 version
   of the same function needed.

### 5.4 Cost Comparison

Fill in the comparison table in the report. The all-NAND counts come from your pre-lab; the Lab 4
figures come from your Lab 4 pre-lab. Note which version uses more gates and which uses more
packages.

### 5.5 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report and hand it in at the end of the lab.

Your report must include the `W` and `P` tables with the Lab 4 columns beside them and the cost
comparison from §5.4. Your logic diagrams stay on the pre-lab you submitted on Canvas. Do not
redraw them here.
