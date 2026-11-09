---
name: edited-mrs-sequence-parameter-configuration
description: Use when generating a vendor- and sequence-specific simulated basis set
  for an edited MRS experiment (MEGA-PRESS, HERMES, HERCULES) and you need to map
  your scanner acquisition parameters onto MRSCloud's simMRS.json options.
when_to_use_negative:
- The experiment uses STEAM or SPECIAL localization — MRSCloud does not implement
  these and the article flags their inclusion as a future addition.
- A non-3T field strength is needed — the current MRSCloud release only supports basis-set
  generation at 3T.
- Exact replication of a vendor's disseminated MEGA-PRESS variant is required — the
  simulations cannot capture all differences between disseminated versions of MEGA-PRESS
  within each vendor, and Philips hardware-specific parameter sets are only approximated.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3384
tools:
- name: MRSCloud
  role: Provides the density-matrix simulation engine and the editable simMRS.json/run_simulations_cloud.m
    entry points that this skill configures for edited sequences (MEGA/HERMES/HERCULES
    with PRESS or sLASER).
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral-simulation toolbox on which MRSCloud is built; its addpath(genpath(...))
    installation step is part of the setup for running the configured simulation.
  repo: https://github.com/CIC-methods/FID-A
- name: Osprey
  role: Supplies the basis-function plotting used to generate the PDF quality-assurance
    image included with each downloaded MRSCloud basis set.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:1f9a2c42eb609884ed6125ad527ddea23d1fa4b91a7b34faa00103c8830dbc7b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MRSCloud
  - Osprey
  - R
  - RStudio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# edited-mrs-sequence-parameter-configuration

## Summary

Configures edited MRS sequence parameters (MEGA-PRESS, HERMES, HERCULES) in MRSCloud's simMRS.json by setting vendor, localization, TE, editing pulse duration, editOn/editOff frequencies, and spatial points, so density-matrix-simulated metabolite basis functions match the user's acquisition. This skill defines the decision points where MRSCloud fixes values (e.g. TE fixed at 68 ms for HERMES, 80 ms for HERCULES) versus where the user must supply sequence-specific parameters (e.g. editTp = 14 ms for MEGA-PRESS/HERMES vs 20 ms for HERCULES).

## When to use

Use when generating a vendor- and sequence-specific simulated basis set for an edited MRS experiment (MEGA-PRESS, HERMES, HERCULES) and you need to map your scanner acquisition parameters onto MRSCloud's simMRS.json options.

## When NOT to use

- The experiment uses STEAM or SPECIAL localization — MRSCloud does not implement these and the article flags their inclusion as a future addition.
- A non-3T field strength is needed — the current MRSCloud release only supports basis-set generation at 3T.
- Exact replication of a vendor's disseminated MEGA-PRESS variant is required — the simulations cannot capture all differences between disseminated versions of MEGA-PRESS within each vendor, and Philips hardware-specific parameter sets are only approximated.

## Inputs

- Scanner acquisition parameters (vendor, sequence type, TE, editing-pulse duration, editing frequencies) for the edited MRS experiment to be simulated
- simMRS.json configuration file shipped with MRSCloud

## Outputs

- Vendor- and sequence-specific simulated metabolite basis functions for the edited sequence (e.g. editOn/editOff spectra for MEGA-PRESS, differenced spectra for HERMES/HERCULES)
- Zipped basis set download including a PDF image of the spectra (generated with the plot function integrated from Osprey) for quality assurance

## How to apply

Edit simMRS.json in the MRSCloud repository (a MATLAB package built on FID-A) to specify: the metabolites of interest (e.g. "GABA" from the healthy-brain list), FieldStr (only 3T is supported), vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), mega_or_hadam (UnEdited/MEGA/HERMES/HERCULES), and localization (PRESS/sLASER). Set TE in ms — HERMES and HERCULES are fixed at 68 ms and 80 ms respectively, while MEGA-PRESS editOn/editOff are flexible. Set editTp (editing-pulse duration) to 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES, keep private parameters unchanged, and set spatial_points to 41 (acceptable) or 101 (ideal, but slower). Then run run_simulations_cloud.m and set output directories (work_dir is unnecessary for MATLAB). Rationale: robust simulated basis sets require vendor-appropriate RF pulse shapes, correct timings, and sufficient spatial resolution; approximations are least tolerable for low-concentration coupled spins such as GABA.

## Related tools

- **MRSCloud** (Provides the density-matrix simulation engine and the editable simMRS.json/run_simulations_cloud.m entry points that this skill configures for edited sequences (MEGA/HERMES/HERCULES with PRESS or sLASER).) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral-simulation toolbox on which MRSCloud is built; its addpath(genpath(...)) installation step is part of the setup for running the configured simulation.) — https://github.com/CIC-methods/FID-A
- **Osprey** (Supplies the basis-function plotting used to generate the PDF quality-assurance image included with each downloaded MRSCloud basis set.)

## Examples

```
Edit simMRS.json: set "vendor": "GE", "mega_or_hadam": "MEGA", "localization": "PRESS", "TE": 80, "editTp": 14, "spatial_points": 41; then run `run_simulations_cloud.m` in MATLAB.
```

## Evaluation signals

- The zipped download contains a PDF of the simulated spectra; visually confirm the expected edited (e.g. GABA editOn–editOff difference) pattern for the chosen sequence/localization/TE combination.
- Check that simMRS.json values are consistent with the constraints: editTp = 14 ms for MEGA-PRESS/HERMES vs 20 ms for HERCULES; TE = 68 ms for HERMES and 80 ms for HERCULES; spatial_points = 41 (acceptable) or 101 (ideal).
- Compare the resulting basis functions against the same sequence simulated with FID-A (expected ICC ≥ 0.98) or MARSS (expected ICC ≥ 0.96) as a cross-tool validation of the parameter configuration.
- A full PRESS basis-set simulation should complete in roughly 11 minutes on the server — large deviations suggest a misconfigured or unusually complex spin system (e.g. Cystat, HCar, iLe, Lys, or Glc drastically lengthen runtime).
- If GABA was simulated, the resulting edited-GABA basis spectra at different spatial_points settings (21/41/101) should agree with ICC > 0.99, indicating sufficient spatial resolution for the configured sequence.

## Limitations

- Product sequences and RF waveforms are not shared in the GitHub repository, so the simulation uses generic vendor pulse shapes rather than the exact disseminated waveforms.
- Simulations correspond to the most commonly used parameter set per vendor and cannot accurately capture all possible Philips experiments (parameter sets vary with hardware configurations and scanner protocol settings).
- Spin-system parameters rest largely on Govindaraju et al. with only minor updates, so chemical-shift/J-coupling accuracy may lag newer literature (e.g. GABA).
- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys — and Glc, which takes extremely long) substantially increase simulation time.

## Evidence

- [methods] run_simulations_cloud.m entry point: [evidence span withheld — anchor status: not_found]
