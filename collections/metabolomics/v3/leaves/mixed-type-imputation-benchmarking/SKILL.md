---
name: mixed-type-imputation-benchmarking
description: Use when comparing imputation methods on continuous-only, categorical-only,
  or mixed-type datasets with artificially introduced missing values (10%, 20%, 30%
  MCAR), to locate where missForest's relative error reduction over the strongest
  baseline is largest.
when_to_use_negative:
- The goal is imputation of a single production dataset rather than method comparison
  against baselines
- Missingness is not MCAR (the article evaluates only artificially introduced missing
  values ranging from 10% to 30% removed completely at random)
- The dataset's dimension makes MissPALasso infeasible and no continuous baseline
  substitute is available, as with the Insulin data set where high dimension makes
  computation not feasible
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: Method under evaluation; iterative random-forest imputation of mixed-type
    data
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: Continuous-only baseline; cross-validated k, standardized variables, and dummy-coded
    variant for categorical data
- name: MissPALasso
  role: Continuous-only baseline on standardized data; excluded for high-dimensional
    Insulin data
- name: MICE
  role: Baseline for categorical and mixed-type settings, run in R with nearly dependent
    variables removed for Children's Hospital data
- name: R
  role: Statistical environment in which the MICE implementation and benchmark analyses
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:6a549c63d2b2fe23f4264006c68ef4ebc6282ffb357473f4bd4ba0cc325c34e0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mixed-type-imputation-benchmarking

## Summary

Benchmark missForest against type-appropriate baselines (KNNimpute, MissPALasso for continuous-only; MICE and dummy-coded KNNimpute for categorical/mixed data) across datasets with 10-30% artificially introduced MCAR values, quantifying relative imputation error reduction per variable-type composition.

## When to use

Use when comparing imputation methods on continuous-only, categorical-only, or mixed-type datasets with artificially introduced missing values (10%, 20%, 30% MCAR), to locate where missForest's relative error reduction over the strongest baseline is largest.

## When NOT to use

- The goal is imputation of a single production dataset rather than method comparison against baselines
- Missingness is not MCAR (the article evaluates only artificially introduced missing values ranging from 10% to 30% removed completely at random)
- The dataset's dimension makes MissPALasso infeasible and no continuous baseline substitute is available, as with the Insulin data set where high dimension makes computation not feasible

## Inputs

- Per-simulation error-rate table with NRMSE (continuous) and PFC (categorical) values for missForest, KNNimpute, MissPALasso, MICE, and dummy-coded KNNimpute
- Ten benchmark datasets with 10%, 20% and 30% of values removed completely at random (e.g. SPECT images, Isoprenoid gene network in A. thaliana, Insulin gene expression, Children's Hospital data)
- Variable-type composition labels (continuous-only, categorical-only, mixed) per dataset

## Outputs

- missforest_gain_summary.csv with stratified relative error reductions, significance flags, and ranked largest-gain conditions
- Ranked list of dataset–missingness conditions by missForest's relative gain over the strongest baseline

## How to apply

Load the reproduced benchmark table of per-simulation NRMSE (continuous) and PFC (categorical) values for missForest, KNNimpute, MissPALasso, MICE, and dummy-coded KNNimpute across the ten datasets under 10%, 20% and 30% values removed completely at random. For each dataset–missingness condition, identify the strongest baseline as the compared method with the lowest mean error rate, noting the baseline set differs by setting (KNNimpute/MissPALasso for continuous-only; MICE/dummy-coded KNNimpute for categorical and mixed). Compute missForest's relative error reduction as (baseline error − missForest error) / baseline error, aggregating over 50 independent simulations (only 10 for the high-dimensional Insulin dataset with p=12,626 genes). Stratify reductions by variable-type composition (continuous-only, categorical-only, mixed) and flag conditions where a paired Wilcoxon test of the compared method's error rates versus missForest's indicates a significant difference, writing stratified reductions and significance flags to a summary CSV.

## Related tools

- **missForest** (Method under evaluation; iterative random-forest imputation of mixed-type data) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Continuous-only baseline; cross-validated k, standardized variables, and dummy-coded variant for categorical data)
- **MissPALasso** (Continuous-only baseline on standardized data; excluded for high-dimensional Insulin data)
- **MICE** (Baseline for categorical and mixed-type settings, run in R with nearly dependent variables removed for Children's Hospital data)
- **R** (Statistical environment in which the MICE implementation and benchmark analyses are executed)

## Examples

```
Rscript missforest_gain_summary.R --errors benchmark_errors.rds --out missforest_gain_summary.csv
```

## Evaluation signals

- Every dataset–missingness condition has a baseline set that matches its variable-type composition (KNNimpute/MissPALasso for continuous-only; MICE/dummy-coded KNNimpute for categorical and mixed)
- Relative error reductions aggregate exactly 50 simulations per condition, except 10 for the Insulin dataset
- Each stratified reduction carries a paired Wilcoxon significance flag; no significance flag is emitted for MICE on Children's Hospital data
- Largest-gain ranking is reproducible (e.g. categorical-only SPECT data shows up to 60% PFC reduction versus MICE, continuous settings show NRMSE reductions up to 25% versus KNNimpute)

## Limitations

- No MissPALasso results for the Insulin data set because the high dimension makes computation not feasible
- Only 10 simulations instead of 50 for the Insulin data set due to computation time
- MICE results on Children's Hospital data require caution due to ill-distributed and nearly dependent variables; no significance statements reported for MICE imputation error there
- Evaluation relies exclusively on artificially introduced MCAR values ranging from 10% to 30%; OOB error estimates serve only as an informal reliability check per variable

## Evidence

- [results] relative gain on categorical-only SPECT data: [evidence span withheld — anchor status: not_found]
