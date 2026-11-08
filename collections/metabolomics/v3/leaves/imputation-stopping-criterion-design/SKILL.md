---
name: imputation-stopping-criterion-design
description: Use when building or tuning an iterative imputation loop (e.g. missForest)
  and you need a criterion to terminate iterations without a test set — e.g. a masked
  n x p matrix with 10%-30% MCAR values and mixed continuous/categorical columns.
when_to_use_negative:
- Single-pass imputers with no iteration loop (e.g. KNNimpute) — there is nothing
  to stop; use cross-validation to choose k instead
- A formal, test-set-validated imputation error assessment is required — the paper
  frames OOB estimates as 'a mean of informal reliability check for each variable',
  not a formal error assessment
- Missingness is not of the evaluated MCAR type — the evaluation used artificially
  introduced missing values of 10% to 30% removed completely at random
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: missForest
  role: R package implementing the iterative random-forest imputation whose stopping
    criterion is designed here; its built-in OOB error estimates supply the no-test-set
    error estimate
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical environment in which the missForest implementation and the stopping
    loop are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:bc2ba6ab3b1e997088bed5453b9bd1f4413783b7a6241369268ec8babfb67314
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# imputation-stopping-criterion-design

## Summary

Design and implement a data-driven stopping rule for iterative random-forest imputation (missForest), where the loop halts at the first iteration in which the difference between newly and previously imputed matrices increases for both variable types.

## When to use

Use when building or tuning an iterative imputation loop (e.g. missForest) and you need a criterion to terminate iterations without a test set — e.g. a masked n x p matrix with 10%-30% MCAR values and mixed continuous/categorical columns.

## When NOT to use

- Single-pass imputers with no iteration loop (e.g. KNNimpute) — there is nothing to stop; use cross-validation to choose k instead
- A formal, test-set-validated imputation error assessment is required — the paper frames OOB estimates as 'a mean of informal reliability check for each variable', not a formal error assessment
- Missingness is not of the evaluated MCAR type — the evaluation used artificially introduced missing values of 10% to 30% removed completely at random

## Inputs

- Masked n x p data matrix (10%-30% of values removed completely at random) with column type annotations (continuous vs. categorical)
- Initial imputation guess (mean for continuous, most frequent class for categorical)
- Random forest OOB error estimates from the fitted forests

## Outputs

- Final imputed matrix (e.g. imputed_matrix.csv)
- Per-iteration difference measures (squared-difference ratio for continuous; proportion of changed entries for categorical)
- OOB-based imputation error estimate averaged per variable type

## How to apply

After initial mean/mode imputation and sorting variables by amount of missing values (least incomplete first), run full random-forest passes fitting each variable's observed values as response on the remaining current matrix state as predictors. After each pass compute two difference measures between the newly and previously imputed matrices: the squared-difference ratio for continuous variables and the proportion of changed entries per number of missing values for categorical variables. Stop the loop as soon as this difference increases for the first time with respect to both variable types present in the data. Record the random-forest out-of-bag error estimates averaged over variables of the same type as a no-test-set imputation error estimate — the paper shows these OOB estimates are adequate in all settings, though they serve as an informal per-variable reliability check rather than a formal error assessment.

## Related tools

- **missForest** (R package implementing the iterative random-forest imputation whose stopping criterion is designed here; its built-in OOB error estimates supply the no-test-set error estimate) — http://stat.ethz.ch/CRAN/
- **R** (Statistical environment in which the missForest implementation and the stopping loop are executed)

## Examples

```
library(missForest); imp <- missForest(xmis); imp$ximp; imp$OOBerror
```

## Evaluation signals

- The loop terminates at the first iteration where the difference measure increases for both variable types present (squared-difference ratio for continuous, proportion of changed entries for categorical)
- Output matrix has the original n x p dimensions with no remaining missing entries
- Imputation quality metrics are near zero: NRMSE close to 0 for continuous variables and PFC close to 0 for categorical variables
- Recorded OOB error estimate per variable type is compared against true NRMSE/PFC on a complete reference matrix and found adequate (informal agreement)
- Variables were sorted by missingness (least incomplete first) before the iterative passes, as the algorithm prescribes

## Limitations

- OOB imputation error estimates are only an informal per-variable reliability check, not a formal error assessment
- The stopping rule's evaluation relied on artificially introduced missing values ranging from 10% to 30% removed completely at random; behavior outside this regime was not assessed
- Convergence is judged by the first increase of the per-type difference measures, so very noisy forests could in principle trigger early or late stopping

## Evidence

- [methods] RF fit/predict step: [evidence span withheld — anchor status: not_found]
