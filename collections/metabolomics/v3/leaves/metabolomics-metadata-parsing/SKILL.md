---
name: metabolomics-metadata-parsing
description: Use when raw inputs are a metabolomics study's processed data matrix
  plus repository metadata (MetaboLights or Metabolomics Workbench accession) that
  must be converted into a SummarizedExperiment before running maplet pipelines.
when_to_use_negative:
- The data are already stored in a SummarizedExperiment container with populated rowData/colData
  — maplet pipelines can proceed directly without re-loading from the repository.
- The task is a downstream pipeline step (annotation, statistical analysis, visualization,
  reporting) rather than ingesting repository data into maplet's container.
- Only a single committed, untagged version of maplet is available — commits without
  a release tag are not guaranteed to be stable, so repository loader behavior may
  not match v1.2.1.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0153
tools:
- name: maplet
  role: Provides the specialized loading functions that fetch/parse Metabolomics Workbench
    and MetaboLights datasets and build the SummarizedExperiment container
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Central container structure receiving the assay matrix and rowData/colData
    annotations
- name: devtools
  role: 'Used to install the pinned maplet release: devtools::install_github(repo="krumsieklab/maplet@v1.2.1",
    subdir="maplet")'
- name: magrittr
  role: Supplies the %>% pipe operator used to connect maplet pipeline steps without
    temporary variables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:4711745daf9531f59420875b664512b23c9618dc8bdfac101f75b6b62f6bf96a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/krumsieklab/maplet
  repo_url: https://github.com/krumsieklab/maplet
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - magrittr
  - maplet
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-metadata-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse metabolomics repository metadata (from MetaboLights or Metabolomics Workbench) and assemble a processed intensity matrix together with structured sample and metabolite annotations into a SummarizedExperiment container, the central data structure maplet pipelines operate on.

## When to use

Use when raw inputs are a metabolomics study's processed data matrix plus repository metadata (MetaboLights or Metabolomics Workbench accession) that must be converted into a SummarizedExperiment before running maplet pipelines.

## When NOT to use

- The data are already stored in a SummarizedExperiment container with populated rowData/colData — maplet pipelines can proceed directly without re-loading from the repository.
- The task is a downstream pipeline step (annotation, statistical analysis, visualization, reporting) rather than ingesting repository data into maplet's container.
- Only a single committed, untagged version of maplet is available — commits without a release tag are not guaranteed to be stable, so repository loader behavior may not match v1.2.1.

## Inputs

- MetaboLights or Metabolomics Workbench accession for a metabolomics study
- Processed (intensity) data matrix (metabolite x sample) downloaded from the repository
- Accompanying sample annotation metadata from the repository
- Accompanying metabolite annotation metadata from the repository
- maplet v1.2.1 installed in R

## Outputs

- summarized_experiment_container.rds — a SummarizedExperiment with the metabolite-by-sample intensity matrix in the assay slot and parsed sample/metabolite annotations in rowData/colData

## How to apply

Start from the processed data matrix of the target metabolomics study, identified by its MetaboLights or Metabolomics Workbench accession, together with the accompanying sample and metabolite annotation metadata. Install the pinned stable maplet release (v1.2.1) via devtools so the specialized repository loader functions are available. Invoke the maplet loader to read the intensity matrix and parse the repository metadata into structured annotation tables. Assemble the matrix and annotations into a single SummarizedExperiment object: place the metabolite-by-sample measurements in the assay slot and the parsed annotations in rowData/colData. Save the assembled container with saveRDS as summarized_experiment_container.rds, since maplet builds on SummarizedExperiment as the central repository for each pipeline's data, analysis steps, and results.

## Related tools

- **maplet** (Provides the specialized loading functions that fetch/parse Metabolomics Workbench and MetaboLights datasets and build the SummarizedExperiment container) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Central container structure receiving the assay matrix and rowData/colData annotations)
- **devtools** (Used to install the pinned maplet release: devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"))
- **magrittr** (Supplies the %>% pipe operator used to connect maplet pipeline steps without temporary variables)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- Reload the saved summarized_experiment_container.rds and confirm it is a valid SummarizedExperiment object.
- Check that assay dimensions match the downloaded processed data matrix (number of metabolites x number of samples).
- Check that dimension names of the assay match the metabolite annotation table (rowData) and sample annotation table (colData).
- Confirm the parsed rowData/colData annotation tables faithfully reflect the repository metadata for the given accession.

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable — pin to the v1.2.1 release for reproducible loading behavior.
- No changelog is published, so tracking loader-function changes across versions relies on comparing commits.
- Loading is limited to the repositories with dedicated loaders (Metabolomics Workbench and MetaboLights); other metabolomics repositories require manual construction of the SummarizedExperiment.

## Evidence

- [methods] loader functions for repositories: [evidence span withheld — anchor status: not_found]
