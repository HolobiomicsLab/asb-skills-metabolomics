---
name: spatial-metabolomics-quality-control
description: Use when a spatial metabolomics feature matrix (pixels × m/z ion intensities,
  e.g., MALDI/DESI imaging MS) must be QC-assessed and noise-filtered before annotation,
  clustering, or differential analysis.
when_to_use_negative:
- Input is raw vendor imaging-MS data (e.g., imzML/RAW) that has not yet been peak-picked
  into a feature matrix — the skill expects an already-processed pixels × ions CSV
- The matrix has already passed noise filtering at threshold ≥30 and background removal;
  re-running would double-filter ions
- Analysis goal is metabolite annotation or differential analysis rather than pre-analysis
  quality assessment — use SMAnalyst's annotation or spatial/differential modules
  instead
edam_operation: http://edamontology.org/operation_4096
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Web/R platform whose Quality Control module implements QC1 (background consistency),
    QC2 (noise ion filtering), QC3 (intensity distribution), and QC4 (missing value
    assessment)
  repo: https://github.com/mzlab-research/SManalyst
- name: spatstat
  role: Provides the quadrat test (v3.1-1) used per ion column to test Complete Spatial
    Randomness and derive noise scores as -log10(p-value)
- name: R (>= v4.1.0)
  role: Runtime environment required to run SMAnalyst and the quadrat-test-based noise
    scoring locally
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:7a0c26b9697ea987c430c29f82b29fbca9663ea7241cffcf0a1f5d4dda81f9a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - R
  - R (>= v4.1.0)
  - R (>=)
  - SMAnalyst
  - SMQVP
  - SMQVP v1.0
  - ggplot2
  - spatstat
  - tidyverse
  edam_operation: http://edamontology.org/operation_4096
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spatial-metabolomics-quality-control

## Summary

SMAnalyst's four-module quality-control workflow for spatial metabolomics feature matrices: background consistency evaluation (QC1), noise ion filtering via a spatstat quadrat-test noise score with default threshold ≥30 (QC2), intensity distribution analysis (QC3), and missing value assessment (QC4).

## When to use

Use when a spatial metabolomics feature matrix (pixels × m/z ion intensities, e.g., MALDI/DESI imaging MS) must be QC-assessed and noise-filtered before annotation, clustering, or differential analysis.

## When NOT to use

- Input is raw vendor imaging-MS data (e.g., imzML/RAW) that has not yet been peak-picked into a feature matrix — the skill expects an already-processed pixels × ions CSV
- The matrix has already passed noise filtering at threshold ≥30 and background removal; re-running would double-filter ions
- Analysis goal is metabolite annotation or differential analysis rather than pre-analysis quality assessment — use SMAnalyst's annotation or spatial/differential modules instead

## Inputs

- CSV feature matrix with columns 1-2 as pixel X/Y coordinates and subsequent columns as ion intensities per m/z value (one row per spatial pixel)
- Mouse brain coronal section example dataset, AFAD-ESI (+) mode, 100 μm resolution, 14,260 pixels × 3,044 ions (NGDC OMIX009541)
- User-delineated tissue and background pixel regions (interactive lasso selection)

## Outputs

- noise_filter_results.csv with per-ion noise scores (-log10 of quadrat-test p-values) and noise/retained labels
- Noise-score distribution plot (Supplementary Figure S6-style)
- QC2 spatial map of median ion intensity per pixel and spectral distribution map of median intensities
- QC4 pixel and ion missing-rate visualizations
- Compiled QC report (qc_assessment_report.html) with QC dashboard metrics

## How to apply

Load the CSV feature matrix in R (≥ v4.1.0) with columns 1–2 as X/Y pixel coordinates and remaining columns as per-pixel ion intensities (e.g., the 14,260-pixel × 3,044-ion mouse brain coronal section, OMIX009541). QC1: interactively delineate tissue/background regions with the lasso tool and compute correlation coefficients between background-region spectra, expecting correlations > 0.99. QC2: for each ion column, run the spatstat (3.1-1) quadrat test against Complete Spatial Randomness, convert the p-value to a noise score of -log10(p), and label ions below the default threshold of 30 as noise — on Dataset 1 this removes 39% of noise ions while retaining 61% for downstream analysis. QC3: map the median ion intensity per pixel spatially and across the spectral distribution (cumulative intensity ~7 × 10^7). QC4: compute the pixel missing ratio (undetected ions per pixel) and ion missing ratio (pixels where each ion is undetected), expecting most pixels below 10% and most ions below 5% missing.

## Related tools

- **SMAnalyst** (Web/R platform whose Quality Control module implements QC1 (background consistency), QC2 (noise ion filtering), QC3 (intensity distribution), and QC4 (missing value assessment)) — https://github.com/mzlab-research/SManalyst
- **spatstat** (Provides the quadrat test (v3.1-1) used per ion column to test Complete Spatial Randomness and derive noise scores as -log10(p-value))
- **R (>= v4.1.0)** (Runtime environment required to run SMAnalyst and the quadrat-test-based noise scoring locally)

## Examples

```
library(spatstat); p <- quadrat.test(ppp(x, y))$p.value; noise_score <- -log10(p); keep <- noise_score >= 30
```

## Evaluation signals

- Background-region spectra should show correlation coefficients > 0.99 (QC1, as reported for Dataset 1)
- At the default noise-score threshold of 30, approximately 39% of noise ions are filtered out and 61% retained, matching Supplementary Figure S6
- Noise scores must equal -log10 of the spatstat quadrat-test p-value and be monotonically comparable against the threshold 30
- Most pixels show a missing rate below 10% and most ions below 5% (QC4)
- Cumulative total ion intensity should reach approximately 7 × 10^7 on Dataset 1 (QC3)

## Limitations

- QC1 background delineation is an interactive lasso selection, so the background-region correlation outcome depends on user-chosen regions
- At the default threshold of 30, filtering removes only 39% of noise ions — 61% of flagged noise ions are still retained for downstream analysis
- Tested at the scale of >14,000 pixels and >3,044 ion peaks (mouse brain dataset); behavior on much larger matrices is not established in this article

## Evidence

- [abstract] Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks): "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [readme] Adjust noise ion threshold (default: score ≥30): "Adjust noise ion threshold (default: score ≥30)"
- [readme] Background consistency evaluation (QC1): "Background consistency evaluation (QC1)"
- [intro] SMAnalyst significantly lowers the analysis barrier, by providing a unified solution that eliminates the need for tool switching and advanced computational skills.: "SMAnalyst significantly lowers the analysis barrier, by providing a unified solution that eliminates the need for tool switching and advanced computational skills."
