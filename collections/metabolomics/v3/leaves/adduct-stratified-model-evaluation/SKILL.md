---
name: adduct-stratified-model-evaluation
description: Use when you need to rank a machine-learning collision cross section
  predictor (e.g. CCSP 2.0) against alternatives like CCSbase, AllCCS or DeepCCS using
  held-out test splits drawn from a measured CCS compendium stratified by adduct type.
when_to_use_negative:
- Input is a single fixed train/test split or cross-validation fold design rather
  than repeated random test sets — the skill's rationale rests on 101 paired splits.
- Your training set does not contain the adduct types needed ([M−H]⁻, [M+H]⁺, [M+Na]⁺);
  CCSP 2.0's models are tailored to those common adducts, and the method requires
  enough measured CCS values per stratum to split.
- You need to predict dimers such as [2 M + H]⁺ or [2 M − H]⁻, or other less common
  adducts or in-source fragments — current training sets do not support those cases,
  so stratified benchmarking of those classes is out of scope.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3335
tools:
- name: CCSP 2.0
  role: Trains linear support vector regression models on each 70% training split
    and predicts CCS values for the paired 30% test sets; the subject of the benchmark.
  repo: https://github.com/facundof2016/CCSP2.0
- name: CCSbase
  role: One of three competing machine-learning CCS prediction methods evaluated on
    the identical test sets.
- name: AllCCS
  role: One of three competing machine-learning CCS prediction methods evaluated on
    the identical test sets.
- name: DeepCCS
  role: One of three competing machine-learning CCS prediction methods evaluated on
    the identical test sets.
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook used to train and score models
    per split.
- name: Mordred
  role: Provides the 1613-molecular-descriptor library used as features for the CCSP
    2.0 linear SVR models.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:b907f087589726090e3135577ad5faaee152c1f2702689bf1531e663724c14d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSBase
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# adduct-stratified-model-evaluation

## Summary

Benchmark a CCS prediction algorithm against competing tools (CCSbase, AllCCS, DeepCCS) by repeating random 30% test-set splits 101 times within each adduct stratum ([M−H]⁻, [M+H]⁺, [M+Na]⁺), holding sampled test sets identical across all algorithms so median MRE and RMSE rankings are directly comparable.

## When to use

Use when you need to rank a machine-learning collision cross section predictor (e.g. CCSP 2.0) against alternatives like CCSbase, AllCCS or DeepCCS using held-out test splits drawn from a measured CCS compendium stratified by adduct type.

## When NOT to use

- Input is a single fixed train/test split or cross-validation fold design rather than repeated random test sets — the skill's rationale rests on 101 paired splits.
- Your training set does not contain the adduct types needed ([M−H]⁻, [M+H]⁺, [M+Na]⁺); CCSP 2.0's models are tailored to those common adducts, and the method requires enough measured CCS values per stratum to split.
- You need to predict dimers such as [2 M + H]⁺ or [2 M − H]⁻, or other less common adducts or in-source fragments — current training sets do not support those cases, so stratified benchmarking of those classes is out of scope.

## Inputs

- 463 CCS measurements retrieved from the McLean CCS Compendium
- stratified adduct labels ([M−H]⁻, [M+H]⁺, [M+Na]⁺)
- four prediction algorithms (CCSP 2.0, CCSbase, AllCCS, DeepCCS)
- 101 randomly selected 30% test sets kept consistent across algorithms

## Outputs

- median relative error (MRE) and RMSE per method, per adduct, per iteration
- aggregated median MRE/RMSE ranking table across methods and adduct types
- ccs_prediction_benchmark_table.csv

## How to apply

Load the 463 CCS measurements from the McLean CCS Compendium (170 [M−H]⁻, 155 [M+H]⁺, 138 [M+Na]⁺ adducts) and stratify them by adduct type. For each of 101 iterations, randomly sample a 30% test set within each adduct stratum, keeping the sampled test sets identical across all four prediction algorithms so comparisons are paired. Train the CCSP 2.0 linear support vector regression models — built on the 1613-descriptor Mordred library — on the remaining 70% of each split using the CCSP 2.0 Jupyter Notebook, and generate CCS predictions for the identical test sets with CCSbase, AllCCS, and DeepCCS. Compute median relative error (MRE) and RMSE per method, per adduct, per iteration, then aggregate median values across the 101 test sets to assemble the MRE/RMSE ranking table. Write the ranked table to a file such as ccs_prediction_benchmark_table.csv.

## Related tools

- **CCSP 2.0** (Trains linear support vector regression models on each 70% training split and predicts CCS values for the paired 30% test sets; the subject of the benchmark.) — https://github.com/facundof2016/CCSP2.0
- **CCSbase** (One of three competing machine-learning CCS prediction methods evaluated on the identical test sets.)
- **AllCCS** (One of three competing machine-learning CCS prediction methods evaluated on the identical test sets.)
- **DeepCCS** (One of three competing machine-learning CCS prediction methods evaluated on the identical test sets.)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook used to train and score models per split.)
- **Mordred** (Provides the 1613-molecular-descriptor library used as features for the CCSP 2.0 linear SVR models.)

## Examples

```
python -m ccsp2.train --mordred 1613 --svr linear --adducts '[M-H]-,[M+H]+,[M+Na]+' --test-size 0.30 --n-splits 101 --seed 42 --out ccs_prediction_benchmark_table.csv
```

## Evaluation signals

- Median relative error for CCSP 2.0 should approximate 1.25% ([M−H]⁻), 1.73% ([M+H]⁺), and 1.87% ([M+Na]⁺) — values within these ranges indicate correct application.
- For the [M−H]⁻ data set the expected median MRE ranking is CCSP 2.0 (1.251%), CCSbase (1.427%), AllCCS (1.855%), then DeepCCS (1.871%).
- By median RMSE the expected ranking is CCSP 2.0 (4.876 Å²), DeepCCS (6.068 Å²), CCSbase (7.359 Å²), then AllCCS (9.919 Å²).
- In most of the 101 data splits CCSP 2.0 should rank equal to or better than CCSbase, AllCCS, and DeepCCS; if it does not, check that test sets were held identical across algorithms.
- Output table contains one MRE and one RMSE row per method per adduct stratum, aggregated across 101 test sets.

## Limitations

- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values; benchmarking requires that training data be curated beforehand.
- Current training sets allow prediction of the most common adduct ion types ([M−H]⁻, [M+H]⁺, [M+Na]⁺, [M+K]⁺) but not dimers such as [2 M + H]⁺ or [2 M − H]⁻, or other less common adducts or in-source fragments.
- The evaluation is limited to linear SVR models in the CCSP 2.0 notebook and specific adduct types, so extrapolation to other architectures or strata is not directly validated by this workflow.

## Evidence

- [discussion] Random 30% test-set sampling repeated 101 times, kept consistent across algorithms: "compared against three popular ML CCS prediction methods reported in the literature using 101 randomly selected test sets that were kept consistent across all algorithms"
- [abstract] Median relative errors per adduct reported in the abstract: [evidence span withheld — anchor status: not_found]
- [abstract] Accuracy claim against competing tools: "CCSP produces predictions with accuracy equal to or greater than existing machine learning approaches such as CCSbase, DeepCCS and AllCCS"
- [readme] README instructions for running the CCSP 2.0 notebook in Google Colab or Jupyter Lab: [evidence span withheld — anchor status: not_found]
- [readme] README recommends Jupyter Lab route for larger runs: [evidence span withheld — anchor status: external_doc]
