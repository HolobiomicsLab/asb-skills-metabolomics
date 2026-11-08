---
name: omics-web-tool-example-data-reproduction
description: Use when a task asks to reproduce a GraphBio module output from its demo
  CSV (e.g. surv_example.csv, venn_example.csv, text_cluster_example.csv) and the
  article names the wrapped R package to reimplement it locally.
when_to_use_negative:
- The demo input is for a different GraphBio module (e.g. roc_example.csv for ROC
  curves or ma_example.csv for MA plot) — swap to the matching package and rendering
  instead of survival analysis
- The goal is data preprocessing or statistical computing rather than plot generation
  — GraphBio 'specifically focuses on facilitating the generation of publication-ready
  plots instead of data preprocessing and computing'
- No reference example file exists for the module, so the expected default figure
  presentation cannot be validated against GraphBio's own demo output
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Source web app whose module output and demo files (surv_example.csv) are being
    reproduced; figures downloadable as PDF/PNG/JPEG/TIFF
  repo: https://github.com/databio2022/GraphBio
- name: survminer
  role: R package wrapped by GraphBio for its survival analysis module; survfit-based
    Kaplan-Meier curves rendered ggsurvplot-style with risk tables
- name: Shiny
  role: R framework underlying GraphBio's dashboard, which the local reproduction
    mimics
- name: ggplot2
  role: Underlying R plotting package in GraphBio's rendering stack for publication-ready
    figures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:6e69bc16d6f8800d6ffb4350a3f095d71a811668975bd00a9d5b3b0c027600ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  - survminer
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# omics-web-tool-example-data-reproduction

## Summary

Reproduces a demo output of the GraphBio Shiny web app locally by feeding its shipped reference example file (e.g. surv_example.csv for survival curves) through the same underlying R packages GraphBio wraps, then validating the rendered figure against the module's default publication-ready presentation.

## When to use

Use when a task asks to reproduce a GraphBio module output from its demo CSV (e.g. surv_example.csv, venn_example.csv, text_cluster_example.csv) and the article names the wrapped R package to reimplement it locally.

## When NOT to use

- The demo input is for a different GraphBio module (e.g. roc_example.csv for ROC curves or ma_example.csv for MA plot) — swap to the matching package and rendering instead of survival analysis
- The goal is data preprocessing or statistical computing rather than plot generation — GraphBio 'specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing'
- No reference example file exists for the module, so the expected default figure presentation cannot be validated against GraphBio's own demo output

## Inputs

- surv_example.csv — GraphBio demo survival input (sample IDs, survival times, censoring status, group labels)
- Other module-specific GraphBio demo CSVs, e.g. text_cluster_example.csv for cluster analysis, venn_example.csv for Venn plot

## Outputs

- survival_curves.png — per-group Kaplan-Meier curves with risk table, in GraphBio's default presentation
- Optionally PDF/JPEG/TIFF exports, matching GraphBio's downloadable figure formats

## How to apply

Identify the GraphBio module and its reference example file listed in the paper's methods and README (e.g. 'surv_example.csv for survival curves', file 13 of the 15 modules). Load the tabular example data — for survival analysis this carries sample IDs, survival times, censoring status, and group labels. Fit per-group Kaplan-Meier curves with survfit as wrapped by survminer (the package GraphBio uses for its survival module) and render the curves with at-risk tables and legends in ggsurvplot style, matching GraphBio's default presentation. Export the plot as PNG (GraphBio figures are downloadable as PDF/PNG/JPEG/TIFF; PNG retained here). Validate that the output file exists, is a valid image, and expert-review that the figure content (curves, risk table) is consistent with the module's defaults.

## Related tools

- **GraphBio** (Source web app whose module output and demo files (surv_example.csv) are being reproduced; figures downloadable as PDF/PNG/JPEG/TIFF) — https://github.com/databio2022/GraphBio
- **survminer** (R package wrapped by GraphBio for its survival analysis module; survfit-based Kaplan-Meier curves rendered ggsurvplot-style with risk tables)
- **Shiny** (R framework underlying GraphBio's dashboard, which the local reproduction mimics)
- **ggplot2** (Underlying R plotting package in GraphBio's rendering stack for publication-ready figures)

## Examples

```
library(survminer); fit <- survfit(Surv(time, status) ~ group, data = read.csv("surv_example.csv")); ggsurvplot(fit, risk.table = TRUE)
```

## Evaluation signals

- survival_curves.png exists and is a valid PNG image
- Expert review confirms the figure shows per-group Kaplan-Meier curves with a risk table consistent with the survminer-based GraphBio survival module
- Curve grouping and censoring marks correspond to the group labels and censoring status columns in surv_example.csv
- Output format matches GraphBio's downloadable formats (PNG retained from PDF/PNG/JPEG/TIFF)
- Reproduced plot style (legends, risk table layout) matches the module's stated default 'generally ideal for publication' presentation

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio's outputs is reported in the paper, so reproduction fidelity relies on default-figure inspection
- No version number or license stated in the article text (README docker images imply v2.2.5-en / v2.2.7); module defaults may drift across versions
- GraphBio focuses on plot generation only — users must prepare input data in Excel format themselves based on the reference example files

## Evidence

- [abstract] GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio.: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [readme] 13. surv_example.csv for survival curves: "13. surv_example.csv for survival curves"
- [methods] The source code of GraphBio is available at https://github.com/databio2022/GraphBio: "The source code of GraphBio is available at https://github.com/databio2022/GraphBio"
