---
name: adduct-specific-ccs-benchmarking
description: Use when you have a dataset of compounds with SMILES, an adduct type
  from {[M+H]+, [M-H]-, [M+Na]+}, and experimental CCS values, and need to quantitatively
  compare two or more CCS predictors.
when_to_use_negative:
- The task is training or fine-tuning a CCS model rather than evaluating existing
  predictors against experimental values
- The compounds carry adduct types outside the three SigmaCCS-encoded types ([M+H]+,
  [M-H]-, [M+Na]+)
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se,
  since SigmaCCS cannot predict them
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SigmaCCS
  role: GNN-based CCS predictor using 3D conformer-derived molecular graphs merged
    with adduct one-hot encoding; applied via GraphData.convertToGraph and Model_prediction
    with the pre-trained model.h5
  repo: https://github.com/zmzhang/SigmaCCS
- name: CCSbase
  role: Baseline CCS predictor for comparison; uses K-Means clustering of chemical
    structures followed by cluster-specific regression models
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:754304586d0e29aa245fac22cb51026ee5811dff7f41829e4d537a6ba27fdf09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - CCSbase
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# adduct-specific CCS benchmarking

## Summary

Benchmark CCS prediction models (e.g. SigmaCCS vs CCSbase) on a molecule set stratified by the three encoded adduct ion types ([M+H]+, [M-H]-, [M+Na]+), comparing experimental CCS values against model predictions using R2, RMSE, and median relative error.

## When to use

Use when you have a dataset of compounds with SMILES, an adduct type from {[M+H]+, [M-H]-, [M+Na]+}, and experimental CCS values, and need to quantitatively compare two or more CCS predictors.

## When NOT to use

- The task is training or fine-tuning a CCS model rather than evaluating existing predictors against experimental values
- The compounds carry adduct types outside the three SigmaCCS-encoded types ([M+H]+, [M-H]-, [M+Na]+)
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se, since SigmaCCS cannot predict them

## Inputs

- Dataset with SMILES strings, adduct type ([M+H]+, [M-H]-, or [M+Na]+), and experimental CCS values (e.g. 114-entry plant dataset, Supplementary Data 6)
- Trained SigmaCCS model files (model.h5, parameter.pkl) from the zmzhang/SigmaCCS repository
- Comparator model predictions (e.g. CCSbase K-Means cluster-specific predictions) for the same entries

## Outputs

- ccs_evaluation_metrics.csv with R2, RMSE, and Median RE (%) per predictor
- Predicted vs experimental CCS scatter plots (Supplementary Fig. 3c,d equivalents)

## How to apply

Load the annotated compound set (e.g. the 114-entry plant dataset from Supplementary Data 6 with SMILES, adduct type, and experimental CCS). For each entry, generate a 3D conformer with ETKDGv3 and MMFF94 minimization, then use GraphData.convertToGraph to build the node feature matrix, adjacency matrix, and edge feature matrix, and run Model_prediction with the repository's trained model.h5 and parameter.pkl to obtain SigmaCCS predictions for the entry's specific adduct. Obtain comparison predictions (e.g. CCSbase, which assigns structures to K-Means clusters and applies cluster-specific regression models) for the same entries. Compute R2, RMSE, and median relative error (%) against experimental CCS with scikit-learn regression metrics, assemble a metrics table (e.g. ccs_evaluation_metrics.csv), and optionally produce predicted-vs-experimental scatter plots reproducing Supplementary Fig. 3c,d.

## Related tools

- **SigmaCCS** (GNN-based CCS predictor using 3D conformer-derived molecular graphs merged with adduct one-hot encoding; applied via GraphData.convertToGraph and Model_prediction with the pre-trained model.h5) — https://github.com/zmzhang/SigmaCCS
- **CCSbase** (Baseline CCS predictor for comparison; uses K-Means clustering of chemical structures followed by cluster-specific regression models)

## Examples

```
from sigma.GraphData import convertToGraph; adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms); pred = Model_prediction(graph, adduct, 'model/model.h5', 'parameter/parameter.pkl')
```

## Evaluation signals

- Computed SigmaCCS metrics on the 114-entry plant dataset reproduce R2 = 0.9655, RMSE = 5.3812, Median RE = 1.4232% within reasonable tolerance
- Computed CCSbase metrics reproduce R2 = 0.9643, RMSE = 5.4720, Median RE = 2.3211% within reasonable tolerance
- Scatter plots of predicted vs experimental CCS visually match Supplementary Fig. 3c,d
- Every dataset entry yields a valid conformer via ETKDG/MMFF94 and a graph built by GraphData.convertToGraph with no dropped predictions

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- ETKDG conformer generation can fail (e.g. 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries), and MMFF94 minimization failed for 662 molecules
- CCSbase can misassign molecules to unsuitable K-Means clusters, causing large deviations between predicted and experimental CCS
- SigmaCCS covers only three adduct ion types; other adducts are out of scope

## Evidence

- [results] On the plant dataset, SigmaCCS (R2 = 0.9655, RMSE = 5.3812, Median RE = 1.4232%) achieves better performance than CCSbase (R2 = 0.9643, RMSE = 5.4720, Median RE = 2.3211%).: [evidence span withheld — anchor status: not_found]
- [discussion] CCSbase method description: "CCSbase uses K-Means clustering for the untargeted classification of chemical structures and then performs CCS predictions using specific models"
- [readme] README graph construction method: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph"
- [methods] prediction procedure: "The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model."
- [discussion] element coverage limitation: "Although SigmaCCS cannot predict molecules of elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se"
