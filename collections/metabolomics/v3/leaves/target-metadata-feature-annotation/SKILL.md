---
name: target-metadata-feature-annotation
description: Use when a targeted metabolomics or proteomics dataset needs to be assembled
  from a metadata (target) CSV and a features CSV into a POMA/MSnbase container so
  downstream filtering, imputation, normalization, and statistics can reference study
  groups.
when_to_use_negative:
- Data is already a processed/annotated container (e.g. an MSnSet or SummarizedExperiment)
  that does not need re-upload — go directly to the preprocessing panel.
- The dataset is raw spectral data (e.g. mzML) rather than a quantified feature matrix;
  spectral processing should be done with RforMassSpectrometry packages first and
  only then migrated to POMA.
- Only one file is available (metadata or features alone) — POMAShiny cannot build
  the annotated container without both CSVs.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: POMAShiny
  role: Web application whose data upload panel ingests the two CSV files and builds
    the annotated MSnbase S4 container
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package providing the S4 data container and back-end functions
    that attach metadata to features
  repo: https://github.com/nutrimetabolomics/POMA
- name: MSnbase
  role: Bioconductor package whose S4 class POMAShiny uses to store the combined metadata
    + feature matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:7019035d3991f3b7701d5e51cd54013944c6085b50f4d5e220094ab3c90b580f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - MSnbase
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# target-metadata-feature-annotation

## Summary

Load a targeted metabolomics/proteomics experiment into POMAShiny as a two-file (target/metadata CSV + features CSV) MSnbase S4 object so that features are annotated with sample metadata before any preprocessing or statistical analysis.

## When to use

Use when a targeted metabolomics or proteomics dataset needs to be assembled from a metadata (target) CSV and a features CSV into a POMA/MSnbase container so downstream filtering, imputation, normalization, and statistics can reference study groups.

## When NOT to use

- Data is already a processed/annotated container (e.g. an MSnSet or SummarizedExperiment) that does not need re-upload — go directly to the preprocessing panel.
- The dataset is raw spectral data (e.g. mzML) rather than a quantified feature matrix; spectral processing should be done with RforMassSpectrometry packages first and only then migrated to POMA.
- Only one file is available (metadata or features alone) — POMAShiny cannot build the annotated container without both CSVs.

## Inputs

- Target (metadata) CSV file with sample identifiers, group labels, and covariates
- Features CSV file with the quantitative feature matrix (e.g. metabolite intensities per sample)
- Optional: a Metabolomics Workbench dataset (e.g. ST000284, ST000336) downloaded as target and features files

## Outputs

- MSnbase S4 object (POMA container) holding both the annotated feature matrix and the sample metadata
- Feature table ready for the missing-value imputation panel (rows = features, columns = samples, with group annotations)

## How to apply

Prepare two comma-separated value files: a target file holding sample metadata (group assignments, covariates) and a features file holding the quantitative matrix (samples as rows or columns, features as the other axis), as required by POMAShiny's data upload panel. Upload both files and assemble them into an MSnbase S4 object through POMA's back-end, which attaches the metadata to the feature matrix so every subsequent panel (missing-value filtering, imputation, normalization, EDA, statistical analysis) operates on a single annotated container. The example datasets ST000284 (three-group targeted metabolomics study) and ST000336 (two-group targeted metabolomics study), both downloadable from Metabolomics Workbench, can be used as templates for the expected file format. Rationale: POMAShiny's workflow is deliberately structured as four sequential panels (data upload, preprocessing, EDA, statistical analysis), and the metadata linkage at upload time is what makes group-specific operations such as removing features with >20% missing values per study group possible.

## Related tools

- **POMAShiny** (Web application whose data upload panel ingests the two CSV files and builds the annotated MSnbase S4 container) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package providing the S4 data container and back-end functions that attach metadata to features) — https://github.com/nutrimetabolomics/POMA
- **MSnbase** (Bioconductor package whose S4 class POMAShiny uses to store the combined metadata + feature matrix)

## Examples

```
shiny::runApp(appDir = "app")
```

## Evaluation signals

- The assembled S4 object's phenoData/metadata dimensions match the row/column count of the target CSV and its sample identifiers match those in the features CSV.
- The feature matrix inside the container equals the values in the uploaded features CSV (no coercion errors or silently dropped rows/columns).
- Group labels from the metadata are correctly attached so that group-aware downstream operations (e.g. >20% missingness filter per study group) receive the correct group assignments.
- Loading the ST000284 or ST000336 example files through the upload panel reproduces the documented three-group or two-group structure without warnings.
- The resulting container passes directly into the missing-value imputation panel without triggering a data-format error.

## Limitations

- POMAShiny requires exactly two CSV inputs (target/metadata and features); datasets supplied in other formats must be converted first.
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions due to substantial breaking changes in the API, so local deployments must pin that dependency.
- No quantitative benchmarking of the upload/annotation step against other tools is reported in the article.

## Evidence

- [results] Two-CSV input requirement: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] MSnbase S4 back-end: [evidence span withheld — anchor status: not_found]
- [results] Four-panel workflow starting with data upload: "1) data upload, 2) preprocessing, 3) exploratory data analysis (EDA) and 4) statistical analysis"
- [results] Example datasets from Metabolomics Workbench: "both freely available at https://www.metabolomicsworkbench.org"
- [results] Group-aware downstream filtering depends on metadata: "remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
- [readme] bs4Dash version pin requirement: "POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API"
