# Power Converter

This repository contains **LTSpice** simulations for four DC–DC switching power converter topologies: a **buck converter** (step-down) and a **boost converter** (step-up), each in both **synchronous** and **non-synchronous** (diode-rectified) variants.

---

## Non-Synchronous Converters

These topologies use a Schottky diode for the rectification path, eliminating the need for a second MOSFET, gate driver, and dead-time logic.

### Buck Converter (`buck-converter/`)

| Parameter         | Value            |
|-------------------|------------------|
| Input voltage     | 12 V             |
| Topology          | Non-synchronous buck |
| Duty cycle        | 0.104            |
| Inductor (L1)     | 4 µH / 2 mΩ DCR |
| Output cap (C1)   | 680 µF           |
| Load resistance   | 0.05 Ω           |
| MOSFET            | IRFZ44N          |
| Freewheeling diode| MBR735 Schottky  |
| PWM frequency     | 100 kHz          |
| Simulation        | 2 ms transient, startup |

### Boost Converter (`boost-converter/`)

| Parameter         | Value            |
|-------------------|------------------|
| Input voltage     | 12 V             |
| Topology          | Non-synchronous boost |
| Duty cycle        | 0.5              |
| Inductor (L1)     | 60 µH / 2 mΩ DCR|
| Input cap         | 10 µF            |
| Output cap (C1)   | 100 µF           |
| Load resistance   | 12 Ω             |
| MOSFET            | IRFZ44N          |
| Output diode      | MBR735 Schottky  |
| PWM frequency     | 100 kHz          |
| Simulation        | 10 ms transient, UIC, initial Vout = 24 V |

---

## Synchronous Converters

These topologies replace the freewheeling diode with a low-side MOSFET (synchronous rectifier) to reduce conduction losses, requiring dead-time generation and an additional gate driver.

### Synchronous Buck Converter (`sync-buck-converter/`)

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

#### How It Works

The high-side MOSFET (M1, IRFZ44N) and low-side synchronous MOSFET (M2, IRFH5004) switch the inductor node between Vin and ground. A dead-time generator produces complementary gate-drive signals separated by a 100 ns non-overlap interval. During the high-side on-time, current ramps up through L1, storing energy. During the off-time, the low-side MOSFET provides a low-resistance path for inductor current, acting as a synchronous rectifier. The output capacitor (C1) filters the switching ripple, delivering a regulated low-voltage, high-current output to the 0.05 Ω load.

Two gate driver instances (U1, U3) provide the peak current needed to charge and discharge the MOSFET gate capacitance rapidly. The low-side driver U1 drives M1; driver U3 drives M2. Schottky diodes D2 (1N5817) and D4 (MBR735) clamp inductive spikes and bootstrap capacitor C2 (10 µF) supplies the high-side driver's floating supply rail. A second Schottky diode D3 (MBR735) serves as the freewheeling path during dead times.

### Synchronous Boost Converter (`sync-boost-converter/`)

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

#### How It Works

During the switch on-time, MOSFET M2 (the main switch) conducts, shorting the output side of L1 to ground and causing inductor current to ramp up, storing magnetic energy. Diode D3 (MBR735) reverse-biases, isolating the output. During the off-time, M2 turns off and the high-side synchronous MOSFET M1 turns on, connecting the inductor to the output. The inductor's stored energy forces current through M1 and D3 into the output capacitor C1 and the 12 Ω load, producing a voltage higher than the input.

Two gate driver stages provide robust switching: U1 (low-side) drives M2 with 6 A peak current capability, while U3 (high-side) drives M1 and is powered by a bootstrap supply formed by diode D1 and capacitor C2. Schottky diodes D1, D2, and D3 (all MBR735) handle bootstrap charging, gate clamping, and output rectification respectively.

---

## Shared Subcircuit Blocks

The `.asy` symbols and `switching.lib` are available in each converter directory:

| Symbol             | Description |
|--------------------|-------------|
| `PWM.asy`          | Trailing-edge pulse-width modulator. Compares a control voltage (`vc`) against a sawtooth ramp (100 kHz, 1 V amplitude) and outputs a duty-cycle-modulated logic-level signal (`c`). Configurable min/max duty limits and offset. |
| `Driver.asy`        | Gate driver with 3 A (or 6 A) peak output current, 30 ns propagation delay, 1 mA quiescent current, and undervoltage lockout (UVLO) at 9 V. |
| `dead_time.asy`     | Dead-time generator (synchronous variants only). Accepts a single PWM input (`c`) and produces two complementary outputs (`c1`, `c2`) separated by a programmable non-overlap interval `Td` (default 100 ns). |

The `switching.lib` file contains the SPICE subcircuit models for these blocks (PWM ramp generator, driver, dead-time logic).

---

## Simulation Directives

**Buck converter (non-synchronous):**
```
.tran 0 2msec 0 20e-9 startup uic
.ic V(Vout)=0
.options reltol=0.0005
```

**Synchronous buck converter:**
```
.tran 0 2msec 0 20e-9 startup uic
.ic V(Vout)=0
.options reltol=0.0005
```

**Boost converter (non-synchronous):**
```
.tran 0 10msec 0 20e-9 UIC
.ic V(Vout)=24
.options reltol=0.0005
```

**Synchronous boost converter:**
```
.tran 0 10msec 0 20e-9 UIC
.ic V(Vout)=24
.options reltol=0.0005
```

Measurement directives in the synchronous buck converter compute the reverse-recovery charge (`Qrr`) of the low-side MOSFET body diode and the switching energy (`Eevent`) during a specific switching event:
```
.meas tran Qrr INTEGRAL I(M2) FROM=1u TO=1.05u
.meas TRAN Eevent INTEGRAL V(n006)*Is(M2) FROM=549.9u TO=550.1u
```

---

## Losses Modeled

All simulations include several physically-based loss mechanisms:

### Conduction Losses

| Mechanism | How It Is Modeled |
|-----------|-------------------|
| **Inductor DCR** | Series resistor `RL = 2 mΩ` on L1 captures DC copper loss |
| **MOSFET conduction** | IRFZ44N and IRFH5004 SPICE models include channel on-resistance `Rds(on)` |
| **Diode forward drop** | MBR735 and 1N5817 Schottky models capture `VF × IF` conduction loss |

### Switching Losses

- **Hard-switching energy** — The buck converter includes a `.meas` directive that integrates instantaneous MOSFET power (`V × I`) during a switching transition to quantify turn-on/turn-off energy loss
- **Gate drive loss** — Driver subcircuits supply the peak current needed to charge/discharge MOSFET gate capacitance (`Cgs`, `Cgd`) at 100 kHz; quiescent loss is modeled via `IQ = 1 mA`
- **Reverse recovery** — The buck `.meas Qrr` directive integrates the low-side MOSFET body-diode reverse-recovery charge, a significant loss contributor in hard-switched converters

### Auxiliary Losses

- Bootstrap diode forward drops
- Gate driver quiescent and cross-conduction current

### What Is *Not* Modeled

- Magnetic core losses (hysteresis, eddy current) — inductors are ideal aside from DCR
- Capacitor ESR — all capacitors are ideal (no series resistance)
- PCB parasitics — trace resistance, inductance, and coupling are omitted
- Thermal effects — `Rds(on)` and diode `VF` are temperature-independent
- Skin/proximity effect in the inductor winding

---

## Getting Started

1. Install [LTSpice](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html) (free).
2. Open the `.asc` schematic file in the desired converter directory.
3. Run the simulation (Simulate > Run).
4. Probe the nodes of interest: `Vout`, `Vsw` (switch node), inductor current, MOSFET gate signals.

---

## License

This project is provided for educational and reference purposes.
