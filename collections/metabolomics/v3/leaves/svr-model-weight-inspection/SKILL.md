---
name: svr-model-weight-inspection
description: Use when a trained linear SVR CCS-prediction model must be inspected
  to report how many descriptors RFE retained and which descriptor classes (ATS, Barysz,
  Kier, TIC, constitutional) dominate its feature weights.
when_to_use_negative:
- The goal is CCS prediction on new molecules rather than model introspection — use
  CCSP 2.0's prediction workflow instead.
- The model is a non-linear ML CCS predictor (e.g. DeepCCS neural network) where linear
  SVR feature weights are not directly interpretable.
- The target models have not yet been trained or the RFE descriptor subsets have not
  been generated.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3339
tools:
- name: CCSP 2.0
  role: Jupyter-notebook ML tool whose trained linear SVR models and RFE-selected
    Mordred descriptor sets are being inspected
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package supplying the 1613 molecular descriptors whose features are
    ranked by SVR weight magnitude
- name: Jupyter Notebook
  role: Execution environment in which the .ipynb notebook is opened and descriptor
    weights are extracted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:55d57e8e96190af9cfc4830b3a2dad25bd1933566c9a8a7cfe9201b3aa44f345
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# SVR model weight inspection

## Summary

Extract and rank the Mordred molecular descriptors retained by recursive feature elimination in the linear SVR models of CCSP 2.0, to understand which chemical property classes drive [M − H]−, [M + H]+, and [M + Na]+ CCS predictions.

## When to use

Use when a trained linear SVR CCS-prediction model must be inspected to report how many descriptors RFE retained and which descriptor classes (ATS, Barysz, Kier, TIC, constitutional) dominate its feature weights.

## When NOT to use

- The goal is CCS prediction on new molecules rather than model introspection — use CCSP 2.0's prediction workflow instead.
- The model is a non-linear ML CCS predictor (e.g. DeepCCS neural network) where linear SVR feature weights are not directly interpretable.
- The target models have not yet been trained or the RFE descriptor subsets have not been generated.

## Inputs

- Trained linear SVR models for [M − H]−, [M + H]+, and [M + Na]+ adducts embedded in the CCSP 2.0 .ipynb notebook
- Mordred descriptor library (1613 molecular descriptors)
- RFE-selected descriptor subsets per adduct model

## Outputs

- Per-adduct RFE-selected descriptor counts (expected averages 370, 303, 414)
- Descriptor-class weight rankings (e.g. ATS, Barysz matrix, Kier, TIC, constitutional) per adduct model
- CSV table of per-adduct descriptor counts and class rankings (e.g. descriptor_class_rankings.csv)

## How to apply

Open the CCSP 2.0 Jupyter notebook (.ipynb) from the facundof2016/CCSP2.0 repository in a Python 3 Jupyter Lab or Google Colab environment, loading the trained linear support vector regression models for the [M−H]−, [M+H]+ and [M+Na]+ adducts alongside the Mordred library of 1613 molecular descriptors. Extract the recursive-feature-elimination-selected descriptor subsets per adduct and compute the per-adduct descriptor counts, which should average 370, 303, and 414 descriptors for [M−H]−, [M+H]+, and [M+Na]+ respectively. Rank descriptor classes (e.g. ATS, Barysz matrix, Kier, TIC) by the magnitude of their SVR feature weights within each adduct model to identify shape, symmetry, and bulk-property drivers. Write the per-adduct counts and class rankings to a CSV (e.g. descriptor_class_rankings.csv) for downstream interpretation of which chemical features most influence predicted CCS values.

## Related tools

- **CCSP 2.0** (Jupyter-notebook ML tool whose trained linear SVR models and RFE-selected Mordred descriptor sets are being inspected) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package supplying the 1613 molecular descriptors whose features are ranked by SVR weight magnitude)
- **Jupyter Notebook** (Execution environment in which the .ipynb notebook is opened and descriptor weights are extracted)

## Examples

```
python -c "import joblib; m = joblib.load('svr_MH.pkl'); print(len(m.coef_[0]))"  # 370 descriptors for [M − H]− (CCSP 2.0 .ipynb context)
```

## Evaluation signals

- Per-adduct descriptor counts reproduce the reported averages of 370, 303, and 414 for [M − H]−, [M + H]+, and [M + Na]+ respectively.
- Dominant descriptor classes match the paper's reported set of ATS, Barysz matrix, Kier, TIC, and constitutional descriptors.
- Class rankings are consistent across the three adduct models in their emphasis on shape, symmetry, and bulk properties.
- Output CSV schema contains per-adduct descriptor counts and per-class weight magnitudes without missing entries.

## Limitations

- Descriptor class dominance was determined from linear SVR feature weights; non-linear relationships between descriptors and CCS are not captured.
- The CCSP 2.0 training sets predict only the most common adduct ion types ([M − H]−, [M + H]+, [M + Na]+, [M + K]+) and not dimers or less common adducts or in-source fragments.
- Descriptors encode aggregate properties; the inspection reveals which chemical features dominate prediction but not a direct mechanistic link to measured CCS values.

## Evidence
