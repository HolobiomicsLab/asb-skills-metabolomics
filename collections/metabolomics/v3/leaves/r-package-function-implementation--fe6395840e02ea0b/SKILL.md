---
name: r-package-function-implementation
description: Use when writing, extending, or contributing a function to an R/Bioconductor
  package such as maplet, or when reconstructing one of its data-loading functions
  (e.g. MetaboLights or Metabolomics Workbench loaders) inside a reproducible pipeline.
when_to_use_negative:
- The analysis is not in R or does not use the SummarizedExperiment container as the
  central data structure
- The task is a one-off analysis script rather than a reusable, name-conforming package
  function intended for collaborative maintenance
- The pipeline must run outside the pinned maplet release (untagged commits are explicitly
  not guaranteed to be stable)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: maplet
  role: Target R package whose modular-function conventions (naming rules, SummarizedExperiment-centric
    design, pipe compatibility) the implemented function must follow
  repo: https://github.com/krumsieklab/maplet
- name: devtools
  role: Installs the pinned maplet release via devtools::install_github during development
    and testing
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor container that every implemented function reads from and writes
    back to (assay, rowData, colData)
- name: magrittr
  role: Provides the %>% pipe operator that connects the implemented functions into
    smooth, modular pipelines
  repo: https://magrittr.tidyverse.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:3b4b560fab68401938a112b9ad6c323f62b174ea35ba5781d45e0b5941c2d737
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-package-function-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement or extend R package functions following the maplet convention: modular, single-responsibility functions that operate on a central SummarizedExperiment container and are chained with pipe operators (%>% or |>). This skill covers installation of a pinned release, adhering to the package's naming conventions, and validating contributions against its testing framework.

## When to use

Use when writing, extending, or contributing a function to an R/Bioconductor package such as maplet, or when reconstructing one of its data-loading functions (e.g. MetaboLights or Metabolomics Workbench loaders) inside a reproducible pipeline.

## When NOT to use

- The analysis is not in R or does not use the SummarizedExperiment container as the central data structure
- The task is a one-off analysis script rather than a reusable, name-conforming package function intended for collaborative maintenance
- The pipeline must run outside the pinned maplet release (untagged commits are explicitly not guaranteed to be stable)

## Inputs

- R session with devtools available
- maplet package source or installed release v1.2.1 from krumsieklab/maplet
- Metabolomics repository accession (MetaboLights or Metabolomics Workbench) with processed data matrix and sample/metabolite annotation metadata
- SummarizedExperiment container under construction (assay + rowData/colData)

## Outputs

- Installed maplet R package at a pinned release tag
- Modular, pipe-compatible R function following maplet naming conventions
- SummarizedExperiment object serialized via saveRDS (e.g. summarized_experiment_container.rds)
- Test results from the package testing framework in /tests

## How to apply

Install the pinned stable release with devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet") rather than an untagged commit, since the README warns any commit without a release tag is not guaranteed to be stable. Study existing functions and the Reference Guide before contributing: maplet functions follow strict naming conventions (section 3.2.1 of the Reference Guide) and non-conforming submissions may be rejected. Design each function as a modular unit operating on the SummarizedExperiment container — e.g. a repository loader parses the intensity matrix into the assay slot and repository metadata into rowData/colData — and connect steps with the magrittr %>% (or base R |>) pipe so no temporary variables are needed. Review the examples folder (e.g. examples/use_case_examples/example_preprocessing_pipeline.R) for canonical pipeline structure, and run the package's testing framework in /tests to confirm the new function behaves as expected after updates.

## Related tools

- **maplet** (Target R package whose modular-function conventions (naming rules, SummarizedExperiment-centric design, pipe compatibility) the implemented function must follow) — https://github.com/krumsieklab/maplet
- **devtools** (Installs the pinned maplet release via devtools::install_github during development and testing) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor container that every implemented function reads from and writes back to (assay, rowData, colData))
- **magrittr** (Provides the %>% pipe operator that connects the implemented functions into smooth, modular pipelines) — https://magrittr.tidyverse.org/

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet")
```

## Evaluation signals

- The installed package reports the pinned release v1.2.1 (devtools::install_github with @v1.2.1), not an untagged commit
- The new function's name complies with maplet's naming conventions (Reference Guide section 3.2.1)
- The function round-trips a SummarizedExperiment: saveRDS then readRDS reproduces assay dimensions and dimension names matching the source matrix and annotation tables
- The function can be inserted into a %>% pipeline without temporary variables or intermediate assignments
- The package testing framework in /tests passes with the new function included

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable, so development must target tagged releases
- Submitted functions may initially be rejected if they do not follow the function naming rules; questions must go to the maintainer
- No changelog or release DOI/version identifier is provided in the article's discussion, complicating version tracking
- Data-loading functions depend on external repository access (MetaboLights, Metabolomics Workbench), adding network availability as a failure mode

## Evidence

- [readme] Pipe-operator design of maplet functions: [evidence span withheld — anchor status: not_found]
