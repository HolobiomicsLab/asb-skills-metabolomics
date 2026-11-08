---
name: mces-leakage-filtering-against-test-fold
description: Use when building an auxiliary pretraining corpus (e.g. fingerprint-molecule
  pairs from unlabeled PubChem molecules) that must not leak structures from held-out
  benchmark test sets such as MassSpecGym test folds or NPLIB1.
when_to_use_negative:
- The corpus is already the MassSpecGym training fold itself (internal split leakage
  is handled by the benchmark, not this filter)
- Input molecules are intended for evaluation on NPLIB1 or MassSpecGym rather than
  for pretraining
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: RDKit
  role: Computes the 4096-bit circular Morgan fingerprints whose Tanimoto similarity
    is thresholded at 0.5 against NPLIB1; RDKit structures are also the basis for
    MCES distance computation
  repo: https://github.com/OpenDFM/MS-BART
- name: MS-BART
  role: Consumer of the filtered pretraining corpus; its preprocessing scripts (preprocess/split_pretrain_dataset.py)
    implement and validate the leakage constraints
  repo: https://github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:afc7e68745530e523fd008b5fb2c82d35afe82902674e4c31779df263761baf8
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - MS-BART
  - RDKit
  - SMILES
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mces-leakage-filtering-against-test-fold

## Summary

Removes molecules from an auxiliary pretraining corpus that are too chemically similar to benchmark evaluation molecules, by excluding pairs with MCES distance < 2 against the MassSpecGym test fold and Tanimoto similarity > 0.5 against NPLIB1, preventing cross-split leakage.

## When to use

Use when building an auxiliary pretraining corpus (e.g. fingerprint-molecule pairs from unlabeled PubChem molecules) that must not leak structures from held-out benchmark test sets such as MassSpecGym test folds or NPLIB1.

## When NOT to use

- The corpus is already the MassSpecGym training fold itself (internal split leakage is handled by the benchmark, not this filter)
- Input molecules are intended for evaluation on NPLIB1 or MassSpecGym rather than for pretraining

## Inputs

- 4M unlabeled molecule table (e.g. data/MassSpecGym/data/molecules/MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv)
- MassSpecGym test-fold molecule structures
- NPLIB1 molecule structures
- RDKit molecular objects / SMILES strings

## Outputs

- Filtered pretraining corpus (pubchem-4M/train.tsv and pubchem-4M/val.tsv under logs/datasets/MassSpecGym/) containing only leakage-safe fingerprint-token/SELFIES pairs
- 10,000-entry reserved validation split

## How to apply

Enumerate unlabeled molecules (MS-BART uses the 4M MassSpecGym molecule table), then for each candidate compute the maximum common edge substructure (MCES) distance against every molecule in the benchmark test fold and drop candidates with MCES distance less than two; separately compute Tanimoto similarity (e.g. on the 4096-bit Morgan fingerprints already generated with RDKit) against all NPLIB1 molecules and exclude candidates with Tanimoto similarity greater than 0.5. Apply both filters before emitting fingerprint-token/SELFIES pairs so that no training pair is structurally near-identical to an evaluation molecule. Rationale: fingerprints are the model's spectral input representation, so near-duplicate scaffolds in pretraining would let the model memorize test-fold structures rather than generalize from spectra.

## Related tools

- **RDKit** (Computes the 4096-bit circular Morgan fingerprints whose Tanimoto similarity is thresholded at 0.5 against NPLIB1; RDKit structures are also the basis for MCES distance computation) — https://github.com/OpenDFM/MS-BART
- **MS-BART** (Consumer of the filtered pretraining corpus; its preprocessing scripts (preprocess/split_pretrain_dataset.py) implement and validate the leakage constraints) — https://github.com/OpenDFM/MS-BART

## Examples

```
python preprocess/split_pretrain_dataset.py --pretrain-data pubchem-4M/ --out logs/datasets/MassSpecGym/ --val-size 10000
```

## Evaluation signals

- No produced pair has MCES distance < 2 against any MassSpecGym test-fold molecule
- No produced pair has Tanimoto similarity > 0.5 against any NPLIB1 molecule
- pubchem-4M/train.tsv and pubchem-4M/val.tsv exist, parse as tab-separated tables with fingerprint-token and SELFIES columns
- The validation split contains exactly 10,000 entries
- Every SELFIES string is within the 185-token vocabulary

## Limitations

- Filtering reduces pretraining corpus size, so overly strict thresholds (e.g. MCES ≥ 3) would discard usable unlabeled molecules
- The MCES filter guards against test-fold similarity but does not remove exact duplicates already absent from the 4M table by construction (the table is pre-filtered as 'MCES2_disjoint_with_test_fold')
- Filter thresholds are benchmark-specific; reusing MCES < 2 / Tanimoto > 0.5 cutoffs against a different benchmark requires recomputing reference structures

## Evidence
