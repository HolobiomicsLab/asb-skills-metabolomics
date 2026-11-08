---
name: missing-value-iterative-imputation
description: Use when an n x p data matrix contains missing values (e.g. 10-30% removed
  completely at random) mixing continuous and categorical variables, and non-linear
  relations or complex interactions are suspected.
when_to_use_negative:
- The analysis requires a parametric, model-based multiple imputation with distributional
  assumptions (use MICE / MissPALasso instead)
- Data are exclusively continuous and well described by linear relations — cross-validated
  KNNimpute or MissPALasso can perform slightly better (e.g. MissPALasso on gene expression
  data)
- The data are so high-dimensional that fitting per-variable random forests is computationally
  infeasible for competitors; note MissPALasso was skipped on the p=12,626 Insulin
  dataset
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: R package implementing the iterative random-forest imputation (Algorithm 1);
    the primary tool for this skill
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: cross-validated (Algorithm 2) continuous-data baseline; also used with {-1,1}
    dummy coding for categorical data
- name: MissPALasso
  role: lasso-penalized missingness-pattern alternating imputation baseline on standardized
    continuous data
- name: MICE
  role: multiple imputation baseline using linear/logistic/polytomous logistic regressions
    for categorical and mixed-type data
- name: R
  role: statistical environment in which missForest, KNNimpute, MissPALasso and MICE
    comparisons are run
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:506bb10058494e1f2246429d4a7d60cfee45f1db4bd334721a743ff19956c831
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
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# missing-value-iterative-imputation

## Summary

Iterative random-forest imputation of missing values in mixed continuous/categorical data matrices (missForest), with built-in out-of-bag error estimation and no test set required.

## When to use

Use when an n x p data matrix contains missing values (e.g. 10-30% removed completely at random) mixing continuous and categorical variables, and non-linear relations or complex interactions are suspected.

## When NOT to use

- The analysis requires a parametric, model-based multiple imputation with distributional assumptions (use MICE / MissPALasso instead)
- Data are exclusively continuous and well described by linear relations — cross-validated KNNimpute or MissPALasso can perform slightly better (e.g. MissPALasso on gene expression data)
- The data are so high-dimensional that fitting per-variable random forests is computationally infeasible for competitors; note MissPALasso was skipped on the p=12,626 Insulin dataset

## Inputs

- n x p data matrix (e.g. CSV) with continuous and/or categorical columns containing missing values
- per-column variable-type annotation (continuous vs. categorical)
- optional masked version of a complete matrix (10%-30% of values removed completely at random) for benchmarking

## Outputs

- final imputed matrix (imputed_matrix.csv)
- per-variable out-of-bag imputation error estimates
- per-iteration difference measures (Delta_N for continuous, Delta_F for categorical) confirming convergence
- NRMSE (continuous) and PFC (categorical) error values, with paired Wilcoxon test outcomes versus competing methods

## How to apply

Initialize all missing entries by mean imputation (continuous) or the most frequent class (categorical), then sort variables by amount of missing values starting with the lowest. For each variable, fit a random forest with y_obs as response and the remaining variables (current matrix state) as predictors, and predict y_mis; repeat this full pass until the stopping criterion is met. The stopping criterion is the first increase of the per-type difference measure — squared-difference ratio Delta_N for continuous variables and the proportion of changed entries Delta_F for categorical ones. Record the random-forest out-of-bag error estimates averaged over variables of the same type as an imputation error estimate without a test set. Assess quality with NRMSE close to 0 for continuous and PFC close to 0 for categorical entries, and benchmark against KNNimpute, MissPALasso and MICE using paired Wilcoxon tests.

## Related tools

- **missForest** (R package implementing the iterative random-forest imputation (Algorithm 1); the primary tool for this skill) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (cross-validated (Algorithm 2) continuous-data baseline; also used with {-1,1} dummy coding for categorical data)
- **MissPALasso** (lasso-penalized missingness-pattern alternating imputation baseline on standardized continuous data)
- **MICE** (multiple imputation baseline using linear/logistic/polytomous logistic regressions for categorical and mixed-type data)
- **R** (statistical environment in which missForest, KNNimpute, MissPALasso and MICE comparisons are run)

## Examples

```
R> library(missForest); iris.imp <- missForest(xmis = iris.mis, variable.types = rep(2, 4))
```

## Evaluation signals

- Per-type difference measures (Delta_N squared-difference ratio, Delta_F proportion of changed entries) increase for the first time, confirming the stopping criterion triggered correctly
- NRMSE over continuous missing entries is close to 0 and PFC over categorical missing entries is close to 0
- Out-of-bag error estimates averaged over variables of the same type are adequate proxies for true imputation error in all settings
- Benchmark comparison shows missForest competitive to or outperforming KNNimpute/MissPALasso/MICE, with paired Wilcoxon tests recorded per comparison (reductions up to 25% NRMSE vs KNNimpute, >50% on musk molecules)
- Output imputed_matrix.csv has the same dimensions (n x p) as the input masked matrix with no remaining missing entries

## Limitations

- Evaluation relied on artificially introduced missing values of 10%-30% removed completely at random; other missingness mechanisms are not covered
- OOB imputation error estimates are only an informal reliability check per variable, not a formal error assessment
- Computation takes about 10 times as long as cross-validated KNNimpute
- For the Insulin data set (p=12,626), MissPALasso is infeasible and only 10 instead of 50 simulations were run
- On the Children's Hospital data, MICE required ad-hoc removal of observations with missing values and nearly dependent variables, so significance statements were not reported and results must be treated cautiously
