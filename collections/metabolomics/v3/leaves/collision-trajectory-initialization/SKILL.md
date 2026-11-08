---
name: collision-trajectory-initialization
description: Use when you have an optimized protonated molecular ion (.xyz) for CIDMD
  and need to set up collision trajectories — placing argon colliders and defining
  velocities before running NVE ab initio MD in TeraChem.
when_to_use_negative:
- Simulations that require multiple successive collisions within one trajectory —
  the article notes 'In CIDMD, an automatic multiple collision method has not yet
  been implemented yet.'
- Anions, radicals, or non-protonated adducts — the framework constructs protonated
  molecular ions at specific protonation sites, and prediction accuracy decreases
  as the number of protomers/tautomers increases
- Very large metabolites where unrestricted B3LYP/6–31G* NVE MD of ≥1000 steps is
  computationally infeasible
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3339
tools:
- name: CIDMD_setup
  role: Generates the argon collider initial positions and velocities (start.xyz,
    vel.xyz) for each accepted trajectory
  repo: https://github.com/jesilee/CIDMD_setup
- name: TeraChem
  role: Executes the initialized NVE ab initio MD collision trajectories at unrestricted
    B3LYP/6–31G* with 1.0 fs timestep
- name: Avogadro
  role: Used beforehand to manually construct the protonated molecular ion structures
    at candidate protonation sites that this skill initializes collisions onto
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:0d3c1d1f9e90cd0f2919553a2c92b1d9ed809da1b5012f68791f76d976485820
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_setup
  tools:
  - 6–31G*
  - Avogadro
  - B3LYP
  - CIDMD_setup
  - TeraChem
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# collision-trajectory-initialization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Generates collisional initial conditions for CID-MS/MS prediction: an argon collider atom is placed on the molecular envelope of an optimized protonated ion and assigned a velocity (vfac 1–12 AKMA), producing start.xyz/vel.xyz trajectory inputs for TeraChem NVE ab initio MD.

## When to use

Use when you have an optimized protonated molecular ion (.xyz) for CIDMD and need to set up collision trajectories — placing argon colliders and defining velocities before running NVE ab initio MD in TeraChem.

## When NOT to use

- Simulations that require multiple successive collisions within one trajectory — the article notes 'In CIDMD, an automatic multiple collision method has not yet been implemented yet.'
- Anions, radicals, or non-protonated adducts — the framework constructs protonated molecular ions at specific protonation sites, and prediction accuracy decreases as the number of protomers/tautomers increases
- Very large metabolites where unrestricted B3LYP/6–31G* NVE MD of ≥1000 steps is computationally infeasible

## Inputs

- Optimized protonated molecular ion structure in .xyz format (built in Avogadro, optimized with TeraChem at unrestricted B3LYP/6–31G*)
- mol_info.in file with molecule metadata (mol_name, mol_id, mol_mf, mol_mw, mol_xw)
- CIDMD_setup.com variable settings (b cutoff, set_distance, n_points, lebedev/cartgrid mode, v_scale)

## Outputs

- start.xyz and vel.xyz pairs per trajectory in cid/calcs/XXX/chunk_0000/ subfolders
- run.in TeraChem input files for NVE MD (ub3lyp/6-31gs, 1.0 fs timestep, ≥1000 steps)
- A user-defined number of collision trajectories varying collisional geometry for fragmentation pathway analysis

## How to apply

Place the argon collider at the user-defined atomic radius of the molecular envelope (set_distance, default 5.0 Å from the point of closest approach) using CIDMD_setup.py in randomized mode, requesting n_points initial conditions (50–400 per the article) and accepting only trajectories whose closest approach falls within the b cutoff (0.2–5 Å). Assign the collider velocity via v_scale/vfac (1–12 AKMA units), scaling velocities upward for heavier molecules since higher collider velocities achieved better similarity scores with increasing molecular mass. Each accepted condition is written as a start.xyz (molecular ion plus Ar coordinates) and vel.xyz (ion velocity default 0, collider velocity) pair under cid/calcs/XXX/chunk_0000/, with a run.in that sets charge 1, spinmult 1, run md, method ub3lyp, basis 6-31gs, timestep 1.0, and nstep 1000 for the NVE ensemble. The rationale is to sample multiple collisional directions so fragmentations can later be accumulated into an in silico mass spectrum.

## Related tools

- **CIDMD_setup** (Generates the argon collider initial positions and velocities (start.xyz, vel.xyz) for each accepted trajectory) — https://github.com/jesilee/CIDMD_setup
- **TeraChem** (Executes the initialized NVE ab initio MD collision trajectories at unrestricted B3LYP/6–31G* with 1.0 fs timestep)
- **Avogadro** (Used beforehand to manually construct the protonated molecular ion structures at candidate protonation sites that this skill initializes collisions onto)

## Examples

```
python3 CIDMD_setup.py --set_distance 5.0 --cutoff 5.0 --n_points 50 --v_scale 8 Psilocin.xyz
```

## Evaluation signals

- Each start.xyz contains the user-provided molecular ion coordinates plus exactly one argon atom positioned at the user-defined envelope distance (default 5.0 Å)
- Each vel.xyz carries collider velocities in AKMA units scaled by v_scale, with ion velocity defaulting to 0
- Number of generated calc folders matches the requested n_points and all accepted trajectories satisfy the b cutoff (0.2–5 Å)
- Every calc folder links a run.in with charge 1, spinmult 1, run md, method ub3lyp, basis 6-31gs, timestep 1.0, and nstep ≥ 1000
- Downstream LearnReactions.py and CIDMD_analysis run without missing-input errors on the produced calcs/ directory

## Limitations

- No automatic multiple collision method exists; each trajectory is a single collision, and many angles/trajectories are summed to approximate experimental MS/MS conditions
- It is unclear how collider trajectory characteristics (b cutoff and velocity) relative to the target molecule's location and vibrational state affect energy transfer efficiency
- Accuracy decreases as the number of protomers and tautomers increases; there is no method to predict protomer formation states before running CIDMD simulations

## Evidence
