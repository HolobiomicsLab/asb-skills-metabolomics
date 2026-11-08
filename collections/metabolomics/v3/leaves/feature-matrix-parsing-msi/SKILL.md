---
name: feature-matrix-parsing-msi
description: Use when input is an MSI feature matrix (e.g., 14,260 pixels x 3,044
  ion peaks from AFAD-ESI positive mode) that must be reshaped so each row is one
  spatial pixel with X/Y coordinate columns before SMAnalyst QC or clustering.
when_to_use_negative:
- Input is raw IMS spectra (e.g., imzML) or peak-picked spectra lists rather than
  a pixel-by-ion feature table — peak picking/feature extraction must happen first
- Data lacks pixel coordinate columns or m/z headers, since SMAnalyst explicitly requires
  columns 1-2 as X/Y and headers as m/z values
- Goal is single-bulk metabolomics (no spatial pixel dimension) — spatial co-expression
  clustering is meaningless without per-pixel rows
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Target platform whose required feature-matrix format (X, Y, then m/z intensity
    columns) dictates the parsing schema; matrix is imported via the Upload tab or
    Tutorial Panel example data
  repo: https://github.com/mzlab-research/SManalyst
- name: R (>= v4.1.0)
  role: Runtime environment for loading and reshaping the matrix with readr, data.table,
    and tidyverse
- name: tidyverse
  role: Used with data.table/readr for reading and tidying the pixel-by-ion matrix
    prior to SMAnalyst QC and SpaGene clustering
- name: SpaGene
  role: 'Downstream consumer: clusters metabolic ions by spatial co-expression once
    the matrix is correctly structured'
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:0ae2df904f4d2dd45ddad6d75c9c1cc4a34cbf6c50822a8a77af96031e58b409
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>= v4.1.0)
  - R (>=)
  - SMAnalyst
  - SpaGene
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# feature-matrix-parsing-msi

## Summary

Parses and structures spatial metabolomics feature matrices (pixel-by-ion intensity tables) into the CSV layout SMAnalyst requires, where columns 1–2 hold X/Y pixel coordinates and remaining columns hold m/z ion intensities, enabling all downstream QC, annotation, and spatial pattern analysis.

## When to use

Use when input is an MSI feature matrix (e.g., 14,260 pixels x 3,044 ion peaks from AFAD-ESI positive mode) that must be reshaped so each row is one spatial pixel with leading X/Y coordinate columns before SMAnalyst QC or clustering.

## When NOT to use

- Input is raw IMS spectra (e.g., imzML) or peak-picked spectra lists rather than a pixel-by-ion feature table — peak picking/feature extraction must happen first
- Data lacks pixel coordinate columns or m/z headers, since SMAnalyst explicitly requires columns 1-2 as X/Y and headers as m/z values
- Goal is single-bulk metabolomics (no spatial pixel dimension) — spatial co-expression clustering is meaningless without per-pixel rows

## Inputs

- CSV feature matrix of spatial metabolomics pixel intensities (columns 1-2: X,Y coordinates; subsequent columns: ion intensities per m/z)
- Raw dataset from NGDC OMIX accession OMIX009541 (mouse brain coronal section, AFAD-ESI (+), 100 um resolution)

## Outputs

- Structured R data frame / CSV with one row per spatial pixel, X/Y coordinate columns, and named m/z intensity columns
- Input-ready matrix for SMAnalyst upload, QC metrics (QC1-QC4), and SpaGene spatial co-expression clustering

## How to apply

Load the feature matrix in R (>= v4.1.0) with readr/data.table, treating the first two columns as X/Y pixel coordinates and all remaining columns as m/z ion intensities, one row per spatial pixel. Verify dimensions against the expected scale (the SMAnalyst mouse brain test set is 14,260 pixels x 3,044 ions acquired at 100 um AFAD-ESI (+) resolution, NGDC OMIX OMIX009541). Ensure the CSV has an explicit header of m/z values (e.g., m/z_100.002, m/z_101.005) and that row order matches the spatial raster; zero entries represent undetected ions and feed later missing-rate QC. Once structured, the matrix can be imported via the SMAnalyst 'Upload' tab or loaded locally for SpaGene co-expression clustering and pixel-level QC (QC1–QC4).

## Related tools

- **SMAnalyst** (Target platform whose required feature-matrix format (X, Y, then m/z intensity columns) dictates the parsing schema; matrix is imported via the Upload tab or Tutorial Panel example data) — https://github.com/mzlab-research/SManalyst
- **R (>= v4.1.0)** (Runtime environment for loading and reshaping the matrix with readr, data.table, and tidyverse)
- **tidyverse** (Used with data.table/readr for reading and tidying the pixel-by-ion matrix prior to SMAnalyst QC and SpaGene clustering)
- **SpaGene** (Downstream consumer: clusters metabolic ions by spatial co-expression once the matrix is correctly structured)

## Examples

```
df <- read.csv("OMIX009541_feature_matrix.csv", check.names = FALSE); coords <- df[, 1:2]; X <- as.matrix(df[, -(1:2)])
```

## Evaluation signals

- Parsed table has exactly 2 leading coordinate columns (X, Y) and one column per m/z value, with m/z values in column headers (e.g., m/z_100.002)
- Row count equals the number of spatial pixels (e.g., 14,260 for the mouse brain test set) and each row corresponds to one pixel
- Column count matches the ion peak count (e.g., 3,044 unique ion peaks) after optional noise filtering at default score >= 30
- Zero-valued entries are preserved as undetected ions so pixel missing ratio and ion missing ratio can be computed in QC3
- Matrix loads in R without coordinate columns leaking into intensity computations, and total per-pixel intensity distributions are plottable for tissue/background delineation

## Limitations

- SMAnalyst accepts only CSV feature matrices in the fixed layout (X, Y, then ion intensities); other formats require prior conversion
- Very large matrices are handled (tested >14,000 pixels, >3,000 ion peaks), but scale beyond this is untested in the source
- Pixel raster ordering is assumed intact; the parser does not validate spatial adjacency, which affects downstream background-region correlation QC

## Evidence

- [readme] SManalyst requires a feature matrix in CSV format: "SManalyst requires a **feature matrix** in CSV format:

*   **Columns 1-2**: Pixel coordinates (X, Y)
*   **Subsequent columns**: Ion intensities at each m/z value"
- [readme] Test dataset mouse brain coronal section acquisition: "Test dataset: **Mouse brain coronal section**

*   Acquisition: AFAD-ESI (+) mode, 100μm resolution
*   Content: 14,260 pixels × 3,044 ions"
- [abstract] SMAnalyst handles large-scale data: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [intro] SMAnalyst integrates quality control and spatial pattern discovery: "SManalyst is an innovative open-source platform for comprehensive analysis of spatial metabolomics data. It integrates quality control, metabolite annotation, spatial pattern discovery, and"
