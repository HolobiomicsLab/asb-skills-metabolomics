---
name: tissue-enriched-ion-identification-wilcoxon
description: Use when preprocessing a spatial metabolomics peak table matrix (X/Y
  pixel coordinates plus ion intensity columns) to separate tissue from background
  pixels, and you have lasso-defined tissue and background region annotations to compare
  per-ion intensity distributions.
when_to_use_negative:
- Input is not a pixel-by-ion peak table matrix with spatial coordinates (e.g. already
  a feature table, raw spectra, or imaging file without a pixel grid)
- No user-annotated tissue and background regions exist, so the two pixel groups required
  for the rank-sum comparison cannot be formed
- The goal is noise ion or isotopic/adduct ion filtering (other SMQVP modules), not
  background pixel removal
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment in which the per-ion Wilcoxon rank-sum tests and fold-change comparisons
    are run
  repo: https://github.com/mzlab-research/SMQVP
- name: SMQVP
  role: Web application whose Process 2 (background pixel removal) implements tissue-enriched
    ion selection and downstream total-intensity thresholding; also provides the lasso
    tool for delineating regions
  repo: https://github.com/mzlab-research/SMQVP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:14428bf1d6150817dceedf5256a3a8813422dc6e3715e4df0ee304d6c7ac512d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Tissue-enriched ion identification by Wilcoxon rank-sum test

## Summary

Identifies spatial-metabolomics ion peaks that are significantly enriched in user-delineated tissue regions versus background regions using a Wilcoxon rank-sum test (fold change > 1, p < 0.05). These tissue-enriched ions are the basis of SMQVP's Process 2 background pixel removal on AFADESI mouse brain data.

## When to use

Use when preprocessing a spatial metabolomics peak table matrix (X/Y pixel coordinates plus ion intensity columns) to separate tissue from background pixels, and you have lasso-defined tissue and background region annotations to compare per-ion intensity distributions.

## When NOT to use

- Input is not a pixel-by-ion peak table matrix with spatial coordinates (e.g. already a feature table, raw spectra, or imaging file without a pixel grid)
- No user-annotated tissue and background regions exist, so the two pixel groups required for the rank-sum comparison cannot be formed
- The goal is noise ion or isotopic/adduct ion filtering (other SMQVP modules), not background pixel removal

## Inputs

- Peak table matrix (first two columns X/Y pixel coordinates, remaining columns ion intensities), e.g. 53,812 pixels x 2,654 ion peaks from OMIX009541
- User-delineated tissue and background region annotations (lasso-defined regions on the interaction image)

## Outputs

- List/mask of tissue-enriched ion peaks (fold change > 1, Wilcoxon p-value < 0.05)
- Total-intensity image built by summing tissue-enriched ion intensities per pixel (basis for thresholding at 107.37)

## How to apply

Load the peak table matrix (e.g. the AFADESI mouse brain table of 53,812 pixels x 2,654 ion peaks from OMIX009541 with pandas, using the first two columns as X/Y pixel coordinates). For each ion column, split pixel intensities into two groups: pixels inside the user-delineated tissue region and pixels inside the lasso-defined background region. Run a Wilcoxon rank-sum test between the two groups and classify an ion as tissue-enriched when its tissue/background fold change is > 1 and the p-value is < 0.05. The rationale is that only tissue-enriched ions meaningfully contribute to tissue signal, so summing their intensities per pixel yields a total-intensity image (Supplementary Figure S3B) whose distribution cleanly separates tissue from background pixels before a total-intensity threshold (e.g. 107.37) is applied for pixel removal.

## Related tools

- **R** (Environment in which the per-ion Wilcoxon rank-sum tests and fold-change comparisons are run) — https://github.com/mzlab-research/SMQVP
- **SMQVP** (Web application whose Process 2 (background pixel removal) implements tissue-enriched ion selection and downstream total-intensity thresholding; also provides the lasso tool for delineating regions) — https://github.com/mzlab-research/SMQVP

## Examples

```
wt <- wilcox.test(int_tissue, int_bg); is_enriched <- (mean(int_tissue)/mean(int_bg) > 1) & (wt$p.value < 0.05)
```

## Evaluation signals

- The output tissue-enriched ion list contains only ions with fold change > 1 and Wilcoxon p-value < 0.05
- Summing tissue-enriched ion intensities per pixel reproduces the total-intensity image shown in Supplementary Figure S3B
- Pixels whose tissue-enriched total intensity falls below the 107.37 threshold correspond to the background areas visible in the paper's segmentation (Supplementary Figure S3D)
- The filtered peak table retains the input schema (X, Y, then ion intensity columns) and contains only retained (tissue) pixels
- SMQVP's Ion Expression Proportion module corroborates that the selected ions have higher expression in the tissue region than in the background

## Limitations

- Depends on the quality of the user-delineated lasso regions; mis-drawn tissue or background regions will bias ion classification
- Thresholds (fold change > 1, p < 0.05, total-intensity cutoff 107.37) were calibrated on the AFADESI mouse brain dataset and may need re-tuning for other datasets
- No software version or license information is stated for SMQVP in the article discussion

## Evidence

- [other] Wilcoxon rank-sum test between tissue and background pixel groups, classifying ions with fold change > 1 and p-value < 0.05 as tissue-enriched: [evidence span withheld — anchor status: not_found]
- [other] Threshold 107.37 separates tissue from background pixels in Process 2: [evidence span withheld — anchor status: not_found]
- [readme] SMQVP module rationale for comparing tissue vs background ion expression: "Ion Expression Proportion: Determine the proportion of ions with higher expression in the tissue region."
- [readme] Background pixel removal is a stated SMQVP capability: "Background Noise Removal: Effectively identify and remove background pixels to improve data quality."
