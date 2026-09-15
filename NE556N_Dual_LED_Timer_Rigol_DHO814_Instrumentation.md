# NE556N Dual LED Timer — Rigol DHO814 Instrumentation Guide

## 1. Purpose

This guide describes how to instrument the **NE556N dual-LED timer project** with a **Rigol DHO814 oscilloscope**.

The goal is not simply to verify that the LEDs flash. The DHO814 can be used to investigate what is actually happening inside each 555 timer:

- Output square waves
- Timing capacitor charge/discharge waveforms
- Trigger and threshold behaviour
- Frequency
- Period
- Duty cycle
- Rise/fall time
- Relationship between the timing waveform and output waveform
- Interaction between the two independent timers

The project operates from a regulated **5 V DC supply**.

---

# 2. Circuit under test

The recommended component values are:

```text
Supply: 5 V

Timer A:
  RA = 10 kΩ
  RB = 68 kΩ
  C  = 10 µF
  Output = NE556 pin 5
  Frequency ≈ 0.99 Hz

Timer B:
  RA = 10 kΩ
  RB = 27 kΩ
  C  = 10 µF
  Output = NE556 pin 9
  Frequency ≈ 2.25 Hz

LED current limiting:
  330 Ω per LED

Control-pin capacitors:
  10 nF on pins 3 and 11

Supply decoupling:
  100 nF between VCC and GND
```

---

# 3. NE556N pins relevant to the oscilloscope

```text
                  NE556N
             ┌─────────────┐
   DISCH A  1│             │14 VCC
   THRES A  2│             │13 DISCH B
    CONT A  3│             │12 THRES B
   RESET A  4│             │11 CONT B
     OUT A  5│             │10 RESET B
    TRIG A  6│             │ 9 OUT B
       GND  7│             │ 8 TRIG B
             └─────────────┘
```

The most useful test points are:

| Test point | NE556 pin | What to observe |
|---|---:|---|
| Timer A output | 5 | ~1 Hz square wave |
| Timer B output | 9 | ~2.25 Hz square wave |
| Timer A timing node | 2/6 | Capacitor charging/discharging |
| Timer B timing node | 12/8 | Capacitor charging/discharging |
| Control A | 3 | Control-voltage node |
| Control B | 11 | Control-voltage node |
| Supply | 14 | 5 V DC |
| Ground | 7 | Scope reference |

---

# 4. Important oscilloscope safety rule

The DHO814 oscilloscope probe ground clips are connected to the oscilloscope's common ground.

For this low-voltage breadboard project, connect the probe ground clips only to the circuit **GND**.

Use:

```text
Probe ground → circuit GND
```

Do not connect a probe ground clip to:

- +5 V
- An NE556 signal pin
- A timing node
- Any other point that is not circuit ground

For this project, all measurements can be made relative to the common circuit ground.

---

# 5. Initial DHO814 setup

Before connecting the probes:

1. Power the DHO814 on.
2. Connect the circuit to the regulated 5 V supply.
3. Confirm the circuit is working.
4. Connect the oscilloscope ground clip to breadboard GND.
5. Connect the probe tip to the desired test point.

Start with:

```text
Vertical:
    2 V/div

Horizontal:
    200 ms/div

Coupling:
    DC

Trigger:
    Edge
    Source = CH1
    Slope = Rising
    Level ≈ 2.5 V
```

At 200 ms/div, the screen shows approximately two seconds across ten horizontal divisions, which makes the ~1 Hz Timer A waveform easy to see.

---

# 6. Experiment 1 — Observe Timer A output

## Connections

```text
DHO814 CH1 probe tip → NE556 pin 5
DHO814 CH1 ground    → circuit GND
```

Pin 5 is Timer A's output.

Expected waveform:

```text
5 V ────┐      ┌──────┐      ┌──────
        │      │      │      │
0 V ────┴──────┘      └──────┘

        <--- approximately 1 second --->
```

## Suggested settings

```text
CH1:
    2 V/div

Time:
    200 ms/div

Trigger:
    CH1
    Rising edge
    Level = 2.5 V
```

The waveform should be approximately a 0 V to 5 V square wave.

---

# 7. Use the DHO814 measurements

Add automatic measurements to CH1.

Useful measurements:

- Frequency
- Period
- Positive pulse width
- Negative pulse width
- Duty cycle
- Vmax
- Vmin
- Vpp
- Rise time
- Fall time

The most important initial measurements are:

```text
Frequency
Period
Duty Cycle
Vmax
Vmin
Vpp
```

Expected approximate results:

```text
Frequency ≈ 0.99 Hz
Period    ≈ 1.01 s
Vmin      ≈ 0 V
Vmax      ≈ 5 V
Vpp       ≈ 5 V
```

Actual values will vary because real resistors, capacitors and the NE556 have tolerances.

---

# 8. Experiment 2 — Observe Timer B

Connect:

```text
DHO814 CH2 probe tip → NE556 pin 9
DHO814 CH2 ground    → circuit GND
```

Leave CH1 connected to pin 5.

Now the DHO814 should show both timers simultaneously.

Expected:

```text
CH1:  ──┐    ┌────┐    ┌────┐
        └────┘    └────┘

CH2:  ─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
       └─┘ └─┘ └─┘ └─┘
```

Timer B should transition considerably faster than Timer A.

Expected frequency:

```text
Timer A ≈ 0.99 Hz
Timer B ≈ 2.25 Hz
```

This is a good first demonstration of the DHO814's ability to compare two signals at once.

---

# 9. Experiment 3 — Measure the timing capacitor

This is the most interesting experiment.

Move CH1 from pin 5 to the Timer A timing node:

```text
CH1 tip → pins 2/6 junction
CH1 ground → GND
```

The timing node is the positive terminal of the 10 µF capacitor.

Instead of a square wave, you should see a repeating charge/discharge waveform.

Conceptually:

```text
Voltage

  3.3 V       /\
              /  \
             /    \
            /      \
  1.7 V    /        \
           /          \
          /            \

          <--- repeating --->
              time
```

For a 5 V supply, the nominal 555 threshold levels are approximately:

```text
1/3 VCC ≈ 1.67 V
2/3 VCC ≈ 3.33 V
```

The capacitor voltage repeatedly moves between these approximate thresholds.

---

# 10. Why the capacitor waveform matters

This is where the project becomes more than an LED flasher.

The NE556 uses the timing capacitor voltage to decide when to change its output state.

Conceptually:

```text
Timing capacitor

       charges
          ↓
      2/3 VCC ───────── threshold
          │
          │       /\
          │      /  \
          │     /    \
      1/3 VCC ───────── trigger
          │
          ↓
       discharges
```

The DHO814 lets you directly see this voltage changing.

This is an excellent way to understand how the classic 555 timer works internally.

---

# 11. Experiment 4 — Observe output and capacitor together

Reconnect:

```text
CH1 → pin 5  (Timer A output)
CH2 → pins 2/6 (Timer A timing capacitor)
```

Use:

```text
Time/div ≈ 200 ms/div
CH1 = 2 V/div
CH2 = 1 V/div
```

Now compare the two waveforms.

You should see:

- The output switching between low and high
- The capacitor voltage continuously charging and discharging
- The output transition occurring when the capacitor reaches the timer's internal threshold levels

This is probably the **single most educational measurement in the entire project**.

---

# 12. Experiment 5 — Trigger on the timing capacitor

Change the trigger source from CH1 to CH2.

```text
Trigger source = CH2
Trigger level ≈ 2.5 V
```

The timing waveform should now be stationary while the output waveform moves in relation to it.

Try different trigger levels:

```text
1.7 V
2.0 V
2.5 V
3.0 V
3.3 V
```

Observe how the display changes.

This provides a practical demonstration of oscilloscope triggering.

---

# 13. Experiment 6 — Measure the timing thresholds

With CH2 connected to the Timer A timing node:

Use cursor measurements to estimate:

```text
Minimum capacitor voltage
Maximum capacitor voltage
```

You should find values in the general vicinity of:

```text
Minimum ≈ 1.67 V
Maximum ≈ 3.33 V
```

Do not expect exact values.

The actual voltage depends on:

- NE556 characteristics
- Supply voltage
- Component tolerances
- Measurement loading
- Breadboard wiring

---

# 14. Experiment 7 — Compare the two timing capacitors

Connect:

```text
CH1 → Timer A timing node (pins 2/6)
CH2 → Timer B timing node (pins 12/8)
```

Now compare the two capacitor waveforms.

Both should swing through approximately the same voltage range, but Timer B should cycle more quickly.

This demonstrates an important concept:

**The frequency is determined by the RC timing network.**

Timer A:

```text
10 kΩ + 68 kΩ + 10 µF
```

Timer B:

```text
10 kΩ + 27 kΩ + 10 µF
```

The smaller RB value makes Timer B run faster.

---

# 15. Experiment 8 — Observe the LED output

The LEDs are driven from the timer outputs through 330 Ω resistors.

You can measure directly at:

```text
Timer A → pin 5
Timer B → pin 9
```

You don't need to probe across the LED for the first experiments.

The output waveform is a much cleaner signal for frequency and timing measurements.

---

# 16. Experiment 9 — Investigate duty cycle

The astable 555 configuration does not normally produce exactly 50% duty cycle.

Measure:

```text
Positive Duty Cycle
Negative Duty Cycle
```

For the standard astable configuration:

```text
tHIGH ≈ 0.693 × (RA + RB) × C

tLOW  ≈ 0.693 × RB × C
```

Therefore:

```text
Timer A:
    tHIGH ≈ 0.693 × (10 kΩ + 68 kΩ) × 10 µF
    tLOW  ≈ 0.693 × 68 kΩ × 10 µF

Timer B:
    tHIGH ≈ 0.693 × (10 kΩ + 27 kΩ) × 10 µF
    tLOW  ≈ 0.693 × 27 kΩ × 10 µF
```

Compare the theoretical values with the DHO814's automatic measurements.

---

# 17. Experiment 10 — Measure rise and fall time

Use the DHO814's automatic:

```text
Rise Time
Fall Time
```

measurement on the output waveform.

Connect:

```text
CH1 → pin 5
```

Zoom in horizontally around one transition.

The output will not transition infinitely quickly.

You can investigate how:

- The NE556 output stage
- Breadboard capacitance
- Probe capacitance
- Wiring
- LED/resistor load

affect the measured transition.

---

# 18. Probe settings

Use the oscilloscope probes in **10X mode**.

Typical setup:

```text
Probe attenuation = 10X
Scope channel attenuation = 10X
```

This reduces probe loading on the circuit.

For the timing-capacitor measurements, this is particularly useful because the capacitor waveform is a relatively high-impedance signal compared with the output.

Use the shortest practical ground connection on the probe when examining faster edges.

For this low-frequency project, a standard ground lead is sufficient for the basic experiments.

---

# 19. Recommended measurement sequence

A good first session with the DHO814 is:

### Stage 1 — Verify the outputs

```text
CH1 → pin 5
CH2 → pin 9
```

Measure:

- Frequency
- Period
- Vpp

### Stage 2 — Investigate Timer A

```text
CH1 → pin 5
CH2 → pins 2/6
```

Observe the relationship between output and capacitor voltage.

### Stage 3 — Investigate Timer B

```text
CH1 → pin 9
CH2 → pins 12/8
```

Repeat the measurements.

### Stage 4 — Compare the timers

```text
CH1 → pin 5
CH2 → pin 9
```

Compare frequencies and periods.

### Stage 5 — Experiment

Change one component at a time and measure the result.

---

# 20. Suggested DHO814 measurement table

Record your actual measurements:

| Measurement | Timer A | Timer B |
|---|---:|---:|
| RA | 10 kΩ | 10 kΩ |
| RB | 68 kΩ | 27 kΩ |
| C | 10 µF | 10 µF |
| Calculated frequency | ~0.99 Hz | ~2.25 Hz |
| Measured frequency | | |
| Calculated period | ~1.01 s | ~0.44 s |
| Measured period | | |
| Measured Vmin | | |
| Measured Vmax | | |
| Measured Vpp | | |
| Duty cycle | | |
| Rise time | | |
| Fall time | | |

This turns the project into a small electronics measurement exercise rather than simply a construction project.

---

# 21. Using cursors

The DHO814 cursors are useful for measurements that aren't necessarily covered by automatic measurements.

Use horizontal cursors to measure:

- Capacitor minimum voltage
- Capacitor maximum voltage
- Approximate 1/3 VCC
- Approximate 2/3 VCC

Use vertical/time cursors to measure:

- Period
- High time
- Low time
- Delay between signals

For example:

```text
Cursor 1 → rising output edge
Cursor 2 → next rising output edge

ΔT ≈ one period
```

For Timer A:

```text
ΔT ≈ 1 second
```

For Timer B:

```text
ΔT ≈ 0.44 seconds
```

---

# 22. Using persistence

A fun DHO814 experiment is to enable waveform persistence.

With the output connected:

```text
CH1 → pin 5
```

observe the stability of the waveform.

Then connect to the timing capacitor:

```text
CH1 → pins 2/6
```

Persistence can make the small variation in the capacitor waveform easier to see.

This is also useful for observing noise and cycle-to-cycle variation.

---

# 23. Using single acquisition

The DHO814 can be used to capture a single event.

For example:

1. Connect CH1 to Timer A output.
2. Configure a rising-edge trigger.
3. Arm single acquisition.
4. Power-cycle or reset the circuit.
5. Capture the first few transitions.

This is useful for investigating startup behaviour.

You can compare:

```text
Power-up
   ↓
Timing capacitor begins charging
   ↓
555 threshold reached
   ↓
Output changes state
   ↓
Oscillation begins
```

---

# 24. Experiment with component values

Once the basic circuit works, turn the DHO814 into a timing laboratory.

Change only one component.

For example, Timer A RB:

```text
68 kΩ → 47 kΩ
68 kΩ → 100 kΩ
68 kΩ → 150 kΩ
```

Measure the frequency after each change.

Create a table:

| RB | Calculated f | Measured f |
|---:|---:|---:|
| 47 kΩ | | |
| 68 kΩ | | |
| 100 kΩ | | |
| 150 kΩ | | |

You should see frequency decrease as RB increases.

---

# 25. Experiment with capacitor values

Try:

```text
10 µF
4.7 µF
2.2 µF
1 µF
```

Keep the resistors unchanged.

The expected relationship is approximately:

```text
Higher C → lower frequency
Lower C  → higher frequency
```

Measure the actual frequency using the DHO814.

---

# 26. A particularly good experiment

Try replacing the 10 µF capacitor with **1 µF**.

The frequency will increase by approximately a factor of ten.

This is a great demonstration of the inverse relationship between capacitance and oscillator frequency.

You can then observe:

```text
10 µF → slow waveform
1 µF  → much faster waveform
```

and watch the waveform transition from something you can easily see with your eyes to something that is much better observed on the oscilloscope.

---

# 27. Troubleshooting with the DHO814

## LED doesn't flash

Check:

```text
Pin 14 = +5 V
Pin 7 = GND
RESET A/B = +5 V
```

Then probe the relevant output.

If pin 5 or 9 is stuck:

1. Check the timing network.
2. Check the electrolytic capacitor polarity.
3. Check the trigger/threshold connections.
4. Check the discharge resistor connection.

---

## Output stuck HIGH

Check:

```text
Timing capacitor
RB resistor
Trigger pin
Discharge pin
```

The timing capacitor may not be reaching the threshold voltage.

---

## Output stuck LOW

Check:

```text
RESET pin
Timing capacitor
RA/RB connections
Power supply
```

---

## Waveform looks noisy

Check:

- 100 nF supply decoupling
- Breadboard connections
- Probe ground connection
- Probe attenuation
- Power supply quality

Keep the 100 nF capacitor physically close to the NE556.

---

# 28. Suggested DHO814 screenshots to capture

For a nice project record, capture these five screens:

### Screenshot 1 — Both outputs

```text
CH1 → pin 5
CH2 → pin 9
```

Shows the two different frequencies.

### Screenshot 2 — Timer A output

```text
CH1 → pin 5
```

Include frequency and period measurements.

### Screenshot 3 — Timer A capacitor

```text
CH1 → pins 2/6
```

Show the charge/discharge waveform.

### Screenshot 4 — Output + capacitor

```text
CH1 → pin 5
CH2 → pins 2/6
```

Shows the relationship between the two.

### Screenshot 5 — Timer B

```text
CH1 → pin 9
CH2 → pins 12/8
```

Shows the faster oscillator and its timing waveform.

---

# 29. Final recommended DHO814 setup

For the first session, use:

```text
Probe:
    10X

CH1:
    Timer A output
    Pin 5

CH2:
    Timer B output
    Pin 9

Vertical:
    2 V/div initially

Horizontal:
    200 ms/div

Trigger:
    Edge
    CH1
    Rising
    ~2.5 V

Measurements:
    Frequency
    Period
    Vpp
    Duty cycle
```

After confirming both outputs, move CH2 to the Timer A timing capacitor and explore the relationship between the square wave and capacitor waveform.

---

# 30. Learning objectives

By the end of the experiment you should be able to explain:

- What an astable 555 timer does
- How an RC network determines oscillator frequency
- Why the timing capacitor repeatedly charges and discharges
- How the 555 threshold and trigger levels control the oscillator
- Why the output is not exactly 50% duty cycle
- How resistor and capacitor values affect frequency
- How to measure frequency and period with a DHO814
- How to use oscilloscope triggering
- How to use automatic measurements
- How to use cursors
- How to compare two independent signals
- How to investigate an analog timing waveform alongside a digital output

---

# 31. The key experiment

If only one experiment is performed, make it this:

```text
DHO814 CH1 → NE556 pin 5
DHO814 CH2 → NE556 pins 2/6
Both grounds → GND
```

Set:

```text
Time/div ≈ 200 ms
CH1 ≈ 2 V/div
CH2 ≈ 1 V/div
Trigger = CH1
```

Then observe:

```text
              TIMER A OUTPUT
5 V     ┌──────────┐          ┌──────────
        │          │          │
0 V ────┘          └──────────┘


             TIMING CAPACITOR
3.3 V       /‾‾‾‾‾\
           /       \
1.7 V ____/         \________
```

The DHO814 lets you see the fundamental mechanism of the 555 timer directly: **the capacitor voltage ramps between the timer's threshold levels, and those threshold events control the output state.**

That makes this small two-LED project a surprisingly good introduction to practical oscilloscope work.
