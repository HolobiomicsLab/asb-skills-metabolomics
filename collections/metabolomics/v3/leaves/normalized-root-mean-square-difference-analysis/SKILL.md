---
name: normalized-root-mean-square-difference-analysis
description: Use when two implementations of the same spectral simulation (or two
  paired sets of metabolite basis functions) must be numerically compared per metabolite
  to demonstrate equivalence, e.g., validating an accelerated simulator against a
  reference codebase.
when_to_use_negative:
- When comparing simulated spectra against experimentally acquired (phantom-measured)
  vendor basis sets such as the LCModel 'press_te30_3t_01a.basis' — waveforms and
  timings are not well documented and the comparison is confounded (e.g., Gln ICC
  dropped to 0.68).
- When the metabolites have complex J-coupled spin systems (Cystat, HCar, iLe, Lys,
  Glc) that make full PRESS simulation prohibitively slow to generate the paired inputs
  needed for the comparison.
- When only one simulation output exists with no paired reference basis functions
  to diff against.
tools:
- name: MRSCloud
  role: 'Test implementation: accelerated basis set simulation (1D projection method,
    coherence pathway filters, pre-calculated propagators) whose output is compared
    against FID-A.'
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: 'Reference implementation: official release-version simulations that provide
    the paired basis functions for NRMS/ICC comparison.'
  repo: https://github.com/CIC-methods/FID-A
- name: MATLAB
  role: Runs both simulation code paths (run_simulations_cloud.m and FID-A) to generate
    the paired metabolite spectra.
- name: R
  role: Statistical environment used to compute intraclass correlation coefficients
    between paired basis spectra.
- name: RStudio
  role: Interface in which the R statistical analysis of ICC (and supporting comparison)
    is performed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:90be5245579dd6a920b55f7f6a44aba33dd0d9bd09efa77177d9f5a6b138655e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MATLAB
  - MRSCloud
  - R
  - RStudio
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# normalized-root-mean-square-difference-analysis

## Summary

Quantify agreement between paired simulated metabolite basis spectra (e.g., MRSCloud vs. native FID-A PRESS simulations at TE 30/68 ms) by computing normalized root-mean-square differences (NRMS) per metabolite, complemented by ICCs. In the MRSCloud validation, NRMS under 1% together with ICC >= 0.98 established equivalence of the accelerated simulation code with the release-version FID-A output.

## When to use

Use when two implementations of the same spectral simulation (or two paired sets of metabolite basis functions) must be numerically compared per metabolite to demonstrate equivalence, e.g., validating an accelerated simulator against a reference codebase.

## When NOT to use

- When comparing simulated spectra against experimentally acquired (phantom-measured) vendor basis sets such as the LCModel 'press_te30_3t_01a.basis' — waveforms and timings are not well documented and the comparison is confounded (e.g., Gln ICC dropped to 0.68).
- When the metabolites have complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) that make full PRESS simulation prohibitively slow to generate the paired inputs needed for the comparison.
- When only one simulation output exists with no paired reference basis functions to diff against.

## Inputs

- MRSCloud-simulated metabolite basis functions (MATLAB output of run_simulations_cloud.m, PRESS TE 30 ms and TE 68 ms, spatial_points = 41)
- Native FID-A release-simulated metabolite basis functions for the same 8 metabolites and identical PRESS parameters
- Metabolite list for the common healthy-brain set (e.g., Asc, Asp, Cr, GABA, GPC, GSH, Gln, Glu, Gly, Lac, mI, NAA, NAAG, PCh, PCr, PE, Ser, sI, Tau subset of 8 used in the study)

## Outputs

- Per-metabolite normalized root-mean-square difference (NRMS) values between paired simulated basis spectra
- Per-metabolite ICC values computed in R/RStudio
- Combined comparison table icc_nrms_comparison.csv with ICC and NRMS per metabolite

## How to apply

First generate the paired basis functions: run density-matrix simulations for the 8 common metabolites with both code paths (MRSCloud's run_simulations_cloud.m and the official FID-A release) using identical acquisition parameters — PRESS localization at TE 30 ms and TE 68 ms with spatial_points = 41 (acceptable; 101 is ideal), keeping private parameters unchanged. Pair each metabolite's simulated spectra from the two implementations and compute the normalized root-mean-square difference (NRMS) between them per metabolite; in parallel compute intraclass correlation coefficients (ICC) between the paired basis spectra using R in RStudio. Judge success against the study's acceptance thresholds: NRMS below 1% and ICC at least 0.98 for all 8 metabolites indicates the accelerated simulation reproduces the FID-A reference. Write per-metabolite ICC and NRMS values to a comparison table (icc_nrms_comparison.csv) for review.

## Related tools

- **MRSCloud** (Test implementation: accelerated basis set simulation (1D projection method, coherence pathway filters, pre-calculated propagators) whose output is compared against FID-A.) — https://github.com/shui5/MRSCloud
- **FID-A** (Reference implementation: official release-version simulations that provide the paired basis functions for NRMS/ICC comparison.) — https://github.com/CIC-methods/FID-A
- **MATLAB** (Runs both simulation code paths (run_simulations_cloud.m and FID-A) to generate the paired metabolite spectra.)
- **R** (Statistical environment used to compute intraclass correlation coefficients between paired basis spectra.)
- **RStudio** (Interface in which the R statistical analysis of ICC (and supporting comparison) is performed.)

## Examples

```
run_simulations_cloud  % MATLAB: PRESS TE 30/68 ms, spatial_points=41, MRSCloud vs FID-A pairing
```

## Evaluation signals

- All 8 metabolites show normalized root-mean-square differences under 1% between the MRSCloud- and FID-A-simulated basis spectra.
- All per-metabolite ICCs are at least 0.98 for the MRSCloud vs. FID-A comparison.
- Both code paths were run with identical parameters (PRESS, TE 30 ms and TE 68 ms, spatial_points = 41, private parameters unchanged).
- Per-metabolite ICC and NRMS values are written to icc_nrms_comparison.csv with all 8 metabolites represented.
- Complex J-coupled metabolites (Cystat, HCar, iLe, Lys, Glc) are excluded from the comparison to keep runtime tractable.

## Limitations

- MRSCloud only supports 3T basis set generation, so the comparison framework is limited to 3T PRESS simulations as tested.
- Higher spatial_points (101 is ideal vs. 41 acceptable) increases simulation time; the 41-point setting trades spatial resolution for speed.
- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time and are impractical to include in the paired comparison.
- Comparison against experimentally measured LCModel basis sets is weaker (ICC lowest for Gln at 0.68, highest for NAA at 0.96) because waveforms and timings of the phantom-measured basis set are not documented.

## Evidence
