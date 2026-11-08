---
name: auc-estimation-and-annotation
description: Use when binary classification performance for an omics predictor must
  be visualized as an ROC curve with an AUC legend entry, especially by users without
  programming skills.
when_to_use_negative:
- The task is data preprocessing or statistical computing rather than visualization
  — GraphBio 'specifically focuses on facilitating the generation of publication-ready
  plots instead of data preprocessing and computing'
- The outcome is continuous or multi-class rather than the binary response labels
  required for pROC's roc()
- A different plot type is needed (e.g., survival curves use surv_example.csv, cluster
  analysis uses text_cluster_example.csv — GraphBio's modules are function-specific)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3577
tools:
- name: pROC
  role: R package used to fit and draw the ROC curve and compute the AUC annotated
    in the legend (Robin et al., 2011)
- name: GraphBio
  role: Shiny web app providing a no-code ROC analysis module with demo input roc_example.csv
    and publication-ready default figures
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework on which the GraphBio ROC module is built, enabling use without
    programming skills
  repo: https://github.com/databio2022/GraphBio
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:408cadeab282aa8322614c61d5dd21cc6850502d87db3ae7403fd62c7ed21ea1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - pROC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# auc-estimation-and-annotation

## Summary

Generate receiver operating characteristic (ROC) curves for omics data and annotate them with the computed AUC value, using the pROC package or the no-code GraphBio Shiny web app with its demo input roc_example.csv, producing publication-ready figures.

## When to use

Use when binary classification performance for an omics predictor must be visualized as an ROC curve with an AUC legend entry, especially by users without programming skills.

## When NOT to use

- The task is data preprocessing or statistical computing rather than visualization — GraphBio 'specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing'
- The outcome is continuous or multi-class rather than the binary response labels required for pROC's roc()
- A different plot type is needed (e.g., survival curves use surv_example.csv, cluster analysis uses text_cluster_example.csv — GraphBio's modules are function-specific)

## Inputs

- roc_example.csv (demo file in graphbio-english/www/): predictor values plus group/response labels as a CSV data frame
- User-prepared two-column CSV with predictor values and binary group labels (formatted by Excel based on the reference example files)

## Outputs

- ROC curve plot with an AUC value annotated in the legend (roc_curve.pdf)
- Exportable figure in PDF/PNG/JPEG/TIFF at customizable size and resolution

## How to apply

Load the demo dataset roc_example.csv from the GraphBio repository (graphbio-english/www/) as a data frame containing predictor values and group labels. Fit the ROC curve with the pROC package (Robin et al., 2011) by calling roc() on the response labels and the predictor column, then plot with pROC's plot function, annotating the computed AUC value in the legend as in GraphBio's default module figure. Alternatively, non-programmers can upload the data to the GraphBio Shiny web app (http://www.graphbio1.com/en/), which wraps pROC among its 15 visualization modules and produces default figures that are generally ideal for publication. Export the result as roc_curve.pdf, matching GraphBio's downloadable plot formats (PDF/PNG/JPEG/TIFF) with customizable size and resolution.

## Related tools

- **pROC** (R package used to fit and draw the ROC curve and compute the AUC annotated in the legend (Robin et al., 2011))
- **GraphBio** (Shiny web app providing a no-code ROC analysis module with demo input roc_example.csv and publication-ready default figures) — https://github.com/databio2022/GraphBio
- **Shiny** (R web framework on which the GraphBio ROC module is built, enabling use without programming skills) — https://github.com/databio2022/GraphBio

## Examples

```
Rscript -e 'df <- read.csv("roc_example.csv"); r <- roc(df$group, df$predictor); plot(r, legacy.axes=TRUE); legend("bottomright", sprintf("AUC=%.3f", auc(r)))'  # or upload roc_example.csv to http://www.graphbio1.com/en/
```

## Evaluation signals

- roc_curve.pdf exists and renders a single ROC curve
- The plot contains an AUC legend entry whose value matches the AUC computed from the same predictor and group labels
- The exported figure reproduces the default figure shown by the GraphBio ROC module for the demo input roc_example.csv
- Output is in a downloadable format (PDF/PNG/JPEG/TIFF) with the chosen size and resolution settings
- When run via GraphBio, the figure meets the paper's 'publication-ready' default standard

## Limitations

- ROC analysis in base R via pROC requires good programming skills; the no-code path depends on the GraphBio web service or a self-hosted Docker image
- GraphBio was published without validation, benchmarking, or user evaluation, and no version number or license is stated in the article text
- GraphBio's scope is plot generation, not data preprocessing or computing; input must be prepared in Excel following the reference example files

## Evidence

- [intro] and pROC for drawing receiver operating characteristic (ROC) curves (Robin et al., 2011): "and pROC for drawing receiver operating characteristic (ROC) curves (Robin et al., 2011)"
- [readme] 11. roc_example.csv for ROC curve: "11. roc_example.csv for ROC curve"
- [abstract] It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
- [intro] GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing: "GraphBio specifically focuses on facilitating the generation of publication-ready"
- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
