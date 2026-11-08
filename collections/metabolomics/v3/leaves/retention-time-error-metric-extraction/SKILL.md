---
name: retention-time-error-metric-extraction
description: Decision support for comparing retention-time prediction accuracy across
  candidate models when multiple methods have been evaluated on the METLIN SMRT dataset
  and you need to rank mean/median relative error.
when_to_use_negative:
- Input is a raw spectral file or molecular graph rather than already-computed retention-time
  error metrics — use the Preprocess/Train pipeline instead.
- The task is training or transfer learning on an in-house database rather than benchmarking
  prediction accuracy of a fitted model.
- Only one model's error is available, so cross-model ranking is impossible.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: Anaconda for python 3.6
  role: Runtime environment in which the metric-extraction pandas analysis is run
- name: rdkit
  role: Chemistry toolkit used by the GNN-RT pipeline whose evaluation outputs feed
    the metric table
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c04071
    title: Prediction of Liquid Chromatographic Retention Time with Graph Neural Networks
      to Assist in Small Molecule Identification
  source_content_hash: sha256:a83f15c218f738d85034f3405c77225a717d9ffe21a32b7840dfcbbcc0f9f39d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Qiong-Yang/GNN-RT
  tools:
  - Anaconda for python 3.6
  - rdkit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# retention-time-error-metric-extraction

## Summary

Extracts and ranks mean and median relative error metrics for GNN-RT versus comparison regression models (random forest, Bayesian ridge, CNN, DLM) on the METLIN SMRT retention-time dataset, confirming the reported benchmark numbers.

## When to use

Decision support for comparing retention-time prediction accuracy across candidate models when multiple methods have been evaluated on the METLIN SMRT dataset and you need to rank mean/median relative error.

## When NOT to use

- Input is a raw spectral file or molecular graph rather than already-computed retention-time error metrics — use the Preprocess/Train pipeline instead.
- The task is training or transfer learning on an in-house database rather than benchmarking prediction accuracy of a fitted model.
- Only one model's error is available, so cross-model ranking is impossible.

## Inputs

- evaluation outputs from GNN-RT and comparison models (RF, Bayesian ridge, CNN, DLM) on the METLIN SMRT dataset
- article-reported mean and median relative error values per model

## Outputs

- model_error_comparison.csv containing one row per model with mean and median relative error, ranked by mean relative error
- validated GNN-RT benchmark numbers (mean 4.9%, median 3.2%)

## How to apply

Load the reported evaluation outputs for GNN-RT and the comparison models (random forest, Bayesian ridge, CNN, deep-learning regression) from the GNN-RT repository and the article's reported metrics, using pandas in the Anaconda for python 3.6 environment. Extract each model's mean relative error and median relative error on the METLIN SMRT dataset. Assemble a single comparison table with one row per model and columns for mean and median relative error, then rank rows by mean relative error. Validate that the GNN-RT row records the reported values — mean relative error 4.9% and median relative error 3.2% — and that GNN-RT has the lowest mean and median relative error among the five methods. Write the ranked table to model_error_comparison.csv.

## Related tools

- **Anaconda for python 3.6** (Runtime environment in which the metric-extraction pandas analysis is run)
- **rdkit** (Chemistry toolkit used by the GNN-RT pipeline whose evaluation outputs feed the metric table) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
python Preprocess.py && python Train.py
```

## Evaluation signals

- GNN-RT row in model_error_comparison.csv shows mean relative error of 4.9% and median relative error of 3.2%, matching the article abstract.
- GNN-RT is the top-ranked row (lowest mean and median relative error) among the five models.
- The table contains exactly five model rows: GNN-RT, random forest, Bayesian ridge, CNN, and DLM.
- Every model row has non-null mean and median relative error values.
- Ranking is ordered strictly by ascending mean relative error.

## Limitations

- The comparison depends on reported metrics; the repository README does not itself list per-model error values, so numbers must come from the article or saved evaluation outputs.
- The metrics are specific to the METLIN SMRT dataset; ranking may not transfer to other chromatographic systems without the transfer-learning workflow.
- The article notes LC retention time prediction 'is not accurate enough for wide adoption', so absolute error magnitudes remain a caveat even after ranking.

## Evidence

- [abstract] GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%.: "GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%."
