# Device Baseline: Dual Field Plate p-GaN HEMT

This is the confirmed-working device architecture and simulation deck. **Do not change** any of this without an explicit decision recorded here.

## Architecture

| Element | Specification |
|---------|---------------|
| AlGaN barrier | 12 nm, x.comp = 0.20 (20% Al) |
| GaN buffer | y = 0.512 to 3.0 μm |
| p-GaN gate cap | 100 nm, x = 4–5 μm; graded doping (1e18 cm⁻³ near AlGaN, 1.5e19 cm⁻³ deeper) |
| Si₃N₄ passivation | source side, drain side, top |
| Gate field plate | x = 5–8 μm |
| Source field plate | x = 1–12 μm, y = −0.2 to −0.1 μm |
| Source/drain ohmics | 1e21 cm⁻³ n+ doping; work = 3.93 eV |
| Gate contact | blank (no work function specified — defaults to ohmic; **do not change**) |
| L_GS / L_g / L_GD | 3 μm / 1 μm / 14 μm |
| Total device width | 20 μm; mesh width = 100 μm |

## Confirmed Working Mesh

```
x.m l=0.0 s=0.5
x.m l=1.0 s=0.1
x.m l=3.0 s=0.05
x.m l=4.0 s=0.02
x.m l=5.0 s=0.02
x.m l=7.0 s=0.02
x.m l=8.0 s=0.05
x.m l=12.0 s=0.1
x.m l=19.0 s=0.2
x.m l=20.0 s=0.5

y.m l=-0.2 s=0.02
y.m l=-0.1 s=0.02
y.m l=0.0 s=0.05
y.m l=0.2 s=0.02   # CRITICAL: fixes electrode-4 shortening warning
y.m l=0.4 s=0.02
y.m l=0.45 s=0.01
y.m l=0.47 s=0.005
y.m l=0.5 s=0.001
y.m l=0.512 s=0.001
y.m l=0.6 s=0.02
y.m l=3.0 s=0.5
```

Resulting mesh: ~27,571 nodes, ~54,288 triangles, 0 obtuse triangles. Healthy.

## Confirmed Working Models and Methods

**Isothermal block:**
```
models srh fldmob print
mobility albrct.n
model polarization calc.strain polar.scale=1.0   # MUST be its own line, not inside `models`
method gummel newton itlimit=50 trap maxtrap=15 climit=1e-4
```

**Thermal block:**
```
models srh fldmob lat.temp print
mobility albrct.n
model polarization calc.strain polar.scale=1.0
material material=GaN tcon.alma hc.alma
thermcontact num=1 x.min=0 x.max=20 y.min=3 y.max=3 ext.temp=300 alpha=1e7
method newton trap itlimit=150 maxtrap=50 climit=1e-3 dvmax=0.1
```

Note: `lat.temp` belongs on the `models` line, **not** on the `output` line. Putting it on `output` causes ATLAS error #3 in 5.28.1.R.

## Confirmed Working Doping Syntax

Use box coordinates with `uniform` keyword first. Do **not** mix `region=N` with y bounds — causes Gaussian default fallback and 2000 K thermal runaway.

```
doping uniform p.type conc=1e18   x.min=4.0  x.max=5.0 y.min=0.47 y.max=0.5
doping uniform p.type conc=1.5e19 x.min=4.0  x.max=5.0 y.min=0.40 y.max=0.47
doping uniform n.type conc=1e21   x.min=0.0  x.max=1.0 y.min=0.5  y.max=0.6
doping uniform n.type conc=1e21   x.min=19.0 x.max=20  y.min=0.5  y.max=0.6
```

## Known Failure Modes (Solved)

| Symptom | Root cause | Fix |
|---------|------------|-----|
| 2000 K thermal runaway | Gaussian doping default (mixed region= with y bounds) | Use `uniform` keyword first, box coords only |
| Vg ≈ 3.95 V matrix collapse | p-GaN/AlGaN diode forward-biases at GaN bandgap (3.4 eV); hole injection collapses transconductance | Split gate ramp through danger zone: vstep=0.05 between Vg=3.0 V and Vg=4.0 V |
| Internal linear solver error | Mesh transition jump | Add `y.m l=0.2 s=0.02` node |
| `tc.const`, `tc=`, `hc=` syntax errors | Invalid in 5.28.1.R | Use `tcon.const ktc=X.XX` if non-default |
| `TEMPER=300` rejected | Wrong syntax | Use `ext.temp=300` |
| `block gummel newton` conflicts | 5.28.1.R bug unless `carriers=1 electron` is added | Don't use `block` — `method gummel newton` for isothermal, `method newton` for thermal |
| Stale `.str` after split-machine run | Thermal block reads `ganon_gan_fp_0.str` written by isothermal block; if the file is from a different deck version, electrode count mismatches | **Never split execution.** Run the entire deck top-to-bottom on one machine. Delete stale `.str` before rerun. Diagnostic: if thermal block reports fewer than 6 electrodes on `mesh inf=...`, the .str is stale. |

## Failed Approaches (Do Not Re-Suggest)

These were tested and broke the device. Marked for record so they aren't re-tried.

- **AlN spacer at AlGaN/GaN interface:** broke E-mode (Vth went negative, 42 mA standing current at Vg = −2 V). Even with p-GaN doping bumped to 4e19 and gate work function 5.1 eV, did not fully restore E-mode.
- **Hybrid Al₂O₃ MIS gate (12 nm Al₂O₃ between metal and p-GaN):** weakened gate coupling; transfer curve became flat (1.4 mA modulation over 10 V swing).
- **Reducing Al% to 15%:** reduced 2DEG (peak 75 mA → 55 mA, confirming 2DEG sensitivity) but did **not** shift the saturation knee. Empirically confirms the saturation ceiling is set by the p-GaN/AlGaN diode wall at 3.4 eV bandgap, not by 2DEG velocity saturation.
- **MIS-HEMT (recessed gate, no p-GaN cap):** different device class; out of scope.
- **1200 V drift region stretching:** Phase 2+ research, out of scope for Phase 1.

## Hard Physical Truth (For Report Framing)

A p-GaN HEMT cannot achieve 6 V linear modulation in I_d–V_g. The metal/p-GaN/AlGaN gate stack contains an intrinsic diode that forward-biases at Vg ≈ 3.4 V (set by GaN bandgap). Once forward-biased, hole injection collapses transconductance. This is bandgap physics; it applies to every commercial p-GaN HEMT (Infineon CoolGaN, GaN Systems, EPC, Navitas, Innoscience, Power Integrations PowiGaN). All commercial datasheets cap operating Vgs at 3.3–5 V for this reason. The 4 V saturation in our transfer curve is a physically correct result, not a failure.

## Recommended Sweeps for Reports

- **Transfer:** Vg from −2 V to +6 V at vstep=0.1 (full curve including saturation, for figure clarity).
- **Isothermal output:** Vg = 2, 4, 6 V (Vg=6 overlays Vg=4, kept for completeness in the existing dataset).
- **Thermal output:** Vg = 2, 3, 4 V (all within active modulation region; Vg=6 thermal provides no extra info because the channel is already saturated).

## Current Status

| Run | Status |
|-----|--------|
| Isothermal section | works cleanly. Vth ≈ +1.2 V, knee ≈ +4 V, peak ~75 mA |
| Thermal Vg=2 V | works |
| Thermal Vg=3 V | works (with split ramp 2.5→3.0 at vstep=0.05) |
| Thermal Vg=4 V | works only when full deck runs top-to-bottom on one machine |

Last confirmed-good baseline matches plots: transfer curve with Vth≈+1.2 V and ~75 mA peak; isothermal output overlay at Vg=2/4/6 (~60 mA / ~250 mA / ~280 mA at Vd=15 V); thermal output overlay at Vg=2/4 (~25 mA / ~125 mA at Vd=8 V).
