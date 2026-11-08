---
name: cidmd-collider-trajectory-sampling
description: Use when an optimized protonated molecular ion structure (xyz format)
  must be converted into a user-defined number of collision input files with varied
  collisional geometry before running CIDMD trajectories.
when_to_use_negative:
- The molecule has not yet been protonated and geometry-optimized at unrestricted
  B3LYP/6–31G* — the setup requires an optimized molecular ion xyz as input.
- Post-processing or spectral comparison is the goal — trajectories must first be
  run in TeraChem and processed with LearnReactions.py; CIDMD_analysis and CIDMD_compare
  handle downstream analysis instead.
- Only a single fixed collision geometry is acceptable — the sampling deliberately
  varies collisional directions; a deterministic single-collision setup would not
  reproduce the multi-trajectory CID spectrum.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0154
tools:
- name: CIDMD_setup
  role: Samples collider initial positions and directions, places the argon collider
    5 Å from the molecular envelope, and writes start.xyz/vel.xyz per trajectory folder.
  repo: https://github.com/jesilee/CIDMD_setup
- name: Avogadro
  role: Manual construction of the protonated molecular ion structure before optimization.
- name: TeraChem
  role: Optimizes the ion 3D shape at unrestricted B3LYP/6–31G* and later runs the
    seeded NVE MD trajectories.
- name: lebedev.py
  role: Provides Lebedev quadrature grid directions for the deterministic collider-position
    algorithm.
  repo: https://github.com/jesilee/CIDMD_setup
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:9e2af968bf5c66c7af85358197570eac3f2987af3e135b13b241e602295047ff
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_setup
  tools:
  - 6–31G*
  - Avogadro
  - B3LYP
  - CIDMD
  - CIDMD_setup
  - TeraChem
  - density functional theory
  - lebedev.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cidmd-collider-trajectory-sampling

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Generates initial collision trajectories for CIDMD by sampling argon collider gas positions and directions around an optimized protonated molecular ion, producing per-trajectory start.xyz and vel.xyz files that seed NVE ab initio MD in TeraChem.

## When to use

Use when an optimized protonated molecular ion structure (xyz format) must be converted into a user-defined number of collision input files with varied collisional geometry before running CIDMD trajectories.

## When NOT to use

- The molecule has not yet been protonated and geometry-optimized at unrestricted B3LYP/6–31G* — the setup requires an optimized molecular ion xyz as input.
- Post-processing or spectral comparison is the goal — trajectories must first be run in TeraChem and processed with LearnReactions.py; CIDMD_analysis and CIDMD_compare handle downstream analysis instead.
- Only a single fixed collision geometry is acceptable — the sampling deliberately varies collisional directions; a deterministic single-collision setup would not reproduce the multi-trajectory CID spectrum.

## Inputs

- Optimized protonated molecular ion structure in xyz file format
- mol_info.in text file describing the molecule (mol_name, mol_id, mol_mf, mol_mw, mol_xw)
- CIDMD_setup.com variable settings (extent, cutoff, set_distance, n_points, seed, deterministic/cartgrid/lebedev options, v_scale)

## Outputs

- User-defined number of CIDMD input files start.xyz and vel.xyz per trajectory, varying collisional geometry, written into subfolders cid/calcs/XXX/chunk_0000/
- Collision trajectories ready for NVE ab initio MD (ub3lyp/6-31gs, 1.0 fs timestep, nstep 1000) in TeraChem

## How to apply

Build the protonated ion at the chosen protonation site in Avogadro and optimize its 3D geometry with TeraChem at unrestricted B3LYP/6–31G*. Clone CIDMD_setup and place CIDMD_setup.com, CIDMD_setup.py, CIDMD_preprun.py and lebedev.py in the working directory, then set variables (extent, cutoff, set_distance, n_points, v_scale, optional --deterministic with --cartgrid and --lebedev) in CIDMD_setup.com and run `bash CIDMD_setup.com`. The script samples initial collider positions either randomly inside a cubic box (default half-edge 50 Å, extent) accepting points whose straight-line path approaches any atom within the b cutoff (default 2.0 Å), or deterministically from a Cartesian grid with directions from a Lebedev quadrature grid; each accepted collider is placed back 5 Å (set_distance) from its point of closest approach to the molecular envelope. The molecular ion keeps zero velocity while the collider's velocity is the direction vector scaled by the user-specified v_scale (AKMA units), written as vel.xyz. Each trajectory folder cid/calcs/XXX/chunk_0000/ must contain start.xyz (ion plus collider coordinates) and vel.xyz (ion zero velocity plus collider velocity); validate that the user-defined number of accepted trajectories each carry both files with the collider 5 Å from the molecular envelope and velocities at the chosen v_scale.

## Related tools

- **CIDMD_setup** (Samples collider initial positions and directions, places the argon collider 5 Å from the molecular envelope, and writes start.xyz/vel.xyz per trajectory folder.) — https://github.com/jesilee/CIDMD_setup
- **Avogadro** (Manual construction of the protonated molecular ion structure before optimization.)
- **TeraChem** (Optimizes the ion 3D shape at unrestricted B3LYP/6–31G* and later runs the seeded NVE MD trajectories.)
- **lebedev.py** (Provides Lebedev quadrature grid directions for the deterministic collider-position algorithm.) — https://github.com/jesilee/CIDMD_setup

## Examples

```
bash CIDMD_setup.com && find calcs -name start.xyz -execdir ln -fs ../../../run.in . \;
```

## Evaluation signals

- Each accepted trajectory folder cid/calcs/XXX/chunk_0000/ contains both a start.xyz and a vel.xyz file.
- In every start.xyz the collider is positioned 5 Å (set_distance default) from the molecular envelope, i.e. set back from its point of closest approach.
- In every vel.xyz the molecular ion velocity is zero and the collider velocity equals the direction vector scaled by the user-specified v_scale in AKMA units.
- The number of generated trajectory folders equals the user-defined n_points (randomized) or the cartgrid/lebedev count (deterministic).
- Randomly accepted initial positions satisfy the b cutoff: the collider's straight-line path comes to within the cutoff distance (default 2.0 Å) of at least one atom of the molecular ion.

## Limitations

- Randomized sampling relies on a random seed (default 2021); different seeds yield different accepted trajectory sets, and rejection sampling may need many candidate points if the molecular envelope is small.
- The Lebedev grid order is limited to odd numbers in [3,31]; higher orders are not recommended, so angular coverage in deterministic mode is bounded.
- An automatic multiple-collision method has not yet been implemented; each trajectory simulates a single collision, and summing many trajectories only partially accounts for experimental multiple-collision conditions.
- It remains unclear how collider trajectory characteristics (b cutoff and velocity) relative to the target molecule's location and vibrational state affect energy-transfer efficiency, so parameter choices (e.g. v_scale, b cutoff) are empirical.

## Evidence
