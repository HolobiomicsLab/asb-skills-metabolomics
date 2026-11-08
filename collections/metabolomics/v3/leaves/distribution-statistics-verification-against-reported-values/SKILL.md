---
name: distribution-statistics-verification-against-reported-values
description: Use when reimplementing a model evaluation pipeline and you need to confirm
  summary metrics like '% of predictions with Tanimoto >= 0.50' or mean similarity
  match the article's reported numbers.
when_to_use_negative:
- The task requires classifying predictions as correct or incorrect from scratch rather
  than verifying already-computed similarity distributions
- The metric is not a pairwise molecular similarity distribution — e.g., verifying
  substructure prediction accuracy or spectrum-to-structure top-1 hit rates needs
  different aggregation logic
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: RDKit
  role: Canonicalizes SMILES, computes Morgan fingerprints and Tanimoto similarity
    between best incorrect predictions and target molecules
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: NMR2Struct
  role: Produces the processed test-set predictions whose best incorrect entries are
    the input to the similarity distribution
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer
  role: The substructure-to-structure model whose predictions are being evaluated
    for scaffold retention
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:e9230f0a8e7d8586f0ccffdd8626ecc3a054d188b6675309960fd4c2599d2c3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - transformer
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Distribution statistics verification against reported values

## Summary

Reproduces a paper's summary distribution statistics (e.g., fraction above a similarity threshold, mean of a metric) from per-item predictions and checks them against the published values to validate a reimplementation.

## When to use

Use when reimplementing a model evaluation pipeline and you need to confirm summary metrics like '% of predictions with Tanimoto >= 0.50' or mean similarity match the article's reported numbers.

## When NOT to use

- The task requires classifying predictions as correct or incorrect from scratch rather than verifying already-computed similarity distributions
- The metric is not a pairwise molecular similarity distribution — e.g., verifying substructure prediction accuracy or spectrum-to-structure top-1 hit rates needs different aggregation logic

## Inputs

- Processed predictions for the substructure-to-structure transformer test set (e.g., from processed_predictions.h5), including per-molecule ranked SMILES predictions and target SMILES
- Target ground-truth SMILES strings for the test set

## Outputs

- Per-molecule Tanimoto similarity values for best incorrect predictions (tab-separated, e.g., tanimoto_similarity_incorrect_predictions.tsv)
- Fraction of best incorrect predictions with Tanimoto similarity >= 0.50
- Mean Tanimoto similarity over best incorrect predictions

## How to apply

For each item, select the relevant subset — here, the substructure-to-structure transformer's best incorrect test prediction (the highest-scoring SMILES that does not match the target). Canonicalize both target and prediction SMILES with RDKit, compute Morgan-fingerprint Tanimoto similarity per pair, then aggregate: count the fraction with similarity >= 0.50 and the mean similarity. Compare these aggregates against the article's reported values (89.6% at similarity >= 0.50, mean 0.68); agreement validates that the prediction selection, SMILES handling, and fingerprint choice in the reimplementation match the original. Write per-molecule values plus the two aggregate statistics to a tab-separated file (e.g., tanimoto_similarity_incorrect_predictions.tsv) so the check is auditable.

## Related tools

- **RDKit** (Canonicalizes SMILES, computes Morgan fingerprints and Tanimoto similarity between best incorrect predictions and target molecules) — https://github.com/MarklandGroup/NMR2Struct
- **NMR2Struct** (Produces the processed test-set predictions whose best incorrect entries are the input to the similarity distribution) — https://github.com/MarklandGroup/NMR2Struct
- **transformer** (The substructure-to-structure model whose predictions are being evaluated for scaffold retention) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; sims = [DataStructs.TanimotoSimilarity(AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(p), 2), AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(t), 2)) for p, t in preds]
```

## Evaluation signals

- Fraction of best incorrect predictions with Tanimoto similarity >= 0.50 reproduces the article's reported 89.6%
- Mean Tanimoto similarity reproduces the article's reported 0.68
- Every best incorrect prediction must be canonicalizable by RDKit — unparseable predictions are skipped and counted
- Per-molecule similarity file contains one row per test molecule with a valid best incorrect prediction

## Limitations

- Applies only to molecules whose predictions are incorrect (exact SMILES mismatch); correct predictions are excluded by construction
- Morgan-fingerprint Tanimoto similarity is a scaffold-level proxy — near-0.50 cases require qualitative substructure comparison (e.g., against the 957 SMART substructures) to interpret functional-group retention
- Relies on the paper's own choice of 'best incorrect' (highest-scoring non-matching prediction); different ranking conventions will shift the distribution

## Evidence

- [results] 89.6% of best incorrect substructure-to-structure predictions had Tanimoto similarity >= 0.50, with an average similarity of 0.68: "89.6% of the best incorrect predictions have a similarity to the correct molecule that is greater than or equal to 0.50, with an average similarity of 0.68."
- [results] Workflow uses RDKit canonicalization and substructure matching for qualitative follow-up: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [results] Best incorrect predictions are generated by top-k sampling with k=5, 15 predictions per input: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [methods] The 957-substructure SMART file provides the qualitative scaffold comparison basis: "the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used"
- [readme] HDF5 is the format in which predictions are stored and written during inference: "Running inference writes the raw predictions as an hdf5 file to the save directory that you specified under the ```global_args``` section"
