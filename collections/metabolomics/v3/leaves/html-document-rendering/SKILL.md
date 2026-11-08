---
name: html-document-rendering
description: Use when a completed maplet pipeline container (SummarizedExperiment-derived
  object saved as .RDS) must be turned into a shareable report documenting every executed
  function, its arguments, and its embedded plots/statistics tables.
when_to_use_negative:
- The pipeline has not been run to completion, so the container lacks recorded annotations
  and stored outputs for all steps
- The analysis object is a plain data frame or feature table rather than a maplet
  container built on SummarizedExperiment — there are no recorded function entries
  to render
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: maplet
  role: Provides the reporting functions that assemble pipeline functions, arguments,
    plots and statistics tables into the rendered report
  repo: https://github.com/krumsieklab/maplet
- name: R
  role: Language in which the container is loaded (readRDS) and the report is generated/compiled
- name: SummarizedExperiment
  role: Data structure serving as the central repository from which report content
    (data, analysis steps, results, visualizations) is drawn
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:3dab7c9a7ff8f691c6df8a9aca3f485de00a288026256baa6fd6c80164143366
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# html-document-rendering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render a maplet pipeline container's recorded analysis steps, arguments, plots and statistics tables into a single self-contained HTML report (maplet_report.html), making the full metabolomics pipeline run shareable and reproducible.

## When to use

Use when a completed maplet pipeline container (SummarizedExperiment-derived object saved as .RDS) must be turned into a shareable report documenting every executed function, its arguments, and its embedded plots/statistics tables.

## When NOT to use

- The pipeline has not been run to completion, so the container lacks recorded annotations and stored outputs for all steps
- The analysis object is a plain data frame or feature table rather than a maplet container built on SummarizedExperiment — there are no recorded function entries to render

## Inputs

- Completed maplet pipeline container (SummarizedExperiment-derived object saved as .RDS)
- Recorded annotations listing executed functions, their arguments, and stored plots/statistics tables

## Outputs

- Self-contained HTML report (maplet_report.html) enumerating every executed function in order with arguments and embedded plots/statistics tables (alternatively PDF or Word)

## How to apply

Load the completed maplet pipeline container with readRDS; the SummarizedExperiment-derived object serves as the central repository of the pipeline's data, analysis steps, results and visualizations. Traverse the container's recorded annotations to recover the full, ordered list of executed pipeline functions together with their recorded arguments — maplet's reporting stage does this automatically for every function run through the %>% or |> pipe. For each recorded function entry, attach the plots and statistics tables that maplet stored in the container so each step's outputs are embedded alongside its description. Render all entries sequentially into one self-contained HTML document (maplet_report.html), which can also be produced as PDF or Word; the goal is that the reporting stage of the pipeline is fully reproducible without re-running analyses.

## Related tools

- **maplet** (Provides the reporting functions that assemble pipeline functions, arguments, plots and statistics tables into the rendered report) — https://github.com/krumsieklab/maplet
- **R** (Language in which the container is loaded (readRDS) and the report is generated/compiled)
- **SummarizedExperiment** (Data structure serving as the central repository from which report content (data, analysis steps, results, visualizations) is drawn)

## Examples

```
readRDS("pipeline_container.rds") %>% mt_reporting_html(output_file = "maplet_report.html")
```

## Evaluation signals

- The generated HTML enumerates every executed function from the container in execution order
- Each function entry includes its recorded arguments
- Each function entry embeds the plots and statistics tables stored in the container
- maplet_report.html exists as a well-formed, self-contained HTML document
- The report is reproducible without re-running the analysis, since all content comes from the container

## Limitations

- The report is only as complete as the container's recorded annotations — steps not executed through the maplet pipe framework will not be captured
- maplet is in active development; commits without a release tag are not guaranteed stable (README), which can affect report rendering behavior
- Output format is limited to the document types maplet compiles (HTML, PDF, Word)

## Evidence

- [intro] data loading, annotation, statistical analysis, visualization, and reporting: "maplet provides a suite of functions for interacting with this container including but not limited to data loading, annotation, statistical analysis, visualization, and reporting."
- [readme] SummarizedExperiment as central repository: [evidence span withheld — anchor status: not_found]
- [abstract] centralized pipeline framework: "It builds on the SummarizedExperiment data structure to create a centralized pipeline framework for storing data, analysis steps, results and visualizations"
- [intro] reproducible pipelines via pipe operator: "This operator allows for smooth connections between pipeline steps, without the need for temporary variables or multiple assignments."
