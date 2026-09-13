# Lab 7: Shift Registers and Counters

**[Download the Lab 7 Report (PDF)](EE210L-Lab7-Report.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Build a ripple counter from JK flip-flops and see why it is called "ripple".
- Use a 74LS163 synchronous counter and compare its behavior with the ripple counter.
- Use synchronous load and clear to make a counter with a shortened modulus.
- Use a 74LS90 decade counter.
- Operate a 74LS194 universal shift register in all four modes.
- Build a ring counter and a Johnson counter.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- Function generator and oscilloscope
- ICs: 74LS73 (dual JK flip-flop), 74LS163 (synchronous 4-bit counter), 74LS90 (decade counter),
  74LS194 (universal shift register), 74LS00 (NAND)

## 3. Background

### 3.1 Ripple vs Synchronous Counters

In a **ripple** (asynchronous) counter, each flip-flop is clocked by the output of the one
before it. Stage 1 toggles, which clocks stage 2, which clocks stage 3, and so on. The
count is correct once everything settles, but the stages do not change at the same instant.
The change ripples down the chain, one propagation delay per stage.

That has a real consequence. Between the clock edge and the moment the last stage settles,
the counter's outputs pass through **transient states that are not part of the count
sequence**. Going from 0111 to 1000, a ripple counter may momentarily show 0110, 0100, and
0000 before landing on 1000. Anything watching those outputs, a decoder for example,
will see brief false pulses called **decoding glitches**.

In a **synchronous** counter, every flip-flop is clocked from the same signal at the same
instant, and combinational logic decides what each one should do next. All bits change
together, so there are no transient states. This is why the 74LS163 exists and why real
designs almost never use ripple counters for anything that other logic watches.

### 3.2 74LS163 Pinout (Synchronous 4-Bit Counter)

```
                74LS163 (synchronous 4-bit binary counter)
                        ┌───────∪───────┐
             CLR̅   1 ─┤               ├─ 16  V_CC
              CLK   2 ─┤               ├─ 15  RCO
                A   3 ─┤               ├─ 14  QA
                B   4 ─┤               ├─ 13  QB
                C   5 ─┤               ├─ 12  QC
                D   6 ─┤               ├─ 11  QD
              ENP   7 ─┤               ├─ 10  ENT
              GND   8 ─┤               ├─  9  LOAD̅
                        └───────────────┘
```

- `QA` is the least significant output bit, `QD` the most significant.
- `A`–`D` are the parallel load data inputs.
- `CLR̅` (pin 1) and `LOAD̅` (pin 9) are both **synchronous**: they take effect on the next
  rising clock edge, not immediately. This is the key difference between the 74LS163 and the
  otherwise-identical 74LS161, whose clear is asynchronous.
- `ENP` and `ENT` must both be HIGH for the counter to count.
- `RCO` (ripple carry out) goes HIGH when the count reaches 1111 and `ENT` is HIGH. It is
  meant to drive the enable of the next counter in a cascade.

### 3.3 Shortening the Modulus

A 74LS163 naturally counts 0 to 15. To make it count a shorter sequence, detect the state
where you want it to restart and use that to assert `CLR̅` or `LOAD̅`.

For a mod-10 counter (0 through 9), decode state 9 (`1001`) with a NAND gate on `QD` and
`QA`, and feed the result into `CLR̅`. On the next rising edge, the synchronous clear takes
the counter to 0000 instead of 10. Because the clear is synchronous, the counter never
actually displays state 10, not even for a nanosecond. Doing this on a chip with an
asynchronous clear produces a brief glitch at state 10, which is a classic source of hard-
to-find bugs.

### 3.4 74LS90 Pinout (Decade Counter)

```
                      74LS90 (decade counter)
                        ┌───────∪───────┐
              CKB   1 ─┤               ├─ 14  CKA
           R0(1)   2 ─┤               ├─ 13  NC
           R0(2)   3 ─┤               ├─ 12  QA
               NC   4 ─┤               ├─ 11  QD
             V_CC   5 ─┤               ├─ 10  GND
           R9(1)   6 ─┤               ├─  9  QB
           R9(2)   7 ─┤               ├─  8  QC
                        └───────────────┘
```

> **Warning.** The 74LS90 also breaks the usual power convention: **V_CC is pin 5 and GND
> is pin 10.** Check before applying power.

The 74LS90 is really two independent counters in one package: a ÷2 section (clock `CKA`,
output `QA`) and a ÷5 section (clock `CKB`, outputs `QB QC QD`). To get a BCD decade
counter, drive `CKA` and wire `QA` into `CKB`, which chains them into ÷10. All four reset
pins must be held LOW for the counter to run.

### 3.5 74LS194 Pinout (Universal Shift Register)

```
              74LS194 (4-bit bidirectional universal shift register)
                        ┌───────∪───────┐
             CLR̅   1 ─┤               ├─ 16  V_CC
           SR SER   2 ─┤               ├─ 15  QA
                A   3 ─┤               ├─ 14  QB
                B   4 ─┤               ├─ 13  QC
                C   5 ─┤               ├─ 12  QD
                D   6 ─┤               ├─ 11  CLK
           SL SER   7 ─┤               ├─ 10  S1
              GND   8 ─┤               ├─  9  S0
                        └───────────────┘
```

The two mode-select pins choose what happens on each rising clock edge:

| S1 | S0 | Mode |
|---|---|---|
| 0 | 0 | Hold: outputs do not change |
| 0 | 1 | Shift right: `QA → QB → QC → QD`, `SR SER` enters at `QA` |
| 1 | 0 | Shift left: `QD → QC → QB → QA`, `SL SER` enters at `QD` |
| 1 | 1 | Parallel load: `A B C D` are captured |

`CLR̅` (pin 1) is asynchronous and active low. Tie it HIGH except when clearing.

### 3.6 Ring and Johnson Counters

Feed a shift register's output back to its own serial input and it counts by circulating a
pattern rather than by binary arithmetic.

- **Ring counter**: `QD` back to `SR SER`. Load a single 1 and it walks around the four
  positions: `1000 → 0100 → 0010 → 0001 → 1000`. Four states, and each state is already
  decoded on its own output line, which is why ring counters are used to sequence machinery
  without any decoding logic.
- **Johnson counter**: `Q̅D` back to `SR SER` instead. The inverted feedback gives eight
  states from four flip-flops: `0000 → 1000 → 1100 → 1110 → 1111 → 0111 → 0011 → 0001 →
  0000`. Twice the states of a ring counter, and only two-input gates are needed to decode
  any one of them.

Because the 74LS194 has no `Q̅` outputs, take `QD` through an inverter for the Johnson
counter, or use a NAND gate wired as an inverter from the 74LS00.

## 4. Pre-Lab

1. Draw a 2-bit ripple counter using both flip-flops of a 74LS73, and write its count
   sequence.
2. Write the full state sequence for a 74LS163 counting from 0000, and mark where `RCO`
   goes HIGH.
3. Work out which two outputs to NAND together to detect state 9 (`1001`) for the mod-10
   circuit of §3.3, and explain why the other two outputs do not need to be included.
4. Write the state sequences for a 4-bit ring counter and a 4-bit Johnson counter.

## 5. Lab Work

### 5.1 Ripple Counter

1. With the power off, wire both JK flip-flops of the 74LS73 in toggle mode
   (`J = K = 1`, `CLR̅` HIGH). Remember: **V_CC on pin 4, GND on pin 11.**
2. Clock the first stage from the function generator at 10 kHz. Wire `1Q` (pin 12) into
   `2CLK` (pin 5).
3. Power on.
4. Put scope channel 1 on the clock and channel 2 on `1Q`. Record the frequency of each.
   Move channel 2 to `2Q` (pin 9) and record that frequency.
5. **Find the ripple.** Put channel 1 on `1Q` and channel 2 on `2Q`. Trigger on the falling
   edge of `1Q` and expand the timebase until you can see that `2Q` changes *after* `1Q`,
   not with it. Measure the delay and capture the waveform.
6. Note in your report how large that delay would grow in a 8-bit ripple counter.

### 5.2 Synchronous Counter

1. Power off. Place the 74LS163 (**pin 16 to +5 V, pin 8 to GND**).
2. Tie `ENP` (pin 7) and `ENT` (pin 10) to +5 V, and `CLR̅` (pin 1) and `LOAD̅` (pin 9) to
   +5 V so the counter simply counts.
3. Clock it from the function generator at about 2 Hz so you can read the outputs with the
   DMM as they change.
4. Power on and record the sequence of `QD QC QB QA` for at least 16 consecutive counts.
   Confirm it wraps from 1111 back to 0000.
5. Record what `RCO` (pin 15) does and at which state it asserts.
6. Raise the clock to 10 kHz. Put channel 1 on the clock and channel 2 on `QA`, then move
   channel 2 to `QD`. Record the frequency at each output.

### 5.3 Mod-10 Counter

1. Power off. Add a NAND gate from the 74LS00 that decodes state 9 as worked out in your
   pre-lab, and feed its output to `CLR̅` (pin 1).
2. Power on with the slow clock. Record the full sequence. Confirm the counter goes
   `... 8 → 9 → 0` and never displays 10.
3. Note in your report why the synchronous clear means state 10 never appears at all, and
   what would be different on a chip with an asynchronous clear.

### 5.4 Decade Counter (74LS90)

1. Power off. Place the 74LS90. **V_CC goes on pin 5 and GND on pin 10.** Check twice.
2. Tie all four reset pins to GND: `R0(1)` (pin 2), `R0(2)` (pin 3), `R9(1)` (pin 6), and
   `R9(2)` (pin 7).
3. Wire `QA` (pin 12) to `CKB` (pin 1) to chain the ÷2 and ÷5 sections into ÷10.
4. Clock `CKA` (pin 14) at about 2 Hz.
5. Power on and record ten consecutive states of `QD QC QB QA`. Confirm it counts 0 through
   9 and wraps.
6. Take `R0(1)` and `R0(2)` HIGH together and record what happens.

### 5.5 Universal Shift Register

1. Power off. Place the 74LS194 (**pin 16 to +5 V, pin 8 to GND**). Tie `CLR̅` (pin 1) HIGH.
2. Clock it from the function generator at about 2 Hz. Wire `S0` (pin 9) and `S1` (pin 10)
   to the rails so you can select modes by hand.
3. Power on.
4. **Parallel load.** Set `S1 S0 = 11`, put `1011` on `A B C D`, and let one clock edge
   pass. Record the outputs.
5. **Hold.** Set `S1 S0 = 00` and let several clock edges pass. Record the outputs and
   confirm nothing changed.
6. **Shift right.** Set `S1 S0 = 01` with `SR SER` (pin 2) tied LOW. Record the outputs
   after each of four clock edges.
7. **Shift left.** Reload `1011`, then set `S1 S0 = 10` with `SL SER` (pin 7) tied LOW.
   Record the outputs after each of four clock edges.

### 5.6 Ring Counter

1. Power off. Wire `QD` (pin 12) back to `SR SER` (pin 2).
2. Power on. Load `1000` using parallel load mode, then switch to shift right.
3. Record the outputs for eight consecutive clock edges. Confirm the pattern circulates
   with a period of four.
4. Note in your report why a ring counter needs no decoding logic.

### 5.7 Johnson Counter

1. Power off. Take `QD` (pin 12) through an inverter, using a spare NAND gate on the 74LS00
   with both inputs tied together, and feed the inverted signal to `SR SER` (pin 2).
2. Power on. Clear the register with `CLR̅`, then shift right.
3. Record the outputs for at least ten consecutive clock edges and confirm the sequence has
   a period of eight.
4. Compare the number of states you get from four flip-flops in a ring counter versus a
   Johnson counter versus a binary counter.

### 5.8 Shut Down

Turn off the supply and the function generator, disconnect the leads, and return the ICs to
your kit.

## 6. Report

Fill in the report linked at the top of this page and submit it on Canvas. Include
your oscilloscope captures for §5.1 and §5.2.
