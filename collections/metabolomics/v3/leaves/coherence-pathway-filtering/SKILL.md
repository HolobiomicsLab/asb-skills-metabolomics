---
name: coherence-pathway-filtering
description: Use when simulating MRS metabolite basis functions (e.g. PRESS, MEGA-PRESS,
  HERMES, HERCULES at 3T) with density-matrix propagation and you need to avoid simulating
  the full four-step phase cycle for each pulse.
when_to_use_negative:
- The localization you need is STEAM or SPECIAL — MRSCloud only implements PRESS and
  sLASER, and the article notes STEAM/SPECIAL would be a valuable future addition.
- A non-3T field strength is required — the exiting version of MRSCloud only supports
  generation of basis sets at 3T.
- You must faithfully reproduce phase-cycling-dependent artifacts or pulse-phase sensitivities
  rather than an idealized single-coherence-pathway simulation.
edam_operation: http://edamontology.org/operation_0438
tools:
- name: MRSCloud
  role: Implements the P-matrix coherence-order filter that zeroes undesired coherence-order
    density-matrix elements, replacing EXORCYCLE phase cycling during spin-echo basis-set
    simulation
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Provides the elementwise density-matrix baseline with traditional EXORCYCLE
    phase cycling used as the reference spectra for ICC validation
  repo: https://github.com/CIC-methods/FID-A
- name: R
  role: Used with RStudio to compute intraclass correlation coefficients between filtered
    and phase-cycled basis spectra
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:2caa03fb09e873f1b0d8113312bf76d3350df518d9785091fce2b59221144c28
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MARSS
  - MRSCloud
  - R
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# coherence-pathway-filtering

## Summary

A density-matrix acceleration technique from MRSCloud that replaces full EXORCYCLE phase cycling by classifying every density-matrix element into a coherence-order P-matrix and zeroing entries outside the desired coherence pathway (the -1 single-quantum detection pathway), substantially reducing runtime for spatially localized spin-echo basis-set simulations.

## When to use

Use when simulating MRS metabolite basis functions (e.g. PRESS, MEGA-PRESS, HERMES, HERCULES at 3T) with density-matrix propagation and you need to avoid simulating the full four-step phase cycle for each pulse.

## When NOT to use

- The localization you need is STEAM or SPECIAL — MRSCloud only implements PRESS and sLASER, and the article notes STEAM/SPECIAL would be a valuable future addition.
- A non-3T field strength is required — the exiting version of MRSCloud only supports generation of basis sets at 3T.
- You must faithfully reproduce phase-cycling-dependent artifacts or pulse-phase sensitivities rather than an idealized single-coherence-pathway simulation.

## Inputs

- Spin-system definitions and pulse-sequence parameters from shui5/MRSCloud (simMRS.json configuration)
- Sequence parameters: spatial_points (41 acceptable, 101 ideal), editTp (14 ms for MEGA-PRESS/HERMES, 20 ms for HERCULES), vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), localization (PRESS/sLASER)
- FID-A EXORCYCLE four-step phase-cycled simulation output as the reference basis spectra

## Outputs

- Coherence-filtered basis functions saved to coherence_filtered_basis_set.mat
- Per-metabolite ICCs between coherence-filtered and EXORCYCLE phase-cycled spectra
- Runtime comparison table for filtered vs. phase-cycled approaches

## How to apply

Load the spin-system and pulse-sequence definitions from the shui5/MRSCloud repository and run the MATLAB entry script run_simulations_cloud.m (work_dir is not required for MATLAB). Construct a P-matrix coherence-order projector that partitions every density-matrix element by p = m_i - m_j, and after each RF pulse zero all density-matrix elements of undesired coherence order, retaining only the -1 single-quantum coherence detection pathway in place of full EXORCYCLE phase cycling. Combine the filter with the other MRSCloud accelerations — the 1D projection method and pre-calculation of pulse propagators — using spatial_points = 41 (101 ideal for final quality) and editTp = 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES. Generate the reference by running a traditional EXORCYCLE four-step phase-cycled simulation in FID-A, then compute intraclass correlation coefficients (ICCs) per metabolite in R/RStudio and compare wall-clock runtimes.

## Related tools

- **MRSCloud** (Implements the P-matrix coherence-order filter that zeroes undesired coherence-order density-matrix elements, replacing EXORCYCLE phase cycling during spin-echo basis-set simulation) — https://github.com/shui5/MRSCloud
- **FID-A** (Provides the elementwise density-matrix baseline with traditional EXORCYCLE phase cycling used as the reference spectra for ICC validation) — https://github.com/CIC-methods/FID-A
- **R** (Used with RStudio to compute intraclass correlation coefficients between filtered and phase-cycled basis spectra)

## Examples

```
matlab -batch "addpath(genpath('MRSCloud')); run_simulations_cloud;"  % after editing simMRS.json: localization=PRESS, spatial_points=101, editTp=14
```

## Evaluation signals

- ICCs between the coherence-filtered and FID-A EXORCYCLE spectra reach at least 0.98 for every metabolite in the basis set
- The filtered simulation exhibits a substantial runtime reduction relative to the four-step phase-cycled reference
- The P-matrix projector correctly retains only the -1 single-quantum coherence detection pathway after each RF pulse (all other entries zeroed)
- Filtered basis functions generated with spatial_points = 101 match those from spatial_points = 41 in spectral shape, with 101 preferred for final quality
- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) still complete without phase-cycling overhead, confirming the filter eliminated full phase-cycle simulation

## Limitations

- Product sequences and RF waveforms are not shared in the GitHub repo, so the filter cannot be validated against exact vendor pulse shapes for all vendors.
- Spin-system parameters rest largely on the work of Govindaraju et al. with only minor updates, so filtered basis functions inherit those parameter uncertainties.
- MRSCloud's simulations correspond to the most commonly used parameter sets and cannot accurately capture all possible vendor experiments or all disseminated versions of MEGA-PRESS within each vendor.
- Very complex spin systems such as Glc (glucose) take extremely long to simulate even with the coherence filter applied.

## Evidence

- [results] P-matrix mechanics: [evidence span withheld — anchor status: not_found]
