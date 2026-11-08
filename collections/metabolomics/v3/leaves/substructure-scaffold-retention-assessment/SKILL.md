---
name: substructure-scaffold-retention-assessment
description: Use when you have multiple SMILES predictions per molecule plus ground-truth
  SMILES (e.g., processed_predictions.h5 from an NMR2Struct test-set run) and want
  to characterize the chemical proximity of wrong predictions via Tanimoto similarity.
when_to_use_negative:
- When you need the accuracy rate of exactly correct predictions (93.2% for the substructure-to-structure
  task) — that is a canonical-SMILES match metric, not a similarity analysis
- When evaluating predictions with only one candidate per input (no ranked pool from
  which to pick a 'best incorrect' prediction)
- When molecular size exceeds the model's scope of up to 19 heavy atoms, since error
  patterns are not characterized there (accuracy already drops below 80% at 18–19
  heavy atoms)
tools:
- name: RDKit
  role: Canonicalizes SMILES, computes Morgan-fingerprint Tanimoto similarities via
    DataStructs, and performs substructure matching against the 957 SMART substructures
- name: NMR2Struct
  role: Framework whose substructure-to-structure transformer inference produces processed_predictions.h5,
    the input pool of scored SMILES predictions
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:b436eed08ec9fec7caf5f0560bef5de9f4039c11edfbe1616a7fb1df209cce00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# substructure-scaffold-retention-assessment

## Summary

Quantifies how structurally close a structure-prediction model's best incorrect predictions are to the ground-truth molecules by computing RDKit Morgan-fingerprint Tanimoto similarities and checking whether target scaffolds (from the 957-SMART substructure set) survive in near-threshold predictions. It diagnoses whether errors are catastrophic scaffold misses or near-misses.

## When to use

Use when you have multiple SMILES predictions per molecule plus ground-truth SMILES (e.g., processed_predictions.h5 from an NMR2Struct test-set run) and want to characterize the chemical proximity of wrong predictions via Tanimoto similarity.

## When NOT to use

- When you need the accuracy rate of exactly correct predictions (93.2% for the substructure-to-structure task) — that is a canonical-SMILES match metric, not a similarity analysis
- When evaluating predictions with only one candidate per input (no ranked pool from which to pick a 'best incorrect' prediction)
- When molecular size exceeds the model's scope of up to 19 heavy atoms, since error patterns are not characterized there (accuracy already drops below 80% at 18–19 heavy atoms)

## Inputs

- processed_predictions.h5 containing highest-scoring per-molecule SMILES predictions from the substructure-to-structure transformer test set
- Ground-truth canonical SMILES for each test molecule
- substructures_957.p — pickle of SMART strings for the 957 substructures used by the model

## Outputs

- tanimoto_similarity_incorrect_predictions.tsv with per-molecule Morgan-fingerprint Tanimoto similarity values
- Aggregate statistics: fraction of best incorrect predictions with Tanimoto similarity ≥0.50 and the mean similarity

## How to apply

Load predictions from processed_predictions.h5 and, for each test molecule, select the highest-scoring prediction whose canonical SMILES does not match the target — the 'best incorrect prediction' (15 predictions per input were generated via top-k sampling, k=5). Canonicalize the ground-truth SMILES and each best incorrect prediction with RDKit, skipping unparseable predictions. Compute the Morgan-fingerprint Tanimoto similarity between each best incorrect prediction and its ground truth using RDKit's DataStructs module. Summarize the distribution: the fraction of similarities ≥0.50 and the mean similarity, which the paper reports as 89.6% and 0.68 respectively. For predictions near the 0.50 threshold, run RDKit substructure matching against the 957 SMART substructures in substructures_957.p to qualitatively verify whether target scaffolds and functional groups are retained. Write per-molecule similarities plus the aggregate statistics to tanimoto_similarity_incorrect_predictions.tsv.

## Related tools

- **RDKit** (Canonicalizes SMILES, computes Morgan-fingerprint Tanimoto similarities via DataStructs, and performs substructure matching against the 957 SMART substructures)
- **NMR2Struct** (Framework whose substructure-to-structure transformer inference produces processed_predictions.h5, the input pool of scored SMILES predictions) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; sim = DataStructs.TanimotoSimilarity(AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(pred_canon), 2), AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(target_canon), 2))
```

## Evaluation signals

- Reproduced fraction of best incorrect predictions with Tanimoto similarity ≥0.50 matches the reported 89.6%
- Reproduced mean Tanimoto similarity matches the reported average of 0.68
- Every similarity is computed against canonical SMILES; unparseable predictions are explicitly skipped rather than silently scored
- tanimoto_similarity_incorrect_predictions.tsv contains one similarity value per test molecule plus the fraction ≥0.50 and mean as columns
- Qualitative SMART substructure matches against substructures_957.p confirm retained scaffolds/functional groups for near-threshold (≈0.50) predictions

## Limitations

- Analysis applies only to incorrect predictions from the 15-prediction pool (top-k sampling, k=5) per input; it cannot characterize failures on molecules where all 15 predictions were correct or absent
- Tanimoto similarity on Morgan fingerprints is a coarse proxy — predictions at ~0.50 similarity may retain substructures but differ in connectivity, so SMART-substructure inspection is needed for interpretation
- Underlying model is limited to molecules with up to 19 heavy atoms and does not predict stereochemistry, and it was trained on simulated (MestreNova) spectra, so incorrect-prediction patterns may differ on real spectra

## Evidence

- [results] 89.6% of the best incorrect predictions have a similarity to the correct molecule that is greater than or equal to 0.50, with an average similarity of 0.68.: "89.6% of the best incorrect predictions have a similarity to the correct molecule that is greater than or equal to 0.50, with an average similarity of 0.68."
- [results] After canonicalizing all the SMILES strings using RDKit, the substructure vector for each molecule was computed using RDKit's substructure match functionality: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [results] top-k random sampling was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [readme] Running inference writes the raw predictions as an hdf5 file to the save directory that you specified under the global_args section.: "Running inference writes the raw predictions as an hdf5 file to the save directory that you specified under the ```global_args``` section"
- [methods] the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used: "the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used"
