---
name: chromatographic-rt-prediction-benchmark-analysis
description: Use when comparing retention time prediction models on a benchmark dataset
  and you need to determine which achieves the lowest mean/median relative error,
  e.g. GNN-RT (4.9% mean, 3.2% median) vs RF, Bayesian ridge, CNN, DLM.
when_to_use_negative:
- When the goal is training a new RT prediction model on your own spectra rather than
  benchmarking existing reported results
- When the comparison involves molecular-descriptor-based models not evaluated on
  the METLIN SMRT dataset
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: rdkit
  role: Used for molecular graph/structure handling when evaluating models on the
    METLIN SMRT dataset
  repo: https://github.com/Qiong-Yang/GNN-RT
- name: Anaconda for python 3.6
  role: Environment for running the comparison workflow
- name: pytorch
  role: Backend used by GNN-RT to produce the evaluated predictions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c04071
    title: Prediction of Liquid Chromatographic Retention Time with Graph Neural Networks
      to Assist in Small Molecule Identification
  source_content_hash: sha256:bf6253cb1f5494c6fa2f62d4c0e43e6697df291f3d9fa6c61e15a37d07f2bcd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Qiong-Yang/GNN-RT
  tools:
  - Anaconda for python 3.6
  - pytorch
  - rdkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chromatographic-rt-prediction-benchmark-analysis

## Summary

Benchmark liquid chromatographic retention time prediction methods (e.g. GNN-RT against RF, Bayesian ridge, CNN, DLM) on the METLIN SMRT dataset by extracting and ranking reported mean and median relative errors.

## When to use

Use when comparing retention time prediction models on a benchmark dataset and you need to determine which achieves the lowest mean/median relative error, e.g. GNN-RT (4.9% mean, 3.2% median) vs RF, Bayesian ridge, CNN, DLM.

## When NOT to use

- When the goal is training a new RT prediction model on your own spectra rather than benchmarking existing reported results
- When the comparison involves molecular-descriptor-based models not evaluated on the METLIN SMRT dataset

## Inputs

- Reported evaluation results for GNN-RT, RF, Bayesian ridge, CNN, and DLM on the METLIN SMRT dataset
- Article-reported metrics (mean and median relative error per model)

## Outputs

- model_error_comparison.csv: one row per model, columns for mean and median relative error, ranked by mean relative error
- Confirmation that GNN-RT has the lowest mean (4.9%) and median (3.2%) relative error

## How to apply

Load the reported evaluation outputs for each compared model (GNN-RT, random forests, Bayesian ridge regression, CNN, and the deep-learning regression model) from the repository's evaluation outputs and the article's metrics. Extract mean relative error and median relative error per model using pandas on the METLIN SMRT dataset results. Assemble a single comparison table with one row per model and columns for mean and median relative error. Rank by mean relative error to confirm GNN-RT achieves the lowest mean (4.9%) and median (3.2%) error. Write the ranked table to model_error_comparison.csv.

## Related tools

- **rdkit** (Used for molecular graph/structure handling when evaluating models on the METLIN SMRT dataset) — https://github.com/Qiong-Yang/GNN-RT
- **Anaconda for python 3.6** (Environment for running the comparison workflow)
- **pytorch** (Backend used by GNN-RT to produce the evaluated predictions)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame({'model': ['GNN-RT','RF','Bayesian ridge','CNN','DLM'], 'mean_rel_err': [4.9, ...], 'median_rel_err': [3.2, ...]}); df.sort_values('mean_rel_err').to_csv('model_error_comparison.csv', index=False)"
```

## Evaluation signals

- Verify GNN-RT row records mean relative error = 4.9% and median relative error = 3.2% as reported in the article abstract
- Confirm GNN-RT has the lowest mean and median relative error among the five compared models
- model_error_comparison.csv contains one row per model with mean and median relative error columns
- Models are ranked by mean relative error, not alphabetically or by row order
- All five models (GNN-RT, RF, Bayesian ridge, CNN, DLM) appear in the table

## Limitations

- Metrics reflect the METLIN SMRT dataset as reported in the article; performance may not generalize to other chromatographic systems
- Comparison is limited to the five models evaluated in the paper; no independent re-training of competing methods is performed
- Repository README does not include a changelog

## Evidence

- [abstract] GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%: "GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%."
- [abstract] SMRT dataset benchmark: "on a METLIN small molecule retention time (SMRT) dataset"
- [readme] Running the model training pipeline: "please put your spectra files in to **data** directory and run [Preprocess.py](Preprocess.py), [Train.py](Train.py) and [Transferlearning.py](Transferlearning.py)"
