---
name: relative-performance-stratification-by-variable-type
description: Use when you have per-simulation NRMSE or PFC error tables for missForest
  and compared imputation methods across datasets that differ in variable-type composition
  (continuous-only, categorical-only, mixed) and in artificially introduced missingness,
  and you need to locate where the largest.
when_to_use_negative:
- The comparison is a single dataset with only one variable type and one missingness
  level — no stratification by variable-type composition is possible
- Only missForest's own out-of-bag error estimates are available; the skill requires
  per-simulation error rates from at least one compared baseline method
- Missing values are not MCAR (the article's evaluation introduced 10–30% of values
  completely at random), so relative gains may not transfer
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: Method under evaluation; its NRMSE/PFC error rates are compared against baselines
    to compute relative gains
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: Baseline for continuous-only data with cross-validated k; also dummy-coded
    as a baseline for categorical and mixed data
- name: MissPALasso
  role: Baseline for continuous-only data; excluded from the Insulin condition because
    high dimension makes computation infeasible
- name: MICE
  role: Baseline for categorical and mixed-type data (e.g. SPECT, Children's Hospital)
- name: R
  role: Implementation environment for the imputation benchmark, Wilcoxon tests, and
    summary CSV generation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:50a762fbe903d3528a22b8ca5dd8d0d18c3f0f90652e23a178552780e7b49689
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# relative-performance-stratification-by-variable-type

## Summary

A benchmark-analysis skill that computes missForest's relative imputation-error reduction over the strongest baseline (KNNimpute/MissPALasso for continuous data; MICE/dummy-coded KNNimpute for categorical and mixed data) and stratifies those gains by variable-type composition and missingness proportion (10–30% MCAR).

## When to use

Use when you have per-simulation NRMSE or PFC error tables for missForest and compared imputation methods across datasets that differ in variable-type composition (continuous-only, categorical-only, mixed) and in artificially introduced missingness, and you need to locate where the largest relative gain occurs.

## When NOT to use

- The comparison is a single dataset with only one variable type and one missingness level — no stratification by variable-type composition is possible
- Only missForest's own out-of-bag error estimates are available; the skill requires per-simulation error rates from at least one compared baseline method
- Missing values are not MCAR (the article's evaluation introduced 10–30% of values completely at random), so relative gains may not transfer

## Inputs

- Reproduced benchmark error-rate table with per-simulation NRMSE and PFC values for missForest, KNNimpute, MissPALasso, MICE, and dummy-coded KNNimpute
- Dataset metadata mapping each of the ten datasets to its variable-type composition (continuous-only, categorical-only, mixed) and sample/variable dimensions
- Missingness condition labels (10%, 20%, 30% of values removed completely at random)
- Simulation identifiers (50 simulations per condition; 10 for the Insulin dataset)

## Outputs

- missforest_gain_summary.csv with per-condition relative error reductions stratified by variable-type composition
- Ranked list of dataset–missingness conditions by largest missForest gain over the strongest baseline
- Per-condition paired Wilcoxon significance flags comparing each method's error rates to missForest's

## How to apply

Load the benchmark error-rate table with per-simulation NRMSE (continuous) and PFC (categorical/mixed) values for missForest, KNNimpute, MissPALasso, MICE and dummy-coded KNNimpute across the ten datasets (e.g. SPECT, Children's Hospital, Insulin) with 10%, 20% and 30% of values removed completely at random. For each dataset–missingness condition, identify the strongest baseline as the compared method with the lowest mean error rate, noting that the baseline set differs by setting (KNNimpute/MissPALasso for continuous-only; MICE and dummy-coded KNNimpute for categorical and mixed). Compute the relative error reduction = (baseline error − missForest error) / baseline error, aggregating over the 50 independent simulations (only 10 for the high-dimensional Insulin data, p=12,626). Stratify the reductions by variable-type composition, rank conditions to find the largest gain, and flag conditions where a paired Wilcoxon test of the compared method's error rates versus missForest's error rates is significant. Write the stratified reductions, significance flags and ranked largest-gain conditions to a summary CSV such as missforest_gain_summary.csv.

## Related tools

- **missForest** (Method under evaluation; its NRMSE/PFC error rates are compared against baselines to compute relative gains) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Baseline for continuous-only data with cross-validated k; also dummy-coded as a baseline for categorical and mixed data)
- **MissPALasso** (Baseline for continuous-only data; excluded from the Insulin condition because high dimension makes computation infeasible)
- **MICE** (Baseline for categorical and mixed-type data (e.g. SPECT, Children's Hospital))
- **R** (Implementation environment for the imputation benchmark, Wilcoxon tests, and summary CSV generation)

## Examples

```
Rscript missforest_gain_summary.R --errors benchmark_nrmse_pfc.csv --stratify variable_type --missingness 0.1,0.2,0.3 --out missforest_gain_summary.csv
```

## Evaluation signals

- The largest stratified gain should reproduce the article's headline result: up to 60% PFC reduction on the categorical-only SPECT data relative to MICE, and up to 25% NRMSE reduction relative to KNNimpute on continuous data
- Relative reductions must be computed per condition over the correct simulation count (50 per condition, but only 10 for Insulin)
- The baseline set must switch by setting: KNNimpute/MissPALasso for continuous-only, MICE/dummy-coded KNNimpute for categorical and mixed
- Every claimed gain should be backed by a paired Wilcoxon significance flag, except for the Children's Hospital MICE condition where significance is intentionally not reported
- Output CSV should contain one row per dataset–missingness condition (10 datasets × 3 missingness levels) with reduction, rank and significance columns populated

## Limitations

- MissPALasso has no results for the Insulin data set because the high dimension makes computation not feasible, so the baseline set is incomplete there
- Only 10 simulations instead of 50 were performed for the Insulin data set due to computation time
- No significance statements are reported for the MICE imputation error on the Children's Hospital data because nearly dependent variables had to be removed ad hoc; those results must be treated cautiously
- Evaluation relies on artificially introduced MCAR missing values of 10–30%; OOB error estimates are only an informal per-variable reliability check, not a formal error assessment

## Evidence
