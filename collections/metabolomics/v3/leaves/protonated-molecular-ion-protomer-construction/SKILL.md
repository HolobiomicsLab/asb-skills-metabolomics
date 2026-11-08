---
name: protonated-molecular-ion-protomer-construction
description: Use when preparing a small molecule (<~205 Da in the benchmark) for CID-MS/MS
  spectrum prediction and all plausible protonation sites/protomers must be enumerated
  and optimized before running collision trajectories.
when_to_use_negative:
- The molecule is large (>~205 Da in this benchmark) or highly flexible, where the
  article notes prediction accuracy drops with increasing molecular mass and protomer/tautomer
  count
- You only need to score already-predicted spectra against experimental data — use
  CIDMD_compare instead
- You want to predict which protomer forms without running simulations; the article
  explicitly notes no method exists yet to predict protomer formation states prior
  to CIDMD
edam_operation: http://edamontology.org/operation_2489
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0081
tools:
- name: Avogadro
  role: Manual construction of protonated molecular ion structures (protomers) before
    optimization
- name: TeraChem
  role: 3D geometry optimization of each ion structure at unrestricted B3LYP/6–31G*
- name: CIDMD_setup
  role: Consumes the optimized protomer xyz files to generate collision initial conditions
    (start.xyz, vel.xyz)
  repo: https://github.com/jesilee/CIDMD_setup
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:be744c48373ebe5217bfd81d40b1bd3b3b9582dd782a7bea622194d3f7ff0d4c
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
  - first-principles molecular dynamics (MD)
  edam_operation: http://edamontology.org/operation_2489
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# protonated-molecular-ion-protomer-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Constructs all relevant protonated molecular-ion structures (protomers and tautomers) of a target metabolite in Avogadro and geometry-optimizes them at unrestricted B3LYP/6–31G* with TeraChem, providing the input geometries that CIDMD collision simulations require.

## When to use

Use when preparing a small molecule (<~205 Da in the benchmark) for CID-MS/MS spectrum prediction and all plausible protonation sites/protomers must be enumerated and optimized before running collision trajectories.

## When NOT to use

- The molecule is large (>~205 Da in this benchmark) or highly flexible, where the article notes prediction accuracy drops with increasing molecular mass and protomer/tautomer count
- You only need to score already-predicted spectra against experimental data — use CIDMD_compare instead
- You want to predict which protomer forms without running simulations; the article explicitly notes no method exists yet to predict protomer formation states prior to CIDMD

## Inputs

- Neutral molecule structure (drawn or imported) for the target metabolite
- List of candidate protonation sites / protomer variants per molecule
- TeraChem executable and unrestricted B3LYP/6–31G* settings

## Outputs

- One geometry-optimized protonated molecular ion structure (.xyz) per protomer, optimized at uB3LYP/6–31G*
- Set of protomers ready as CIDMD_setup inputs for collision trajectory generation

## How to apply

Draw the neutral molecule, then manually add a proton at each chemically plausible site (e.g. amine N, ring N, carbonyl or ether O) using Avogadro to generate one candidate protomer per site. Optimize each 3D structure with TeraChem at unrestricted B3LYP/6–31G* (the same level of theory later used for the NVE collision dynamics), since accurate starting geometries govern collision energy transfer. Keep separate files for each protomer because the article shows different protomers fragment substantially differently, so each must be simulated independently. Feed the optimized xyz of each protomer into CIDMD_setup; the protomer whose parameters give the best entropy similarity against NIST20 experimental spectra identifies the most probable protomer. Remember that accuracy degrades as the number of protomers and tautomers grows, so exhaustive site enumeration matters for multi-basic molecules.

## Related tools

- **Avogadro** (Manual construction of protonated molecular ion structures (protomers) before optimization)
- **TeraChem** (3D geometry optimization of each ion structure at unrestricted B3LYP/6–31G*)
- **CIDMD_setup** (Consumes the optimized protomer xyz files to generate collision initial conditions (start.xyz, vel.xyz)) — https://github.com/jesilee/CIDMD_setup

## Examples

```
terachem run.in &> run.out  # run.in: coordinates start.xyz; charge 1; run md; method ub3lyp; basis 6-31gs
```

## Evaluation signals

- Each protomer xyz is charge +1 and fully geometry-optimized under uB3LYP/6–31G* before any collision setup
- Molecular formula and exact mass in mol_info.in match the protonated species (e.g. mol_mf = C12H16N2O, mol_mw = 204 for psilocin)
- All chemically plausible protonation sites are represented by at least one optimized structure for multi-basic molecules
- Downstream CIDMD spectra built from the most probable protomer achieve entropy similarity ≈ 828 ± 77 against NIST20 reference spectra
- Simulation outputs remain stable across repeated optimizations of the same protomer (no SCF convergence failures at maxit 100, convthre 1e-4)

## Limitations

- Prediction accuracy decreases as the number of protomers and tautomers increases
- No method exists to accurately predict protomer formation states before running the computationally expensive CIDMD simulations
- Specific fragment ions can be produced from different protomers via multiple fragmentation pathways, so omitting a protomer can miss real fragments

## Evidence

- [methods] The 3D shape of ion structures was then optimized using TeraChem with an unrestricted B3LYP density functional and the 6–31G* basis set.: [evidence span withheld — anchor status: not_found]
- [readme] mol_name = Psilocin
mol_id   = PS
mol_mf   = C12H16N2O
mol_mw   = 204: "mol_name = Psilocin
mol_id   = PS
mol_mf   = C12H16N2O
mol_mw   = 204"
