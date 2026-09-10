# Lab 2: Boolean Simplification and K-Maps

**[Download the Lab 2 Report Template (PDF)](EE210L-Lab2-Report-Template.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Write a sum-of-products (SOP) expression directly from a truth table.
- Minimize a Boolean function using a Karnaugh map.
- Simulate a logic circuit before building it.
- Build a minimized circuit and verify it against the original truth table.
- Compare the hardware cost of a minimized circuit against the unminimized one.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- Computer with the logic simulator specified by your instructor
- ICs: 74LS04 (NOT), 74LS08 (AND), 74LS32 (OR)

Pinouts for these chips are in [Lab 1, §3.3](../lab-01-logic-gates-and-truth-tables/README.md#33-pinouts).

## 3. Background

### 3.1 From Truth Table to SOP

A **minterm** is an AND term that is true for exactly one row of the truth table. For a row
where an input is 1 you use the variable itself; where it is 0 you use its complement. The
SOP expression is the OR of the minterms for every row whose output is 1.

For example, the row `A=1, B=0, C=1 → F=1` contributes the minterm `A·B'·C`.

This expression is always correct, but it is almost never efficient. A 4-variable function
with 10 true rows produces 10 four-input AND gates feeding a 10-input OR gate. That is a
lot of silicon for something that often reduces to two or three gates.

### 3.2 Karnaugh Maps

A K-map is the truth table redrawn so that physically adjacent cells differ in exactly one
variable. That adjacency is what lets you spot terms that cancel.

A 4-variable map is laid out with the row and column labels in **Gray code** order
(`00, 01, 11, 10`) — not counting order. That ordering is the whole point: it guarantees
neighbors differ by one bit.

```
                        CD
                 00    01    11    10
              ┌─────┬─────┬─────┬─────┐
     AB   00  │  0  │  1  │  3  │  2  │
              ├─────┼─────┼─────┼─────┤
          01  │  4  │  5  │  7  │  6  │
              ├─────┼─────┼─────┼─────┤
          11  │ 12  │ 13  │ 15  │ 14  │
              ├─────┼─────┼─────┼─────┤
          10  │  8  │  9  │ 11  │ 10  │
              └─────┴─────┴─────┴─────┘
```

Rules for grouping:

1. Groups must contain 1, 2, 4, 8, ... cells — always a power of two.
2. Groups must be rectangular and contain only 1s (or don't-cares).
3. Bigger groups are better. Each doubling of a group removes one variable from the term.
4. Groups may wrap around the edges of the map — left to right and top to bottom.
5. Groups may overlap. Cover every 1 at least once, using as few and as large groups as
   possible.

### 3.3 Don't-Care Conditions

Some input combinations never occur in a real system. A BCD digit, for instance, never
takes the values 10 through 15. Those rows are marked `X` (don't-care) and you may treat
each one as either 0 or 1 — whichever makes your groups larger. Don't-cares are free
simplification; use them.

## 4. Pre-Lab

Do all of this before you come to lab. You will build these circuits, so bring the results.

### 4.1 Majority Voter

A 3-input majority voter outputs 1 when two or more of its inputs are 1. This is real
engineering — redundant flight and reactor control systems vote three sensors against each
other so that a single failed sensor cannot control the output.

1. Write the complete truth table for `M(A,B,C)`.
2. Write the unminimized SOP expression straight from the truth table.
3. Draw the 3-variable K-map, group it, and write the minimized expression.
4. Count the gates each version needs, remembering that the chips in your kit have only
   **2-input** gates. A 3-input AND must be built from two 2-input ANDs.

### 4.2 Four-Variable Function

Minimize

```
F(A,B,C,D) = Σm(0, 1, 2, 3, 5, 7, 8, 9, 10, 11)
```

1. Fill in the 4-variable K-map above with the 10 minterms.
2. Find the largest legal groups. One of them covers eight cells.
3. Write the minimized expression and confirm it uses no more than three gates.

## 5. Lab Work

### 5.1 Simulate Before You Build

1. Enter your **minimized** majority voter from §4.1 in the simulator.
2. Drive all eight input combinations and record the output for each. Confirm it matches
   the truth table you wrote in the pre-lab.
3. Repeat for your minimized 4-variable function from §4.2. Check all 16 input
   combinations against the minterm list.
4. Save or export a screenshot of each schematic. Both go in your report.

If the simulation does not match your truth table, fix it now. Debugging on the screen is
far faster than debugging on a breadboard.

### 5.2 Build the Majority Voter

1. With the power supply off, build your minimized majority voter on the breadboard.
2. Power both chips you use: pin 14 to +5 V, pin 7 to GND on each.
3. Wire the three inputs to the rails with jumper wires.
4. Power it on.
5. Step through all eight input combinations. Record the measured output voltage and the
   logic level for each.
6. Confirm the result matches your pre-lab truth table.

### 5.3 Build the Four-Variable Function

1. Power off, then build the minimized version of `F` from §4.2.
2. Step through all 16 input combinations and record the measured output voltage and logic
   level for each.
3. Compare against the minterm list. Every minterm in the list must produce a 1, and every
   combination not in the list must produce a 0.

### 5.4 Cost Comparison

For **both** functions, fill in the comparison table in your report: gate count, chip
count, and number of IC packages required for the unminimized SOP versus your minimized
version. You do not have to build the unminimized versions — count them on paper.

### 5.5 Shut Down

Turn off the supply, disconnect the leads, and return the ICs to your kit.

## 6. Report

Fill in the report template linked at the top of this page and submit it on Canvas.

Include your K-maps with the groupings clearly drawn, both simulator screenshots, all
measured data, and the cost comparison.
