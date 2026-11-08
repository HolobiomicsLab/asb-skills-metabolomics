---
name: lasso-regularized-em-imputation
description: Use when imputing missing values in standardized, continuous-only datasets
  of moderate dimension where sparse linear regressions are appropriate, or when a
  lasso-penalized baseline is needed to compare against missForest or KNNimpute.
when_to_use_negative:
- The dataset is high-dimensional such as the Insulin gene expression data (p=12,626
  genes), where the paper states computation of MissPALasso is not feasible.
- The data contain categorical or mixed-type variables; for those the paper uses MICE
  or dummy-coded {-1,1} KNNimpute instead of MissPALasso.
- You need an imputation method without tuning parameters — the paper notes missForest
  has no such parameter, whereas MissPALasso requires cross-validated selection of
  λ.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: MissPALasso
  role: Lasso-regularized alternating imputation baseline applied to standardized
    continuous data with cross-validated λ
- name: missForest
  role: Random-forest-based imputation method against which MissPALasso error rates
    are compared (paired Wilcoxon test)
  repo: http://stat.ethz.ch/CRAN/
- name: R
  role: Statistical software environment used to implement the imputation comparisons
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:48515a192aecc45082382cc2964a97d4cf11a3bfbfe8ea4b6430d6b1635f417f
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lasso-regularized-em-imputation

## Summary

Run the MissPALasso imputation algorithm, which regresses each set of missing variables on the observed ones using an l1-penalty (lasso) within an alternating imputation scheme, and benchmark it against missForest and KNNimpute on standardized, continuous-only data.

## When to use

Use when imputing missing values in standardized, continuous-only datasets of moderate dimension where sparse linear regressions are appropriate, or when a lasso-penalized baseline is needed to compare against missForest or KNNimpute.

## When NOT to use

- The dataset is high-dimensional such as the Insulin gene expression data (p=12,626 genes), where the paper states computation of MissPALasso is not feasible.
- The data contain categorical or mixed-type variables; for those the paper uses MICE or dummy-coded {-1,1} KNNimpute instead of MissPALasso.
- You need an imputation method without tuning parameters — the paper notes missForest has no such parameter, whereas MissPALasso requires cross-validated selection of λ.

## Inputs

- Continuous-only numeric data matrix with artificially introduced missing values (e.g. 10-30% missing completely at random), standardized to mean 0 and unit standard deviation
- Candidate lasso penalty values λ selected via cross-validation (same scheme as for KNNimpute's k)

## Outputs

- Imputed data matrix on the original scales (after retransformation)
- NRMSE over the continuous missing entries per simulation, comparable with missForest and KNNimpute results
- Paired Wilcoxon significance outcome of MissPALasso error rates versus missForest error rates

## How to apply

Standardize the data so that a single lasso penalty λ applies across regressions, since regularization with one λ requires all regressions to be on the same scale. Run MissPALasso (the Missingness Pattern Alternating Imputation and l1-penalty algorithm), in which the missing variables are regressed on the observed ones using the lasso penalty, following the alternating imputation scheme. Tune the lasso penalty λ by the same cross-validation scheme used for KNNimpute (cross-validated choice of the tuning parameter on the standardized data). For comparability, center variables at zero and scale them to unit standard deviation, and after imputation retransform the imputed matrix to the original scales before computing NRMSE. Evaluate with the paired Wilcoxon test of error rates versus missForest. Note that in the paper's benchmark, MissPALasso performs slightly better than missForest on the gene expression data, but it is skipped on the high-dimensional Insulin data (p=12,626 genes) because computation is not feasible.

## Related tools

- **MissPALasso** (Lasso-regularized alternating imputation baseline applied to standardized continuous data with cross-validated λ)
- **missForest** (Random-forest-based imputation method against which MissPALasso error rates are compared (paired Wilcoxon test)) — http://stat.ethz.ch/CRAN/
- **R** (Statistical software environment used to implement the imputation comparisons) — http://stat.ethz.ch/CRAN/

## Examples

```
R> library(missForest); X.std <- scale(X); X.imp <- missForest(X.std, ...)
```

## Evaluation signals

- NRMSE values for MissPALasso fall on the paper's scale (good performance near 0, bad performance around 1) after retransformation to the original scales.
- On gene expression data, MissPALasso is expected to perform slightly better than missForest; deviations warrant investigation.
- The lasso penalty λ is chosen by the same cross-validation scheme used for KNNimpute's k_best, on data standardized to mean 0 and unit standard deviation.
- MissPALasso is absent from results for the Insulin data set (p=12,626) — presence there would indicate an incorrect application relative to the paper's protocol.
- A paired Wilcoxon test of MissPALasso error rates versus missForest error rates is recorded per comparison.

## Limitations

- No MissPALasso results for the Insulin data set because the high dimension makes computation not feasible.
- Requires data standardization since regularization with a single λ requires the different regressions to be on the same scale.
- Limited to continuous data; categorical and mixed-type settings in the paper were handled with MICE and dummy-coded KNNimpute instead.
- Imputation takes substantially longer than the cross-validated KNNimpute in the paper's timing comparison.

## Evidence

- [methods] In their Missingness Pattern Alternating Imputation and l1-penalty (MissPALasso) algorithm the missing variables are regressed on the observed ones using the lasso penalty: [evidence span withheld — anchor status: not_found]
- [methods] Standardize data for MissPALasso: [evidence span withheld — anchor status: not_found]
