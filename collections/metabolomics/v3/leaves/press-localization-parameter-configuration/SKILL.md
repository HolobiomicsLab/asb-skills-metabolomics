---
name: press-localization-parameter-configuration
description: Use when a vendor-, sequence-, and field-strength-specific (3T PRESS)
  metabolite basis set must be simulated in MATLAB, or when validating simulated basis
  functions against release FID-A or phantom-measured (LCModel) basis sets.
when_to_use_negative:
- The experiment requires STEAM or SPECIAL localization — MRSCloud does not implement
  these sequences.
- The target field strength is not 3T — the existing MRSCloud version only supports
  3T basis-set generation.
- The sequence requires product-specific RF waveforms or replicating all disseminated
  vendor versions (e.g., MEGA-PRESS variants) — product sequences and RF waveforms
  are not shared in the GitHub repo and simulations cannot capture all such differences.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MRSCloud
  role: MATLAB code that simulates metabolite basis functions using the 1D projection
    method, coherence pathway filters, and pre-calculated propagators; configured
    via simMRS. and executed with run_simulations_cloud.m
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Reference spectral-simulation package upon which MRSCloud is built; used as
    the comparison code path for validating PRESS-simulated basis functions
  repo: https://github.com/CIC-methods/FID-A
- name: MATLAB
  role: Runtime environment in which simMRS. parameters are set and run_simulations_cloud.m
    is executed for both MRSCloud and FID-A simulations
- name: R
  role: Used (in RStudio) to compute intraclass correlation coefficients between MRSCloud-
    and FID-A-simulated basis spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:10f3ece1775a097fcd352b518ed6d7c6d34748ebdec0976f97b9064a7bb93ef9
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# press-localization-parameter-configuration

## Summary

Configure and run density-matrix basis-set simulations for 1H MRS with PRESS localization (e.g., TE 30/68 ms) using MRSCloud's MATLAB code (built on FID-A), tuning parameters such as simMRS. fields, spatial_points (41 acceptable, 101 ideal), and output directories before executing run_simulations_cloud.m.

## When to use

Use when a vendor-, sequence-, and field-strength-specific (3T PRESS) metabolite basis set must be simulated in MATLAB, or when validating simulated basis functions against release FID-A or phantom-measured (LCModel) basis sets.

## When NOT to use

- The experiment requires STEAM or SPECIAL localization — MRSCloud does not implement these sequences.
- The target field strength is not 3T — the existing MRSCloud version only supports 3T basis-set generation.
- The sequence requires product-specific RF waveforms or replicating all disseminated vendor versions (e.g., MEGA-PRESS variants) — product sequences and RF waveforms are not shared in the GitHub repo and simulations cannot capture all such differences.

## Inputs

- simMRS. configuration file (metabolite list, FieldStr, vendor, mega_or_hadam, localization=PRESS, TE in ms, editOn/editOff, editTp, spatial_points, output directories)
- List of metabolites to simulate (e.g., common healthy-brain metabolites such as NAA, Cr, GABA, Glu, Gln, GPC, PCh, mI)
- MRSCloud MATLAB code including run_simulations_cloud.m
- FID-A release codebase (github.com/CIC-methods/FID-A)

## Outputs

- Simulated vendor- and sequence-specific metabolite basis functions for PRESS localization at the specified TE (e.g., 30 ms and 68 ms)
- Zipped basis set download including a PDF image of the spectra (generated with the plot basis function integrated from Osprey) for quality assurance
- Comparison table such as icc_nrms_comparison.csv with per-metabolite ICC and NRMS values when validating against FID-A

## How to apply

Obtain the MRSCloud code (github.com/shui5/MRSCloud, including run_simulations_cloud.m) and the official FID-A release (github.com/CIC-methods/FID-A). Edit simMRS.: set FieldStr to 3T, vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), mega_or_hadam (UnEdited/MEGA/HERMES/HERCULES), localization to PRESS, TE to the desired echo time in ms (e.g., 30 or 68 ms), and spatial_points (41 is acceptable, 101 ideal; higher values increase runtime). Keep the private parameters unchanged and set the output directories (work_dir is not necessary for MATLAB), then run the run_simulations_cloud.m script. Avoid complex J-coupled spin systems such as Cystat, HCar, iLe, Lys, and Glc, which dramatically increase simulation time; a full PRESS basis set takes approximately 11 minutes on the server thanks to the 1D projection method, coherence order filtering, and pre-calculation of propagators. Validate the output by comparing paired basis spectra (e.g., ICC >= 0.98 and normalized RMS differences < 1% versus FID-A, computed in R/RStudio).

## Related tools

- **MRSCloud** (MATLAB code that simulates metabolite basis functions using the 1D projection method, coherence pathway filters, and pre-calculated propagators; configured via simMRS. and executed with run_simulations_cloud.m) — https://github.com/shui5/MRSCloud
- **FID-A** (Reference spectral-simulation package upon which MRSCloud is built; used as the comparison code path for validating PRESS-simulated basis functions) — https://github.com/CIC-methods/FID-A
- **MATLAB** (Runtime environment in which simMRS. parameters are set and run_simulations_cloud.m is executed for both MRSCloud and FID-A simulations)
- **R** (Used (in RStudio) to compute intraclass correlation coefficients between MRSCloud- and FID-A-simulated basis spectra)

## Examples

```
Edit simMRS. (localization: "PRESS", TE: 30, spatial_points: 41, vendor: "GE", FieldStr: "3T") then run run_simulations_cloud.m in MATLAB
```

## Evaluation signals

- Per-metabolite ICC between MRSCloud- and FID-A-simulated basis spectra is at least 0.98 for all 8 commonly used metabolites
- Normalized root-mean-square values of the differences between paired simulated basis spectra are under 1% for all metabolites
- A full PRESS basis set simulation completes in approximately 11 minutes on the server; substantial deviations (absent complex spin systems like Cystat, HCar, iLe, Lys, Glc) suggest misconfiguration
- The downloaded basis set includes a PDF image of the spectra for quality assurance, and spectra visually match expected metabolite patterns
- Configuration matches the intended acquisition: localization=PRESS, FieldStr=3T, correct vendor, TE in ms (e.g., 30/68), and spatial_points=41 (or 101)

## Limitations

- Only 3T is supported; STEAM and SPECIAL localization are not implemented and would be valuable future additions.
- Cystat, HCar, iLe, and Lys have complex J-coupling spin systems and significantly increase simulation time; Glc takes extremely long.
- Product sequences and RF waveforms are not shared in the GitHub repo; simulations cannot capture all possible vendor experiments or all differences between disseminated MEGA-PRESS versions within each vendor.
- MRSCloud favors ease of use by limiting the number of user-specified parameters, reducing simulation flexibility.
- Spin-system parameters rest largely on the work of Govindaraju et al. with only minor updates.

## Evidence

- [methods] run script: [evidence span withheld — anchor status: not_found]
