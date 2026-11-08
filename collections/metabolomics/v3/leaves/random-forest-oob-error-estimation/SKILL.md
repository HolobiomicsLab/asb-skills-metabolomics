---
name: random-forest-oob-error-estimation
description: Use when running missForest on data with artificially or naturally missing
  values and you need an imputation error estimate without holding out a test set,
  e.g. to report NRMSE/PFC reliability per simulation run.
when_to_use_negative:
- You need a formal, per-variable error assessment — the paper states OOB estimates
  represent 'an informal reliability check for each variable', not a formal evaluation
- Missingness does not fall within the evaluated 10-30% MCAR range — reliability of
  OOB estimates was only established for artificially introduced missing values from
  10% to 30%
- The imputation method is not random-forest-based (e.g. KNNimpute, MissPALasso, MICE)
  — those methods lack built-in OOB error estimates
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: missForest
  role: Imputation algorithm whose iterative random forest fits yield the built-in
    OOB error estimates used to approximate the true imputation error
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Environment in which the missForest package is executed
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:91ebba26f85e143e06870d2e66338003a8e1333ed8415de48318568383d596ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-forest-oob-error-estimation

## Summary

Assess the true imputation error of missForest using its built-in random forest out-of-bag (OOB) error estimates, avoiding a separate test set; the OOB estimates approximate the true error within ~10-15% proportional deviation.

## When to use

Use when running missForest on data with artificially or naturally missing values and you need an imputation error estimate without holding out a test set, e.g. to report NRMSE/PFC reliability per simulation run.

## When NOT to use

- You need a formal, per-variable error assessment — the paper states OOB estimates represent 'an informal reliability check for each variable', not a formal evaluation
- Missingness does not fall within the evaluated 10-30% MCAR range — reliability of OOB estimates was only established for artificially introduced missing values from 10% to 30%
- The imputation method is not random-forest-based (e.g. KNNimpute, MissPALasso, MICE) — those methods lack built-in OOB error estimates

## Inputs

- Data matrix (complete or with values missing completely at random at 10%, 20%, or 30%) containing continuous and/or categorical variables
- Known complete data matrix (when validating OOB estimates against the true imputation error)

## Outputs

- Per-run OOB imputation error estimates averaged per variable type (NRMSE-like for continuous, PFC-like for categorical)
- Comparison table of paired OOB vs true imputation errors, e.g. oob_error_comparison.csv

## How to apply

Run missForest per Algorithm 1 (mean-imputation initialization, variables sorted by increasing amount of missing values, iterative RF fits until the stopping criterion gamma). During each random forest fit on the observed part of a variable, record the built-in OOB error estimate; once gamma is met, average the OOB estimates over variables of the same type (continuous vs categorical). Compute the true imputation error against the known complete data — NRMSE for continuous variables and PFC (proportion of falsely classified entries) for categorical variables — then take the absolute difference between OOB and true error per run. Aggregate deviations across variable types and missingness levels (10%, 20%, 30% values removed completely at random); the mean proportional deviation should stay within roughly 10-15%, matching the paper's benchmark of 50 simulations per setting on ten datasets.

## Related tools

- **missForest** (Imputation algorithm whose iterative random forest fits yield the built-in OOB error estimates used to approximate the true imputation error) — http://stat.ethz.ch/CRAN/
- **R** (Environment in which the missForest package is executed) — http://stat.ethz.ch/CRAN/

## Examples

```
library(missForest); set.seed(123); X.mis <- prodNA(X, noNA = 0.1); X.imp <- missForest(X.mis); X.imp$OOBerror
```

## Evaluation signals

- Average absolute proportional deviation between OOB estimate and true imputation error stays within ~10-15% across all variable-type and missingness settings
- OOB estimate for categorical variables is reported as a proportion of falsely classified entries (PFC), for continuous variables as NRMSE
- OOB estimates are averaged over the set of variables of the same type (continuous vs categorical) after the stopping criterion gamma is met
- Comparison table (oob_error_comparison.csv) contains one paired (OOB, true) error row per simulation run, e.g. 50 runs per dataset per missingness level

## Limitations

- OOB imputation error estimates are an informal reliability check for each variable, not a formal error assessment
- Reliability was established only for artificially introduced missing values (10-30%, completely at random); performance under other missingness mechanisms is untested here
- Due to computation time, only 10 simulations instead of 50 were performed for the high-dimensional Insulin data set (p=12,626 genes)
- Mean-imputation initialization and the gamma stopping criterion are required for the estimates to be comparable to the paper's benchmark

## Evidence

- [results] task_004 finding: [evidence span withheld — anchor status: not_found]
- [methods] methods: [evidence span withheld — anchor status: not_found]
