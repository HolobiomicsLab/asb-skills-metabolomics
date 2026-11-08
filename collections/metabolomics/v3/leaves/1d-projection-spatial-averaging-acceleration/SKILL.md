---
name: 1d-projection-spatial-averaging-acceleration
description: Use when simulating vendor- and sequence-specific MRS basis sets (PRESS/MEGA-PRESS/HERMES/HERCULES)
  at 3T where a full 3D spatial loop over every voxel location is too slow and voxel-averaged
  FIDs suffice.
when_to_use_negative:
- When the target experiment needs full per-voxel spatial fidelity, not voxel-averaged
  basis functions (the method collapses spatial dimensions by averaging).
- When simulating STEAM or SPECIAL localization — these are not implemented in MRSCloud
  and the 1D projection method as described is built for PRESS/sLASER.
- When the basis set must replicate a specific phantom-measured vendor basis set (e.g.
  press_te30_3t_01a.basis) whose waveforms and timings are not documented — simulation
  cannot capture those differences.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3423
tools:
- name: MRSCloud
  role: Provides the 1D projection method, coherence order filtering, and propagator
    pre-calculation implementation to accelerate basis-set simulation; configured
    via simMRS.json and run_simulations_cloud.m
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Supplies the density-matrix spectral simulation functionality MRSCloud builds
    upon and the full spatial-loop reference simulation used as ground truth for ICC
    comparison
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Independent generic simulator whose metabolite basis functions are compared
    against the 1D-projection-accelerated MRSCloud outputs (ICC >= 0.96)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:a5df7941e837a188183a4787bdfe39f2dd5c3f039d98324d5a76bd0d51db41db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - MARSS
  - MRSCloud
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3423
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# 1d-projection-spatial-averaging-acceleration

## Summary

Accelerates spatially resolved density-matrix simulation of PRESS/sLASER localization by treating the excitation pulse as ideal and collapsing the density matrix via averaging along one spatial dimension after each slice-selective refocusing pulse, so the spatial loop shrinks from ny x nz to (ny + nz)/2 operations while keeping ICC >= 0.98 versus full-loop FID-A.

## When to use

Use when simulating vendor- and sequence-specific MRS basis sets (PRESS/MEGA-PRESS/HERMES/HERCULES) at 3T where a full 3D spatial loop over every voxel location is too slow and voxel-averaged FIDs suffice.

## When NOT to use

- When the target experiment needs full per-voxel spatial fidelity, not voxel-averaged basis functions (the method collapses spatial dimensions by averaging).
- When simulating STEAM or SPECIAL localization — these are not implemented in MRSCloud and the 1D projection method as described is built for PRESS/sLASER.
- When the basis set must replicate a specific phantom-measured vendor basis set (e.g. press_te30_3t_01a.basis) whose waveforms and timings are not documented — simulation cannot capture those differences.

## Inputs

- Spin-system definitions and simulation settings (simMRS.json) from MRSCloud: metabolite list, vendor (GE/Philips/Siemens/Universal_Philips/Universal_Siemens), mega_or_hadam (UnEdited/MEGA/HERMES/HERCULES), localization (PRESS/sLASER), TE, editOn/editOff, editTp, spatial_points
- Sequence RF pulse waveforms and timings (product sequence waveforms are not shared in the repo)
- Full 3D spatial-loop FID-A density-matrix reference FID for the selected metabolite

## Outputs

- Voxel-averaged metabolite basis function FID generated via the 1D projection method
- 1d_projection_validation.csv with ICC (vs FID-A reference) and runtime comparison
- Coherence-filtered simulated spectra per metabolite (e.g. NAA, Gln, GABA)

## How to apply

Load spin-system definitions and simMRS.json settings from the shui5/MRSCloud repo, setting spatial_points to 101 (41 acceptable) and editTp to 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES. Run a reference full spatial-loop density-matrix propagation in FID-A style to obtain the ground-truth voxel-averaged FID for the metabolite of interest. Re-implement with the 1D projection method: approximate the excitation pulse as ideal, and after each slice-selective refocusing pulse collapse the density matrix by averaging along the excited dimension, reducing simulations to the factor equivalent to the number of spatial points. Apply coherence order filtering so only the desired coherence pathway is retained, avoiding full phase-cycling simulation, and pre-calculate propagators to replace repeated time-evolution-operator calculations per pulse. Compute the ICC of the accelerated voxel-averaged basis function against the FID-A full-loop reference; accept only if ICC >= 0.98.

## Related tools

- **MRSCloud** (Provides the 1D projection method, coherence order filtering, and propagator pre-calculation implementation to accelerate basis-set simulation; configured via simMRS.json and run_simulations_cloud.m) — https://github.com/shui5/MRSCloud
- **FID-A** (Supplies the density-matrix spectral simulation functionality MRSCloud builds upon and the full spatial-loop reference simulation used as ground truth for ICC comparison) — https://github.com/CIC-methods/FID-A
- **MARSS** (Independent generic simulator whose metabolite basis functions are compared against the 1D-projection-accelerated MRSCloud outputs (ICC >= 0.96))

## Examples

```
Edit simMRS.json (spatial_points=101, editTp=14, mega_or_hadam='MEGA', localization='PRESS') then run: run_simulations_cloud.m
```

## Evaluation signals

- ICC between the accelerated voxel-averaged FID and the full spatial-loop FID-A reference is at least 0.98
- Runtime is substantially reduced relative to the full-loop simulation (a full PRESS basis set takes ~11 minutes on the server even accelerated; the 1D projection reduces the number of simulations to the factor equivalent to the number of spatial points)
- Coherence order filter retains only the desired coherence pathway — output spectrum should match the expected product coherence pathway without phase-cycling artifacts
- Spatial parameters used: spatial_points = 101 (or 41 minimally), editTp = 14 ms (MEGA-PRESS/HERMES) or 20 ms (HERCULES), fixed TE = 68/80 ms for HERMES/HERCULES
- Generated basis spectra are visually consistent with FID-A/MARSS comparisons and vendor phantom-measured sets (ICCs vs MARSS >= 0.96; vs LCModel measured set ranging 0.68 for Gln to 0.96 for NAA)

## Limitations

- MRSCloud supports only 3T field strength and a limited subset of experiments; STEAM and SPECIAL localization are not implemented.
- The simulation corresponds to the most commonly used parameter sets and cannot capture all Philips experiments or all disseminated MEGA-PRESS versions within each vendor.
- Complex J-coupled metabolites (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time even with the acceleration.
- Product sequence RF waveforms are not shared in the GitHub repo; higher spatial_points values increase runtime.
- Spin-system parameters rest largely on Govindaraju et al. with only minor updates.

## Evidence

- [discussion] 1D projection method reduces number of spatial simulations: "The 1D projection method accelerates the process by decreasing the number of simulations to the factor equivalent to the number of spatial points."
- [discussion] Coherence order filtering avoids full phase cycling: "Coherence order filtering helps the selection of appropriate coherence pathway to avoid the necessity to simulate full phase cycling"
- [abstract] ICC acceptance threshold vs FID-A: "ICCs were at least 0.98 between MRSCloud and FID-A and were at least 0.96 between MRSCloud and MARSS"
- [readme] editTp parameter configuration: "editTp is the duration of the editing pulses. It is normally 14 ms for MEGA-PRESS/HERMES and 20 ms for HERCULES."
- [readme] spatial_points configuration trade-off: "spatial_points: 41 is acceptable and 101 is ideal. The higher the number of spatial points, the longer it takes for the simulation."
- [discussion] STEAM/SPECIAL not implemented: "MRSCloud also simulates a limited subset of experiments and the inclusion of e.g. STEAM or SPECIAL localization would be a valuable future addition."
