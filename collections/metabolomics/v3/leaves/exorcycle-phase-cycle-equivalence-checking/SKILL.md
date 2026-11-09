---
name: exorcycle-phase-cycle-equivalence-checking
description: Use when replacing EXORCYCLE phase cycling with a coherence pathway filter
  in a spatially localized density-matrix MRS simulation (e.g. PRESS/sLASER, MEGA-PRESS,
  HERMES, HERCULES) and you must demonstrate spectral equivalence before adopting
  the faster approach.
when_to_use_negative:
- The goal is to simulate the full EXORCYCLE phase cycle itself (e.g. for reference
  spectra generation) rather than to validate a filtered shortcut
- The pulse sequence relies on coherence orders other than the -1 single-quantum detection
  pathway (e.g. multiple-quantum experiments not supported by MRSCloud)
- Validating against experimentally acquired basis sets (e.g. the LCModel phantom-measured
  press_te30_3t_01a.basis) whose waveforms and timings are not well documented — ICCs
  there can be much lower (0.68 for Gln)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MRSCloud
  role: MATLAB simulation engine applying the P-matrix coherence pathway filter and
    1D projection method to generate filtered basis functions
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Produces the traditional EXORCYCLE four-step phase-cycled reference basis
    spectra against which the filtered spectra are compared
  repo: https://github.com/CIC-methods/FID-A
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:d4947c7b96a4743c9f8f6c26e11e2df5d1b12d61d667aa704645fdd4fa64a62c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MRSCloud
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# exorcycle-phase-cycle-equivalence-checking

## Summary

Validates that coherence-order filtering of density-matrix MRS simulations reproduces traditional EXORCYCLE four-step phase cycling, by computing intraclass correlation coefficients (ICCs) between filtered and phase-cycled metabolite basis spectra and comparing wall-clock runtimes.

## When to use

Use when replacing EXORCYCLE phase cycling with a coherence pathway filter in a spatially localized density-matrix MRS simulation (e.g. PRESS/sLASER, MEGA-PRESS, HERMES, HERCULES) and you must demonstrate spectral equivalence before adopting the faster approach.

## When NOT to use

- The goal is to simulate the full EXORCYCLE phase cycle itself (e.g. for reference spectra generation) rather than to validate a filtered shortcut
- The pulse sequence relies on coherence orders other than the -1 single-quantum detection pathway (e.g. multiple-quantum experiments not supported by MRSCloud)
- Validating against experimentally acquired basis sets (e.g. the LCModel phantom-measured press_te30_3t_01a.basis) whose waveforms and timings are not well documented — ICCs there can be much lower (0.68 for Gln)

## Inputs

- Spin-system and pulse-sequence definitions (e.g. from the shui5/MRSCloud MATLAB repository, run via run_simulations_cloud.m)
- Sequence parameters from simMRS.json: metabolite list, vendor (GE/Philips/Siemens/Universal variants), localization (PRESS/sLASER), TE, editTp, spatial_points, editOn/editOff
- EXORCYCLE four-step phase-cycled reference basis spectra simulated in FID-A

## Outputs

- Coherence-filtered basis functions saved to coherence_filtered_basis_set.mat
- Comparison table of ICCs between coherence-filtered and EXORCYCLE phase-cycled basis spectra per metabolite
- Wall-clock runtime comparison of filtered vs. phase-cycled simulations

## How to apply

Construct a P-matrix that classifies every density-matrix element by its coherence order p = m_i - m_j, then after each RF pulse zero all entries whose coherence order does not match the desired pathway, retaining the -1 single-quantum coherence detection pathway in place of full EXORCYCLE phase cycling. Run the filtered simulation with the sequence parameters used by MRSCloud (spatial_points = 41, or 101 for final quality; editTp = 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES) and save the filtered basis functions. Generate reference spectra via a traditional EXORCYCLE four-step phase-cycled simulation in FID-A, then compute intraclass correlation coefficients between filtered and phase-cycled basis spectra for each metabolite and record the wall-clock runtime of both approaches. Accept the filtered approach only when ICCs reach at least 0.98 while showing a substantial runtime reduction.

## Related tools

- **MRSCloud** (MATLAB simulation engine applying the P-matrix coherence pathway filter and 1D projection method to generate filtered basis functions) — https://github.com/shui5/MRSCloud
- **FID-A** (Produces the traditional EXORCYCLE four-step phase-cycled reference basis spectra against which the filtered spectra are compared) — https://github.com/CIC-methods/FID-A

## Examples

```
Modify simMRS.json (vendor, localization=PRESS, TE, editTp=14, spatial_points=41) then run: run_simulations_cloud.m
```

## Evaluation signals

- Intraclass correlation coefficients between coherence-filtered and FID-A EXORCYCLE basis spectra reach at least 0.98 for each metabolite
- Filtered simulation shows a substantial runtime reduction relative to the full phase-cycled simulation
- P-matrix correctly partitions every density-matrix element by coherence order p = m_i - m_j and zeros only undesired-order entries
- Filtered basis functions are saved to coherence_filtered_basis_set.mat and match the configured spatial_points (41 acceptable, 101 ideal) and editTp (14 ms MEGA-PRESS/HERMES, 20 ms HERCULES) settings

## Limitations

- Product sequences and RF waveforms are not shared in the GitHub repository, limiting replication of exact vendor sequences
- The simulation corresponds to the most commonly used parameter set and cannot capture all vendor/hardware variations or all disseminated versions of MEGA-PRESS within each vendor
- MRSCloud supports only 3T basis sets and a limited subset of experiments (STEAM and SPECIAL localization are not implemented)
- Complex spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time, which affects runtime comparison results

## Evidence
