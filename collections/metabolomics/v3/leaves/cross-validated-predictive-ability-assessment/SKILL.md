---
name: cross-validated-predictive-ability-assessment
description: Use when evaluating a supervised discriminant model (OPLS-DA) fitted
  to a metabolomics feature table with far more features than samples, to decide whether
  reported class separation reflects real predictive signal or overfitting.
when_to_use_negative:
- Input data are already-encoded grayscale MS2toImg molecular fingerprint images —
  the CNN pipeline bypasses feature-table supervised modeling entirely, so R²Y/Q²Y
  cross-validation of OPLS-DA is not applicable
- The feature table is low-dimensional (samples greatly outnumber features) — the
  paper's overfitting concern applies specifically when the number of features greatly
  exceeds the number of samples
- You need a performance measure for the final CNN classifier (accuracy on independent
  test sets) rather than an overfitting diagnosis for a latent-variable model
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ropls
  role: Fits the supervised OPLS-DA model on the feature table and provides the R²Y,
    seven-fold cross-validated Q²Y, and permutation-test machinery
- name: RStudio
  role: Environment in which the ropls-based OPLS-DA fitting, cross-validation, and
    metrics export are executed
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:6c6dc531e13c226b78a25bee3c33f58e858c94eea9f5e630880efbf72226e944
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RStudio
  - ropls
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-validated-predictive-ability-assessment

## Summary

Diagnoses supervised multivariate overfitting (e.g., OPLS-DA in ropls) on high-dimensional untargeted metabolomics feature tables by comparing goodness-of-fit R²Y against cross-validated predictive ability Q²Y, backed by permutation tests. In the MS2toImg benchmark, the large R²Y–Q²Y gap on the DPPH task (R²Y ≈ 0.976 vs Q²Y ≈ 0.287) is the paper's stated classic indicator of severe overfitting when features vastly outnumber samples.

## When to use

Use when evaluating a supervised discriminant model (OPLS-DA) fitted to a metabolomics feature table with far more features than samples, to decide whether reported class separation reflects real predictive signal or overfitting.

## When NOT to use

- Input data are already-encoded grayscale MS2toImg molecular fingerprint images — the CNN pipeline bypasses feature-table supervised modeling entirely, so R²Y/Q²Y cross-validation of OPLS-DA is not applicable
- The feature table is low-dimensional (samples greatly outnumber features) — the paper's overfitting concern applies specifically when the number of features greatly exceeds the number of samples
- You need a performance measure for the final CNN classifier (accuracy on independent test sets) rather than an overfitting diagnosis for a latent-variable model

## Inputs

- Merged untargeted metabolomics feature table (features merged across 545 wild soybean accessions at 0.8 average similarity threshold over MS1, RT, MS/MS)
- DPPH bioactivity class labels (normalized activity: < 0.4 = Low, > 0.6 = High)

## Outputs

- oplsd_metrics.csv containing R²Y, Q²Y, and permutation-test results
- oplsda_scores_plot.png showing High and Low groups projected onto the first two OPLS-DA components

## How to apply

Load the merged feature table (built from 545 wild soybean accessions, merging features whose average MS1, RT, and MS/MS similarity score exceeds 0.8) together with the DPPH High/Low class labels (normalized activity < 0.4 = Low, > 0.6 = High). Fit the supervised OPLS-DA model with the ropls package in R and record the goodness-of-fit R²Y of the fitted model. Estimate predictive ability Q²Y via seven-fold cross-validation as used for the baseline comparison. Assess model validity with permutation tests as specified in the methods, then compare: a high R²Y paired with a very poor cross-validated Q²Y (≈ 0.976 vs ≈ 0.287 on the DPPH task) constitutes the classic overfitting diagnosis. Write R²Y, Q²Y, and permutation-test results to oplsd_metrics.csv and render the High/Low scores projection onto the first two components to oplsda_scores_plot.png.

## Related tools

- **ropls** (Fits the supervised OPLS-DA model on the feature table and provides the R²Y, seven-fold cross-validated Q²Y, and permutation-test machinery)
- **RStudio** (Environment in which the ropls-based OPLS-DA fitting, cross-validation, and metrics export are executed)

## Examples

```
Rscript -e 'library(ropls); fit <- opls(X, y, predI = 1, orthoI = 1, crossvalI = 7, permTestI = 200); write.csv(c(R2Y = fit@modelDF$R2Y, Q2Y = fit@modelDF$Q2Y), "oplsd_metrics.csv")'
```

## Evaluation signals

- Reproduced metrics match the paper's reference values: R²Y ≈ 0.976 and cross-validated Q²Y ≈ 0.287 on the DPPH High/Low task
- The R²Y–Q²Y gap is large (≈ 0.69), matching the paper's stated classic indicator of severe overfitting in high-dimensional feature tables
- Permutation tests, as specified in the methods, yield no significant model validity (expected for an overfit model)
- oplsd_metrics.csv contains R²Y, Q²Y, and permutation-test entries, and oplsda_scores_plot.png shows High/Low groups projected onto the first two components
- Q²Y was computed with seven-fold cross-validation, the same scheme described for the baseline comparison

## Limitations

- OPLS-DA appears to achieve strong class separation and performs well on training data while generalizing poorly to unseen samples; high R²Y alone must never be reported as evidence of predictive power
- Overfitting is exacerbated by feature alignment errors in the feature table, so the diagnosis is sensitive to upstream preprocessing (e.g., the 0.8 similarity merging threshold and 1-minute RT alignment tolerance)
- The paper's quantitative R²Y/Q²Y values are only given as approximate reproduction targets; exact figures depend on feature-table version and cross-validation split details

## Evidence

- [full_text] OPLS-DA models appear to achieve strong class separation but overfit, performing well on training data while generalizing poorly to unseen samples: [evidence span withheld — anchor status: not_found]
