# Power Converter

This repository contains **LTSpice** simulations for two fundamental DC–DC switching power converter topologies: a **synchronous buck converter** (step-down) and a **synchronous boost converter** (step-up). The simulations model closed-loop control with a **trailing-edge pulse-width modulator (PWM)**, **gate drivers** with bootstrap supply, and **dead-time generation** for synchronous rectification.

---

## Buck Converter (`buck-converter/`)

| Parameter         | Value            |
|-------------------|------------------|
| Input voltage     | 12 V             |
| Topology          | Synchronous buck |
| Duty cycle        | 0.104            |
| Inductor (L1)     | 4 µH / 2 mΩ DCR |
| Output cap (C1)   | 680 µF           |
| Load resistance   | 0.05 Ω           |
| MOSFETs           | IRFZ44N (HS), IRFH5004 (LS) |
| Freewheeling diode| MBR735 Schottky  |
| PWM frequency     | 100 kHz          |
| Dead time         | 100 ns           |
| Simulation        | 2 ms transient, startup |

### How It Works

The **synchronous buck converter** steps down the 12 V input to a lower output voltage. A PWM modulator generates a switching signal at the duty cycle set by `Vduty` (0.104). This signal passes through a **dead-time generator**, which produces two complementary gate-drive signals separated by a 100 ns non-overlap interval. The high-side MOSFET (M1, IRFZ44N) and low-side synchronous MOSFET (M2, IRFH5004) switch the inductor node between Vin and ground. During the high-side on-time, current ramps up through L1, storing energy. During the off-time, the low-side MOSFET provides a low-resistance path for inductor current, acting as a synchronous rectifier. The output capacitor (C1) filters the switching ripple, delivering a regulated low-voltage, high-current output to the 0.05 Ω load.

Two **gate driver** instances (U1, U3) provide the peak current needed to charge and discharge the MOSFET gate capacitance rapidly, minimising switching losses. The low-side driver U1 drives M1; driver U3 drives M2. Schottky diodes D2 (1N5817) and D4 (MBR735) clamp inductive spikes and bootstrap capacitor C2 (10 µF) supplies the high-side driver's floating supply rail. A second Schottky diode D3 (MBR735) serves as the freewheeling path during dead times.

---

## Boost Converter (`boost-converter/`)

| Parameter         | Value            |
|-------------------|------------------|
| Input voltage     | 12 V             |
| Topology          | Synchronous boost|
| Duty cycle        | 0.5              |
| Inductor (L1)     | 60 µH / 2 mΩ DCR|
| Input cap (C2)    | 10 µF            |
| Output cap (C1)   | 100 µF           |
| Load resistance   | 12 Ω             |
| MOSFETs           | IRFZ44N (both)   |
| Output diode      | MBR735 Schottky  |
| PWM frequency     | 100 kHz          |
| Dead time         | 100 ns           |
| Simulation        | 10 ms transient, UIC, initial Vout = 24 V |

### How It Works

The **synchronous boost converter** steps up the 12 V input to a regulated 24 V output (preset via `.ic V(Vout)=24`). The PWM modulator operates at a 0.5 duty ratio, producing a train of pulses that pass through the dead-time generator to create complementary gate signals with a 100 ns non-overlap guard band.

During the switch on-time, MOSFET M2 (the main switch) conducts, shorting the output side of L1 to ground and causing inductor current to ramp up, storing magnetic energy. Diode D3 (MBR735) reverse-biases, isolating the output. During the off-time, M2 turns off and the high-side synchronous MOSFET M1 turns on, connecting the inductor to the output. The inductor's stored energy forces current through M1 and D3 into the output capacitor C1 and the 12 Ω load, producing a voltage higher than the input.

Two gate driver stages provide robust switching: U1 (low-side) drives M2 with 6 A peak current capability, while U3 (high-side) drives M1 and is powered by a bootstrap supply formed by diode D1 and capacitor C2. Schottky diodes D1, D2, and D3 (all MBR735) handle bootstrap charging, gate clamping, and output rectification respectively.

---

## Shared Subcircuit Blocks

All three `.asy` files are shared between both converter directories:

| Symbol             | Description |
|--------------------|-------------|
| `PWM.asy`          | Trailing-edge pulse-width modulator. Compares a control voltage (`vc`) against a sawtooth ramp (100 kHz, 1 V amplitude) and outputs a duty-cycle-modulated logic-level signal (`c`). Configurable min/max duty limits and offset. |
| `Driver.asy`        | Gate driver with 3 A (or 6 A) peak output current, 30 ns propagation delay, 1 mA quiescent current, and undervoltage lockout (UVLO) at 9 V. |
| `dead_time.asy`     | Dead-time generator. Accepts a single PWM input (`c`) and produces two complementary outputs (`c1`, `c2`) separated by a programmable non-overlap interval `Td` (default 100 ns). |

The `switching.lib` file contains the SPICE subcircuit models for these blocks (PWM ramp generator, driver, dead-time logic).

---

## Simulation Directives

**Buck converter:**
```
.tran 0 2msec 0 20e-9 startup uic
.ic V(Vout)=0
.options reltol=0.0005
```

**Boost converter:**
```
.tran 0 10msec 0 20e-9 UIC
.ic V(Vout)=24
.options reltol=0.0005
```

Measurement directives in the buck converter compute the reverse-recovery charge (`Qrr`) of the low-side MOSFET body diode and the switching energy (`Eevent`) during a specific switching event:
```
.meas tran Qrr INTEGRAL I(M2) FROM=1u TO=1.05u
.meas TRAN Eevent INTEGRAL V(n006)*Is(M2) FROM=549.9u TO=550.1u
```

---

## Getting Started

1. Install [LTSpice](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html) (free).
2. Open the `.asc` schematic file in either `buck-converter/` or `boost-converter/`.
3. Run the simulation (Simulate > Run).
4. Probe the nodes of interest: `Vout`, `Vsw` (switch node), inductor current, MOSFET gate signals.

---

## License

This project is provided for educational and reference purposes.
