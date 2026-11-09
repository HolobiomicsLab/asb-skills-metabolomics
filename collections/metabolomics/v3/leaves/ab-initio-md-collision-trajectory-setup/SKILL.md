---
name: ab-initio-md-collision-trajectory-setup
description: Use when a protonated molecular ion structure (.xyz) must be paired with
  collider gas-atom initial conditions (position, impact-parameter cutoff, velocity
  scale) to launch collision-induced-dissociation MD trajectories.
when_to_use_negative:
- The input is an experimental spectrum or spectral library entry — this skill generates
  collision initial conditions, not spectra or scores (use CIDMD_analysis/CIDMD_compare
  downstream).
- The molecular ion geometry has not yet been quantum-chemically optimized (e.g. raw
  Avogadro sketch) — geometry optimization with unrestricted B3LYP/6–31G* in TeraChem
  must precede setup.
- Multiple-collision sequences are required — the paper notes 'In CIDMD, an automatic
  multiple collision method has not yet been implemented yet.'
edam_operation: http://edamontology.org/operation_0438
tools:
- name: CIDMD_setup
  role: Generates collider initial positions and velocities (start.xyz, vel.xyz) for
    each accepted collision trajectory via randomized or Lebedev-deterministic sampling
  repo: https://github.com/jesilee/CIDMD_setup
- name: TeraChem
  role: Executes the generated trajectories as unrestricted B3LYP/6–31G* NVE ab initio
    MD with 1.0 fs timestep and bond order output
- name: Avogadro
  role: Used to manually construct the protonated molecular ion structures that serve
    as the .xyz input to setup
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:0d99ff23c68c90467bfe9683114d48b57b49d23b73bbc381c4d771009e533da9
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
  - density functional theory
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ab-initio-md-collision-trajectory-setup

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Sets up CIDMD collision initial conditions by placing an argon collider atom on a user-defined envelope surface around a protonated molecular ion and sampling impact geometries (randomized or Lebedev-deterministic) to generate start.xyz/vel.xyz inputs for NVE ab initio MD with TeraChem.

## When to use

Use when a protonated molecular ion structure (.xyz) must be paired with collider gas-atom initial conditions (position, impact-parameter cutoff, velocity scale) to launch collision-induced-dissociation MD trajectories.

## When NOT to use

- The input is an experimental spectrum or spectral library entry — this skill generates collision initial conditions, not spectra or scores (use CIDMD_analysis/CIDMD_compare downstream).
- The molecular ion geometry has not yet been quantum-chemically optimized (e.g. raw Avogadro sketch) — geometry optimization with unrestricted B3LYP/6–31G* in TeraChem must precede setup.
- Multiple-collision sequences are required — the paper notes 'In CIDMD, an automatic multiple collision method has not yet been implemented yet.'

## Inputs

- Geometry-optimized protonated molecular ion structure in .xyz format
- mol_info.in text file (mol_name, mol_id, mol_mf, mol_mw, mol_xw)
- CIDMD_setup variables in CIDMD_setup.com (set_distance, cutoff, n_points/seed or lebedev/cartgrid, v_scale)
- TeraChem run.in template (charge 1, run md, ub3lyp/6-31gs, timestep 1.0, nstep 1000)

## Outputs

- Per-trajectory subfolders cid/calcs/XXX/chunk_0000/ each containing start.xyz and vel.xyz
- A user-defined number of collision trajectories with varied collisional geometry ready for TeraChem NVE ab initio MD
- Linked run.in input files enabling TeraChem execution of all trajectories

## How to apply

Prepare the geometry-optimized protonated ion (e.g. TeraChem unrestricted B3LYP/6–31G*) and a mol_info.in file, then run CIDMD_setup.com, which drives CIDMD_setup.py. The collider argon atom is placed on a molecular-envelope surface at --set_distance (default 5.0 Å, matching the paper's 5 Å closest-approach placement) and an initial position is accepted only if its straight-line path passes within the --cutoff distance (b cutoff; paper swept 0.2–5 Å) of any atom. Direction sampling uses either the randomized algorithm (--n_points, --seed) or the deterministic Lebedev grid (--lebedev, odd orders 3–31, or --cartgrid). Collider speed is set via --v_scale in AKMA units (1 AKMA velocity unit = 2045.482 m/s); the paper swept vfac 1–12. Each accepted initial condition produces a cid/calcs/XXX/chunk_0000/ folder with start.xyz (ion + collider coordinates) and vel.xyz (velocities), which are run with a run.in file specifying ub3lyp/6-31gs, NVE ensemble, 1.0 fs timestep, nstep 1000, and bond_order_list yes for downstream LearnReactions.py fragmentation analysis.

## Related tools

- **CIDMD_setup** (Generates collider initial positions and velocities (start.xyz, vel.xyz) for each accepted collision trajectory via randomized or Lebedev-deterministic sampling) — https://github.com/jesilee/CIDMD_setup
- **TeraChem** (Executes the generated trajectories as unrestricted B3LYP/6–31G* NVE ab initio MD with 1.0 fs timestep and bond order output)
- **Avogadro** (Used to manually construct the protonated molecular ion structures that serve as the .xyz input to setup)

## Examples

```
bash CIDMD_setup.com  # or: python3 CIDMD_setup.py mol.xyz --set_distance 5.0 --cutoff 5.0 --v_scale 8 --n_points 50
```

## Evaluation signals

- Each accepted initial condition yields a start.xyz containing exactly the molecular ion atoms plus one argon collider atom positioned at the user-defined set_distance (5 Å) from the molecular envelope
- Every retained collider position has a straight-line path passing within the b-factor cutoff (default 2.0 Å in the tool; 0.2–5 Å swept in the paper) of at least one target atom
- vel.xyz carries a collider velocity consistent with the chosen v_scale in AKMA units (1 unit = 2045.482 m/s); ion velocities default to 0
- Generated run.in files specify charge 1, run md, method ub3lyp, basis 6-31gs, timestep 1.0, nstep 1000, and bond_order_list yes
- The number of cid/calcs/XXX subfolders matches the requested number of initial conditions (n_points for randomized mode)

## Limitations

- Only single-collision setups are supported; an automatic multiple-collision method is not implemented, and the paper notes this mismatch with experimental multi-collision CID conditions
- It is unclear how collider trajectory characteristics (b cutoff and velocity) relative to the target molecule's location and vibrational state affect energy-transfer efficiency, so parameter choices must be explored empirically
- The randomized sampling depends on the seed; deterministic mode requires odd Lebedev orders in [3, 31], with higher orders not recommended

## Evidence

- [methods] place an argon collider atom 5 Å back from its closest-approach point on a straight-line trajectory, accepting trajectories whose perpendicular distance to any atomic center is below the b cutoff: [evidence span withheld — anchor status: not_found]
- [methods] the trajectory is accepted for simulation if this distance is less than a b factor cutoff parameter (here called the b cutoff): "the trajectory is accepted for simulation if this distance is less than a b factor cutoff parameter (here called the b cutoff)"
- [readme] start.xyz contains the coordinates for the molecular ion user-provided and the collider gas argon atom cooridiates located an user-defined distance away from the molecular ion.: "start.xyz contains the coordinates for the molecular ion user-provided and the collider gas argon atom cooridiates located an user-defined distance away from the molecular ion."
- [readme] --cutoff CUTOFF       Keep only the collider initial positions whose paths come to within this distance of any atom on the molecular ion (in Angstrom) (default: 2.0): "--cutoff CUTOFF       Keep only the collider initial positions whose paths come to within this distance of any atom on the molecular ion (in Angstrom) (default: 2.0)"
- [readme] Given a molecular ion (protonated molecule), CIDMD predicts a theoretical CID-MS/MS spectrum by automatically placing a collider gas argon atom: "Given a molecular ion (protonated molecule), CIDMD predicts a theoretical CID-MS/MS spectrum by automatically placing a collider gas argon atom"
