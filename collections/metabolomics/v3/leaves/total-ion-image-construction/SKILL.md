---
name: total-ion-image-construction
description: Use when you have a spatial metabolomics peak table matrix (X/Y pixel
  coordinates plus ion intensity columns) and need to distinguish tissue pixels from
  background pixels via a total-intensity image.
when_to_use_negative:
- Input is not a pixel-by-ion peak table matrix (e.g. raw mass spectrometry imaging
  files like imzML) — convert to a peak table first
- No user-annotated tissue and background regions exist to support the Wilcoxon rank-sum
  ion classification
- The goal is noise ion filtering or other SMQVP quality visualization points rather
  than background pixel removal
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMQVP
  role: Web application whose Process 2 performs background pixel removal by constructing
    a total-intensity image from tissue-enriched ions and thresholding at 107.37
  repo: github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP is implemented and in which
    the total-intensity computation can be reproduced
- name: Shiny
  role: Package powering SMQVP's user interface for uploading peak tables and lasso-delineating
    tissue/background regions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:e25754287709c2abdccb93389d561857155fdba7d32d53b2247332ebac76923d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  - Shiny
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# total-ion-image-construction

## Summary

Constructs a total-intensity image from tissue-enriched ions in spatial metabolomics peak tables (as in SMQVP's Process 2), enabling background/tissue pixel discrimination on the AFADESI mouse brain dataset using a total-intensity threshold of 107.37.

## When to use

Use when you have a spatial metabolomics peak table matrix (X/Y pixel coordinates plus ion intensity columns) and need to distinguish tissue pixels from background pixels via a total-intensity image.

## When NOT to use

- Input is not a pixel-by-ion peak table matrix (e.g. raw mass spectrometry imaging files like imzML) — convert to a peak table first
- No user-annotated tissue and background regions exist to support the Wilcoxon rank-sum ion classification
- The goal is noise ion filtering or other SMQVP quality visualization points rather than background pixel removal

## Inputs

- Peak table matrix CSV (first two columns = X/Y pixel coordinates; remaining columns = ion peak intensities), e.g. 53,812 pixels x 2,654 ion peaks from OMIX009541
- User-delineated tissue and background region annotations (lasso-defined regions on the interaction image)

## Outputs

- Total-intensity image over all pixels (summed tissue-enriched ion intensities, cf. Supplementary Figure S3B)
- Filtered peak intensity table containing only retained tissue pixels (background_removed_peak_table.csv)

## How to apply

Load the peak table matrix (e.g., the AFADESI mouse brain table of 53,812 pixels x 2,654 ion peaks from OMIX009541) with the first two columns as X/Y pixel coordinates. Identify tissue-enriched ions by running a Wilcoxon rank-sum test per ion between user-delineated tissue and background pixel groups (lasso-defined regions), keeping ions with fold change > 1 and p-value < 0.05. Sum the intensities of these tissue-enriched ions for every pixel to build the total-intensity image (corresponding to Supplementary Figure S3B). Apply the paper's threshold of 107.37: pixels below it are classified as background and removed, pixels at or above it are retained as tissue, matching the segmentation in Supplementary Figure S3D. Write the retained pixels' peak intensity table to background_removed_peak_table.csv.

## Related tools

- **SMQVP** (Web application whose Process 2 performs background pixel removal by constructing a total-intensity image from tissue-enriched ions and thresholding at 107.37) — github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP is implemented and in which the total-intensity computation can be reproduced)
- **Shiny** (Package powering SMQVP's user interface for uploading peak tables and lasso-delineating tissue/background regions)

## Examples

```
df = pd.read_csv('peak_table.csv'); ti = df.loc[:, tissue_enriched_ions].sum(axis=1); keep = df[ti >= 107.37]; keep.to_csv('background_removed_peak_table.csv', index=False)
```

## Evaluation signals

- Output file background_removed_peak_table.csv exists and retains the peak-table schema (X, Y, then ion intensity columns)
- Every retained pixel has a summed tissue-enriched-ion total intensity >= 107.37
- Removed pixels correspond to the background areas visible in the paper's segmentation (Supplementary Figure S3D)
- Constructed total-intensity image matches Supplementary Figure S3B
- Tissue-enriched ion set is reproducible: fold change > 1 and Wilcoxon rank-sum p-value < 0.05 per ion

## Limitations

- Requires manual lasso delineation of tissue and background regions, so results depend on the quality of user annotations
- The threshold 107.37 is dataset-specific (AFADESI mouse brain data); it must be re-derived for other datasets or platforms
- No changelog, software version, or license information is available for the SMQVP repository

## Evidence

- [other] Process 2 constructs a total intensity image from tissue-enriched ions and applies a total intensity threshold of 107.37, clearly distinguishing background pixels and effectively removing them from the AFADESI mouse brain dataset.: [evidence span withheld — anchor status: not_found]
- [readme] Background Noise Removal: Effectively identify and remove background pixels to improve data quality.: "Background Noise Removal: Effectively identify and remove background pixels to improve data quality."
- [readme] The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks.: "spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks."
- [readme] First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each pixel: "First two columns: X and Y coordinates of the pixels"
- [results] SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality.: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
