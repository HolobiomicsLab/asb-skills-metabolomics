---
name: metabolomics-pca-visualization
description: Use when exploring sample clustering or batch effects in a metabolomics
  feature matrix (e.g. a MetaboLights dataset) and you need an exploratory PCA whose
  provenance is recorded alongside preprocessing and annotation steps.
when_to_use_negative:
- Input is raw instrument-level data (not yet peak-picked into a metabolite-by-sample
  matrix) that still needs preprocessing outside maplet's container framework
- The analysis goal is supervised statistical modeling rather than exploratory dimensionality
  reduction and sample visualization
- A one-off PCA outside a reproducible pipeline is sufficient and container-embedded
  provenance is not required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3383
tools:
- name: maplet
  role: R package providing the statistical-analysis and visualization functions (including
    PCA) that operate on the SummarizedExperiment pipeline container
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor data structure serving as the central repository holding the
    metabolomics data, PCA results and visualizations
  repo: https://bioconductor.org/packages/release/bioc/vignettes/SummarizedExperiment/inst/doc/SummarizedExperiment.html
- name: magrittr
  role: Supplies the %>% pipe operator used to chain preprocessing and PCA steps into
    a smooth, modular pipeline
- name: devtools
  role: Used to install the stable maplet release v1.2.1 from GitHub before running
    the pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:b322aa1156972d2c5b7c7cde3434d21b89aa8ed64390390112a08287acfd6b76
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-pca-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and visualize principal component analysis on metabolomics data within a maplet pipeline, using the SummarizedExperiment-based container so the PCA result and its plot are embedded alongside all upstream preprocessing steps.

## When to use

Use when exploring sample clustering or batch effects in a metabolomics feature matrix (e.g. a MetaboLights dataset) and you need an exploratory PCA whose provenance is recorded alongside preprocessing and annotation steps.

## When NOT to use

- Input is raw instrument-level data (not yet peak-picked into a metabolite-by-sample matrix) that still needs preprocessing outside maplet's container framework
- The analysis goal is supervised statistical modeling rather than exploratory dimensionality reduction and sample visualization
- A one-off PCA outside a reproducible pipeline is sufficient and container-embedded provenance is not required

## Inputs

- Public metabolomics dataset from the MetaboLights repository (e.g. accession MTBLS1)
- Metabolomics feature/abundance matrix read into R and wrapped in a SummarizedExperiment container

## Outputs

- PCA result and visualization embedded in the SummarizedExperiment pipeline container
- Persisted maplet_pipeline_output.RData containing data, applied steps, parameters, results and plots

## How to apply

Load the metabolomics dataset into R and wrap it in maplet's SummarizedExperiment-based pipeline container, which serves as the central repository for data, analysis steps, results and visualizations. Chain data-loading, annotation and preprocessing modules onto the container using the %>% pipe operator from magrittr (or base R's |>) so that the PCA is computed on consistently preprocessed measurements without temporary variables. Invoke maplet's PCA/statistical-analysis functions on the preprocessed metabolite measurements as part of the chained pipeline. Because maplet embeds all results and visualizations back into the container, the PCA scores/plots become inspectable intermediate results; persist the container to an .RData file so every step and parameter can be retraced.

## Related tools

- **maplet** (R package providing the statistical-analysis and visualization functions (including PCA) that operate on the SummarizedExperiment pipeline container) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor data structure serving as the central repository holding the metabolomics data, PCA results and visualizations) — https://bioconductor.org/packages/release/bioc/vignettes/SummarizedExperiment/inst/doc/SummarizedExperiment.html
- **magrittr** (Supplies the %>% pipe operator used to chain preprocessing and PCA steps into a smooth, modular pipeline)
- **devtools** (Used to install the stable maplet release v1.2.1 from GitHub before running the pipeline)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- The PCA is computed inside the SummarizedExperiment container, so downstream inspection of the container reveals the PCA result, its plot, and all prior steps and parameters
- The final persisted .RData embeds data, results, plots, R/package versions and analysis steps, allowing every intermediate result (including PCA) to be retraced
- The pipeline runs as a chain of maplet function calls connected by %>% (or |>) with no temporary variables or multiple assignments
- The installed package corresponds to the stable release v1.2.1; untagged commits are not guaranteed to be stable

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable
- maplet functions follow strict naming conventions, and non-conforming functions may be rejected for contribution
- No changelog or release DOI/version identifier beyond the v1.2.1 tag was available in the provided context

## Evidence
