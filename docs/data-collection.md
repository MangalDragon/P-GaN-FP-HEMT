# Phase 1 Data Collection Checklist

Plots, tables, cutlines, and contours required for the Phase 1 report and Phase 2 handoff. Each item lists what to collect, where it comes from in the deck, and how to extract it in Tonyplot 2019.

## From the Working Deck (Already Generated)

| # | Artifact | Source file | Status |
|---|----------|-------------|--------|
| 1 | Material/region map | `ganon_gan_fp_0.str` | ✅ have |
| 2 | Mesh visualization | `ganon_gan_fp_0.str` | extract |
| 3 | Transfer curve, Vg = −2 to +6 V | `ganon_gan_fp_transfer.log` | ✅ have |
| 4 | Isothermal output, Vg = 2/4/6 V | `ganon_gan_fp_Vg2.log`, `Vg4.log`, `Vg6.log` | ✅ have |
| 5 | Thermal output, Vg = 2/3/4 V | `ganon_gan_fp_Vg2_lat.log`, `Vg3_lat.log`, `Vg4_lat.log` | rerun (corrupted .str fixed) |
| 6 | Thermal-peak structure | `ganon_gan_fp_thermal_peak.str` | ✅ have |

## Cutlines and Contours to Extract from .str Files

| # | Plot | Source | Coordinate / quantity |
|---|------|--------|----------------------|
| 7 | Net doping (2D contour, log scale) | `ganon_gan_fp_0.str` | "Net doping" |
| 8 | Conduction-band Ec (2D contour) | `ganon_gan_fp_0.str` | "Conduction Band Edge" |
| 9 | Electron concentration (2D contour, log scale) | `ganon_gan_fp_0.str` | "Electron Conc" |
| 10 | Polarization charge density | `ganon_gan_fp_0.str` | "Polarization Charge" |
| 11 | E-field magnitude contour | `ganon_gan_fp_thermal_peak.str` | "Electric Field" |
| 12 | Lattice temperature contour | `ganon_gan_fp_thermal_peak.str` | "Lattice Temperature" |
| 13 | Vertical cutline at x = 4.5 μm: Ec, Ev, Ef vs. y | `ganon_gan_fp_0.str` | band diagram, E-mode proof |
| 14 | Horizontal cutline at y = 0.5115 μm: electron conc vs. x | `ganon_gan_fp_0.str` | 2DEG depletion under p-GaN |
| 15 | Horizontal cutline at y = 0.512 μm: \|E\| vs. x | `ganon_gan_fp_thermal_peak.str` | dual-FP field redistribution (two peaks) |
| 16 | Horizontal cutline at y = 0.512 μm: Tlat vs. x | `ganon_gan_fp_thermal_peak.str` | hot-spot location |
| 17 | Vertical cutline at hot-spot x: Tlat vs. y | `ganon_gan_fp_thermal_peak.str` | heat path to substrate |

## Extracted Numerical Values (For Tables)

| # | Quantity | How |
|---|----------|-----|
| 18 | Vth (linear extrapolation) | from transfer log, Tools → Extract → Vt |
| 19 | gm,max | derivative of transfer log, Tools → Functions → Derivative |
| 20 | Ron at Vg=4 V, Vd=1 V | from output log slope at low Vd |
| 21 | Saturation current Idsat at Vg = 2/3/4 V | from output logs |
| 22 | Thermal droop ΔId/Id at fixed (Vg, Vd) | (Id_iso − Id_lat) / Id_iso |
| 23 | Peak Tlat at Vg=4 V, Vd=8 V | probe on thermal contour |
| 24 | Rth_jc (junction to substrate) | (Tj − 300) / (Vd × Id) at peak |
| 25 | 2DEG sheet density | integrate electron conc cutline at AlGaN/GaN interface |

## To Be Added to the Deck (Next Iteration)

These are not yet in `decks/ganon_gan_fp.in` but are required for the Phase 2 handoff and the paper.

### Off-state breakdown (BVdss)
```
go atlas
mesh inf=ganon_gan_fp_0.str width=100
contact name=gate
contact name=source work=3.93
contact name=drain  work=3.93
models srh fldmob print
mobility albrct.n
model polarization calc.strain polar.scale=1.0
impact selb
method newton trap itlimit=200 maxtrap=50 climit=1e-3 dvmax=0.1

solve init
solve name=gate vgate=0.0 vfinal=-2 vstep=-0.5
log outf=ganon_gan_fp_BVdss.log
solve name=drain vdrain=0.0 vfinal=700 vstep=5.0
log off
```
Note: requires drain-side mesh refinement (s=0.05 or finer near drain) and may need `dvmax` reduction near breakdown. Watch for filamentation; if convergence breaks at ~600 V, that's likely your BVdss — record the last converged Vd.

### Small-signal AC analysis
At a chosen operating point (e.g., Vg=4 V, Vd=5 V):
```
solve init
solve name=gate vgate=0.0 vfinal=4 vstep=0.5
solve name=drain vdrain=0.0 vfinal=5 vstep=0.5
log outf=ganon_gan_fp_yparams_op4_5.log
solve ac freq=1e3 fstop=1e10 nfsteps=70
log off
```
This produces Y11, Y12, Y21, Y22 vs. frequency. Extract:
- Cgs ≈ Im(Y11+Y12) / ω at low f
- Cgd ≈ −Im(Y12) / ω at low f
- Cds ≈ Im(Y22+Y12) / ω at low f
- gm ≈ Re(Y21) at low f
- fT where |Y21/Y11| = 1
- fmax where MSG = 1 (derivable from Y-matrix)

Repeat at multiple bias points for the compact-model fit.

### Polarization sanity check
After `solve init`, save a structure file and probe `Polarization Charge` at the AlGaN/GaN interface (y = 0.512). Should be ~+1×10¹³ cm⁻² at the AlGaN side and −1×10¹³ at the GaN side. If you get an order of magnitude off, recheck `polar.scale`.

## Tonyplot 2019 — Quick Reference

**Loading:**
```
tonyplot ganon_gan_fp_0.str
tonyplot ganon_gan_fp_thermal_peak.str
tonyplot ganon_gan_fp_transfer.log
tonyplot -overlay file1.log file2.log file3.log
```

**Switching the displayed quantity (in a .str window):**
- **Plot → Display** → select one quantity (Conduction Band Edge, Electron Conc, |E|, Lattice Temperature, Net Doping, Polarization Charge, etc.).
- **Plot → Display → Properties** → log/linear, contour count, color map.

**Cutline:**
- **Tools → Cutline** → Horizontal or Vertical → click on the structure or type the coordinate.
- A new 1D plot window opens with the cutline data.

**Probe a single point value:**
- **Tools → Probe** → click on the contour.

**Curve fitting / extraction:**
- **Tools → Functions → Derivative** for gm.
- **Tools → Extract** has presets including Vt linear-extrapolation.

**Export:**
- 1D cutline data: **File → Export → .dat** (tab-separated x–y, replot in MATLAB/Origin for paper figures).
- Image: **File → Print → Postscript / PNG** for visual inspection. For paper-quality, use the .dat and replot.

**Annotation:**
- **Edit → Properties** → axis labels, titles, line styles.

## Recommended Figure Set for the Phase 1 Report

These cover the device, the validation, and the parametric handoff. Phase 2 builds on them.

1. Cross-section schematic (annotated structure plot).
2. Mesh density plot (justifies refinement near 2DEG).
3. Net doping contour (validates p-GaN cap, ohmics).
4. Band diagram cutline at x=4.5 μm (E-mode proof).
5. 2DEG profile cutline at y=0.5115 μm (depletion under p-GaN).
6. Polarization charge map (validates `polar.scale=1.0`).
7. Transfer curve (Vth, gm extraction).
8. Output curves, isothermal Vg=2/4/6 V.
9. Output curves, thermal Vg=2/3/4 V (with overlay vs. isothermal showing droop).
10. E-field cutline at y=0.512 μm (two peaks → field-plate validation).
11. Lattice temperature contour at peak Id.
12. Off-state breakdown sweep (after impact-ionization run is added).
13. Small-signal model summary table (Cgs, Cgd, Cds, gm, fT, fmax) at one operating point.
