---
name: tanimoto-similarity-computation-of-incorrect-smiles-predictions
description: Use when evaluating a SMILES-generating model (e.g., substructure-to-structure
  transformer) on a test set where exact-match accuracy was computed, and you need
  to characterize near-miss predictions structurally via Tanimoto similarity.
when_to_use_negative:
- The prediction already matches the target SMILES exactly — exact-match accuracy,
  not similarity, is the relevant metric
- Input structures lack valid SMILES strings that RDKit can parse/canonicalize (unparseable
  predictions must be skipped, so similarity cannot be computed)
- Evaluating substructure-prediction performance itself (binary 957-substructure vectors),
  where substructure-vector accuracy rather than molecular similarity is the target
  metric
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3474
tools:
- name: RDKit
  role: Canonicalizes SMILES, computes Morgan fingerprints and Tanimoto similarities
    via DataStructs, and performs substructure matching against 957 SMART patterns
- name: transformer
  role: Substructure-to-structure model whose best incorrect SMILES predictions are
    analyzed; generates predictions via top-k sampling (k=5, 15 per input)
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: NMR2Struct
  role: Package providing the inference pipeline that writes raw predictions as hdf5
    and the example_configs/substructures_957.p SMART set
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:f44ab1276c4400031cfb9dc1c5c373290c30d278f76091c609cf25636757751d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - transformer
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tanimoto-similarity-computation-of-incorrect-smiles-predictions

## Summary

Quantifies how structurally close a generative model's wrong SMILES predictions are to the ground-truth molecules by computing Morgan-fingerprint Tanimoto similarities of the best incorrect predictions. Used in NMR2Struct to show that even incorrect substructure-to-structure transformer predictions retain substantial target scaffold information.

## When to use

Use when evaluating a SMILES-generating model (e.g., substructure-to-structure transformer) on a test set where exact-match accuracy was computed, and you need to characterize near-miss predictions structurally via Tanimoto similarity.

## When NOT to use

- The prediction already matches the target SMILES exactly — exact-match accuracy, not similarity, is the relevant metric
- Input structures lack valid SMILES strings that RDKit can parse/canonicalize (unparseable predictions must be skipped, so similarity cannot be computed)
- Evaluating substructure-prediction performance itself (binary 957-substructure vectors), where substructure-vector accuracy rather than molecular similarity is the target metric

## Inputs

- processed_predictions.h5 — hdf5 file of model test-set predictions with per-molecule candidate SMILES strings and scores
- Ground-truth SMILES strings for each test molecule (numpy binary-encoded array; decode with x.decode('utf-8'))
- RDKit Morgan fingerprint parameters (default radius/length)
- substructures_957.p — pickle of 957 SMART substructure strings for optional scaffold-retention checks

## Outputs

- tanimoto_similarity_incorrect_predictions.tsv — per-molecule Tanimoto similarity values plus fraction ≥0.50 and mean similarity
- Summary statistics: fraction of best incorrect predictions with Tanimoto similarity ≥0.50 (reported: 89.6%) and average similarity (reported: 0.68)

## How to apply

From the saved inference output (processed_predictions.h5), select for each molecule the highest-scoring prediction whose canonical SMILES does not match the target — the 'best incorrect prediction'. Canonicalize both the prediction and ground-truth SMILES with RDKit, skipping unparseable predictions. Compute Morgan-fingerprint Tanimoto similarity between each pair with RDKit's DataStructs module. Summarize the distribution as the fraction of similarities ≥0.50 and the mean similarity; for NMR2Struct's test set the expected values are 89.6% ≥0.50 and mean 0.68, indicating wrong predictions still carry target scaffold information. Optionally, for predictions near 0.50, run RDKit substructure matching against the 957 SMART substructures in substructures_957.p to check qualitatively whether scaffolds and functional groups are retained. Write per-molecule similarities plus the fraction ≥0.50 and the mean to a tab-separated output file (e.g., tanimoto_similarity_incorrect_predictions.tsv).

## Related tools

- **RDKit** (Canonicalizes SMILES, computes Morgan fingerprints and Tanimoto similarities via DataStructs, and performs substructure matching against 957 SMART patterns)
- **transformer** (Substructure-to-structure model whose best incorrect SMILES predictions are analyzed; generates predictions via top-k sampling (k=5, 15 per input)) — https://github.com/MarklandGroup/NMR2Struct
- **NMR2Struct** (Package providing the inference pipeline that writes raw predictions as hdf5 and the example_configs/substructures_957.p SMART set) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; sim = DataStructs.TanimotoSimilarity(AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(pred), 2), AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(target), 2))
```

## Evaluation signals

- Reproduced fraction of best incorrect predictions with Tanimoto similarity ≥0.50 should match 89.6%
- Reproduced average Tanimoto similarity should match 0.68
- All compared SMILES pairs must be RDKit-canonicalizable; count and report skipped unparseable predictions
- Selected 'best incorrect' prediction per molecule must be the highest-scoring candidate whose canonical SMILES differs from the target
- Output TSV must contain per-molecule similarity values, fraction ≥0.50, and mean similarity as tab-separated values

## Limitations

- Similarity is computed on Morgan fingerprints, so high similarity does not guarantee connectivity differences are chemically trivial — qualitative substructure checks near 0.50 are needed
- Analysis applies only to molecules the NMR2Struct framework covers (up to 19 heavy atoms, C/N/O-containing molecules trained on simulated MestreNova spectra)
- Requires multiple sampled predictions per input (15 via top-k sampling with k=5); it is undefined for greedy single-prediction decoders without modification

## Evidence

- [results] 89.6% of the best incorrect predictions have a similarity to the correct molecule that is greater than or equal to 0.50, with an average similarity of 0.68.: "89.6% of the best incorrect predictions have a similarity to the correct molecule that is greater than or equal to 0.50, with an average similarity of 0.68."
- [results] top-k sampling details: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] RDKit canonicalization and substructure matching: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [methods] substructures file location: "the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used"
- [readme] predictions written as hdf5: "Running inference writes the raw predictions as an hdf5 file to the save directory that you specified under the ```global_args``` section"
