---
name: kaplan-meier-survival-curve-estimation
description: Use when the input is a survival table with sample IDs, survival times,
  censoring status, and group labels (like surv_example.csv) and the goal is a publication-ready
  survival curve figure per group.
when_to_use_negative:
- The input omics dataset calls for one of GraphBio's other modules (e.g. heatmap,
  volcano plots, MA plots, PCA, ROC analysis) rather than survival analysis — GraphBio
  provides 15 distinct visualization modules.
- Only the raw omics expression matrix is available without per-sample survival times
  and censoring status; the module requires survival outcome data formatted like surv_example.csv.
- The goal is data preprocessing or statistical modeling rather than generating a
  visualization figure — GraphBio specifically focuses on facilitating the generation
  of publication-ready plots instead of data preprocessing and computing.
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GraphBio
  role: Shiny web app whose survival analysis module this skill reproduces; provides
    the surv_example.csv demo file and default publication-ready figure styling
  repo: https://github.com/databio2022/GraphBio
- name: survminer
  role: R package wrapping survfit for fitting Kaplan-Meier curves and rendering ggsurvplot-style
    curves with at-risk tables and legends
- name: Shiny
  role: R web framework underlying GraphBio's interactive survival analysis module
- name: ggplot2
  role: R plotting package underlying the ggsurvplot-style rendering of the survival
    curves
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:a5aa6cf7d997dfb4fea17783f7e37e8f946a960eb1704941874e268c3f1ea43d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  - survminer
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# kaplan-meier-survival-curve-estimation

## Summary

Generates per-group Kaplan-Meier survival curves with at-risk tables from a tabular survival input, reproducing the survival analysis module of GraphBio, which wraps the survminer R package for rendering publication-ready figures.

## When to use

Use when the input is a survival table with sample IDs, survival times, censoring status, and group labels (like surv_example.csv) and the goal is a publication-ready survival curve figure per group.

## When NOT to use

- The input omics dataset calls for one of GraphBio's other modules (e.g. heatmap, volcano plots, MA plots, PCA, ROC analysis) rather than survival analysis — GraphBio provides 15 distinct visualization modules.
- Only the raw omics expression matrix is available without per-sample survival times and censoring status; the module requires survival outcome data formatted like surv_example.csv.
- The goal is data preprocessing or statistical modeling rather than generating a visualization figure — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing.

## Inputs

- surv_example.csv — tabular survival data with sample IDs, survival times, censoring status, and group labels

## Outputs

- survival_curves.png — Kaplan-Meier survival curve figure with per-group curves, at-risk table, and legend

## How to apply

Load the tabular survival input file surv_example.csv, which contains sample IDs, survival times, censoring status, and group labels. Fit Kaplan-Meier survival curves per group using survfit as wrapped by the survminer R package, the backend GraphBio uses for its survival analysis module. Render the curves with at-risk tables and legends via survminer's ggsurvplot-style presentation, matching the default output of GraphBio's survival module, which is one of its 15 popular visualization analysis methods for omics data. Export the figure as survival_curves.png, since GraphBio plots are downloadable as PDF/PNG/JPEG/TIFF and PNG is retained here. GraphBio's design goal is that default figures are generally ideal for publication, so no manual post-styling is expected; users prepare data via Excel based on reference example files.

## Related tools

- **GraphBio** (Shiny web app whose survival analysis module this skill reproduces; provides the surv_example.csv demo file and default publication-ready figure styling) — https://github.com/databio2022/GraphBio
- **survminer** (R package wrapping survfit for fitting Kaplan-Meier curves and rendering ggsurvplot-style curves with at-risk tables and legends)
- **Shiny** (R web framework underlying GraphBio's interactive survival analysis module)
- **ggplot2** (R plotting package underlying the ggsurvplot-style rendering of the survival curves)

## Examples

```
km <- survfit(Surv(time, status) ~ group, data = read.csv("surv_example.csv")); ggsurvplot(km, data = read.csv("surv_example.csv"), risk.table = TRUE)
```

## Evaluation signals

- survival_curves.png exists and is a valid PNG file
- Expert review confirms the figure shows per-group Kaplan-Meier curves with a risk table consistent with the survminer-based GraphBio survival module
- Input file parses as a survival table with sample IDs, survival times, censoring status, and group labels matching surv_example.csv's schema
- Figure presentation matches GraphBio defaults (at-risk table and legend present), which are generally ideal for publication
- Group assignments in the figure match the group labels column of surv_example.csv without reordering or dropping groups

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio's visualization modules was reported in the paper.
- GraphBio focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing, so upstream survival-data cleaning must be done elsewhere.
- No version number stated for GraphBio in the article text (the README references Docker tags such as v2.2.7-manual for the Chinese and v2.2.5-en-manual for the English deployment).

## Evidence

- [abstract] GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [methods] 13. surv_example.csv for survival curves: "13. surv_example.csv for survival curves"
- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio.: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [readme] 13. surv_example.csv for survival curves (README demo data listing): "13. surv_example.csv for survival curves"
