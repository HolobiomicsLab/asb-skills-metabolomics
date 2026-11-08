---
name: summarizedexperiment-container-wrangling
description: Use when building reproducible metabolomics (or other omics) pipelines
  in R where intermediate results, applied steps, and parameters must be retraced
  end-to-end from a single serialized object.
when_to_use_negative:
- The analysis goal is a one-off script whose intermediates do not need to be inspected
  or retraced — a plain data frame workflow may be simpler
- The toolchain is not R/Bioconductor (e.g. Python scRNA/omics workflows); maplet's
  SE container is R-specific, though maplet offers compatibility hooks for some external
  packages such as scgen
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: maplet
  role: R package providing the modular functions (data loading, annotation, statistical
    analysis, visualization, reporting) that operate on and record steps in the SE
    container
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor data structure serving as the centralized pipeline container
    that stores data, analysis steps, results, and visualizations
- name: magrittr
  role: supplies the %>% pipe operator used to chain maplet functions smoothly onto
    the container without temporary variables
- name: devtools
  role: used to install the pinned maplet release v1.2.1 from GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:ff12c56fc48769f665fbaf60448f98ece0f3d0aaefa2ae185a9a9e5b834490be
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# SummarizedExperiment container wrangling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Wraps metabolomics data and every pipeline stage in a centralized SummarizedExperiment (SE) container, as designed in the maplet R package, so that data, analysis steps, parameters, results, and visualizations all travel together through a %>%-piped workflow.

## When to use

Use when building reproducible metabolomics (or other omics) pipelines in R where intermediate results, applied steps, and parameters must be retraced end-to-end from a single serialized object.

## When NOT to use

- The analysis goal is a one-off script whose intermediates do not need to be inspected or retraced — a plain data frame workflow may be simpler
- The toolchain is not R/Bioconductor (e.g. Python scRNA/omics workflows); maplet's SE container is R-specific, though maplet offers compatibility hooks for some external packages such as scgen

## Inputs

- metabolomics dataset from a public repository (e.g. MetaboLights accession MTBLS1) read into R
- SummarizedExperiment (SE) container serving as the pipeline's central data repository
- maplet R package installed at stable release v1.2.1

## Outputs

- SummarizedExperiment container embedding data, analysis steps, parameters, results, and visualizations
- serialized pipeline object (e.g. maplet_pipeline_output.RData) from which every intermediate result can be retraced

## How to apply

Install maplet at the stable tagged release v1.2.1 via devtools::install_github(repo='krumsieklab/maplet@v1.2.1', subdir='maplet') — any commit without a release tag is not guaranteed to be stable. Load the metabolomics dataset (e.g. a MetaboLights accession such as MTBLS1) into R and wrap it in the SummarizedExperiment-based container that maplet uses as the central repository for data, analysis steps, and results. Chain annotation, preprocessing, statistical analysis (e.g. linear models), and visualization functions onto the container using the magrittr %>% pipe operator (or base R |>), avoiding temporary variables and multiple assignments so each step is recorded in the container. Finally persist the container (e.g. maplet_pipeline_output.RData), which embeds the data, all applied steps, parameters, results, and plots. Review the maplet Reference Guide sections 1–2 and the example_preprocessing_pipeline.R script before composing a new pipeline.

## Related tools

- **maplet** (R package providing the modular functions (data loading, annotation, statistical analysis, visualization, reporting) that operate on and record steps in the SE container) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor data structure serving as the centralized pipeline container that stores data, analysis steps, results, and visualizations)
- **magrittr** (supplies the %>% pipe operator used to chain maplet functions smoothly onto the container without temporary variables)
- **devtools** (used to install the pinned maplet release v1.2.1 from GitHub)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- The persisted RData file loads as a SummarizedExperiment object that embeds all applied analysis steps, parameters, and results, so each intermediate result can be retraced and inspected
- The installation used the tagged release v1.2.1 (not an untagged development commit) for stability
- The pipeline script reads as a simple, highly modular chain of maplet functions connected with %>% or |>, without temporary variables or multiple assignments
- Recorded R/package versions and analysis steps within the container match the environment and functions actually executed
- Statistical outputs (e.g. linear model fits, PCA) are retrievable directly from the container rather than from side files

## Limitations

- maplet is in active development: any commit without a release tag is not guaranteed to be stable
- Contributed functions must follow strict naming conventions (maplet Reference Guide section 3.2.1) or may be rejected
- No changelog or release DOI/version identifier was found in the article's discussion

## Evidence

- [intro] maplet achieves full pipeline transparency via a container passed between functions: [evidence span withheld — anchor status: not_found]
