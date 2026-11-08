---
name: permutation-test-model-validation
description: Use when a supervised classifier (OPLS-DA, RF) fitted on a feature table
  where features vastly outnumber samples reports high training fit (R²Y) and you
  must establish whether it generalizes, via k-fold CV and permutation testing.
when_to_use_negative:
- Input is already image-encoded (MS2toImg grayscale molecular fingerprints) rather
  than an aligned feature table — the CNN pipeline bypasses feature matching and this
  validation diagnosis targets feature-table models
- The sample count exceeds or is comparable to the feature count, so p >> n overfitting
  is not the primary risk
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: ropls
  role: R package used to fit the supervised OPLS-DA discriminant model and run cross-validation
    and permutation testing on the feature table
- name: RStudio
  role: R environment in which the ropls model fit, metrics export, and scores-plot
    rendering are executed
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:fcd1606d57763dcf6ced4f9b1dd325c7e4b61d6c1dffdd2650e259de1c273d80
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# permutation-test-model-validation

## Summary

Diagnose supervised multivariate model overfitting (e.g., OPLS-DA on high-dimensional metabolomics feature tables) by contrasting goodness-of-fit R²Y with cross-validated Q²Y and permutation-test results, as demonstrated for the DPPH High/Low task on wild soybean extracts.

## When to use

Use when a supervised classifier (OPLS-DA, RF) fitted on a feature table where features vastly outnumber samples reports high training fit (R²Y) and you must establish whether it generalizes, via k-fold CV and permutation testing.

## When NOT to use

- Input is already image-encoded (MS2toImg grayscale molecular fingerprints) rather than an aligned feature table — the CNN pipeline bypasses feature matching and this validation diagnosis targets feature-table models
- The sample count exceeds or is comparable to the feature count, so p >> n overfitting is not the primary risk

## Inputs

- Merged untargeted metabolomics feature table (samples × features; e.g., 545 wild soybean accessions after 0.8-similarity feature merging)
- Class label vector (e.g., DPPH High/Low from normalized activity <0.4 vs >0.6)

## Outputs

- oplsd_metrics.csv containing R²Y, cross-validated Q²Y, and permutation-test results
- oplsda_scores_plot.png (High vs Low groups on first two OPLS-DA components)

## How to apply

Load the merged feature table (545 wild soybean accessions, features merged when the average MS1/RT/MS-MS similarity exceeds 0.8) together with class labels assigned from normalized DPPH activity (<0.4 = Low, >0.6 = High). In R, fit the OPLS-DA discriminant model with the ropls package, projecting High and Low groups onto the first two components, and record R²Y. Estimate Q²Y by seven-fold cross-validation, then run permutation tests (shuffling class labels) to obtain a null distribution of predictive performance. Write R²Y, Q²Y, and permutation results to oplsd_metrics.csv and render the two-component scores plot to oplsda_scores_plot.png. Judge success by the gap: the reference reproduction gives R²Y ≈ 0.976 versus Q²Y ≈ 0.287 — a large gap with weak permutation statistics is the classic indicator of severe overfitting when features greatly exceed samples.

## Related tools

- **ropls** (R package used to fit the supervised OPLS-DA discriminant model and run cross-validation and permutation testing on the feature table)
- **RStudio** (R environment in which the ropls model fit, metrics export, and scores-plot rendering are executed)

## Examples

```
Rscript -e 'library(ropls); oplsda <- opls(x, y = dpph_class, predI = 2, permI = 200, printL = TRUE)'
```

## Evaluation signals

- Reproduced R²Y ≈ 0.976 and cross-validated Q²Y ≈ 0.287 for the DPPH task; the large R²Y–Q²Y gap must be present
- oplslsd metrics file (oplsd_metrics.csv) exists and contains R²Y, Q²Y, and permutation-test result columns
- oplsda_scores_plot.png shows High and Low groups projected onto the first two OPLS-DA components
- Permutation-test statistics from label-shuffled runs fall well below the unpermuted model's Q²Y, confirming the overfitting diagnosis
- Class labels match the paper's thresholds: normalized DPPH <0.4 = Low, >0.6 = High

## Limitations

- Strong apparent class separation in the OPLS-DA scores plot does not imply validity; R²Y ≈ 0.976 with Q²Y ≈ 0.287 shows training performance can be near-perfect while generalization is poor
- Feature alignment errors in the merged feature table exacerbate overfitting of supervised multivariate models
- Traditional multivariate statistics (PCA, PCoA, OPLS-DA) are hampered when the number of features greatly exceeds the number of samples, which is almost always the case in metabolomics

## Evidence

- [other] OPLS-DA appears to separate classes but overfits, performing well on training data while generalizing poorly: [evidence span withheld — anchor status: not_found]
