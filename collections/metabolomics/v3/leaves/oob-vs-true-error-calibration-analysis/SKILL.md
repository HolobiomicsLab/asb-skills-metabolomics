---
name: oob-vs-true-error-calibration-analysis
description: Use when imputing missing values with missForest and you need to assess
  imputation accuracy per simulation run but have no complete reference matrix available
  at prediction time.
when_to_use_negative:
- True imputation error can be computed directly against held-out complete data —
  use NRMSE/PFC directly instead of the OOB surrogate
- The imputation method is not missForest and provides no built-in OOB error estimates
  (e.g. KNNimpute, MissPALasso, or MICE)
- Missingness is not completely at random and the 10-30% MCAR calibration range does
  not apply — the OOB adequacy was established only for artificially introduced missing
  values in this range
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: Iterative random-forest imputation whose built-in OOB error estimates are
    recorded per variable and averaged over variables of the same type to approximate
    the true imputation error
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical environment in which the missForest package is run to produce
    OOB estimates and compute NRMSE/PFC comparisons
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:ca0c3b31f0418176d798c8cdddc0aa5e3555f2fe06f7b7fc4f4e7fb71b502e2e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# oob-vs-true-error-calibration-analysis

## Summary

Validates missForest's built-in random-forest out-of-bag (OOB) error estimates as a surrogate for the true imputation error (NRMSE for continuous, PFC for categorical variables), without holding out a separate test set.

## When to use

Use when imputing missing values with missForest and you need to assess imputation accuracy per simulation run but have no complete reference matrix available at prediction time.

## When NOT to use

- True imputation error can be computed directly against held-out complete data — use NRMSE/PFC directly instead of the OOB surrogate
- The imputation method is not missForest and provides no built-in OOB error estimates (e.g. KNNimpute, MissPALasso, or MICE)
- Missingness is not completely at random and the 10-30% MCAR calibration range does not apply — the OOB adequacy was established only for artificially introduced missing values in this range

## Inputs

- Complete data matrix (mixed continuous/categorical) with artificially introduced missing values (10%, 20%, 30% completely at random)
- Per-variable random forest OOB error estimates recorded during missForest iterations
- Imputed data matrix plus the known complete matrix for computing true NRMSE/PFC

## Outputs

- Paired OOB vs true imputation errors per simulation run and variable type
- Average absolute proportional deviation between OOB estimate and true error (target: within ~10-15%)
- Summary table/plot, e.g. oob_error_comparison.csv

## How to apply

Run missForest per Algorithm 1 (mean-imputation initialization, variables sorted by increasing amount of missing values, iterating RF fits until the stopping criterion gamma is met). During each random forest fit on the observed part of a variable, record the built-in OOB error estimate, and once gamma is met average the OOB estimates over variables of the same type (continuous vs categorical). When a complete reference matrix is available (e.g. in simulation with 10%, 20%, or 30% of values removed completely at random), compute the true imputation error as NRMSE for continuous and PFC (proportion of falsely classified entries) for categorical variables, then compare paired OOB vs true errors per run. Across all settings the absolute proportional deviation between OOB and true error should stay within ~10-15% on average, which is the calibration benchmark validating OOB as a test-set-free error estimate.

## Related tools

- **missForest** (Iterative random-forest imputation whose built-in OOB error estimates are recorded per variable and averaged over variables of the same type to approximate the true imputation error) — http://stat.ethz.ch/CRAN/
- **R** (Statistical environment in which the missForest package is run to produce OOB estimates and compute NRMSE/PFC comparisons)

## Examples

```
library(missForest); res <- missForest(X.incomplete, x.true = X.complete); err <- res$OOBerror  # compare res$OOBerror against NRMSE/PFC computed from x.true
```

## Evaluation signals

- Average absolute deviation between OOB imputation error estimate and true imputation error stays within ~10-15% proportionally across all variable-type and missingness settings
- OOB estimates are averaged over the set of variables of the same type (continuous vs categorical) after the stopping criterion gamma is met
- True error metrics are correctly paired: NRMSE for continuous variables, PFC (proportion of falsely classified entries) for categorical variables
- Deviations are aggregated over 50 independent simulations per missingness level (10 only for the high-dimensional Insulin data) across the ten benchmark datasets
- Summary output (e.g. oob_error_comparison.csv) contains paired OOB and true errors per run for every variable-type and missingness setting

## Limitations

- OOB imputation error estimates are presented only as an informal reliability check for each variable, not a formal error assessment
- Calibration was established for artificially introduced missing values ranging from 10% to 30% (completely at random); behavior outside this range is untested
- Due to computation time, only 10 simulations instead of 50 were performed for the Insulin data set (p=12,626 genes), reducing the evidence base in that high-dimensional setting

## Evidence

- [methods] RF fit on observed values and prediction of missing values: [evidence span withheld — anchor status: not_found]
