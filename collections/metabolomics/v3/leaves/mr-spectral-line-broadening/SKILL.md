---
name: mr-spectral-line-broadening
description: Use when comparing density-matrix-simulated basis functions (e.g., MRSCloud
  unedited PRESS TE 30 ms output) against an experimentally acquired LCModel phantom
  basis file and linewidth mismatch must be removed before spectral agreement metrics.
when_to_use_negative:
- When both basis sets being compared are simulated (e.g., MRSCloud vs. FID-A or MARSS)
  and no linewidth mismatch from phantom acquisition exists — the article handled
  those comparisons without this broadening step
- When the target basis set was simulated rather than phantom-measured, since the
  vendor-phantom linewidth matching rationale ('measured using phantom rather than
  simulation') does not apply
- When field strength is not 3T, since MRSCloud 'only supports generation of basis
  sets in 3T' and cannot produce the simulated spectra this step needs
tools:
- name: MRSCloud
  role: Generates the simulated metabolite basis spectra that require line broadening
    before comparison with the phantom-acquired basis set
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation and processing toolbox on which MRSCloud is
    built; handles the basis-function spectra being broadened
  repo: https://github.com/CIC-methods/FID-A
- name: LCModel
  role: Source of the phantom-acquired reference basis set (press_te30_3t_01a.basis)
    whose linewidths the simulated spectra must be broadened to match
- name: R
  role: Used in RStudio to compute per-metabolite ICCs between the line-broadened
    simulated and phantom-acquired basis spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:87ff649ca63968f27db27f47966660e1df1d38395b3da626f363192feb36c5c1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - LCModel
  - MRSCloud
  - R
  - RStudio
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mr-spectral-line-broadening

## Summary

Applies 1 Hz FWHM Gaussian line broadening to MRSCloud-simulated MRS basis spectra so their linewidths match a phantom-acquired LCModel basis set (press_te30_3t_01a.basis), enabling a fair per-metabolite ICC comparison.

## When to use

Use when comparing density-matrix-simulated basis functions (e.g., MRSCloud unedited PRESS TE 30 ms output) against an experimentally acquired LCModel phantom basis file and linewidth mismatch must be removed before spectral agreement metrics.

## When NOT to use

- When both basis sets being compared are simulated (e.g., MRSCloud vs. FID-A or MARSS) and no linewidth mismatch from phantom acquisition exists — the article handled those comparisons without this broadening step
- When the target basis set was simulated rather than phantom-measured, since the vendor-phantom linewidth matching rationale ('measured using phantom rather than simulation') does not apply
- When field strength is not 3T, since MRSCloud 'only supports generation of basis sets in 3T' and cannot produce the simulated spectra this step needs

## Inputs

- MRSCloud-simulated unedited PRESS TE 30 ms metabolite basis spectra (from run_simulations_cloud.m with spatial_points >= 41)
- Phantom-acquired LCModel basis set file press_te30_3t_01a.basis (downloaded from http://s-provencher.com/lcm-basis.shtml)

## Outputs

- 1 Hz FWHM Gaussian line-broadened simulated metabolite basis spectra matched in linewidth to the LCModel phantom basis
- Per-metabolite ICC values written to mrscloud_lcmodel_icc.csv

## How to apply

First generate the simulated basis set with MRSCloud by running run_simulations_cloud.m after configuring simMRS.json (3T field strength, UnEdited PRESS, TE 30 ms, spatial_points 41 or higher since 41 is acceptable and 101 is ideal). Obtain the phantom-acquired LCModel basis set press_te30_3t_01a.basis from s-provencher.com/lcm-basis.shtml, which is measured on phantom rather than simulated. Apply 1 Hz FWHM Gaussian line broadening to each simulated metabolite function so linewidths match the phantom-acquired basis functions — this is the rationale for the step, since the measured waveforms and timings are undocumented and would otherwise differ from the simulation. Match each metabolite basis function between the simulated and measured sets by name, then compute per-metabolite intraclass correlation coefficients (ICCs) between the broadened MRSCloud spectra and the LCModel basis spectra using R in RStudio. Correct application is validated if ICCs reproduce the reported pattern: lowest for Gln at 0.68 and highest for NAA at 0.96.

## Related tools

- **MRSCloud** (Generates the simulated metabolite basis spectra that require line broadening before comparison with the phantom-acquired basis set) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation and processing toolbox on which MRSCloud is built; handles the basis-function spectra being broadened) — https://github.com/CIC-methods/FID-A
- **LCModel** (Source of the phantom-acquired reference basis set (press_te30_3t_01a.basis) whose linewidths the simulated spectra must be broadened to match)
- **R** (Used in RStudio to compute per-metabolite ICCs between the line-broadened simulated and phantom-acquired basis spectra)

## Evaluation signals

- Reproduced per-metabolite ICCs match the reported pattern: lowest ICC for Gln at 0.68 and highest for NAA at 0.96
- ICCs between MRSCloud and reference sets fall in a reasonable-agreement range (>= 0.96 was seen against FID-A/MARSS; >= 0.68 against the phantom-measured LCModel set)
- After broadening, the simulated basis functions have ~1 Hz FWHM linewidths matching the phantom-acquired LCModel basis functions
- Every metabolite in the simulated set is matched by name to a corresponding LCModel basis function before ICC computation
- Per-metabolite ICC values are present for all metabolites in the output CSV mrscloud_lcmodel_icc.csv

## Limitations

- Waveforms and timings for the LCModel vendor-provided (phantom-measured) basis set are not well documented and likely differ from the MRSCloud simulation, so linewidth matching cannot fully reconcile acquisition differences
- Basis set comparison between MRSCloud and LCModel is challenging because limited information has been provided for the acquisition of the basis sets
- The existing version of MRSCloud only supports generation of basis sets in 3T, restricting the simulated spectra this step can be applied to
- Lower spatial_points (41 vs. 101) trades spatial-resolution fidelity for simulation time, which can affect the simulated lineshapes being broadened

## Evidence

- [discussion] LCModel phantom rather than simulation: [evidence span withheld — anchor status: not_found]
