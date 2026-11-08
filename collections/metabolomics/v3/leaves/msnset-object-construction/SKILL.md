---
name: msnset-object-construction
description: Use when raw metabolomics/proteomics inputs are a features intensity
  CSV plus a separate target/metadata CSV and they must be merged into one MSnSet
  object with covariates in pData before analysis.
when_to_use_negative:
- Input is raw spectral data (e.g. mzML) rather than a processed features table —
  POMAShiny expects the two CSV files; spectral processing should be done first, e.g.
  with the RforMassSpectrometry packages, before migrating to POMA.
- The data already exist as an MSnSet or other structured Bioconductor container —
  rebuilding from CSVs risks re-introducing row-order mismatches.
- Target and features files cannot be aligned by sample identifiers (different row/column
  counts or ordering) — the skill requires identical row order and will silently mis-map
  metadata otherwise.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3368
tools:
- name: MSnbase
  role: Provides the MSnSet class that holds the intensity matrix (assay slot) and
    target covariates (pData slot) as POMAShiny's back-end does
- name: Biobase
  role: Bioconductor base dependency installed for the MSnSet/phenodata data structures
    used in the construction step
- name: POMA
  role: Downstream R/Bioconductor package that consumes the MSnSet for preprocessing
    and statistical analysis outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: tidyverse
  role: Used to read the two CSV input files and align target rows with features columns
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:367b505d473c4699856d118d31352ef2390e2ecdfeff2887e0567c957e83be49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMA
  tools:
  - Biobase
  - MSnbase
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# msnset-object-construction

## Summary

Constructs an analysis-ready MSnSet object (MSnbase/Biobase) from the two CSV files POMAShiny requires — a target (metadata) file and a features file — so downstream POMA preprocessing, EDA and statistics can operate on a single container holding both the intensity matrix and sample covariates.

## When to use

Use when raw metabolomics/proteomics inputs are a features intensity CSV plus a separate target/metadata CSV and they must be merged into one MSnSet object with covariates in pData before analysis.

## When NOT to use

- Input is raw spectral data (e.g. mzML) rather than a processed features table — POMAShiny expects the two CSV files; spectral processing should be done first, e.g. with the RforMassSpectrometry packages, before migrating to POMA.
- The data already exist as an MSnSet or other structured Bioconductor container — rebuilding from CSVs risks re-introducing row-order mismatches.
- Target and features files cannot be aligned by sample identifiers (different row/column counts or ordering) — the skill requires identical row order and will silently mis-map metadata otherwise.

## Inputs

- features CSV: feature-by-sample intensity matrix (rows = features, columns = samples)
- target CSV: sample names, group labels, optional study covariates, with rows ordered identically to the features matrix columns

## Outputs

- MSnSet object serialized as msnset_data.rds (assay = feature intensities, pData = target covariates)
- covariate_mapping.tsv: per-sample summary of attached metadata values

## How to apply

Read the features matrix CSV and the target CSV with tidyverse readers, aligning target rows to features columns by sample identifier — the two files must have identical row order. Build the object with MSnbase::MSnSet (Biobase back-end, as POMAShiny does): the feature intensity matrix goes in the assay slot and the target covariates in the pData (phenodata) slot. Record each sample with its attached metadata in a covariate-mapping summary table so the metadata merge can be audited. Serialize the object as msnset_data.rds and the summary as covariate_mapping.tsv. This mirrors POMAShiny's data upload panel, whose output feeds the preprocessing panel (missing-value imputation, normalization with options like autoscaling or log pareto scaling, outlier detection) and then EDA/statistics.

## Related tools

- **MSnbase** (Provides the MSnSet class that holds the intensity matrix (assay slot) and target covariates (pData slot) as POMAShiny's back-end does)
- **Biobase** (Bioconductor base dependency installed for the MSnSet/phenodata data structures used in the construction step)
- **POMA** (Downstream R/Bioconductor package that consumes the MSnSet for preprocessing and statistical analysis outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **tidyverse** (Used to read the two CSV input files and align target rows with features columns)

## Evaluation signals

- msnset_data.rds exists and loads in R as an MSnSet object (inherits from class 'MSnSet').
- pData(msnset) contains every covariate column from the target file, populated for every sample listed in the target.
- Number of samples in pData equals the number of columns in the features assay; row order of the target matches features column order.
- covariate_mapping.tsv has one row per sample and reproduces the target file's metadata values.
- Assay values match the features CSV numerically (no reordering or dropped features introduced at construction).

## Limitations

- The target and features files must have identical row/sample order; misalignment silently attaches the wrong metadata to samples.
- Dependency drift: the original nutrimetabolomics README pins Biobase, POMA and MSnbase, while the newer pcastellanoescuder README lists SummarizedExperiment and POMA instead, so MSnbase-based construction may not match the current upstream code.
- POMAShiny was built against bs4Dash 0.5.0 and is not compatible with newer bs4Dash versions due to breaking API changes, complicating reproduction of the upstream pipeline environment.

## Evidence

- [results] two CSV inputs define the upload format: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] backend packages include MSnbase: [evidence span withheld — anchor status: not_found]
- [readme] Bioconductor dependencies pinned in README: "pk2 <- c('Biobase', 'POMA', 'MSnbase')"
- [intro] migration path from spectral processing to POMA: "users can perform the spectral data processing and other routine MS workflow operations using the RforMassSpectrometry complementary packages, and then easily migrate to POMA"
