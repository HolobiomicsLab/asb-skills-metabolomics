---
name: msi-pixel-filtering
description: Use when a spatial metabolomics peak table matrix (pixels x m/z ion columns)
  contains off-tissue/background pixels that must be removed before downstream clustering,
  e.g. when an intensity image shows clearly separable background signal.
when_to_use_negative:
- Input is not a peak table matrix (first two columns X/Y coordinates followed by
  m/z intensity columns) — SMQVP's documented input format
- The goal is to filter noise IONS rather than background PIXELS — that is a separate
  SMQVP module (noise ion filtering)
- No tissue and background regions can be delineated, so tissue-enriched ions cannot
  be identified via the Wilcoxon rank-sum comparison
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMQVP
  role: Web application (R/Shiny) used to delineate tissue vs. background regions
    with the lasso tool, identify tissue-enriched ions, and apply the total-intensity
    threshold to remove background pixels
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) for SMQVP; used to perform the Wilcoxon rank-sum
    test and threshold filtering on the peak table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:ad28a937a05f8abd1bec44b866f76aa5c8741501d6011f9be067795c50536834
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msi-pixel-filtering

## Summary

Background pixel removal for spatial metabolomics (SMQVP Process 2): construct a total-intensity image from tissue-enriched ions and apply a fixed total-intensity threshold (107.37 in the AFADESI mouse brain example) to remove background pixels from the peak table matrix.

## When to use

Use when a spatial metabolomics peak table matrix (pixels x m/z ion columns) contains off-tissue/background pixels that must be removed before downstream clustering, e.g. when an intensity image shows clearly separable background signal.

## When NOT to use

- Input is not a peak table matrix (first two columns X/Y coordinates followed by m/z intensity columns) — SMQVP's documented input format
- The goal is to filter noise IONS rather than background PIXELS — that is a separate SMQVP module (noise ion filtering)
- No tissue and background regions can be delineated, so tissue-enriched ions cannot be identified via the Wilcoxon rank-sum comparison

## Inputs

- Peak table matrix CSV with first two columns as X/Y pixel coordinates and remaining columns as ion (m/z) intensities (e.g. 53,812 pixels x 2,654 peaks from AFADESI mouse brain, OMIX009541)
- User-delineated tissue and background region annotations (lasso-defined regions in SMQVP)

## Outputs

- Filtered peak intensity table containing only retained (tissue) pixels, preserving the original schema (X, Y, then ion intensity columns), e.g. background_removed_peak_table.csv

## How to apply

Load the peak table matrix (e.g. 53,812 pixels x 2,654 ion peaks from AFADESI mouse brain data, OMIX009541) with the first two columns as X/Y pixel coordinates. Using user-delineated tissue and background regions (lasso-defined in SMQVP), classify each ion as tissue-enriched via a Wilcoxon rank-sum test with fold change > 1 and p < 0.05. Sum the intensities of the tissue-enriched ions per pixel to build the total-intensity image (Supplementary Figure S3B), then apply the intensity threshold — 107.37 for this dataset — retaining pixels at or above it and dropping those below (Supplementary Figure S3D). Rationale: tissue-enriched ions concentrate signal in tissue so the threshold cleanly separates background; the filtered table improves downstream clustering so results better align with tissue morphology.

## Related tools

- **SMQVP** (Web application (R/Shiny) used to delineate tissue vs. background regions with the lasso tool, identify tissue-enriched ions, and apply the total-intensity threshold to remove background pixels) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) for SMQVP; used to perform the Wilcoxon rank-sum test and threshold filtering on the peak table)

## Examples

```
import pandas as pd; df = pd.read_csv('peak_table.csv'); tot = df[tissue_enriched_ions].sum(axis=1); kept = df[tot >= 107.37]; kept.to_csv('background_removed_peak_table.csv', index=False)
```

## Evaluation signals

- Output file exists and retains the peak-table schema: X, Y, followed by the same ion intensity columns as the input
- All retained pixels have tissue-enriched-ion total intensity >= 107.37; all pixels below the threshold are removed
- Removed pixels spatially correspond to background areas visible in the paper's segmentation (Supplementary Figure S3D)
- The tissue-enriched ion set satisfies fold change > 1 and Wilcoxon rank-sum p < 0.05 for every retained ion
- Downstream clustering on the filtered table aligns better with underlying tissue morphology than clustering on the unprocessed data

## Limitations

- The threshold value (107.37) is dataset-specific to the AFADESI mouse brain example; other datasets require re-deriving an appropriate threshold
- Requires manually delineated tissue and background regions (lasso tool), so quality depends on the user's region annotation
- Article/README do not specify an SMQVP software version, license, or changelog

## Evidence

- [readme] Background Noise Removal: Effectively identify and remove background pixels to improve data quality.: "Background Noise Removal: Effectively identify and remove background pixels to improve data quality."
- [readme] Use the lasso tool to delineate background and tissue regions on the interaction image: "Use the lasso tool to delineate background and tissue regions on the interaction image"
- [readme] The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode.: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode."
- [results] SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality.: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
- [abstract] This rigorous preprocessing resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses performed on unprocessed data.: "This rigorous preprocessing resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses performed on unprocessed data."
