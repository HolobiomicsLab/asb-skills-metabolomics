---
name: density-matrix-spin-system-simulation
description: Use when vendor- and sequence-matched metabolite basis sets (e.g. PRESS/sLASER
  at 3T, edited GABA with MEGA/HERMES/HERCULES) must be simulated with exact RF pulse
  shapes and timings rather than reused from off-site approximate-parameter libraries.
when_to_use_negative:
- Field strength other than 3T — the exiting version of MRSCloud only supports generation
  of basis sets in 3T
- STEAM or SPECIAL localization experiments — these are not implemented and would
  require custom extension
- Vendor experiments with non-standard parameter sets — the simulation corresponds
  to the most commonly used parameter set and cannot accurately capture all possible
  vendor configurations or all disseminated MEGA-PRESS variants within each vendor
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3335
tools:
- name: MRSCloud
  role: Primary density-matrix simulation tool (MATLAB) with 1D projection, coherence
    pathway filters, and pre-calculated propagators; run via run_simulations_cloud.m
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying spectral simulation functionality MRSCloud was built upon; also
    used for EXORCYCLE phase-cycled reference spectra
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Independent generic simulation package used as a cross-validation reference
    for the simulated basis functions
- name: Osprey
  role: Provides the plot basis function used to generate the QA PDF of downloaded
    basis sets
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:2a40210491ce36fbe76b025e7d345d2e95de083ee8c91a2c591e39b64e4f653d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MARSS
  - MRSCloud
  - Osprey
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# density-matrix-spin-system-simulation

## Summary

Simulate vendor- and sequence-specific MRS metabolite basis functions via density-matrix propagation accelerated by a 1D projection method, coherence pathway filtering, and pre-calculation of propagators, as implemented in MRSCloud (built on FID-A) for 3T brain MRS.

## When to use

Use when vendor- and sequence-matched metabolite basis sets (e.g. PRESS/sLASER at 3T, edited GABA with MEGA/HERMES/HERCULES) must be simulated with exact RF pulse shapes and timings rather than reused from off-site approximate-parameter libraries.

## When NOT to use

- Field strength other than 3T — the exiting version of MRSCloud only supports generation of basis sets in 3T
- STEAM or SPECIAL localization experiments — these are not implemented and would require custom extension
- Vendor experiments with non-standard parameter sets — the simulation corresponds to the most commonly used parameter set and cannot accurately capture all possible vendor configurations or all disseminated MEGA-PRESS variants within each vendor

## Inputs

- simMRS.json configuration file specifying metabolites, FieldStr (3T), vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), mega_or_hadam (UnEdited/MEGA/HERMES/HERCULES), localization (PRESS/sLASER), TE, editOn/editOff, editTp, and spatial_points
- Spin-system J-coupling definitions shipped with MRSCloud (derived largely from Govindaraju et al. with minor updates, e.g. GABA)
- Vendor RF pulse waveforms and sequence timings (product waveforms not shared in the GitHub repo)

## Outputs

- Simulated metabolite basis functions (voxel-averaged FIDs/spectra) for each configured metabolite and sequence
- Zipped basis set download including a PDF image of the spectra for quality assurance, generated using the plot basis function integrated from Osprey
- coherence_filtered_basis_set.mat when running the coherence pathway-filtered simulation

## How to apply

Configure the spin-system and sequence definitions by editing simMRS.json: set metabolites, FieldStr (3T only), vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), mega_or_hadam (UnEdited/MEGA/HERMES/HERCULES), localization (PRESS/sLASER), TE (68 ms fixed for HERMES, 80 ms for HERCULES), editTp (14 ms for MEGA-PRESS/HERMES, 20 ms for HERCULES), and spatial_points (41 acceptable, 101 ideal). Keep the private parameters unchanged, set the output directories (work_dir not required for MATLAB), then run the run_simulations_cloud.m entry script, which propagates the density matrix through each slice-selective RF pulse while applying a P-matrix coherence order projector that zeroes density-matrix elements of undesired coherence order (retaining the -1 single-quantum detection pathway) instead of full EXORCYCLE phase cycling, and collapses the spatial loop via 1D projection averaging across each dimension after each 1D frequency-selective pulse. Rationale: these accelerations plus pre-calculation of propagators reduce a full PRESS basis set to approximately 11 minutes on the server while preserving fidelity, validated by ICCs of at least 0.98 versus FID-A and at least 0.96 versus MARSS. Higher spatial-point counts increase runtime substantially (e.g. Cystat, HCar, iLe, Lys, and especially Glc with complex J-coupling spin systems), so choose 101 points for final-quality basis sets and 41 for exploratory runs.

## Related tools

- **MRSCloud** (Primary density-matrix simulation tool (MATLAB) with 1D projection, coherence pathway filters, and pre-calculated propagators; run via run_simulations_cloud.m) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying spectral simulation functionality MRSCloud was built upon; also used for EXORCYCLE phase-cycled reference spectra) — https://github.com/CIC-methods/FID-A
- **MARSS** (Independent generic simulation package used as a cross-validation reference for the simulated basis functions)
- **Osprey** (Provides the plot basis function used to generate the QA PDF of downloaded basis sets)

## Examples

```
matlab -batch "run_simulations_cloud.m"  % after editing simMRS.json: spatial_points=101, editTp=14 (MEGA), localization=PRESS, TE=80
```

## Evaluation signals

- ICCs between MRSCloud and FID-A basis spectra reach at least 0.98; ICCs between MRSCloud and MARSS reach at least 0.96
- ICCs among spatial resolutions (21x21, 41x41, 101x101) of edited GABA signals exceed 0.99 for MEGA, HERMES, and HERCULES in both PRESS and sLASER localization
- Coherence-filtered spectra match EXORCYCLE phase-cycled FID-A reference with ICC >= 0.98 at substantially reduced runtime
- A full PRESS basis set simulates in approximately 11 minutes on the server; excessive runtime suggests misconfigured spatial_points or complex spin systems (Glc, Cystat, HCar, iLe, Lys)
- Each downloaded zipped basis set includes a PDF spectra plot (Osprey-derived) for visual quality assurance

## Limitations

- Ease of use is favored over flexibility by limiting the number of user-specified parameters
- Simulations cannot capture all possible vendor experiments (e.g. Philips hardware/protocol variations) or all disseminated versions of MEGA-PRESS within each vendor
- Spin-system parameters rest largely on Govindaraju et al. with only minor updates
- Complex J-coupling spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time; Glc takes extremely long
- Product sequences and RF waveforms are not shared in the GitHub repo

## Evidence
