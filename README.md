# NE556N Dual LED Timer — Complete Specification & Implementation Plan

## 1. Project overview

Build a simple, fun two-LED timer/flasher on a solderless breadboard using a single **NE556N dual 555 timer**.

The circuit runs from a regulated **5 V DC supply** and uses **fixed resistors only** — no potentiometer or variable resistor.

The two halves of the NE556N operate independently:

- **Timer A:** approximately 1 Hz — slow LED flash
- **Timer B:** approximately 2 Hz — faster LED flash
- Both outputs are suitable for observing on a Rigol oscilloscope.

### Project goals

1. Learn the basic astable/free-running 555 timer circuit.
2. Learn how a dual timer can implement two independent oscillators.
3. Build the circuit on a breadboard.
4. Observe the square-wave outputs on an oscilloscope.
5. Experiment with resistor and capacitor values later to change the timing.

---

## 2. Electrical specification

| Parameter | Specification |
|---|---|
| Timer IC | NE556N |
| Timer sections | 2 independent 555 timers |
| Supply voltage | Regulated 5 V DC |
| Timer A frequency | ~1 Hz |
| Timer B frequency | ~2 Hz |
| LED current limiting | 330 Ω per LED |
| Timing capacitors | 10 µF electrolytic |
| Control-pin capacitors | 10 nF |
| Supply decoupling | 100 nF |
| Variable resistor | None |
| Construction | Solderless breadboard |

The NE556N is a dual 555 timer. Each half is configured as an astable oscillator.

---

## 3. Bill of materials

### Integrated circuit

| Qty | Part | Notes |
|---:|---|---|
| 1 | NE556N | 14-pin DIP dual timer |

### LEDs

| Qty | Part | Notes |
|---:|---|---|
| 1 | LED1 | Red recommended |
| 1 | LED2 | Green or blue recommended |

### Resistors

| Qty | Value | Function |
|---:|---:|---|
| 2 | 10 kΩ | RA for each timer |
| 1 | 68 kΩ | RB for Timer A |
| 1 | 33 kΩ | RB for Timer B |
| 2 | 330 Ω | LED current limiting |

### Capacitors

| Qty | Value | Function |
|---:|---:|---|
| 2 | 10 µF, ≥6.3 V | Timing capacitors |
| 2 | 10 nF ceramic | Control-voltage filtering |
| 1 | 100 nF ceramic | Supply decoupling |

### Other

- 1 × solderless breadboard
- Jumper wires
- Regulated 5 V USB or bench supply
- Oscilloscope and probes
- Optional: multimeter

---

## 4. NE556N pinout

With the IC viewed from above and the notch at the top:

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

Important:

- Pin 14 = +5 V
- Pin 7 = GND
- Timer A output = pin 5
- Timer B output = pin 9

---

## 5. Timer A — approximately 1 Hz

Timer A uses the standard 555 astable configuration.

### Timing components

- RA = 10 kΩ
- RB = 68 kΩ
- C = 10 µF

Approximate frequency:

```text
f ≈ 1.44 / ((RA + 2 × RB) × C)
```

Using the selected values:

```text
RA = 10,000 Ω
RB = 68,000 Ω
C  = 10 µF

f ≈ 1.44 / ((10,000 + 136,000) × 0.000010)

f ≈ 0.986 Hz
```

So the LED will flash approximately once per second.

### Timer A connections

| NE556 pin | Connection |
|---:|---|
| 1 DISCH A | Junction between 10 kΩ and 68 kΩ |
| 2 THRES A | Timing capacitor junction |
| 3 CONT A | 10 nF to GND |
| 4 RESET A | +5 V |
| 5 OUT A | 330 Ω → LED1 → GND |
| 6 TRIG A | Timing capacitor junction |
| 7 GND | GND |
| 14 VCC | +5 V |

Timing network:

```text
+5 V
  |
  10 kΩ
  |
  +------ pin 1 DISCH A
  |
  68 kΩ
  |
  +------ pins 2 & 6
  |
  10 µF
  |
 GND
```

LED:

```text
pin 5 OUT A
     |
    330 Ω
     |
   LED1
     |
    GND
```

---

## 6. Timer B — approximately 2 Hz

Timer B uses the same configuration but with a smaller RB value.

### Timing components

- RA = 10 kΩ
- RB = 33 kΩ
- C = 10 µF

Approximate frequency:

```text
f ≈ 1.44 / ((RA + 2 × RB) × C)
```

Using the selected values:

```text
RA = 10,000 Ω
RB = 33,000 Ω
C  = 10 µF

f ≈ 1.44 / ((10,000 + 66,000) × 0.000010)

f ≈ 1.34 Hz
```

### Important correction

The nominal 10 kΩ / 33 kΩ / 10 µF combination produces approximately **1.34 Hz**, not 2 Hz.

Therefore, for a genuine ~2 Hz Timer B, use:

- RA = 10 kΩ
- RB = **15 kΩ**
- C = 10 µF

This gives:

```text
f ≈ 1.44 / ((10,000 + 30,000) × 0.000010)
  ≈ 3.6 Hz
```

That is still not 2 Hz.

A better fixed-resistor choice is:

- RA = 10 kΩ
- RB = **28 kΩ**
- C = 10 µF

giving approximately:

```text
f ≈ 1.44 / ((10,000 + 56,000) × 0.000010)
  ≈ 2.18 Hz
```

### Recommended practical value

Use a standard **27 kΩ** resistor for RB:

```text
f ≈ 1.44 / ((10,000 + 54,000) × 0.000010)
  ≈ 2.25 Hz
```

This gives a clearly faster second LED and uses an easily available standard resistor.

### Timer B connections

| NE556 pin | Connection |
|---:|---|
| 13 DISCH B | Junction between 10 kΩ and 27 kΩ |
| 12 THRES B | Timing capacitor junction |
| 11 CONT B | 10 nF to GND |
| 10 RESET B | +5 V |
| 9 OUT B | 330 Ω → LED2 → GND |
| 8 TRIG B | Timing capacitor junction |

Timing network:

```text
+5 V
  |
  10 kΩ
  |
  +------ pin 13 DISCH B
  |
  27 kΩ
  |
  +------ pins 12 & 8
  |
  10 µF
  |
 GND
```

LED:

```text
pin 9 OUT B
     |
    330 Ω
     |
   LED2
     |
    GND
```

---

## 7. Power and decoupling

Connect:

```text
+5 V ───────── pin 14 VCC
GND ────────── pin 7 GND

+5 V ───────── pin 4 RESET A
+5 V ───────── pin 10 RESET B
```

Place the **100 nF ceramic capacitor directly between pins 14 and 7**, physically close to the NE556.

This is especially important on a breadboard because the wiring has more parasitic inductance and resistance than a PCB.

---

## 8. Complete connection list

### Power

```text
Pin 14 → +5 V
Pin 7  → GND
Pin 4  → +5 V
Pin 10 → +5 V
100 nF → between +5 V and GND
```

### Timer A

```text
+5 V → 10 kΩ → pin 1
pin 1 → 68 kΩ → pins 2 + 6
pins 2 + 6 → + terminal of 10 µF
− terminal of 10 µF → GND

pin 3 → 10 nF → GND

pin 5 → 330 Ω → LED1 anode
LED1 cathode → GND
```

### Timer B

```text
+5 V → 10 kΩ → pin 13
pin 13 → 27 kΩ → pins 12 + 8
pins 12 + 8 → + terminal of 10 µF
− terminal of 10 µF → GND

pin 11 → 10 nF → GND

pin 9 → 330 Ω → LED2 anode
LED2 cathode → GND
```

---

## 9. Schematic

The circuit can be represented conceptually as two independent 555 astable circuits inside the NE556:

```text
                         +5 V
                          |
             +------------+-------------+
             |                          |
           10 kΩ                       10 kΩ
             |                          |
             +-- pin 1                 pin 13 --+
             |                                   |
           68 kΩ                                27 kΩ
             |                                   |
          pins 2,6                            pins 12,8
             |                                   |
            10 µF                               10 µF
             |                                   |
            GND                                 GND


                 ┌──────────────────────────┐
                 │          NE556N          │
                 │                          │
        GND ──── │ 7                    14 │ ─── +5 V
                 │                          │
   Timer A ───── │ 1 DISCH          DISCH 13│ ───── Timer B
                 │ 2 THRES          THRES 12│
                 │ 3 CONT            CONT 11│
                 │ 4 RESET          RESET 10│
                 │ 5 OUT              OUT  9│
                 │ 6 TRIG            TRIG  8│
                 └──────────────────────────┘
                    |                    |
                   330 Ω                330 Ω
                    |                    |
                   LED1                 LED2
                    |                    |
                   GND                  GND
```

For the actual build, use the pin-by-pin connection table above as the authoritative wiring reference.

---

## 10. Breadboard implementation plan

### Step 1 — Place the NE556N

Place the NE556N across the central breadboard gap.

Orient the notch toward the top of the breadboard.

Verify that the pins are numbered correctly before adding any wiring.

### Step 2 — Establish the power rails

Connect:

```text
Breadboard + rail → regulated +5 V
Breadboard − rail → GND
```

If the breadboard has split power rails, bridge the sections so that +5 V and GND are available along the entire working area.

### Step 3 — Connect the NE556 power

Add:

```text
Pin 14 → +5 V
Pin 7  → GND
Pin 4  → +5 V
Pin 10 → +5 V
```

Add the 100 nF capacitor close to pins 14 and 7.

### Step 4 — Build Timer A

Install:

- 10 kΩ
- 68 kΩ
- 10 µF capacitor
- 10 nF capacitor
- 330 Ω
- LED1

Double-check electrolytic capacitor polarity.

The positive terminal of the 10 µF capacitor connects to the timing-node junction.

### Step 5 — Build Timer B

Install:

- 10 kΩ
- 27 kΩ
- 10 µF capacitor
- 10 nF capacitor
- 330 Ω
- LED2

Again, verify the electrolytic capacitor polarity.

### Step 6 — Visual inspection

Before powering the circuit:

- Check that +5 V is not shorted to GND.
- Check NE556 orientation.
- Check pins 14 and 7.
- Check both RESET pins.
- Check LED polarity.
- Check electrolytic capacitor polarity.
- Check that the timing resistors are connected to the correct pins.
- Check that pins 2/6 and 12/8 are joined respectively.

### Step 7 — Power up

Apply regulated 5 V.

Both LEDs should begin flashing.

If either LED remains continuously on or off, immediately remove power and check the corresponding timer section.

---

## 11. Oscilloscope test plan

This is an excellent circuit for learning with the Rigol oscilloscope.

### Channel 1

Connect:

```text
CH1 probe tip → NE556 pin 5
CH1 ground   → circuit GND
```

Expected waveform:

- Square wave
- Approximately 1 Hz
- Approximately 0–5 V

### Channel 2

Connect:

```text
CH2 probe tip → NE556 pin 9
CH2 ground   → circuit GND
```

Expected waveform:

- Square wave
- Approximately 2.2 Hz
- Approximately 0–5 V

### Suggested initial scope settings

```text
Time/div:    200 ms/div
Voltage/div: 2 V/div
Trigger:     CH1
Trigger level: ~2.5 V
Coupling:    DC
```

You should see the slower Timer A waveform clearly while Timer B produces several transitions during the same period.

---

## 12. Interesting scope experiments

Once the circuit works, use the oscilloscope to investigate:

### Experiment 1 — Measure frequency

Use the scope's automatic frequency measurement.

Compare the measured frequency with the theoretical value.

Real components will not produce exactly the calculated frequency because resistor and capacitor tolerances affect the result.

### Experiment 2 — Measure duty cycle

The 555 astable circuit does not produce a perfect 50/50 square wave.

Measure:

- Frequency
- Period
- High time
- Low time
- Duty cycle

### Experiment 3 — Observe the timing capacitor

Move CH1 to the Timer A timing capacitor.

You should see the capacitor voltage ramp up and down between the approximate 1/3 and 2/3 supply thresholds.

This is one of the most useful demonstrations of how a 555 timer actually works.

### Experiment 4 — Compare both timers

Put:

```text
CH1 → pin 5
CH2 → pin 9
```

and observe the two oscillators simultaneously.

### Experiment 5 — Change the blink rate

Because there is no potentiometer, simply swap the fixed RB resistor.

For example:

```text
27 kΩ → faster
33 kΩ → slower
47 kΩ → slower again
68 kΩ → much slower
```

You can keep a small collection of resistors and turn this into a mini timing laboratory.

---

## 13. Safety and build notes

### Supply voltage

Use a **regulated 5 V supply**.

Do not connect an unknown or unregulated voltage source directly to the circuit.

### LED polarity

The LED's:

- Longer lead = anode (+)
- Shorter lead = cathode (−)

The cathode normally has a flat edge on the LED body.

### Electrolytic capacitor polarity

The 10 µF capacitors are polarized.

The positive terminal connects to the timing node.

The negative terminal connects to GND.

### LED resistor

Never connect an LED directly to the NE556 output.

The 330 Ω resistor limits current.

---

## 14. Expected results

After successful construction:

```text
Power ON

LED1:  ON ... OFF ... ON ... OFF ...
       ~1 flash/second

LED2:  ON . OFF . ON . OFF . ON ...
       ~2.2 flashes/second
```

The exact frequencies will vary somewhat with component tolerances.

The oscilloscope should show clean digital-looking waveforms at the two output pins.

---

## 15. Suggested project extensions

Once the basic circuit works, several fun extensions are possible.

### Extension A — Make them alternate

Connect the output of Timer A into the trigger/reset arrangement of Timer B so the LEDs interact rather than simply running independently.

### Extension B — Add a third LED

Use logic/transistor circuitry to derive another visual indication from the two timer outputs.

### Extension C — Make a "heartbeat" pattern

Choose resistor and capacitor values that create a short-short-long repeating pattern.

### Extension D — Add a push button

Use a push button to reset one timer and create a manually triggered effect.

### Extension E — Investigate the waveform

Use the Rigol to capture:

1. Output square wave
2. Timing capacitor ramp
3. Relationship between threshold voltage and output transition

This turns the project from simply an LED flasher into a useful **555 timer laboratory experiment**.

---

## 16. Final recommended configuration

For the first build, use:

```text
Supply:          5 V

NE556N:
  Timer A:
    RA = 10 kΩ
    RB = 68 kΩ
    C  = 10 µF
    LED = red
    f ≈ 0.99 Hz

  Timer B:
    RA = 10 kΩ
    RB = 27 kΩ
    C  = 10 µF
    LED = green
    f ≈ 2.25 Hz

LED resistors:
    330 Ω each

Control capacitors:
    10 nF each

Supply bypass:
    100 nF
```

This gives a nice visible difference between the two LEDs while keeping the circuit simple and using only readily available fixed-value components.

---

## 17. Build checklist

- [ ] NE556N correctly oriented
- [ ] Pin 14 → +5 V
- [ ] Pin 7 → GND
- [ ] Pin 4 → +5 V
- [ ] Pin 10 → +5 V
- [ ] 100 nF across supply
- [ ] Timer A timing network installed
- [ ] Timer B timing network installed
- [ ] Both 10 µF capacitors correctly polarized
- [ ] Both LEDs correctly oriented
- [ ] 330 Ω resistor in series with each LED
- [ ] 5 V supply verified before connection
- [ ] LED1 flashing at roughly 1 Hz
- [ ] LED2 flashing at roughly 2 Hz
- [ ] Rigol CH1 connected to pin 5
- [ ] Rigol CH2 connected to pin 9
- [ ] Timing capacitor waveform observed

## 18. Project outcome

The finished project demonstrates:

- Dual 555 timer operation
- Astable oscillators
- RC timing
- Capacitor charging/discharging
- Threshold and trigger operation
- Digital output waveforms
- LED current limiting
- Breadboard construction
- Oscilloscope measurement

It is a compact project that can be built in well under an hour and provides considerably more to explore than a simple single-LED flasher.
