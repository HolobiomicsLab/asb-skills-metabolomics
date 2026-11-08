---
name: monte-carlo-missingness-simulation
description: Use when you have complete reference datasets with known values and need
  to quantify imputation error (NRMSE/PFC) by injecting artificial missingness completely
  at random at 10–30% rates, or to check whether a method's internal error estimate
  tracks the true imputation error.
when_to_use_negative:
- The data contain genuinely (non-artificially) missing values with no complete ground
  truth available — NRMSE/PFC against true values cannot be computed, so the simulation
  loop is pointless.
- The goal is evaluating predictive performance on a held-out test set rather than
  imputation error — the whole point of this skill's OOB comparison is estimating
  error without a separate test set.
- The dataset is so high-dimensional or large that 50 simulations per missingness
  level are computationally infeasible (the paper reduced to 10 simulations for the
  Insulin data set, p=12,626 genes).
edam_operation: http://edamontology.org/operation_0283
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: Imputation engine run inside each simulation; its built-in random forest out-of-bag
    error estimate is the quantity being validated against the true (NRMSE/PFC) imputation
    error
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical environment in which the missForest package and the benchmark
    simulations are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:e4e9ef5b1da94db147a644d9d77306a7d874f00b7f9b79f447666ba86b712063
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_0283
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# monte-carlo-missingness-simulation

## Summary

A benchmark skill that repeatedly and artificially deletes values (10%, 20%, 30%, completely at random) from complete ground-truth datasets so imputation error can be computed exactly, using NRMSE for continuous and PFC for categorical variables, and validates missForest's built-in out-of-bag error estimate against the true error.

## When to use

Use when you have complete reference datasets with known values and need to quantify imputation error (NRMSE/PFC) by injecting artificial missingness completely at random at 10–30% rates, or to check whether a method's internal error estimate tracks the true imputation error.

## When NOT to use

- The data contain genuinely (non-artificially) missing values with no complete ground truth available — NRMSE/PFC against true values cannot be computed, so the simulation loop is pointless.
- The goal is evaluating predictive performance on a held-out test set rather than imputation error — the whole point of this skill's OOB comparison is estimating error without a separate test set.
- The dataset is so high-dimensional or large that 50 simulations per missingness level are computationally infeasible (the paper reduced to 10 simulations for the Insulin data set, p=12,626 genes).

## Inputs

- Complete (ground-truth) data matrices with mixed variable types, e.g., the ten benchmark datasets including continuous gene-expression data and categorical data like SPECT images or Lymphography domain data
- Specification of missingness levels (10%, 20%, 30%) and number of simulations per level (50)
- Variable-type annotation (continuous vs categorical) for each column

## Outputs

- Per-simulation true imputation errors: NRMSE for continuous variables, PFC for categorical variables
- Per-simulation OOB imputation error estimates averaged over variables of the same type
- oob_error_comparison.csv: paired OOB vs true imputation errors per run with absolute deviations

## How to apply

For each of the ten benchmark datasets (e.g., Isoprenoid gene network in A. thaliana, Insulin gene expression, Children's Hospital data), run 50 independent simulations per missingness level (10%, 20%, 30% of values removed completely at random). Per simulation, impute with missForest per Algorithm 1: mean-imputation initialization, sorting variables by increasing amount of missing values, then iterating random forest fits until the stopping criterion gamma is met. Record the random forest's built-in out-of-bag error estimate for each variable during each fit, and after convergence average OOB estimates over variables of the same type (continuous vs categorical). Compute true imputation error by comparing the imputed matrix against the known complete data — NRMSE for continuous, PFC (proportion of falsely classified entries) for categorical — then aggregate the absolute deviations between OOB and true errors over all variable-type and missingness settings.

## Related tools

- **missForest** (Imputation engine run inside each simulation; its built-in random forest out-of-bag error estimate is the quantity being validated against the true (NRMSE/PFC) imputation error) — http://stat.ethz.ch/CRAN/
- **R** (Statistical environment in which the missForest package and the benchmark simulations are executed)

## Examples

```
library(missForest); X.imp <- missForest(X.mis); NRMSE <- sqrt(sum((X.imp$Ximp - X.true)^2[is.na(X.mis)]) / sum((X.true)^2[is.na(X.mis)]))
```

## Evaluation signals

- Average absolute proportional deviation between the OOB imputation error estimate and the true imputation error stays within ~10–15% across all variable-type and missingness settings.
- Each benchmark dataset is run at exactly three missingness levels (10%, 20%, 30% complete-at-random) with 50 independent simulations per level.
- Continuous-variable errors are reported as NRMSE and categorical-variable errors as PFC (proportion of falsely classified entries), never as a single pooled metric.
- The OOB estimate is averaged over variables of the same type (continuous vs categorical) only after the stopping criterion gamma is met.
- The paired per-run table (OOB vs true error) is persisted as oob_error_comparison.csv with one row per simulation.

## Limitations

- Due to computation time, only 10 simulations instead of 50 were performed for the Insulin data set (p=12,626 genes) — high-dimensional data may require reducing the simulation count.
- The evaluation is restricted to artificially introduced missing values ranging from 10% to 30% (completely at random); other missingness mechanisms or rates are not covered.
- The OOB imputation error estimates are presented as an informal reliability check for each variable, not a formal error assessment.

## Evidence

- [methods] Algorithm 1 initialization: [evidence span withheld — anchor status: not_found]
