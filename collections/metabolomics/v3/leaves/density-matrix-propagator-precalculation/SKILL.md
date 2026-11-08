---
name: density-matrix-propagator-precalculation
description: Use when simulating metabolite basis sets for PRESS/sLASER or edited
  sequences (MEGA-PRESS, HERMES, HERCULES) in MRSCloud/FID-A and per-pulse propagator
  recalculation dominates runtime.
when_to_use_negative:
- You need STEAM or SPECIAL localization — these are not implemented in MRSCloud.
- You require exact replication of a vendor's disseminated MEGA-PRESS variant or all
  Philips hardware/protocol parameter sets — the simulation corresponds only to the
  most commonly used parameter set.
- Your basis set must match a phantom-measured vendor basis (e.g., press_te30_3t_01a.basis
  in LCModel), whose waveforms and timings are not well documented and likely differ
  from the simulation.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: MRSCloud
  role: MATLAB implementation that applies 1D projection, coherence pathway filters,
    and propagator pre-calculation to accelerate basis-set simulation
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Elementwise spectral simulation baseline whose per-pulse propagator calculations
    are replaced by the pre-calculation approach
  repo: https://github.com/CIC-methods/FID-A
- name: R
  role: Used in RStudio for ICC-based statistical validation of the accelerated spectra
    against the FID-A baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:839a93a16aab239b712288862daac4bb17f85e4044b7c279bc8e0fa207c76ead
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MRSCloud
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# density-matrix-propagator-precalculation

## Summary

Accelerates density-matrix MRS basis-set simulation by computing each shaped refocusing/editing pulse's propagator exactly once and reusing it across repeated pulse applications, phase cycles, and spatial points, instead of recalculating it elementwise per sample as in FID-A.

## When to use

Use when simulating metabolite basis sets for PRESS/sLASER or edited sequences (MEGA-PRESS, HERMES, HERCULES) in MRSCloud/FID-A and per-pulse propagator recalculation dominates runtime.

## When NOT to use

- You need STEAM or SPECIAL localization — these are not implemented in MRSCloud.
- You require exact replication of a vendor's disseminated MEGA-PRESS variant or all Philips hardware/protocol parameter sets — the simulation corresponds only to the most commonly used parameter set.
- Your basis set must match a phantom-measured vendor basis (e.g., press_te30_3t_01a.basis in LCModel), whose waveforms and timings are not well documented and likely differ from the simulation.

## Inputs

- MRSCloud simulation scripts (run_simulations_cloud.m) with simMRS.json configuration
- Sequence parameters: editTp (14 ms MEGA-PRESS/HERMES or 20 ms HERCULES), spatial_points (41–101), TE, vendor, mega_or_hadam, localization
- Metabolite spin-system definitions (e.g., the 20 common-brain metabolites plus specialty list including Cystat, HCar, iLe, Lys, Glc)
- FID-A elementwise simulation code as the comparison baseline

## Outputs

- Pre-calculated propagator per shaped pulse, reused across coherence pathways, phase cycles, and spatial points
- Simulated metabolite basis functions (full PRESS basis set), comparable to FID-A output
- propagator_benchmark.csv with per-metabolite ICCs and runtimes for both implementations

## How to apply

In run_simulations_cloud.m, compute the density-matrix time-evolution propagator for each shaped refocusing/editing pulse once, then apply that pre-calculated propagator everywhere the pulse recurs — across repeated refocusing/editing pulse applications, coherence pathways/phase cycles, and all spatial points (editTp = 14 ms for MEGA-PRESS/HERMES, 20 ms for HERCULES; spatial_points 41 acceptable, 101 ideal). Combine with the 1D projection method and coherence order filtering so the full phase cycle need not be simulated. Expect the largest speedups for pulses repeated many times, but note that complex J-coupled spin systems (Cystat, HCar, iLe, Lys, and especially Glc) still dominate total simulation time. Validate by computing ICCs in R/RStudio between the pre-calculated-propagator basis spectra and the FID-A elementwise baseline: accept when every metabolite's ICC is at least 0.98 and runtime measurably drops (full PRESS basis set ran ~11 minutes on the MRSCloud server).

## Related tools

- **MRSCloud** (MATLAB implementation that applies 1D projection, coherence pathway filters, and propagator pre-calculation to accelerate basis-set simulation) — https://github.com/shui5/MRSCloud
- **FID-A** (Elementwise spectral simulation baseline whose per-pulse propagator calculations are replaced by the pre-calculation approach) — https://github.com/CIC-methods/FID-A
- **R** (Used in RStudio for ICC-based statistical validation of the accelerated spectra against the FID-A baseline)

## Examples

```
MATLAB: edit simMRS.json (set metabolites, vendor, mega_or_hadam, editTp=14, spatial_points=41), then run >> run_simulations_cloud.m
```

## Evaluation signals

- ICCs between pre-calculated-propagator spectra and the FID-A elementwise baseline are at least 0.98 for every metabolite.
- Wall-clock runtime per metabolite is measurably reduced relative to the elementwise FID-A approach (full PRESS basis set ~11 minutes on the MRSCloud server).
- Each shaped pulse's propagator appears exactly once in the computation log rather than being recalculated for all 256 serial pulse applications/spatial samples.
- Downloaded basis set includes the PDF spectra image (plot basis function from Osprey) for quality assurance.
- Cystat, HCar, iLe, Lys, and Glc show the longest runtimes, confirming complex J-coupled spin systems dominate simulation time even after acceleration.

## Limitations

- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys) significantly increase simulation time; Glc takes extremely long.
- Higher spatial_points values (101 ideal vs 41 acceptable) proportionally increase runtime.
- Product sequences and RF waveforms are not shared in the GitHub repo.
- Only 3T brain MRS has been tested; the FieldStr option is fixed at 3T.
- Spin-system parameters rest largely on Govindaraju et al. with only minor updates.

## Evidence
