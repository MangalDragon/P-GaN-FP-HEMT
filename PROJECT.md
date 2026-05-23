# Design and Optimization of a High-Power and High-Frequency GaN HEMT Inverter for EVs

Phase 1 Major Project. Lead: MangalDragon. Faculty advisor: Divakar Sir. Phase 2 collaborators: Ujwal, Shreyash, Sheehari, Ronith.

## Project Scope

A two-phase major project investigating a 650 V-class enhancement-mode dual field plate p-GaN HEMT and its integration into a 3-level Active Neutral Point Clamped (3L-ANPC) inverter for 800 V EV traction. The novelty is in the operating point and the cross-domain methodology, not in the device structure.

## Target Operating Point

| Parameter | Value | Notes |
|-----------|-------|-------|
| DC bus | 800 V | EV traction, post-400V transition |
| Topology | 3-level ANPC | Each switch sees ~Vbus/2 ≈ 400 V |
| Device | 650 V dual FP p-GaN HEMT | Commercial-class architecture |
| Switching frequency target | >200 kHz | vs. 10–20 kHz for production Si IGBTs, 50–100 kHz for current GaN demos |
| Power | scalable, 11 kVA prototype reference | Phase 2 system simulation up to 100 kW class |

## What This Project Is and Is Not

**Is:**
- Applied research on **high-frequency GaN traction inverter co-design** (device → compact model → circuit → parasitics → thermal → system).
- A TCAD-validated parametric model for a commercial-class 650 V GaN HEMT.
- Quantification of the 200+ kHz operating regime where GaN system-level advantages over SiC become measurable.
- Honest framing of bandgap-limited gate modulation (~4 V ceiling) as a hard physical constraint shared with all commercial p-GaN HEMTs.

**Is not:**
- A novel device architecture. Dual FP p-GaN gate is shipped by Infineon CoolGaN, GaN Systems, Innoscience, ROHM, EPC, and others.
- A claim that GaN replaces SiC in production EVs. As of mid-2026, no production EV traction inverter uses GaN; production is IGBTs (400 V) or SiC MOSFETs (800 V).
- Device-physics frontier work (no AlN spacer, no MIS gate, no novel materials, no 1200 V drift).

## Publishable Contribution

Working title (placeholder):
> "TCAD-to-System Co-Design of a 650 V Dual Field Plate p-GaN HEMT for High-Frequency 3-Level ANPC Traction Inverters at 800 V"

Concrete deliverables that constitute the contribution:
1. TCAD-validated DC + small-signal compact model (ASM-HEMT or MVSG fit) for a 650 V dual-FP p-GaN HEMT.
2. Frequency sweep (20 kHz → 500 kHz) of 3L-ANPC efficiency at 800 V, with parasitic-aware gate driver model.
3. Thermal derating curve as a function of switching frequency at fixed output power.
4. Identification of the bandgap-limited gate modulation ceiling and its impact on dynamic Ron and Vth instability at the chosen Vgs operating point.
5. Comparison vs. SiC MOSFET baseline at the same power and bus voltage.

## Phase Split

### Phase 1 — TCAD (current)
Tool: Silvaco ATLAS 5.28.1.R. See `docs/device-baseline.md` for the working deck, mesh, models, and known failure modes.

Outputs from Phase 1:
- DC I-V (transfer + output curves, isothermal and lattice-thermal).
- Threshold voltage Vth, on-resistance Ron, transconductance gm.
- Off-state breakdown voltage BVdss (with `impact selb` — to be added).
- Small-signal AC parameters (Y-matrix, Cgs, Cgd, Cds, fT, fmax — to be added via `ac freq=...` sweeps).
- Thermal map at peak Id, lattice temperature contour and cutlines.
- Compact model parameter set for handoff to IC-CAP / Phase 2.

### Phase 2 — System Simulation (handoff)
Tools: Keysight IC-CAP (compact model fitting), Keysight ADS (HF switching), Ansys Q3D (parasitics), Ansys Icepak (thermal), MATLAB/Simulink (3L-ANPC + motor + control). Owners: Ujwal, Shreyash, Sheehari, Ronith.

See `docs/tool-stack.md` for the full workflow and inter-tool data flow.

## Key Documents in This Repo

- `decks/ganon_gan_fp.in` — working ATLAS deck (Phase 1 baseline)
- `docs/device-baseline.md` — device architecture, working models, failure modes, physical limits
- `docs/tool-stack.md` — purpose of each tool and how outputs flow between them
- `docs/research-positioning.md` — honest assessment of novelty vs reproduction, with literature citations
- `docs/data-collection.md` — list of plots, cutlines, contours required for the report; how to extract them in Tonyplot

## Hard Constraints (Do Not Re-Litigate)

These have been tested and confirmed. Do not re-propose changes to any of these without explicit approval:
- p-GaN cap (no AlN spacer, no Al₂O₃ MIS gate, no work function tweak on gate).
- AlGaN at 20% Al composition.
- Vg operating window: 0 → 4 V. Hard ceiling at ~4 V set by p-GaN/AlGaN diode forward bias (GaN bandgap, 3.4 eV). This is industry-wide for p-GaN HEMTs, not a project limitation.
- 650 V class, not 1200 V (drift region stretching is a Phase 2+ research target, out of scope).
- TCAD device width 20 μm, mesh width 100 μm, total y from −0.2 to 3.0 μm.
- Run the deck top-to-bottom on one machine. Do not split execution.
