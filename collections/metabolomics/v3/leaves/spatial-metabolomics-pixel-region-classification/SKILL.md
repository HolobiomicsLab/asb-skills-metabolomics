---
name: spatial-metabolomics-pixel-region-classification
description: Use when working with an AFADESI- or MALDI-style spatial metabolomics
  peak table (pixels x m/z ions) and you need to partition pixels into user-defined
  tissue vs. background regions to assess acquisition quality or tissue-enriched ion
  expression.
when_to_use_negative:
- Input is raw MS imaging data (e.g., imzML) that has not yet been peak-picked into
  a pixel-by-ion peak table matrix
- The acquisition has no background pixels outside the tissue (e.g., tightly cropped
  datasets), so background regions cannot be delineated
- The goal is ion identity annotation rather than pixel/region quality assessment
  — region classification does not assign m/z identities
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Web application whose lasso tool is used to delineate background and tissue
    regions on the interaction image for quality visualization
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment (with Shiny/SMQVP) for loading the peak table and programmatically
    grouping pixels and computing region-level statistics
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:581d701bca2bb5cdbc2a4bf16bf989dab5267112f836e8ffd7348d72d8e23df5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spatial-metabolomics-pixel-region-classification

## Summary

Classify pixels of a spatial metabolomics peak table matrix into tissue and background regions by lasso-based delineation on the median pixel intensity image, enabling downstream background-consistency (QVP2) and tissue-enriched ion (QVP3) quality checks in SMQVP.

## When to use

Use when working with an AFADESI- or MALDI-style spatial metabolomics peak table (pixels x m/z ions) and you need to partition pixels into user-defined tissue vs. background regions to assess acquisition quality or tissue-enriched ion expression.

## When NOT to use

- Input is raw MS imaging data (e.g., imzML) that has not yet been peak-picked into a pixel-by-ion peak table matrix
- The acquisition has no background pixels outside the tissue (e.g., tightly cropped datasets), so background regions cannot be delineated
- The goal is ion identity annotation rather than pixel/region quality assessment — region classification does not assign m/z identities

## Inputs

- Spatial metabolomics peak table matrix (first two columns = X/Y pixel coordinates, subsequent columns = m/z ion intensities per pixel)
- AFADESI mouse brain dataset from OMIX009541 (53,812 pixels x 2,654 ion peaks) as reference example

## Outputs

- Pixel-to-region assignment (region labels for each of the 53,812 pixels)
- Median per-ion region spectra used for background consistency and tissue-enrichment analyses
- Region-level tables such as background_correlations.csv (pairwise Pearson matrix) and tissue_enriched_ions.csv (fold change, p-value, tissue-enriched flag, overall proportion)

## How to apply

Load the peak table matrix where the first two columns hold X/Y pixel coordinates and the remaining columns are m/z ion intensities (e.g., the AFADESI mouse brain table of 53,812 pixels x 2,654 ion peaks from OMIX009541). Render the median pixel intensity image and use the lasso tool (SMQVP GUI) or programmatic intensity-based grouping to delineate user-defined tissue and background regions, mirroring the paper's five manually selected tissue and background regions. Treat low-intensity pixels outside the tissue as background; group them into multiple background regions so pairwise consistency can later be tested. For each region, compute per-ion summary spectra (e.g., median intensity across region pixels) that feed QVP2 Pearson correlation (threshold > 0.9, reported values > 0.99) and QVP3 Wilcoxon rank-sum classification (fold change > 1, p < 0.05). The rationale is that correct region delineation is a prerequisite: mislabeled background pixels contaminate both noise-ion filtering and tissue-enrichment statistics.

## Related tools

- **SMQVP** (Web application whose lasso tool is used to delineate background and tissue regions on the interaction image for quality visualization) — https://github.com/mzlab-research/SMQVP
- **R** (Environment (with Shiny/SMQVP) for loading the peak table and programmatically grouping pixels and computing region-level statistics)

## Examples

```
peak <- read.csv('peak_table.csv'); regions <- assign_regions(peak, median_image, lasso_selected_polygons); write.csv(regions, 'pixel_regions.csv', row.names=FALSE)
```

## Evaluation signals

- All pairwise Pearson correlations between delineated background-region median spectra exceed 0.99 (paper reports values consistently above 0.99; SMQVP criterion: > 0.9 indicates stable acquisition)
- Wilcoxon rank-sum classification of ions as tissue-enriched (fold change > 1 and p < 0.05) yields a proportion near the reported 37% of 2,654 detected ions (acceptance range [0.35, 0.39])
- Every pixel in the 53,812-pixel table receives exactly one region label and every tissue/background region contributes a non-empty median spectrum
- Output tables (background_correlations.csv, tissue_enriched_ions.csv) contain all required per-ion or per-pair fields (correlation, fold change, p-value, classification)

## Limitations

- Region delineation is manual/user-defined via lasso selection, so results depend on the analyst's choice of the five tissue and background regions
- No software version or license is specified for SMQVP, complicating exact reproducibility of region-delineation behavior
- The reported 37% tissue-enriched ion proportion is specific to the AFADESI mouse brain dataset and its particular region selections; other datasets require re-validation

## Evidence

- [readme] lasso region delineation on interaction image: "Use the lasso tool to delineate background and tissue regions on the interaction image."
- [readme] input format: "The input for SMQVP is a peak table matrix with the following structure: First two columns: X and Y coordinates of the pixels"
- [full_text] background consistency criterion: [evidence span withheld — anchor status: not_found]
- [full_text] tissue-enriched ion proportion: [evidence span withheld — anchor status: not_found]
- [readme] dataset availability: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541."
