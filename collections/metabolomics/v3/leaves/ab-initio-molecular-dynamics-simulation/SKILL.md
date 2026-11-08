---
name: ab-initio-molecular-dynamics-simulation
description: Use when a small molecule (<205 Da initially, extendable to larger metabolites)
  has no experimental MS/MS library spectrum and a protonated ion structure can be
  built, so in silico CID spectra are needed.
when_to_use_negative:
- The molecule is large (>205 Da in benchmarked cases) and you lack a reference experimental
  spectrum to optimize vfac/collider parameters against.
- The molecule has many protomers/tautomers — average CIDMD prediction accuracy decreases
  with increasing numbers of protomers and tautomers.
- You need a full fragmentation tree with radical ions; no systematic investigation
  of radical fragment ions in CID has been done.
edam_operation: http://edamontology.org/operation_3874
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3473
tools:
- name: CIDMD_setup
  role: Sets up collision geometries and writes start.xyz/vel.xyz trajectories for
    TeraChem
  repo: https://github.com/jesilee/CIDMD_setup
- name: CIDMD_analysis
  role: Post-processes CIDMD trajectories into cidmd.msp / cidmd.jdx spectra and reports
  repo: https://github.com/jesilee/CIDMD_analysis
- name: CIDMD_compare
  role: Compares predicted spectra to reference experimental mass spectra
  repo: https://github.com/jesilee/CIDMD_compare
- name: nanoreactor (LearnReactions.py)
  role: Extracts reactions and fragments from reactive MD trajectories before analysis
  repo: https://github.com/leeping/nanoreactor
- name: TeraChem
  role: Runs the unrestricted B3LYP/6–31G* NVE ab initio MD simulations
- name: Avogadro
  role: Manual construction of protonated molecular ion structures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:8bf36ea9c22f5072ea637fb14698417d032340b5095929041afc6d7a2105496c
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
  - nanoreactor (LearnReactions.py)
  edam_operation: http://edamontology.org/operation_3874
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ab-initio-molecular-dynamics-simulation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Runs NVE-ensemble ab initio MD collisions (CIDMD) between a protonated molecular ion and an argon collider using unrestricted B3LYP/6–31G* in TeraChem, to simulate collision-induced dissociation and predict CID-MS/MS spectra for small metabolites lacking library spectra.

## When to use

Use when a small molecule (<205 Da initially, extendable to larger metabolites) has no experimental MS/MS library spectrum and a protonated ion structure can be built, so in silico CID spectra are needed.

## When NOT to use

- The molecule is large (>205 Da in benchmarked cases) and you lack a reference experimental spectrum to optimize vfac/collider parameters against.
- The molecule has many protomers/tautomers — average CIDMD prediction accuracy decreases with increasing numbers of protomers and tautomers.
- You need a full fragmentation tree with radical ions; no systematic investigation of radical fragment ions in CID has been done.

## Inputs

- Optimized protonated molecular ion structure in .xyz format
- mol_info.in file with molecule name, formula, molecular weight (e.g. mol_name, mol_id, mol_mf, mol_mw, mol_xw)
- CIDMD_setup.com / CIDMD_setup.py variables (extent, cutoff, set_distance, n_points, seed, v_scale)
- run.in TeraChem input specifying charge 1, spinmult 1, method ub3lyp, basis 6-31gs, timestep 1.0, nstep 1000
- learn_rxn.log files from LearnReactions.py post-processing located in Molecule/cid/calcs/XXX/gathered/

## Outputs

- start.xyz and vel.xyz collision input files per trajectory in cid/calcs/XXX/chunk_0000/
- CIDMD trajectories (.xyz) processed by LearnReactions.py into reaction_123.xyz, bonds.dat, color.dat, charge.dat and spin.dat
- cidmd.msp and cidmd.jdx theoretical mass spectra (NIST-compatible formats)
- Plot outputs: cidmd.PS_case1_RCID_vfac8-cut05norm.png, cidmd.PS_case1_RCID_vfac8-cut05.png, ar_vel_KE.png
- Analysis logs: sy_found.out, pop_found.out, rxn_found.out, report_found.out, total_gputime.txt

## How to apply

Build protonated molecular ions at candidate protonation sites (primarily heteroatoms) manually in Avogadro, then optimize 3D structures with unrestricted B3LYP/6–31G* in TeraChem. Run CIDMD_setup (git clone https://github.com/jesilee/CIDMD_setup) to place an argon collider 5 Å from the molecular envelope, accepting trajectories with a b cutoff of 0.2–5 Å, n_points of 50–400, and collider velocities (vfac 1–12, AKMA units) scaled per molecular mass since higher velocities achieve better results for larger molecules. Execute TeraChem MD at unrestricted B3LYP/6–31G* in the NVE ensemble with a 1.0 fs timestep for ≥1000 steps (1–5 ps) per trajectory, repeating collisions from multiple directions. Identify fragments via low-pass-filtered Mayer bond order time series (200 cm−1 cutoff, 0.2 discretization threshold, ≥50 fs stability) and Mulliken charge averaging (0.70 threshold); accumulate fragmentations over repeated trajectories to produce in silico mass spectra in cidmd.msp format.

## Related tools

- **CIDMD_setup** (Sets up collision geometries and writes start.xyz/vel.xyz trajectories for TeraChem) — https://github.com/jesilee/CIDMD_setup
- **CIDMD_analysis** (Post-processes CIDMD trajectories into cidmd.msp / cidmd.jdx spectra and reports) — https://github.com/jesilee/CIDMD_analysis
- **CIDMD_compare** (Compares predicted spectra to reference experimental mass spectra) — https://github.com/jesilee/CIDMD_compare
- **nanoreactor (LearnReactions.py)** (Extracts reactions and fragments from reactive MD trajectories before analysis) — https://github.com/leeping/nanoreactor
- **TeraChem** (Runs the unrestricted B3LYP/6–31G* NVE ab initio MD simulations)
- **Avogadro** (Manual construction of protonated molecular ion structures)

## Examples

```
bash CIDMD_setup.com && find calcs -name start.xyz -execdir ln -fs ../../../run.in . \; && terachem run.in &> run.out
```

## Evaluation signals

- cidmd.msp / cidmd.jdx exist in the results folder under Molecule/cid/ and parse in NIST programs.
- Benchmarked entropy similarity of 624 ± 189 for all 261 spectra vs experimental, rising to 828 ± 77 with optimal protomer parameters.
- nstep in run.in is at least 1000 with timestep 1.0 fs in the NVE ensemble.
- Fragments recognized require ≥50 fs stability and Mulliken charge threshold of 0.70.
- Total runtime in total_gputime.txt reflects repeated multi-direction trajectories (50–400 n_points).

## Limitations

- 261 predicted spectra averaged 624 ± 189 entropy similarity overall; only 828 ± 77 when protomer/tautomer parameters are optimal.
- Different protomers fragment very differently, so accuracy decreases with more protomers and tautomers.
- No automatic multiple-collision method is implemented yet; trajectories are single collisions with multiple angles summed.
- Relying on LearnReactions.py from the nanoreactor repository which was noted as 'not yet released'.
- CIDMD failed on ribono-γ-lactone: m/z 69.033 ion not observed from the m/z 103.039 simulation because the two oxygens were geometrically too far apart.
- Unclear how b cutoff and collider velocity affect energy transfer efficiency relative to target molecule location/vibrational state.

## Evidence

- [readme] CIDMD framework uses NVE-ensemble ab initio MD at ub3lyp/6-31gs with 1.0 fs timestep in TeraChem: "CIDMD simulations were performed at unrestricted B3LYP/6–31G* in the microcanonical (NVE) ensemble with a time step of 1.0 fs utilizing TeraChem."
- [readme] CIDMD_setup places an argon collider 5 Å away and generates start.xyz / vel.xyz: "start.xyz contains the coordinates for the molecular ion user-provided and the collider gas argon atom cooridiates located an user-defined distance away from the molecular ion."
- [methods] Fragments identified via Mayer bond order time series with 200 cm−1 low-pass filter and 0.2 threshold: "The raw time series were low-pass-filtered with an upper frequency cutoff corresponding to 200 cm−1, then discretized by applying a threshold of 0.2."
- [abstract] Average entropy similarity scores of 624 ± 189 (all) and 828 ± 77 (optimal) against experimental spectra: "These in silico spectra resulted in entropy similarity scores of an average 624 ± 189 for all 261 spectra compared to their corresponding experimental spectra, which improved to 828 ± 77"
- [abstract] Higher collider velocities achieve better results for molecules of increasing mass: "With increasing molecular mass, higher velocities achieved better results."
- [abstract] Explicit workflow: build ion → collide with argon → simulate ≥1000 fs → accumulate repeated collisions to generate in silico spectra: "Subsequent bond breakages are simulated over time for at least 1,000 fs."
- [readme] CIDMD_analysis expects trajectories pre-processed with LearnReactions.py from leeping/nanoreactor repo: [evidence span withheld — anchor status: external_doc]
