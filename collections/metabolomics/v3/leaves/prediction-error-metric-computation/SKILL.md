---
name: prediction-error-metric-computation
description: Use when several prediction tools are being benchmarked on the same randomly
  sampled test sets and you need per-tool error rankings (MRE/RMSE) aggregated across
  iterations, e.g. evaluating CCS predictors on ion-mobility test data stratified
  by adduct type.
when_to_use_negative:
- The task is to generate CCS predictions themselves (use the CCSP 2.0 prediction
  workflow, not metric aggregation)
- Only a single prediction method is being evaluated and no cross-tool ranking is
  required
- The goal is false-positive filtering of structural annotations via ΔCCS thresholds
  (a different downstream skill)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_0092
tools:
- name: CCSP 2.0
  role: Source of the SVR-based CCS predictions whose accuracy is measured and benchmarked
  repo: https://github.com/facundof2016/CCSP2.0
- name: CCSbase
  role: Competing ML CCS prediction method included in the paired-error benchmark
- name: AllCCS
  role: Competing ML CCS prediction method included in the paired-error benchmark
- name: DeepCCS
  role: Competing deep neural network CCS prediction method included in the paired-error
    benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:2be16bfcc8b0a61709c23d08b75beb5b4376089b9ac19e64e82b9a3ed6256edd
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
  - Mordred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# prediction-error-metric-computation

## Summary

Computes median relative error (MRE) and RMSE per adduct type across multiple randomized test splits so that CCS prediction algorithms (CCSP 2.0, CCSbase, AllCCS, DeepCCS) can be ranked on identical held-out data. Grounded in the 101-subset benchmark from CCSP 2.0 (Analytical Chemistry, 2022).

## When to use

Use when several prediction tools are being benchmarked on the same randomly sampled test sets and you need per-tool error rankings (MRE/RMSE) aggregated across iterations, e.g. evaluating CCS predictors on ion-mobility test data stratified by adduct type.

## When NOT to use

- The task is to generate CCS predictions themselves (use the CCSP 2.0 prediction workflow, not metric aggregation)
- Only a single prediction method is being evaluated and no cross-tool ranking is required
- The goal is false-positive filtering of structural annotations via ΔCCS thresholds (a different downstream skill)

## Inputs

- Set of CCS values for adduct types [M − H]−, [M + H]+, [M + Na]+ (e.g. from the McLean CCS Compendium)
- CCS predictions for each method on the identical test sets
- Predefined number of iterations (101) and test fraction (30%)

## Outputs

- Per-method, per-adduct, per-iteration MRE and RMSE values
- Median MRE and median RMSE per adduct type per method (e.g. CCSP 2.0: 1.25%, 1.73%, 1.87% for [M − H]−, [M + H]+, [M + Na]+)
- Benchmark ranking table (e.g. ccs_prediction_benchmark_table.csv)

## How to apply

Load the curated CCS measurements (463 test values from the McLean CCS Compendium: 170 [M − H]−, 155 [M + H]+, 138 [M + Na]+ adducts) and stratify them by adduct. For each of 101 iterations, randomly sample a 30% test set within each adduct stratum, keeping the identical test sets across all prediction algorithms so comparisons are paired. Generate predictions for the same test sets from each method (e.g. CCSP 2.0's linear support vector regression models built on Mordred's 1613-descriptor library, versus CCSbase, AllCCS and DeepCCS). For each method, adduct, and iteration compute relative error and RMSE, then take the median across the 101 test sets; the paper reports CCSP 2.0 median MREs of 1.25% ([M − H]−), 1.73% ([M + H]+) and 1.87% ([M + Na]+). Rank methods by median MRE and median RMSE within each adduct class and export the resulting comparison table (e.g. ccs_prediction_benchmark_table.csv).

## Related tools

- **CCSP 2.0** (Source of the SVR-based CCS predictions whose accuracy is measured and benchmarked) — https://github.com/facundof2016/CCSP2.0
- **CCSbase** (Competing ML CCS prediction method included in the paired-error benchmark)
- **AllCCS** (Competing ML CCS prediction method included in the paired-error benchmark)
- **DeepCCS** (Competing deep neural network CCS prediction method included in the paired-error benchmark)

## Examples

```
python compute_ccs_benchmark.py --dataset mclean_ccs_compendium.csv --splits 101 --test_fraction 0.30 --methods CCSP2.0,CCSbase,AllCCS,DeepCCS --output ccs_prediction_benchmark_table.csv
```

## Evaluation signals

- Median relative errors for CCSP 2.0 should be approximately 1.25% ([M − H]−), 1.73% ([M + H]+) and 1.87% ([M + Na]+).
- Identical 30% test sets must be reused across all prediction algorithms for each of the 101 splits so errors are directly comparable.
- For most of the 101 data splits, CCSP 2.0 should rank equal to or better than CCSbase, AllCCS and DeepCCS by MRE.
- [M − H]− median RMSE ranking should put CCSP 2.0 at 4.876 Å2, ahead of DeepCCS (6.068 Å2), CCSbase (7.359 Å2) and AllCCS (9.919 Å2).
- Output must cover all three adduct strata with median values computed over the 101 test sets.

## Limitations

- Metric ranking is only meaningful when the exact same test sets are given to every algorithm; unequal splits invalidate the comparison.
- The accuracy evaluation relies on McLean CCS Compendium values and does not cover instrument platforms such as drift-tube or traveling-wave IM, which the authors flag as future work.
- Errors from each algorithm depend on its own training set definition, which the authors note must be fixed prior to prediction.

## Evidence

- [discussion] Performance of the new algorithm was compared against three popular ML CCS prediction methods using 101 randomly selected test sets kept consistent across all algorithms: [evidence span withheld — anchor status: not_found]
