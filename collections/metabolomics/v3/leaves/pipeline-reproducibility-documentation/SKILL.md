---
name: pipeline-reproducibility-documentation
description: Use when a completed maplet pipeline (a SummarizedExperiment-derived
  container saved as RDS) must be turned into a shareable, reproducible report enumerating
  all executed analysis steps and their outputs.
when_to_use_negative:
- The pipeline has not been run to completion, so the container lacks recorded function
  annotations, plots, or statistics tables to compile
- The input is raw metabolomics data (e.g. peak intensity files) rather than a processed
  maplet pipeline container — data loading and annotation stages should run first
- Only a single function's output is needed rather than documentation of the full
  ordered pipeline; inspect the container's annotations directly instead of rendering
  a report
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: maplet
  role: Provides the reporting functionality and the SummarizedExperiment-based container
    whose recorded annotations drive the auto-generated report
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Underlying Bioconductor data structure acting as the central repository for
    each pipeline's data, analysis steps, and results that the report is compiled
    from
- name: R
  role: Language in which the container is loaded (readRDS) and the R markdown/knitr
    report is rendered
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:ea23a83542b6a9e36a13710d624fec075f0526506604713995e27bf7a586eea6
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
  - maplet
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pipeline-reproducibility-documentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a single self-contained report (HTML, PDF, or Word) from a completed maplet pipeline container that documents every executed function in order, with its arguments and embedded plots/statistics tables, making the metabolomics analysis stage fully reproducible and shareable.

## When to use

Use when a completed maplet pipeline (a SummarizedExperiment-derived container saved as RDS) must be turned into a shareable, reproducible report enumerating all executed analysis steps and their outputs.

## When NOT to use

- The pipeline has not been run to completion, so the container lacks recorded function annotations, plots, or statistics tables to compile
- The input is raw metabolomics data (e.g. peak intensity files) rather than a processed maplet pipeline container — data loading and annotation stages should run first
- Only a single function's output is needed rather than documentation of the full ordered pipeline; inspect the container's annotations directly instead of rendering a report

## Inputs

- Completed maplet pipeline container stored as an RDS file (SummarizedExperiment-derived object holding data, analysis steps, results and visualizations)
- Container annotation slots recording executed function names and arguments

## Outputs

- A single self-contained report document (maplet_report.html, or PDF/Word) enumerating every executed pipeline function in execution order with its arguments and embedded plots and statistics tables

## How to apply

Load the completed pipeline container with readRDS in R; the container is a SummarizedExperiment-derived object serving as the centralized repository of data, analysis steps, results and visualizations. Traverse the container's recorded annotations to recover the full, ordered list of executed pipeline functions together with their recorded arguments. For each recorded function entry, attach the plots and statistics tables stored in the container so each step's outputs sit alongside its description. Render all entries sequentially via R markdown/knitr into one self-contained document (e.g. maplet_report.html); this 'reporting' stage is one of maplet's five supported pipeline stages (data loading, annotation, statistical analysis, visualization, reporting). Verify the rendered file enumerates every executed function in execution order with arguments and embedded outputs before sharing.

## Related tools

- **maplet** (Provides the reporting functionality and the SummarizedExperiment-based container whose recorded annotations drive the auto-generated report) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Underlying Bioconductor data structure acting as the central repository for each pipeline's data, analysis steps, and results that the report is compiled from)
- **R** (Language in which the container is loaded (readRDS) and the R markdown/knitr report is rendered)

## Examples

```
devtools::install_github(repo="krumsieklab/maplet@v1.2.1", subdir="maplet"); se <- readRDS("pipeline_container.rds")  # then render maplet_report.html from the container's recorded annotations
```

## Evaluation signals

- The generated HTML (maplet_report.html) exists and is a well-formed, self-contained document
- The report enumerates every executed function from the container, each with its name and recorded arguments
- Functions appear in their actual execution order as recorded in the container annotations
- Plots and statistics tables produced by each function are embedded alongside that function's description
- The report renders to HTML, PDF, or Word so it can be shared as a single reproducible artifact

## Limitations

- Reporting is the final stage: it can only document what was actually executed and recorded in the container, so nothing is emitted for skipped or failed steps
- Any commit of maplet without a release tag is not guaranteed to be stable, which can affect reproducibility of the reporting stage; the stable release (v1.2.1) should be pinned
- No changelog or release DOI was found for the repository, so version traceability must be established manually (e.g. by pinning the install commit)

## Evidence

- [readme] which serves as a central repository for each pipeline’s data, analysis steps, and results: [evidence span withheld — anchor status: not_found]
