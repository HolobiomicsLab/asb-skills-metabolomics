---
name: metabolite-basis-function-alignment
description: Use when you have a simulated MRS metabolite basis set and an experimentally
  acquired LCModel basis set and need to validate simulation fidelity per metabolite
  via ICCs.
when_to_use_negative:
- The comparison basis set is itself simulated rather than phantom-acquired — ICCs
  against another simulator (e.g., FID-A/MARSS) are a separate, near-identical validation
  with different expected ICC ranges (>=0.98 and >=0.96 respectively).
- The sequence is not supported by MRSCloud (e.g., STEAM or SPECIAL localization are
  not implemented) or the field strength is not 3T.
- The vendor basis set's waveforms and timings are unknown and you need exact spectral
  correspondence rather than statistical agreement — the article notes measured basis
  sets are poorly documented, limiting interpretability.
edam_operation: http://edamontology.org/operation_2239
tools:
- name: MRSCloud
  role: Generates the vendor- and sequence-specific simulated basis set (density-matrix
    simulation via 1D projection method, coherence order filtering, pre-calculated
    propagators) that is compared to the measured basis
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation toolbox upon which MRSCloud is built; provides
    the FID/spectra data structures being aligned
  repo: https://github.com/CIC-methods/FID-A
- name: LCModel
  role: Source of the phantom-acquired reference basis set (press_te30_3t_01a.basis)
    used as the ground truth for ICC comparison
- name: R
  role: Statistical environment used to compute per-metabolite intraclass correlation
    coefficients
- name: RStudio
  role: IDE in which the ICC analysis in R is performed and results are exported to
    mrscloud_lcmodel_icc.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:221ce0fe389671a3a3deb9c69231a310cba7fca269f1affd69d09c8eeb6aae60
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-basis-function-alignment

## Summary

Aligns density-matrix-simulated metabolite basis functions (e.g., MRSCloud unedited PRESS TE 30 ms) to a phantom-acquired LCModel basis set (press_te30_3t_01a.basis) by matching metabolites by name after matching linewidths, then quantifies agreement with per-metabolite intraclass correlation coefficients (ICCs).

## When to use

Use when you have a simulated MRS metabolite basis set and an experimentally acquired LCModel basis set and need to validate simulation fidelity per metabolite via ICCs.

## When NOT to use

- The comparison basis set is itself simulated rather than phantom-acquired — ICCs against another simulator (e.g., FID-A/MARSS) are a separate, near-identical validation with different expected ICC ranges (>=0.98 and >=0.96 respectively).
- The sequence is not supported by MRSCloud (e.g., STEAM or SPECIAL localization are not implemented) or the field strength is not 3T.
- The vendor basis set's waveforms and timings are unknown and you need exact spectral correspondence rather than statistical agreement — the article notes measured basis sets are poorly documented, limiting interpretability.

## Inputs

- MRSCloud-simulated unedited PRESS TE 30 ms basis spectra (produced via run_simulations_cloud.m with spatial_points >= 41)
- Phantom-acquired LCModel basis set press_te30_3t_01a.basis (from http://s-provencher.com/lcm-basis.shtml)
- Metabolite name list from simMRS.json (e.g., Gln, Glu, NAA, GABA, Cr) for name-based pairing

## Outputs

- Per-metabolite ICC values comparing MRSCloud-simulated vs LCModel-acquired basis spectra
- mrscloud_lcmodel_icc.csv with per-metabolite ICCs
- Aligned simulated/measureed basis function pairs (after 1 Hz FWHM Gaussian line broadening)

## How to apply

Generate the simulated basis spectra in MRSCloud by configuring simMRS.json (vendor, UnEdited, PRESS, TE 30 ms) and running run_simulations_cloud.m with spatial_points set to at least 41 (101 is ideal), since insufficient spatial resolution degrades the basis functions. Download the phantom-acquired LCModel basis set press_te30_3t_01a.basis and apply 1 Hz FWHM Gaussian line broadening to the simulated spectra so linewidths match the phantom-acquired basis functions — this is the key preprocessing step before comparison. Align each metabolite basis function between the two sets by matching metabolites by name (e.g., Gln, NAA, GABA from the MRSCloud metabolite list). Compute per-metabolite ICCs between the paired simulated and measured spectra using R in RStudio and write results to mrscloud_lcmodel_icc.csv. Validate against the reported pattern: Gln lowest at 0.68 (largest root-mean-square deviation) and NAA highest at 0.96 (smallest RMS).

## Related tools

- **MRSCloud** (Generates the vendor- and sequence-specific simulated basis set (density-matrix simulation via 1D projection method, coherence order filtering, pre-calculated propagators) that is compared to the measured basis) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation toolbox upon which MRSCloud is built; provides the FID/spectra data structures being aligned) — https://github.com/CIC-methods/FID-A
- **LCModel** (Source of the phantom-acquired reference basis set (press_te30_3t_01a.basis) used as the ground truth for ICC comparison)
- **R** (Statistical environment used to compute per-metabolite intraclass correlation coefficients)
- **RStudio** (IDE in which the ICC analysis in R is performed and results are exported to mrscloud_lcmodel_icc.csv)

## Examples

```
R> icc <- ICC(x = data.frame(mrscloud_sim, lcmodel_phantom)); write.csv(icc, "mrscloud_lcmodel_icc.csv")
```

## Evaluation signals

- Reproduced ICC pattern matches the reported values: Gln lowest at 0.68, NAA highest at 0.96
- Simulated spectra have been broadened with 1 Hz FWHM Gaussian line broadening so linewidths match the phantom-acquired basis functions before ICC computation
- Every metabolite name present in both sets (e.g., all 21 common-brain metabolites) is paired and has a non-missing ICC entry in mrscloud_lcmodel_icc.csv
- As a simulator sanity check upstream of alignment, MRSCloud vs FID-A ICCs should be at least 0.98 and MRSCloud vs MARSS at least 0.96
- Metabolites with the lowest ICCs (e.g., Gln) should correspond to the largest root-mean-square values between the two basis sets

## Limitations

- Waveforms and timings of the LCModel vendor-provided (phantom-measured) basis set are not well documented and likely differ from the MRSCloud simulation, limiting interpretability of ICC differences
- Basis set comparison is challenging because limited information has been provided for the acquisition of the LCModel basis sets
- MRSCloud simulation quality depends on spatial_points (41 acceptable, 101 ideal) and only supports 3T with PRESS/sLASER localization; STEAM and SPECIAL are not implemented

## Evidence

- [other] Apply 1 Hz FWHM Gaussian line broadening to the simulated spectra so linewidths match the phantom-acquired basis functions: [evidence span withheld — anchor status: not_found]
