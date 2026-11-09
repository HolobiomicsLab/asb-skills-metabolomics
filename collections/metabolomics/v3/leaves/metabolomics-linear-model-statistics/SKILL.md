---
name: metabolomics-linear-model-statistics
description: Use when a metabolomics dataset (e.g. a MetaboLights accession such as
  MTBLS1) has been loaded and preprocessed in an R/SummarizedExperiment container
  and per-metabolite association testing (linear models, statistical analysis) is
  the next analytical step.
when_to_use_negative:
- Input is raw instrument data (e.g. unprocessed vendor files) — maplet expects data
  to first be loaded, annotated and preprocessed into the SummarizedExperiment container
  before statistics are applied
- The analysis is not in R (e.g. a Python/Scikit-learn workflow) — maplet is an R
  package built on the Bioconductor SummarizedExperiment structure
- Only a single descriptive summary is needed with no statistical testing — maplet's
  value is in recording fitted models and parameters, which is overhead for one-off
  summaries
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: maplet
  role: Provides the statistical analysis functions (including linear models) that
    operate on and record results in the SE container
  repo: https://github.com/krumsieklelab/maplet
- name: R
  role: Runtime language in which the linear models are fitted and the pipeline is
    executed
- name: SummarizedExperiment
  role: Bioconductor data structure serving as the central repository for data, analysis
    steps and statistical results
- name: magrittr
  role: Supplies the %>% pipe operator that chains preprocessing and statistical steps
    without temporary variables
- name: devtools
  role: Used to install the pinned maplet release (krumsieklab/maplet@v1.2.1) from
    GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:93ea9d3eeb1d8a99c1eb40a696a06f36740f13191a2c7efc09f4074f0a443bac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/krumsieklab/maplet
  repo_url: https://github.com/krumsieklelab/maplet
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - magrittr
  - maplet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-linear-model-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits linear models and other statistical tests on metabolomics feature tables using maplet's R functions, embedding each fitted model, its parameters and results into the SummarizedExperiment pipeline container so every statistical step is retraced and reproducible.

## When to use

Use when a metabolomics dataset (e.g. a MetaboLights accession such as MTBLS1) has been loaded and preprocessed in an R/SummarizedExperiment container and per-metabolite association testing (linear models, statistical analysis) is the next analytical step.

## When NOT to use

- Input is raw instrument data (e.g. unprocessed vendor files) — maplet expects data to first be loaded, annotated and preprocessed into the SummarizedExperiment container before statistics are applied
- The analysis is not in R (e.g. a Python/Scikit-learn workflow) — maplet is an R package built on the Bioconductor SummarizedExperiment structure
- Only a single descriptive summary is needed with no statistical testing — maplet's value is in recording fitted models and parameters, which is overhead for one-off summaries

## Inputs

- SummarizedExperiment container holding a metabolomics feature table (e.g. loaded from a MetaboLights dataset such as MTBLS1)
- Sample metadata/annotations required as covariates for the linear model design
- maplet R package installed at stable release v1.2.1

## Outputs

- Linear model results (per-metabolite statistics) stored within the SummarizedExperiment container
- Container with embedded analysis steps, parameters and versions (e.g. saved to maplet_pipeline_output.RData)
- Optional PCA visualization embedded alongside the statistical results

## How to apply

Install maplet at the stable release tag v1.2.1 via devtools::install_github(repo='krumsieklab/maplet@v1.2.1', subdir='maplet'), since untagged commits are not guaranteed stable. Wrap the metabolomics data in the SummarizedExperiment-based container and chain annotation and preprocessing modules with the magrittr %>% (or base |>) pipe operator so no temporary variables are needed. Then apply maplet's statistical analysis functions — the package provides over 90 functions covering widely used statistical approaches — to fit linear models on the preprocessed metabolite measurements, optionally following with a principal component analysis for exploratory visualization. Each function writes the applied step, its parameters, results and plots back into the centralized SE container, so the full statistics trail is stored alongside the data. Finally persist the container (e.g. as maplet_pipeline_output.RData) to make the fitted models inspectable and reproducible.

## Related tools

- **maplet** (Provides the statistical analysis functions (including linear models) that operate on and record results in the SE container) — https://github.com/krumsieklelab/maplet
- **R** (Runtime language in which the linear models are fitted and the pipeline is executed)
- **SummarizedExperiment** (Bioconductor data structure serving as the central repository for data, analysis steps and statistical results)
- **magrittr** (Supplies the %>% pipe operator that chains preprocessing and statistical steps without temporary variables)
- **devtools** (Used to install the pinned maplet release (krumsieklab/maplet@v1.2.1) from GitHub)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- The SummarizedExperiment container passed between pipeline functions records all applied steps, parameters and results, so the fitted linear models can be retraced and inspected from the container alone
- The persisted maplet_pipeline_output.RData embeds the data, all applied steps, parameters, results and visualizations, and reloading it reproduces the statistics trail
- The installed maplet version is the tagged stable release v1.2.1, matching the documented install command
- The pipeline chains annotation, preprocessing and linear modeling via %>% without intermediate temporary variables, matching maplet's modular design
- Statistical functions are drawn from maplet's >90-function suite of widely used statistical approaches rather than ad hoc code

## Limitations

- maplet is in active development: any commit without a release tag is not guaranteed to be stable, so non-pinned installs can change statistical behavior
- No changelog or release DOI/version identifier is provided for tracing changes between releases
- The framework is R/Bioconductor-bound (SummarizedExperiment-based), though it offers some compatibility with external packages such as scgen

## Evidence

- [abstract] centralized pipeline framework on SummarizedExperiment: "It builds on the SummarizedExperiment data structure to create a centralized pipeline framework for storing data, analysis steps, results and visualizations"
