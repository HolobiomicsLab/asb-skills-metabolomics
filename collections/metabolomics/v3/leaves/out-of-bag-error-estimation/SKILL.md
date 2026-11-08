---
name: out-of-bag-error-estimation
description: Use when imputing a masked n x p matrix with missForest and you need
  an imputation error estimate (NRMSE for continuous, PFC for categorical variables)
  without a test set of true values.
when_to_use_negative:
- When true (complete) values are available and you want a formal validation metric
  — use the actual NRMSE/PFC against the masked entries instead of the informal OOB
  estimate
- When the data contain nearly dependent or ill-distributed variables that require
  ad-hoc manipulation (as with MICE on the Children's Hospital data), where error
  reporting must be treated cautiously
- When you need a baseline cross-validated k for KNNimpute — that requires the separate
  Algorithm 2 cross-validation procedure, not OOB estimation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: R package implementing the iterative random-forest imputation whose random
    forests supply the built-in out-of-bag error estimates used to approximate the
    imputation error without a test set
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical software environment in which the missForest package is run to
    obtain OOB error estimates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:eca05f3f7ff2736e90e9d18bd85158fe25c0746723a68175136d591a71ffdacb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# out-of-bag-error-estimation

## Summary

Estimates imputation error for the missForest random-forest imputation algorithm using the built-in out-of-bag (OOB) error estimates of random forest, removing the need for a held-out test set or true values.

## When to use

Use when imputing a masked n x p matrix with missForest and you need an imputation error estimate (NRMSE for continuous, PFC for categorical variables) without a test set of true values.

## When NOT to use

- When true (complete) values are available and you want a formal validation metric — use the actual NRMSE/PFC against the masked entries instead of the informal OOB estimate
- When the data contain nearly dependent or ill-distributed variables that require ad-hoc manipulation (as with MICE on the Children's Hospital data), where error reporting must be treated cautiously
- When you need a baseline cross-validated k for KNNimpute — that requires the separate Algorithm 2 cross-validation procedure, not OOB estimation

## Inputs

- Masked n x p data matrix (xmis) with 10%-30% of values removed completely at random, containing continuous and/or categorical columns
- Per-column variable type annotation (continuous vs. categorical)

## Outputs

- Final imputed matrix (imputed_matrix.csv / Ximp)
- Per-type OOB imputation error estimates: NRMSE for continuous variables, PFC (proportion of falsely classified entries) for categorical variables

## How to apply

During each missForest iteration, variables are sorted by amount of missingness (lowest first) and a random forest is fit with response y_obs and predictors x_obs to predict the missing entries. Because each forest prediction for a missing entry is made from out-of-bag samples, the package accumulates per-variable OOB error as a by-product of the iterative imputation loop, which stops when the per-type difference measure first increases (squared-difference ratio for continuous, proportion of changed entries for categorical variables). At convergence, record the random-forest OOB error estimates averaged over variables of the same type as a no-test-set imputation error estimate: NRMSE for continuous columns and PFC for categorical columns. Treat these estimates as an informal per-variable reliability check rather than a formal error assessment.

## Related tools

- **missForest** (R package implementing the iterative random-forest imputation whose random forests supply the built-in out-of-bag error estimates used to approximate the imputation error without a test set) — http://stat.ethz.ch/CRAN/
- **R** (Statistical software environment in which the missForest package is run to obtain OOB error estimates)

## Examples

```
library(missForest); res <- missForest(xmis); res$OOBerror  # NRMSE (continuous) / PFC (categorical) without a test set
```

## Evaluation signals

- OOB error estimate is reported per variable type: NRMSE for continuous columns and PFC for categorical columns, each averaged over variables of the same type
- NRMSE close to 0 (continuous) and PFC close to 0 (categorical) indicate good imputation quality on the masked entries
- The iterative loop stopped on the first increase of the per-type difference measure (squared-difference ratio for continuous, proportion of changed entries per number of missing values for categorical variables), confirming convergence
- OOB estimate should be consistent with the known artificially introduced missingness rate (10%-30% of values removed completely at random)
- Estimates prove adequate across mixed-type, high-dimensional settings (e.g. Insulin data p=12,626 genes, n=110 muscle biopsies) as reported in the paper's benchmark

## Limitations

- OOB imputation error estimates are only an informal reliability check for each variable, not a formal error assessment
- Reliability was demonstrated only with artificially introduced missing values ranging from 10% to 30% (MCAR); other missingness regimes were not evaluated
- Comparison baselines have caveats: no MissPALasso results for the high-dimensional Insulin data set, and only 10 simulations instead of 50 due to computation time

## Evidence

- [methods] the missing values are imputed by first fitting a random forest with response y_obs and predictors x_obs; then, predicting the missing values: [evidence span withheld — anchor status: not_found]
