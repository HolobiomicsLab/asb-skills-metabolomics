---
name: sample-annotation-integration
description: Use when the input is a metabolomics dataset downloaded from Metabolomics
  Workbench or MetaboLights (processed data matrix plus sample/metabolite annotation
  tables) and it must be assembled into a single annotated container for maplet statistical
  pipelines.
when_to_use_negative:
- The data are already held in a SummarizedExperiment with populated rowData/colData
  and need no repository metadata parsing
- The data come from a non-repository source (e.g. local peak table or mzML-derived
  matrix) rather than a Metabolomics Workbench / MetaboLights accession
- Only raw vendor files are available with no processed matrix or annotation tables,
  since the loaders fetch processed repository data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: maplet
  role: provides the specialized loading functions for Metabolomics Workbench and
    MetaboLights and parses the metadata into SummarizedExperiment annotation slots
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor container whose assay, rowData and colData slots hold the intensity
    matrix and parsed annotations
  repo: https://bioconductor.org/packages/release/bioc/vignettes/SummarizedExperiment/inst/doc/SummarizedExperiment.html
- name: devtools
  role: installs the pinned stable maplet release (v1.2.1) from GitHub
  repo: https://github.com/krumsieklab/maplet
- name: R
  role: language in which the maplet loader and SummarizedExperiment assembly are
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:b12937f34a53d1be4d5df7d57857669a0f1a52de30c0465648194c591c2633f1
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# sample-annotation-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrate sample and metabolite annotation metadata from online metabolomics data repositories (e.g. Metabolomics Workbench, MetaboLights) into a maplet SummarizedExperiment container, using maplet's specialized repository loader functions so the processed intensity matrix and its annotations travel together through downstream pipeline steps.

## When to use

Use when the input is a metabolomics dataset downloaded from Metabolomics Workbench or MetaboLights (processed data matrix plus sample/metabolite annotation tables) and it must be assembled into a single annotated container for maplet statistical pipelines.

## When NOT to use

- The data are already held in a SummarizedExperiment with populated rowData/colData and need no repository metadata parsing
- The data come from a non-repository source (e.g. local peak table or mzML-derived matrix) rather than a Metabolomics Workbench / MetaboLights accession
- Only raw vendor files are available with no processed matrix or annotation tables, since the loaders fetch processed repository data

## Inputs

- Metabolomics Workbench or MetaboLights study accession (processed data matrix download)
- sample annotation metadata table from the repository
- metabolite annotation metadata table from the repository

## Outputs

- SummarizedExperiment object with metabolite-by-sample assay, rowData (metabolite annotations) and colData (sample annotations)
- summarized_experiment_container.rds saved to disk via saveRDS

## How to apply

Start from a maplet release installed with devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"), since any commit without a release tag is not guaranteed to be stable. Download the target study's processed data matrix and accompanying sample and metabolite annotation metadata from the repository accession. Invoke the maplet loader function for the repository (maplet provides specialized loading functions for the online metabolomics data repositories 'Metabolomics Workbench' and 'MetaboLights') to read the intensity matrix and parse the repository metadata into structured annotation tables. Assemble the matrix and annotations into a single SummarizedExperiment object: place the metabolite-by-sample measurements in the assay slot and the parsed annotations in rowData/colData, then save the container with saveRDS as summarized_experiment_container.rds. This works because maplet builds on the SummarizedExperiment data structure, which serves as a central repository for each pipeline's data, analysis steps, and results.

## Related tools

- **maplet** (provides the specialized loading functions for Metabolomics Workbench and MetaboLights and parses the metadata into SummarizedExperiment annotation slots) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor container whose assay, rowData and colData slots hold the intensity matrix and parsed annotations) — https://bioconductor.org/packages/release/bioc/vignettes/SummarizedExperiment/inst/doc/SummarizedExperiment.html
- **devtools** (installs the pinned stable maplet release (v1.2.1) from GitHub) — https://github.com/krumsieklab/maplet
- **R** (language in which the maplet loader and SummarizedExperiment assembly are executed)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"); se <- maplet::mt_load_metabolights(accesion = "MTBLS1"); saveRDS(se, "summarized_experiment_container.rds")
```

## Evaluation signals

- Reload the saved .rds with readRDS and confirm it is a valid SummarizedExperiment object
- Assay dimensions match the downloaded data matrix (n metabolites x n samples)
- rownames/colnames of the assay match the identifiers in the repository annotation tables
- rowData and colData columns correspond to the parsed metabolite and sample annotation tables from the repository
- The pipeline runs without temporary variables when chained with %>% from magrittr

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable, so install the v1.2.1 release
- Loaders are specific to Metabolomics Workbench and MetaboLights; other repositories are not covered by the specialized loading functions

## Evidence

- [results] specialized loaders for repository data: [evidence span withheld — anchor status: not_found]
- [results] container assembly workflow: [evidence span withheld — anchor status: not_found]
