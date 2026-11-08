---
name: summarizedexperiment-container-assembly
description: 'Use when a maplet pipeline needs a starting container: you have an intensity
  matrix (e.g. from MetaboLights/Metabolomics Workbench or a raw CSV) plus annotation
  tables and must bind them into one SummarizedExperiment .rds.'
when_to_use_negative:
- The pipeline step is statistical analysis, visualization, or reporting rather than
  loading/annotating — maplet provides separate functions for those stages.
- Data is single-cell or non-metabolomics 'omics where SummarizedExperiment is not
  the intended container; maplet's compatibility with external packages like scgen
  is discussed but not part of this skill.
- The matrix and annotation tables have not been downloaded or parsed yet — the loader
  step (repository loaders) precedes assembly.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: maplet
  role: provides the data-loading and annotation functions used to populate the SummarizedExperiment
    container
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: the Bioconductor data structure used as the central container; assays hold
    the matrix, rowData/colData hold annotations
- name: R
  role: implementation language for maplet and the assembly script
- name: devtools
  role: installs the pinned maplet release v1.2.1 that supplies the loading/annotation
    functions
- name: magrittr
  role: pipe operator (%>%) connecting loading, annotation and assembly steps into
    one pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:30431119cbb0d8e1385cb4b1ee7a8e44805efdfe1135f6c507a9b9be053988a0
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# summarizedexperiment-container-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a metabolomics data matrix plus its sample and metabolite annotations into a single SummarizedExperiment object, the central container maplet pipelines operate on.

## When to use

Use when a maplet pipeline needs a starting container: you have an intensity matrix (e.g. from MetaboLights/Metabolomics Workbench or a raw CSV) plus annotation tables and must bind them into one SummarizedExperiment .rds.

## When NOT to use

- The pipeline step is statistical analysis, visualization, or reporting rather than loading/annotating — maplet provides separate functions for those stages.
- Data is single-cell or non-metabolomics 'omics where SummarizedExperiment is not the intended container; maplet's compatibility with external packages like scgen is discussed but not part of this skill.
- The matrix and annotation tables have not been downloaded or parsed yet — the loader step (repository loaders) precedes assembly.

## Inputs

- processed metabolomics intensity matrix (metabolite x sample)
- sample metadata table
- metabolite/feature annotation table
- repository accession (e.g. MetaboLights or Metabolomics Workbench study ID)

## Outputs

- SummarizedExperiment container saved as summarized_experiment_container.rds

## How to apply

Download the study's processed intensity matrix and its sample and metabolite annotation metadata (e.g. for a MetaboLights accession). Load the matrix into the assay slot and parse the repository annotations into rowData (metabolite-level) and colData (sample-level). maplet then uses this SummarizedExperiment as the central repository for the pipeline's data, analysis steps, and results, so dimension names must match the matrix rows and columns. Save the assembled container with saveRDS as summarized_experiment_container.rds and reload it to confirm it is a valid SummarizedExperiment with consistent assay dimensions and dimnames.

## Related tools

- **maplet** (provides the data-loading and annotation functions used to populate the SummarizedExperiment container) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (the Bioconductor data structure used as the central container; assays hold the matrix, rowData/colData hold annotations)
- **R** (implementation language for maplet and the assembly script)
- **devtools** (installs the pinned maplet release v1.2.1 that supplies the loading/annotation functions)
- **magrittr** (pipe operator (%>%) connecting loading, annotation and assembly steps into one pipeline)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet") # then: se <- metaseR(...); saveRDS(se, "summarized_experiment_container.rds")
```

## Evaluation signals

- reloaded .rds object is a valid SummarizedExperiment
- assay dimensions match the number of metabolite rows and sample columns in the source matrix
- dimnames/colData/rowData names correspond to the matrix rows and columns
- the container was written with saveRDS to summarized_experiment_container.rds
- piped steps run without temporary variables, consistent with the magrittr %>% pipeline style

## Limitations

- maplet is in active development; only release-tagged commits such as v1.2.1 are guaranteed stable — non-release commits may break loading/assembly functions.
- The package provides over 90 functions and strict naming conventions; use section 1–2 of the maplet Reference Guide and the examples folder before assembling non-standard containers.
- Non-metabolomics container compatibility (e.g. scgen) is a compatibility note, not a tested loader path.
- No changelog or release DOI was found in the article's discussion section.

## Evidence

- [readme] specialized loaders for Metabolomics Workbench and MetaboLights: [evidence span withheld — anchor status: not_found]
- [readme] SE as central repository for pipeline data, analysis steps, and results: [evidence span withheld — anchor status: not_found]
- [readme] the toolbox builds upon SummarizedExperiment as central data/results container: [evidence span withheld — anchor status: not_found]
