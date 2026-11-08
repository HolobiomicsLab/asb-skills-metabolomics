---
name: pixel-region-delineation-msi
description: Use when a raw spatial metabolomics feature matrix (CSV with X/Y pixel
  columns followed by m/z intensity columns, e.g. 14,260 pixels x 3,044 ions) must
  be partitioned into tissue and background regions to set preprocessing thresholds.
when_to_use_negative:
- Input is already a cleaned, annotated metabolite table with background pixels removed
  — region delineation would re-filter already-processed data
- Data are raw instrument files (e.g. imaging MS acquisitions) not yet converted to
  the pixel-by-m/z feature matrix SMAnalyst requires
- The goal is differential or spatial clustering analysis on a matrix that has not
  yet passed QC1/QC2/QC3 assessment
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: SMAnalyst
  role: Web/R-Shiny platform providing the interactive lasso tool for tissue/background
    region selection, noise-ion threshold adjustment (default score >= 30), and QC
    dashboards
  repo: https://github.com/mzlab-research/SManalyst
- name: ggplot2
  role: R plotting library used to render the total-intensity distribution map, median-intensity
    spatial/spectral maps, and missing-rate plots
- name: tidyverse
  role: R data-manipulation stack for loading the feature matrix and computing per-pixel
    and per-ion missing ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:5747162edd59995fc7f564aaa8a67f9b3601ebd91d25c22b61555b87da7039c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>=)
  - SMAnalyst
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pixel-region-delineation-msi

## Summary

Interactive delineation of tissue vs. background pixels on a spatial metabolomics feature matrix (pixel coordinates X/Y plus m/z ion intensity columns), followed by background-consistency (QC1), noise-ion filtering (score >= 30), and missing-value QC in SMAnalyst. It is the first quality-control gate that determines which pixels and ions are removed before downstream metabolite annotation.

## When to use

Use when a raw spatial metabolomics feature matrix (CSV with X/Y pixel columns followed by m/z intensity columns, e.g. 14,260 pixels x 3,044 ions) must be partitioned into tissue and background regions to set preprocessing thresholds.

## When NOT to use

- Input is already a cleaned, annotated metabolite table with background pixels removed — region delineation would re-filter already-processed data
- Data are raw instrument files (e.g. imaging MS acquisitions) not yet converted to the pixel-by-m/z feature matrix SMAnalyst requires
- The goal is differential or spatial clustering analysis on a matrix that has not yet passed QC1/QC2/QC3 assessment

## Inputs

- Feature matrix CSV with columns 1-2 as X/Y pixel coordinates and subsequent columns as ion intensities per m/z (row = one spatial pixel)
- Mouse brain coronal section test dataset (AFAD-ESI (+) mode, 100 um resolution, 14,260 pixels x 3,044 ions; NGDC OMIX ID OMIX009541 or built-in tutorial example)

## Outputs

- Tissue/background pixel assignments from interactive lasso selection on the total intensity distribution map
- QC1 background-region pairwise spectral correlation coefficients
- Background pixel removal threshold (total ion intensity, e.g. 107.4) and filtered noise ion set (default score >= 30)
- QC2 median-intensity spatial and spectral maps and QC3 pixel/ion missing-rate metrics compiled into qc_assessment_report.html

## How to apply

Load the feature matrix (CSV, columns 1-2 are pixel X/Y coordinates, remaining columns are m/z ion intensities) into SMAnalyst or its R/Shiny environment. Generate the total-intensity distribution map of pixels and interactively delineate tissue and background pixel sets using the lasso tool. For each delineated background region, compute pairwise spectral correlation coefficients as QC1 — coefficients > 0.99 indicate spatially consistent background signals, and the interactive inspection determines the total ion intensity threshold at which background pixels are removed (107.4 on the mouse brain Dataset 1). Proceed to noise-ion filtering at the default noise score threshold of 30 (39% of ions removed on Dataset 1) and compute QC3 missing ratios: pixel missing rate (undetected ions per pixel) and ion missing rate (pixels where each ion is undetected). Compile the QC maps, spectral distribution plots, and threshold parameters into a report; on Dataset 1 most pixels and ions had missing rates below 10% and 5%, respectively.

## Related tools

- **SMAnalyst** (Web/R-Shiny platform providing the interactive lasso tool for tissue/background region selection, noise-ion threshold adjustment (default score >= 30), and QC dashboards) — https://github.com/mzlab-research/SManalyst
- **ggplot2** (R plotting library used to render the total-intensity distribution map, median-intensity spatial/spectral maps, and missing-rate plots)
- **tidyverse** (R data-manipulation stack for loading the feature matrix and computing per-pixel and per-ion missing ratios)

## Examples

```
install.packages(c("shinydashboard","ggplot2","shiny","shinythemes","tidyverse"))  # then launch SManalyst and load the 14,260 x 3,044 feature matrix CSV
```

## Evaluation signals

- Pairwise spectral correlation coefficients between delineated background regions exceed 0.99 (Dataset 1 expectation)
- After thresholding, most pixels show missing rates below 10% and most ions below 5% (QC3)
- Noise-ion filter at the default score threshold of 30 removes roughly 39% of ions on the 14,260-pixel x 3,044-ion mouse brain dataset
- Background pixels are removed at an explicit total ion intensity threshold (107.4 on Dataset 1) recorded from the interactive intensity map
- QC maps (QC2 median-intensity spatial map, spectral distribution, missing-rate plots) are produced and compiled into qc_assessment_report.html

## Limitations

- Delineation of tissue vs. background is interactive (lasso on the total-intensity map), so region choice is operator-dependent and thresholds such as the 107.4 total ion intensity cutoff are dataset-specific
- The 0.99 background correlation and missing-rate benchmarks are specific to the mouse brain Dataset 1 and may not transfer to other tissue types or acquisition modes
- No changelog or substantive discussion content was available in the source article

## Evidence

- [intro] Quality control: select tissue/background regions, adjust noise ion threshold, review QC metrics: "**Quality Control**:
    *   Select tissue/background regions (interactive lasso tool)
    *   Adjust noise ion threshold (default: score ≥30)
    *   Review QC metrics in dashboard"
- [readme] QC modules enumerated in README: "Background consistency evaluation (QC1)
    *   Noise ion filtering (QC2)
    *   Intensity distribution analysis (QC3)
    *   Missing value assessment (QC4)"
- [other] Background correlation validation criterion from source task card: [evidence span withheld — anchor status: not_found]
- [readme] Input format specification: "**Columns 1-2**: Pixel coordinates (X, Y)
*   **Subsequent columns**: Ion intensities at each m/z value
*   **Row order**: Each row represents one spatial pixel"
