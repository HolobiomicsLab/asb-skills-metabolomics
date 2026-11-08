---
name: oplsda-supervised-discriminant-analysis-metabolomics
description: Use when fitting a supervised OPLS-DA baseline on an untargeted LC-MS/MS
  feature table (features >> samples) with High/Low class labels, to obtain R²Y, cross-validated
  Q²Y, permutation-test diagnostics, and a scores plot for overfitting assessment.
when_to_use_negative:
- Input is already encoded as 2D grayscale molecular-fingerprint images destined for
  the MS2toImg CNN — the paper's image-based route bypasses the feature table entirely
  and was shown to outperform conventional baselines.
- The goal is unsupervised exploratory structure discovery (e.g., PCA), not supervised
  discrimination with predefined High/Low class labels.
- The feature table is small or low-dimensional relative to sample count, so the feature-vastly-outnumber-samples
  overfitting regime the skill diagnoses does not apply.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: ropls
  role: R package used to fit the supervised OPLS-DA discriminant model, run seven-fold
    cross-validation for Q²Y, and run permutation tests on the DPPH High/Low feature
    table
- name: RStudio
  role: R environment in which the OPLS-DA fitting, metrics export, and scores-plot
    rendering are performed
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:ae167f614fc5f4c697cf132e5af8315b81edf3eb1772f67d9b6a85bb358951ab
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RStudio
  - ropls
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# oplsda-supervised-discriminant-analysis-metabolomics

## Summary

Fits a supervised OPLS-DA discriminant model (ropls, R) on a high-dimensional untargeted metabolomics feature table with categorical bioactivity labels, and reports R²Y, cross-validated Q²Y, and permutation-test validity. In this paper it serves as the conventional baseline that demonstrates severe overfitting (R²Y ≈ 0.976 vs Q²Y ≈ 0.287) when features vastly outnumber samples.

## When to use

Use when fitting a supervised OPLS-DA baseline on an untargeted LC-MS/MS feature table (features >> samples) with High/Low class labels, to obtain R²Y, cross-validated Q²Y, permutation-test diagnostics, and a scores plot for overfitting assessment.

## When NOT to use

- Input is already encoded as 2D grayscale molecular-fingerprint images destined for the MS2toImg CNN — the paper's image-based route bypasses the feature table entirely and was shown to outperform conventional baselines.
- The goal is unsupervised exploratory structure discovery (e.g., PCA), not supervised discrimination with predefined High/Low class labels.
- The feature table is small or low-dimensional relative to sample count, so the feature-vastly-outnumber-samples overfitting regime the skill diagnoses does not apply.

## Inputs

- Merged untargeted metabolomics feature table (545 wild soybean accessions, merged at 0.8 average similarity-score threshold over MS1, RT, and MS/MS)
- Categorical class labels for DPPH activity (normalized < 0.4 = Low, > 0.6 = High)

## Outputs

- oplsd_metrics.csv with R²Y, cross-validated Q²Y, and permutation-test results
- oplsda_scores_plot.png showing High and Low groups projected on the first two components

## How to apply

Load the merged feature table — built from 545 wild soybean (Glycine soja Sieb. & Zucc.) accessions by merging features whose average similarity score across MS1, RT, and MS/MS exceeds 0.8 — together with the DPPH High/Low labels (normalized activity < 0.4 = Low, > 0.6 = High). Fit a supervised OPLS-DA model in R with the ropls package and project the High and Low groups onto the first two components for the scores plot. Record the goodness-of-fit R²Y of the fitted model, then estimate predictive ability Q²Y via seven-fold cross-validation. Assess model validity with permutation tests as specified in the methods. Write the metrics to oplsd_metrics.csv and render the scores plot to oplsda_scores_plot.png; a large R²Y–Q²Y gap (here R²Y ≈ 0.976 vs Q²Y ≈ 0.287) is the paper's stated classic indicator of severe overfitting in high-dimensional metabolomics data.

## Related tools

- **ropls** (R package used to fit the supervised OPLS-DA discriminant model, run seven-fold cross-validation for Q²Y, and run permutation tests on the DPPH High/Low feature table)
- **RStudio** (R environment in which the OPLS-DA fitting, metrics export, and scores-plot rendering are performed)

## Examples

```
library(ropls); op <- opls(X, y, predI = 1, orthoI = 1, crossvalI = 7, permI = 200)
```

## Evaluation signals

- R²Y of the fitted model should be very high (≈ 0.976), while seven-fold cross-validated Q²Y should be much lower (≈ 0.287); this large gap is the expected overfitting signature on this data.
- Permutation tests should indicate the nominal model is not valid (poor cross-validated performance despite strong class separation in training).
- The scores plot must show High vs Low DPPH groups projected onto the first two components, visually demonstrating strong training separation.
- oplsmetrics output must be written to oplsd_metrics.csv and the scores plot to oplsda_scores_plot.png.
- Class labels must follow the paper's cutoffs: normalized DPPH activity < 0.4 = Low, > 0.6 = High (intermediate samples excluded).

## Limitations

- OPLS-DA overfits when the number of features greatly exceeds the number of samples — performing well on training data while generalizing poorly to unseen samples; treat outputs as a cautionary diagnostic rather than a predictive model.
- Performance is exacerbated by feature alignment errors in the merged feature table; the paper notes its image-based approach is less sensitive to such errors.
- The paper reports these OPLS-DA results as the baseline that MS2toImg-CNN outperforms, so they are not the paper's recommended predictive solution.

## Evidence
