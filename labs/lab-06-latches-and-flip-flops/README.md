# Lab 6: Latches and Flip-Flops

**[Download the Lab 6 Report Template (PDF)](EE210L-Lab6-Report-Template.pdf)**

Check Canvas for deliverables, deadlines, and grading rubric.

## 1. Objectives

- Build an SR latch from NAND gates and find its forbidden state.
- Use a 74LS279 quad SR latch.
- Distinguish level-sensitive latches from edge-triggered flip-flops.
- Operate a 74LS74 D flip-flop and a 74LS73 JK flip-flop.
- Measure propagation delay with the oscilloscope.
- Build a toggle flip-flop and observe frequency division.

## 2. Equipment and Parts

- Breadboard, jumper wires, DC power supply, DMM
- Function generator and oscilloscope
- ICs: 74LS00 (NAND), 74LS279 (quad SR latch), 74LS74 (dual D flip-flop), 74LS73 (dual JK flip-flop)

## 3. Background

### 3.1 Storage from Feedback

Everything so far has been combinational — the output depended only on the present inputs.
Feed a gate's output back to its own input and the circuit gains **memory**: the output now
depends on what happened before.

Cross-couple two NAND gates and you get an SR latch:

```
        S̅ ──┐
            ├─NAND──┬── Q
        ┌───┘       │
        │   ┌───────┘
        │   │
        └───┼───────┐
            │       │
        R̅ ──┴─NAND──┴── Q̅
```

With `S̅` and `R̅` both HIGH the latch holds whatever it was last set to. Pulling `S̅` LOW
sets `Q = 1`; pulling `R̅` LOW resets `Q = 0`.

### 3.2 The Forbidden State

Pull `S̅` and `R̅` LOW at the same time and both outputs go HIGH — so `Q` and `Q̅` are equal,
which contradicts what those labels mean. Worse, when you release both inputs together the
latch settles into whichever state wins a race between the two gates. Which one wins
depends on manufacturing tolerances and temperature, so the result is genuinely
unpredictable. That is why this input combination is called **forbidden**, and why every
practical flip-flop design makes it impossible to reach.

### 3.3 Level-Sensitive vs Edge-Triggered

A **latch** is transparent: while its enable is asserted, the output follows the input
continuously. Whatever noise appears on `D` during that window lands in the latch.

A **flip-flop** samples its input only at a clock **edge** — an instant rather than a
window. Everything in a synchronous digital system is built from edge-triggered flip-flops
for exactly this reason: it confines the moment when data can change to a single, known
point in time.

The 74LS74 is positive-edge-triggered: it captures `D` on the rising edge of the clock and
ignores it the rest of the time.

### 3.4 Setup and Hold Time

Data cannot arrive at the same instant as the clock edge. It has to be stable for a
**setup time** before the edge and a **hold time** after it. Violate either and the
flip-flop can enter a **metastable** state, sitting at an invalid voltage between 0 and 1
for an unpredictable time before falling to one side. Timing analysis in real design is
mostly about proving setup and hold are never violated.

### 3.5 74LS74 Pinout (Dual D Flip-Flop)

```
                    74LS74 (dual D flip-flop)
                        ┌───────∪───────┐
            1CLR̅   1 ─┤               ├─ 14  V_CC
               1D   2 ─┤               ├─ 13  2CLR̅
             1CLK   3 ─┤               ├─ 12  2D
            1PRE̅   4 ─┤               ├─ 11  2CLK
               1Q   5 ─┤               ├─ 10  2PRE̅
               1Q̅   6 ─┤               ├─  9  2Q
              GND   7 ─┤               ├─  8  2Q̅
                        └───────────────┘
```

`PRE̅` (preset) and `CLR̅` (clear) are **active-low, asynchronous** — they force the output
immediately, ignoring the clock. **Tie both to +5 V** whenever you are not deliberately
using them, or the flip-flop will not respond to its clock at all.

### 3.6 74LS73 Pinout (Dual JK Flip-Flop)

```
                    74LS73 (dual JK flip-flop)
                        ┌───────∪───────┐
             1CLK   1 ─┤               ├─ 14  1J
            1CLR̅   2 ─┤               ├─ 13  1Q̅
               1K   3 ─┤               ├─ 12  1Q
             V_CC   4 ─┤               ├─ 11  GND
             2CLK   5 ─┤               ├─ 10  2K
            2CLR̅   6 ─┤               ├─  9  2Q
               2J   7 ─┤               ├─  8  2Q̅
                        └───────────────┘
```

> **Warning.** The 74LS73 does **not** use the usual power pins. **V_CC is pin 4 and GND is
> pin 11.** Wiring it like a normal 14-pin chip will destroy it. Check this twice before
> you apply power.

The JK flip-flop removes the forbidden state: where SR would be illegal, `J = K = 1`
toggles the output instead. Its behavior:

| J | K | Q after clock edge |
|---|---|---|
| 0 | 0 | hold |
| 0 | 1 | 0 (reset) |
| 1 | 0 | 1 (set) |
| 1 | 1 | toggle |

### 3.7 74LS279 Pinout (Quad SR Latch)

```
                     74LS279 (quad S̅R̅ latch)
                        ┌───────∪───────┐
              1R̅   1 ─┤               ├─ 16  V_CC
             1S̅1   2 ─┤               ├─ 15  4S̅
             1S̅2   3 ─┤               ├─ 14  4R̅
               1Q   4 ─┤               ├─ 13  4Q
              2R̅   5 ─┤               ├─ 12  3S̅2
              2S̅   6 ─┤               ├─ 11  3S̅1
               2Q   7 ─┤               ├─ 10  3R̅
              GND   8 ─┤               ├─  9  3Q
                        └───────────────┘
```

Latches 1 and 3 have two set inputs that are internally ANDed; tie the unused one HIGH.
16-pin package: **pin 16 = V_CC, pin 8 = GND**.

## 4. Pre-Lab

1. Draw the NAND SR latch and write its complete state table, including the forbidden row.
2. Write the characteristic table for a D flip-flop and for a JK flip-flop.
3. Explain in one or two sentences why `J = K = 1` is well defined while `S = R = 1` on an
   SR latch is not.
4. If a flip-flop is wired to toggle on every clock edge and you drive it with a 10 kHz
   clock, what frequency appears at `Q`? What appears if you chain two such stages?

## 5. Lab Work

### 5.1 NAND SR Latch

1. With the power off, build the cross-coupled NAND latch of §3.1 using two gates of the
   74LS00. Wire `S̅` and `R̅` to the rails.
2. Power on with both inputs HIGH.
3. Pulse `S̅` LOW and back HIGH. Record `Q` and `Q̅`.
4. Pulse `R̅` LOW and back HIGH. Record `Q` and `Q̅`.
5. Return both inputs HIGH and confirm the latch holds its last state. Move the input wires
   away and back several times to convince yourself it is really storing a bit.
6. **Forbidden state.** Take both `S̅` and `R̅` LOW at once. Record `Q` and `Q̅`, noting that
   they are now equal. Release both simultaneously and record where the latch lands. Repeat
   this five times and record the result each time.

### 5.2 74LS279 Quad SR Latch

1. Power off. Place the 74LS279 (**pin 16 to +5 V, pin 8 to GND**).
2. Use latch 2, which has a single set input: `2R̅` (pin 5), `2S̅` (pin 6), `2Q` (pin 7).
3. Power on and exercise set, reset, and hold. Record `Q` for each input combination and
   compare it with the discrete latch you just built.

### 5.3 D Flip-Flop

1. Power off. Place the 74LS74 (**pin 14 to +5 V, pin 7 to GND**).
2. **Tie `1PRE̅` (pin 4) and `1CLR̅` (pin 1) to +5 V.**
3. Drive `1CLK` (pin 3) from the function generator: 1 kHz square wave, 0 V to 5 V. Verify
   the levels on the scope **before** connecting it to the chip.
4. Wire `1D` (pin 2) to the rails so you can set it by hand.
5. Power on. Put `D` HIGH, let a few clock edges pass, and record `Q`. Put `D` LOW, wait,
   and record `Q` again.
6. **Transparency check.** Set the clock aside for a moment: disconnect the generator and
   hold `CLK` at a fixed HIGH level, then change `D` several times. Record whether `Q`
   follows `D`. Repeat with `CLK` held LOW. Note in your report what this proves about
   edge-triggering.
7. **Asynchronous inputs.** With the clock running and `D` HIGH, pulse `1CLR̅` LOW. Record
   what happens to `Q` and whether it waited for a clock edge.

### 5.4 Propagation Delay

1. Put scope channel 1 on `1CLK` (pin 3) and channel 2 on `1Q` (pin 5).
2. Raise the clock to 100 kHz and set `D` so the output is actually changing — the easiest
   way is the toggle wiring of §5.6.
3. Trigger on the rising edge of the clock and expand the horizontal scale until you can see
   the gap between the clock edge and the output transition.
4. Use the cursors to measure the delay from the clock's rising edge to the 50% point of
   the `Q` transition. Record the value and capture the waveform for your report.
5. Compare your measurement with the datasheet value for the 74LS74.

### 5.5 JK Flip-Flop

1. Power off. Place the 74LS73 — **V_CC on pin 4, GND on pin 11.** Check it twice.
2. Tie `1CLR̅` (pin 2) to +5 V.
3. Drive `1CLK` (pin 1) from the function generator at 1 kHz.
4. Wire `1J` (pin 14) and `1K` (pin 3) to the rails.
5. Power on and test all four `J`/`K` combinations. For each, record `1Q` (pin 12) and
   confirm it matches the table in §3.6. For the toggle case, use the scope rather than the
   DMM — the output is a square wave, not a static level.

### 5.6 Toggle Mode and Frequency Division

1. Tie both `1J` and `1K` HIGH so the flip-flop toggles on every clock edge.
2. Put scope channel 1 on the clock and channel 2 on `1Q`. Capture both together.
3. Measure the clock frequency and the `Q` frequency. Record the ratio.
4. Feed `1Q` into the clock input of the second flip-flop on the same chip (`2CLK`, pin 5),
   with `2J` and `2K` tied HIGH and `2CLR̅` (pin 6) to +5 V. Measure the frequency at `2Q`
   (pin 9).
5. Note in your report what a chain of toggle flip-flops is actually counting.

### 5.7 Shut Down

Turn off the supply and the function generator, disconnect the leads, and return the ICs to
your kit.

## 6. Report

Fill in the report template linked at the top of this page and submit it on Canvas. Include
your oscilloscope captures for §5.4 and §5.6.
