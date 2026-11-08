---
name: spectral-basis-set-comparison
description: Use when you have a simulated MRS basis set and a phantom-acquired or
  vendor-provided basis set for the same sequence/field (e.g., 3T PRESS TE 30 ms)
  and need per-metabolite agreement metrics such as ICCs.
when_to_use_negative:
- When comparing two simulated basis sets (e.g., MRSCloud vs FID-A or MARSS) where
  no line broadening to match a phantom-measured set is needed — the article handles
  those as separate comparisons with ICCs >= 0.98 and >= 0.96 respectively
- When the vendor/sequence parameters do not match — the existing MRSCloud version
  only supports 3T basis sets, and the comparison assumes unedited PRESS TE 30 ms
- When you need a rigorous validation of simulated vs measured basis sets — the article
  notes the comparison is challenging because limited information is available about
  the LCModel basis set acquisition and its waveforms/timings likely differ from the
  simulation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MRSCloud
  role: Generates the simulated unedited PRESS TE 30 ms metabolite basis spectra being
    compared against the LCModel phantom-acquired basis set
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation functionality on which MRSCloud is built; used
    for handling basis spectra
  repo: https://github.com/CIC-methods/FID-A
- name: LCModel
  role: Source of the experimentally phantom-acquired basis set (press_te30_3t_01a.basis)
    used as the comparison reference
- name: R
  role: Statistical analysis environment used to compute per-metabolite ICCs between
    simulated and acquired basis spectra
- name: RStudio
  role: IDE in which the R-based ICC statistical analysis is performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:38240e183aa4a0344e5c072ea1a819e037842313e6b69d67eb2c676ab7901816
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-basis-set-comparison

## Summary

Quantifies agreement between density-matrix-simulated MRS metabolite basis functions (e.g., MRSCloud unedited PRESS TE 30 ms at 3T) and an experimentally acquired basis set (LCModel phantom-measured press_te30_3t_01a.basis) by computing per-metabolite intraclass correlation coefficients in R/RStudio.

## When to use

Use when you have a simulated MRS basis set and a phantom-acquired or vendor-provided basis set for the same sequence/field (e.g., 3T PRESS TE 30 ms) and need per-metabolite agreement metrics such as ICCs.

## When NOT to use

- When comparing two simulated basis sets (e.g., MRSCloud vs FID-A or MARSS) where no line broadening to match a phantom-measured set is needed — the article handles those as separate comparisons with ICCs >= 0.98 and >= 0.96 respectively
- When the vendor/sequence parameters do not match — the existing MRSCloud version only supports 3T basis sets, and the comparison assumes unedited PRESS TE 30 ms
- When you need a rigorous validation of simulated vs measured basis sets — the article notes the comparison is challenging because limited information is available about the LCModel basis set acquisition and its waveforms/timings likely differ from the simulation

## Inputs

- MRSCloud-simulated unedited PRESS TE 30 ms basis spectra (from run_simulations_cloud.m with spatial_points = 41 or higher)
- LCModel phantom-acquired basis set file press_te30_3t_01a.basis (from http://s-provencher.com/lcm-basis.shtml)

## Outputs

- Per-metabolite ICC values between simulated and acquired basis spectra (e.g., Gln ICC = 0.68, NAA ICC = 0.96)
- CSV file of per-metabolite ICCs (mrscloud_lcmodel_icc.csv)

## How to apply

Obtain the simulated basis spectra (MRSCloud run_simulations_cloud.m with spatial_points set to 41 or higher; 101 is ideal) and download the LCModel phantom-acquired basis set press_te30_3t_01a.basis from s-provencher.com/lcm-basis.shtml. Apply 1 Hz FWHM Gaussian line broadening to the simulated spectra so their linewidths match the phantom-acquired basis functions. Align each metabolite basis function between the two sets, matching metabolites by name. Compute per-metabolite ICCs between simulated and measured spectra using R in RStudio, and write the results to a CSV (mrscloud_lcmodel_icc.csv). The comparison is accepted as reproduced when the ICC pattern matches the reported result: lowest ICC for Gln at 0.68 and highest for NAA at 0.96, corresponding to the largest and smallest root-mean-square values respectively.

## Related tools

- **MRSCloud** (Generates the simulated unedited PRESS TE 30 ms metabolite basis spectra being compared against the LCModel phantom-acquired basis set) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation functionality on which MRSCloud is built; used for handling basis spectra) — https://github.com/CIC-methods/FID-A
- **LCModel** (Source of the experimentally phantom-acquired basis set (press_te30_3t_01a.basis) used as the comparison reference)
- **R** (Statistical analysis environment used to compute per-metabolite ICCs between simulated and acquired basis spectra)
- **RStudio** (IDE in which the R-based ICC statistical analysis is performed)

## Examples

```
Rscript compute_icc.R --sim mrscloud_press_te30_sim/ --ref press_te30_3t_01a.basis --out mrscloud_lcmodel_icc.csv
```

## Evaluation signals

- Reproduced ICCs match the reported pattern: lowest ICC for Gln at 0.68 and highest for NAA at 0.96
- The highest/lowest ICCs correspond to the smallest/largest root-mean-square values, as reported in the article
- Output file mrscloud_lcmodel_icc.csv contains one ICC per metabolite, matched by metabolite name between the two basis sets
- Simulated spectra linewidths match the phantom-acquired basis functions after applying 1 Hz FWHM Gaussian line broadening
- Simulated basis set was generated with spatial_points >= 41 (101 ideal) so spectral shapes are spatially converged before comparison

## Limitations

- Waveforms and timings of the LCModel vendor-provided (phantom-measured) basis set are not well documented, and likely differ from the MRSCloud simulation
- Basis set comparison between MRSCloud and LCModel is challenging because limited information has been provided for the acquisition of the LCModel basis sets
- The existing MRSCloud version only supports generation of basis sets at 3T, restricting this comparison to 3T PRESS data
- Spin-system parameters underlying the simulation rely largely on the work of Govindaraju et al. with only minor updates, so simulated spectra may not fully reflect measured metabolite behavior

## Evidence
