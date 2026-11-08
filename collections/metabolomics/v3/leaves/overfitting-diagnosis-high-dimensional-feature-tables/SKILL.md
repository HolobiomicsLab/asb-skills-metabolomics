---
name: overfitting-diagnosis-high-dimensional-feature-tables
description: Use when fitting supervised discriminant models (e.g. OPLS-DA in the
  ropls package) to high-dimensional LC-MS/MS feature tables (thousands of features,
  hundreds of samples) and you must verify whether reported class separation generalizes.
when_to_use_negative:
- The input is raw LC-MS/MS data intended for the image-based MS2toImg-CNN approach,
  which bypasses the feature-table-and-alignment step entirely
- The feature table is low-dimensional or the number of samples is comparable to the
  number of features, so the features-outnumber-samples overfitting regime does not
  apply
- Only unsupervised exploration (PCA) is needed — the diagnosis targets supervised
  models that can achieve spuriously strong class separation
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: ropls
  role: R package used to fit the supervised OPLS-DA discriminant model, compute R²Y,
    run seven-fold cross-validated Q²Y, and perform permutation tests
- name: RStudio
  role: Environment in which the ropls-based OPLS-DA fitting and metrics export are
    executed
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:204e24f9c0cfe96355711e0f8abd593009b2cb6769d2587cf46095f5c926cf9d
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RStudio
  - ropls
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# overfitting-diagnosis-high-dimensional-feature-tables

## Summary

Diagnose overfitting of supervised multivariate models (OPLS-DA) on untargeted metabolomics feature tables where features vastly outnumber samples, by comparing goodness-of-fit R²Y against cross-validated Q²Y and running permutation tests.

## When to use

Use when fitting supervised discriminant models (e.g. OPLS-DA in the ropls package) to high-dimensional LC-MS/MS feature tables (thousands of features, hundreds of samples) and you must verify whether reported class separation generalizes.

## When NOT to use

- The input is raw LC-MS/MS data intended for the image-based MS2toImg-CNN approach, which bypasses the feature-table-and-alignment step entirely
- The feature table is low-dimensional or the number of samples is comparable to the number of features, so the features-outnumber-samples overfitting regime does not apply
- Only unsupervised exploration (PCA) is needed — the diagnosis targets supervised models that can achieve spuriously strong class separation

## Inputs

- Merged LC-MS/MS feature table (features × 545 wild soybean accessions, merged at 0.8 average similarity score over MS1, RT, MS/MS)
- DPPH class labels (High/Low; normalized activity < 0.4 = Low, > 0.6 = High)

## Outputs

- oplsd_metrics.csv containing R²Y, cross-validated Q²Y, and permutation-test results
- oplsda_scores_plot.png scores plot of High vs Low groups on the first two OPLS-DA components

## How to apply

Load the merged feature table — built from 545 wild soybean (Glycine soja) accessions by merging features whose average of the three similarity scores (MS1, RT, MS/MS) exceeded 0.8 — together with the DPPH High/Low class labels (normalized activity < 0.4 = Low, > 0.6 = High). Fit a supervised OPLS-DA model in the ropls package in R, projecting the High and Low groups onto the first two components for a scores plot. Record the fitted model's R²Y, then estimate predictive ability Q²Y via seven-fold cross-validation. Assess validity with permutation tests, and treat a large R²Y–Q²Y gap as the classic overfitting indicator: in this study R²Y ≈ 0.976 was paired with a very poor Q²Y ≈ 0.287. Write R²Y, Q²Y, and permutation-test results to oplsd_metrics.csv and render the scores plot to oplsda_scores_plot.png.

## Related tools

- **ropls** (R package used to fit the supervised OPLS-DA discriminant model, compute R²Y, run seven-fold cross-validated Q²Y, and perform permutation tests)
- **RStudio** (Environment in which the ropls-based OPLS-DA fitting and metrics export are executed)

## Examples

```
library(ropls); oplsda <- opls(featureTable, y = dpphClass, predI = 1, orthoI = 1, permI = 200, crossvalI = 7, printL = TRUE, fig.pdfC = 'oplsda_scores_plot.png')
```

## Evaluation signals

- Reproduced R²Y should be ≈ 0.976 while cross-validated Q²Y should be ≈ 0.287; the large R²Y–Q²Y gap is the expected indicator of severe overfitting
- Permutation-test results should show that permuted-label models fail to achieve the fitted model's apparent separation
- oplsd_metrics.csv must contain all three result blocks (R²Y, Q²Y, permutation tests) and oplsda_scores_plot.png must show High/Low groups projected onto the first two components
- Class assignment must follow the exact thresholds: normalized DPPH activity < 0.4 = Low, > 0.6 = High
- The feature table must be the 0.8-average-similarity merged table (MS1, RT, MS/MS scores), not raw MS-DIAL output

## Limitations

- OPLS-DA can appear to achieve strong class separation while generalizing poorly to unseen samples; the fit statistics alone (R²Y) are misleading without cross-validation
- Feature alignment errors across samples exacerbate the overfitting, since precise feature matching is required to build the table
- Q²Y here is only a seven-fold cross-validated estimate; it does not replace an independent test set or full permutation-based model validation

## Evidence
