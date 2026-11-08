---
name: metabolomics-data-ingestion
description: Use when two CSVs — a target/metadata file (sample names, group labels,
  optional covariates) and a features file — must be merged into an analysis-ready
  MSnSet object for metabolomics/proteomics analysis.
when_to_use_negative:
- The data already exists as an MSnSet or SummarizedExperiment object and only needs
  downstream pre-processing, EDA or statistical analysis panels.
- Input is raw spectral data (unprocessed MS runs) rather than a tabular features
  file — the article recommends spectral processing via RforMassSpectrometry packages
  before migrating to POMA.
- Only a single combined data matrix is available with no separate target/metadata
  file, since group labels and covariates must come from the target CSV.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3828
- http://edamontology.org/topic_3520
tools:
- name: POMAShiny
  role: Reference workflow whose data-upload panel defines the two-CSV input contract
    and internal MSnSet conversion
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: MSnbase
  role: Provides the MSnSet class used as the internal analysis-ready data container
- name: Biobase
  role: Bioconductor base package underpinning the MSnSet assay/pData slots
- name: POMA
  role: R/Bioconductor back-end package that consumes the constructed MSnSet for downstream
    analysis
  repo: https://github.com/nutrimetabolomics/POMA
- name: tidyverse
  role: Used by the POMAShiny back-end to read the target and features CSV files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:3acd92f1a958f0e23bfbf75f898e081fbde49f46bb54f52c8fa7b1d72944e193
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
  - http://edamontology.org/topic_3828
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# metabolomics-data-ingestion

## Summary

Ingests POMAShiny-style two-file input (a target/metadata CSV and a features intensity CSV) and converts them into an MSnSet object with sample covariates stored in the pData slot, mirroring POMAShiny's data-upload panel.

## When to use

Use when two CSVs — a target/metadata file (sample names, group labels, optional covariates) and a features file — must be merged into an analysis-ready MSnSet object for metabolomics/proteomics analysis.

## When NOT to use

- The data already exists as an MSnSet or SummarizedExperiment object and only needs downstream pre-processing, EDA or statistical analysis panels.
- Input is raw spectral data (unprocessed MS runs) rather than a tabular features file — the article recommends spectral processing via RforMassSpectrometry packages before migrating to POMA.
- Only a single combined data matrix is available with no separate target/metadata file, since group labels and covariates must come from the target CSV.

## Inputs

- target CSV file (sample names, group labels, optional covariates)
- features CSV file (feature intensity matrix, samples as columns)

## Outputs

- msnset_data.rds — serialized MSnSet object with intensities in the assay slot and covariates in pData
- covariate_mapping.tsv — per-sample table of attached metadata values

## How to apply

Read the features matrix CSV and the target (metadata) CSV with tidyverse readers, aligning rows of the target with columns of the features matrix by sample identifiers — POMAShiny requires the two files to be in identical row order. Construct the object with MSnbase::MSnSet (Biobase back-end, as POMAShiny does via the POMA/MSnbase back-end), placing the feature intensity matrix in the assay slot and the target covariates in the pData (phenodata) slot. Build a covariate-mapping summary table recording each sample with its attached metadata values. Serialize the MSnSet to msnset_data.rds and write the summary to covariate_mapping.tsv. Validate by reloading the RDS, confirming it is an MSnSet, and checking that pData contains every target covariate field for every sample listed in the target file.

## Related tools

- **POMAShiny** (Reference workflow whose data-upload panel defines the two-CSV input contract and internal MSnSet conversion) — https://github.com/nutrimetabolomics/POMAShiny
- **MSnbase** (Provides the MSnSet class used as the internal analysis-ready data container)
- **Biobase** (Bioconductor base package underpinning the MSnSet assay/pData slots)
- **POMA** (R/Bioconductor back-end package that consumes the constructed MSnSet for downstream analysis) — https://github.com/nutrimetabolomics/POMA
- **tidyverse** (Used by the POMAShiny back-end to read the target and features CSV files)

## Examples

```
msnset <- MSnbase::MSnSet(assayData = as.matrix(features), pData = Biobase::AnnotatedDataFrame(target)); saveRDS(msnset, "msnset_data.rds")
```

## Evaluation signals

- msnset_data.rds exists and loads as a valid MSnSet object
- pData contains every covariate field from the target CSV for every sample listed in the target file
- Sample identifiers in the target file align one-to-one, in identical row order, with columns of the features matrix
- Assay slot dimensions match the features CSV (features x samples)
- covariate_mapping.tsv contains one row per sample with its metadata values

## Limitations

- The target and features CSVs must have identical row/sample order; a mismatch will misassign metadata to samples unless explicitly aligned by sample identifiers.
- Input must already be a processed tabular intensity matrix — spectral processing and other routine MS operations are expected to be done beforehand with RforMassSpectrometry packages.
- Newer POMAShiny versions depend on SummarizedExperiment rather than MSnbase (README dependency lists differ between releases), so the container class may vary across versions.

## Evidence

- [results] POMAShiny requires two CSV input files: a target (metadata) file and a features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] POMAShiny's back-end structure uses the POMA, MSnbase and tidyverse family R packages: [evidence span withheld — anchor status: not_found]
- [readme] POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels.: "POMAShiny's workflow is structured
in four sequential and well-defined panels: 1) data upload, 2)
pre-processing, 3) EDA and 4) statistical analysis panels."
