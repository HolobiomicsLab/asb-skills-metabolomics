---
name: mrs-basis-set-simulation
description: Use when a vendor-specific, sequence-specific (PRESS/sLASER, unedited
  or MEGA/HERMES/HERCULES edited) metabolite basis set is needed for LCModel-style
  fitting at 3T and no experimentally acquired basis set is available.
when_to_use_negative:
- You need STEAM or SPECIAL localization — these are not implemented in MRSCloud
- You require exact capture of vendor-specific product RF waveforms, hardware-dependent
  Philips parameter sets, or all disseminated MEGA-PRESS variants — these are not
  shared in the repo and the simulations cannot capture them
- Your scanner field strength is not 3T — FieldStr only supports 3T basis set generation
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MRSCloud
  role: Cloud/MATLAB tool that performs the density-matrix basis set simulation with
    1D projection, coherence pathway filters, and propagator pre-calculation
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation toolbox on which MRSCloud is built; also used
    for elementwise reference simulations and EXORCYCLE phase-cycled comparison
  repo: https://github.com/CIC-methods/FID-A
- name: LCModel
  role: Source of the phantom-acquired reference basis set (press_te30_3t_01a.basis)
    used for ICC validation
- name: R
  role: Statistical environment for computing per-metabolite ICCs between simulated
    and reference basis spectra
- name: RStudio
  role: IDE in which the R-based ICC analysis is performed
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:a2b42c871d9f2c839b5a2e1527d7b7b6d22ce5da2c93e93b0580d0b393f0f237
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# mrs-basis-set-simulation

## Summary

Simulate vendor- and sequence-specific metabolite basis sets for 3T brain MRS using MRSCloud (built on FID-A), which accelerates density-matrix simulations via the 1D projection method, coherence pathway filters, and pre-calculation of propagators.

## When to use

Use when a vendor-specific, sequence-specific (PRESS/sLASER, unedited or MEGA/HERMES/HERCULES edited) metabolite basis set is needed for LCModel-style fitting at 3T and no experimentally acquired basis set is available.

## When NOT to use

- You need STEAM or SPECIAL localization — these are not implemented in MRSCloud
- You require exact capture of vendor-specific product RF waveforms, hardware-dependent Philips parameter sets, or all disseminated MEGA-PRESS variants — these are not shared in the repo and the simulations cannot capture them
- Your scanner field strength is not 3T — FieldStr only supports 3T basis set generation

## Inputs

- simMRS.json configuration file (metabolite list, vendor, mega_or_hadam, localization, TE, editOn/editOff, editTp, spatial_points)
- Spin-system and pulse-sequence definitions from the shui5/MRSCloud repository
- Optional reference basis set for validation, e.g. phantom-acquired LCModel press_te30_3t_01a.basis from s-provencher.com

## Outputs

- Vendor- and sequence-specific simulated metabolite basis set (zipped, with a PDF image of the spectra for quality assurance)
- Basis spectra comparable to LCModel/FID-A/MARSS outputs, suitable for downstream linear-combination modeling

## How to apply

Clone shui5/MRSCloud (built on CIC-methods/FID-A), edit simMRS.json to specify metabolites (e.g. Asc, Asp, Cr, GABA, Gln, Glu, NAA...), vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), mega_or_hadam (UnEdited/MEGA/HERMES/HERCULES), localization (PRESS/sLASER), TE in ms, editTp (14 ms for MEGA-PRESS/HERMES, 20 ms for HERCULES), and spatial_points (41 acceptable, 101 ideal; higher values increase runtime). Keep private parameters unchanged, set output directories (work_dir not needed for MATLAB), and run run_simulations_cloud.m; a full PRESS basis set takes roughly 11 minutes on the server, but complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically extend runtime. The tool substitutes coherence order filtering (zeroing density-matrix elements whose coherence order p = m_i - m_j does not match the desired -1 single-quantum pathway) for full EXORCYCLE phase cycling and pre-calculates shaped-pulse propagators instead of computing them per sample. Zipped downloads include a PDF image of the spectra (plotted with the Osprey-derived plot_basis function) for quality assurance. Validate by comparing simulated basis functions against FID-A, MARSS, or phantom-acquired LCModel basis sets using per-metabolite intraclass correlation coefficients computed in R/RStudio, with 1 Hz FWHM Gaussian line broadening applied to match linewidths before comparison.

## Related tools

- **MRSCloud** (Cloud/MATLAB tool that performs the density-matrix basis set simulation with 1D projection, coherence pathway filters, and propagator pre-calculation) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation toolbox on which MRSCloud is built; also used for elementwise reference simulations and EXORCYCLE phase-cycled comparison) — https://github.com/CIC-methods/FID-A
- **LCModel** (Source of the phantom-acquired reference basis set (press_te30_3t_01a.basis) used for ICC validation)
- **R** (Statistical environment for computing per-metabolite ICCs between simulated and reference basis spectra)
- **RStudio** (IDE in which the R-based ICC analysis is performed)

## Examples

```
edit simMRS.json (e.g. metabolites ["NAA","GABA"], vendor "GE", mega_or_hadam "UnEdited", localization "PRESS", TE 30, spatial_points 41) then run: run_simulations_cloud.m
```

## Evaluation signals

- ICCs between MRSCloud and FID-A basis spectra reach at least 0.98 (and at least 0.96 vs MARSS) for every metabolite
- ICCs between simulated and phantom-acquired LCModel basis spectra span 0.68 (Gln, lowest) to 0.96 (NAA, highest), with the lowest ICCs corresponding to the largest root-mean-square values
- ICCs among edited GABA signals simulated at spatial_points = 21, 41 and 101 exceed 0.99, confirming 41 points is acceptable and 101 ideal
- A full PRESS basis set completes in approximately 11 minutes on the server when complex spin systems (Cystat, HCar, iLe, Lys, Glc) are excluded
- Downloaded zipped basis set includes a PDF image of the spectra for quality assurance

## Limitations

- Ease of use is favored by limiting the number of user-specified parameters, reducing simulation flexibility
- Simulations correspond to the most commonly used parameter sets and cannot accurately capture all possible Philips experiments or all disseminated MEGA-PRESS variants within each vendor
- Spin-system parameters rely largely on Govindaraju et al. with only minor updates (e.g. GABA)
- Cystat, HCar, iLe, Lys significantly increase simulation time, and Glc takes extremely long due to its complex spin system
- Product sequences and RF waveforms are not shared in the GitHub repo
- Comparison with the LCModel phantom-measured basis set is challenging because waveforms, timings, and acquisition details are not well documented and likely differ from the simulation

## Evidence
