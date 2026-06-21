# DC-DC Power Converter Simulation Library

**LTspice reference designs for classical, synchronous, and GaN-based converter topologies**

![LTspice](https://img.shields.io/badge/LTspice-26%2B-DC143C?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-Power%20Electronics-1F6FEB?style=flat-square)
![Topologies](https://img.shields.io/badge/Topologies-10-2EA043?style=flat-square)
![Purpose](https://img.shields.io/badge/Purpose-Education%20%26%20Reference-6E7781?style=flat-square)

[Simulation Library](#simulation-library) ·
[Shared Models](#shared-subcircuit-blocks) ·
[Simulation Setup](#simulation-setup) ·
[Model Scope](#model-scope) ·
[Getting Started](#getting-started)

---

This repository provides a structured collection of DC–DC converter simulations covering buck, boost, buck-boost, and Ćuk-family topologies. Designs include diode-rectified, synchronous, and 650 V cascode GaN buck and boost implementations with reusable PWM, dead-time, and gate-driver subcircuits.

> [!IMPORTANT]
> The Nexperia GaN model requires **LTspice 26.0.1 or newer**. Other schematics can be used with earlier LTspice releases.

## Simulation Library

Select **Open schematic** to view the LTspice source or **Design notes** to jump to its technical summary.

| Architecture | Topology | Output | Schematic | Documentation |
|:-------------|:---------|:------:|:----------|:--------------|
| Diode-rectified | Buck | Positive | [Open schematic](buck-converter/buck.asc) | [Design notes](#buck-converter) |
| Diode-rectified | Boost | Positive | [Open schematic](boost-converter/boost.asc) | [Design notes](#boost-converter) |
| Diode-rectified | Buck-boost | Negative | [Open schematic](buck-boost-converter/BuckBoost.asc) | [Design notes](#buck-boost-converter) |
| Diode-rectified | Ćuk-family / SEPIC | Positive | [Open schematic](cuk-converter/Cuk.asc) | [Design notes](#cuk-sepic-converter) |
| Synchronous | Buck | Positive | [Open schematic](sync-buck-converter/Buckdiode.asc) | [Design notes](#synchronous-buck-converter) |
| Synchronous | Boost | Positive | [Open schematic](sync-boost-converter/SynchBoost.asc) | [Design notes](#synchronous-boost-converter) |
| Synchronous | Buck-boost | Negative | [Open schematic](sync-buck-boost-converter/SyncBuckBoost.asc) | [Design notes](#synchronous-buck-boost-converter) |
| Synchronous | Ćuk-family / SEPIC | Positive | [Open schematic](sync-cuk-converter/SyncCuk.asc) | [Design notes](#synchronous-cuk-sepic-converter) |
| Cascode GaN | Synchronous buck | Positive | [Open schematic](gan-sync-buck-converter/GanSyncBuckdiode.asc) | [Design notes](#gan-synchronous-buck-converter) |
| Cascode GaN | Synchronous boost | Positive | [Open schematic](gan-sync-boost-converter/GanSynchBoost.asc) | [Design notes](#gan-synchronous-boost-converter) |

---

## Non-Synchronous Converters

These topologies use a Schottky diode for the rectification path, eliminating the need for a second MOSFET, gate driver, and dead-time logic.

### Buck Converter

**Schematic:** [`buck-converter/buck.asc`](buck-converter/buck.asc)

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

### Boost Converter

**Schematic:** [`boost-converter/boost.asc`](boost-converter/boost.asc)

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

### Buck-Boost Converter

**Schematic:** [`buck-boost-converter/BuckBoost.asc`](buck-boost-converter/BuckBoost.asc)

This inverting buck-boost converter uses one IRFS4010 MOSFET and an MBR735 diode. With a 12 V input and 0.4 duty cycle, the ideal target output is approximately −8 V.

| Parameter | Value |
|-----------|-------|
| Input voltage | 12 V |
| Duty cycle | 0.4 |
| Inductor | 47 µH / 100 mΩ DCR |
| Output capacitor | 47 µF |
| Load resistance | 10 Ω |
| MOSFET | IRFS4010 |
| Rectifier | MBR735 |
| Simulation | 5 ms transient, startup |

### Cuk-SEPIC Converter

**Schematic:** [`cuk-converter/Cuk.asc`](cuk-converter/Cuk.asc)

The schematic implements a non-inverting SEPIC-style member of the Ćuk converter family. It transfers energy through a coupling capacitor and uses two inductors to produce a positive output.

| Parameter | Value |
|-----------|-------|
| Input voltage | 12 V |
| Duty cycle | 0.4 |
| Inductors | L1 = L2 = 100 µH |
| Coupling capacitor | 1 µF |
| Output capacitor | 47 µF |
| Load resistance | 10 Ω |
| MOSFET | IRFS4010 |
| Rectifier | MBR735 |
| Simulation | 5 ms transient, startup |

[Back to simulation library](#simulation-library)

---

## Synchronous Converters

These topologies replace the freewheeling diode with a low-side MOSFET (synchronous rectifier) to reduce conduction losses, requiring dead-time generation and an additional gate driver.

### Synchronous Buck Converter

**Schematic:** [`sync-buck-converter/Buckdiode.asc`](sync-buck-converter/Buckdiode.asc)

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

### Synchronous Boost Converter

**Schematic:** [`sync-boost-converter/SynchBoost.asc`](sync-boost-converter/SynchBoost.asc)

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

### Synchronous Buck-Boost Converter

**Schematic:** [`sync-buck-boost-converter/SyncBuckBoost.asc`](sync-buck-boost-converter/SyncBuckBoost.asc)

This inverting synchronous buck-boost converter replaces the rectifier with a second IRFZ44N MOSFET. Complementary gate signals and 100 ns dead time reduce simultaneous conduction.

| Parameter | Value |
|-----------|-------|
| Input voltage | 12 V |
| Duty cycle | 0.4 |
| Target output | −8 V |
| Inductor | 47 µH / 100 mΩ DCR |
| Output capacitor | 47 µF |
| Load resistance | 10 Ω |
| MOSFETs | IRFZ44N |
| Simulation | 5 ms transient, startup |

### Synchronous Cuk-SEPIC Converter

**Schematic:** [`sync-cuk-converter/SyncCuk.asc`](sync-cuk-converter/SyncCuk.asc)

This non-inverting synchronous SEPIC implementation replaces the output diode with a second IRFZ44N MOSFET driven from a floating supply referenced to switching node `B`.

| Parameter | Value |
|-----------|-------|
| Input voltage | 12 V |
| Duty cycle | 0.4 |
| Target output | +8 V |
| Inductors | L1 = L2 = 100 µH |
| Coupling capacitor | 1 µF |
| Output capacitor | 47 µF |
| Load resistance | 10 Ω |
| MOSFETs | IRFZ44N |
| Simulation | 5 ms transient, startup |

### GaN Synchronous Buck Converter

**Schematic:** [`gan-sync-buck-converter/GanSyncBuckdiode.asc`](gan-sync-buck-converter/GanSyncBuckdiode.asc)

This synchronous buck design replaces both silicon MOSFETs with Nexperia `GAN039-650NxB` 650 V cascode GaN FET models. Complementary PWM signals and dead time drive the high-side and low-side devices, while 15 V Zener clamps protect each gate.

| Parameter | Value |
|-----------|-------|
| Input voltage | 12 V |
| Duty cycle | 0.4 |
| Ideal output voltage | 4.8 V |
| Inductor | 4 µH / 2 mΩ DCR |
| Output capacitor | 680 µF |
| Bootstrap capacitor | 10 µF |
| Load resistance | 0.05 Ω |
| GaN FETs | Nexperia GAN039-650NxB |
| Gate resistors | 22 Ω |
| Gate clamps | BZX84C15L, 15 V |
| PWM frequency | 100 kHz |
| Dead time | 100 ns |
| Simulation | 2 ms transient, startup |

The schematic depends on these local model and control files:

- [`GAN039-650NxB.asy`](gan-sync-buck-converter/GAN039-650NxB.asy)
- [`GAN039-650NxB_LTspice.lib`](gan-sync-buck-converter/GAN039-650NxB_LTspice.lib)
- [`Driver.asy`](gan-sync-buck-converter/Driver.asy)
- [`PWM.asy`](gan-sync-buck-converter/PWM.asy)
- [`dead_time.asy`](gan-sync-buck-converter/dead_time.asy)
- [`switching.lib`](gan-sync-buck-converter/switching.lib)

The encrypted Nexperia model requires **LTspice 26.0.1 or newer**.

### GaN Synchronous Boost Converter

**Schematic:** [`gan-sync-boost-converter/GanSynchBoost.asc`](gan-sync-boost-converter/GanSynchBoost.asc)

The [`GanSynchBoost.asc`](gan-sync-boost-converter/GanSynchBoost.asc) schematic replaces the silicon IRFZ44N switches with two Nexperia `GAN039-650NxB` 650 V cascode GaN FET models.

| Parameter         | Value |
|-------------------|-------|
| Input voltage     | 12 V |
| Duty cycle        | 0.5 |
| Inductor (L1)     | 60 µH / 2 mΩ DCR |
| Input capacitor   | 100 µF |
| Bootstrap capacitor | 10 µF |
| Output capacitor  | 100 µF |
| Load resistance   | 12 Ω |
| GaN FETs          | Nexperia GAN039-650NxB |
| Gate resistors    | 22 Ω |
| Gate clamps       | BZX84C15L, 15 V |
| PWM frequency     | 100 kHz |
| Dead time         | 100 ns |
| Simulation        | 10 ms transient, UIC, initial Vout = 24 V |

The vendor model and matching symbol are stored beside the schematic:

- [`GAN039-650NxB.asy`](gan-sync-boost-converter/GAN039-650NxB.asy)
- [`GAN039-650NxB_LTspice.lib`](gan-sync-boost-converter/GAN039-650NxB_LTspice.lib)

The encrypted model is version 1.0.5 and requires **LTspice 26.0.1 or newer**. Older LTspice releases, including 17.0.x, fail while decoding the model and may report errors such as `Unknown symbol: aa`.

[Back to simulation library](#simulation-library)

---

## Shared Subcircuit Blocks

Reusable `.asy` symbols and the corresponding `switching.lib` implementation are stored with each applicable converter design.

| Block | Function | Key parameters |
|:------|:---------|:---------------|
| `PWM.asy` | Trailing-edge pulse-width modulation using a sawtooth comparator | 100 kHz nominal frequency, 1 V ramp, configurable duty limits |
| `Driver.asy` | Low-side or floating high-side gate driver | 3–6 A peak current, 30 ns delay, 1 mA quiescent current, 9 V UVLO |
| `dead_time.asy` | Complementary non-overlap signal generator | 100 ns default dead time |
| `switching.lib` | SPICE definitions for the reusable control blocks | PWM, driver, and dead-time subcircuits |

### File Types

| Extension | Purpose |
|:----------|:--------|
| `.asc` | LTspice schematic |
| `.asy` | Reusable schematic symbol |
| `.lib` | SPICE subcircuit or device model |
| `.raw`, `.log`, `.net` | Generated simulation outputs; excluded from Git |

---

## Simulation Setup

The designs use transient analysis with a maximum timestep of 20 ns. Initial conditions vary by topology to shorten startup time and focus the simulation on switching behavior.

### Directive Reference

**Buck converter (non-synchronous):**

```spice
.tran 0 2msec 0 20e-9 startup uic
.ic V(Vout)=0
.options reltol=0.0005
```

**Synchronous buck converter:**

```spice
.tran 0 2msec 0 20e-9 startup uic
.ic V(Vout)=0
.options reltol=0.0005
```

**Boost converter (non-synchronous):**

```spice
.tran 0 10msec 0 20e-9 UIC
.ic V(Vout)=24
.options reltol=0.0005
```

**Synchronous boost converter:**

```spice
.tran 0 10msec 0 20e-9 UIC
.ic V(Vout)=24
.options reltol=0.0005
```

**Ćuk converter:**

```spice
.tran 0 10msec 0 20e-9 startup uic
.options reltol=0.0005
```

Efficiency measurement directives in the Ćuk converter compute input power, output power, and efficiency over the last 2 ms:

```spice
.meas TRAN Pin AVG V(Vin)*I(Vg) FROM=8m TO=10m
.meas TRAN Pout AVG V(Vout)**2/5 FROM=8m TO=10m
.meas TRAN efficiency PARAM Pout/Pin*100
```

Measurement directives in the synchronous buck converter compute the reverse-recovery charge (`Qrr`) of the low-side MOSFET body diode and the switching energy (`Eevent`) during a specific switching event:

```spice
.meas tran Qrr INTEGRAL I(M2) FROM=1u TO=1.05u
.meas TRAN Eevent INTEGRAL V(n006)*Is(M2) FROM=549.9u TO=550.1u
```

---

## Model Scope

The simulations include the following electrical loss mechanisms and modeling assumptions.

### Included

| Mechanism | How It Is Modeled |
|:----------|:------------------|
| **Inductor DCR** | Series resistor `RL = 2 mΩ` on L1 captures DC copper loss |
| **MOSFET conduction** | IRFZ44N and IRFH5004 SPICE models include channel on-resistance `Rds(on)` |
| **GaN conduction** | The Nexperia GAN039-650NxB model includes nonlinear channel behavior, leakage, capacitances, and package interconnect parasitics |
| **Diode forward drop** | MBR735 and 1N5817 Schottky models capture `VF × IF` conduction loss |
| **Switching energy** | Selected designs integrate instantaneous MOSFET voltage and current during switching transitions |
| **Gate-drive loss** | Driver models charge and discharge nonlinear gate capacitances and include quiescent current |
| **Reverse recovery** | Selected designs integrate body-diode reverse-recovery current |
| **Auxiliary loss** | Bootstrap diode drops and driver cross-conduction are represented |

### Not Included

- Magnetic core losses (hysteresis, eddy current) — inductors are ideal aside from DCR
- Capacitor ESR — all capacitors are ideal (no series resistance)
- PCB parasitics — trace resistance, inductance, and coupling are omitted; the Nexperia GaN model still includes device-package interconnect parasitics
- Thermal effects — `Rds(on)` and diode `VF` are temperature-independent
- Skin/proximity effect in the inductor winding

---

## Getting Started

### Requirements

- [LTspice](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html)
- LTspice **26.0.1+** for the encrypted Nexperia GaN model

### Run a Simulation

1. Choose a design from the [simulation library](#simulation-library).
2. Download or clone the complete repository so local symbols and model libraries remain available.
3. Open the linked `.asc` schematic in LTspice.
4. Select **Simulate → Run**.
5. Probe `Vout`, `Vsw`, inductor current, and gate-drive signals.

> [!TIP]
> Keep each schematic beside its `.asy` and `.lib` dependencies. Opening an isolated `.asc` file may cause missing-symbol or missing-subcircuit errors.

---

## License

This project is provided for educational and reference purposes.

[Back to top](#dc-dc-power-converter-simulation-library)
