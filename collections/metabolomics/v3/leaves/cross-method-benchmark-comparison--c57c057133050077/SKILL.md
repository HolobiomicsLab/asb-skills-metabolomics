---
name: cross-method-benchmark-comparison
description: Use when two computational prediction methods must be compared on the
  same external test data and headline metrics (R2, RMSE, Median RE) are needed to
  judge which performs better without training-set leakage.
when_to_use_negative:
- The comparison set contains molecules present in either method's training set —
  deduplication against both training sets is required first to exclude data leakage
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se, or
  adducts other than [M+H]+, [M-H]-, [M+Na]+, since SigmaCCS cannot predict them
- Only one method's predictions are available — without matched predictions from both
  methods on identical entries, the benchmark is not meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: SigmaCCS
  role: GNN-based CCS predictor run on the deduplicated external test set via Model_prediction
    with model.h5 and parameter.pkl
  repo: https://github.com/zmzhang/SigmaCCS
- name: CCSbase
  role: 'Baseline predictor: K-Means cluster assignment of chemical structures followed
    by cluster-specific CCS models'
- name: RDKit
  role: Generates and optimizes one 3D conformer per molecule (ETKDGv3, MMFFOptimizeMoleculeConfs)
    and converts SMILES to molecular graphs
- name: scikit-learn
  role: Computes R2, RMSE, and median relative error regression metrics for both methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:363a5b77f868e8e146f4eb235b3d3f99c57ba3b5c072d1085b3ca68d2626be06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - CCSbase
  - RDKit
  - SigmaCCS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-method-benchmark-comparison

## Summary

Benchmark two CCS prediction methods (SigmaCCS vs CCSbase) on a shared, deduplicated external test set by computing R2, RMSE, and Median relative error against experimental CCS values, eliminating data-leakage confounds before comparing metrics.

## When to use

Use when two computational prediction methods must be compared on the same external test data and headline metrics (R2, RMSE, Median RE) are needed to judge which performs better without training-set leakage.

## When NOT to use

- The comparison set contains molecules present in either method's training set — deduplication against both training sets is required first to exclude data leakage
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se, or adducts other than [M+H]+, [M-H]-, [M+Na]+, since SigmaCCS cannot predict them
- Only one method's predictions are available — without matched predictions from both methods on identical entries, the benchmark is not meaningful

## Inputs

- 294-entry external test set CSV (Supplementary Data 5) with SMILES, adduct type, and experimental CCS values, deduplicated against both training sets
- Trained SigmaCCS model files: model.h5 and parameter.pkl
- CCSbase K-Means clustering and cluster-specific prediction models

## Outputs

- Predicted CCS values per molecule and adduct from SigmaCCS
- Predicted CCS values per molecule and adduct from CCSbase
- external_test_evaluation.csv containing R2, RMSE, and Median RE for both methods (SigmaCCS R2=0.9780, RMSE=6.7012, Median RE=1.8211%; CCSbase R2=0.9778, RMSE=6.7240, Median RE=1.3608%)

## How to apply

Start from the 294-entry external test set (Supplementary Data 5) with SMILES, adduct type ([M+H]+, [M-H]-, [M+Na]+), and experimental CCS values, deduplicated against the training sets of BOTH methods so neither gains an unfair advantage. Generate one 3D conformer per molecule with RDKit ETKDGv3 (EmbedMultipleConfs with numConfs=1, useRandomCoords=True, maxAttempts=1), optimize with MMFFOptimizeMoleculeConfs, then convert to molecular graphs via convertToGraph and run SigmaCCS Model_prediction(ifile, ParameterPath, mfileh5, ofile) with model.h5 and parameter.pkl. Obtain CCSbase predictions via K-Means cluster assignment followed by cluster-specific model prediction. Finally compute R2, RMSE, and Median relative error for both methods with scikit-learn regression metrics and write results to external_test_evaluation.csv.

## Related tools

- **SigmaCCS** (GNN-based CCS predictor run on the deduplicated external test set via Model_prediction with model.h5 and parameter.pkl) — https://github.com/zmzhang/SigmaCCS
- **CCSbase** (Baseline predictor: K-Means cluster assignment of chemical structures followed by cluster-specific CCS models)
- **RDKit** (Generates and optimizes one 3D conformer per molecule (ETKDGv3, MMFFOptimizeMoleculeConfs) and converts SMILES to molecular graphs)
- **scikit-learn** (Computes R2, RMSE, and median relative error regression metrics for both methods)

## Examples

```
re = AllChem.EmbedMultipleConfs(mol, numConfs=1, params=ps); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads=0); Model_prediction(ifile, ParameterPath, mfileh5, ofile)
```

## Evaluation signals

- Computed SigmaCCS metrics on the 294-entry set match R2=0.9780, RMSE=6.7012, Median RE=1.8211% within numerical tolerance (Supplementary Fig. 3a,b)
- Computed CCSbase metrics match R2=0.9778, RMSE=6.7240, Median RE=1.3608% within numerical tolerance
- Confirm the external set was deduplicated against both training sets — no entry should appear in TrainData.csv or the CCSbase training data
- Every entry has predictions from both methods for the correct adduct type ([M+H]+, [M-H]-, or [M+Na]+)
- Results file external_test_evaluation.csv contains all three metrics for both methods

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- ETKDG conformer generation failed for a small fraction of molecules (e.g., 5 of 5,645 CCSbase entries), and 662 molecules' ETKDG conformers could not be minimized by MMFF94
- CCSbase can suffer large deviations if a molecule is assigned to an unsuitable K-Means cluster

## Evidence

- [methods] Deduplication rationale: [evidence span withheld — anchor status: not_found]
- [methods] Conformer generation step: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [discussion] CCSbase baseline description: "CCSbase uses K-Means clustering for the untargeted classification of chemical structures and then performs CCS predictions using specific models trained on the corresponding cluster data."
- [results] Headline benchmark metrics: [evidence span withheld — anchor status: not_found]
- [methods] Graph conversion: "adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)"
