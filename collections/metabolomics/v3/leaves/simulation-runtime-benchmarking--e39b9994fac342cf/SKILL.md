---
name: simulation-runtime-benchmarking
description: Use when an MRS basis-set simulation pipeline has been accelerated (e.g.
  coherence filtering instead of EXORCYCLE phase cycling) and you need to quantify
  runtime savings while demonstrating spectral equivalence against a reference implementation.
when_to_use_negative:
- You need a validated basis set for routine LCModel/analysis use — the phantom-measured
  LCModel basis set (e.g. press_te30_3t_01a.basis) is the reference there, and simulated-vs-acquired
  ICCs can drop to 0.68 (Gln)
- The spin system is highly complex (e.g. Glc, Cystat, HCar, iLe, Lys) — simulation
  time increases dramatically, confounding a runtime benchmark
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MRSCloud
  role: 'Accelerated simulation being benchmarked: coherence order filtering, 1D projection
    method, and pre-calculation of propagators'
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Produces the traditional four-step EXORCYCLE phase-cycled reference spectra
    against which ICCs and runtime are compared
  repo: https://github.com/CIC-methods/FID-A
- name: R / RStudio
  role: 'Statistical analysis: computing intraclass correlation coefficients between
    simulated basis spectra'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:de730341d8009e29a25864142a94da6b4404716f126bf60f3e30bf688cd86380
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MRSCloud
  - R / RStudio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# simulation-runtime-benchmarking

## Summary

Benchmarks wall-clock runtime of an accelerated density-matrix MRS basis-set simulation (coherence pathway filtering, 1D projection, propagator pre-calculation) against the traditional full phase-cycled simulation, validating equivalence with ICCs.

## When to use

Use when an MRS basis-set simulation pipeline has been accelerated (e.g. coherence filtering instead of EXORCYCLE phase cycling) and you need to quantify runtime savings while demonstrating spectral equivalence against a reference implementation.

## When NOT to use

- You need a validated basis set for routine LCModel/analysis use — the phantom-measured LCModel basis set (e.g. press_te30_3t_01a.basis) is the reference there, and simulated-vs-acquired ICCs can drop to 0.68 (Gln)
- The spin system is highly complex (e.g. Glc, Cystat, HCar, iLe, Lys) — simulation time increases dramatically, confounding a runtime benchmark

## Inputs

- Spin-system and pulse-sequence definitions from the shui5/MRSCloud repository (simMRS.json configuration: vendor, PRESS/sLASER, TE, editTp, spatial_points)
- FID-A toolbox for the traditional EXORCYCLE phase-cycled reference simulation

## Outputs

- coherence_filtered_basis_set.mat — filtered basis functions
- Comparison table of ICCs (per metabolite) and wall-clock runtimes for filtered vs phase-cycled simulations

## How to apply

Run the accelerated simulation in MRSCloud (MATLAB entry script run_simulations_cloud.m) with coherence pathway filtering applied after each RF pulse — construct a P-matrix of coherence orders p = m_i - m_j and zero all density-matrix elements of undesired coherence order, retaining the -1 single-quantum detection pathway. Set spatial_points = 41 (101 ideal for final quality) and editTp = 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES, saving the filtered basis to coherence_filtered_basis_set.mat. Generate the reference spectra via the traditional four-step EXORCYCLE phase-cycled simulation in FID-A, record the wall-clock runtime of both approaches, and compute intraclass correlation coefficients (ICCs) between the filtered and phase-cycled basis spectra for each metabolite. Accept the accelerated approach when ICCs reach at least 0.98 and the runtime reduction is substantial.

## Related tools

- **MRSCloud** (Accelerated simulation being benchmarked: coherence order filtering, 1D projection method, and pre-calculation of propagators) — https://github.com/shui5/MRSCloud
- **FID-A** (Produces the traditional four-step EXORCYCLE phase-cycled reference spectra against which ICCs and runtime are compared) — https://github.com/CIC-methods/FID-A
- **R / RStudio** (Statistical analysis: computing intraclass correlation coefficients between simulated basis spectra)

## Examples

```
matlab -r "addpath(genpath('MRSCloud')); set_output_directories(); run_simulations_cloud"
```

## Evaluation signals

- ICCs between coherence-filtered MRSCloud and FID-A EXORCYCLE spectra reach at least 0.98 for each metabolite
- ICCs between MRSCloud and MARSS are at least 0.96 (secondary cross-implementation check)
- Wall-clock runtime of the filtered approach shows a substantial reduction versus full phase cycling (full PRESS basis set reference point: ~11 minutes on the server)
- Basis sets simulated with spatial_points = 101 match the spatial_points = 41 runtime-optimized runs when reviewed via the QA PDF of spectra included in the download

## Limitations

- Higher spatial_points increases runtime (41 acceptable, 101 ideal), so the benchmark must fix spatial resolution to be a fair comparison
- Complex spin systems (Cystat, HCar, iLe, Lys, Glc) significantly increase simulation time, limiting measured speedups
- Simulations cannot capture all vendor-specific parameter sets or differences between disseminated versions of MEGA-PRESS within each vendor
- MRSCloud's ease-of-use bias limits user-specified parameters, constraining which sequences can be benchmarked

## Evidence
