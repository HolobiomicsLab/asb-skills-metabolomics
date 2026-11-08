---
name: pipe-based-r-pipeline-construction
description: Use when building a multi-step R statistical workflow (e.g. metabolomics
  data loading, annotation, statistical analysis, visualization, reporting) where
  intermediate results must remain inspectable and the pipeline must be reproducible
  without temporary variables.
when_to_use_negative:
- The analysis is a single one-off function call with no need to trace intermediate
  results or record parameters
- The target environment is not R — the pipe-and-container pattern here is built specifically
  on R's magrittr/base pipe operators and the SummarizedExperiment data structure
- You need a bleeding-edge untagged commit's behaviour and cannot tolerate instability,
  since any commit without a release tag is not guaranteed to be stable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: maplet
  role: R toolbox whose modular functions are chained by the pipe operator to build
    the metabolomics analysis pipeline
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor data structure serving as the central repository for each pipeline's
    data, analysis steps and results
- name: magrittr
  role: Provides the %>% pipe operator used to connect pipeline steps smoothly without
    temporary variables
- name: devtools
  role: Used to install the pinned stable maplet release (v1.2.1) from GitHub
- name: R
  role: Implementation language of the package and the pipeline execution environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:5d9b9942fae813e0aeb5842f472280272dc9b1722f2860bec8864beab749dc01
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pipe-based-r-pipeline-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct self-contained, reproducible R analysis pipelines by chaining package functions with the %>% (magrittr) or |> (base R) pipe operator, using SummarizedExperiment as the central container that carries data, analysis steps, parameters, results and plots through the whole workflow.

## When to use

Use when building a multi-step R statistical workflow (e.g. metabolomics data loading, annotation, statistical analysis, visualization, reporting) where intermediate results must remain inspectable and the pipeline must be reproducible without temporary variables.

## When NOT to use

- The analysis is a single one-off function call with no need to trace intermediate results or record parameters
- The target environment is not R — the pipe-and-container pattern here is built specifically on R's magrittr/base pipe operators and the SummarizedExperiment data structure
- You need a bleeding-edge untagged commit's behaviour and cannot tolerate instability, since any commit without a release tag is not guaranteed to be stable

## Inputs

- Public metabolomics dataset from a repository such as MetaboLights (e.g. accession MTBLS1)
- Raw metabolomics data table read into R and wrapped in a SummarizedExperiment container
- Sample and feature annotation metadata

## Outputs

- SummarizedExperiment container embedding data, applied analysis steps, parameters, results and visualizations
- Persisted pipeline object, e.g. maplet_pipeline_output.RData
- Statistical analysis results (linear model fits, PCA) and plots recorded inside the container

## How to apply

Install the tool at a stable tagged release with devtools::install_github(repo='krumsieklab/maplet@v1.2.1', subdir='maplet') rather than an untagged commit, since untagged commits are not guaranteed to be stable. Load the raw dataset into the SummarizedExperiment-based pipeline container, then chain annotation, preprocessing, statistical modelling (e.g. linear models), and PCA functions onto that container using the magrittr %>% pipe so that each function receives and returns the container, automatically recording applied steps and parameters. Review the example pipelines in the package's examples folder (e.g. example_preprocessing_pipeline.R) and Reference Guide sections 1 and 2 before writing a new pipeline, following the package's strict function naming conventions if adding custom functions. Finally, persist the container (which embeds data, results, versions and visualizations) to an RData file so every intermediate result can be retraced.

## Related tools

- **maplet** (R toolbox whose modular functions are chained by the pipe operator to build the metabolomics analysis pipeline) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor data structure serving as the central repository for each pipeline's data, analysis steps and results)
- **magrittr** (Provides the %>% pipe operator used to connect pipeline steps smoothly without temporary variables)
- **devtools** (Used to install the pinned stable maplet release (v1.2.1) from GitHub)
- **R** (Implementation language of the package and the pipeline execution environment)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- The SummarizedExperiment container returned by the final piped function contains metadata recording every analysis step and its parameters, so each intermediate result can be retraced
- The pipeline script contains no temporary variables or repeated intermediate assignments — data flows end-to-end through pipe operators
- The installed package version matches the tagged stable release v1.2.1 (check recorded R/package versions stored in the container)
- The persisted output file (e.g. maplet_pipeline_output.RData) embeds data, results and plots, and reloading it reproduces the full analysis state
- Pipeline structure matches the worked examples in the package's examples folder (e.g. example_preprocessing_pipeline.R)

## Limitations

- maplet is in active development: any commit without a release tag is not guaranteed to be stable, so pipelines must pin a tagged release
- Custom contributed functions are subject to strict naming conventions (Reference Guide section 3.2.1) and may initially be rejected if naming rules are not followed
- No changelog or release DOI/version identifier is provided in the discussion, complicating version tracking beyond the GitHub tag

## Evidence

- [readme] maplet is designed to work with a pipe operator - either the popular %>% operator from the magrittr package or the recently introduced |> from base R: [evidence span withheld — anchor status: not_found]
