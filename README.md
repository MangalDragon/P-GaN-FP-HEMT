# P-GaN-FP-HEMT

Phase 1 TCAD work for the major project: **Design and Optimization of a High-Power and High-Frequency GaN HEMT Inverter for EVs**.

A 650 V-class enhancement-mode dual field plate p-GaN HEMT, simulated in Silvaco ATLAS 5.28.1.R, targeting integration into a 3-level ANPC inverter on an 800 V EV traction bus.

## Quick Navigation

- [`PROJECT.md`](PROJECT.md) — full project scope, deliverables, hard constraints
- [`docs/device-baseline.md`](docs/device-baseline.md) — confirmed-working device architecture, mesh, models, failure modes
- [`docs/tool-stack.md`](docs/tool-stack.md) — ATLAS / IC-CAP / ADS / Q3D / Icepak / Simulink roles and data flow
- [`docs/research-positioning.md`](docs/research-positioning.md) — honest novelty-vs-reproduction assessment with citations
- [`docs/data-collection.md`](docs/data-collection.md) — Phase 1 plots, cutlines, contours, and Tonyplot extraction steps
- [`decks/ganon_gan_fp.in`](decks/ganon_gan_fp.in) — working ATLAS deck

## Phase Split

| Phase | Tools | Owner |
|-------|-------|-------|
| 1 — Device TCAD | Silvaco ATLAS | MangalDragon (current) |
| 2 — System | IC-CAP, ADS, Q3D, Icepak, MATLAB/Simulink | Ujwal, Shreyash, Sheehari, Ronith |

Faculty advisor: Divakar Sir.

## Honest Framing

The device structure is commercial (Infineon CoolGaN, GaN Systems, Innoscience, ROHM, EPC, Power Integrations). The project's contribution is at the **system-application level**: TCAD-to-system co-design methodology, high-frequency operating-point analysis (>200 kHz, beyond the 50–100 kHz of current published GaN 3L-ANPC demos), and parametric sensitivity analysis. See [`docs/research-positioning.md`](docs/research-positioning.md) for the full breakdown.

## Hard Physical Limit (For Anyone Reviewing the Deck)

A p-GaN HEMT cannot achieve linear modulation past Vgs ≈ 4 V. The metal/p-GaN/AlGaN gate stack contains an intrinsic diode that forward-biases at the GaN bandgap (3.4 eV); past that, hole injection collapses transconductance. This is bandgap physics and is shared by every commercial p-GaN HEMT. The 4 V saturation in the transfer curve is a correct result, not a failure. Do not propose AlN spacers, MIS gates, or work-function tweaks to "fix" it.
