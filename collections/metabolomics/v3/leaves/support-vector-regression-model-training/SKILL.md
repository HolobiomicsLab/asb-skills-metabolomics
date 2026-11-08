---
name: support-vector-regression-model-training
description: Use when you need to build or rebuild a CCS prediction model for a specific
  adduct type or molecular class — e.g. when published CCSP 2.0 models ([M−H]−, [M+H]+,
  [M+Na]+) do not cover your ions, or when you have a unique experimental CCS training
  set to fit.
when_to_use_negative:
- You only need to predict CCS for the standard monomer adducts [M−H]−, [M+H]+ or
  [M+Na]+ using the already-published CCSP 2.0 models — no retraining is required
- Your data are non-linear in descriptor space and a linear-kernel SVR is inappropriate
  — the paper's benchmark performance is specific to linear SVR models
- You cannot assemble a training set of experimental CCS measurements for your adduct
  type (a training set must be defined prior to predicting CCS values, and the current
  published sets do not cover dimers such as [2M+H]+)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: 'Open-source Jupyter notebook that hosts the SVR training workflow: descriptor
    computation, RFE, GridSearchCV model tuning and prediction'
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 .ipynb notebook (Jupyter Lab or Jupyter
    Notebook; Google Colab variant available for <10,000 molecules)
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package used to compute the library of 1613 molecular descriptors that
    serve as SVR input features
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:b6fa6d58a2ceea9a2681ce9f1d4f0986ba10426f8290a81d86956298fb97b90c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# support-vector-regression-model-training

## Summary

Train linear-kernel support vector regression (SVR) models for collision cross section (CCS) prediction, as implemented in CCSP 2.0: compute 1613 Mordred molecular descriptors, apply recursive feature elimination, then fit per-adduct SVR models on user-curated training sets such as the McLean CCS Compendium values for [M−H]−, [M+H]+ and [M+Na]+ ions.

## When to use

Use when you need to build or rebuild a CCS prediction model for a specific adduct type or molecular class — e.g. when published CCSP 2.0 models ([M−H]−, [M+H]+, [M+Na]+) do not cover your ions, or when you have a unique experimental CCS training set to fit.

## When NOT to use

- You only need to predict CCS for the standard monomer adducts [M−H]−, [M+H]+ or [M+Na]+ using the already-published CCSP 2.0 models — no retraining is required
- Your data are non-linear in descriptor space and a linear-kernel SVR is inappropriate — the paper's benchmark performance is specific to linear SVR models
- You cannot assemble a training set of experimental CCS measurements for your adduct type (a training set must be defined prior to predicting CCS values, and the current published sets do not cover dimers such as [2M+H]+)

## Inputs

- Training set table of experimental CCS measurements per adduct ion (e.g. McLean CCS Compendium values for [M−H]−, [M+H]+, [M+Na]+, or user-curated dimers such as [2M+H]+)
- Molecular structures for each training compound (e.g. retrieved from PubChem or HMDB)
- CCSP 2.0 Jupyter notebook (.ipynb) downloaded from github.com/facundof2016/CCSP2.0

## Outputs

- Trained linear-kernel SVR CCS prediction model per adduct type
- Median relative error (MRE) and root mean squared error of validation (RMSEV) per adduct, e.g. 1.251% / 4.998 Å² for [M−H]−, 1.729% / 5.127 Å² for [M+H]+, 1.874% / 6.612 Å² for [M+Na]+
- Reduced molecular descriptor subset selected by recursive feature elimination from the 1613-entry Mordred library

## How to apply

Open the 'CCSP 2.0 - CCS Prediction in Jupyter' notebook (.ipynb) from github.com/facundof2016/CCSP2.0 in Jupyter Lab or Jupyter Notebook (Python 3, e.g. via Anaconda; requires Sklearn V1.0.2 or later, see requirements.txt). Assemble your training set of experimental CCS measurements — for the reference benchmark, 463 values from the McLean CCS Compendium partitioned by adduct type (170 [M−H]−, 155 [M+H]+, 138 [M+Na]+ compounds) — and retrieve molecular structures (e.g. from PubChem) for each compound. Compute the 1613-entry Mordred molecular descriptor library for each structure, then apply recursive feature elimination (RFE) to reduce the descriptors to the informative subset used for regression. Generate ~101 randomly selected train/test splits per adduct type, tuning and training a linear-kernel SVR model on each split with GridSearchCV while keeping test sets consistent across models, and predict CCS on the held-out test sets. For dimer or uncommon adducts such as [2M+H]+ or [2M−H]− (not covered by the published models), curate your own training set from the Compendium and rebuild the SVR model following the same notebook workflow — the paper explicitly designed CCSP 2.0 for user-curated training sets.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook that hosts the SVR training workflow: descriptor computation, RFE, GridSearchCV model tuning and prediction) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 .ipynb notebook (Jupyter Lab or Jupyter Notebook; Google Colab variant available for <10,000 molecules)) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package used to compute the library of 1613 molecular descriptors that serve as SVR input features)

## Examples

```
Download 'CCSP 2.0 - CCS Prediction in Jupyter.ipynb' from github.com/facundof2016/CCSP2.0 and open it in Jupyter Lab; e.g. `jupyter lab CCSP\ 2.0\ -\ CCS\ Prediction\ in\ Jupyter.ipynb`
```

## Evaluation signals

- Median relative error per adduct should be lower than 2% (paper benchmark: 1.25% for [M−H]−, 1.73% for [M+H]+, 1.87% for [M+Na]+)
- RMSEV per adduct should fall near the reported values of 4.998 Å² ([M−H]−), 5.127 Å² ([M+H]+) and 6.612 Å² ([M+Na]+)
- Descriptors are reduced from the 1613-entry Mordred library to the RFE-selected subset before fitting
- Approximately 101 randomly selected train/test splits are generated per adduct, with test sets kept consistent across models when comparing algorithms
- For a user-curated adduct type, the rebuilt SVR model's accuracy should be comparable to the paper's class-matched benchmarks (median relative errors lower than 2%)

## Limitations

- The current published training sets allow prediction of [M−H]−, [M+H]+, [M+Na]+ and [M+K]+ but not dimers such as [2M+H]+ or [2M−H]− or other less common adducts or in-source fragments; a user-curated training set is needed for those
- The need to define a specific training set prior to predicting CCS values is an acknowledged drawback of this ML approach
- Sklearn V1.0.2 or later is required; a full list of package requirements is in requirements.txt
- Google Colab limits continuous operation to up to 12 hours and disconnects after ~20 minutes idle; use the locally hosted Jupyter Lab version for large training or target sets or workflow integration

## Evidence
