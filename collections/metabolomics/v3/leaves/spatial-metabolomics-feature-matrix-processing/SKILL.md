---
name: spatial-metabolomics-feature-matrix-processing
description: Use when the input is an IMS feature matrix CSV (pixel X/Y coordinates
  plus ion-intensity columns) that needs QC, isotope/adduct pair identification, metabolite
  annotation against a custom LC-MS/MS library, or ROI-based differential testing.
when_to_use_negative:
- Input is raw imaging-MS data that has not yet been peak-picked into a pixel × m/z
  feature matrix
- The data are bulk (non-spatial) metabolomics measurements lacking pixel X/Y coordinates
- The matrix is already annotated and the goal is only downstream pathway visualization
  rather than SMAnalyst QC/identification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMAnalyst
  role: 'Web/R platform that executes the whole pipeline: upload, QC, isotope/adduct
    identification, annotation, spatial pattern discovery, differential analysis'
  repo: https://github.com/mzlab-research/SManalyst
- name: R (>= v4.1.0)
  role: Runtime environment with the required CRAN/Bioconductor packages (MetaboCoreUtils,
    spdep, enviPat, msentropy, Seurat, etc.)
- name: MetaboCoreUtils
  role: isotopologues for isotope-pair screening within ppm tolerance and mz2mass
    for neutral-mass computation over the adduct list
- name: spdep
  role: moran_bv spatial correlation validation of candidate isotope and adduct pairs
    across pixels (default threshold 0.5)
- name: enviPat
  role: check_ded to eliminate impossible adduct assignments; isopattern for theoretical
    isotopic-pattern generation
- name: msentropy
  role: msentropy_similarity quantifies similarity of observed vs theoretical isotopic
    peak distributions
- name: Seurat
  role: FindMarkers Wilcoxon rank-sum differential testing between user-defined pixel
    groups with Benjamini–Hochberg FDR
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_007
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:8f8f32a9a32a082536a5a7e1c6c069a6a332f90a9a377fc2b3dfe3c22a393da1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - R (>= v4.1.0)
  - R (>=)
  - SMAnalyst
  - Seurat
  - enviPat
  - msentropy
  - spdep
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# spatial-metabolomics-feature-matrix-processing

## Summary

Load, quality-control, and analyze an imaging-MS pixel × m/z feature matrix (e.g., the 14,260-pixel × 3,044-ion mouse brain AFAD-ESI(+) dataset) in SMAnalyst, spanning isotope/adduct identification, LC-MS/MS-library annotation, and regional differential analysis in R.

## When to use

Use when the input is an IMS feature matrix CSV (pixel X/Y coordinates plus ion-intensity columns) that needs QC, isotope/adduct pair identification, metabolite annotation against a custom LC-MS/MS library, or ROI-based differential testing.

## When NOT to use

- Input is raw imaging-MS data that has not yet been peak-picked into a pixel × m/z feature matrix
- The data are bulk (non-spatial) metabolomics measurements lacking pixel X/Y coordinates
- The matrix is already annotated and the goal is only downstream pathway visualization rather than SMAnalyst QC/identification

## Inputs

- Feature matrix CSV: columns 1–2 = pixel X/Y coordinates, subsequent columns = ion intensities per m/z, one row per spatial pixel
- Custom LC-MS/MS library (mouse brain, 1269 metabolites, identification levels 1–3) or built-in HMDB/KEGG/LIPIDMAPS database
- Pixel cluster assignments (e.g., 25 UMAP-kmeans clusters) for defining ROI groups

## Outputs

- isotope_adduct_identification.csv: isotope/adduct pair assignments and fractions of total ion peaks
- annotation_results.csv: per-ion annotation table with multiplicity of annotation results
- differential_metabolites_MB_vs_HIP.csv: filtered differential metabolites with fold change, significance, and annotation status
- Ion distribution images and QC metric dashboards (CSV tables, PNG images)

## How to apply

Import the feature matrix into SMAnalyst's R (≥ 4.1.0) pipeline, where columns 1–2 hold pixel X/Y coordinates and remaining columns hold ion intensities at each m/z (Dataset 1: mouse brain coronal section, AFAD-ESI(+) mode, 100 μm resolution, 14,260 pixels × 3,044 ions, NGDC OMIX OMIX009541). Run QC by selecting tissue/background regions with the interactive lasso tool and filtering noise ions at the default threshold score ≥ 30. For isotope/adduct identification, screen peak pairs with the MetaboCoreUtils isotopologues function within a user-specified ppm tolerance, validate each pair spatially with the spdep moran_bv function (default correlation threshold 0.5), then screen monoisotopic ions for [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, and [M+H-H2O]+ adducts. For annotation, remove non-monoisotopic peaks, compute neutral masses (directly for clear adducts, else via MetaboCoreUtils mz2mass over the adduct list), reject impossible adducts with enviPat check_ded (e.g., no oxygen means no [M+H-H2O]+), score candidates by mass accuracy, enviPat isopattern/msentropy pattern similarity, and adduct evidence against the 1269-metabolite mouse brain LC-MS/MS library. For regional differential analysis, assign pixels to groups from UMAP-kmeans clusters and run Seurat FindMarkers (Wilcoxon rank-sum) with Benjamini–Hochberg FDR, retaining FC > 2 or < 0.5 and adjusted p < 0.01.

## Related tools

- **SMAnalyst** (Web/R platform that executes the whole pipeline: upload, QC, isotope/adduct identification, annotation, spatial pattern discovery, differential analysis) — https://github.com/mzlab-research/SManalyst
- **R (>= v4.1.0)** (Runtime environment with the required CRAN/Bioconductor packages (MetaboCoreUtils, spdep, enviPat, msentropy, Seurat, etc.))
- **MetaboCoreUtils** (isotopologues for isotope-pair screening within ppm tolerance and mz2mass for neutral-mass computation over the adduct list)
- **spdep** (moran_bv spatial correlation validation of candidate isotope and adduct pairs across pixels (default threshold 0.5))
- **enviPat** (check_ded to eliminate impossible adduct assignments; isopattern for theoretical isotopic-pattern generation)
- **msentropy** (msentropy_similarity quantifies similarity of observed vs theoretical isotopic peak distributions)
- **Seurat** (FindMarkers Wilcoxon rank-sum differential testing between user-defined pixel groups with Benjamini–Hochberg FDR)

## Examples

```
Rscript -e "df <- read.csv('feature_matrix.csv'); library(MetaboCoreUtils); iso_pairs <- isotopologues(df[, -c(1,2)])"
```

## Evaluation signals

- On Dataset 1, the isotope/adduct workflow must yield 5.92% of total ion peaks classified as isotopic and 4.19% as adduct ions
- Annotation must reproduce 669 identified ions, split as 374 single, 148 double, and 147 multiple annotations (secondary check: 91% overlap of single annotations with MSIannotator)
- Differential table must contain exactly 76 metabolites upregulated in MB (clusters 6/1) and 82 in HIP (clusters 23/12) under FC > 2 or < 0.5 and FDR < 0.01
- Output CSVs must contain per-ion pair assignments, annotation multiplicity, and differential statistics with the expected column schema
- Feature matrix must load as 14,260 rows (pixels) × 3,044 ion columns for Dataset 1

## Limitations

- Mass-error tolerance (ppm) and adduct list are user-specified, so results depend on those settings; spatial-correlation thresholds affect which pairs are retained
- Large-scale matrices (>14,000 pixels, >3,000 ions) are the tested upper bound; performance beyond that is not established
- Impossible adducts (e.g., [M+H-H2O]+ for oxygen-free molecules) must be caught by enviPat check_ded; skipping this step produces false annotations

## Evidence

- [readme] Feature matrix format: "SManalyst requires a **feature matrix** in CSV format"
- [readme] Dataset dimensions: "Content: 14,260 pixels × 3,044 ions"
- [intro] Platform scope: "SManalyst is an innovative open-source platform for comprehensive analysis of spatial metabolomics data."
- [readme] Isotope/adduct module: "Isotope/adduct identification with spatial correlation validation"
- [abstract] Scalability: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [readme] Data availability: "Raw data: [NGDC OMIX Repository](https://ngdc.cncb.ac.cn/omix) ID: OMIX009541"
