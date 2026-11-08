---
name: metabolomics-data-loading-and-annotation
description: Use when a metabolomics dataset (e.g. from MetaboLights) must be read
  into R and annotated as the entry point of a reproducible maplet pipeline built
  on SummarizedExperiment.
when_to_use_negative:
- The data is already a processed feature table outside R and the goal is a one-off
  statistical test rather than a modular, reproducible maplet pipeline
- You need a stable, versioned release workflow but only an untagged development commit
  of maplet is available — any commit without a release tag is not guaranteed to be
  stable
- The analysis targets a domain outside maplet's metabolomics/omics scope and none
  of its 90+ data loading, annotation, statistical analysis, visualization, or reporting
  functions apply
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3370
- http://edamontology.org/topic_0219
tools:
- name: maplet
  role: R package whose data loading and annotation functions wrap the dataset in
    the SummarizedExperiment pipeline container
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor data structure used as the centralized pipeline container for
    data, analysis steps, and results
  repo: https://bioconductor.org/packages/release/bioc/vignettes/SummarizedExperiment/inst/doc/SummarizedExperiment.html
- name: magrittr
  role: Provides the %>% pipe operator used to smoothly connect loading, annotation,
    and downstream pipeline steps without temporary variables
  repo: https://magrittr.tidyverse.org/
- name: devtools
  role: Used to install the stable maplet release v1.2.1 from GitHub
  repo: https://github.com/krumsieklab/maplet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:032c4f37c3c61c3a3a77e14481d6b7e5a117dd95308c8b34d2ed37ac75cb2085
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3370
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomics-data-loading-and-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load a metabolomics dataset into R and wrap it in a SummarizedExperiment-based maplet pipeline container, then attach metabolite/sample annotation steps so the data becomes the centralized repository for all downstream analysis, results, and visualizations.

## When to use

Use when a metabolomics dataset (e.g. from MetaboLights) must be read into R and annotated as the entry point of a reproducible maplet pipeline built on SummarizedExperiment.

## When NOT to use

- The data is already a processed feature table outside R and the goal is a one-off statistical test rather than a modular, reproducible maplet pipeline
- You need a stable, versioned release workflow but only an untagged development commit of maplet is available — any commit without a release tag is not guaranteed to be stable
- The analysis targets a domain outside maplet's metabolomics/omics scope and none of its 90+ data loading, annotation, statistical analysis, visualization, or reporting functions apply

## Inputs

- Public metabolomics dataset from a repository such as MetaboLights (e.g. accession MTBLS1)
- Metabolite and sample annotation/metadata tables readable into R
- R environment with devtools for installing maplet v1.2.1

## Outputs

- SummarizedExperiment (SE) container holding the loaded metabolomics data plus annotation, applied steps, parameters, and package versions
- Pipeline container ready to be extended with statistical analysis (e.g. linear models, PCA) and visualization modules

## How to apply

Install maplet at the stable release v1.2.1 with devtools::install_github(repo='krumsieklab/maplet@v1.2.1', subdir='maplet'); untagged commits are not guaranteed to be stable. Read the metabolomics dataset into R using maplet's data-loading function so it is wrapped in the SummarizedExperiment (SE) container that serves as the central repository for the pipeline's data, analysis steps, and results. Chain annotation and subsequent preprocessing modules onto the container using a pipe operator — the magrittr %>% operator or base R's |>. Follow the structure of the large example pipeline in the examples folder (examples/use_case_examples/example_preprocessing_pipeline.R) and the maplet Reference Guide sections 1 and 2 before composing custom steps. Rationale: because every module writes its data, parameters, results, and plots back into the SE container, loading and annotating correctly at the start guarantees the whole downstream pipeline remains self-contained and retracable.

## Related tools

- **maplet** (R package whose data loading and annotation functions wrap the dataset in the SummarizedExperiment pipeline container) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor data structure used as the centralized pipeline container for data, analysis steps, and results) — https://bioconductor.org/packages/release/bioc/vignettes/SummarizedExperiment/inst/doc/SummarizedExperiment.html
- **magrittr** (Provides the %>% pipe operator used to smoothly connect loading, annotation, and downstream pipeline steps without temporary variables) — https://magrittr.tidyverse.org/
- **devtools** (Used to install the stable maplet release v1.2.1 from GitHub) — https://github.com/krumsieklab/maplet

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- The loaded object is a SummarizedExperiment container rather than a plain data frame, and metadata/annotation are embedded in it
- The container records the applied analysis steps, parameters, and R/package versions, so each intermediate result can be retraced and inspected
- maplet was installed from the tagged stable release v1.2.1 (not an untagged commit)
- The pipeline reads as a linear chain of maplet function calls connected by %>% with no temporary variables or multiple assignments
- Downstream modules (linear models, PCA) run on the annotated container without re-loading raw data, confirming the container is self-contained

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable
- No release DOI or version identifier beyond the v1.2.1 install tag and no changelog are available
- Contributed functions must follow strict naming conventions (Reference Guide section 3.2.1) or may be rejected, constraining custom loading/annotation extensions

## Evidence

- [readme] The toolbox builds upon the bioconductor package SummarizedExperiment (SE), which serves as a central repository for each pipeline’s data, analysis steps, and results.: [evidence span withheld — anchor status: not_found]
- [readme] maplet provides a suite of functions for interacting with this container including but not limited to data loading, annotation, statistical analysis, visualization, and reporting.: "maplet provides a suite of functions for interacting with this container including but not limited to data loading, annotation, statistical analysis, visualization, and reporting."
- [methods] devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"): "devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")"
- [readme] maplet is in active development. Any commit without a release tag is not guaranteed to be stable.: "maplet is in active development. Any commit without a release tag is not guaranteed to be stable."
