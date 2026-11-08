---
name: omics-covariate-mapping
description: Use when you have two CSVs — a features matrix and a target/metadata
  file with sample names, group labels and optional covariates — that must be merged
  into a single analysis-ready S4 object with intact sample-to-covariate mapping.
when_to_use_negative:
- Input is already an MSnSet, ExpressionSet or SummarizedExperiment — remapping covariates
  would risk sample misalignment
- Data arrives as raw spectral files (mzML/vendor formats) — this skill covers tabular
  CSV ingestion only, not spectral processing; the paper routes those via RforMassSpectrometry
  packages first
- The features file and target file cannot be row/column aligned by shared sample
  identifiers — silently reordering would produce wrong covariate assignments
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3170
tools:
- name: MSnbase
  role: Provides the MSnSet container class that holds the feature matrix and pData
    covariates
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: Biobase
  role: Underlying Bioconductor infrastructure (pData/ExpressionSet basis) for the
    phenodata slot
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end that consumes the covariate-annotated object for subsequent
    preprocessing and statistics
  repo: https://github.com/nutrimetabolomics/POMA
- name: POMAShiny
  role: Reference implementation whose data-upload panel performs the two-CSV ingestion
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: tidyverse
  role: CSV readers used to load the target and features files before object construction
  repo: https://github.com/nutrimetabolomics/POMAShiny
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:528a9d7680f6f4e47b1372ea48160395853c1d04373995ac10b5ae04136b09bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - Biobase
  - MSnbase
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 25
---

# omics-covariate-mapping

## Summary

Reconstructs the POMAShiny two-file ingestion pattern: a target (metadata) CSV and a features CSV are combined into an MSnSet object (MSnbase/Biobase) with the feature intensity matrix in the assay slot and covariates in pData, then serialized with a covariate-mapping summary.

## When to use

Use when you have two CSVs — a features matrix and a target/metadata file with sample names, group labels and optional covariates — that must be merged into a single analysis-ready S4 object with intact sample-to-covariate mapping.

## When NOT to use

- Input is already an MSnSet, ExpressionSet or SummarizedExperiment — remapping covariates would risk sample misalignment
- Data arrives as raw spectral files (mzML/vendor formats) — this skill covers tabular CSV ingestion only, not spectral processing; the paper routes those via RforMassSpectrometry packages first
- The features file and target file cannot be row/column aligned by shared sample identifiers — silently reordering would produce wrong covariate assignments

## Inputs

- features CSV: feature-by-sample intensity matrix (metabolomics or proteomics)
- target/metadata CSV: sample names, group labels, optional covariates
- sample identifier column used to align target rows with feature matrix columns

## Outputs

- msnset_data.rds: serialized MSnSet with assay = features, pData = covariates
- covariate_mapping.tsv: per-sample table of attached metadata values

## How to apply

Load the features matrix and the target file with tidyverse readers; POMAShiny requires exactly two comma-separated value (CSV) inputs, the target (metadata) file and the features file. Verify that sample identifiers in the target align with the columns of the features matrix and that row order matches, since the ingestion assumes identical ordering. Construct the container with MSnbase::MSnSet, placing the feature intensity matrix in the assay/exprs slot and the target covariates in the pData (phenodata) slot, mirroring POMAShiny's POMA/MSnbase back-end. Build a covariate-mapping summary table recording each sample with its attached metadata values. Serialize the MSnSet to msnset_data.rds and write the summary to covariate_mapping.tsv so downstream POMA preprocessing (imputation, normalization, outlier removal) operates on a covariate-aware object.

## Related tools

- **MSnbase** (Provides the MSnSet container class that holds the feature matrix and pData covariates) — https://github.com/nutrimetabolomics/POMAShiny
- **Biobase** (Underlying Bioconductor infrastructure (pData/ExpressionSet basis) for the phenodata slot) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end that consumes the covariate-annotated object for subsequent preprocessing and statistics) — https://github.com/nutrimetabolomics/POMA
- **POMAShiny** (Reference implementation whose data-upload panel performs the two-CSV ingestion) — https://github.com/nutrimetabolomics/POMAShiny
- **tidyverse** (CSV readers used to load the target and features files before object construction) — https://github.com/nutrimetabolomics/POMAShiny

## Examples

```
msnset <- MSnbase::MSnSet(exprs = as.matrix(features)); pData(msnset) <- DataFrame(target, row.names = rownames(target))
```

## Evaluation signals

- msnset_data.rds exists and loads as an MSnSet object
- pData of the loaded MSnSet contains every covariate field from the target file for every listed sample
- Number of rows in covariate_mapping.tsv equals the number of rows in the target CSV
- Sample identifiers in pData match the column names of the assay/exprs matrix one-to-one
- No samples dropped: object dimensions equal the input features matrix dimensions (features x samples)

## Limitations

- The target and features CSVs must have identical row/sample ordering; misalignment silently corrupts covariate mapping
- POMAShiny was built against bs4Dash 0.5.0 and is not compatible with newer versions, affecting local reproduction of the ingestion panel
- Newer POMAShiny READMEs depend on SummarizedExperiment instead of Biobase/MSnbase, so the container class may differ across versions; no software version number is stated in the paper

## Evidence

- [results] results: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] intro: [evidence span withheld — anchor status: not_found]
- [readme] readme: "pk2 <- c('Biobase', 'POMA', 'MSnbase')"
- [readme] readme-panels: "POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels."
