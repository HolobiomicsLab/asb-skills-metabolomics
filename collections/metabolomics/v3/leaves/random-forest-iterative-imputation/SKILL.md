---
name: random-forest-iterative-imputation
description: Use when an n x p matrix with 10–30% of values removed completely at
  random contains mixed variable types (continuous + categorical) and a non-parametric
  imputation is needed, or when nonlinear relations/complex interactions are suspected.
when_to_use_negative:
- Data where MissPALasso-style lasso imputation is preferred and dimensionality is
  low — MissPALasso performed slightly better than missForest on the gene expression
  data
- Datasets with ill-distributed and nearly dependent variables intended for MICE-style
  regression imputation (required ad-hoc removal of nearly dependent variables on
  the Children's Hospital data)
- Data with missingness mechanisms other than completely at random — evaluation covered
  only artificially introduced MCAR values of 10% to 30%
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: Core iterative random-forest imputation algorithm (Algorithm 1) applied to
    the masked matrix
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: Continuous-data baseline run with cross-validated k_best on standardized,
    retransformed data (Algorithm 2), also dummy-coded for categorical/mixed data
- name: MissPALasso
  role: Cross-validated lasso-penalty imputation baseline on standardized continuous
    data (skipped for high-dimensional Insulin data)
- name: MICE
  role: Multiple-imputation baseline using linear regression with normal errors, logistic
    regression (binary), and polytomous logistic regression (multi-category) variables
- name: R
  role: Statistical environment hosting the missForest and MICE implementations and
    all benchmark scripts
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:3126d63ea103359835622e8810c3ba430a39563fc2c44cbde19291f569003445
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# random-forest-iterative-imputation

## Summary

Imputes missing values in an n x p data matrix by iteratively training random forests on observed entries to predict missing entries, handling mixed continuous/categorical data with no structural assumptions. Includes OOB-based error estimation without a test set.

## When to use

Use when an n x p matrix with 10–30% of values removed completely at random contains mixed variable types (continuous + categorical) and a non-parametric imputation is needed, or when nonlinear relations/complex interactions are suspected.

## When NOT to use

- Data where MissPALasso-style lasso imputation is preferred and dimensionality is low — MissPALasso performed slightly better than missForest on the gene expression data
- Datasets with ill-distributed and nearly dependent variables intended for MICE-style regression imputation (required ad-hoc removal of nearly dependent variables on the Children's Hospital data)
- Data with missingness mechanisms other than completely at random — evaluation covered only artificially introduced MCAR values of 10% to 30%

## Inputs

- n x p data matrix with missing values (10%–30% removed completely at random)
- per-column variable-type annotation (continuous vs. categorical)

## Outputs

- imputed_matrix.csv (final imputed n x p matrix on original scales)
- per-variable out-of-bag (OOB) imputation error estimates averaged by variable type
- imputation_comparison.csv (mean NRMSE/PFC per method per dataset with paired Wilcoxon significance outcomes)

## How to apply

Initialize missing entries via mean imputation (continuous) or most-frequent class (categorical), then sort variables by increasing amount of missing values so the least incomplete are imputed first. For each variable, fit a random forest with y_obs as response and the remaining current-state columns as predictors, and predict y_mis for the missing rows. After each full pass compute the per-type difference measures (squared-difference ratio for continuous, proportion of changed categorical entries) and stop on the first increase with respect to both types (criterion gamma). Record the random-forest out-of-bag error estimates averaged per variable type as a no-test-set error estimate. For benchmarking, run cross-validated KNNimpute (k_best chosen over a candidate range K on data centered at zero and scaled to unit standard deviation, then retransformed to original scales), cross-validated MissPALasso on standardized data (skip on high-dimensional Insulin data), MICE with linear/logistic/polytomous regression, and dummy-coded {-1,1} KNNimpute for categorical variables; score all methods with NRMSE (continuous) and PFC (categorical) and test each competitor versus missForest with a paired Wilcoxon test.

## Related tools

- **missForest** (Core iterative random-forest imputation algorithm (Algorithm 1) applied to the masked matrix) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Continuous-data baseline run with cross-validated k_best on standardized, retransformed data (Algorithm 2), also dummy-coded for categorical/mixed data)
- **MissPALasso** (Cross-validated lasso-penalty imputation baseline on standardized continuous data (skipped for high-dimensional Insulin data))
- **MICE** (Multiple-imputation baseline using linear regression with normal errors, logistic regression (binary), and polytomous logistic regression (multi-category) variables)
- **R** (Statistical environment hosting the missForest and MICE implementations and all benchmark scripts) — http://stat.ethz.ch/CRAN/

## Examples

```
library(missForest); X.imp <- missForest(X)$ximp
```

## Evaluation signals

- NRMSE on continuous missing entries and PFC on categorical missing entries are on the paper's scale: good performance near 0, bad performance around 1
- The iteration converges via the first increase of the per-type difference measures (Delta_N / Delta_F stopping criterion gamma)
- OOB error estimates are recorded per variable and averaged per type, and track the actual NRMSE/PFC without a test set
- Paired Wilcoxon tests of each competitor's error rates versus missForest's are recorded per comparison
- Average NRMSE reduction of up to 25% versus KNNimpute on continuous data (above 50% on musk molecules data) is reproduced on the benchmark datasets

## Limitations

- No MissPALasso results for the Insulin data set (p=12,626 genes) because the high dimension makes computation not feasible
- Due to computation time only 10 simulations instead of 50 were performed for the Insulin data set
- MICE results on the Children's Hospital data required removing observations with missing values and nearly dependent variables, so no significance statements are reported and results must be treated cautiously
- OOB imputation error estimates are an informal reliability check per variable, not a formal error assessment
- Evaluation relies on artificially introduced missing values ranging from 10% to 30% only

## Evidence

- [methods] Iterative RF fit and prediction step: [evidence span withheld — anchor status: not_found]
- [methods] Variable ordering by missingness: [evidence span withheld — anchor status: not_found]
