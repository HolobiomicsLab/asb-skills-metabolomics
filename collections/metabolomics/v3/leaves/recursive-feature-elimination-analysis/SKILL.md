---
name: recursive-feature-elimination-analysis
description: Use when you have trained CCSP 2.0 linear-kernel SVR models per adduct
  type ([M−H]−, [M+H]+, [M+Na]+) and want to know which of the 1613 Mordred descriptors
  RFE retained, or which descriptor classes drive CCS predictions.
when_to_use_negative:
- You are predicting CCS values for new molecules rather than interrogating model
  feature selection — use the CCSP 2.0 prediction workflow instead
- The models are not linear-kernel SVR (e.g. deep neural network or random forest
  CCS predictors) — RFE descriptor-retention analysis as described applies specifically
  to the CCSP 2.0 SVR framework
- You lack the trained model objects and only have raw descriptor matrices — RFE must
  be re-run, not 'analysed'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Hosts the trained linear SVR models and the RFE workflow being analysed; notebook
    source code
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Generates the 1613 molecular descriptors that RFE eliminates down from
- name: Jupyter Notebook
  role: Environment in which the CCSP 2.0 notebook is executed to extract RFE-selected
    subsets
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:35bb164f8254b907b858ea0f53720228f39c2a64f9ed99218094673e7efaec75
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# recursive-feature-elimination-analysis

## Summary

Extracts the descriptor subsets retained by recursive feature elimination (RFE) from the CCSP 2.0 linear-kernel SVR models — ~370, 303 and 414 descriptors for [M−H]−, [M+H]+ and [M+Na]+ respectively out of a 1613-entry Mordred library — and ranks descriptor classes (ATS, Barysz, Kier, TIC) by SVR feature-weight magnitude.

## When to use

Use when you have trained CCSP 2.0 linear-kernel SVR models per adduct type ([M−H]−, [M+H]+, [M+Na]+) and want to know which of the 1613 Mordred descriptors RFE retained, or which descriptor classes drive CCS predictions.

## When NOT to use

- You are predicting CCS values for new molecules rather than interrogating model feature selection — use the CCSP 2.0 prediction workflow instead
- The models are not linear-kernel SVR (e.g. deep neural network or random forest CCS predictors) — RFE descriptor-retention analysis as described applies specifically to the CCSP 2.0 SVR framework
- You lack the trained model objects and only have raw descriptor matrices — RFE must be re-run, not 'analysed'

## Inputs

- Trained linear-kernel SVR models from CCSP 2.0 (per adduct type)
- Mordred 1613-descriptor library
- McLean CCS Compendium reference data (170 [M−H]−, 155 [M+H]+, 138 [M+Na]+ compounds)
- CCSP 2.0 Jupyter notebook (.ipynb)

## Outputs

- Per-adduct RFE-retained descriptor counts (370/303/414 expected)
- Descriptor-class rankings by SVR feature weight magnitude
- descriptor_class_rankings.csv

## How to apply

Open the 'CCSP 2.0 - CCS Prediction in Jupyter' notebook (.ipynb) from facundof2016/CCSP2.0 in a Python 3 environment (Sklearn ≥1.0.2 required per requirements.txt). Load the trained linear SVR models for each adduct type and compute the 1613-entry Mordred descriptor library on the McLean CCS Compendium structures (170 [M−H]−, 155 [M+H]+, 138 [M+Na]+ compounds). Apply recursive feature elimination to reduce each model's descriptor set to the informative subset; verify per-adduct retained counts against the published averages of 370 ([M−H]−), 303 ([M+H]+) and 414 ([M+Na]+). Rank descriptor classes by SVR feature-weight magnitude to identify dominant contributors (ATS, Barysz matrix, Kier, TIC, constitutional descriptors encoding shape/symmetry/bulk properties). Write the per-adduct counts and class rankings to descriptor_class_rankings.csv.

## Related tools

- **CCSP 2.0** (Hosts the trained linear SVR models and the RFE workflow being analysed; notebook source code) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Generates the 1613 molecular descriptors that RFE eliminates down from)
- **Jupyter Notebook** (Environment in which the CCSP 2.0 notebook is executed to extract RFE-selected subsets)

## Examples

```
jupyter notebook 'CCSP 2.0 - CCS Prediction in Jupyter.ipynb'  # then run RFE feature-count cells to reproduce the 370/303/414 descriptor averages
```

## Evaluation signals

- Per-adduct RFE-retained descriptor counts match the published averages: 370 ([M−H]−), 303 ([M+H]+), 414 ([M+Na]+) — out of 1613 initial Mordred descriptors
- Dominant descriptor classes reproduce the reported ATS, Barysz matrix, Kier, TIC, and constitutional descriptors
- Descriptor-class rankings are derived from SVR feature-weight magnitude, not raw descriptor value variance
- Model CCS prediction accuracy remains consistent (median relative errors ~1.25/1.73/1.87%) after descriptor reduction
- Output CSV contains one ranking table per adduct type

## Limitations

- Descriptor counts are per-split averages — individual 101 train/test splits may deviate from the 370/303/414 values
- Dominant-class identification depends on linear-kernel interpretability; nonlinear kernels would obscure class contributions
- RFE settings and the 1613-descriptor library are hard-coded in the notebook — customising them requires code modification
- Requires Sklearn V1.0.2 or later per the repository README

## Evidence

- [full_text] RFE descriptor class detail from source cards: [evidence span withheld — anchor status: not_found]
