\
============================================================
      ANALOGUE FIRING & SINGLE‑PHASE RECTIFIER – LTspice
============================================================

Project type  : Power Electronics / Analog Control (LTspice)
Simulator     : LTspice XVII
Files         : 
  • project 1 - analogue firing circuit.asc   (schematic)
  • project 1 - analogue firing circuit.raw   (waveforms)
  • project 1 - analogue firing circuit.log   (sim log)
  • Screenshots / sketches of the circuit idea

This repository contains an LTspice implementation of an
**analogue firing (phase‑angle) control circuit** for a
**single‑phase controlled rectifier**. The project focuses on
generating *isolated gate pulses* synchronized with the mains
(50 Hz) and demonstrates the complete signal chain:
ZERO‑CROSS → PHASE SHIFT → PULSE → ISOLATION → GATE.

It also includes an AC/transformer front‑end and a rectifier/
filter example so you can visualise how the firing angle
affects the output waveforms.

------------------------------------------------------------
1) WHAT YOU’LL SEE (AT A GLANCE)
------------------------------------------------------------
• **Sine input (220 Vrms, 50 Hz)** → transformer → low‑voltage
  control rails.
• **Zero‑cross detector** using a precision op‑amp (AD549).
• **Phase‑shift network** (RC + comparator) that turns a knob
  (threshold) into an adjustable **firing angle α**.
• **Pulse shaper/driver** (BJT stage) that drives a small
  **pulse transformer** for galvanic isolation to the gate.
• **Rectifier & load** to show the effect of α on Vout.

ASCII block sketch:
  ┌────────────┐   ┌──────────────┐   ┌───────────┐   ┌─────────┐
  │  220 Vac   │──▶│  Transformer │──▶│  ZCD +    │──▶│  Phase  │
  └────────────┘   └──────┬───────┘   │  Comparator│   │  Shift  │
                           │           └───────────┘   └────┬────┘
                           │                                │
                           ▼                                ▼
                     ±Vcc rails for op‑amps           Gate Pulse (α)
                           │                                │
                           ▼                                ▼
                        ┌───────────────────────────────────────────┐
                        │ Pulse Driver (BJT) + Pulse Transformer    │
                        └──────────────┬────────────────────────────┘
                                       │ (isolated)
                                       ▼
                                    ┌─────┐
                                    │SCR/T│───► Rectifier/Load
                                    └─────┘

------------------------------------------------------------
2) SCHEMATIC TOUR (what’s in the .ASC)
------------------------------------------------------------
Transformer & Mains (labels as in .asc):
  • V6  = SINE(0 220 50) → 220 V, 50 Hz mains source.
  • L1 = 48.4 kH (primary), L2 = 144 mH, L3 = 144 mH.
    Couplings: K1 L1 L2 1, K2 L1 L3 1, K3 L2 L3 1
    ⇒ Turns ratio ≈ sqrt(48400/144) ≈ 18.3:1 → **~12‑0‑12 Vac**.
  • D1–D4 + C2/C3/C4/C5/C6/C7: rectification & small filters to
    derive **±Vcc** for the analogue stages.

Analogue Control (left/middle of schematic):
  • U1/U2/U3 = **AD549** (precision op‑amp) used as comparators.
  • Zero‑Cross Detect (ZCD): U1 monitors the secondary sine
    (V3: SINE(0 10 50)) and produces a clean square wave at each
    zero crossing.
  • Phase Shift: R1/C1 with **Q1 (NPN)** create a ramp/edge‑
    shaping node that is compared in U2 to a DC setpoint (pot).
  • Threshold / Setpoint: adjust the reference (via V5 = 5 V and
    R network) to change the phase delay ⇒ **firing angle α**.

Gate Drive & Isolation (right):
  • Q2 (NPN) + R6 (68 Ω) drive **L4/L5** (a 1:1 pulse transformer)
    to generate a narrow, isolated gate pulse.
  • D5/D6/C8/R7 shape and clamp the driver to avoid over/undershoot.
  • D7 symbolised LED/optocoupler/indicator for “gate fired”.

Output Stage (demo):
  • A rectifier/load branch is included to **visualise** the effect
    of firing angle on the output waveform (scope trace **V(n008)**).

Simulation Control:
  • .tran 0.5   → **500 ms** (25 cycles at 50 Hz) transient run.
  • The .lib placeholder is present → *see Troubleshooting*.

------------------------------------------------------------
3) HOW THE ANALOGUE FIRING WORKS
------------------------------------------------------------
A) Zero‑Cross Detect (ZCD)
   The secondary sine is squared by U1 so every time the sine
   crosses 0 V we get a precise time origin t0 (α = 0°).

B) Phase‑Angle Generation
   After each zero cross, the RC network charges toward a set
   level. When the ramp exceeds the **threshold** (set by a pot/
   reference), U2 toggles and produces a narrow high pulse.
   The time from the zero cross to the pulse defines **α**.

C) Isolation & Drive
   The pulse drives Q2 which dumps a short current into L4,
   inducing a clean, isolated pulse on L5 (to the gate).

D) Controlled Rectification
   If the gate (SCR/Triac) is triggered at angle α each half‑cycle,
   the rectifier conducts for (π−α) radians. With a resistive load,
   the average output is theoretically:
      **Vdc = (Vm/π) · (1 + cos α)**  (full‑wave, ideal).
   In the plots you will see the “chopped” sine depending on α.

------------------------------------------------------------
4) RUNNING THE SIMULATION
------------------------------------------------------------
1. Open **project 1 - analogue firing circuit.asc** in LTspice XVII.
2. Hit the **Run** (running man) → `.tran 0.5` will execute.
3. Plot signals:
   • Add trace: **V(n008)** → gate/rectified waveform (green pulses
     in the screenshot).
   • Add trace: the sine reference node (blue) to see timing.
   • You can probe at the comparator outputs (U1/U2/U3) to follow
     the chain: ZCD → phase ramp → pulse.

Tips:
  • Right‑click the axes to **add cursors**. Measure the delay
    between the sine zero cross and the firing pulse to compute α:
       α(°) = 360° · (Δt / 20 ms)   (for 50 Hz).
  • Use **View → FFT** on the output node to examine harmonics.

------------------------------------------------------------
5) TUNING KNOBS (how to change α and pulse width)
------------------------------------------------------------
• **Threshold / Setpoint**: modify the U2 non‑inverting input
  reference (the divider around V5 = 5 V) — this directly
  changes α from early (small α) to late (large α).
• **Ramp Speed**: change **R1** and **C1 (10 nF)** to alter the
  ramp slope per half‑cycle → coarser/finer control range.
• **Pulse Width**: add a small RC at U2 output or adjust Q2 base
  drive (R6) to tailor the energy delivered to the gate.
• **Isolation**: **L4/L5** are a pulse transformer; set the ratio
  and magnetising inductance to model your real part.

------------------------------------------------------------
6) WHAT TO EXPECT (waveforms)
------------------------------------------------------------
(ASCII sketch of typical traces)

Sine (blue):     ~~~~~~~~^^^^^^^^~~~~~~~vvvvvvvv~~~~~~~^^^^^^^^~~~~
Gate (green):    ____|‾|___________|‾|___________|‾|_____________|‾|

• As you increase α, the green pulse moves to the right within
  each half‑cycle → the rectified output starts later, reducing
  the average/ RMS delivered to the load.
• With α ≈ 0° the output is near full conduction; with α → 180°,
  there’s almost no output (except narrow spikes).

------------------------------------------------------------
7) FILES & NODES (quick reference)
------------------------------------------------------------
Nodes of interest (as delivered):
  • **V(n008)**  – main plotted rectified/gate waveform.
  • Secondary sine for ZCD – probe the input of **U1**.
  • Comparator outputs – **U1/U2/U3** outputs for debugging.
  • Driver node around **Q2** / **L4‑L5** to see isolation pulse.

Key components (values pulled from .ASC):
  • R1 = 1 kΩ,  C1 = 10 nF   (phase ramp)
  • R2 = 1 kΩ (bias for Q1), Q1/Q2 = NPN
  • R6 = 68 Ω (driver series), R7 = 1 kΩ (clamp/bleed)
  • D5/D6 = diodes in driver clamp, C8 = 1 µF
  • Power: **V6** = 220 Vac, **V3** = 10 Vac (sense), **V5** = 5 V ref
  • Coupled L: **L1=48.4 kH**, **L2=L3=144 mH** -> ≈12‑0‑12 Vac
  • Pulse transformer: **L4/L5 = 1 mH** each, K = 1.

------------------------------------------------------------
8) TROUBLESHOOTING & LOG NOTES
------------------------------------------------------------
Your .log shows:
  • `Error: .include filename missing` → a stray “.lib”/“.include”
    without a file path. **Fix:** remove it or point to the actual
    library file (e.g. `.include AD549.sub`).
  • “Node … is floating / Less than two connections” warnings for
    NC_01/NC_02/NC_03 and V1. **Fix:** ensure each symbol pin is
    either wired or terminated with a component / 0V reference.
  • If the **AD549** model is unavailable, you can temporarily
    replace it with LTspice’s **opamp2** (idealized) for functional
    testing.

General stability tips:
  • Keep op‑amp supplies within model limits (±12 V typical).
  • Add small hysteresis (e.g., 100 kΩ feedback) to comparators to
    prevent chatter around zero cross in noisy sims.
  • Limit driver current with R6 and clamp with D5/D6 as already
    drawn to avoid unrealistic saturation.

------------------------------------------------------------
9) SAFETY (when moving to hardware)
------------------------------------------------------------
⚠️ **Mains (220 Vac) is lethal.** Use an **isolation transformer**,
proper fuses, MOV/TVS surge parts, and creepage/clearance rules.
Gate isolation (pulse transformer or optotriac/SCR driver) is
**mandatory**. Verify thermal ratings and use snubbers across the
power device for inductive loads.

------------------------------------------------------------
10) EXTENSIONS / IDEAS
------------------------------------------------------------
• Replace the analogue phase generator with a **digital** one
  (MCU/FPGA) and compare jitter/accuracy.
• Add a **PI controller** to regulate output voltage vs. α.
• Move to a **3‑phase fully‑controlled bridge** and reuse the
  same timing core (three phase‑shifted ZCDs).
• Run **parametric sweeps** of α and plot RMS/THD vs. α.
• Build a **Simulink** model that mirrors this LTspice version
  for cross‑validation / control co‑design.

------------------------------------------------------------
11) HOW TO PACKAGE THIS REPO
------------------------------------------------------------
Suggested structure:
  /hardware-docs   – hand sketches, calculations
  /ltspice         – .asc, .raw, .log, models (.sub)
  /results         – screenshots, .csv exports, FFTs
  README.md        – a web‑friendly version (optional)

Repo one‑liner (GitHub description):
  “Single‑phase analogue firing circuit (phase‑angle control) in
   LTspice with isolated gate drive and rectifier demo.”

============================================================
END – Analogue Firing & Rectifier (LTspice)
============================================================
