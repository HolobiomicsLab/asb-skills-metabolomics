---
name: mrs-metabolite-spectra-pairwise-comparison
description: Use when two sets of simulated MRS metabolite basis functions (e.g.,
  from MRSCloud and FID-A under PRESS at TE 30/68 ms) must be validated for agreement
  per metabolite using ICC and NRMS thresholds.
when_to_use_negative:
- When comparing simulated basis spectra against the experimentally measured (phantom-acquired)
  LCModel vendor basis set (e.g., press_te30_3t_01a.basis) — agreement is much weaker
  (lowest ICC 0.68 for Gln) because waveforms and timings of the measured basis set
  are not well documented
- When the goal is to simulate localization methods MRSCloud does not support, such
  as STEAM or SPECIAL
- When simulating complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) where
  runtime makes pairwise re-simulation impractical
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: MRSCloud
  role: Accelerated MATLAB-based simulator producing the first set of metabolite basis
    functions via 1D projection method, coherence pathway filters, and pre-calculated
    propagators
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Native (release-version) spectral simulation toolbox producing the reference
    basis functions for pairwise comparison
  repo: https://github.com/CIC-methods/FID-A
- name: MATLAB
  role: Environment in which run_simulations_cloud.m is executed for both simulation
    code paths
- name: R
  role: Language used to compute per-metabolite intraclass correlation coefficients
- name: RStudio
  role: IDE in which the statistical comparison (ICC analysis) is performed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:a85785bf7c5d139feb6d441c48b298e9ab31d71137cd8bb2f7984a0d667a7ee2
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mrs-metabolite-spectra-pairwise-comparison

## Summary

Quantitatively compare metabolite basis functions simulated by two MRS simulation implementations (e.g., MRSCloud vs. native FID-A) by pairing metabolite spectra and computing intraclass correlation coefficients (ICC) and normalized root-mean-square (NRMS) differences per metabolite in R/RStudio.

## When to use

Use when two sets of simulated MRS metabolite basis functions (e.g., from MRSCloud and FID-A under PRESS at TE 30/68 ms) must be validated for agreement per metabolite using ICC and NRMS thresholds.

## When NOT to use

- When comparing simulated basis spectra against the experimentally measured (phantom-acquired) LCModel vendor basis set (e.g., press_te30_3t_01a.basis) — agreement is much weaker (lowest ICC 0.68 for Gln) because waveforms and timings of the measured basis set are not well documented
- When the goal is to simulate localization methods MRSCloud does not support, such as STEAM or SPECIAL
- When simulating complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) where runtime makes pairwise re-simulation impractical

## Inputs

- MRSCloud-simulated metabolite basis functions (e.g., PRESS TE 30/68 ms, 41x41 spatial points, 3T)
- FID-A (release version) simulated metabolite basis functions under identical acquisition parameters
- simMRS. configuration specifying metabolites, vendor (GE/Philips/Siemens), localization (PRESS), TE, and spatial_points

## Outputs

- icc_nrms_comparison.csv with per-metabolite ICC and normalized RMS difference values
- Paired per-metabolite comparison verdicts (ICC >= 0.98, NRMS < 1%)

## How to apply

Run density-matrix simulations for the 8 commonly used metabolites with both code paths (MRSCloud via run_simulations_cloud.m and the native FID-A release) under matched PRESS parameters (TE 30 ms and 68 ms, spatial_points = 41; 101 is ideal but slower). Load the simulated basis functions from both implementations and pair the metabolite spectra one-to-one. Compute intraclass correlation coefficients (ICC) per metabolite using R in RStudio, then compute normalized root-mean-square differences (NRMS) between the paired spectra. Write per-metabolite ICC and NRMS values to icc_nrms_comparison.csv and judge success by ICC >= 0.98 and NRMS < 1% for all 8 metabolites (the article reports MRSCloud-vs-FID-A ICCs of at least 0.98 and NRMS under 1%). Avoid complex J-coupled spin systems such as Cystat, HCar, iLe, Lys, and Glc, which dramatically increase simulation runtime; a full PRESS basis set takes approximately 11 minutes on the server.

## Related tools

- **MRSCloud** (Accelerated MATLAB-based simulator producing the first set of metabolite basis functions via 1D projection method, coherence pathway filters, and pre-calculated propagators) — https://github.com/shui5/MRSCloud
- **FID-A** (Native (release-version) spectral simulation toolbox producing the reference basis functions for pairwise comparison) — https://github.com/CIC-methods/FID-A
- **MATLAB** (Environment in which run_simulations_cloud.m is executed for both simulation code paths)
- **R** (Language used to compute per-metabolite intraclass correlation coefficients)
- **RStudio** (IDE in which the statistical comparison (ICC analysis) is performed)

## Examples

```
Run run_simulations_cloud.m in MATLAB (simMRS.: metabolites=["NAA","Cr",...], localization="PRESS", TE=30, spatial_points=41), then in R: icc_nrms <- compare_basis(mrscloud_dir, fida_dir); write.csv(icc_nrms, "icc_nrms_comparison.csv")
```

## Evaluation signals

- All 8 per-metabolite ICCs between the two simulation implementations are at least 0.98
- All normalized root-mean-square differences between paired simulated basis spectra are below 1%
- icc_nrms_comparison.csv contains ICC and NRMS rows for exactly the 8 tested metabolites at both TE 30 ms and TE 68 ms
- Simulation parameters match across code paths (PRESS localization, same TE, spatial_points = 41) so that differences reflect implementation, not acquisition settings

## Limitations

- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) significantly or extremely increase simulation time and should be avoided to limit runtime
- MRSCloud is tested only on brain MRS data from 3T scanners and supports only 3T basis set generation; product sequences and RF waveforms are not shared in the GitHub repo
- A full PRESS basis set takes approximately 11 minutes on the server, and spatial_points = 101 (ideal) takes much longer than 41
- Spin-system parameters rely largely on Govindaraju et al. with only minor updates, so agreement with simulated reference sets does not guarantee physiological accuracy

## Evidence
