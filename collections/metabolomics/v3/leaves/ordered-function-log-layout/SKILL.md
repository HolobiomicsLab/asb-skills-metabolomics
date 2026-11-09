---
name: ordered-function-log-layout
description: Use when a completed maplet pipeline container exists and you need a
  reproducible, human-readable document that enumerates all executed functions in
  order with their arguments and outputs.
when_to_use_negative:
- Input is raw metabolomics data or an unfinished pipeline — the report is compiled
  from a completed container that already stores the executed function annotations
  and outputs
- You need interactive or exploratory analysis rather than a static, shareable record
  of an already-executed pipeline
- The object was not built with maplet/SummarizedExperiment and therefore lacks the
  recorded annotations of executed functions and their arguments
edam_topics: []
tools:
- name: maplet
  role: R package whose pipeline container records executed functions, arguments,
    plots and statistics tables that the report lays out in execution order
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor data structure serving as the central repository for each pipeline's
    data, analysis steps, and results from which the report is assembled
- name: R
  role: Language in which the container is loaded with readRDS and the report is rendered
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:72b87eb1cc6012c4984f8ddff182b01f71edc8096b7961f282ec6f83873c7e69
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ordered-function-log-layout

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a maplet report that lays out every executed pipeline function in execution order — each with its name, arguments, and embedded plots/statistics tables — compiled into a single shareable HTML, PDF, or Word document from the SummarizedExperiment-derived pipeline container.

## When to use

Use when a completed maplet pipeline container exists and you need a reproducible, human-readable document that enumerates all executed functions in order with their arguments and outputs.

## When NOT to use

- Input is raw metabolomics data or an unfinished pipeline — the report is compiled from a completed container that already stores the executed function annotations and outputs
- You need interactive or exploratory analysis rather than a static, shareable record of an already-executed pipeline
- The object was not built with maplet/SummarizedExperiment and therefore lacks the recorded annotations of executed functions and their arguments

## Inputs

- Completed maplet pipeline container (SummarizedExperiment-derived R object saved as .rds, containing data, recorded function annotations, arguments, results and visualizations)

## Outputs

- Single self-contained HTML report (maplet_report.html) enumerating every executed function in execution order with its arguments and embedded plots/statistics tables (also renderable as PDF or Word)

## How to apply

Load the completed maplet pipeline container (a SummarizedExperiment-derived object that serves as the central repository of data, analysis steps, results and visualizations) in R with readRDS. Traverse the container's recorded annotations to recover the full, ordered list of executed pipeline functions together with their recorded arguments. For each recorded function entry, attach the plots and statistics tables stored in the container so each step's outputs sit alongside its description. Render all entries sequentially into one self-contained document (e.g. maplet_report.html), or PDF/Word, so the reporting stage of the pipeline — the last of maplet's data loading, annotation, statistical analysis, visualization, and reporting stages — is fully reproducible.

## Related tools

- **maplet** (R package whose pipeline container records executed functions, arguments, plots and statistics tables that the report lays out in execution order) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor data structure serving as the central repository for each pipeline's data, analysis steps, and results from which the report is assembled)
- **R** (Language in which the container is loaded with readRDS and the report is rendered)

## Examples

```
pipeline_se <- readRDS("pipeline_se.rds")  # load maplet SummarizedExperiment container, then render its ordered function log into maplet_report.html
```

## Evaluation signals

- The generated HTML enumerates every executed function from the container in execution order
- Each reported function entry includes its recorded arguments
- Each reported function entry embeds the plots and statistics tables stored in the container for that step
- The output exists as a well-formed, self-contained HTML document (maplet_report.html)
- Rendering from the same .rds container reproduces the same ordered function log, confirming reproducibility

## Limitations

- maplet is in active development; any commit without a release tag is not guaranteed to be stable, so report behavior may differ across versions (stable release v1.2.1 installed via devtools::install_github)
- Report completeness depends on the container faithfully recording annotations and outputs during execution; steps whose outputs were not stored cannot be embedded
- No release DOI or version identifier/changelog was provided for finer provenance tracking

## Evidence

- [intro] The toolbox builds upon the bioconductor package SummarizedExperiment (SE), which serves as a central repository for each pipeline’s data, analysis steps, and results.: [evidence span withheld — anchor status: not_found]
