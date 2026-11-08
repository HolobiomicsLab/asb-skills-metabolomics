---
name: r-summarizedexperiment-container-introspection
description: Use when you have a saved maplet pipeline container (.rds, read via readRDS)
  and need to recover what the pipeline did — function order, arguments, and per-step
  outputs — e.g. to auto-generate a report or audit reproducibility.
when_to_use_negative:
- You only have raw metabolomics data files and no completed maplet container — introspection
  requires the recorded annotations inside the SE-derived object, not raw spectra
  or feature tables.
- The pipeline was run outside maplet (e.g. a plain magrittr chain of non-maplet functions)
  — the container will not carry the recorded function entries and arguments this
  skill depends on.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: maplet
  role: Source of the pipeline container being introspected; its recorded annotations
    carry the executed function names, arguments, and stored plots/statistics tables
  repo: https://github.com/krumsieklab/maplet
- name: SummarizedExperiment
  role: Bioconductor data structure that underlies the container and serves as the
    central repository for the pipeline's data, analysis steps, and results
- name: R
  role: Language in which the container is loaded (readRDS), traversed, and rendered
    into the report
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab741
    title: '<tt>maplet</tt> : an extensible R toolbox for modular and reproducible
      metabolomics pipelines'
  source_content_hash: sha256:9ecb234205427b79eb83e0b5b69fafc5f60077fa38183f256c259a72a47ca607
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# r-summarizedexperiment-container-introspection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Traverse a maplet pipeline container (a SummarizedExperiment-derived R object) to recover the ordered list of executed functions, their arguments, and the plots/statistics tables each step produced, enabling faithful reconstruction and reporting of a completed metabolomics pipeline.

## When to use

Use when you have a saved maplet pipeline container (.rds, read via readRDS) and need to recover what the pipeline did — function order, arguments, and per-step outputs — e.g. to auto-generate a report or audit reproducibility.

## When NOT to use

- You only have raw metabolomics data files and no completed maplet container — introspection requires the recorded annotations inside the SE-derived object, not raw spectra or feature tables.
- The pipeline was run outside maplet (e.g. a plain magrittr chain of non-maplet functions) — the container will not carry the recorded function entries and arguments this skill depends on.

## Inputs

- maplet pipeline container: a SummarizedExperiment-derived R object saved as .rds (loaded with readRDS), holding data, recorded analysis steps, results, and visualizations

## Outputs

- Ordered enumeration of every executed pipeline function with its recorded arguments
- Per-step plots and statistics tables extracted from the container
- Self-contained report document (maplet_report.html, or PDF/Word)

## How to apply

Load the completed pipeline container in R with readRDS; it is a SummarizedExperiment (SE)-derived object that serves as the central repository for the pipeline's data, analysis steps, and results. Traverse the container's recorded annotations to recover the full, ordered list of executed pipeline functions together with their recorded arguments. For each recorded function entry, attach the plots and statistics tables stored in the container so each step's outputs are embedded alongside its description. Finally, render the entries sequentially — maplet itself compiles them into a single shareable HTML (maplet_report.html), PDF, or Word document — so the reporting stage is fully reproducible. This works because maplet's design stores every analysis step, result, and visualization inside the SE container as the pipeline runs via the %>% (magrittr) or base-R |> pipe operator.

## Related tools

- **maplet** (Source of the pipeline container being introspected; its recorded annotations carry the executed function names, arguments, and stored plots/statistics tables) — https://github.com/krumsieklab/maplet
- **SummarizedExperiment** (Bioconductor data structure that underlies the container and serves as the central repository for the pipeline's data, analysis steps, and results)
- **R** (Language in which the container is loaded (readRDS), traversed, and rendered into the report)

## Examples

```
library(maplet); se <- readRDS("pipeline_container.rds")  # traverse annotations to recover executed functions, args, plots and stats tables
```

## Evaluation signals

- The generated report enumerates every executed function from the container in execution order
- Each reported function entry includes its recorded arguments
- Each entry has the plots and statistics tables it produced embedded alongside its description
- The output exists as a well-formed self-contained HTML document (maplet_report.html)
- Re-running introspection on the same container reproduces the same ordered function list (reproducibility invariant)

## Limitations

- maplet is in active development: any commit without a release tag is not guaranteed to be stable (pin a release, e.g. v1.2.1 via devtools::install_github)
- Introspection fidelity depends on the pipeline having been run through maplet with the SE container as the central repository; unrecorded steps cannot be recovered

## Evidence

- [intro] Traverse container annotations for executed functions: [evidence span withheld — anchor status: not_found]
