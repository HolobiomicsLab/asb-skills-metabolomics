---
name: prediction-vs-verified-structure-comparison
description: Use when G.A.M.E. predictions from csccp-solver-cli (--dec 5, -r 3) exist
  for the Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum or Angelica
  sp. datasets and you must confirm the expected verified compounds (5, 7, 7, 45 constitutions)
  were recovered.
when_to_use_negative:
- The task is to generate new G.A.M.E. predictions rather than validate existing ones
  against verified structures
- Input structures come from in silico benchmarking over the NPDB scaffold set (26,641
  scaffolds, 51,562 configurations) rather than the four verified natural-product
  testing datasets
- Masses are provided at fewer than the five decimal digits the practical automatic-elucidation
  validation requires, so a like-for-like comparison at D=5 is impossible
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: G.A.M.E.
  role: csccp-solver-cli produces the ranked predictions (probability, weight, SMILES)
    that are compared against verified structures
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Drives the GPU-parallelized DP (idp) kernel used to generate predictions on
    the NVIDIA Tesla K40c
- name: CUDA 7.5
  role: GPU toolkit underlying the idp dynamic-programming kernel executed during
    prediction runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:8a0af8f6b901224670c00c2702d6259576b2fe5c580789688701ab178fc062bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - CUDA 7.5
  - G.A.M.E.
  - NVIDIA Tesla K40c
  - PyCuda 2017.1
  - Python 2.7
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Prediction-vs-verified-structure comparison

## Summary

Validates G.A.M.E. csccp-solver-cli predictions by comparing the ranked predicted structures (probability, weight, SMILES) for each mass peak of the four natural-product testing datasets against the verified constitutions deposited in GAME/examples/Datasets/structures.

## When to use

Use when G.A.M.E. predictions from csccp-solver-cli (--dec 5, -r 3) exist for the Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum or Angelica sp. datasets and you must confirm the expected verified compounds (5, 7, 7, 45 constitutions) were recovered.

## When NOT to use

- The task is to generate new G.A.M.E. predictions rather than validate existing ones against verified structures
- Input structures come from in silico benchmarking over the NPDB scaffold set (26,641 scaffolds, 51,562 configurations) rather than the four verified natural-product testing datasets
- Masses are provided at fewer than the five decimal digits the practical automatic-elucidation validation requires, so a like-for-like comparison at D=5 is impossible

## Inputs

- Ranked solver output per mass peak: probability, weight, SMILES from csccp-solver-cli (DP/idp kernel, top-3, --dec 5)
- Verified constitutions in GAME/examples/Datasets/structures for the four natural-product datasets
- Target molecular weights per dataset from GAME/examples/MW/
- Seed scaffold index filenames (*.cIdx) per dataset from GAME/examples/Datasets/scaffolds and configuration indices (-c)

## Outputs

- One prediction CSV per dataset (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp) with ranked probability, weight, SMILES for every mass peak
- Match/mismatch report of predicted structures vs. the deposited verified constitutions, with per-dataset recovered-constitution counts (5, 7, 7, 45)

## How to apply

First aggregate the csccp-solver-cli output into one CSV per dataset (cuscuta_chinensis, ophiopogon_japonicus, polygonum_multiflorum, angelica_sp), recording for every mass peak the ranked predictions with probability, weight, and SMILES, using --dec 5 (five decimal digits) and -r 3 top ranking. Then load the verified (validated) main structures from GAME/examples/Datasets/structures and the seed scaffold lists from GAME/examples/Datasets/scaffolds, which also give the configuration index (-c) for each dataset (the configuration index runs from 0 to x-1 where x is the number on line 3 of the *.cIdx file). Compare each predicted SMILES/weight against the deposited verified constitutions; the correct run contains the expected compound identifiers for all four datasets (5, 7, 7, and 45 verified constitutions respectively). A prediction is judged correct when the solver, run with the seed scaffolds in GAME/examples/core_index and the target molecular weight from GAME/examples/MW/, recovers the verified structure among the top-ranked outputs within the mass_peak_min/mass_peak_max window implied by the ppm tolerance (e.g. ppm=5).

## Related tools

- **G.A.M.E.** (csccp-solver-cli produces the ranked predictions (probability, weight, SMILES) that are compared against verified structures) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Drives the GPU-parallelized DP (idp) kernel used to generate predictions on the NVIDIA Tesla K40c)
- **CUDA 7.5** (GPU toolkit underlying the idp dynamic-programming kernel executed during prediction runs)

## Examples

```
csccp-solver-cli -s GAME/examples/core_index -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- Each dataset's prediction CSV contains the expected compound identifiers for all four datasets (5, 7, 7, and 45 verified constitutions respectively)
- Predicted weights fall inside the solver-reported mass_peak_min/mass_peak_max window for each peak (ppm tolerance, e.g. ppm=5)
- Each prediction CSV matches the corresponding deposited verified structures in GAME/examples/Datasets/structures
- Solver output is well-formed: per-peak csccp info block plus top-3 ranked list of Probability, Weight, Smile entries with mass 168.195105-style formatting at 5 decimals
- Configuration index (-c) used for each dataset lies in the valid range 0 to x-1, where x is the number on line 3 of the seed *.cIdx file

## Limitations

- Validation is only possible for the four testing datasets with verified constitutions (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) for which deposited structures are provided
- The DP/G.A.M.E. ranking depends on the scaffold database with sidechain linking probabilities; only 26,641 of 83,242 scaffold files were suitable for testing with wmax = 500
- MS alone cannot elucidate structures of partially or completely unknown compounds, so comparison is against mixture constituents verified by prior natural-product studies

## Evidence

- [methods] Verified constitutions from four natural products: "verified constitutions from four natural products (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum and Angelica sp.)"
- [readme] Verified structures directory in repo: "The identified(validated) main structures in our four tesing natural products were provided in the `GAME/examples/Datasets/structures`"
- [readme] Top-3 ranked output format: "1.	1.972266e-04	1.681951e+02	C1C/C(=C\CC)C(=O)CC1CO"
- [abstract] Practical validation claim: "As exemplified by four testing datasets with verified constitutions from natural products, G.A.M.E. allows for efficient and automatic structural elucidation of unknown mixtures for practical"
- [intro] Five decimal digits requirement: "This method significantly decreases the computational time when five decimal digits of mass are required, thus allowing for efficient and automatic structure elucidation in practical cases."
