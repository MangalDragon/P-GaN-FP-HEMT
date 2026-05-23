# Research Positioning: Novelty vs. Reproduction

Honest assessment of where this project sits relative to the published state of the art. Used to frame the paper without overclaiming.

## What's Reproduction

The device structure itself — dual field plate p-GaN gate HEMT at 650 V class — is **the** commercial GaN power device architecture. Vendors shipping equivalent devices in volume:

- Infineon CoolGaN (acquired GaN Systems)
- Innoscience (INN650 series, e.g. INN650D080BS used in published 850 V demos)
- ROHM (GNP2070TD-Z, 650 V GaN HEMT in TOLL package)
- EPC, Navitas, Power Integrations (PowiGaN)
- Cambridge GaN Devices (ICeGaN)

3-level ANPC topology with 650 V GaN at 800 V bus has also been demonstrated in hardware:

- **800 V / 11 kVA, 3-phase 3L-ANPC, 650 V e-mode GaN HEMTs** for EV traction. ([Springer, Electrical Engineering, 2025](https://link.springer.com/article/10.1007/s00202-025-03442-8); preprint [arXiv 2406.15624](https://arxiv.org/abs/2406.15624)) — content paraphrased for compliance.
- **800 V / 100 kW full-GaN 3-level automotive inverter** using Cambridge GaN Devices parts. ([SAE 2025-24-0133](https://www.sae.org/content/2025-24-0133/)) — content paraphrased for compliance.
- **850 VDC 3L-ANPC reference demo** by Innoscience and University of Bern, no SiC diodes, no snubber capacitors. ([Electronic Specifier, 2023](https://www.electronicspecifier.com/news/innoscience-and-university-of-bern-develop-multilevel-topology-reference-demo)) — content paraphrased for compliance.
- **800 V / 50 kW 3L-ANPC** using automotive-qualified GaN Systems 650 V/60 A HEMTs at McMaster. ([McMaster Experts](https://experts.mcmaster.ca/scholarly-works/2658448)) — content paraphrased for compliance.

If the paper claims novelty in the device structure or the topology, reviewers will reject it. The structure is commercial, the topology is well-published.

## What's Open Research (the actual contribution)

Three genuinely open problems where this project can contribute:

### 1. High-frequency operation (>200 kHz) for EV traction

Production EV traction inverters today operate at switching frequencies up to ~20 kHz, near the ceiling of Si IGBTs. ([EDN, 2020](https://www.edn.com/gan-enables-efficient-cost-effective-800v-ev-traction-inverters/)) — content paraphrased. GaN can switch 5–20× faster than Si IGBTs in automotive converters. ([Patsnap analysis, 2025](https://eureka.patsnap.com/blog/hotspot/gan-power-devices-automotive-power-electronics-frequency-thermal-design-reliability/)) — content paraphrased.

But the published GaN 3L-ANPC traction demos (Springer 2025, SAE 2025-24-0133, McMaster, Bern) operate at 50–100 kHz, not 200+ kHz. The reasons are concrete and unsolved:
- Switching losses become dominant (linear in fsw).
- Layout parasitics (5–10 nH gate loop) cause ringing that distorts the switching waveform.
- Dynamic Ron degradation at Vgs = 3–4 V is significantly worse than at Vgs = 5–6 V. ([Maximizing the Performance of 650-V p-GaN Gate HEMTs, IEEE TPEL](https://www.researchgate.net/publication/308276525)) — content paraphrased for compliance.
- Vth instability under high-Vd stress: ~0.7 V positive shift as Vd rises from 0 to 650 V. ([Stability Analysis of p-GaN Gate AlGaN/GaN HEMTs](http://microbiologycommunity.nature.com/posts/stability-analysis-of-p-gan-gate-algan-gan-hemts-under-high-voltage-stress)) — content paraphrased for compliance.
- Thermal margin shrinks linearly with frequency.
- Motor harmonic interaction at 200+ kHz is not well-characterized.

A TCAD-validated, parasitic-aware system simulation that quantifies the efficiency/loss/derating envelope from 20 kHz to 500 kHz on an 800 V bus is a publishable contribution.

### 2. Cross-domain co-design methodology

Most published GaN inverter papers cover one or two domains: device, or circuit, or thermal, or system. Few span the full chain. A paper that demonstrates:

> ATLAS device → IC-CAP compact model → ADS HF circuit with Q3D parasitics + Icepak thermal → Simulink system → efficiency map vs. fsw

…with concrete numerical results at each stage is methodologically novel even if no single stage is new.

### 3. Parametric sensitivity to manufacturing tolerances

The dual-FP geometry (gate FP length, source FP length, FP-to-channel spacing) and p-GaN doping profile have known sensitivities. A TCAD parametric sweep showing how Vth, Ron, BVdss, and switching loss vary with these parameters — at the 200+ kHz operating point — gives device designers actionable guidance. None of the cited inverter demos do this.

## What's Not Open Research (Don't Claim)

- "Novel device structure" — it isn't.
- "First GaN 3L-ANPC EV inverter" — it isn't (multiple groups, 2023–2025).
- "GaN replaces SiC in production EVs" — too early to claim. Production EVs use SiC (Tesla Model 3, Hyundai E-GMP, Porsche Taycan, Lucid Air, Kia EV6). GaN traction is at prototype/demo stage. CGD announced 100 kW+ GaN EV powertrain technology in March 2025. ([eeNews Europe, 2025](https://www.eenewseurope.com/en/cgd-announces-breakthrough-100-kw-gan-ev-technology/)) — content paraphrased.
- "1200 V GaN" — research-grade only, not commercially available at scale. Out of scope. ([Reaching Beyond 1200 V: Lateral GaN HEMTs, IEEE](https://ieeexplore.ieee.org/document/10654197/)) — for context only.
- "6 V linear gate modulation on p-GaN" — bandgap-forbidden. The 4 V ceiling is industry-wide.

## Suggested Paper Title and Abstract Skeleton

**Working title:**
> "TCAD-to-System Co-Design of a 650 V Dual Field Plate p-GaN HEMT for High-Frequency 3-Level ANPC Traction Inverters at 800 V"

**Abstract skeleton (paraphrase, fill in numbers from your runs):**

> This paper presents a cross-domain co-design methodology for a 650 V class dual field plate p-GaN HEMT operating in a 3-level Active Neutral Point Clamped inverter at 800 V DC bus voltage targeting electric vehicle traction. A 2D drift-diffusion + lattice-thermal TCAD model in Silvaco ATLAS is used to extract DC, small-signal, breakdown, and self-heating characteristics, including the bandgap-limited Vgs ceiling at ~4 V intrinsic to all p-GaN gate stacks. The TCAD-derived compact model, fit using IC-CAP to the ASM-HEMT/MVSG framework, drives parasitic-aware switching simulations in ADS with Q3D-extracted busbar inductances, coupled to Icepak thermal simulation. A MATLAB/Simulink system model evaluates inverter efficiency, harmonics, and thermal derating across switching frequencies from 20 kHz to 500 kHz. Results identify the operating-frequency window where GaN system-level advantages over SiC become quantifiable, and characterize the device, parasitic, and thermal headroom available before dynamic Ron and Vth instability dominate.

This frames the contribution honestly: methodology + operating-point analysis, not device novelty.

## Citation List (Phase 1 Reading)

For the literature review section. Add to as needed.

**Hardware demonstrations:**
- Springer Electrical Engineering 2025 — 800V/11kVA 3-phase 3L-ANPC GaN ([link](https://link.springer.com/article/10.1007/s00202-025-03442-8))
- SAE 2025-24-0133 — 800V/100kW full-GaN 3-level automotive ([link](https://www.sae.org/content/2025-24-0133/))
- McMaster — 800V/50kW 3L-ANPC GaN with CMV analysis ([link](https://experts.mcmaster.ca/scholarly-works/2658448))
- Innoscience/Bern — 850VDC 3L-ANPC reference ([link](https://www.electronicspecifier.com/news/innoscience-and-university-of-bern-develop-multilevel-topology-reference-demo))

**Device reliability:**
- Maximizing 650V p-GaN HEMT performance, IEEE TPEL ([link](https://www.researchgate.net/publication/308276525))
- Vth instability under high-Vd stress, Nature/Microbiology Community ([link](http://microbiologycommunity.nature.com/posts/stability-analysis-of-p-gan-gate-algan-gan-hemts-under-high-voltage-stress))
- Surge current capability and failure modes of 650V p-GaN HEMTs, MDPI Electronics 2025 ([link](https://www.mdpi.com/2079-9292/14/7/1321))
- Threshold voltage instability in normally-off GaN HEMT, Cambridge thesis ([link](https://www.repository.cam.ac.uk/items/28cc228e-c285-440b-9982-c336aee10858))

**Compact models:**
- ASM-HEMT GitHub (Berkeley, Si2 standard) ([link](https://github.com/sourabhberkeley/ASM-HEMT))
- MVSG flow with TCAD (Silvaco) ([link](https://silvaco.com/simulation-standard/tcad-based-gan-hemt-scalable-modeling-flow-using-the-mvsg-compact-model/))

**Industry context:**
- GaN enables 800V EV traction inverters, EDN 2020 ([link](https://www.edn.com/gan-enables-efficient-cost-effective-800v-ev-traction-inverters/))
- CGD 100kW+ GaN EV technology, eeNews 2025 ([link](https://www.eenewseurope.com/en/cgd-announces-breakthrough-100-kw-gan-ev-technology/))
- ROHM GNP2070TD-Z 650V GaN ([link](https://www.evengineeringonline.com/new-gan-hemt-technology-supports-high-power-efficiency-in-evs/))
- Patsnap GaN automotive analysis ([link](https://eureka.patsnap.com/blog/hotspot/gan-power-devices-automotive-power-electronics-frequency-thermal-design-reliability/))

All web citations: content paraphrased for licensing compliance.
