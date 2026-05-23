# Tool Stack and Workflow

The full project spans device physics, compact modeling, parasitic extraction, thermal CFD, and system simulation. Each tool owns one piece. This document defines the data flow between them.

## Tool Roles

### Silvaco ATLAS 5.28.1.R — Device Physics (Phase 1)
**Owner:** MangalDragon (current).

**Purpose:** First-principles 2D drift-diffusion + thermal simulation of the dual-FP p-GaN HEMT.

**Outputs needed for downstream tools:**
- DC I-V (transfer + output, isothermal + lattice-thermal)
- Off-state breakdown (BVdss) with `impact selb`
- Small-signal Y-parameters at multiple bias points (`ac freq=...` sweep)
- Cgs, Cgd, Cds, gm, fT, fmax (derived from Y-params)
- Lattice temperature map at peak Id (.str file)
- 2D field maps for the Phase 1 report (cutlines, contours)

**Built-in capabilities relevant to this project:**
- `ac` analysis statement performs small-signal AC analysis around a DC bias point and outputs the Y-parameter matrix per electrode. From this you derive the full small-signal model. ([Silvaco app note](https://silvaco.com/dynamicweb/jsp/downloads/DownloadDocStepsAction.do?nm=simstd_nov_1999_hints.pdf&req=download)) — content paraphrased for compliance.
- `impact selb` enables Selberherr impact ionization for breakdown.
- `tcon.alma hc.alma` give physically correct thermal conductivity and heat capacity for GaN.

### Keysight IC-CAP — Compact Model Parameter Extraction
**Owner:** Phase 2 team.

**Purpose:** Take TCAD I-V + S-parameters (or measured data on a real device) and fit them to a SPICE-compatible compact model.

**Compact models for GaN HEMTs (industry standard):**
- **ASM-HEMT** — Berkeley, surface-potential based, includes velocity saturation, access-region resistance, DIBL, temperature dependence, gate current, and noise. Adopted by Si2 Compact Model Coalition. ([ASM-HEMT GitHub](https://github.com/sourabhberkeley/ASM-HEMT), [model paper](https://www.researchgate.net/publication/275045749_ASM-HEMT_Compact_model_for_GaN_HEMTs)) — content paraphrased for compliance.
- **MVSG (MIT Virtual Source GaN)** — physics-based, predicts a wide range of GaN HEMT effects. Also Si2 CMC standard. ([Silvaco MVSG flow](https://silvaco.com/simulation-standard/tcad-based-gan-hemt-scalable-modeling-flow-using-the-mvsg-compact-model/)) — content paraphrased for compliance.

For a power-electronics project (kHz–MHz, switching), ASM-HEMT is generally the better fit because it cleanly handles dynamic Ron, trapping, and self-heating. For RF (GHz), either works.

**Workflow:**
1. ATLAS produces I-V curves at multiple Vg, Vd and S/Y params at multiple bias points.
2. Export ATLAS log files / Y-matrix data.
3. IC-CAP loads measured/simulated data, runs optimizer, fits ASM-HEMT parameters.
4. Output: a `.va` Verilog-A model file or a SPICE subcircuit consumable by ADS or Simulink.

### Keysight ADS — Switching Circuit Simulation
**Owner:** Phase 2 team.

**Purpose:** High-frequency (kHz–MHz) circuit-level simulation including the gate driver, parasitics, and power-stage waveforms.

**What ADS does that MATLAB cannot:**
- Harmonic balance for ringing, EMI, dv/dt and di/dt analysis.
- Native support for distributed elements (transmission lines, S-parameter blocks).
- Direct import of compact models (ASM-HEMT/MVSG `.va`) and Q3D-extracted parasitic models (Touchstone or SPICE).
- Eye diagrams, frequency-domain spectra at the switch node.

**Inputs:** Compact model from IC-CAP, parasitic SPICE deck from Q3D, gate driver IC model.
**Outputs:** Switching loss per cycle, gate-loop ringing, dv/dt, peak overshoot, EMI spectrum.

### Ansys Q3D Extractor — Parasitic Extraction
**Owner:** Phase 2 team.

**Purpose:** 3D quasi-static R, L, C extraction of busbars, PCB, package, and wire bonds. Critical for >100 kHz operation where parasitic inductance dominates switching behavior.

**What it produces:**
- Frequency-dependent self and mutual inductance matrices (typically 1 kHz to 100 MHz sweep)
- DC and AC resistance including skin and proximity effects
- Capacitance matrix for coupling between busbar segments

**Why it matters for GaN at HF:**
A 5–10 nH gate-loop inductance, harmless at 20 kHz, causes catastrophic ringing at 200 kHz with GaN's 5–10 ns transitions. Q3D extraction lets the team optimize PCB layout before fabrication. Q3D outputs feed back into ADS as a SPICE subcircuit. ([Ansys Q3D power module white paper](https://www.ansys.com/resource-center/white-paper/extracting-parasitic-impedance-semiconductor-power-modules)) — content paraphrased for compliance.

### Ansys Icepak — Thermal / CFD
**Owner:** Phase 2 team.

**Purpose:** 3D thermal simulation of the inverter assembly: device junction, package, heatsink, airflow.

**Inputs:** Switching loss vs. frequency from ADS, thermal resistance network derived from device + package geometry.
**Outputs:** Steady-state and transient junction temperatures, heatsink sizing, airflow requirements.

**Coupling with ATLAS:** ATLAS gives you device-level junction-to-back-of-substrate thermal resistance. Icepak picks up from there with package, TIM, heatsink, and ambient. The two tools together give a full thermal stack from 2DEG to ambient air.

### MATLAB / Simulink — System-Level Inverter
**Owner:** Phase 2 team (Ujwal, Shreyash, Sheehari, Ronith).

**Purpose:** 3L-ANPC topology simulation, modulation, motor drive, control loop, efficiency map, harmonic analysis.

**Inputs:** Compact model wrapped as Simulink block (or efficiency lookup table from ADS), thermal derating curves from Icepak, parasitic loop inductance from Q3D.
**Outputs:** Inverter efficiency vs. switching frequency, vs. load, vs. modulation index. Motor harmonic content. THD. The headline figures of the paper.

## End-to-End Data Flow

```
                    ┌──────────────┐
                    │ Silvaco ATLAS│  (Phase 1, current)
                    │  device 2D   │
                    └──────┬───────┘
                           │ I-V, S/Y-params, Tj, BVdss, .str
                           ▼
                    ┌──────────────┐
                    │   IC-CAP     │  fit ASM-HEMT or MVSG
                    │ compact-model│
                    │   extract    │
                    └──────┬───────┘
                           │ .va or SPICE subckt
                           ▼
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
┌────────────┐      ┌────────────┐      ┌────────────┐
│  Ansys Q3D │      │ Keysight   │      │ Ansys      │
│  parasitic │─────►│   ADS      │      │ Icepak     │
│  extraction│ L,R,C│ HF circuit │      │ thermal    │
└────────────┘      │ + gate drv │      │ + heatsink │
                    └─────┬──────┘      └─────┬──────┘
                          │ Eswitch, dv/dt    │ Tj(t), Rth
                          └─────────┬─────────┘
                                    ▼
                            ┌────────────────┐
                            │  MATLAB /      │
                            │  Simulink      │
                            │  3L-ANPC + PMSM│
                            │  + control     │
                            └────────┬───────┘
                                     │
                                     ▼
                       Efficiency map vs. f_sw,
                       THD, derating curves,
                       paper figures
```

## What Phase 1 (this repo) Owes the Rest of the Stack

Concrete handoff artifacts to produce before closing Phase 1:

1. **DC I-V tables** at minimum (Vg, Vd, Id, Ig) covering Vg = −2 to +6 V in 0.1 V steps and Vd = 0 to 15 V in 0.2 V steps, isothermal.
2. **Thermal I-V tables** at Vg = 2, 3, 4 V from 0 to 8 V Vd.
3. **AC small-signal table** (Y-params) at the operating point Vg = 4 V, Vd = 5 V, frequency 1 kHz to 10 GHz log-spaced. (Sweep added to deck in next iteration.)
4. **Breakdown sweep:** Vg = −2 V, Vd ramp from 0 to 700 V with `impact selb`. (To be added in next iteration.)
5. **Extracted small-signal lumped model**: Cgs, Cgd, Cds, Rg, Ri, gm, τ, Cdsub at the operating point.
6. **Thermal resistance:** Rth_jc from ATLAS junction to substrate (300 K thermcontact at y=3).
7. **Polarization charge density** at the AlGaN/GaN interface (sanity-check value, ~1×10¹³ cm⁻²).

These are the inputs Phase 2 needs to start.
