---
name: metabolomics-dataset-loading-from-public-repositories
description: Use when the analysis target is a metabolomics dataset identified by
  a MetaboLights or Metabolomics Workbench accession and the raw processed matrix
  must be pulled from the repository and converted into a SummarizedExperiment for
  downstream maplet pipelines.
when_to_use_negative:
- The intensity matrix and annotations are already available as local files and no
  repository retrieval is needed — use a standard local data-loading function instead.
- The data are already held in a SummarizedExperiment container and only later pipeline
  stages (statistical analysis, visualization, reporting) are required.
- The dataset is not hosted on MetaboLights or Metabolomics Workbench and no matching
  maplet repository loader exists.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: maplet
  role: Provides the specialized loading functions for the Metabolomics Workbench
    and MetaboLights repositories that read the intensity matrix and parse repository
    metadata into structured annotation tables
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Central container in which the downloaded metabolite-by-sample assay and rowData/colData
    annotations are assembled
- name: devtools
  role: Installs the pinned maplet release (v1.2.1) from GitHub so the repository
    loader functions are available
- name: magrittr
  role: Supplies the %>% pipe operator used to connect the loading step into maplet
    pipelines without temporary variables
- name: R
  role: Language environment in which the maplet loaders and SummarizedExperiment
    assembly are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:fdc1bcdac120041637b6872de8a07db26aa313e6ae79986b7810b0db53977a60
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolomics-dataset-loading-from-public-repositories

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fetches a metabolomics study's processed intensity matrix and its sample/metabolite metadata from public repositories such as MetaboLights and Metabolomics Workbench using maplet's specialized loader functions, and assembles the result into a SummarizedExperiment container.

## When to use

Use when the analysis target is a metabolomics dataset identified by a MetaboLights or Metabolomics Workbench accession and the raw processed matrix must be pulled from the repository and converted into a SummarizedExperiment for downstream maplet pipelines.

## When NOT to use

- The intensity matrix and annotations are already available as local files and no repository retrieval is needed — use a standard local data-loading function instead.
- The data are already held in a SummarizedExperiment container and only later pipeline stages (statistical analysis, visualization, reporting) are required.
- The dataset is not hosted on MetaboLights or Metabolomics Workbench and no matching maplet repository loader exists.

## Inputs

- MetaboLights or Metabolomics Workbench study accession identifier
- processed metabolite-by-sample intensity matrix downloaded from the repository
- sample and metabolite annotation metadata files accompanying the repository dataset

## Outputs

- SummarizedExperiment object with the intensity matrix in the assay slot and parsed repository annotations in rowData/colData
- summarized_experiment_container.rds saved via saveRDS

## How to apply

Install the pinned maplet release (v1.2.1) with devtools::install_github so the repository loader functions are available. Take the MetaboLights / Metabolomics Workbench accession for the target study and use the corresponding maplet loader to download the processed data matrix plus the accompanying sample and metabolite annotation metadata; the loader parses the repository metadata into structured annotation tables. Assemble the matrix and annotations into a single SummarizedExperiment object, placing the metabolite-by-sample measurements in the assay slot and the parsed annotations in rowData/colData, consistent with maplet's design where SummarizedExperiment serves as the central repository for each pipeline's data. Save the assembled container to disk with saveRDS as summarized_experiment_container.rds so subsequent maplet pipeline stages (annotation, statistical analysis, visualization, reporting) can consume it via the %>% or |> pipe operator.

## Related tools

- **maplet** (Provides the specialized loading functions for the Metabolomics Workbench and MetaboLights repositories that read the intensity matrix and parse repository metadata into structured annotation tables) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Central container in which the downloaded metabolite-by-sample assay and rowData/colData annotations are assembled)
- **devtools** (Installs the pinned maplet release (v1.2.1) from GitHub so the repository loader functions are available)
- **magrittr** (Supplies the %>% pipe operator used to connect the loading step into maplet pipelines without temporary variables)
- **R** (Language environment in which the maplet loaders and SummarizedExperiment assembly are executed)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"); se <- maplet::mt_load_metabolights(accession="MTBLS1"); saveRDS(se, "summarized_experiment_container.rds")
```

## Evaluation signals

- Reload summarized_experiment_container.rds and confirm it is a valid SummarizedExperiment object.
- The assay dimensions of the reloaded container match the downloaded data matrix (metabolites × samples).
- rownames (metabolite annotations) and colnames (sample annotations) of the container match the annotation tables parsed from the repository metadata.
- The maplet installation is the pinned stable release (v1.2.1), not an untagged development commit.

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable, so the pinned v1.2.1 release should be used.
- The loader coverage is limited to the supported repositories ('Metabolomics Workbench' and 'MetaboLights'); other repositories are not handled by these functions.

## Evidence

- [readme] maplet provides a suite of functions for interacting with this container including but not limited to data loading, annotation, statistical analysis, visualization, and reporting.: "maplet provides a suite of functions for interacting with this container including but not limited to data loading, annotation, statistical analysis, visualization, and reporting."
- [readme] The toolbox builds upon the bioconductor package SummarizedExperiment (SE), which serves as a central repository for each pipeline's data, analysis steps, and results.: [evidence span withheld — anchor status: not_found]
- [intro] maplet is an R package for statistical data analysis with a special focus on metabolomics datasets. It allows users to create self-contained analytical pipelines.: "maplet is an R package for statistical data analysis with a special focus on metabolomics datasets. It allows users to create self-contained analytical pipelines."
- [methods] devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"): "devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")"
- [readme] maplet is in active development. Any commit without a release tag is not guaranteed to be stable.: "maplet is in active development. Any commit without a release tag is not guaranteed to be stable."
