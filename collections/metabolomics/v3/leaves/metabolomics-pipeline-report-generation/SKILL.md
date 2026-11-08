---
name: metabolomics-pipeline-report-generation
description: Use when a completed maplet pipeline container (a SummarizedExperiment-derived
  object) must be turned into a reproducible report enumerating every executed function,
  its arguments, and its plots/statistics tables.
when_to_use_negative:
- The input is raw metabolomics data or an unfinished pipeline — reporting requires
  a completed container whose annotations already record the executed functions and
  their outputs
- The goal is a one-off ad-hoc analysis without reproducibility requirements — a hand-written
  R Markdown script would be simpler than reconstructing the container's report
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: maplet
  role: R package whose pipeline container annotations drive the auto-generated report;
    provides the reporting function suite
  repo: https://github.com/krumsieklab/maplet
- name: R
  role: Language in which the container is loaded with readRDS and the report is rendered
    via R Markdown/knitr
- name: SummarizedExperiment
  role: Bioconductor data structure underlying the container that stores the data,
    analysis steps, results, and visualizations compiled into the report
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:de4d9248bf86fee7e10f9db6ce56d9f4d6d19dd80a8cb940f7c95d674c9e7aca
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-pipeline-report-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the auto-generated R Markdown/knitr report from a completed maplet pipeline container: recover the ordered list of executed functions with their arguments and embedded outputs, and render them into a single shareable HTML, PDF, or Word document.

## When to use

Use when a completed maplet pipeline container (a SummarizedExperiment-derived object) must be turned into a reproducible report enumerating every executed function, its arguments, and its plots/statistics tables.

## When NOT to use

- The input is raw metabolomics data or an unfinished pipeline — reporting requires a completed container whose annotations already record the executed functions and their outputs
- The goal is a one-off ad-hoc analysis without reproducibility requirements — a hand-written R Markdown script would be simpler than reconstructing the container's report

## Inputs

- Completed maplet pipeline container saved as an RDS file (SummarizedExperiment-derived object holding data, analysis steps, results, and visualizations)
- Recorded function annotations in the container (ordered function names and arguments)

## Outputs

- A single self-contained report document (maplet_report.html by default; PDF or Word also possible) listing every executed pipeline function in execution order with its arguments and embedded plots/statistics tables

## How to apply

Load the finished maplet container with readRDS in R; the container is a SummarizedExperiment-derived object that serves as the centralized repository of data, analysis steps, results, and visualizations. Traverse the container's recorded annotations to recover the full, ordered list of executed pipeline functions together with their recorded arguments — this mirrors the reporting stage maplet places alongside data loading, annotation, statistical analysis, and visualization. For each recorded function entry, attach the plots and statistics tables stored in the container so each step's outputs are embedded alongside its description. Render all entries sequentially into one self-contained document (e.g. maplet_report.html; PDF or Word are also supported), which is what maplet's auto-assembled report does. Validate by confirming the HTML enumerates every executed function in execution order with arguments and embedded outputs, and is well-formed HTML.

## Related tools

- **maplet** (R package whose pipeline container annotations drive the auto-generated report; provides the reporting function suite) — https://github.com/krumsieklab/maplet
- **R** (Language in which the container is loaded with readRDS and the report is rendered via R Markdown/knitr)
- **SummarizedExperiment** (Bioconductor data structure underlying the container that stores the data, analysis steps, results, and visualizations compiled into the report)

## Examples

```
container <- readRDS("maplet_pipeline_container.rds"); rmarkdown::render("maplet_report.Rmd", output_file = "maplet_report.html")
```

## Evaluation signals

- The generated HTML enumerates every executed function from the container in execution order
- Each function entry includes its recorded arguments
- Plots and statistics tables produced by each function are embedded alongside its description in the report
- The output file (maplet_report.html) exists and is a well-formed, self-contained HTML document
- The report covers all maplet pipeline stages: data loading, annotation, statistical analysis, visualization, and reporting

## Limitations

- maplet is in active development — any commit without a release tag is not guaranteed to be stable, so report rendering behavior may change between commits
- The report is only as complete as the container's recorded annotations; steps not captured in the container will not appear
- Stable release install is pinned to a specific version (e.g. v1.2.1 via devtools::install_github); no changelog or release DOI is provided in the article

## Evidence

- [readme] The toolbox builds upon the bioconductor package SummarizedExperiment (SE), which serves as a central repository for each pipeline’s data, analysis steps, and results.: [evidence span withheld — anchor status: not_found]
