---
name: mixed-type-missing-value-imputation
description: Use when a data matrix mixes continuous and categorical variables (e.g.
  gene expression plus nominal annotations) and contains missing values needing imputation
  without parametric distributional assumptions.
when_to_use_negative:
- Data are purely continuous and high-dimensional gene expression where MissPALasso
  is known to perform slightly better (e.g. the Insulin gene expression dataset, p=12,626)
- The missingness pattern is not completely at random — the evaluation only covers
  10–30% of values removed completely at random
- A formal, not informal, per-variable error assessment is required — OOB estimates
  are described as an informal reliability check
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: missForest
  role: Core iterative random forest imputation method; provides built-in OOB error
    estimates for imputation quality
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: Comparison baseline for continuous (and dummy-coded categorical) data with
    cross-validated k
- name: MICE
  role: Comparison baseline based on multivariate multiple imputation, used on categorical
    and mixed-type data
- name: MissPALasso
  role: Lasso-based comparison baseline for continuous data (skipped for the high-dimensional
    Insulin dataset)
- name: R
  role: Implementation platform; the missForest package is freely available from CRAN
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:342fcb3531a1140f2fbec776e7f6a894d709eb202626aa11c47c9ed7449f307b
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mixed-type-missing-value-imputation

## Summary

Imputes missing values in mixed continuous/categorical data matrices using the non-parametric missForest iterative random forest scheme, which also yields built-in out-of-bag error estimates so imputation error can be assessed without a test set.

## When to use

Use when a data matrix mixes continuous and categorical variables (e.g. gene expression plus nominal annotations) and contains missing values needing imputation without parametric distributional assumptions.

## When NOT to use

- Data are purely continuous and high-dimensional gene expression where MissPALasso is known to perform slightly better (e.g. the Insulin gene expression dataset, p=12,626)
- The missingness pattern is not completely at random — the evaluation only covers 10–30% of values removed completely at random
- A formal, not informal, per-variable error assessment is required — OOB estimates are described as an informal reliability check

## Inputs

- Data matrix X with mixed continuous and categorical variables containing missing values (e.g. 10 benchmark datasets such as the Isoprenoid gene network in A. thaliana, Parkinson's voice measures, E. coli promoter sequences, Children's Hospital data with n=55, p=124)
- Artificially masked complete matrices with 10–30% of values removed completely at random for error evaluation

## Outputs

- Completed data matrix with imputed entries for continuous and categorical variables
- Out-of-bag imputation error estimates per variable type (NRMSE for continuous, PFC for categorical)
- True imputation error against known complete data and paired OOB-vs-true comparison summary (e.g. oob_error_comparison.csv)
- Wilcoxon-test significance statements of error rates versus KNNimpute, MissPALasso, and MICE

## How to apply

Initialize the missing entries in X by mean imputation, then sort variables Xs (s=1,…,p) by increasing amount of missing values. For each variable in that order, fit a random forest with response y_obs and predictors x_obs on the observed part, predict the missing entries, and iterate until the stopping criterion gamma (change in imputed values) is met; impute continuous variables by RF regression (error tracked as NRMSE) and categorical variables by RF classification (error tracked as PFC, the proportion of falsely classified entries). Record the random forest's built-in out-of-bag error estimate during each variable's fit and average it over variables of the same type to obtain an OOB imputation error estimate without holding out a test set. Benchmark against KNNimpute (with cross-validated k on standardized data, dummy-coding categoricals), MissPALasso, and MICE using paired Wilcoxon tests of error rates over 50 simulations per missingness level (10%, 20%, 30% values removed completely at random; only 10 simulations for the high-dimensional Insulin data set).

## Related tools

- **missForest** (Core iterative random forest imputation method; provides built-in OOB error estimates for imputation quality) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Comparison baseline for continuous (and dummy-coded categorical) data with cross-validated k)
- **MICE** (Comparison baseline based on multivariate multiple imputation, used on categorical and mixed-type data)
- **MissPALasso** (Lasso-based comparison baseline for continuous data (skipped for the high-dimensional Insulin dataset))
- **R** (Implementation platform; the missForest package is freely available from CRAN) — http://stat.ethz.ch/CRAN/

## Examples

```
library(missForest); data(iris); iris.mis <- prodNA(iris, noNA = 0.1); iris.imp <- missForest(iris.mis); iris.imp$OOBerror
```

## Evaluation signals

- Out-of-bag imputation error estimates deviate from the true imputation error by no more than ~10–15% proportionally across all variable-type and missingness settings
- True error metrics computed against known complete data stay within expected ranges: NRMSE for continuous variables and PFC for categorical variables
- Per paired Wilcoxon test, missForest error rates are competitive to or significantly lower than KNNimpute, MissPALasso, and MICE (average NRMSE reductions up to 25% vs KNNimpute)
- Iteration terminates via the stopping criterion gamma, and variables are processed in order of increasing missingness
- All 50 independent simulations per missingness level (10%, 20%, 30%; 10 for Insulin) complete and produce paired OOB/true error records

## Limitations

- OOB imputation error estimates are only an informal reliability check for each variable, not a formal error assessment
- Only 10 simulations instead of 50 were performed for the Insulin data set due to computation time; MissPALasso is infeasible there due to high dimension
- For the Children's Hospital data, MICE required ad-hoc removal of observations with missing values and nearly dependent variables, so MICE results there must be treated cautiously and no significance statements are reported
- Evaluation relied on artificially introduced missing values (10–30%) removed completely at random

## Evidence

- [methods] Sorting variables: [evidence span withheld — anchor status: not_found]
