---
name: benchmark-error-aggregation-across-missingness-levels
description: Use when imputation methods have been benchmarked at several completely-at-random
  missingness proportions (e.g. 10–30%) on continuous, categorical or mixed-type datasets
  and you must localize where a method's error reduction is largest and whether it
  is significant.
when_to_use_negative:
- The goal is to impute a single dataset or estimate its error without ground truth
  — use missForest's out-of-bag error estimates, which the paper positions as an informal
  per-variable reliability check, not benchmark aggregation.
- Missing values are not completely at random — the benchmark design covers only artificially
  introduced 10–30% MCAR values.
- You need significance claims for MICE on the Children's Hospital data — the paper
  explicitly does not report significance there due to ad-hoc removal of nearly dependent
  variables.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: missForest
  role: 'Method under evaluation: its per-simulation NRMSE/PFC error rates at 10–30%
    MCAR levels are aggregated and form the numerator of the relative error reduction'
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: 'Continuous-only baseline: run with cross-validated k on unit-variance standardized
    data and retransformed, its NRMSE values serve as the baseline-error denominator'
- name: MissPALasso
  role: Continuous-only baseline using lasso-penalized regressions on standardized
    data; omitted for the high-dimensional Insulin dataset
- name: MICE
  role: Categorical and mixed-type baseline implemented in R whose PFC values are
    compared with missForest's; no significance statements reported on Children's
    Hospital data
- name: R
  role: Statistical environment hosting the MICE implementation and the CRAN missForest
    package used to run the simulations and aggregate error rates
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:1fd3396e2ae650942cddd20cdebc836113484327f8bff5e36d64140f07cb8b8b
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# benchmark-error-aggregation-across-missingness-levels

## Summary

Aggregates per-simulation imputation benchmark errors (NRMSE for continuous, PFC for categorical data) over 50 simulations at 10%, 20% and 30% completely-at-random missingness, then computes missForest's relative error reduction against the strongest setting-specific baseline with paired Wilcoxon significance flags.

## When to use

Use when imputation methods have been benchmarked at several completely-at-random missingness proportions (e.g. 10–30%) on continuous, categorical or mixed-type datasets and you must localize where a method's error reduction is largest and whether it is significant.

## When NOT to use

- The goal is to impute a single dataset or estimate its error without ground truth — use missForest's out-of-bag error estimates, which the paper positions as an informal per-variable reliability check, not benchmark aggregation.
- Missing values are not completely at random — the benchmark design covers only artificially introduced 10–30% MCAR values.
- You need significance claims for MICE on the Children's Hospital data — the paper explicitly does not report significance there due to ad-hoc removal of nearly dependent variables.

## Inputs

- Per-simulation error-rate table (NRMSE for continuous, PFC for categorical) for missForest, KNNimpute, MissPALasso, MICE and dummy-coded KNNimpute over 50 runs per dataset
- Ten benchmark datasets with artificially introduced 10%, 20% and 30% MCAR missing values (e.g. SPECT images n=267/p=22, Insulin gene expression p=12,626/n=110, Children's Hospital n=55/p=124)
- Setting metadata mapping each dataset to its variable-type composition (continuous-only, categorical-only, mixed) and its applicable baseline set

## Outputs

- Stratified relative-error-reduction summary table (e.g. missforest_gain_summary.csv) per dataset–missingness condition
- Ranked list of conditions where missForest's gain over the strongest baseline is largest (e.g. up to 60% PFC reduction on SPECT versus MICE)
- Per-condition significance flags from paired Wilcoxon tests of compared-method versus missForest error rates

## How to apply

Aggregate per-simulation error rates — NRMSE for continuous variables and PFC (proportion of falsely classified) for categorical variables — over the 50 independent simulations (10 for the high-dimensional Insulin data, p=12,626, n=110) for each dataset–missingness condition at 10%, 20% and 30% values removed completely at random. For each condition, define the strongest baseline as the compared method with the lowest mean error, using the setting-specific baseline set: KNNimpute (cross-validated k, unit-variance standardization, retransformation to original scales) and MissPALasso (lasso regressions on standardized data) for continuous-only data; MICE and dummy-coded KNNimpute for categorical and mixed-type data. Compute missForest's relative error reduction as (baseline error − missForest error) / baseline error. Stratify the reductions by variable-type composition (continuous-only, categorical-only, mixed) and rank conditions to locate the largest gains, which the paper reports on the categorical-only SPECT data (up to 60% PFC reduction versus MICE) and up to 25% average NRMSE reduction versus KNNimpute on continuous data. Flag conditions where the paired Wilcoxon test of the compared method's error rates versus missForest's error rates is significant, but omit significance claims for MICE on the Children's Hospital data (n=55, p=124) where nearly dependent variables were removed after each introduction of missing values.

## Related tools

- **missForest** (Method under evaluation: its per-simulation NRMSE/PFC error rates at 10–30% MCAR levels are aggregated and form the numerator of the relative error reduction) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Continuous-only baseline: run with cross-validated k on unit-variance standardized data and retransformed, its NRMSE values serve as the baseline-error denominator)
- **MissPALasso** (Continuous-only baseline using lasso-penalized regressions on standardized data; omitted for the high-dimensional Insulin dataset)
- **MICE** (Categorical and mixed-type baseline implemented in R whose PFC values are compared with missForest's; no significance statements reported on Children's Hospital data)
- **R** (Statistical environment hosting the MICE implementation and the CRAN missForest package used to run the simulations and aggregate error rates) — http://stat.ethz.ch/CRAN/

## Examples

```
Rscript -e 'e <- read.csv("benchmark_errors.csv"); g <- (mean(e$baseline) - mean(e$missforest))/mean(e$baseline); write.csv(g, "missforest_gain_summary.csv")'
```

## Evaluation signals

- Every dataset–missingness condition (10/20/30% MCAR) has one relative reduction computed against the strongest baseline in its own setting (KNNimpute/MissPALasso for continuous-only; MICE/dummy-coded KNNimpute for categorical and mixed).
- Simulation counts match the paper: 50 per dataset, 10 for Insulin, with no MissPALasso entry for Insulin.
- Ranking reproduces the paper's headline results: largest gain on categorical-only SPECT data (up to 60% PFC reduction versus MICE) and up to 25% average NRMSE reduction versus KNNimpute on continuous data.
- Each significance flag traces to a paired Wilcoxon test of compared-method error rates versus missForest error rates; Children's Hospital MICE conditions carry no significance flag.
- Error metric type is consistent per setting: NRMSE only for continuous data (after standardization and retransformation to original scales), PFC for categorical and mixed-type data.

## Limitations

- Only artificially introduced completely-at-random missingness (10–30%) is assessed; natural or MNAR missingness mechanisms are out of scope.
- For the Insulin dataset (p=12,626, n=110) only 10 of 50 simulations were run and MissPALasso was excluded because the high dimension made computation not feasible.
- MICE results on the Children's Hospital data (n=55, p=124) must be treated cautiously: nearly dependent variables were removed after each introduction of missing values, so no significance statements are reported.
- Random forest out-of-bag imputation error estimates are an informal per-variable reliability check, not a formal substitute for true-error benchmarking.

## Evidence

- [abstract] Evaluation is performed on multiple datasets coming from a diverse selection of biological fields with artificially introduced missing values ranging from 10% to 30%.: "Evaluation is performed on multiple datasets coming from a diverse selection of biological fields with artificially introduced missing values ranging from 10% to 30%."
- [intro] We show that our approach is competitive to or outperforms the compared methods on the used data sets irrespectively of the variable type composition: "We show that our approach is competitive to or outperforms the compared methods on the used data sets irrespectively of the variable type composition"
- [methods] We perform a paired Wilcoxon test of the error rates of the compared methods versus the error rates of missForest.: "We perform a paired Wilcoxon test of the error rates of the compared methods versus the error rates of missForest."
- [results] We can see that missForest performs well, sometimes reducing the average NRMSE by up to 25% with respect to KNNimpute.: "We can see that missForest performs well, sometimes reducing the average NRMSE by up to 25% with respect to KNNimpute."
- [results] Due to computation time we only perform 10 simulations instead of 50.: "Due to computation time we only perform 10 simulations instead of 50."
- [results] there are no results for the MissPALasso in case of the Insulin data set because the high dimension makes computation not feasible.: "there are no results for the MissPALasso in case of the Insulin data set because the high dimension makes computation not feasible."
- [results] Due to this ad-hoc manipulation for making the MICE implementation work, we do not report significance statements for the imputation error.: "Due to this ad-hoc manipulation for making the MICE implementation work, we do not report significance statements for the imputation error."
