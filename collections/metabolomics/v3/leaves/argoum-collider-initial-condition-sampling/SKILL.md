---
name: argoum-collider-initial-condition-sampling
description: Use when you have an optimized protonated molecular ion (.xyz) and need
  collisional starting geometries/velocities for CID-MS/MS in silico spectra via ab
  initio MD.
when_to_use_negative:
- 'You need multi-collision trajectories: CIDMD models a single collision per trajectory;
  an automatic multiple-collision method has not yet been implemented'
- The molecular ion geometry is unoptimized or the protomer/charge site is undetermined
  — protomer choice strongly changes fragmentation and no method predicts protomer
  formation states a priori
- The target exceeds the small-molecule regime validated here (benchmark metabolites
  <205 Da); computational cost grows rapidly beyond that scope
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: CIDMD_setup
  role: Generates collider initial positions, b-cutoff-filtered trajectories, and
    velocity files (start.xyz, vel.xyz) for each sampled collision geometry
  repo: https://github.com/jesilee/CIDMD_setup
- name: TeraChem
  role: Executes the resulting trajectories as unrestricted B3LYP/6–31G* NVE ab initio
    MD with 1.0 fs timestep, recording bond order lists
- name: CIDMD_analysis
  role: Downstream analysis of the sampled trajectories into theoretical mass spectra
    (cidmd.jdx, cidmd.msp)
  repo: https://github.com/jesilee/CIDMD_analysis
- name: nanoreactor (LearnReactions.py)
  role: Post-processing of trajectories from the sampled initial conditions to extract
    fragmentation reactions before spectrum generation
  repo: https://github.com/leeping/nanoreactor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:17d4f0ed4d2d29ce011a5058a7331a02909db9b5ee817e94bae8606378941dba
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_setup
  tools:
  - 6–31G*
  - Avogadro
  - B3LYP
  - CIDMD_analysis
  - CIDMD_setup
  - TeraChem
  - density functional theory
  - nanoreactor (LearnReactions.py)
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# argoum-collider-initial-condition-sampling

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Generates collision initial conditions for CIDMD by automatically placing an argon collider atom on a molecular-envelope surface and sampling impact geometries (b cutoff, direction, velocity scale) before ab initio NVE MD in TeraChem.

## When to use

Use when you have an optimized protonated molecular ion (.xyz) and need collisional starting geometries/velocities for CID-MS/MS in silico spectra via ab initio MD.

## When NOT to use

- You need multi-collision trajectories: CIDMD models a single collision per trajectory; an automatic multiple-collision method has not yet been implemented
- The molecular ion geometry is unoptimized or the protomer/charge site is undetermined — protomer choice strongly changes fragmentation and no method predicts protomer formation states a priori
- The target exceeds the small-molecule regime validated here (benchmark metabolites <205 Da); computational cost grows rapidly beyond that scope

## Inputs

- Optimized protonated molecular ion structure in .xyz format (from Avogadro construction + TeraChem unrestricted B3LYP/6–31G* optimization)
- mol_info.in file (mol_name, mol_id, mol_mf, mol_mw, mol_xw) in the Molecule folder
- Parameter choices: b cutoff (Å), set_distance (Å), n_points/seed or Lebedev order, v_scale (AKMA units)

## Outputs

- Per-trajectory start.xyz (molecular ion + argon collider coordinates) and vel.xyz (ion velocity, default 0, and collider velocity) under cid/calcs/XXX/chunk_0000/
- A user-defined number of collision trajectories ready to run with TeraChem and later analyzed with CIDMD_analysis and LearnReactions.py

## How to apply

Run CIDMD_setup.py on the B3LYP/6–31G*-optimized protonated ion: the collider is placed on a molecular envelope surface at --set_distance (default 5.0 Å) from the ion and trajectory initial positions are accepted only if the collider's straight-line path comes within the --cutoff b-factor distance (default 2.0 Å; swept 0.2–5 Å in the study) of any atomic center. Directions are sampled either randomly (--n_points, --seed; default 50 accepted conditions) or deterministically via Lebedev grids (--lebedev order, odd 3–31, or --cartgrid). Set the collider velocity through --v_scale (vfac swept 1–12 AKMA units, 1 AKMA velocity unit = 2045.482 m/s); per-combination repeats over np 50–400 and 1–5 ps reaction time are accumulated into the 261 in silico spectra of the benchmark. Outputs are per-trajectory start.xyz and vel.xyz subfolders consumed by a TeraChem run.in (ub3lyp/6-31gs, run md, timestep 1.0, nstep 1000, bond_order_list yes).

## Related tools

- **CIDMD_setup** (Generates collider initial positions, b-cutoff-filtered trajectories, and velocity files (start.xyz, vel.xyz) for each sampled collision geometry) — https://github.com/jesilee/CIDMD_setup
- **TeraChem** (Executes the resulting trajectories as unrestricted B3LYP/6–31G* NVE ab initio MD with 1.0 fs timestep, recording bond order lists)
- **CIDMD_analysis** (Downstream analysis of the sampled trajectories into theoretical mass spectra (cidmd.jdx, cidmd.msp)) — https://github.com/jesilee/CIDMD_analysis
- **nanoreactor (LearnReactions.py)** (Post-processing of trajectories from the sampled initial conditions to extract fragmentation reactions before spectrum generation) — https://github.com/leeping/nanoreactor

## Examples

```
python3 CIDMD_setup.py Psilocin_ion.xyz --cutoff 2.0 --set_distance 5.0 --n_points 50 --seed 2021 --v_scale 8
```

## Evaluation signals

- Every generated subfolder cid/calcs/XXX/chunk_0000/ contains exactly start.xyz and vel.xyz with the argon atom at the user-defined set_distance (default 5.0 Å) from the molecular envelope
- Accepted collider initial positions have straight-line paths within the b cutoff distance of at least one atomic center (parameter swept 0.2–5 Å)
- vel.xyz collider velocities lie in the intended AKMA-unit range (vfac swept 1–12; 1 AKMA velocity unit = 2045.482 m/s)
- The number of accepted initial conditions matches --n_points (default 50) for randomized mode, up to the study maximum of np = 400 per parameter combination
- TeraChem run.in confirms charge 1, run md, method ub3lyp, basis 6-31gs, timestep 1.0, nstep 1000, bond_order_list yes

## Limitations

- Only a single collision is modeled per trajectory; summing many single-collision trajectories may mismatch experimental multiple-collision CID conditions
- It is unclear how collider trajectory characteristics (b cutoff and velocity) relative to the target molecule's location and vibrational state affect energy-transfer efficiency
- Higher collider velocities are needed for larger molecules (velocity optimum depends on molecular mass), so a fixed vfac is not transferable across analyte sizes
- Randomized sampling is seed-dependent (--seed default 2021); deterministic Lebedev mode with high orders is not recommended (odd orders in [3,31])

## Evidence

- [other] place an argon collider atom 5 Å back from its closest-approach point on a straight-line trajectory, accepting trajectories whose perpendicular distance to any atomic center is below the b cutoff (swept 0.2–5 Å): [evidence span withheld — anchor status: not_found]
- [other] Sweep the collider velocity scale factor vfac from 1 to 12 (AKMA units, 1 AKMA velocity unit = 2045.482 m/s): [evidence span withheld — anchor status: not_found]
