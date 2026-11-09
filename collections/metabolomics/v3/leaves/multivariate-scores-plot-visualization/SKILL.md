---
name: multivariate-scores-plot-visualization
description: Use when a feature table has vastly more features than samples and you
  need to assess whether a supervised classifier (OPLS-DA) overfits by comparing in-sample
  R²Y against cross-validated Q²Y with a scores plot.
when_to_use_negative:
- Input is raw LC-MS/MS data rather than an aligned feature table — use the MS2toImg
  image-encoding route instead, which bypasses feature alignment entirely
- The goal is accurate bioactivity prediction rather than diagnosing overfitting of
  classical multivariate statistics; the article shows OPLS-DA generalizes poorly
  (Q²Y ≈ 0.287) despite high fit
- Samples cannot be split into labeled classes (e.g., DPPH High/Low groups) since
  OPLS-DA is a supervised discriminant method
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ropls
  role: R package used to fit the supervised OPLS-DA model, compute R²Y and cross-validated
    Q²Y, run permutation tests, and render the two-component scores plot
- name: RStudio
  role: Environment for running the ropls-based OPLS-DA analysis and exporting the
    metrics table and scores plot
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:7e2ae2d3aec446c8974d07ab1d20b87da2567a6a7c1299037ee7ced63dc9b5fc
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RStudio
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multivariate-scores-plot-visualization

## Summary

Fit a supervised OPLS-DA discriminant model (ropls in R) on the merged wild soybean feature table with DPPH High/Low labels, render a two-component scores plot, and record R²Y, cross-validated Q²Y, and permutation-test results to diagnose overfitting in high-dimensional untargeted metabolomics data.

## When to use

Use when a feature table has vastly more features than samples and you need to assess whether a supervised classifier (OPLS-DA) overfits by comparing in-sample R²Y against cross-validated Q²Y with a scores plot.

## When NOT to use

- Input is raw LC-MS/MS data rather than an aligned feature table — use the MS2toImg image-encoding route instead, which bypasses feature alignment entirely
- The goal is accurate bioactivity prediction rather than diagnosing overfitting of classical multivariate statistics; the article shows OPLS-DA generalizes poorly (Q²Y ≈ 0.287) despite high fit
- Samples cannot be split into labeled classes (e.g., DPPH High/Low groups) since OPLS-DA is a supervised discriminant method

## Inputs

- Merged LC-MS/MS feature table from 545 wild soybean (Glycine soja) accessions (features merged at 0.8 average MS1/RT/MS/MS similarity threshold)
- DPPH High/Low class labels (normalized activity < 0.4 = Low, > 0.6 = High)

## Outputs

- oplsda_scores_plot.png (two-component OPLS-DA scores plot of High vs Low groups)
- oplsd_metrics.csv (R²Y, Q²Y, permutation-test results)

## How to apply

Load the merged feature table (built from 545 wild soybean accessions with a 0.8 average similarity-score threshold over MS1, RT, and MS/MS) and the DPPH class labels where normalized activity < 0.4 = Low and > 0.6 = High. Fit an OPLS-DA model with the ropls package in R, projecting the High and Low groups onto the first two components for the scores plot. Record goodness-of-fit R²Y and estimate predictive ability Q²Y via seven-fold cross-validation, then run permutation tests to assess validity. A very large R²Y–Q²Y gap (here R²Y ≈ 0.976 vs Q²Y ≈ 0.287) is the classic indicator of severe overfitting when features vastly outnumber samples; write metrics to oplsd_metrics.csv and render oplsda_scores_plot.png.

## Related tools

- **ropls** (R package used to fit the supervised OPLS-DA model, compute R²Y and cross-validated Q²Y, run permutation tests, and render the two-component scores plot)
- **RStudio** (Environment for running the ropls-based OPLS-DA analysis and exporting the metrics table and scores plot)

## Examples

```
library(ropls); oplsda <- opls(featureTable, y = dpphClass, predI = 2, crossvalI = 7, permTestI = 200, fig.pdfC = "oplsda_scores_plot")
```

## Evaluation signals

- Reproduced R²Y ≈ 0.976 on the DPPH High/Low task, matching the reported near-perfect fit
- Reproduced cross-validated Q²Y ≈ 0.287, with the R²Y–Q²Y gap of roughly 0.69 confirming severe overfitting
- Permutation-test results should indicate the observed separation is not better than random models trained on permuted labels
- oplsda_scores_plot.png shows the High and Low DPPH groups separated along the first two OPLS-DA components
- oplsd_metrics.csv contains R²Y, Q²Y, and permutation-test entries in a parseable schema

## Limitations

- The strong R²Y ≈ 0.976 versus poor Q²Y ≈ 0.287 demonstrates the model generalizes poorly to unseen samples and should not be trusted for prediction
- Feature alignment errors exacerbate overfitting; the image-based MS2toImg approach is explicitly less sensitive to such errors because it leverages the overall fragmentation landscape
- The repository is intended solely to support reproducibility during peer review; redistribution, modification, or other use is not permitted until formal publication

## Evidence
