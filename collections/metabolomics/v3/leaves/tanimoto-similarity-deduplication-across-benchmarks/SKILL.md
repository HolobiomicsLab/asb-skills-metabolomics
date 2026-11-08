---
name: tanimoto-similarity-deduplication-across-benchmarks
description: Use when assembling a large pretraining corpus of fingerprint–molecule
  or spectrum–molecule pairs where unlabeled sources (e.g. PubChem) could structurally
  overlap held-out test molecules of MassSpecGym or NPLIB1, inflating benchmark scores
  via memorization.
when_to_use_negative:
- The corpus being assembled is already disjoint from benchmark test molecules (e.g.
  sourced from a curated subset guaranteed not to overlap MassSpecGym/NPLIB1), so
  no deduplication is needed.
- The goal is to evaluate on NPLIB1 or MassSpecGym without modifying pretraining data
  — this skill modifies the pretraining corpus, not the benchmark.
- The molecular structures lack computable Morgan fingerprints (e.g. no valid SMILES/SELFIES),
  so MCES/Tanimoto calculations cannot be performed.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: computes the 4096-bit circular Morgan fingerprints from molecular structures
    used for both the pretraining input and the Tanimoto similarity comparisons against
    NPLIB1 molecules
- name: SELFIES
  role: canonical molecular string representation whose tokens are denoised and translated
    from fingerprint tokens during pretraining; the deduplicated corpus feeds these
    translation/denoising tasks
- name: MS-BART
  role: consumes the deduplicated fingerprint–molecule pairs for multi-task pretraining
    (SELFIES denoising, fingerprint-to-molecule translation, hybrid denoising), ensuring
    the pretrained model does not see test-fold structures
  repo: github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:395703110439a3702187e525c0c034feecc99a02ccae008cd25e14755242d276
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-BART
  - RDKit
  - SELFIES
  - SMILES
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tanimoto-similarity-deduplication-across-benchmarks

## Summary

Filters pretraining pairs (e.g. fingerprint–molecule pairs computed for 4M unlabeled molecules) by removing molecules too structurally similar to molecules held out in benchmark test folds — MCES distance < 2 from MassSpecGym test-fold molecules and Tanimoto similarity > 0.5 to NPLIB1 molecules — to prevent structural leakage from pretraining data into evaluation sets.

## When to use

Use when assembling a large pretraining corpus of fingerprint–molecule or spectrum–molecule pairs where unlabeled sources (e.g. PubChem) could structurally overlap held-out test molecules of MassSpecGym or NPLIB1, inflating benchmark scores via memorization.

## When NOT to use

- The corpus being assembled is already disjoint from benchmark test molecules (e.g. sourced from a curated subset guaranteed not to overlap MassSpecGym/NPLIB1), so no deduplication is needed.
- The goal is to evaluate on NPLIB1 or MassSpecGym without modifying pretraining data — this skill modifies the pretraining corpus, not the benchmark.
- The molecular structures lack computable Morgan fingerprints (e.g. no valid SMILES/SELFIES), so MCES/Tanimoto calculations cannot be performed.

## Inputs

- unlabeled molecule TSV table with SMILES/SELFIES and structure identifiers (e.g. MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv, 4M molecules)
- reference set of MassSpecGym test-fold molecules
- reference set of NPLIB1 molecules

## Outputs

- filtered pretraining corpus of fingerprint-token/SELFIES pairs disjoint from benchmark test molecules (train.tsv and val.tsv, with 10,000 held-out entries in val)
- clean pretrain data directory (e.g. logs/datasets/MassSpecGym/pubchem-4M/)

## How to apply

Load the unlabeled molecule table (e.g. data/MassSpecGym/data/molecules/MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv, 4 million molecules). Compute a 4096-bit circular Morgan fingerprint for each molecule using RDKit, and encode structures as canonical SELFIES strings. For each candidate, compute MCES distance against every MassSpecGym test-fold molecule and exclude candidates with MCES < 2; compute Tanimoto similarity against every NPLIB1 molecule and exclude candidates with Tanimoto > 0.5. Only pairs passing both filters are written to the pretrain split, producing a corpus structurally disjoint from evaluation molecules so pretraining cannot leak test-set structures. The resulting train/val tables (e.g. pubchem-4M/train.tsv and pubchem-4M/val.tsv under logs/datasets/MassSpecGym/) are then used for MS-BART multi-task pretraining without contaminating benchmark evaluation.

## Related tools

- **RDKit** (computes the 4096-bit circular Morgan fingerprints from molecular structures used for both the pretraining input and the Tanimoto similarity comparisons against NPLIB1 molecules)
- **SELFIES** (canonical molecular string representation whose tokens are denoised and translated from fingerprint tokens during pretraining; the deduplicated corpus feeds these translation/denoising tasks)
- **MS-BART** (consumes the deduplicated fingerprint–molecule pairs for multi-task pretraining (SELFIES denoising, fingerprint-to-molecule translation, hybrid denoising), ensuring the pretrained model does not see test-fold structures) — github.com/OpenDFM/MS-BART

## Examples

```
python preprocess/split_pretrain_dataset.py --input data/MassSpecGym/data/molecules/MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv --output logs/datasets/MassSpecGym/pubchem-4M/ --val-size 10000
```

## Evaluation signals

- Every molecule in the output train.tsv/val.tsv must have MCES distance ≥ 2 from all MassSpecGym test-fold molecules (verify by re-running MCES against the test fold).
- Every molecule in the output corpus must have Tanimoto similarity ≤ 0.5 against all NPLIB1 molecules (verify by re-computing RDKit Morgan Tanimoto against the NPLIB1 set).
- Validation split size should be 10,000 entries as specified by preprocess/split_pretrain_dataset.py.
- Output tables must parse as tab-separated files with fingerprint-token and SELFIES columns.
- Leakage filter is applied before any model training; if the pretrain-data directory is populated with a 'clean' subset (e.g. pretrain-data under CANOPUS), that subset should correspond to the filtered corpus.

## Limitations

- The 0.5 Tanimoto threshold and MCES-2 cutoff are specific to MassSpecGym/NPLIB1 benchmarks; other benchmarks may require different similarity thresholds.
- MCES distance computation across 4M molecules against test folds is computationally expensive and may require approximate or indexed similarity search.
- The filter only guards against structural leakage into the named benchmarks — it does not guarantee the pretraining corpus is free from other forms of contamination (e.g. formula overlap).
- Excluding structurally similar molecules may reduce the diversity of pretraining pairs, potentially limiting generalization to novel structures.

## Evidence

- [methods] Pretrain data MCES-distance filter: [evidence span withheld — anchor status: not_found]
