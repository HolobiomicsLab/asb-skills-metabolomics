---
name: time-to-event-survival-data-handling
description: Use when the input is a CSV of survival times, censoring status, and
  group labels (like GraphBio's surv_example.csv) and the goal is publication-ready
  per-group Kaplan-Meier survival curves.
when_to_use_negative:
- The input is a gene-expression or other omics matrix intended for one of GraphBio's
  other 14 modules (e.g., heatmap, volcano plot, MA plot, Venn diagram) rather than
  time-to-event data with censoring status.
- The analysis target is not survival/time-to-event visualization — e.g., ROC curves,
  PCA, or correlation analysis, which GraphBio exposes as separate modules.
- No group labels or censoring status are present, so Kaplan-Meier curves per group
  cannot be fitted from the file.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3391
tools:
- name: GraphBio
  role: Shiny web app whose survival analysis module defines the reference output
    (surv_example.csv demo data, survminer-based Kaplan-Meier rendering, downloadable
    PNG/PDF/JPEG/TIFF plots)
  repo: https://github.com/databio2022/GraphBio
- name: survminer
  role: R package wrapped by GraphBio for fitting Kaplan-Meier curves (survfit) and
    rendering ggsurvplot-style figures with risk tables and legends
- name: Shiny
  role: R framework underlying the GraphBio web app that hosts the survival analysis
    module
- name: ggplot2
  role: Underlying R plotting grammar used by GraphBio's visualization modules
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:4c709a088615a6a773c6cb1066fa0d3abf5d9fef02be1f320b1f217a15eb8f5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  - survminer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# time-to-event-survival-data-handling

## Summary

Generates Kaplan-Meier survival curves with at-risk tables from tabular time-to-event data, mirroring the survival analysis module of the GraphBio Shiny web app, which wraps the survminer R package.

## When to use

Use when the input is a CSV of survival times, censoring status, and group labels (like GraphBio's surv_example.csv) and the goal is publication-ready per-group Kaplan-Meier survival curves.

## When NOT to use

- The input is a gene-expression or other omics matrix intended for one of GraphBio's other 14 modules (e.g., heatmap, volcano plot, MA plot, Venn diagram) rather than time-to-event data with censoring status.
- The analysis target is not survival/time-to-event visualization — e.g., ROC curves, PCA, or correlation analysis, which GraphBio exposes as separate modules.
- No group labels or censoring status are present, so Kaplan-Meier curves per group cannot be fitted from the file.

## Inputs

- surv_example.csv-style tabular file with sample IDs, survival times, censoring status, and group labels

## Outputs

- survival_curves.png — Kaplan-Meier survival curves per group with at-risk (risk) table and legend

## How to apply

Load the tabular survival input (e.g., surv_example.csv, with columns for sample IDs, survival times, censoring status, and group labels) in R. Fit Kaplan-Meier survival curves per group using survfit as wrapped by the survminer R package, which is what the GraphBio survival module uses internally. Render the curves in the survminer ggsurvplot style, including at-risk tables and legends so the presentation matches GraphBio's default output. Export the figure as survival_curves.png (GraphBio plots are downloadable as PDF/PNG/JPEG/TIFF; PNG is retained here). Validate that survival_curves.png exists and is a valid PNG, and confirm the figure shows per-group Kaplan-Meier curves with a risk table consistent with the survminer-based GraphBio module.

## Related tools

- **GraphBio** (Shiny web app whose survival analysis module defines the reference output (surv_example.csv demo data, survminer-based Kaplan-Meier rendering, downloadable PNG/PDF/JPEG/TIFF plots)) — https://github.com/databio2022/GraphBio
- **survminer** (R package wrapped by GraphBio for fitting Kaplan-Meier curves (survfit) and rendering ggsurvplot-style figures with risk tables and legends)
- **Shiny** (R framework underlying the GraphBio web app that hosts the survival analysis module)
- **ggplot2** (Underlying R plotting grammar used by GraphBio's visualization modules)

## Examples

```
fit <- survfit(Surv(time, status) ~ group, data = read.csv("surv_example.csv")); ggsurvplot(fit, risk.table = TRUE); ggsave("survival_curves.png")
```

## Evaluation signals

- survival_curves.png exists and is a valid PNG file.
- The figure displays per-group Kaplan-Meier survival curves, one curve per group label in the input CSV.
- An at-risk (risk) table is present beneath the curves, consistent with survminer ggsurvplot defaults as used by GraphBio.
- Legend labels correspond to the group labels in surv_example.csv, matching GraphBio's default presentation.
- The rendered output is reproducible against GraphBio's own survival module demo (surv_example.csv ships with the app in graphbio-english/www/).

## Limitations

- GraphBio focuses on facilitating publication-ready plot generation rather than data preprocessing or computing, so upstream survival-data cleaning is out of scope.
- The article reports no validation, benchmarking, or user evaluation of GraphBio, and states no version number or license in the paper (the README ships Docker tags v2.2.7-manual for the Chinese version and v2.2.5-en-manual for the English version).
- Default figures are 'generally ideal for publication' per the paper, implying limited fine-grained customization compared with calling survminer/ggplot2 directly.

## Evidence

- [abstract] cumulative distribution curves, PCA, survival analysis, ROC analysis, correlation analysis and text cluster analysis: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [readme] 13. surv_example.csv for survival curves: "13. surv_example.csv for survival curves"
- [discussion] It includes 15 popular visualization analysis methods, such as heatmaps, volcano plots, and MA plots.: "It includes 15 popular visualization analysis methods, such as heatmaps, volcano plots, and MA plots."
- [methods] surv_example.csv for survival curves: "13. surv_example.csv for survival curves"
