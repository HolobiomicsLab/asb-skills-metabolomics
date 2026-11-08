---
name: ccs-prediction-accuracy-evaluation
description: Reference for validating that a CCS prediction model (e.g., a newly trained
  or user-curated SVR model) reaches the accuracy level expected of published ML predictors
  before using it to filter metabolite annotations.
when_to_use_negative:
- The goal is structural annotation filtering using a ΔCCS threshold (e.g., 2.8% with
  10 ppm mass error) rather than quantifying model prediction accuracy — that is a
  separate CCSP 2.0 workflow.
- The adducts of interest are dimers or uncommon ions such as [2M+H]+ or [2M−H]− with
  no curated experimental CCS training set, since the published models cover only
  [M−H]−, [M+H]+, [M+Na]+, and [M+K]+.
- Only a small number of molecules (< 10,000) are needed with no accuracy benchmarking
  against peer algorithms — in that case simple prediction may suffice without full
  validation.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Provides the open-source Jupyter notebook in which the linear SVR models are
    trained, tested, and benchmarked against peer ML CCS predictors.
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Generates the 1613 molecular descriptors used as features by the SVR model
    whose prediction accuracy is being evaluated.
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook (Google Colab or local Jupyter
    Lab) in which accuracy tests are run.
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:24dc5fd728b2cca69768afd7ee28b449d9fb32f60e5b57a9ebf8d5ec715693ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ccs-prediction-accuracy-evaluation

## Summary

Evaluate the accuracy of ML-based CCS predictors such as CCSP 2.0 by training/testing linear SVR models on curated experimental CCS sets (e.g., McLean CCS Compendium measurements for [M−H]−, [M+H]+, [M+Na]+ adducts) under cross-validation and held-out test-set conditions, comparing median relative errors against peer tools.

## When to use

Reference for validating that a CCS prediction model (e.g., a newly trained or user-curated SVR model) reaches the accuracy level expected of published ML predictors before using it to filter metabolite annotations.

## When NOT to use

- The goal is structural annotation filtering using a ΔCCS threshold (e.g., 2.8% with 10 ppm mass error) rather than quantifying model prediction accuracy — that is a separate CCSP 2.0 workflow.
- The adducts of interest are dimers or uncommon ions such as [2M+H]+ or [2M−H]− with no curated experimental CCS training set, since the published models cover only [M−H]−, [M+H]+, [M+Na]+, and [M+K]+.
- Only a small number of molecules (< 10,000) are needed with no accuracy benchmarking against peer algorithms — in that case simple prediction may suffice without full validation.

## Inputs

- Experimental CCS measurements for a training/test set (e.g., McLean CCS Compendium values for [M−H]−, [M+H]+, [M+Na]+ adducts)
- Molecular structures (e.g., from PubChem) for the training/test compounds
- CCSP 2.0 Jupyter notebook (.ipynb) with the 1613 Mordred descriptor generation and linear SVR modeling workflow
- Predicted CCS values from comparison algorithms (CCSBase, DeepCCS, AllCCS) on consistent test sets

## Outputs

- Per-adduct median relative error for the trained SVR model (target < 2%: 1.25–1.87% on the McLean CCS Compendium benchmarks)
- Cross-validation and test set validation accuracy results
- Comparison table of CCSP 2.0 vs CCSBase/DeepCCS/AllCCS on the same 101 randomly selected test sets

## How to apply

Compute CCS predictions for the test compounds with the linear support-vector-regression model built on the 1613 Mordred molecular descriptors, then calculate the relative error for each adduct against its experimental CCS value and report the median relative error per adduct type. Run accuracy testing under both cross-validation and test set validation conditions, keeping the randomly selected test sets consistent when benchmarking against competing algorithms (CCSBase, DeepCCS, AllCCS, MetCCS, LipidCCS). Compare the achieved median relative error to the published benchmark of lower than 2% (1.25% for the 170 [M−H]−, 1.73% for the 155 [M+H]+, and 1.87% for the 138 [M+Na]+ McLean CCS Compendium adducts). A model is judged acceptable when its median relative error is at or below these levels and its performance matches or exceeds the competing ML approaches on the same 101 randomly selected test sets.

## Related tools

- **CCSP 2.0** (Provides the open-source Jupyter notebook in which the linear SVR models are trained, tested, and benchmarked against peer ML CCS predictors.) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Generates the 1613 molecular descriptors used as features by the SVR model whose prediction accuracy is being evaluated.)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook (Google Colab or local Jupyter Lab) in which accuracy tests are run.) — https://github.com/facundof2016/CCSP2.0

## Evaluation signals

- Median relative error per adduct ([M−H]−, [M+H]+, [M+Na]+) is below 2% (matching the reported 1.25%, 1.73%, 1.87% benchmarks).
- Accuracy is assessed under both cross-validation and held-out test set validation conditions, not just one.
- When benchmarking against competing ML approaches, the same 101 randomly selected test sets are kept consistent across all algorithms.
- CCSP 2.0's performance matches or exceeds that of CCSBase, DeepCCS, and AllCCS on the benchmark sets.
- Adduct counts are consistent with the McLean CCS Compendium evaluation: 170 [M−H]−, 155 [M+H]+, and 138 [M+Na]+ adducts.

## Limitations

- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values.
- Current training sets allow prediction of the most common adduct ion types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2M+H]+ or [2M−H]− or other less common adducts or in-source fragments.
- Sklearn V1.0.2 or later is required for the current version of CCSP 2.0.
- Google Colab limits continuous notebook operation to 12 hours and disconnects after ~20 minutes idle; large training/target sets are better handled in local Jupyter Lab.
- Future work still needed on CCS predictions across multiple IM-MS instrument types (drift tube, traveling wave) and on multi-adduct/fragment-ion predictions.

## Evidence
