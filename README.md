# COMSOL model: capillary imbibition in a paper-based porous silicon biosensor

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

This repository contains the COMSOL Multiphysics® model file accompanying:

> H. An, N. Mangnike, D. Hyde, P. E. Laibinis, A. K. Locke, and S. M. Weiss.
> **Controlling Incubation Time and Flow Dynamics in Porous Silicon
> Paper-Based Biosensors Combining Vertical Flow and Lateral Flow.**
> Manuscript under review (2026).

The model simulates water transport through a porous silicon (PSi) sensing
pad mounted on a paper microfluidic channel, and predicts the incubation
time t_inc for paper channels of widths 0.5–5 mm. It solves Richards'
equation in 3D with a volume-budget-triggered boundary condition that
tracks the loading droplet via a global ODE and switches the top-surface
pressure once the droplet is fully imbibed.

## Repository contents

```
.
├── README.md                    ← you are here
├── LICENSE                      ← CC-BY 4.0
└── PSi_paper_imbibition.mph     ← COMSOL model file (solutions cleared)
```

The model file ships with **all solutions cleared** so that users
recompute results from scratch — this is intentional for reproducibility
and keeps the repository lightweight (~1 MB). A full parametric sweep
populates the solution caches and brings the working file to ~200 MB
on disk; clear solutions before committing changes back.

## Requirements

| Component | Version |
|---|---|
| COMSOL Multiphysics® | 6.1 (developed in 6.1) |
| Subsurface Flow Module | required (Richards' Equation interface) |

The model was developed and tested on COMSOL 6.1. It should open and
run on later versions and on all supported operating systems (Windows,
macOS, Linux) without modification. Total runtime for the full 6-channel
parametric sweep is approximately 4–8 hours on a workstation with 16
cores and 64 GB RAM; single-channel runs complete in 30–60 minutes.

## Quick start

1. Clone or download this repository.
2. Open `PSi_paper_imbibition.mph` in COMSOL 6.1+. The model contains:
   - **Component 1** — 3D geometry (PSi stack + paper channel + absorbent pad)
   - **Richards' Equation (dl)** — primary physics interface
   - **Global ODEs and DAEs (V_in)** — cumulative-inflow integral V_in(t)
   - **Step 1 (step1)** — smoothed Heaviside function used by the BC
   - **Study 1: Parametric Sweep** — sweeps the `Channel` parameter over
     {0.0005, 0.001, 0.002, 0.003, 0.004, 0.005} m
3. Compute Study 1. The incubation time t_inc is the switching time of
   the top boundary condition, read from the simulated pressure
   trajectory at the top boundary — the moment the BC transitions from
   ~0 Pa to the dry-reservoir suction.
4. Compare simulated t_inc against the experimental values shown in
   Fig. 4a of the paper and Fig. S3 of the Supporting Information.

## Model description (brief)

A 3D porous-medium domain is constructed to mirror the experimental µPAD
stack of the paper: three thin PSi sub-layers (PSi1 / PSi2 / PSi3, with
the thin top layer PSi1 being the sensing layer), a paper microfluidic
channel below them, and a super-absorbent pad acting as a sink. Richards'
equation is solved on the union of all porous regions with a van
Genuchten retention curve and Mualem relative permeability. See
**Sections S1–S2 of the Supporting Information** for the governing
equations and the full parameter set.

The single forcing boundary condition is applied at the top of PSi1, where
a 20 µL loading droplet rests. The droplet is *not* resolved as a free
3D body; instead its effect enters through:

- a **global ODE** that integrates the inward Darcy flux to give the
  cumulative volume V_in(t) drawn into the device;
- a **linear evaporation budget** V_evap(t) = q_evap · t with q_evap
  from the diffusion-limited sessile-droplet model of Hu & Larson (2002),
  giving an effective available volume
  V_drop_eff(t) = V_drop,0 − V_evap(t);
- a **volume-budget-triggered switched pressure BC** (`Pressure 1`)
  of the form
  p_top(t) = P_res · step1((V_in − V_drop_eff)/(0.1·V_drop,0)) − δp_ε,
  where `step1` is a COMSOL smoothed Heaviside function that rises from
  0 to 1 over 5×10⁻⁴ ≤ x ≤ 1.5×10⁻³ (location 0.001, smoothing width
  0.001). The BC therefore stays at ~0 Pa while V_in < V_drop_eff (the
  droplet is still supplying water), and switches to
  P_res = −1×10⁵ Pa once V_in exceeds V_drop_eff (the droplet is
  consumed and the top surface sees the dry ambient environment). With
  0.1·V_drop,0 = 2 µL, the switch completes within about 2 nL of inflow,
  i.e. effectively at the instant the droplet is used up.

This design avoids hard-coding the per-channel switching time t_sw and
makes the model fully predictive: t_sw is **emergent** from the coupled
Richards' equation and global-ODE solution, not an input parameter.
The legacy time-based BC `Pressure 2` (a tanh switch that uses the
parameter `t_inc`) is present in the model tree but disabled; it is
retained only for diagnostic comparison.

## Known limitations

The model is a simplified surrogate for a fully multiphase free-surface
formulation, and includes several modeling choices documented in the
Supporting Information. The most consequential are:

1. The droplet is tracked as a scalar volume budget, not resolved as a
   3D free surface; contact-angle dynamics are folded into the constant
   evaporation rate.
2. The simulated saturation cannot reach S_e = 0 or 1 exactly within
   a tractable solver window; a linear normalization (Eq. S16 of the
   Supporting Information) maps the accessible range [0.402, 1.000]
   onto [0, 1] for presentation purposes only. This rescaling does
   **not** affect the t_inc determination, which is read from the
   simulated pressure trajectory directly.
3. Permeability, van Genuchten parameters, and the Mualem
   pore-connectivity exponent (l = 1) are calibrated within ranges
   reported in the literature; per-material values are listed in
   Table S2 of the Supporting Information.

## How to cite

If you use this model, please cite the paper above. Full bibliographic
details will be added here once the paper is published.

## License

All content in this repository — the COMSOL model file and documentation
— is released under the
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC-BY 4.0). You are free to share and adapt the material for any purpose,
including commercially, provided you give appropriate credit.

COMSOL Multiphysics® is a registered trademark of COMSOL AB. Use of
the software requires a separate, valid COMSOL license.

## Contact

Questions, bug reports, or requests for clarification: please open an
issue on this repository, or contact the corresponding author at the
address listed in the paper.
