---
name: background-spectral-correlation-analysis
description: Use when a spatial metabolomics feature matrix (pixels x m/z ion intensities)
  must be QC-checked before preprocessing, specifically to verify that background-region
  spectra are spatially consistent prior to background pixel removal and noise ion
  filtering.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (e.g., mzML/imzML) rather than a pixel-by-ion
  feature matrix — SMAnalyst QC1 requires the pre-built feature matrix
- The goal is metabolite annotation or spatial clustering, not quality assessment
  of background signal stability
- The dataset contains no distinguishable background/non-tissue pixels, so no background
  region can be delineated for correlation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMAnalyst
  role: Web/R platform providing the QC1 background consistency evaluation with interactive
    lasso selection of tissue/background regions
  repo: https://github.com/mzlab-research/SManalyst
- name: ggplot2
  role: R visualization of total intensity distribution maps and QC metrics
- name: tidyverse
  role: R data wrangling of the pixel-by-ion feature matrix during QC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:8f493c93f6f86ed4856ecaf94a992958448c8f3f24b503f873b8dcd4e18f9ef1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>=)
  - SMAnalyst
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# background-spectral-correlation-analysis

## Summary

QC1 from SMAnalyst: evaluate the spatial consistency of background (non-tissue) spectra in an imaging mass spectrometry feature matrix by computing pairwise correlation coefficients between spectra of interactively selected background pixel regions, confirming background signal stability before noise filtering and preprocessing.

## When to use

Use when a spatial metabolomics feature matrix (pixels x m/z ion intensities) must be QC-checked before preprocessing, specifically to verify that background-region spectra are spatially consistent prior to background pixel removal and noise ion filtering.

## When NOT to use

- Input is raw mass-spectrometry imaging data (e.g., mzML/imzML) rather than a pixel-by-ion feature matrix — SMAnalyst QC1 requires the pre-built feature matrix
- The goal is metabolite annotation or spatial clustering, not quality assessment of background signal stability
- The dataset contains no distinguishable background/non-tissue pixels, so no background region can be delineated for correlation

## Inputs

- Spatial metabolomics feature matrix in CSV format (columns 1-2: pixel X/Y coordinates; remaining columns: ion intensities at each m/z; each row one spatial pixel)
- Total intensity distribution map of pixels (for interactive tissue/background delineation)

## Outputs

- Pairwise spectral correlation coefficients between background-region pixels (QC1 metric)
- QC assessment report figures/tables (e.g., qc_assessment_report.html) supporting background pixel removal and subsequent preprocessing parameters

## How to apply

Load the feature matrix (e.g., the mouse brain Dataset 1 of 14,260 pixels x 3,044 ions from NGDC OMIX009541, AFAD-ESI (+) mode, 100um resolution) in R with the first two columns as X/Y pixel coordinates. Use SMAnalyst's interactive lasso tool on the total intensity distribution map of pixels to delineate tissue and background pixel sets. Compute pairwise correlation coefficients between spectra of the selected background regions to quantify spatial consistency of background signals (SMAnalyst QC1). Accept the assessment when background-region spectral correlation coefficients exceed 0.99, as reported for Dataset 1, then proceed to downstream QC2/QC3 checks (median intensity maps, missing-rate metrics: most pixels < 10% missing, most ions < 5%) and noise ion filtering at the default threshold score >= 30.

## Related tools

- **SMAnalyst** (Web/R platform providing the QC1 background consistency evaluation with interactive lasso selection of tissue/background regions) — https://github.com/mzlab-research/SManalyst
- **ggplot2** (R visualization of total intensity distribution maps and QC metrics)
- **tidyverse** (R data wrangling of the pixel-by-ion feature matrix during QC)

## Examples

```
Rscript -e 'm <- read.csv("feature_matrix.csv"); bg <- m[m$total_intensity < 107.4, 3:ncol(m)]; cor_bg <- cor(t(bg)); min(cor_bg[upper.tri(cor_bg)])'
```

## Evaluation signals

- Background-region pairwise spectral correlation coefficients exceed 0.99, matching the value reported for Dataset 1
- Background pixels are removed at the total ion intensity threshold (reported value: 107.4 for Dataset 1)
- After downstream QC3, most pixels have missing rates below 10% and most ions below 5%
- Cumulative intensity after filtering reaches ~7 x 10^7 and ~39% of ions are filtered as noise at the default noise score threshold of 30

## Limitations

- Background delineation depends on interactive lasso selection of tissue/background regions, so region choice affects the correlation assessment
- QC1 only evaluates background consistency; it does not by itself identify noise ions — a separate noise ion filtering step (default score >= 30) is required
- Validation thresholds (correlation > 0.99) were established on the mouse brain AFAD-ESI (+) 100um dataset and may not transfer to other tissue types or acquisition modes

## Evidence

- [readme] Background consistency evaluation (QC1): "Systematic Quality Control: * Background consistency evaluation (QC1) * Noise ion filtering (QC2) * Intensity distribution analysis (QC3) * Missing value assessment (QC4)"
- [readme] Select tissue/background regions (interactive lasso tool): "**Quality Control**: * Select tissue/background regions (interactive lasso tool) * Adjust noise ion threshold (default: score ≥30) * Review QC metrics in dashboard"
- [results] background-region spectra were highly consistent (correlation coefficients > 0.99): [evidence span withheld — anchor status: not_found]
- [abstract] Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
