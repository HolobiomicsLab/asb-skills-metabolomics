---
name: noise-ion-filtering
description: Use when preprocessing a pixel × m/z feature matrix (X/Y coordinate columns
  plus ion intensity columns) to remove spatially random, likely non-biological ions
  before metabolite annotation, UMAP-kmeans/Seurat pixel clustering, SpaGene co-expression
  analysis, or differential analysis.
when_to_use_negative:
- The input is already a filtered or annotated ion list rather than a raw pixel ×
  m/z feature matrix with X/Y coordinates — QC2 operates on the full spatial feature
  matrix.
- The goal is background consistency evaluation (QC1), intensity distribution analysis
  (QC3), or missing value assessment (QC4) — these are separate SMAnalyst QC modules,
  not noise-ion filtering.
- The dataset has no meaningful spatial structure or coordinates, since the score
  is derived from the spatstat quadrat test on spatial point patterns.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3174
tools:
- name: SMAnalyst
  role: 'Web/R platform exposing noise-ion filtering as QC module QC2 with an adjustable
    noise ion threshold (default: score ≥ 30) and a QC dashboard'
  repo: https://github.com/mzlab-research/SManalyst
- name: spatstat
  role: Provides the quadrat test used per ion to assess conformity to Complete Spatial
    Randomness and generate the p-value converted into the noise score
- name: R
  role: Runtime environment (≥ v4.1.0) for loading the feature matrix, running the
    quadrat tests, and exporting the per-ion score table
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:2d8dc9cc235bf0ea6111f37037e7062c686f1b6a9b9c2a938e5dda533a7e4d93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - R (>=)
  - SMAnalyst
  - Seurat
  - SpaGene
  - ggplot2
  - spatstat
  - tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# noise-ion-filtering

## Summary

SMAnalyst's noise-ion filtering (QC2) scores every ion in a spatial metabolomics feature matrix by spatial randomness — the noise score is the negative log10 of the spatstat quadrat-test p-value against Complete Spatial Randomness — and removes ions below the default threshold (score ≥ 30 retained), so only spatially structured ions proceed to annotation, clustering, and differential analysis.

## When to use

Use when preprocessing a pixel × m/z feature matrix (X/Y coordinate columns plus ion intensity columns) to remove spatially random, likely non-biological ions before metabolite annotation, UMAP-kmeans/Seurat pixel clustering, SpaGene co-expression analysis, or differential analysis.

## When NOT to use

- The input is already a filtered or annotated ion list rather than a raw pixel × m/z feature matrix with X/Y coordinates — QC2 operates on the full spatial feature matrix.
- The goal is background consistency evaluation (QC1), intensity distribution analysis (QC3), or missing value assessment (QC4) — these are separate SMAnalyst QC modules, not noise-ion filtering.
- The dataset has no meaningful spatial structure or coordinates, since the score is derived from the spatstat quadrat test on spatial point patterns.

## Inputs

- Feature matrix in CSV format: columns 1–2 are pixel X/Y coordinates, subsequent columns are ion intensities at each m/z value, one row per spatial pixel (test set: 14,260 pixels × 3,044 ions, from NGDC OMIX OMIX009541 or the SMAnalyst Tutorial Panel → Example Data)
- Noise ion threshold (default: score ≥ 30)
- Spatial pixel coordinates used to build point patterns for the quadrat test

## Outputs

- Per-ion noise score table (−log10 quadrat-test p-value per ion) with retained/removed labels, e.g. noise_filter_results.csv
- Noise-score distribution plot
- Summary of the fraction of ions filtered out versus retained for downstream analysis
- Filtered ion list used as input for metabolite annotation, pixel clustering, and spatial co-expression analysis

## How to apply

Load the feature matrix in R (≥ v4.1.0) with columns 1–2 holding X/Y pixel coordinates and remaining columns holding per-pixel ion intensities; the SMAnalyst test dataset is a mouse brain coronal section (AFAD-ESI positive mode, 100 μm pixel size, 14,260 pixels × 3,044 ions). For each ion column, run the spatstat (3.1-1) quadrat test to assess whether the ion's spatial distribution conforms to Complete Spatial Randomness, then convert the p-value to a noise score defined as −log10(p-value). Apply the default noise-score threshold of 30, labeling ions with scores below the threshold as potential noise to remove and keeping ions with score ≥ 30 for downstream analysis. Summarize the removed/retained fractions (the paper reports ~39% of noise ions filtered and ~61% retained on Dataset 1 at the default threshold), write per-ion scores and labels to CSV, and plot the noise-score distribution. Rationale: noise filtering is applied before spatial pattern discovery because it is shown to improve cluster coherence in downstream UMAP-kmeans and SpaGene analyses.

## Related tools

- **SMAnalyst** (Web/R platform exposing noise-ion filtering as QC module QC2 with an adjustable noise ion threshold (default: score ≥ 30) and a QC dashboard) — https://github.com/mzlab-research/SManalyst
- **spatstat** (Provides the quadrat test used per ion to assess conformity to Complete Spatial Randomness and generate the p-value converted into the noise score)
- **R** (Runtime environment (≥ v4.1.0) for loading the feature matrix, running the quadrat tests, and exporting the per-ion score table)

## Examples

```
Rscript -e 'library(spatstat); q <- quadrat.test(ppp(x, y)); score <- -log10(q$p.value); keep <- score >= 30'
```

## Evaluation signals

- Reproduce the paper's split on Dataset 1 at the default threshold of 30: approximately 39% of noise ions filtered out and 61% of ions retained (Supplementary Figure S6).
- Per-ion score equals the negative log base 10 of the spatstat quadrat-test p-value for that ion's spatial distribution.
- The output table contains one score and one retained/removed label for every input ion (3,044 ions in the test dataset), and the spatial pixel count is unchanged (14,260 pixels).
- Downstream check: after filtering, UMAP-kmeans (k=25) resolves recognizable cortex, hippocampus, midbrain, hindbrain, and fiber tract divisions, and SpaGene yields eight spatial expression patterns (Pattern 4 cortex-enriched, Pattern 7 tissue-edge-enriched), since noise filtering improves cluster coherence.

## Limitations

- The threshold is user-adjustable in SMAnalyst; the 39%/61% split is validated specifically on the mouse brain Dataset 1 at the default score ≥ 30 and may differ for other tissues, modes, or resolutions.
- Noise scores depend on a prior tissue/background region selection (interactive lasso tool) and on pixel coordinates being correctly formatted in columns 1–2 of the CSV feature matrix.
- Filtering retains ~61% of noise ions, so the default threshold is a conservative screen rather than an exhaustive removal of non-biological ions.

## Evidence

- [readme] Adjust noise ion threshold (default: score ≥30): "Adjust noise ion threshold (default: score ≥30)"
- [readme] Noise ion filtering (QC2): "Noise ion filtering (QC2)"
- [readme] Feature matrix input format: "SManalyst requires a **feature matrix** in CSV format:"
- [abstract] Scale of validated dataset: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
