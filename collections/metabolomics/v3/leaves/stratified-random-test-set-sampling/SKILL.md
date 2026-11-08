---
name: stratified-random-test-set-sampling
description: 'Decision support for comparing CCS prediction algorithms on a common
  footing: when benchmarking CCSP 2.0, CCSbase, AllCCS or DeepCCS against the same
  463 McLean CCS Compendium measurements, or designing stratified hold-out splits
  for IM-MS regression evaluation.'
when_to_use_negative:
- The goal is only to train a single model on a large training set and predict CCS
  values for small target sets (<10,000 molecules) — no cross-algorithm benchmarking
  split is needed
- The data set is not stratified by adduct type, or dimers and less common adducts
  ([2M+H]+, [2M−H]−) are present — current CCSP 2.0 training sets only cover the common
  adduct ion types
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Trained linear support vector regression models on the 70% training portion
    of each stratified split, then predictions on the shared 30% test sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Environment in which the CCSP 2.0 prediction notebook and the benchmark workflow
    are executed
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package providing the large library of 1613 molecular descriptors used
    as regression features in CCSP 2.0
- name: CCSBase
  role: Competing ML CCS prediction method evaluated on the identical randomly sampled
    test sets
- name: AllCCS
  role: Competing SVR CCS prediction method evaluated on the identical randomly sampled
    test sets
- name: DeepCCS
  role: Competing deep neural network CCS prediction method evaluated on the identical
    randomly sampled test sets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:813e23dce56f4e84dde7ee930628de111b5df16501b7f46098020e709fb06664
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSBase
  - CCSP 2.0
  - DeepCCS
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# stratified-random-test-set-sampling

## Summary

Draws per-iteration test sets by randomly sampling ~30% of annotated measurements within each adduct stratum ([M−H]−, [M+H]+, [M+Na]+), holding the identical sampled test sets constant across all prediction algorithms so benchmark MRE/RMSE rankings are fair. This is how CCSP 2.0 was evaluated against CCSbase, AllCCS and DeepCCS in the McLean CCS Compendium benchmark.

## When to use

Decision support for comparing CCS prediction algorithms on a common footing: when benchmarking CCSP 2.0, CCSbase, AllCCS or DeepCCS against the same 463 McLean CCS Compendium measurements, or designing stratified hold-out splits for IM-MS regression evaluation.

## When NOT to use

- The goal is only to train a single model on a large training set and predict CCS values for small target sets (<10,000 molecules) — no cross-algorithm benchmarking split is needed
- The data set is not stratified by adduct type, or dimers and less common adducts ([2M+H]+, [2M−H]−) are present — current CCSP 2.0 training sets only cover the common adduct ion types

## Inputs

- 463 CCS measurements retrieved from the McLean CCS Compendium
- Adduct labels for each measurement ([M−H]−, [M+H]+, [M+Na]+)
- CCSP 2.0 Jupyter notebook with Mordred 1613-descriptor library
- Competing algorithm predictions (CCSbase, AllCCS, DeepCCS) for the same test sets

## Outputs

- 101 randomly selected 30% test sets per adduct stratum
- Per-method, per-adduct, per-iteration MRE and RMSE values
- Aggregated median MRE and RMSE ranking table across methods and adduct types (e.g. CCSP 2.0: 1.251%, CCSbase: 1.427%, AllCCS: 1.855%, DeepCCS: 1.871% for [M−H]−)

## How to apply

Load the 463 CCS measurements from the McLean CCS Compendium and stratify them by adduct type — 170 [M−H]−, 155 [M+H]+ and 138 [M+Na]+ measurements. For each of 101 iterations, randomly sample a 30% test set within each adduct stratum; keep the sampled test sets identical across all four prediction algorithms so each sees the same held-out molecules. Train the CCSP 2.0 linear support vector regression models (built on the Mordred 1613-descriptor library) on the remaining 70% of each split, then generate predictions for the common test sets with CCSbase, AllCCS and DeepCCS as well as the trained CCSP 2.0 models. Compute median relative error (MRE) and RMSE per method, per adduct, per iteration, then aggregate median values across the 101 test sets. Rationale: stratifying by adduct avoids pooling heterogeneous ion types, and the shared 101 splits isolate algorithm performance from split-to-split variance.

## Related tools

- **CCSP 2.0** (Trained linear support vector regression models on the 70% training portion of each stratified split, then predictions on the shared 30% test sets) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Environment in which the CCSP 2.0 prediction notebook and the benchmark workflow are executed) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package providing the large library of 1613 molecular descriptors used as regression features in CCSP 2.0)
- **CCSBase** (Competing ML CCS prediction method evaluated on the identical randomly sampled test sets)
- **AllCCS** (Competing SVR CCS prediction method evaluated on the identical randomly sampled test sets)
- **DeepCCS** (Competing deep neural network CCS prediction method evaluated on the identical randomly sampled test sets)

## Examples

```
"Follow the instructions embedded within the notebook." — run the CCSP 2.0 Jupyter notebook per split: train the SVR on the 70% stratified training subset, then evaluate MRE/RMSE on the shared 30% test set across 101 iterations.
```

## Evaluation signals

- Across all 101 splits, the sampled test sets must be identical for CCSP 2.0, CCSbase, AllCCS and DeepCCS — any split-to-split difference invalidates the ranking
- CCSP 2.0 median relative errors should reproduce approximately 1.25%, 1.73% and 1.87% for [M−H]−, [M+H]+ and [M+Na]+ respectively
- For [M−H]−, median MRE ordering should be CCSP 2.0 (1.251%) < CCSbase (1.427%) < AllCCS (1.855%) < DeepCCS (1.871%)
- Median RMSE values should fall near 4.876 Å² (CCSP 2.0), 6.068 Å² (DeepCCS), 7.359 Å² (CCSbase) and 9.919 Å² (AllCCS) for [M−H]−
- Stratification must keep stratum sizes at 170 [M−H]−, 155 [M+H]+ and 138 [M+Na]+ measurements so each split's test set is ~30% within each adduct

## Limitations

- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values
- Current training sets allow the prediction of the most common adduct ion types including [M − H]−, [M + H]+, [M + Na]+, and [M + K]+ but not dimers such as [2 M + H]+ or [2 M − H]−, or other less common adducts or in-source fragments
- Google Colab will only allow continuous notebook operation for up to 12 hours and will disconnect after ~20 minutes if left idle

## Evidence
