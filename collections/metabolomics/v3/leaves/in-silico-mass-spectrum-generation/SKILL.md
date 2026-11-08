---
name: in-silico-mass-spectrum-generation
description: Use when you need to predict an MS/MS spectrum for a small molecule lacking
  library spectra — e.g. a metabolite, gut-microbiome compound, or any structure whose
  experimental CID-MS/MS spectrum is absent — and you can afford quantum-chemistry
  MD.
when_to_use_negative:
- The molecule already has a high-quality experimental MS/MS spectrum in NIST20 or
  similar library — experimental comparison is preferable and far cheaper than quantum-chemistry
  MD
- The compound is large (>~205 Da was benchmarked; multiple protomers/tautomers inflate
  cost and reduce accuracy) or requires multiple collisions, which CIDMD does not
  yet implement automatically
- The analysis goal is speed at scale — no tool in this paper's ecosystem combines
  accuracy with speed, so the computational cost of ab initio MD cannot cover even
  a fraction of the chemosphere
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CIDMD_setup
  role: 'Prepares collision initial conditions: places an argon collider atom at a
    set distance, samples trajectories (random or Lebedev-grid), and writes start.xyz/vel.xyz
    inputs for TeraChem'
  repo: https://github.com/jesilee/CIDMD_setup
- name: TeraChem
  role: Runs the NVE-ensemble unrestricted B3LYP/6-31G* ab initio MD collision trajectories
    with bond order recording
- name: CIDMD_analysis
  role: Post-processes LearnReactions.py outputs (learn_rxn.log) and bond order time
    series to identify fragments, assign charges, and write cidmd.msp/cidmd.jdx spectra
  repo: https://github.com/jesilee/CIDMD_analysis
- name: CIDMD_compare
  role: Compares the CIDMD-predicted spectrum against a reference mass spectrum using
    cosine, mass-weighted dot, or entropy scores
  repo: https://github.com/jesilee/CIDMD_compare
- name: LearnReactions.py (nanoreactor)
  role: Processes raw CIDMD trajectories into learn_rxn.log files that CIDMD_analysis
    consumes
  repo: https://github.com/leeping/nanoreactor
- name: Avogadro
  role: Manually constructs protonated molecular ion structures at candidate protonation
    sites
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:a88a02027d1d3cc1b0f59da44c0f0aed52f958e17246d9c48c697e7c2ca11a61
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
  - LearnReactions.py (nanoreactor)
  - TeraChem
  - density functional theory
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# in-silico-mass-spectrum-generation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Generates theoretical CID-MS/MS mass spectra from protonated molecular ion structures by running repeated NVE ab initio MD collision trajectories (unrestricted B3LYP/6-31G* in TeraChem) against argon collider atoms and accumulating fragmentations into an msp-formatted library spectrum.

## When to use

Use when you need to predict an MS/MS spectrum for a small molecule lacking library spectra — e.g. a metabolite, gut-microbiome compound, or any structure whose experimental CID-MS/MS spectrum is absent — and you can afford quantum-chemistry MD.

## When NOT to use

- The molecule already has a high-quality experimental MS/MS spectrum in NIST20 or similar library — experimental comparison is preferable and far cheaper than quantum-chemistry MD
- The compound is large (>~205 Da was benchmarked; multiple protomers/tautomers inflate cost and reduce accuracy) or requires multiple collisions, which CIDMD does not yet implement automatically
- The analysis goal is speed at scale — no tool in this paper's ecosystem combines accuracy with speed, so the computational cost of ab initio MD cannot cover even a fraction of the chemosphere

## Inputs

- Optimized protonated molecular ion structure in .xyz format (all protomers/tautomers for the compound, as fragmentation differs strongly between protomers)
- mol_info.in file defining molecule name, id, formula (mol_mf), and molecular weight (mol_mw/mol_xw)
- CIDMD_setup.com variable settings: v_scale, cutoff, n_points/lebedev grid, set_distance

## Outputs

- cidmd.msp — theoretical mass spectrum in MSP format for NIST library search or CIDMD_compare
- cidmd.jdx — the same spectrum in JCAMP-DX format
- Per-collision input files start.xyz and vel.xyz in cid/calcs/XXX/chunk_0000/ subfolders
- Analysis outputs: sy_found.out, pop_found.out, rxn_found.out, report_found.out, ar_vel_KE.png

## How to apply

Construct the protonated molecular ion manually in Avogadro at candidate heteroatom protonation sites and optimize the 3D structure in TeraChem at unrestricted B3LYP/6-31G*. Run CIDMD_setup, which places an argon collider 5 Å from the molecular envelope and generates accepted collision initial conditions (b cutoff 0.2–5 Å, 50–400 trajectories via random or Lebedev-grid deterministic sampling) with collider velocities scaled by vfac 1–12 in AKMA units — higher velocities for larger molecules. Execute NVE-ensemble ab initio MD at ub3lyp/6-31gs with a 1.0 fs timestep for ≥1000 steps (1–5 ps), recording Mayer bond order lists at each step. Identify fragments by low-pass-filtering bond order time series at 200 cm−1, discretizing at a 0.2 threshold, requiring ≥50 fs fragment stability, and assigning charges via Mulliken populations with a 0.70 threshold. Accumulate fragmentations over repeated collisions from varied directions and convert observed ion frequencies into simulated spectra written to cidmd.msp for downstream comparison.

## Related tools

- **CIDMD_setup** (Prepares collision initial conditions: places an argon collider atom at a set distance, samples trajectories (random or Lebedev-grid), and writes start.xyz/vel.xyz inputs for TeraChem) — https://github.com/jesilee/CIDMD_setup
- **TeraChem** (Runs the NVE-ensemble unrestricted B3LYP/6-31G* ab initio MD collision trajectories with bond order recording)
- **CIDMD_analysis** (Post-processes LearnReactions.py outputs (learn_rxn.log) and bond order time series to identify fragments, assign charges, and write cidmd.msp/cidmd.jdx spectra) — https://github.com/jesilee/CIDMD_analysis
- **CIDMD_compare** (Compares the CIDMD-predicted spectrum against a reference mass spectrum using cosine, mass-weighted dot, or entropy scores) — https://github.com/jesilee/CIDMD_compare
- **LearnReactions.py (nanoreactor)** (Processes raw CIDMD trajectories into learn_rxn.log files that CIDMD_analysis consumes) — https://github.com/leeping/nanoreactor
- **Avogadro** (Manually constructs protonated molecular ion structures at candidate protonation sites)

## Examples

```
bash CIDMD_setup.com && find calcs -name start.xyz -execdir ln -fs ../../../run.in . \; && terachem run.in &> run.out
```

## Evaluation signals

- cidmd.msp exists, parses as valid MSP (one spectrum per metabolite, precursor m/z equals protonated mass, fragment peaks at exact masses to 0.01 mDa), and loads in the CIDMD_compare/NIST workflow
- Spectrum similarity to the experimental NIST20 spectrum reaches entropy scores in the reported benchmark range — average 624 ± 189 across 261 spectra, improving to 828 ± 77 for optimal most-probable-protomer parameters
- Fragments in the msp file account for observed ions from at least one protomer pathway; a fragment must exist ≥50 fs and its Mulliken-averaged charge must exceed 0.70 to be counted
- The trajectory set spans the swept parameter space (vfac 1–12, 50–400 collisions, 1–5 ps, b cutoff 0.2–5 Å); the full sweep over 12 metabolites totals 261 spectra
- Each generated start.xyz/vel.xyz pair places the argon collider 5 Å from the molecular envelope with a trajectory whose closest approach is within the b cutoff

## Limitations

- Prediction accuracy decreases as the number of protomers and tautomers increases, because different protomers show large differences in fragmentation and no method exists to accurately predict protomer formation states before simulation
- An automatic multiple-collision method has not been implemented; the paper's summing of many single-collision trajectories from varied angles partially compensates but mismatches real experimental MS/MS conditions
- The framework is computationally expensive (ab initio MD at B3LYP/6-31G* per trajectory) — predicting high-quality MS/MS spectra for even a fraction of the chemosphere is beyond reach
- No systematic treatment of radical fragment ions; unclear how collider trajectory characteristics (b cutoff, velocity) relative to target vibrational state affect energy-transfer efficiency
- CIDMD may miss fragment ions entirely — e.g. it failed to predict the ribono-γ-lactone m/z 69.033 ion because the two oxygens in the m/z 103.039 precursor were geometrically too far apart

## Evidence

- [readme] Analysis output files: [evidence span withheld — anchor status: external_doc]
