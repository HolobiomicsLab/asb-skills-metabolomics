---
name: bond-order-based-fragment-identification
description: Use when you have raw CIDMD trajectories (from TeraChem NVE B3LYP/6–31G*
  runs) and need to extract stable, charged fragment ions by analyzing Mayer bond
  order time series to build an in silico CID-MS/MS spectrum.
when_to_use_negative:
- Input trajectories have not been processed with LearnReactions.py and gathered into
  Molecule/cid/calcs/XXX/gathered/
- Working directory does not follow the required layout (Molecule/cid/CIDMD_analysis
  with mol_info.in present)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CIDMD_analysis
  role: Analyzes CIDMD trajectories using bond order and Mulliken charge criteria
    to identify stable charged fragments
  repo: https://github.com/jesilee/CIDMD_analysis
- name: nanoreactor (LearnReactions.py)
  role: Post-processes CIDMD trajectories to produce learn_rxn.log files that CIDMD_analysis
    consumes
  repo: https://github.com/leeping/nanoreactor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:2275564767eb5821efe4f74587b2091afeb3d3b2ab706fa8a20b8476b89fec01
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_analysis
  tools:
  - 6–31G*
  - Avogadro
  - B3LYP
  - CIDMD_analysis
  - Mayer bond orders
  - TeraChem
  - nanoreactor (LearnReactions.py)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Bond-order-based-fragment-identification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Identifies fragments in CIDMD trajectories by low-pass-filtering Mayer bond order time series with a 200 cm⁻¹ cutoff, discretizing at a 0.2 threshold, requiring ≥50 fs stability, and assigning charges via Mulliken populations with a 0.70 threshold. This converts raw ab initio MD collision outputs into stable fragment ion populations for spectra generation.

## When to use

Use when you have raw CIDMD trajectories (from TeraChem NVE B3LYP/6–31G* runs) and need to extract stable, charged fragment ions by analyzing Mayer bond order time series to build an in silico CID-MS/MS spectrum.

## When NOT to use

- Input trajectories have not been processed with LearnReactions.py and gathered into Molecule/cid/calcs/XXX/gathered/
- Working directory does not follow the required layout (Molecule/cid/CIDMD_analysis with mol_info.in present)

## Inputs

- CIDMD trajectories post-processed with LearnReactions.py (learn_rxn.log files under Molecule/cid/calcs/XXX/gathered/)
- mol_info.in file describing molecule name, formula, mass

## Outputs

- Molecule/cid/results/ folder containing cidmd.jdx and cidmd.msp (theoretical mass spectra in JDX and MSP format for NIST or other mass spectral libraries)
- Molecule/cid/results/ folder containing cidmd.PS_case1_RCID_vfac8-cut05norm.png, cidmd.PS_case1_RCID_vfac8-cut05.png, ar_vel_KE.png, sy_found.out, total_gputime.txt, pop_found.out, rxn_found.out, report_found.out

## How to apply

Run CIDMD_analysis on trajectories post-processed with LearnReactions.py, after placing this repo at Molecule/cid/CIDMD_analysis and supplying mol_info.in. Raw Mayer bond order time series are low-pass-filtered with a 200 cm⁻¹ upper frequency cutoff, then discretized by applying a threshold of 0.2 to call bonds broken/formed. Fragments must persist for at least 50 fs to be counted stable; a fragment's charge is assigned as the total Mulliken population averaged over its existence, counted as charged if the mean exceeds 0.70. Each trajectory's fragments are accumulated over repeated collisions from different directions to produce the final fragment distribution that feeds spectra writing (cidmd.msp/cidmd.jdx).

## Related tools

- **CIDMD_analysis** (Analyzes CIDMD trajectories using bond order and Mulliken charge criteria to identify stable charged fragments) — https://github.com/jesilee/CIDMD_analysis
- **nanoreactor (LearnReactions.py)** (Post-processes CIDMD trajectories to produce learn_rxn.log files that CIDMD_analysis consumes) — https://github.com/leeping/nanoreactor

## Examples

```
cd Molecule/cid && git clone https://github.com/jesilee/CIDMD_analysis && cd CIDMD_analysis && python3 analyze.py
```

## Evaluation signals

- Molecule/cid/results/ folder exists containing cidmd.jdx and cidmd.msp
- sy_found.out, pop_found.out, rxn_found.out, and report_found.out are populated with fragment statistics
- A fragment must persist for ≥50 fs to be recognized as stable; fragments below this cutoff should be excluded
- Mean Mulliken charge of a fragment over its existence must exceed 0.70 for it to be counted as a charged ion

## Limitations

- Accurate protomer formation itself cannot yet be predicted; fewer computationally intensive CIDMD simulations would be needed if protomer formation could be accurately predicted beforehand
- Discretization and thresholding of bond order time series may miss fragments whose bond orders fluctuate near the 0.2 threshold
- Results depend on upstream trajectory quality (vfac, b cutoff, number of trajectories)

## Evidence

- [methods] Fragment stability and charge thresholds: [evidence span withheld — anchor status: not_found]
- [readme] Repo organization and required files: [evidence span withheld — anchor status: external_doc]
- [readme] Output file list: [evidence span withheld — anchor status: external_doc]
