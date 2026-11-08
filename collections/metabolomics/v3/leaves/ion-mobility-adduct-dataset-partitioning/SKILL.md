---
name: ion-mobility-adduct-dataset-partitioning
description: Use when a CCS prediction model must be built or reproduced per ion species
  — e.g. splitting McLean CCS Compendium measurements into [M−H]−, [M+H]+ and [M+Na]+
  subsets for SVR regression and reporting median relative error and RMSEV.
when_to_use_negative:
- The target ion species are dimers such as [2M+H]+ or [2M−H]−, or other less common
  adducts or in-source fragments — current training sets only cover [M−H]−, [M+H]+,
  [M+Na]+ and [M+K]+
- The input data are already curated model-ready descriptor/feature tables rather
  than adduct-annotated CCS reference measurements
- A non-linear or deep-learning CCS predictor (e.g. DeepCCS, AllCCS) is specifically
  required rather than a transparent, customizable linear SVR workflow
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3391
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook providing the full adduct-partitioned SVR training
    and CCS prediction workflow
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package used to compute the 1613-entry molecular descriptor library
    for each compound structure
- name: Jupyter Notebook
  role: Execution environment (Jupyter Lab or Jupyter Notebook, Python 3) in which
    the CCSP 2.0 .ipynb is run
- name: scikit-learn
  role: Provides the linear-kernel SVR and GridSearchCV used for the per-adduct regression
    models; V1.0.2 or later required
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:290d4be26ee39de40d9b87effaf3a0490728ae0759a22ae59b81fdc5d17a0f89
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ion-mobility-adduct-dataset-partitioning

## Summary

Partitions ion-mobility CCS reference measurements into per-adduct training sets ([M−H]−, [M+H]+, [M+Na]+) and trains linear-kernel SVR models to predict collision cross sections with per-adduct accuracy metrics, as implemented in the CCSP 2.0 Jupyter notebook.

## When to use

Use when a CCS prediction model must be built or reproduced per ion species — e.g. splitting McLean CCS Compendium measurements into [M−H]−, [M+H]+ and [M+Na]+ subsets for SVR regression and reporting median relative error and RMSEV.

## When NOT to use

- The target ion species are dimers such as [2M+H]+ or [2M−H]−, or other less common adducts or in-source fragments — current training sets only cover [M−H]−, [M+H]+, [M+Na]+ and [M+K]+
- The input data are already curated model-ready descriptor/feature tables rather than adduct-annotated CCS reference measurements
- A non-linear or deep-learning CCS predictor (e.g. DeepCCS, AllCCS) is specifically required rather than a transparent, customizable linear SVR workflow

## Inputs

- McLean CCS Compendium reference spreadsheets with CCS values and adduct annotations ([M−H]−, [M+H]+, [M+Na]+)
- Compound structure records (e.g. from PubChem) for descriptor computation
- CCSP 2.0 Jupyter notebook file (.ipynb) with embedded instructions

## Outputs

- Per-adduct linear-kernel SVR models trained on 101 random train/test splits
- Median relative error (MRE) and RMSEV per adduct, expected ≈ 1.251% / 4.998 Å² ([M−H]−), 1.729% / 5.127 Å² ([M+H]+), 1.874% / 6.612 Å² ([M+Na]+)
- Metrics table written to ccs_model_performance.csv

## How to apply

Load the McLean CCS Compendium reference spreadsheets into the 'CCSP 2.0 - CCS Prediction in Jupyter' notebook (.ipynb, Python 3 via Anaconda) and partition the 463 CCS measurements by adduct type: 170 [M−H]−, 155 [M+H]+, and 138 [M+Na]+ compounds. For each compound structure, compute the 1613-entry Mordred molecular descriptor library, then apply recursive feature elimination (RFE) to reduce descriptors to the informative subset for regression. For each adduct type, generate 101 randomly selected train/test splits (test sets kept consistent across models) and tune/train a linear-kernel SVR on each split using GridSearchCV. Predict CCS values on the held-out test sets and compute the median relative error (MRE) and root-mean-square error of validation (RMSEV) per adduct; the notebook requires scikit-learn V1.0.2 or later.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook providing the full adduct-partitioned SVR training and CCS prediction workflow) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package used to compute the 1613-entry molecular descriptor library for each compound structure)
- **Jupyter Notebook** (Execution environment (Jupyter Lab or Jupyter Notebook, Python 3) in which the CCSP 2.0 .ipynb is run)
- **scikit-learn** (Provides the linear-kernel SVR and GridSearchCV used for the per-adduct regression models; V1.0.2 or later required)

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"
```

## Evaluation signals

- Median relative errors reproduce 1.25% ([M−H]−), 1.73% ([M+H]+) and 1.87% ([M+Na]+) (i.e. 1.251/1.729/1.874%) within reasonable tolerance
- RMSEV values fall near 4.998 Å² ([M−H]−), 5.127 Å² ([M+H]+) and 6.612 Å² ([M+Na]+)
- Partition counts match 170 [M−H]−, 155 [M+H]+ and 138 [M+Na]+ adducts (463 total test measurements)
- Exactly 101 randomly selected train/test splits are generated per adduct, with test sets kept consistent across all compared models
- Metrics table is written to ccs_model_performance.csv with per-adduct MRE and RMSEV rows

## Limitations

- CCSP 2.0 requires scikit-learn V1.0.2 or later (see requirements.txt in the repository)
- Current training sets allow only the most common adduct ion types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2M+H]+ or [2M−H]−, other less common adducts, or in-source fragments
- The specific training set must be defined prior to predicting CCS values; Google Colab execution is limited to ~12 hours of continuous operation and disconnects after ~20 minutes idle, so large training/target sets should use the locally hosted Jupyter Lab version

## Evidence
