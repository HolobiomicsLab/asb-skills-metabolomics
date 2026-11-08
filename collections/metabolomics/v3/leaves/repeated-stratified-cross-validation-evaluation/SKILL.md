---
name: repeated-stratified-cross-validation-evaluation
description: Use when a labeled single-cell feature matrix (e.g., the SpaceM MORPHnMOL.csv
  spatio-molecular matrix with 88 metabolite intensity columns and fluorescence-derived
  cell-type labels) must be scored with an honest accuracy estimate before claiming
  a prediction benchmark.
when_to_use_negative:
- The input is already a scored prediction or a per-fold metric table — no further
  model fitting or cross-validation is needed.
- The task is unsupervised (e.g., discovering metabolic states in dHepaRG hepatocytes
  by clustering), where no ground-truth labels exist for stratified evaluation.
- Labels cannot be derived from fluorescence channels (no mCherry/GFP distinction),
  so stratification would be meaningless.
edam_operation: http://edamontology.org/operation_3799
tools:
- name: SCANPY
  role: Single-cell analysis toolkit used alongside the downstream processing of the
    SpaceM spatio-molecular matrix
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Execution environment for the scikit-learn LinearDiscriminantAnalysis and
    RepeatedStratifiedKFold evaluation
  repo: github.com/alexandrovteam/SpaceM
- name: SpaceM
  role: Produces the MORPHnMOL.csv spatio-molecular matrix that supplies the metabolite
    features and fluorescence labels for classification
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:62de39053101c6f5f290297ae2a0bb14816e280e1819e4d4665ba9e336150b6b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# repeated-stratified-cross-validation-evaluation

## Summary

Evaluates a supervised classifier (e.g., LinearDiscriminantAnalysis on SpaceM single-cell metabolite intensities) using repeated stratified k-fold cross-validation with scikit-learn's RepeatedStratifiedKFold, to obtain a robust accuracy estimate such as the 91.3% HeLa vs NIH3T3 cell-type prediction reported for SpaceM.

## When to use

Use when a labeled single-cell feature matrix (e.g., the SpaceM MORPHnMOL.csv spatio-molecular matrix with 88 metabolite intensity columns and fluorescence-derived cell-type labels) must be scored with an honest accuracy estimate before claiming a prediction benchmark.

## When NOT to use

- The input is already a scored prediction or a per-fold metric table — no further model fitting or cross-validation is needed.
- The task is unsupervised (e.g., discovering metabolic states in dHepaRG hepatocytes by clustering), where no ground-truth labels exist for stratified evaluation.
- Labels cannot be derived from fluorescence channels (no mCherry/GFP distinction), so stratification would be meaningless.

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv (per-cell normalized metabolite intensities plus microscopy-derived fluorescence channels)
- Ground-truth cell-type labels derived from the mCherry/GFP fluorescence decision boundary (HeLa = mCherry-positive, NIH3T3 = GFP-positive)
- Selected metabolite feature columns (88 features in the published HeLa/NIH3T3 analysis)

## Outputs

- Cross-validated classification accuracy (target: ~91.3% for HeLa vs NIH3T3)
- CSV containing per-cell metabolite intensities, cell-type labels, and the reported cross-validation accuracy result

## How to apply

Load the spatio-molecular matrix (MORPHnMOL.csv) and derive ground-truth labels from the microscopy fluorescence decision boundary — HeLa cells are mCherry-positive, NIH3T3 cells are GFP-positive. Select the published metabolite feature set (88 columns here) as X and the fluorescence-derived labels as y. Fit a scikit-learn LinearDiscriminantAnalysis classifier and wrap it in RepeatedStratifiedKFold (n_splits=10, repeated) inside cross_val_score so each fold preserves the cell-type class proportions. Average the fold accuracies into a single performance estimate and compare it against the reported value (~91.3%) to judge replication. Stratification matters because class sizes differ between cell types, and repetition reduces variance from fold assignment.

## Related tools

- **SCANPY** (Single-cell analysis toolkit used alongside the downstream processing of the SpaceM spatio-molecular matrix) — github.com/alexandrovteam/SpaceM
- **Python** (Execution environment for the scikit-learn LinearDiscriminantAnalysis and RepeatedStratifiedKFold evaluation) — github.com/alexandrovteam/SpaceM
- **SpaceM** (Produces the MORPHnMOL.csv spatio-molecular matrix that supplies the metabolite features and fluorescence labels for classification) — github.com/alexandrovteam/SpaceM

## Examples

```
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis; from sklearn.model_selection import cross_val_score, RepeatedStratifiedKFold; scores = cross_val_score(LinearDiscriminantAnalysis(), X, y, cv=RepeatedStratifiedKFold(n_splits=10, n_repeats=10))
```

## Evaluation signals

- Repeated stratified 10-fold cross-validation accuracy on the 88 metabolite features should reproduce the reported ~91.3% for HeLa vs NIH3T3 prediction.
- Class proportions should be preserved across folds (stratified split), verifiable by checking per-fold label distributions.
- Labels must match the fluorescence decision boundary: all HeLa cells mCherry-positive, all NIH3T3 cells GFP-positive.
- Output CSV must contain per-cell metabolite intensities, cell-type labels, and the cross-validation accuracy result.
- Accuracy should be averaged over repeated folds rather than a single train/test split, to reduce fold-assignment variance.

## Limitations

- Co-sampling of neighboring cells by the same ablation mark only minimally affects cell-type prediction accuracy but is a known confounder in spatially-heterogeneous co-cultures.
- The paper notes a current lack of established validation approaches in single-cell metabolomics, so cross-validated accuracy is an internal rather than gold-standard benchmark.
- No version numbers are given for CellProfiler, Fiji, or Python, so exact reproduction of the environment is not specified.

## Evidence
