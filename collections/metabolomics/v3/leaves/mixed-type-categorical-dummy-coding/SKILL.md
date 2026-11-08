---
name: mixed-type-categorical-dummy-coding
description: Use when benchmarking imputation methods on mixed-type or categorical-only
  data (e.g. the Children's Hospital data with 48 continuous and 76 categorical variables)
  and a numeric-only baseline such as KNNimpute must also handle categorical entries.
when_to_use_negative:
- When the target method is missForest itself — it handles mixed-type data natively
  without dummy coding; the {-1,1} encoding is specific to the numeric-only KNNimpute
  baseline.
- When the dataset is purely continuous — the continuous pipeline (NRMSE, cross-validated
  KNNimpute/MissPALasso on standardized data) applies instead, with no dummy coding
  or PFC step.
- When the categorical data contain ill-distributed or nearly dependent variables
  for MICE, as on the Children's Hospital data, where results must be treated cautiously
  and significance statements cannot be reported.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: KNNimpute
  role: Numeric-only imputation baseline run on the {-1,1} dummy-encoded, standardized
    matrix (cross-validated Algorithm 2), then retransformed and decoded back to categories
- name: MICE
  role: Mixed-type comparison method handling categorical variables via logistic (binary)
    and polytomous logistic (>2 categories) regression instead of dummy coding
- name: missForest
  role: Reference mixed-type imputation method against which the dummy-coded baseline's
    PFC is compared via paired Wilcoxon tests
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical environment in which the missForest and MICE implementations are
    operated for the mixed-type comparison
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:23ffd71faa18fe73e01a845d4d0cac7e3a94e4393003d2c98770f0a1dce5be72
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# mixed-type-categorical-dummy-coding

## Summary

Encodes categorical variables of a mixed-type dataset into {-1,1} dichotomous dummy variables so that a purely numeric imputer (cross-validated KNNimpute, Algorithm 2) can be run as a baseline, then decodes the imputed dummy matrix back to the original categorical labels for PFC-based error evaluation.

## When to use

Use when benchmarking imputation methods on mixed-type or categorical-only data (e.g. the Children's Hospital data with 48 continuous and 76 categorical variables) and a numeric-only baseline such as KNNimpute must also handle categorical entries.

## When NOT to use

- When the target method is missForest itself — it handles mixed-type data natively without dummy coding; the {-1,1} encoding is specific to the numeric-only KNNimpute baseline.
- When the dataset is purely continuous — the continuous pipeline (NRMSE, cross-validated KNNimpute/MissPALasso on standardized data) applies instead, with no dummy coding or PFC step.
- When the categorical data contain ill-distributed or nearly dependent variables for MICE, as on the Children's Hospital data, where results must be treated cautiously and significance statements cannot be reported.

## Inputs

- Mixed-type data matrix with continuous and categorical columns (e.g. Children's Hospital data: n=55 patients, p=124 variables, 48 continuous and 76 categorical)
- Masked versions with 10%, 20% or 30% of values removed completely at random
- Category level definitions per categorical variable for dummy encoding/decoding

## Outputs

- Dummy-encoded {-1,1} matrix suitable for numeric imputation
- Decoded imputed matrix with categorical variables restored to their original labels
- Per-simulation PFC values over categorical missing entries for each method
- Paired Wilcoxon test outcomes of each competitor's error rates versus missForest's

## How to apply

For each categorical variable Xj with m levels, code it into m dichotomous {-1,1} dummy variables so the numeric imputer can operate; this is the dummy-coded KNNimpute baseline against which missForest is compared. Standardize all resulting variables to mean 0 and standard deviation 1 (and center at zero) to avoid variance-based weighting, then run the cross-validated KNNimpute (Algorithm 2) selecting k_best over a candidate range K. After imputation, retransform the data matrix to the original scales and code the dummy variables back to categorical variables. Compute the proportion of falsely classified (PFC) entries over the categorical missing positions per simulation and compare against missForest and MICE via paired Wilcoxon tests. As an alternative mixed-type baseline, run MICE with logistic regression for binary variables and polytomous logistic regression for variables with more than two categories (linear regression with normal errors or mean imputation for continuous ones).

## Related tools

- **KNNimpute** (Numeric-only imputation baseline run on the {-1,1} dummy-encoded, standardized matrix (cross-validated Algorithm 2), then retransformed and decoded back to categories)
- **MICE** (Mixed-type comparison method handling categorical variables via logistic (binary) and polytomous logistic (>2 categories) regression instead of dummy coding)
- **missForest** (Reference mixed-type imputation method against which the dummy-coded baseline's PFC is compared via paired Wilcoxon tests) — http://stat.ethz.ch/CRAN/
- **R** (Statistical environment in which the missForest and MICE implementations are operated for the mixed-type comparison) — http://stat.ethz.ch/CRAN/

## Examples

```
library(missForest); X.imp <- missForest(X.mis)
```

## Evaluation signals

- Every dummy-encoded entry is restricted to the {-1,1} coding before standardization to mean 0 and unit standard deviation.
- After decoding, all imputed categorical values map back to valid original category levels (no residual dummy codes).
- PFC values are on the paper's scale: good performance near 0, bad performance around 1.
- On the Children's Hospital data, the processed set matches n=55 patients and p=124 variables (48 continuous, 76 categorical) after removing observations with missing values.
- Paired Wilcoxon test outcomes are recorded per method comparison across simulations, except for MICE on the Children's Hospital data where significance is intentionally omitted.

## Limitations

- MICE results on the Children's Hospital data must be treated cautiously: nearly dependent variables had to be removed after each introduction of missing values and observations with missing values were dropped (n=55, p=124), so no significance statements are reported for the imputation error.
- The {-1,1} dummy encoding plus retransformation/decoding is an ad-hoc bridge for numeric-only imputers; errors should be assessed with PFC on the decoded categorical entries, not NRMSE.
- Evaluation relies on artificially introduced missing values (10%–30% completely at random); behavior under other missingness mechanisms is not established by this benchmark.

## Evidence

- [methods] dummy coding step: [evidence span withheld — anchor status: not_found]
- [results] Children's Hospital data scale: [evidence span withheld — anchor status: not_found]
