---
name: spatial-metabolomics-peak-table-processing
description: Use when a spatial metabolomics peak table matrix (X/Y pixel coordinates
  + ion intensity columns) contains background pixels that must be removed prior to
  clustering or downstream spatial analysis.
when_to_use_negative:
- Input is already a preprocessed/filtered pixel table with background pixels removed
  (no lasso tissue/background delineation available).
- Goal is ion-level noise filtering rather than pixel-level background removal (use
  SMQVP's noise ion filtering module instead).
- Data is not spatial metabolomics pixel-positioned peak table data (e.g. bulk metabolomics
  intensity tables without X/Y coordinates).
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: 'Web application providing background noise removal: lasso delineation of
    tissue/background regions and quality visualization points used to derive the
    tissue-enriched ion set and total-intensity threshold.'
  repo: github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP is run and in which the peak
    table loading, Wilcoxon rank-sum testing, and pixel filtering are scripted.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:69e393c1aad6e9242ab7cda7a1dd6f56440da4dc0da0df5e4c85c0caa6e86d23
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spatial-metabolomics-peak-table-processing

## Summary

Reconstructs SMQVP's background pixel removal (Process 2) for spatial metabolomics: tissue-enriched ions are identified by Wilcoxon rank-sum test, summed into a total-intensity image, and pixels below the 107.37 threshold are removed from the AFADESI mouse brain peak table.

## When to use

Use when a spatial metabolomics peak table matrix (X/Y pixel coordinates + ion intensity columns) contains background pixels that must be removed prior to clustering or downstream spatial analysis.

## When NOT to use

- Input is already a preprocessed/filtered pixel table with background pixels removed (no lasso tissue/background delineation available).
- Goal is ion-level noise filtering rather than pixel-level background removal (use SMQVP's noise ion filtering module instead).
- Data is not spatial metabolomics pixel-positioned peak table data (e.g. bulk metabolomics intensity tables without X/Y coordinates).

## Inputs

- Peak table matrix CSV from OMIX009541 (53,812 pixels x 2,654 ion peaks; first two columns = X, Y pixel coordinates, remaining columns = ion intensities)
- User-delineated tissue and background region annotations (lasso-defined regions from SMQVP interaction image)

## Outputs

- background_removed_peak_table.csv — filtered peak table retaining only tissue pixels (tissue-enriched total intensity >= 107.37) with original schema (X, Y, ion intensity columns)
- Total-intensity image of tissue-enriched ions (cf. Supplementary Figure S3B)
- Segmentation of tissue vs background pixels (cf. Supplementary Figure S3D)

## How to apply

Load the AFADESI mouse brain peak table matrix (53,812 pixels x 2,654 ion peaks, first two columns X/Y) from OMIX009541 with pandas in R. Using user-delineated lasso tissue and background regions, run a per-ion Wilcoxon rank-sum test between tissue and background pixel groups, classifying ions with fold change > 1 and p-value < 0.05 as tissue-enriched. Sum tissue-enriched ion intensities per pixel to build the total-intensity image (cf. Supplementary Figure S3B), then apply the 107.37 intensity threshold, retaining only pixels at or above it (cf. Supplementary Figure S3D). Rationale: SMQVP's background noise removal removes background pixels whose tissue-enriched total signal falls below the paper-derived cutoff, improving clustering fidelity to tissue morphology.

## Related tools

- **SMQVP** (Web application providing background noise removal: lasso delineation of tissue/background regions and quality visualization points used to derive the tissue-enriched ion set and total-intensity threshold.) — github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP is run and in which the peak table loading, Wilcoxon rank-sum testing, and pixel filtering are scripted.)

## Evaluation signals

- Output file background_removed_peak_table.csv exists and retains the peak-table schema (X, Y, then ion intensity columns).
- Every retained pixel has tissue-enriched total intensity >= 107.37; all removed pixels are below the threshold.
- Removed pixels spatially correspond to background areas visible in the paper's segmentation (Supplementary Figure S3D).
- Constructed total-intensity image reproduces Supplementary Figure S3B when visualized.
- Downstream clustering on the filtered table aligns better with tissue morphology than clustering on unprocessed data.

## Limitations

- No software version specified for SMQVP and no license information stated in the repository.
- No changelog available for the repository, complicating reproduction of version-specific behavior.
- Requires user-delineated lasso tissue/background regions; the threshold 107.37 is dataset-specific (AFADESI mouse brain) and should be re-derived for other datasets.

## Evidence

- [readme] Background Noise Removal: Effectively identify and remove background pixels to improve data quality.: "Background Noise Removal: Effectively identify and remove background pixels to improve data quality."
- [results] SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality.: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
- [intro] Use the lasso tool to delineate background and tissue regions on the interaction image: "Use the lasso tool to delineate background and tissue regions on the interaction image"
- [abstract] showing that the tool successfully identified and removed noise signals: "showing that the tool successfully identified and removed noise signals"
- [readme] This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541: "This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541"
