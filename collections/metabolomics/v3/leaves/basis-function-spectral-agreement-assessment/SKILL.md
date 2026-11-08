---
name: basis-function-spectral-agreement-assessment
description: Use when a newly simulated metabolite basis set (e.g. from 1D-projection
  density-matrix simulation) must be checked against a ground-truth FID-A reference
  or vendor/measured basis set before accepting it for LCModel-style analysis.
when_to_use_negative:
- When the goal is spectral quantification of in vivo data rather than validating
  a simulated basis set — use a full LCM pipeline instead.
- 'When strict pass/fail agreement against the LCModel phantom-measured basis set
  is required: limited acquisition information and undocumented waveforms/timings
  make such ICCs interpretive, not decisive (Gln reached only 0.68).'
- When the metabolite has a very complex spin system (e.g. Glc, Cystat, HCar, iLe,
  Lys) and simulation time, not accuracy, is the constraint — the ICC check does not
  address runtime of the reference itself.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3418
tools:
- name: MRSCloud
  role: Generates the accelerated (1D projection, coherence-pathway-filtered) simulated
    metabolite basis set to be validated
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Provides the reference spectral-simulation functionality against which MRSCloud
    basis functions are compared via ICC
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Generic simulator used as an independent reference for ICC comparison of basis
    functions
- name: LCModel
  role: Source of the experimentally measured (phantom) basis set used for plausibility
    comparison
- name: R / RStudio
  role: Statistical analysis platform for computing intraclass correlation coefficients
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:93c24582c5b49c7a887cacc5babe0a46f9ccf9d2d60dfb1d5e8cafda7b50dbac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - LCModel
  - MARSS
  - MRSCloud
  - R / RStudio
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# basis-function-spectral-agreement-assessment

## Summary

Validates an accelerated MRS basis-set simulation (e.g. MRSCloud's 1D projection method) by comparing its voxel-averaged metabolite basis spectra against a reference simulation or measured basis set using intraclass correlation coefficients (ICCs).

## When to use

Use when a newly simulated metabolite basis set (e.g. from 1D-projection density-matrix simulation) must be checked against a ground-truth FID-A reference or vendor/measured basis set before accepting it for LCModel-style analysis.

## When NOT to use

- When the goal is spectral quantification of in vivo data rather than validating a simulated basis set — use a full LCM pipeline instead.
- When strict pass/fail agreement against the LCModel phantom-measured basis set is required: limited acquisition information and undocumented waveforms/timings make such ICCs interpretive, not decisive (Gln reached only 0.68).
- When the metabolite has a very complex spin system (e.g. Glc, Cystat, HCar, iLe, Lys) and simulation time, not accuracy, is the constraint — the ICC check does not address runtime of the reference itself.

## Inputs

- Simulated voxel-averaged basis-function FID/spectrum from MRSCloud (run_simulations_cloud.m output, configured via simMRS.json)
- Reference basis-function spectrum from full FID-A density-matrix simulation (CIC-methods/FID-A)
- Alternative reference: MARSS-generated basis functions
- Optional: vendor-provided measured basis set, e.g. press_te30_3t_01a.basis from LCModel

## Outputs

- Per-metabolite ICC values comparing simulated vs reference basis spectra (e.g. written to 1d_projection_validation.csv)
- Accept/reject decision for the accelerated basis function at the ICC >= 0.98 threshold
- Runtime comparison between accelerated and reference simulations

## How to apply

Simulate the metabolite basis function with the accelerated method (1D projection with ideal-excitation approximation, coherence order filtering, and pre-calculated propagators in MRSCloud, built on FID-A), then simulate or obtain a reference basis function (full spatial-loop FID-A propagation, MARSS, or the LCModel phantom-measured press_te30_3t_01a.basis). Compute the intraclass correlation coefficient (ICC) between the two spectra per metabolite; accept the accelerated basis set when ICC is at least 0.98 against FID-A (MRSCloud achieved ICCs of at least 0.98 vs FID-A and at least 0.96 vs MARSS). For comparisons against the experimentally measured LCModel basis set, treat ICC as a plausibility check rather than a pass/fail gate, since waveforms and timings of that basis set are not well documented (Gln 0.68 to NAA 0.96 in the paper). Use spatial_points = 101 (41 acceptable) and editTp = 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES when configuring the simulation.

## Related tools

- **MRSCloud** (Generates the accelerated (1D projection, coherence-pathway-filtered) simulated metabolite basis set to be validated) — https://github.com/shui5/MRSCloud
- **FID-A** (Provides the reference spectral-simulation functionality against which MRSCloud basis functions are compared via ICC) — https://github.com/CIC-methods/FID-A
- **MARSS** (Generic simulator used as an independent reference for ICC comparison of basis functions)
- **LCModel** (Source of the experimentally measured (phantom) basis set used for plausibility comparison)
- **R / RStudio** (Statistical analysis platform for computing intraclass correlation coefficients)

## Examples

```
edit simMRS.json (spatial_points=101, editTp=14); matlab -batch "run_simulations_cloud.m"  # then compute ICC vs FID-A reference in R and write 1d_projection_validation.csv
```

## Evaluation signals

- ICC between MRSCloud-simulated and FID-A reference basis spectra is at least 0.98 for the metabolite under test
- ICC against MARSS reference basis spectra is at least 0.96
- Simulated full PRESS basis set completes in roughly 11 minutes on the server (runtime sanity check for the accelerated pipeline)
- Output CSV (e.g. 1d_projection_validation.csv) contains per-metabolite ICC and runtime rows and the acceleration is accepted only when ICC >= 0.98
- Downloaded MRSCloud basis set includes the PDF spectra plot (from Osprey's plot_basis_function) for quality assurance inspection

## Limitations

- Spin-system parameters rely largely on Govindaraju et al. with only minor updates (e.g. GABA), bounding achievable ICC against measured spectra
- The LCModel phantom-measured basis set (e.g. press_te30_3t_01a.basis) has undocumented waveforms and timings, so low ICCs vs MRSCloud (Gln 0.68) do not necessarily indicate simulation error
- MRSCloud's 3T-only implementation and limited vendor parameter sets mean agreement may not transfer to other field strengths or vendor-specific MEGA-PRESS variants
- Cystat, HCar, iLe, Lys, and Glc have complex J-coupling spin systems that drastically increase simulation time for both accelerated and reference runs

## Evidence

- [abstract] ICCs at least 0.98 vs FID-A threshold: "ICCs were at least 0.98 between MRSCloud and FID-A and were at least 0.96 between MRSCloud and MARSS"
- [abstract] LCModel comparison ICC range: "ICCs between simulated MRSCloud basis spectra and acquired LCModel basis spectra were lowest for Gln at 0.68 and highest for NAA at 0.96"
- [discussion] 1D projection acceleration description: "The 1D projection method accelerates the process by decreasing the number of simulations to the factor equivalent to the number of spatial points."
- [readme] spatial_points parameter guidance: "spatial_points: 41 is acceptable and 101 is ideal. The higher the number of spatial points, the longer it takes for the simulation."
- [results] PDF QA output: "Each zipped basis set downloaded online from MRSCloud also includes a PDF image of the spectra, generated using the plot basis function integrated from Osprey (19), for quality assurance"
