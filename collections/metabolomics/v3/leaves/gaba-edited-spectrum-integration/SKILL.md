---
name: gaba-edited-spectrum-integration
description: Use when you have edited GABA difference spectra simulated at multiple
  spatial resolutions (e.g. 21×21, 41×41, 101×101 spatial points) and need to quantify
  GABA signal amplitude at 3 ppm per setting to test resolution convergence.
when_to_use_negative:
- Input is unedited (OFF-only) MRS basis-set output rather than edited GABA difference
  spectra — the 3-ppm edited GABA integral is only meaningful for edit-on/edit-off
  difference signals.
- Comparing MRSCloud against FID-A, MARSS, or LCModel phantom-measured basis sets
  — that is a cross-tool basis-set comparison task judged by ICCs on metabolite basis
  functions (e.g. Gln 0.68 to NAA 0.96), not a spatial-resolution sensitivity analysis.
- Simulating metabolites other than edited GABA or localization schemes not implemented
  in MRSCloud (e.g. STEAM or SPECIAL).
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3384
- http://edamontology.org/topic_3420
tools:
- name: MRSCloud
  role: Generates the edited GABA basis sets/difference spectra at each spatial-point
    setting via run_simulations_cloud.m
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation toolbox on which MRSCloud is built; provides
    the simulated signals from which the 3-ppm GABA integral is extracted
  repo: https://github.com/CIC-methods/FID-A
- name: R
  role: Used in RStudio to compute intraclass correlation coefficients among the 21,
    41 and 101 spatial-point resolutions
- name: RStudio
  role: Environment for the statistical analysis of integrals and ICC computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:d9eb49986cfda6ba96a4855146d8f8b5277bc3451775c7556464e8254062647c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MRSCloud
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3384
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gaba-edited-spectrum-integration

## Summary

Extracts edited GABA difference spectra from MRSCloud/FID-A simulation runs (MEGA-PRESS, HERMES, HERCULES with PRESS or sLASER at TE 80 ms) and integrates the GABA signal at 3 ppm for each spatial-point setting, enabling cross-resolution agreement analysis via ICCs.

## When to use

Use when you have edited GABA difference spectra simulated at multiple spatial resolutions (e.g. 21×21, 41×41, 101×101 spatial points) and need to quantify GABA signal amplitude at 3 ppm per setting to test resolution convergence.

## When NOT to use

- Input is unedited (OFF-only) MRS basis-set output rather than edited GABA difference spectra — the 3-ppm edited GABA integral is only meaningful for edit-on/edit-off difference signals.
- Comparing MRSCloud against FID-A, MARSS, or LCModel phantom-measured basis sets — that is a cross-tool basis-set comparison task judged by ICCs on metabolite basis functions (e.g. Gln 0.68 to NAA 0.96), not a spatial-resolution sensitivity analysis.
- Simulating metabolites other than edited GABA or localization schemes not implemented in MRSCloud (e.g. STEAM or SPECIAL).

## Inputs

- Edited GABA difference spectra from MRSCloud/FID-A simulation runs at spatial_points = 21, 41 and 101 (MEGA, HERMES, HERCULES for PRESS and sLASER at TE 80 ms)
- MRSCloud simulation output directories generated via run_simulations_cloud.m with editTp = 14 ms (MEGA/HERMES) or 20 ms (HERCULES)

## Outputs

- GABA signal integrals at 3 ppm per spatial-point setting and per sequence/localization combination
- gaba_resolution_icc.csv containing 3-ppm GABA integrals and ICC values among 21, 41 and 101 spatial-point resolutions

## How to apply

After running run_simulations_cloud.m separately for spatial_points = 21, 41 and 101 (keeping editTp = 14 ms for MEGA-PRESS/HERMES and 20 ms for HERCULES, TE 80 ms, localization PRESS or sLASER, and private parameters unchanged), extract the edited GABA difference spectra (edit-on minus edit-off) from each run. Integrate the GABA signal at 3 ppm within each difference spectrum, producing one integral value per sequence/localization/spatial-point combination. Compile these integrals together with intraclass correlation coefficients (computed in R/RStudio) among the 21, 41 and 101 spatial-point resolutions into a summary table such as gaba_resolution_icc.csv. The rationale is that integrals should be stable across resolutions — slightly larger at higher spatial-point counts — with ICCs exceeding 0.99 confirming that 41 spatial points is acceptable and 101 is ideal for edited GABA simulations.

## Related tools

- **MRSCloud** (Generates the edited GABA basis sets/difference spectra at each spatial-point setting via run_simulations_cloud.m) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation toolbox on which MRSCloud is built; provides the simulated signals from which the 3-ppm GABA integral is extracted) — https://github.com/CIC-methods/FID-A
- **R** (Used in RStudio to compute intraclass correlation coefficients among the 21, 41 and 101 spatial-point resolutions)
- **RStudio** (Environment for the statistical analysis of integrals and ICC computation)

## Examples

```
for sp in 21 41 101; do modify simMRS.json spatial_points=$sp editTp=14; matlab -run run_simulations_cloud.m; done; Rscript compute_icc.R --integrals gaba_resolution_icc.csv
```

## Evaluation signals

- ICCs among the 21, 41 and 101 spatial-point resolutions of edited GABA signals exceed 0.99 for all three edited sequences (MEGA, HERMES, HERCULES) in both PRESS and sLASER localization.
- GABA integrals at 3 ppm are slightly larger for higher spatial-point counts, with 101 points yielding the reference values.
- Per the MRSCloud documentation, spatial_points = 41 is acceptable and 101 is ideal, so integrals at 41 should deviate only marginally from those at 101.
- Simulations must keep the private parameters unchanged and editTp fixed at 14 ms (MEGA-PRESS/HERMES) or 20 ms (HERCULES) — deviations invalidate cross-resolution comparability.
- Compiled output (gaba_resolution_icc.csv) contains integrals and ICCs for all six sequence/localization combinations (MEGA/HERMES/HERCULES × PRESS/sLASER).

## Limitations

- Higher spatial-point settings substantially increase simulation time, so 101 points may be impractical for exploratory runs.
- The exiting version of MRSCloud only supports generation of basis sets in 3T, so this analysis does not generalize to other field strengths without validation.
- Product sequences and RF waveforms are not shared in the GitHub repo, limiting exact reproduction of vendor-specific edited sequences.
- Simulations cannot capture all differences between the different disseminated versions of MEGA-PRESS within each vendor.

## Evidence

- [results] ICCs among the different spatial resolutions (21×21, 41×41, 101×101) of edited GABA signals were higher than 0.99 for all three edited sequences: [evidence span withheld — anchor status: not_found]
