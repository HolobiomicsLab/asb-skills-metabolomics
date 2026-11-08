---
name: cid-parameter-sweep-configuration
description: Use when generating a library of in silico CID-MS/MS spectra from optimized
  protonated molecular ions and need to explore multiple collider velocities, trajectory
  counts, reaction times, and b cutoffs per protomer rather than committing to a single
  parameter set.
when_to_use_negative:
- The metabolite exceeds the validated size range — the framework was tested only
  on 12 small metabolites (<205 Da)
- Only a single spectrum with fixed parameters is needed and no parameter exploration
  is desired — a single CIDMD run suffices without the sweep machinery
- No experimental reference spectrum (e.g., NIST20) exists for the target compound,
  since the sweep is benchmarked and parameter selection relies on entropy similarity
  scoring against library spectra
edam_operation: http://edamontology.org/operation_4004
edam_topics:
- http://edamontology.org/topic_3369
- http://edamontology.org/topic_3517
tools:
- name: CIDMD_setup
  role: Generates the swept collision initial conditions — places the argon collider
    on the molecular envelope (set_distance 5.0 Å default), filters paths by the --cutoff
    b-factor parameter, and emits start.xyz/vel.xyz pairs for each parameter combination
  repo: https://github.com/jesilee/CIDMD_setup
- name: TeraChem
  role: Executes each swept trajectory as NVE ensemble unrestricted B3LYP/6–31G* ab
    initio MD with 1.0 fs time step, recording bond order lists at each step
- name: Avogadro
  role: Used to manually construct the protonated molecular ion structures for each
    protomer before the parameter sweep
- name: nanoreactor (LearnReactions.py)
  role: Post-processes the swept CIDMD trajectories to extract fragmentation reactions
    for spectrum accumulation
  repo: https://github.com/leeping/nanoreactor
- name: CIDMD_analysis
  role: Converts accumulated fragmentations from the swept run set into cidmd.jdx/cidmd.msp
    theoretical mass spectra
  repo: https://github.com/jesilee/CIDMD_analysis
- name: CIDMD_compare
  role: Scores each swept spectrum against reference mass spectra to select optimal
    parameter combinations
  repo: https://github.com/jesilee/CIDMD_compare
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:7b2a80d5ce9deb882fed4d88cccd9fe2db4bf88de443b758c80a41ac5d2460a8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_setup
  tools:
  - 6–31G*
  - Avogadro
  - B3LYP
  - CIDMD_analysis
  - CIDMD_compare
  - CIDMD_setup
  - TeraChem
  - density functional theory
  - nanoreactor (LearnReactions.py)
  edam_operation: http://edamontology.org/operation_4004
  edam_topics:
  - http://edamontology.org/topic_3369
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# cid-parameter-sweep-configuration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Systematically sweeps CIDMD simulation parameters — collider velocity scale factor (vfac 1–12 AKMA), number of accepted collision trajectories (np 50–400), reaction time (1–5 ps), and impact-parameter b cutoff (0.2–5 Å) — across protonation states of small metabolites to generate a library of in silico CID-MS/MS spectra (261 spectra for 12 benchmark metabolites) that can be scored against NIST20 experimental spectra.

## When to use

Use when generating a library of in silico CID-MS/MS spectra from optimized protonated molecular ions and need to explore multiple collider velocities, trajectory counts, reaction times, and b cutoffs per protomer rather than committing to a single parameter set.

## When NOT to use

- The metabolite exceeds the validated size range — the framework was tested only on 12 small metabolites (<205 Da)
- Only a single spectrum with fixed parameters is needed and no parameter exploration is desired — a single CIDMD run suffices without the sweep machinery
- No experimental reference spectrum (e.g., NIST20) exists for the target compound, since the sweep is benchmarked and parameter selection relies on entropy similarity scoring against library spectra

## Inputs

- Geometry-optimized protonated molecular ion structure (.xyz) for each protomer, built in Avogadro and optimized with TeraChem at unrestricted B3LYP/6–31G*
- mol_info.in file with molecule name, ID, formula, and molecular weight
- TeraChem run.in input file specifying charge 1, spinmult 1, method ub3lyp, basis 6-31gs, timestep 1.0, nstep 1000, bond_order_list yes

## Outputs

- Per-collision input files (start.xyz with coordinates, vel.xyz with collider velocity in AKMA units) in subfolders cid/calcs/XXX/chunk_0000/
- One in silico CID-MS/MS spectrum per parameter combination (261 total for 12 metabolites), emitted as cidmd.jdx and cidmd.msp files
- Entropy similarity scores vs NIST20 experimental spectra (average 624 ± 189 over all spectra; 828 ± 77 for optimal most-probable-protomer parameters)

## How to apply

Prepare geometry-optimized protonated molecular ions (all protomers) at unrestricted B3LYP/6–31G* in TeraChem, then run CIDMD_setup to place an argon collider 5 Å back from its closest-approach point, accepting trajectories whose perpendicular distance to any atomic center is below the b cutoff (swept 0.2–5 Å) with direction sampling via random or Lebedev-grid deterministic modes. Sweep the velocity scale factor vfac from 1 to 12 AKMA units (1 AKMA velocity unit = 2045.482 m/s), the number of accepted trajectories np from 50 to 400, and reaction time from 1 to 5 ps per protomer. Execute each trajectory as NVE ensemble ab initio MD at unrestricted B3LYP/6–31G*, 1.0 fs time step, recording bond order lists at each step for at least 1,000 fs, then accumulate fragmentations over repeated collisions to produce each spectrum. Expect one in silico spectrum per parameter combination — 261 spectra across the 12 benchmark metabolites (<205 Da). Higher collider velocities give better results with increasing molecular mass, so bias the sweep toward higher vfac for heavier metabolites, and prioritize the most probable protomer for the highest-fidelity configuration.

## Related tools

- **CIDMD_setup** (Generates the swept collision initial conditions — places the argon collider on the molecular envelope (set_distance 5.0 Å default), filters paths by the --cutoff b-factor parameter, and emits start.xyz/vel.xyz pairs for each parameter combination) — https://github.com/jesilee/CIDMD_setup
- **TeraChem** (Executes each swept trajectory as NVE ensemble unrestricted B3LYP/6–31G* ab initio MD with 1.0 fs time step, recording bond order lists at each step)
- **Avogadro** (Used to manually construct the protonated molecular ion structures for each protomer before the parameter sweep)
- **nanoreactor (LearnReactions.py)** (Post-processes the swept CIDMD trajectories to extract fragmentation reactions for spectrum accumulation) — https://github.com/leeping/nanoreactor
- **CIDMD_analysis** (Converts accumulated fragmentations from the swept run set into cidmd.jdx/cidmd.msp theoretical mass spectra) — https://github.com/jesilee/CIDMD_analysis
- **CIDMD_compare** (Scores each swept spectrum against reference mass spectra to select optimal parameter combinations) — https://github.com/jesilee/CIDMD_compare

## Examples

```
bash CIDMD_setup.com  # configure vfac, --cutoff, --set_distance, --n_points, and Lebedev options in CIDMD_setup.com before executing
```

## Evaluation signals

- The run set spans the sweep dimensions: vfac 1–12 AKMA, np 50–400 accepted trajectories, reaction time 1–5 ps, b cutoff 0.2–5 Å
- Total count of in silico spectra matches one per parameter combination per protomer — 261 spectra across the 12 metabolites
- Each collision subfolder contains start.xyz and vel.xyz with the collider placed 5 Å from the molecular envelope and velocity in AKMA units
- Trajectories ran at unrestricted B3LYP/6–31G*, NVE ensemble, 1.0 fs time step, for at least 1,000 fs with bond order lists recorded
- Entropy similarity scores against NIST20 spectra average 624 ± 189 across all 261 spectra, improving to 828 ± 77 for the most-probable-protomer optimal parameters

## Limitations

- Prediction accuracy decreases as the number of protomers and tautomers increases; different protomers show large differences in fragmentation
- There is no method to accurately predict protomer formation states before running the sweep, so computationally intensive simulations across all protomers are required
- Higher molecular mass requires higher collider velocities for good results; the sweep must be biased accordingly
- An automatic multiple collision method has not been implemented in CIDMD; simulating multiple collision angles and summing trajectories only partially accounts for the mismatch with experimental MS/MS conditions
- The sweep is computationally expensive — high-quality MS/MS prediction for even a fraction of the chemosphere is beyond the reach of current methods

## Evidence
