---
name: sample-metadata-attachment
description: Use when data arrive as two separate CSVs (a features matrix and a target/metadata
  file) that must be combined into one covariate-annotated analysis object (e.g. MSnSet)
  before preprocessing or statistics.
when_to_use_negative:
- Data are already held in an annotated container such as an MSnSet, ExpressionSet
  or SummarizedExperiment with populated pData — re-attachment would duplicate metadata.
- The task is a later POMAShiny stage (missing-value imputation, normalization, outlier
  removal, EDA or statistical analysis) rather than the data-upload/annotation step.
- Input is raw spectral files (e.g. mzML) needing signal processing first; the article
  routes those through RforMassSpectrometry packages before migrating to POMA.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0121
tools:
- name: POMAShiny
  role: Reference implementation whose data-upload panel performs the two-file target+features
    ingestion into an analysis-ready object
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: MSnbase
  role: Provides the MSnSet container class used to hold the feature matrix and sample
    covariates
- name: Biobase
  role: Back-end infrastructure ( pData / AnnotatedDataFrame) for attaching phenotype
    metadata to the assay data
- name: POMA
  role: R/Bioconductor package through which POMAShiny's back-end operates on the
    assembled MSnSet
  repo: https://github.com/nutrimetabolomics/POMA
- name: tidyverse
  role: Used by POMAShiny's back-end to read the target and features CSV files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:1a80f89c016eb24d8e91a8d3b641de2d0fca01faa59c4ede2c0aafd2484f221d
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
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# sample-metadata-attachment

## Summary

Merges a features intensity matrix with a separate target (metadata) CSV — sample names, group labels and optional covariates — into a single annotated MSnSet object, as POMAShiny does with its MSnbase/Biobase back-end, so downstream preprocessing and statistics operate on one covariate-aware container.

## When to use

Use when data arrive as two separate CSVs (a features matrix and a target/metadata file) that must be combined into one covariate-annotated analysis object (e.g. MSnSet) before preprocessing or statistics.

## When NOT to use

- Data are already held in an annotated container such as an MSnSet, ExpressionSet or SummarizedExperiment with populated pData — re-attachment would duplicate metadata.
- The task is a later POMAShiny stage (missing-value imputation, normalization, outlier removal, EDA or statistical analysis) rather than the data-upload/annotation step.
- Input is raw spectral files (e.g. mzML) needing signal processing first; the article routes those through RforMassSpectrometry packages before migrating to POMA.

## Inputs

- features CSV: sample-by-feature intensity matrix (columns = samples, rows = features)
- target CSV: sample names, group labels, and optional covariate columns
- expectation that target rows and features columns are in identical order

## Outputs

- msnset_data.rds: serialized MSnSet object with the intensity matrix in the assay slot and covariates in pData
- covariate_mapping.tsv: per-sample summary table of attached metadata values

## How to apply

Load the features matrix CSV and the target (metadata) CSV with tidyverse readers; the target carries sample names, group labels and optional covariate columns. Require identical row/sample ordering between the two files, then construct the object with MSnbase::MSnSet (Biobase), placing the feature intensity matrix in the assay slot and the covariates in the pData (phenodata) slot, exactly as POMAShiny's data-upload step does via its POMA/MSnbase back-end. Build a covariate-mapping summary table recording each sample with its attached metadata values. Serialize the MSnSet to msnset_data.rds and write the summary to covariate_mapping.tsv. Validate that the RDS reloads as an MSnSet and that pData contains every target covariate field for every sample listed in the target file before proceeding to the preprocessing panel (imputation, normalization, outlier detection).

## Related tools

- **POMAShiny** (Reference implementation whose data-upload panel performs the two-file target+features ingestion into an analysis-ready object) — https://github.com/nutrimetabolomics/POMAShiny
- **MSnbase** (Provides the MSnSet container class used to hold the feature matrix and sample covariates)
- **Biobase** (Back-end infrastructure ( pData / AnnotatedDataFrame) for attaching phenotype metadata to the assay data)
- **POMA** (R/Bioconductor package through which POMAShiny's back-end operates on the assembled MSnSet) — https://github.com/nutrimetabolomics/POMA
- **tidyverse** (Used by POMAShiny's back-end to read the target and features CSV files)

## Examples

```
BiocManager::install(c('Biobase','POMA','MSnbase')); msnset <- MSnbase::MSnSet(exprs = as.matrix(features), pData = Biobase::AnnotatedDataFrame(target)); saveRDS(msnset, 'msnset_data.rds')
```

## Evaluation signals

- msnset_data.rds exists and loads with class MSnSet (MSnbase/Biobase).
- pData of the loaded MSnSet contains every covariate column present in the target CSV, populated for every sample listed in the target file.
- Number of samples in the assay dimension matches the number of rows in the target file, and sample identifiers align in identical order across both input files.
- covariate_mapping.tsv has one row per target sample with its metadata values recorded.
- Group labels from the target are retrievable from the object so downstream POMA preprocessing (imputation, normalization, outlier detection) can use them.

## Limitations

- The target and features CSVs must have identical row/sample order; misalignment silently attaches wrong covariates.
- Only comma-separated CSV input files are accepted (the target/metadata file and the features file); no other upload formats are supported at the data-upload step.
- This skill covers ingestion/annotation only — missing-value filtering (default 20% per study group), imputation and normalization are separate downstream steps.

## Evidence

- [results] POMAShiny requires two CSV input files: a target (metadata) file and a features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] POMAShiny's back-end structure uses the POMA [13], MSnbase [14] and tidyverse family [15] R packages: [evidence span withheld — anchor status: not_found]
- [readme] POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels.: "POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2)
pre-processing, 3) EDA and 4) statistical analysis panels."
- [intro] users can perform the spectral data processing and other routine MS workflow operations using the RforMassSpectrometry complementary packages, and then easily migrate to POMA: "users can perform the spectral data processing and other routine MS workflow operations using the RforMassSpectrometry complementary packages, and then easily migrate to POMA"
