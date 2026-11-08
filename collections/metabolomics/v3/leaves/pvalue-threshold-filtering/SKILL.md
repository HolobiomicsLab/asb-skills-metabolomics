---
name: pvalue-threshold-filtering
description: Use when a spatial metabolomics feature matrix (pixel coordinates + per-pixel
  ion intensities) must be quality-controlled before annotation or clustering, and
  a decision is needed on which ions are noise versus spatially informative.
when_to_use_negative:
- The input is already a filtered or annotated feature table and no noise-ion re-evaluation
  is needed (the skill targets raw feature matrices entering QC2).
- The goal is differential metabolite detection with fold-change/significance filters
  — that is a separate SMAnalyst filter operating on group comparisons, not CSR-based
  noise scoring.
- The data are not spatially resolved pixel measurements (no X/Y coordinates per row),
  since the quadrat test requires per-pixel spatial positions.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: SMAnalyst
  role: 'Web/R platform exposing noise ion filtering as QC2, with an adjustable noise
    ion threshold (default: score >= 30) and a QC metrics dashboard'
  repo: mzlab-research/SManalyst
- name: spatstat
  role: R package (3.1-1) supplying the quadrat test used to compute per-ion CSR p-values
    that are converted to noise scores
- name: R
  role: Runtime (>= v4.1.0) in which the feature matrix is loaded, quadrat tests are
    run per ion, and scores/labels are exported
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:f81bc532182cd5075d8aa90be28b6defd5e6cc0e50a12b4b3286cef1babbea05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - SMAnalyst
  - spatstat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pvalue-threshold-filtering

## Summary

Noise-ion filtering for spatial metabolomics feature matrices in which each ion's spatial distribution is tested against Complete Spatial Randomness (CSR) with the spatstat quadrat test, its p-value is converted to a noise score (-log10 p), and ions are retained or removed by a score threshold (default 30) in SMAnalyst's QC2 step.

## When to use

Use when a spatial metabolomics feature matrix (pixel coordinates + per-pixel ion intensities) must be quality-controlled before annotation or clustering, and a decision is needed on which ions are noise versus spatially informative.

## When NOT to use

- The input is already a filtered or annotated feature table and no noise-ion re-evaluation is needed (the skill targets raw feature matrices entering QC2).
- The goal is differential metabolite detection with fold-change/significance filters — that is a separate SMAnalyst filter operating on group comparisons, not CSR-based noise scoring.
- The data are not spatially resolved pixel measurements (no X/Y coordinates per row), since the quadrat test requires per-pixel spatial positions.

## Inputs

- CSV feature matrix with columns 1-2 = X/Y pixel coordinates and subsequent columns = ion intensities at each m/z (one row per spatial pixel), e.g. 14,260 pixels x 3,044 ions
- Noise-score threshold (default: score >= 30 retained)

## Outputs

- Per-ion noise-score and keep/remove label table (e.g. noise_filter_results.csv)
- Filtered ion list / filtered feature matrix retained for downstream metabolite annotation and spatial analysis
- Summary of fraction of noise ions filtered out vs retained (e.g. 39% filtered / 61% retained)
- Noise-score distribution plot

## How to apply

Load the feature matrix in R (columns 1-2 = X/Y pixel coordinates, remaining columns = per-pixel ion intensities; e.g. the mouse brain test set of 14,260 pixels x 3,044 ions). For each ion column, run the spatstat (3.1-1) quadrat test to assess whether the ion's spatial distribution conforms to Complete Spatial Randomness, then define noise_score = -log10(p-value) so that strongly non-random (structured) ions get high scores. Apply the default threshold of score >= 30: ions below the threshold are labeled noise and removed, the rest are retained for downstream in-depth analysis; the threshold is user-adjustable in the SMAnalyst Quality Control tab. Summarize removed vs retained fractions, write per-ion scores and labels (e.g. noise_filter_results.csv), and plot the noise-score distribution. Rationale: spatially random signal is treated as instrument/chemical noise, and the score-based cutoff thresholds the CSR-test p-values in one place within SMAnalyst's QC2 step.

## Related tools

- **SMAnalyst** (Web/R platform exposing noise ion filtering as QC2, with an adjustable noise ion threshold (default: score >= 30) and a QC metrics dashboard) — mzlab-research/SManalyst
- **spatstat** (R package (3.1-1) supplying the quadrat test used to compute per-ion CSR p-values that are converted to noise scores)
- **R** (Runtime (>= v4.1.0) in which the feature matrix is loaded, quadrat tests are run per ion, and scores/labels are exported)

## Examples

```
Rscript -e 'library(spatstat); q <- quadrat.test(ion_ppp); noise_score <- -log10(q$p.value); keep <- noise_score >= 30'
```

## Evaluation signals

- On the mouse brain Dataset 1 at the default threshold of 30, the filtered/retained split should reproduce approximately 39% of noise ions filtered out and 61% retained (Supplementary Figure S6).
- Every noise_score must equal -log10 of its ion's quadrat-test p-value (monotone: smaller p-value -> higher score).
- All ions with noise score >= 30 are labeled retained; all ions below 30 are labeled noise/removed.
- Output CSV contains one row per ion (3,044 ions for the test dataset) with score and label columns, no missing per-ion entries.
- The noise-score distribution plot matches the per-ion scores in the exported CSV.

## Limitations

- At the default threshold of 30, 61% of noise ions are still retained for downstream analysis, so filtering is conservative rather than exhaustive.
- The CSR-based quadrat test judges only spatial randomness; ions removed as 'noise' may include genuinely uniform biological signals.
- Requires a spatially ordered feature matrix with pixel coordinates; ordering/geometry of pixels affects quadrat tessellation and thus p-values.

## Evidence

- [intro] noise ion threshold default: "Adjust noise ion threshold (default: score ≥30)"
- [readme] QC2 noise ion filtering: "Background consistency evaluation (QC1)
    *   Noise ion filtering (QC2)"
- [readme] QC tab usage: "Select tissue/background regions (interactive lasso tool)
    *   Adjust noise ion threshold (default: score ≥30)
    *   Review QC metrics in dashboard"
- [readme] input format: "Columns 1-2: Pixel coordinates (X, Y)
*   Subsequent columns: Ion intensities at each m/z value"
