---
name: nrmse-pfc-imputation-error-assessment
description: Use when missForest has imputed a matrix with known ground truth (or
  held-out complete data) and you need to quantify true imputation error via NRMSE/PFC,
  or to judge whether the built-in OOB error estimate is an adequate test-set-free
  error proxy.
when_to_use_negative:
- The complete (uncorrupted) dataset is unavailable, so true NRMSE/PFC cannot be computed
  against ground truth — only the OOB estimate can be reported, and its adequacy cannot
  be validated here.
- Missingness is not completely at random in the 10-30% range evaluated; the paper's
  OOB-adequacy finding is grounded solely on artificially introduced MCAR values of
  10% to 30%.
- You need a formal statistical error assessment per variable rather than an informal
  one — the discussion explicitly frames the OOB estimates as a mean of informal reliability
  check for each variable.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: missForest
  role: Produces the imputed matrix and the built-in random forest OOB error estimates
    that are compared against true NRMSE/PFC; available as an R package from CRAN.
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:1c1c8326fc1bd84cabf6af6076a38187876c004d47991c9ea465e0ca03ffba28
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - missForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# nrmse-pfc-imputation-error-assessment

## Summary

Quantifies missForest imputation error with NRMSE (continuous variables) and PFC (categorical variables) against known complete data, and validates the random forest's built-in out-of-bag (OOB) error estimates as a test-set-free proxy for the true error.

## When to use

Use when missForest has imputed a matrix with known ground truth (or held-out complete data) and you need to quantify true imputation error via NRMSE/PFC, or to judge whether the built-in OOB error estimate is an adequate test-set-free error proxy.

## When NOT to use

- The complete (uncorrupted) dataset is unavailable, so true NRMSE/PFC cannot be computed against ground truth — only the OOB estimate can be reported, and its adequacy cannot be validated here.
- Missingness is not completely at random in the 10-30% range evaluated; the paper's OOB-adequacy finding is grounded solely on artificially introduced MCAR values of 10% to 30%.
- You need a formal statistical error assessment per variable rather than an informal one — the discussion explicitly frames the OOB estimates as a mean of informal reliability check for each variable.

## Inputs

- Complete reference data matrix (ground truth for NRMSE/PFC computation, e.g. one of the ten benchmark datasets such as the A. thaliana isoprenoid gene network or voice measurements in Parkinson's patients)
- Matrix with artificially introduced missing values (10%, 20%, or 30% removed completely at random)
- missForest-imputed data matrix and per-variable OOB error estimates recorded across iterations until the stopping criterion gamma is met

## Outputs

- NRMSE per continuous variable per simulation run
- PFC (proportion of falsely classified entries) per categorical variable per simulation run
- Paired true vs OOB imputation error values per run, aggregated over variable-type and missingness settings
- Summary table, e.g. oob_error_comparison.csv

## How to apply

Run missForest per Algorithm 1 (mean-imputation initialization, sorting variables by increasing amount of missing values, iterating RF fits until the stopping criterion gamma is met) on matrices where 10%, 20%, or 30% of values have been removed completely at random, using 50 independent simulations per missingness level. For each simulation, compute the true imputation error by comparing the imputed matrix against the known complete data: NRMSE for continuous variables and PFC (proportion of falsely classified entries) for categorical variables. In parallel, record the built-in OOB error estimate produced during each random forest fit and average it over the set of variables of the same type (continuous vs categorical) after gamma is met. For each run, take the absolute difference between the true error and the OOB estimate and aggregate deviations over all variable-type and missingness settings. The OOB estimate is accepted as adequate if the average absolute proportional deviation stays within the ~10-15% reported across all settings.

## Related tools

- **missForest** (Produces the imputed matrix and the built-in random forest OOB error estimates that are compared against true NRMSE/PFC; available as an R package from CRAN.) — http://stat.ethz.ch/CRAN/

## Examples

```
library(missForest); res <- missForest(X.mis); nrmse(X.true, ximp=res$ximp, xobs=!is.na(X.mis)); pfc(X.true, ximp=res$ximp, xobs=!is.na(X.mis))
```

## Evaluation signals

- Average absolute deviation between the OOB imputation error estimate and the true imputation error stays within ~10-15% proportionally across all variable-type and missingness settings.
- NRMSE values for continuous variables and PFC values for categorical variables are computed on the original scales of the known complete data, not on imputed/guessed values.
- Each simulation cell contains 50 independent runs per missingness level (10%, 20%, 30% MCAR), except the Insulin gene expression dataset (p=12,626) where only 10 simulations are run due to computation time.
- OOB estimates are averaged over variables of the same type (continuous vs categorical) only after the stopping criterion gamma is met.
- Paired true/OOB error pairs per run are tabulated and consistent across settings, as in oob_error_comparison.csv.

## Limitations

- Evaluation relied on artificially introduced missing values ranging from 10% to 30%, so OOB adequacy outside this range or under non-random missingness mechanisms is not established.
- The paper's discussion notes the OOB estimates 'represent a mean of informal reliability check for each variable', not a formal error assessment.
- For very high-dimensional data (e.g. the Insulin dataset with p=12,626 genes), computational cost forces fewer simulations, reducing the resolution of the deviation estimate.

## Evidence

- [methods] the missing values are imputed by first fitting a random forest with response y_obs and predictors x_obs; then, predicting the missing values: [evidence span withheld — anchor status: not_found]
