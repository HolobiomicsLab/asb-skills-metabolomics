---
name: j-coupling-spin-system-parameterization
description: Use when building metabolite basis functions for MRS fitting (e.g., PRESS,
  MEGA-PRESS, HERMES, HERCULES) and the spin-system Hamiltonians — J-coupling values
  and chemical shifts per metabolite — must be specified before density-matrix simulation.
when_to_use_negative:
- 'The goal is to model every possible vendor experiment: MRSCloud simulations cannot
  capture all possible Philips experiments or all disseminated versions of MEGA-PRESS
  within each vendor.'
- 'The spin-system parameterization itself must be re-derived or updated: the literature
  parameters still rest largely on Govindaraju et al. with only minor updates, so
  novel compounds outside the listed metabolites (e.g., beyond EtOH/MSM exogenous
  entries) are not covered.'
- Non-brain tissue or non-3T field strengths are the target — this work has only been
  focused and tested on brain MRS data using 3T scanners, and the exiting version
  only supports 3T.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3331
tools:
- name: MRSCloud
  role: MATLAB tool that consumes the spin-system definitions (via simMRS.json) together
    with vendor RF waveforms to simulate 3T metabolite basis functions using 1D projection,
    coherence pathway filters, and precalculated propagators
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Provides the spectral simulation functionality and spin-system framework upon
    which MRSCloud's metabolite simulations are built
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Independent density-matrix simulator used as a cross-validation reference
    against which MRSCloud's parameterized basis functions are compared (ICCs at least
    0.96)
- name: LCModel
  role: Provides an experimentally measured (phantom-acquired) basis set (e.g., press_te30_3t_01a.basis)
    used as an external comparison target for the simulated basis functions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:b07d6d504f5c65157828bb859d3c7b521009d2540dcd372b9cdf125b77d950ba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - LCModel
  - MARSS
  - MRSCloud
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3331
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# j-coupling-spin-system-parameterization

## Summary

Defines the quantum-mechanical spin-system description (chemical shifts, scalar J-couplings, and spin counts) for each metabolite so density-matrix simulations in MRSCloud/FID-A produce vendor- and sequence-specific MRS basis functions at 3T.

## When to use

Use when building metabolite basis functions for MRS fitting (e.g., PRESS, MEGA-PRESS, HERMES, HERCULES) and the spin-system Hamiltonians — J-coupling values and chemical shifts per metabolite — must be specified before density-matrix simulation.

## When NOT to use

- The goal is to model every possible vendor experiment: MRSCloud simulations cannot capture all possible Philips experiments or all disseminated versions of MEGA-PRESS within each vendor.
- The spin-system parameterization itself must be re-derived or updated: the literature parameters still rest largely on Govindaraju et al. with only minor updates, so novel compounds outside the listed metabolites (e.g., beyond EtOH/MSM exogenous entries) are not covered.
- Non-brain tissue or non-3T field strengths are the target — this work has only been focused and tested on brain MRS data using 3T scanners, and the exiting version only supports 3T.

## Inputs

- simMRS.json configuration file listing metabolites of interest from the MRSCloud spin-system library
- Spin-system definitions (chemical shifts and J-coupling tables) inherited from FID-A / Govindaraju et al. parameter sets
- Acquisition parameters: vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), sequence (UnEdited/MEGA/HERMES/HERCULES), TE, editOn/editOff, editTp (14 ms for MEGA-PRESS/HERMES, 20 ms for HERCULES), spatial_points (41 acceptable, 101 ideal)

## Outputs

- Density-matrix-simulated, vendor- and sequence-specific metabolite basis functions at 3T (zipped basis set with a PDF spectra image for quality assurance)
- Per-metabolite voxel-averaged FIDs usable for LCModel/Osprey/FSL-MRS/spant-style linear-combination fitting

## How to apply

Specify the metabolites of interest in simMRS.json from MRSCloud's curated spin-system lists: common healthy-brain metabolites (Asc, Asp, Cr, EA, GABA, GPC, GSH, Gln, Glu, Gly, H2O, Lac, mI, NAA, NAAG, PCh, PCr, PE, Ser, sI, Tau), metabolites for specific interests (Ala, Ace, AcO, AcAc, Cit, Cystat, HCar, Lys, Thr, bHG, Tyros, Val, Phenyl, bHB, Gua, iLe, Pyr, Suc, Tryp), and exogenous compounds (EtOH, MSM). Base the J-coupling and chemical-shift parameters on the literature values that MRSCloud relies on — largely Govindaraju et al. with only minor updates (e.g., to GABA) — rather than re-deriving them. Anticipate runtime: complex J-coupled spin systems such as Cystat, HCar, iLe, and Lys significantly increase simulation time, and Glc takes extremely long, so restrict the metabolite list to what the analysis needs. Keep the private parameters in simMRS.json unchanged, since the parameterization is coupled to the vendor (GE/Philips/Siemens/Universal_*) and sequence options (UnEdited/MEGA/HERMES/HERCULES) at 3T.

## Related tools

- **MRSCloud** (MATLAB tool that consumes the spin-system definitions (via simMRS.json) together with vendor RF waveforms to simulate 3T metabolite basis functions using 1D projection, coherence pathway filters, and precalculated propagators) — https://github.com/shui5/MRSCloud
- **FID-A** (Provides the spectral simulation functionality and spin-system framework upon which MRSCloud's metabolite simulations are built) — https://github.com/CIC-methods/FID-A
- **MARSS** (Independent density-matrix simulator used as a cross-validation reference against which MRSCloud's parameterized basis functions are compared (ICCs at least 0.96))
- **LCModel** (Provides an experimentally measured (phantom-acquired) basis set (e.g., press_te30_3t_01a.basis) used as an external comparison target for the simulated basis functions)

## Examples

```
Edit simMRS.json: set metabolites to ["GABA","Gln","Glu","NAA"], vendor "Siemens", mega_or_hadam "MEGA", localization "PRESS", TE 68, editTp 14, spatial_points 101; then run: run_simulations_cloud.m
```

## Evaluation signals

- Intraclass correlation coefficients (ICCs) between MRSCloud and FID-A basis spectra are at least 0.98, and at least 0.96 against MARSS
- ICCs against the experimentally measured LCModel basis set fall in the reported range (lowest Gln 0.68, highest NAA 0.96); values outside this pattern warrant investigation
- Each zipped basis set includes a PDF image of the spectra (plot_basis function integrated from Osprey) that must be visually checked for quality assurance
- Simulation of a full PRESS basis set completes in approximately 11 minutes on the server; dramatic overruns suggest misconfigured spin-system complexity (e.g., Glc, Cystat, HCar, iLe, Lys)
- Hermes/HERCULES TE values are fixed at 68 ms and 80 ms respectively; a successful run reflects these constraints alongside editTp of 14 ms or 20 ms

## Limitations

- Spin-system parameters rely largely on Govindaraju et al. with only minor updates (e.g., GABA), so outdated or coarse J-coupling values propagate into all simulated basis functions
- Cystat, HCar, iLe, and Lys have complex J-coupling spin systems that significantly increase simulation time, and Glc takes extremely long
- Product sequences and RF waveforms are not shared in the GitHub repo, limiting exact reproduction of vendor-specific excitation/refocusing behavior
- STEAM and SPECIAL localization are not implemented; the parameterized spin systems currently serve only PRESS and sLASER localization
- Simulations cannot capture all possible vendor experiments (parameter sets vary with hardware configurations and scanner protocol settings)

## Evidence
