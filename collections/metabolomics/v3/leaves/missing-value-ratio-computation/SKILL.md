---
name: missing-value-ratio-computation
description: Use when a spatial metabolomics feature matrix (pixels × m/z ion intensities,
  e.g. 14,260 pixels × 3,044 ions) needs quality assessment of detection completeness,
  or when deciding which pixels/ions to retain before preprocessing.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (e.g. imzML vendor files) rather than
  an already-picked feature matrix of m/z intensities per pixel — SMAnalyst QC operates
  on the uploaded feature table only
- The goal is peak picking or ion feature detection from spectra; missing-value ratios
  presuppose a completed peak-picking step with pixels × ions dimensions
- Assessing background signal spatial consistency (QC1) or median intensity distributions
  (QC2) — those use correlation and intensity metrics, not missing-value counts
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Web/R Shiny platform whose QC module (QC3/QC4) computes and visualizes pixel
    and ion missing ratios on the uploaded feature matrix
  repo: https://github.com/mzlab-research/SManalyst
- name: R
  role: Local runtime (≥ v4.1.0) for running SMAnalyst and computing missing-value
    ratios outside the web interface
- name: ggplot2
  role: Used to visualize the pixel and ion missing-rate distributions
- name: tidyverse
  role: Supporting R package stack for matrix manipulation and missing-rate summarization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:7f840a186ade3cb0002109249bfd691890e68f05ed8c9f1c82c035438702c844
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - R (>=)
  - SMAnalyst
  - ggplot2
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# missing-value-ratio-computation

## Summary

Computation of missing-value rates in spatial metabolomics feature matrices as part of SMAnalyst's QC assessment: per-pixel missing ratios (proportion of undetected ions per pixel) and per-ion missing ratios (proportion of pixels where each ion was undetected), used to judge data completeness before and after noise-ion filtering.

## When to use

Use when a spatial metabolomics feature matrix (pixels × m/z ion intensities, e.g. 14,260 pixels × 3,044 ions) needs quality assessment of detection completeness, or when deciding which pixels/ions to retain before preprocessing.

## When NOT to use

- Input is raw mass-spectrometry imaging data (e.g. imzML vendor files) rather than an already-picked feature matrix of m/z intensities per pixel — SMAnalyst QC operates on the uploaded feature table only
- The goal is peak picking or ion feature detection from spectra; missing-value ratios presuppose a completed peak-picking step with pixels × ions dimensions
- Assessing background signal spatial consistency (QC1) or median intensity distributions (QC2) — those use correlation and intensity metrics, not missing-value counts

## Inputs

- Feature matrix CSV with columns 1-2 as pixel X/Y coordinates and subsequent columns as ion intensities at each m/z value (one row per spatial pixel), e.g. OMIX009541 mouse brain coronal section, AFAD-ESI (+) mode, 100 μm resolution, 14,260 pixels × 3,044 ions
- Noise-filtered ion set (ions retained at default noise score threshold ≥30)

## Outputs

- Per-pixel missing ratio values (proportion of undetected ions per pixel)
- Per-ion missing ratio values (proportion of pixels where each ion was undetected)
- Missing-rate distribution visualizations (SMAnalyst QC dashboard plots / Supplementary Figure S7-style figures)

## How to apply

Load the feature matrix in R with the first two columns as X/Y pixel coordinates and remaining columns as m/z ion intensities (CSV format, one row per spatial pixel). Compute the pixel missing ratio as the proportion of undetected (zero/NA) ions per pixel across all m/z columns, and the ion missing ratio as the proportion of pixels where each ion was undetected. Run this after background-pixel removal and noise-ion filtering (default noise score threshold ≥30) so the missing-value profile reflects retained features, then visualize both distributions (e.g. with ggplot2 histograms, SMAnalyst QC3/QC4 dashboard panels). Interpret results against Dataset 1 benchmarks: most pixels should have missing rates below 10% and most ions below 5%; pixels or ions exceeding these rates are candidates for removal or flagging in the preprocessing parameter set.

## Related tools

- **SMAnalyst** (Web/R Shiny platform whose QC module (QC3/QC4) computes and visualizes pixel and ion missing ratios on the uploaded feature matrix) — https://github.com/mzlab-research/SManalyst
- **R** (Local runtime (≥ v4.1.0) for running SMAnalyst and computing missing-value ratios outside the web interface)
- **ggplot2** (Used to visualize the pixel and ion missing-rate distributions)
- **tidyverse** (Supporting R package stack for matrix manipulation and missing-rate summarization)

## Examples

```
Rscript -e "fm <- read.csv('feature_matrix.csv'); pix_miss <- rowMeans(fm[, -(1:2)] == 0); ion_miss <- colMeans(fm[, -(1:2)] == 0)"
```

## Evaluation signals

- On the reference Dataset 1 (OMIX009541, 14,260 × 3,044), most pixels should show a missing rate below 10%
- Most ions should show a missing rate below 5% after default noise filtering (score ≥30)
- Missing-rate distributions should be computed only over retained pixels and ions (after removal of 39% noise ions and background pixels at total ion intensity threshold 107.4)
- Per-pixel missing ratios must be derived from the m/z intensity columns only, excluding the first two X/Y coordinate columns
- Resulting figures and metrics should be reproducible via the SMAnalyst QC dashboard panels

## Limitations

- Thresholds like <10% pixel missing and <5% ion missing are benchmarks reported for the mouse brain Dataset 1; other tissue types or acquisition modes may show different acceptable ranges
- Missing values are defined as undetected ions within the picked feature matrix; ions lost entirely during peak picking are invisible to this metric
- No substantive discussion-section content or changelog available in the provided document for additional caveats

## Evidence

- [readme] Missing value assessment (QC4): "Background consistency evaluation (QC1) Noise ion filtering (QC2) Intensity distribution analysis (QC3) Missing value assessment (QC4)"
- [readme] feature matrix format: "SManalyst requires a **feature matrix** in CSV format: * **Columns 1-2**: Pixel coordinates (X, Y) * **Subsequent columns**: Ion intensities at each m/z value"
- [intro] noise ion threshold: "Adjust noise ion threshold (default: score ≥30)"
- [intro] QC workflow: [evidence span withheld — anchor status: not_found]
- [abstract] dataset scale: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
