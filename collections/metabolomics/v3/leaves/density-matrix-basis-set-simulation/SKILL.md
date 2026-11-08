---
name: density-matrix-basis-set-simulation
description: Use when a LCModel/Osprey/FSL-MRS-style analysis needs a metabolite basis
  set matched to a specific vendor, sequence (PRESS/MEGA-PRESS/HERMES/HERCULES), and
  TE, and no appropriate basis set is available off-site.
when_to_use_negative:
- Field strengths other than 3T — the current MRSCloud version only supports 3T basis
  sets
- Localization sequences other than PRESS or sLASER (e.g. STEAM or SPECIAL are not
  implemented)
- When exact vendor RF waveforms/timings are required and the target protocol deviates
  from the most commonly used parameter set MRSCloud simulates (e.g. Philips experiments
  varying with hardware configuration, or disseminated MEGA-PRESS variants within
  a vendor)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: MRSCloud
  role: Density-matrix simulation engine used to generate vendor- and sequence-specific
    metabolite basis sets online or via run_simulations_cloud.m in MATLAB
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Spectral simulation functionality upon which MRSCloud was built; also used
    to generate the native reference basis set for comparison
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Source of the reference Philips PRESS TE 30 ms unedited basis set used to
    cross-validate MRSCloud simulations
- name: LCModel
  role: Provides the experimentally (phantom-)measured reference basis set (e.g. press_te30_3t_01a.basis)
    for qualitative comparison
- name: R
  role: 'Statistical analysis: computing per-metabolite ICCs and normalized RMS differences
    in RStudio'
- name: MATLAB
  role: Environment in which run_simulations_cloud.m and the FID-A-based simulations
    are executed
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:135a2dce740bcf05fcce4e6a4d0073a969fa6c968fcf14e783185dad48a60336
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# density-matrix-basis-set-simulation

## Summary

Simulate vendor- and sequence-specific MRS metabolite basis sets using density-matrix simulations accelerated by the 1D projection method, coherence order filtering, and pre-calculation of propagators (as implemented in MRSCloud, built on FID-A), then validate the resulting basis functions against reference sets (native FID-A, MARSS, or phantom-measured LCModel) using ICC and normalized RMS differences.

## When to use

Use when a LCModel/Osprey/FSL-MRS-style analysis needs a metabolite basis set matched to a specific vendor, sequence (PRESS/MEGA-PRESS/HERMES/HERCULES), and TE, and no appropriate basis set is available off-site.

## When NOT to use

- Field strengths other than 3T — the current MRSCloud version only supports 3T basis sets
- Localization sequences other than PRESS or sLASER (e.g. STEAM or SPECIAL are not implemented)
- When exact vendor RF waveforms/timings are required and the target protocol deviates from the most commonly used parameter set MRSCloud simulates (e.g. Philips experiments varying with hardware configuration, or disseminated MEGA-PRESS variants within a vendor)

## Inputs

- simMRS. configuration file (metabolite list, vendor, sequence, localization, TE, editTp, spatial_points)
- Metabolite spin-system definitions shipped with MRSCloud/FID-A
- Reference basis set for validation (e.g. MARSS Philips PRESS TE 30 ms .basis file from juchem.bme.columbia.edu or native FID-A simulation output)

## Outputs

- Simulated vendor- and sequence-specific metabolite basis functions
- Zipped basis set download including a PDF image of the spectra for quality assurance (generated with the plot function integrated from Osprey)
- Per-metabolite ICC and NRMS comparison table (e.g. icc_nrms_comparison.csv or mrscloud_marss_icc_nrms.csv)

## How to apply

Obtain MRSCloud (github.com/shui5/MRSCloud) and FID-A (github.com/CIC-methods/FID-A); in MATLAB edit simMRS. to specify metabolites (e.g. NAA, Cr, GABA, Gln, Glu), vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), sequence (UnEdited/MEGA/HERMES/HERCULES), localization (PRESS/sLASER), TE (e.g. 30 or 68 ms for PRESS), editTp (14 ms for MEGA-PRESS/HERMES, 20 ms for HERCULES), and spatial_points (41 acceptable, 101 ideal). Keep private parameters unchanged, set output directories (work_dir is not needed for MATLAB), and run run_simulations_cloud.m; a full PRESS basis set takes about 11 minutes on the server. The acceleration comes from the 1D projection method, coherence order filtering, and pre-calculated propagators. Validate by pairing simulated spectra with a reference basis set (native FID-A, MARSS, or LCModel) and computing per-metabolite ICCs and normalized RMS differences in R/RStudio, expecting ICC >= 0.98 vs FID-A, ICC >= 0.96 vs MARSS, and NRMS < 1% (FID-A) or < 9% (MARSS).

## Related tools

- **MRSCloud** (Density-matrix simulation engine used to generate vendor- and sequence-specific metabolite basis sets online or via run_simulations_cloud.m in MATLAB) — https://github.com/shui5/MRSCloud
- **FID-A** (Spectral simulation functionality upon which MRSCloud was built; also used to generate the native reference basis set for comparison) — https://github.com/CIC-methods/FID-A
- **MARSS** (Source of the reference Philips PRESS TE 30 ms unedited basis set used to cross-validate MRSCloud simulations)
- **LCModel** (Provides the experimentally (phantom-)measured reference basis set (e.g. press_te30_3t_01a.basis) for qualitative comparison)
- **R** (Statistical analysis: computing per-metabolite ICCs and normalized RMS differences in RStudio)
- **MATLAB** (Environment in which run_simulations_cloud.m and the FID-A-based simulations are executed)

## Examples

```
In MATLAB: edit simMRS. (vendor='Philips', mega_or_hadam='UnEdited', localization='PRESS', TE=30, spatial_points=101) then run run_simulations_cloud.m
```

## Evaluation signals

- ICCs between MRSCloud and native FID-A basis spectra are at least 0.98 for all tested metabolites
- Normalized root-mean-square differences vs FID-A are below 1% for all metabolites; vs MARSS below 9% (NRMS 5–8% tolerated for strongly J-coupled spins such as GABA, Gln, and Glu)
- ICCs vs MARSS are at least 0.96 for all tested metabolites
- A full PRESS basis set completes in approximately 11 minutes on the server (runtime sanity check)
- Downloaded basis set zip contains a PDF image of the spectra (Osprey-based plot) for quality assurance

## Limitations

- Cystat, HCar, iLe, Lys, and Glc have complex J-coupled spin systems and dramatically increase simulation time
- Product sequences and RF waveforms are not shared in the GitHub repo
- User-specified parameters are limited to favor ease of use; simulations capture only the most commonly used parameter set and cannot reproduce all vendor/hardware/protocol variants
- Spin-system parameters rely largely on Govindaraju et al. with only minor updates
- Comparison to the LCModel phantom-measured basis set is hindered because waveforms, timings, and acquisition details of that basis set are not well documented (ICCs ranged 0.68 for Gln to 0.96 for NAA)
- Only tested on brain MRS data from 3T scanners; only 3T generation is supported

## Evidence
