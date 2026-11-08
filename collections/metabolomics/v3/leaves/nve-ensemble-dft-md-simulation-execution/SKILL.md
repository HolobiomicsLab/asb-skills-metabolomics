---
name: nve-ensemble-dft-md-simulation-execution
description: Use when a protonated molecular ion (e.g. a metabolite <205 Da with a
  prepared protonation site) must be collided with an argon collider atom and bond
  breakages simulated from first principles to generate in silico CID-MS/MS spectra.
when_to_use_negative:
- When only fragment structures or spectra of stable molecules are needed without
  simulating the collision itself — a thermodynamic or rule-based predictor is far
  cheaper and CIDMD's ab initio MD is computationally heavy.
- When the molecule is too large or has many protomers/tautomers — the article reports
  average CIDMD prediction accuracy decreases as the number of protomers and tautomers
  increases.
- When a ground-state optimization or single-point energy is sufficient — this skill
  is specifically for reactive collision dynamics in the NVE (microcanonical) ensemble,
  not for geometry optimization.
- When simulating multiple successive collisions inside one trajectory — the article
  states 'an automatic multiple collision method has not yet been implemented yet',
  so one trajectory = one collision.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0154
tools:
- name: TeraChem
  role: GPU-accelerated quantum chemistry engine used to run the NVE-ensemble unrestricted
    B3LYP/6-31G* MD trajectories and emit the bond order list each step.
- name: CIDMD_setup
  role: Generates start.xyz and vel.xyz collision initial conditions (argon placement
    at 5 Å, b-cutoff filtering, vfac velocity scaling) that feed the TeraChem run.in.
  repo: https://github.com/jesilee/CIDMD_setup
- name: nanoreactor (LearnReactions.py)
  role: Post-processes the CIDMD trajectory using the Mayer bond order time series
    to detect fragmentation events and produce learn_rxn.log for downstream analysis.
  repo: https://github.com/leeping/nanoreactor
- name: CIDMD_analysis
  role: Analyzes the LearnReactions output into cidmd.jdx / cidmd.msp theoretical
    spectra and diagnostic plots (ar_vel_KE.png, total_gputime.txt).
  repo: https://github.com/jesilee/CIDMD_analysis
- name: Avogadro
  role: Manual construction of the protonated molecular ion structure before geometry
    optimization and MD setup.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:31208334161c7e0426092680d81ff0f0fe8208d67c88bea9eadc87ac63415d53
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 22
---

# nve-ensemble-dft-md-simulation-execution

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Run collision-induced-dissociation dynamics of protonated metabolites as NVE-ensemble ab initio MD with TeraChem at unrestricted B3LYP/6-31G*, a 1.0 fs time step, and Mayer bond order tracking over at least 1,000 fs, so that bond breakages driving CID-MS/MS fragmentation can be extracted.

## When to use

Use when a protonated molecular ion (e.g. a metabolite <205 Da with a prepared protonation site) must be collided with an argon collider atom and bond breakages simulated from first principles to generate in silico CID-MS/MS spectra.

## When NOT to use

- When only fragment structures or spectra of stable molecules are needed without simulating the collision itself — a thermodynamic or rule-based predictor is far cheaper and CIDMD's ab initio MD is computationally heavy.
- When the molecule is too large or has many protomers/tautomers — the article reports average CIDMD prediction accuracy decreases as the number of protomers and tautomers increases.
- When a ground-state optimization or single-point energy is sufficient — this skill is specifically for reactive collision dynamics in the NVE (microcanonical) ensemble, not for geometry optimization.
- When simulating multiple successive collisions inside one trajectory — the article states 'an automatic multiple collision method has not yet been implemented yet', so one trajectory = one collision.

## Inputs

- start.xyz: coordinates of the protonated molecular ion and the argon collider atom (from CIDMD_setup)
- vel.xyz: initial velocities of the molecular ion (default 0) and the collider in AKMA units
- run.in: TeraChem input file specifying method, basis, ensemble, time step, and bond order options
- mol_info.in: molecule metadata (mol_name, mol_id, mol_mf, mol_mw, mol_xw) in the Molecule folder

## Outputs

- Mayer bond order time series recorded at each MD step (>= 1,000 fs) for the trajectory
- CIDMD trajectories under Molecule/cid/calcs/XXX/chunk_0000/ ready for LearnReactions.py post-processing
- learn_rxn.log files in Molecule/cid/calcs/XXX/gathered/ feeding CIDMD_analysis
- total_gputime.txt and per-trajectory reports consumed by CIDMD_analysis

## How to apply

Take start.xyz (molecular ion plus an argon collider placed 5 Å back along the straight-line collision trajectory, filtered by a b-factor cutoff against the molecular envelope) and vel.xyz (user-defined collider velocity in AKMA units; vfac is swept 1-12, where 1 AKMA velocity unit = 2045.482 m/s) from CIDMD_setup. Execute each trajectory with TeraChem as NVE-ensemble unrestricted B3LYP/6-31G* MD with a 1.0 fs time step (run.in sets method=ub3lyp, basis=6-31gs, run=md, charge=1, spinmult=1, nstep=1000), enabling bond_order_list with bond_order_thresh 0.1 so a Mayer bond order time series is recorded at every step. Run trajectories for at least 1,000 fs so bond breakages are captured. Rationale: DFT-level first-principles MD reproduces the physical collision and energy transfer of CID, and bond order tracking (rather than geometry alone) enables automated fragment identification downstream in LearnReactions.py. Repeat simulations from varied collisional directions and parameter combinations (vfac, np, reaction time 1-5 ps) to accumulate fragmentations into CIDMD in silico mass spectra.

## Related tools

- **TeraChem** (GPU-accelerated quantum chemistry engine used to run the NVE-ensemble unrestricted B3LYP/6-31G* MD trajectories and emit the bond order list each step.)
- **CIDMD_setup** (Generates start.xyz and vel.xyz collision initial conditions (argon placement at 5 Å, b-cutoff filtering, vfac velocity scaling) that feed the TeraChem run.in.) — https://github.com/jesilee/CIDMD_setup
- **nanoreactor (LearnReactions.py)** (Post-processes the CIDMD trajectory using the Mayer bond order time series to detect fragmentation events and produce learn_rxn.log for downstream analysis.) — https://github.com/leeping/nanoreactor
- **CIDMD_analysis** (Analyzes the LearnReactions output into cidmd.jdx / cidmd.msp theoretical spectra and diagnostic plots (ar_vel_KE.png, total_gputime.txt).) — https://github.com/jesilee/CIDMD_analysis
- **Avogadro** (Manual construction of the protonated molecular ion structure before geometry optimization and MD setup.)

## Examples

```
bash CIDMD_setup.com && terachem run.in &> run.out && python LearnReactions.py traj.xyz
```

## Evaluation signals

- Trajectory length meets the minimum 1,000 fs requirement, recorded with a 1.0 fs time step (nstep >= 1000 in run.in).
- TeraChem run.in settings match the protocol: run=md, method=ub3lyp, basis=6-31gs, charge=1, spinmult=1, timestep=1.0, removecomm=no, bond_order_list=yes.
- The bond order list is produced every step with bond_order_thresh 0.1 so LearnReactions.py can detect fragments; verify bonds.dat / reaction_*.xyz outputs exist after post-processing.
- total_gputime.txt in the results folder reflects non-zero GPU time consistent with completed trajectories (not failed runs).
- Across the parameter sweep (vfac 1-12, np 50-400, time 1-5 ps, 12 metabolites) the total run set yields 261 in silico spectra whose entropy similarity scores against NIST20 experimental spectra average 624 ± 189 (828 ± 77 with optimal most-probable-protomer parameters).

## Limitations

- A single trajectory models a single collision; automatic multiple collisions per trajectory are not implemented, so fragment accumulation requires many independent trajectories.
- Energy transfer efficiency is sensitive to collider trajectory parameters (b cutoff and velocity) relative to the target molecule's location and vibrational state, and it is still unclear how these interact.
- Computational cost (GPU ab initio MD) limits coverage; predicting high-quality MS/MS spectra for even a fraction of the chemosphere is beyond reach of current methods.
- Accuracy degrades as the number of protomers and tautomers increases, and no method exists to predict protomer formation states before running CIDMD simulations.
- No systematic investigation of radical fragment ions in CID tandem mass spectra has been done; CIDMD may miss or mis-assign radical pathways.
- The mismatch between summed single-collision trajectories and experimental multiple-collision CID conditions may account for discrepancies between modeled and experimental spectra.

## Evidence
