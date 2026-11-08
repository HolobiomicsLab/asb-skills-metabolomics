---
name: mixed-type-variable-handling
description: Use when an n x p matrix with missing entries mixes continuous and categorical
  columns (e.g. gene expression plus binary/nominal traits) and a non-parametric,
  type-aware imputation is needed without test-set error estimation.
when_to_use_negative:
- The data are continuous-only and a well-tuned competitor such as MissPALasso is
  feasible — the article shows MissPALasso performs slightly better than missForest
  on gene expression data
- The matrix's categorical columns are so ill-distributed or nearly dependent that
  alternative implementations require ad-hoc removal of variables, undermining reliable
  error statements
- You need a formally validated per-variable error guarantee rather than an informal
  reliability check via out-of-bag estimates
edam_operation: http://edamontology.org/operation_3557
tools:
- name: missForest
  role: Implements the iterative random-forest imputation scheme that trains forests
    separately on continuous and categorical columns of a mixed-type matrix
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical environment in which the missForest package is installed and the
    masked/ground-truth matrices and NRMSE/PFC computations are executed
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:7d32d75eb269f0d751e0a1a87eb6d488ef6a8b1f7d51cd1194ce04c83444fbd9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3557
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mixed-type-variable-handling

## Summary

Handles missing values in a data matrix whose columns mix continuous and categorical variables by running missForest's iterative random-forest imputation, fitting a separate regression or classification forest per variable type so no dummy-coding, scaling, or distributional assumptions are required.

## When to use

Use when an n x p matrix with missing entries mixes continuous and categorical columns (e.g. gene expression plus binary/nominal traits) and a non-parametric, type-aware imputation is needed without test-set error estimation.

## When NOT to use

- The data are continuous-only and a well-tuned competitor such as MissPALasso is feasible — the article shows MissPALasso performs slightly better than missForest on gene expression data
- The matrix's categorical columns are so ill-distributed or nearly dependent that alternative implementations require ad-hoc removal of variables, undermining reliable error statements
- You need a formally validated per-variable error guarantee rather than an informal reliability check via out-of-bag estimates

## Inputs

- Masked n x p data matrix with missing values (e.g. 10%-30% removed completely at random) containing continuous, categorical, or mixed-type variables
- Per-column variable-type annotation (continuous vs. categorical)
- Optional complete ground-truth matrix paired to the masked matrix for NRMSE/PFC validation

## Outputs

- imputed_matrix.csv — the final imputed matrix preserving original variable types and scales
- Per-type out-of-bag imputation error estimates averaged over variables of the same type (NRMSE for continuous, PFC for categorical)
- Convergence trace of the per-type difference measures across imputation passes

## How to apply

Record each column's variable type (continuous vs. categorical), then seed all missing entries with an initial guess: mean imputation for continuous variables and the most frequent class for categorical variables. Sort variables by amount of missing values starting with the lowest so the least incomplete variables are imputed first. For each variable, fit a random forest with the observed values of that variable as response and the remaining variables' current state as predictors, then predict the missing entries; repeat iteratively. Track two type-specific difference measures per full pass — the squared-difference ratio for continuous variables and the proportion of changed entries per number of missing values for categorical variables — and stop as soon as the difference increases for the first time with respect to both variable types. Retain the built-in random-forest out-of-bag error estimates, averaged over variables of the same type, as the no-test-set imputation error estimate (reported as NRMSE for continuous and PFC for categorical columns).

## Related tools

- **missForest** (Implements the iterative random-forest imputation scheme that trains forests separately on continuous and categorical columns of a mixed-type matrix) — http://stat.ethz.ch/CRAN/
- **R** (Statistical environment in which the missForest package is installed and the masked/ground-truth matrices and NRMSE/PFC computations are executed)

## Examples

```
library(missForest); imp <- missForest(X.mis); imputed <- imp$ximp; imp$OOBerr
```

## Evaluation signals

- Continuous-column imputation error (NRMSE) is close to 0 and categorical-column error (PFC) is close to 0 against the ground-truth matrix
- Convergence verified by the first increase of the per-type difference measures (squared-difference ratio for continuous; proportion of changed entries for categorical)
- Each masked replicate contains exactly the specified fraction of removed entries (10%, 20% or 30%) with values removed completely at random (MCAR)
- Out-of-bag error estimates averaged per variable type are available and roughly track the true NRMSE/PFC without needing a held-out test set
- Original variable types and scales are retained in the imputed matrix (no residual dummy-coded or standardized columns)

## Limitations

- Out-of-bag imputation error estimates are an informal per-variable reliability check, not a formal error assessment
- For extremely high-dimensional data (e.g. the insulin gene expression set with p=12,626), competing methods like MissPALasso become computationally infeasible, and the original evaluation reduced replicate counts (10 instead of 50 simulations) due to computation time
- Performance depends on the missingness mechanism tested (values removed completely at random at 10%-30%); the evaluation relied entirely on artificially introduced MCAR missing values

## Evidence

- [methods] the missing values are imputed by first fitting a random forest with response y_obs and predictors x_obs; then, predicting the missing values: [evidence span withheld — anchor status: not_found]
