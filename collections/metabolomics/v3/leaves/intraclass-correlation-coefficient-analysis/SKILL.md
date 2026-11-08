---
name: intraclass-correlation-coefficient-analysis
description: Use when two or more simulated or experimentally acquired MRS basis sets
  (e.g. MRSCloud vs FID-A, MRSCloud vs MARSS, MRSCloud vs LCModel .basis, or 21/41/101
  spatial-point runs) must be compared metabolite-by-metabolite for agreement.
when_to_use_negative:
- The inputs are raw in-vivo MRS data to be quantified with LCModel analysis rather
  than basis sets to be cross-validated — this skill compares simulated/acquired basis
  functions, not fits.
- The basis sets use localization or vendor settings outside MRSCloud's scope (e.g.
  STEAM or SPECIAL), since no comparable simulated basis functions exist to correlate.
- Waveforms and timings of the comparison basis set are undocumented (as with the
  LCModel vendor-provided basis set), so ICC differences cannot be attributed to simulation
  accuracy rather than acquisition differences.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Statistical environment used to compute per-metabolite ICCs and normalized
    root-mean-square differences between paired basis spectra
- name: RStudio
  role: IDE in which the R statistical analysis of ICCs and NRMS is performed
- name: MRSCloud
  role: Generates the simulated metabolite basis functions (one comparison arm) via
    run_simulations_cloud.m
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Release-version simulation code producing the reference basis functions compared
    against MRSCloud via ICC/NRMS
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Source of the independently generated Philips PRESS TE 30 ms basis set compared
    to MRSCloud using ICCs and NRMS
- name: LCModel
  role: Provides the phantom-acquired basis set (press_te30_3t_01a.basis) whose per-metabolite
    ICCs against MRSCloud simulations are computed
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:0ca4724c1ac5fc4dcd5c3844dbf1c279f2ad2dbe4b91b59a1325352dd7fcf3b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - LCModel
  - MARSS
  - MATLAB
  - MRSCloud
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# intraclass-correlation-coefficient-analysis

## Summary

A statistical validation skill that quantifies agreement between pairs or sets of MRS metabolite basis functions by computing per-metabolite intraclass correlation coefficients (ICCs) — and optionally normalized root-mean-square differences (NRMS) — in R/RStudio.

## When to use

Use when two or more simulated or experimentally acquired MRS basis sets (e.g. MRSCloud vs FID-A, MRSCloud vs MARSS, MRSCloud vs LCModel .basis, or 21/41/101 spatial-point runs) must be compared metabolite-by-metabolite for agreement.

## When NOT to use

- The inputs are raw in-vivo MRS data to be quantified with LCModel analysis rather than basis sets to be cross-validated — this skill compares simulated/acquired basis functions, not fits.
- The basis sets use localization or vendor settings outside MRSCloud's scope (e.g. STEAM or SPECIAL), since no comparable simulated basis functions exist to correlate.
- Waveforms and timings of the comparison basis set are undocumented (as with the LCModel vendor-provided basis set), so ICC differences cannot be attributed to simulation accuracy rather than acquisition differences.

## Inputs

- MRSCloud-simulated metabolite basis spectra (generated via run_simulations_cloud.m, e.g. unedited Philips PRESS TE 30 ms or PRESS TE 30/68 ms at 41x41 spatial points)
- Release-version FID-A simulated metabolite basis functions (CIC-methods/FID-A)
- MARSS-generated Philips PRESS TE 30 ms basis set (downloaded from http://juchem.bme.columbia.edu/mr-spectroscopy-basis-sets)
- Phantom-acquired LCModel basis set press_te30_3t_01a.basis (from http://s-provencher.com/lcm-basis.shtml)
- Edited GABA difference spectra from MRSCloud runs at spatial_points = 21, 41, and 101 (MEGA/HERMES/HERCULES, PRESS and sLASER, TE 80 ms)

## Outputs

- Per-metabolite ICC values (CSV, e.g. mrscloud_lcmodel_icc.csv or gaba_resolution_icc.csv)
- Per-metabolite ICC and NRMS values (CSV, e.g. icc_nrms_comparison.csv or mrscloud_marss_icc_nrms.csv)
- 3-ppm GABA integrals per spatial-point setting when comparing edited GABA resolutions

## How to apply

Pair each metabolite's basis spectra between the two (or more) sets, first aligning them on a common spectral grid and, when comparing simulated to phantom-acquired LCModel spectra (e.g. press_te30_3t_01a.basis), applying 1 Hz FWHM Gaussian line broadening so linewidths match. Load the paired spectra into R/RStudio and compute per-metabolite ICCs; optionally compute normalized root-mean-square differences for paired simulations. Interpret against paper-specific thresholds: ICC >= 0.98 and NRMS < 1% for MRSCloud vs release FID-A (PRESS TE 30/68 ms), ICC >= 0.96 and NRMS < 9% for MRSCloud vs MARSS (unedited Philips PRESS TE 30 ms), ICC > 0.99 across 21x21 vs 41x41 vs 101x101 spatial resolutions of edited GABA signals, and lowest/highest ICCs of 0.68 (Gln) / 0.96 (NAA) for MRSCloud vs the LCModel phantom basis set. Tolerate larger discrepancies (NRMS 5-8%) for strongly J-coupled spins such as GABA, Gln, and Glu, and write the per-metabolite values to a CSV (e.g. icc_nrms_comparison.csv, mrscloud_marss_icc_nrms.csv, mrscloud_lcmodel_icc.csv, gaba_resolution_icc.csv).

## Related tools

- **R** (Statistical environment used to compute per-metabolite ICCs and normalized root-mean-square differences between paired basis spectra)
- **RStudio** (IDE in which the R statistical analysis of ICCs and NRMS is performed)
- **MRSCloud** (Generates the simulated metabolite basis functions (one comparison arm) via run_simulations_cloud.m) — https://github.com/shui5/MRSCloud
- **FID-A** (Release-version simulation code producing the reference basis functions compared against MRSCloud via ICC/NRMS) — https://github.com/CIC-methods/FID-A
- **MARSS** (Source of the independently generated Philips PRESS TE 30 ms basis set compared to MRSCloud using ICCs and NRMS)
- **LCModel** (Provides the phantom-acquired basis set (press_te30_3t_01a.basis) whose per-metabolite ICCs against MRSCloud simulations are computed)

## Examples

```
Rscript -e 'icc_fit <- irr::icc(cbind(mrscloud, fida), type="agreement"); nrms <- sqrt(mean((mrscloud-fida)^2))/sqrt(mean(fida^2))'
```

## Evaluation signals

- MRSCloud vs native FID-A (PRESS TE 30/68 ms, 41x41 spatial points): every metabolite ICC >= 0.98 and NRMS < 1%
- MRSCloud vs MARSS (unedited Philips PRESS TE 30 ms): every metabolite ICC > 0.96 and NRMS < 9%; NRMS of 5-8% is tolerated for strongly J-coupled GABA, Gln, and Glu
- Edited GABA signals across 21x21, 41x41 and 101x101 spatial points: ICCs > 0.99 for MEGA, HERMES and HERCULES in both PRESS and sLASER, with slightly larger 3-ppm GABA integrals at higher spatial-point counts
- MRSCloud vs phantom-acquired LCModel basis set: ICC pattern matches the report with lowest ICC for Gln at 0.68 and highest for NAA at 0.96, and metabolites matched by name after 1 Hz FWHM Gaussian broadening
- Output CSVs contain one ICC (and where applicable one NRMS) value per metabolite across all 8 commonly used metabolites (avoiding complex J-coupled Cystat, HCar, iLe, Lys, and Glc)

## Limitations

- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time and should be excluded to keep comparisons tractable.
- Higher spatial_points settings (101 vs 41) substantially increase simulation time before ICC computation is possible.
- ICCs against the LCModel phantom basis set are hard to interpret because limited information was provided for the basis-set acquisition, and its waveforms and timings are not well documented and likely differ from the MRSCloud simulation.

## Evidence
