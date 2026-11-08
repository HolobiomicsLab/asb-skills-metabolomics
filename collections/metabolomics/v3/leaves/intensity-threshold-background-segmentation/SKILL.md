---
name: intensity-threshold-background-segmentation
description: Use when a spatial metabolomics peak table (X/Y pixel coordinates plus
  m/z intensity columns) contains off-tissue background pixels that degrade clustering,
  and a total-intensity threshold must be derived or applied to segment tissue from
  background.
when_to_use_negative:
- Input is already a pixel-segmented or tissue-only peak table with no background
  pixels present
- The task is noise ion filtering (ion-level filtering) rather than pixel-level background
  removal — SMQVP's Process for noise ions is a separate step
- Peak table lacks pixel coordinate columns or the analysis goal is not spatial pixel
  segmentation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3963
tools:
- name: SMQVP
  role: Web application whose Process 2 (Background Noise Removal) implements tissue/background
    region selection and threshold-based background pixel removal
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment (with Shiny) in which SMQVP runs; used to load the peak table
    with pandas-equivalent tooling and apply the Wilcoxon test and intensity threshold
- name: Shiny
  role: Package underpinning SMQVP's user-friendly graphical interface for lasso region
    delineation and guided processing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:798539d582160ebfeb8e4de5aeb311df6efbda236e211a86abab3481097df937
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3963
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# intensity-threshold-background-segmentation

## Summary

Background pixel removal step (SMQVP Process 2) that classifies pixels in a spatial metabolomics peak table as tissue or background using a tissue-enriched total-intensity image and a total intensity threshold (107.37 for the AFADESI mouse brain dataset), then retains only tissue pixels for downstream analysis.

## When to use

Use when a spatial metabolomics peak table (X/Y pixel coordinates plus m/z intensity columns) contains off-tissue background pixels that degrade clustering, and a total-intensity threshold must be derived or applied to segment tissue from background.

## When NOT to use

- Input is already a pixel-segmented or tissue-only peak table with no background pixels present
- The task is noise ion filtering (ion-level filtering) rather than pixel-level background removal — SMQVP's Process for noise ions is a separate step
- Peak table lacks pixel coordinate columns or the analysis goal is not spatial pixel segmentation

## Inputs

- Peak table matrix CSV with first two columns as X/Y pixel coordinates and remaining columns as ion m/z intensities (e.g. 53,812 pixels x 2,654 ion peaks from AFADESI mouse brain, OMIX009541)
- User-delineated tissue and background region annotations (lasso-defined regions from SMQVP interaction image)

## Outputs

- Filtered peak table (background_removed_peak_table.csv) containing only retained tissue pixels, preserving the X, Y, then ion intensity column schema
- Total-intensity image of tissue-enriched ions used for threshold segmentation (cf. Supplementary Figure S3B/S3D)

## How to apply

Load the peak table matrix (e.g. the AFADESI mouse brain table of 53,812 pixels x 2654 ion peaks from OMIX009541) with the first two columns as X/Y pixel coordinates. Delineate tissue and background regions (lasso-defined, as in SMQVP QVP1-QVP3) and, per ion, run a Wilcoxon rank-sum test between tissue and background pixel groups, classifying ions with fold change > 1 and p-value < 0.05 as tissue-enriched. Sum tissue-enriched ion intensities per pixel to build a total-intensity image, then apply the intensity threshold — 107.37 in the paper's case — removing pixels below it and retaining pixels at or above it. Write the retained pixels to a filtered peak table (background_removed_peak_table.csv) preserving the X, Y, ion-intensity schema. Rationale: tissue-enriched ions carry the spatial signal, so their summed intensity separates true tissue from background, improving downstream clustering alignment with tissue morphology.

## Related tools

- **SMQVP** (Web application whose Process 2 (Background Noise Removal) implements tissue/background region selection and threshold-based background pixel removal) — https://github.com/mzlab-research/SMQVP
- **R** (Environment (with Shiny) in which SMQVP runs; used to load the peak table with pandas-equivalent tooling and apply the Wilcoxon test and intensity threshold)
- **Shiny** (Package underpinning SMQVP's user-friendly graphical interface for lasso region delineation and guided processing)

## Examples

```
Rscript -e 'df <- read.csv("peak_table.csv"); m <- rowSums(df[,-c(1,2)]) >= 107.37; write.csv(df[m,], "background_removed_peak_table.csv", row.names=FALSE)'
```

## Evaluation signals

- Output file background_removed_peak_table.csv exists and retains the peak-table schema (X, Y, then ion intensity columns)
- Every retained pixel has tissue-enriched total intensity >= the threshold (107.37) and every removed pixel falls below it
- Retained/removed pixel counts match the segmentation visible in the paper's Supplementary Figure S3D
- Tissue-enriched ion set agrees with the Wilcoxon rank-sum criteria: fold change > 1 and p-value < 0.05 between tissue and background regions
- Downstream clustering on the filtered table aligns better with tissue morphology than clustering on unprocessed data

## Limitations

- The total intensity threshold (107.37) is dataset-specific; it must be re-derived for other tissues, platforms, or acquisition modes
- Classification of tissue-enriched ions depends on user-delineated lasso regions, so poor region annotation propagates into the segmentation
- No changelog available for the repository and no software version or license information is specified for SMQVP

## Evidence

- [readme] Background Noise Removal: Effectively identify and remove background pixels to improve data quality.: "Background Noise Removal: Effectively identify and remove background pixels to improve data quality."
- [readme] SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality.: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
- [intro] Background Noise Removal: Effectively identify and remove background pixels: "Background Noise Removal: Effectively identify and remove background pixels to improve data quality."
- [other] Apply the paper's intensity threshold of 107.37, removing pixels classified as background (below the threshold) and retaining tissue pixels: [evidence span withheld — anchor status: not_found]
- [other] classifying ions with fold change > 1 and p-value < 0.05 as tissue-enriched: [evidence span withheld — anchor status: not_found]
