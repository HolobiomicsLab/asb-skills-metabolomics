---
name: spectral-basis-function-grid-alignment
description: Use when two MRS basis sets (e.g., an MRSCloud-simulated set and a reference
  set such as MARSS or LCModel) must be compared metabolite-by-metabolite and their
  basis functions first need to be placed on a shared spectral grid.
when_to_use_negative:
- Comparing MRSCloud output against the LCModel phantom-measured basis set (press_te30_3t_01a.basis)
  — waveforms and timings of that vendor-provided set are not well documented, making
  grid-aligned comparison of limited interpretability
- The target sequence is STEAM or SPECIAL localization — these are not implemented
  in MRSCloud, so no MRSCloud basis functions exist to align
- Field strengths other than 3T — the current MRSCloud version only supports 3T basis
  set generation
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MRSCloud
  role: Generates the simulated metabolite basis functions to be aligned and compared
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation functionality upon which MRSCloud's basis set
    simulation is built
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Source of the reference basis set against which aligned MRSCloud basis functions
    are compared
- name: R
  role: Computes per-metabolite ICCs and normalized root-mean-square differences between
    aligned basis functions
- name: RStudio
  role: Environment in which the statistical comparison of aligned basis functions
    is performed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:2fc1cbd531a37b0fbce8365167feb775cd502c71317437f177246bed9fa43952
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MARSS
  - MRSCloud
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-basis-function-grid-alignment

## Summary

Aligns simulated MRS metabolite basis functions from different sources (e.g., MRSCloud density-matrix simulations vs. MARSS-generated basis sets) onto a common spectral grid so per-metabolite agreement can be quantified with intraclass correlation coefficients (ICCs) and normalized root-mean-square differences (NRMS).

## When to use

Use when two MRS basis sets (e.g., an MRSCloud-simulated set and a reference set such as MARSS or LCModel) must be compared metabolite-by-metabolite and their basis functions first need to be placed on a shared spectral grid.

## When NOT to use

- Comparing MRSCloud output against the LCModel phantom-measured basis set (press_te30_3t_01a.basis) — waveforms and timings of that vendor-provided set are not well documented, making grid-aligned comparison of limited interpretability
- The target sequence is STEAM or SPECIAL localization — these are not implemented in MRSCloud, so no MRSCloud basis functions exist to align
- Field strengths other than 3T — the current MRSCloud version only supports 3T basis set generation

## Inputs

- MRSCloud-simulated metabolite basis set (e.g., unedited Philips PRESS TE 30 ms, spatial_points=101, generated via run_simulations_cloud.m)
- Reference basis set (e.g., MARSS-generated Philips PRESS TE 30 ms basis set downloaded from http://juchem.bme.columbia.edu/mr-spectroscopy-basis-sets)
- MRSCloud download PDF image of the spectra for quality assurance
- Metabolite list (e.g., Asc, Asp, Cr, EA, GABA, GPC, GSH, Gln, Glu, Gly, H2O, Lac, mI, NAA, NAAG, PCh, PCr, PE, Ser, sI, Tau)

## Outputs

- Per-metabolite ICC and NRMS comparison table (e.g., mrscloud_marss_icc_nrms.csv)
- Common spectral grid onto which each metabolite's basis function from both sets has been aligned

## How to apply

Generate or obtain both basis sets under matched acquisition parameters — for the MRSCloud side, configure the FID-A-based density-matrix simulation (run_simulations_cloud.m) for e.g. unedited Philips PRESS at TE 30 ms, setting spatial_points to 101 (41 acceptable) and keeping private parameters unchanged. Download the reference basis set (e.g., the MARSS Philips PRESS TE 30 ms set from the Juchem lab site) and load both sets, including the PDF quality-assurance image MRSCloud ships with each zipped download. Align each metabolite's simulated basis function (Glu, Gln, NAA, GABA, etc.) onto a common spectral grid so the complex time-domain signals from the two simulations can be compared pointwise. In R/RStudio, compute per-metabolite ICCs and normalized root-mean-square differences between the aligned MRSCloud and MARSS basis functions, tolerating larger discrepancies (NRMS 5–8%) for strongly J-coupled spins such as GABA, Gln, and Glu. Write the per-metabolite ICC and NRMS values to a results table (e.g., mrscloud_marss_icc_nrms.csv).

## Related tools

- **MRSCloud** (Generates the simulated metabolite basis functions to be aligned and compared) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation functionality upon which MRSCloud's basis set simulation is built) — https://github.com/CIC-methods/FID-A
- **MARSS** (Source of the reference basis set against which aligned MRSCloud basis functions are compared)
- **R** (Computes per-metabolite ICCs and normalized root-mean-square differences between aligned basis functions)
- **RStudio** (Environment in which the statistical comparison of aligned basis functions is performed)

## Examples

```
Rscript compare_basis_sets.R --mrscloud basis_mrscloud_press_te30.mat --marss marss_press_te30.basis --out mrscloud_marss_icc_nrms.csv
```

## Evaluation signals

- Every metabolite's ICC between MRSCloud and MARSS basis functions is >= 0.96 (article reports ICCs at least 0.96 between MRSCloud and MARSS)
- Normalized root-mean-square differences are below 9% for all metabolites
- NRMS of 5–8% is expected and tolerated only for strongly J-coupled spins such as GABA, Gln, and Glu; larger unexplained discrepancies indicate misalignment
- Output CSV contains one ICC and one NRMS row per tested metabolite (e.g., mrscloud_marss_icc_nrms.csv)
- The MRSCloud download's included PDF spectra image passes quality assurance before alignment

## Limitations

- MRSCloud limits the number of user-specified parameters to favor ease of use, restricting simulation flexibility for grid alignment across non-standard parameter sets
- Simulations correspond to the most commonly used parameter set and cannot accurately capture all possible Philips experiments, hardware configurations, or scanner protocol settings
- MRSCloud cannot capture all differences between disseminated versions of MEGA-PRESS within each vendor
- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time, and Glc takes extremely long to simulate

## Evidence

- [other] workflow validation threshold: [evidence span withheld — anchor status: not_found]
