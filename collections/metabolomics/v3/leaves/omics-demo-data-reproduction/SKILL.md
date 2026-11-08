---
name: omics-demo-data-reproduction
description: Use when the goal is to verify or reproduce a GraphBio visualization
  module's output, or when a user needs a template input file matching a module's
  expected schema before running their own omics data.
when_to_use_negative:
- The user's own omics data is already formatted and the goal is analysis, not validating
  or templating a module's expected input/output
- The analysis requires data preprocessing or computation — GraphBio is explicitly
  not designed for that, focusing only on plot generation
- The desired plot type is not among the 15 modules GraphBio provides (e.g., other
  visualization methods not yet integrated)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: GraphBio
  role: Source of the demo datasets and the reference default publication-ready figures
    to reproduce
  repo: https://github.com/databio2022/GraphBio
- name: pROC
  role: R package used to fit and plot the ROC curve from roc_example.csv, annotating
    the AUC value as in GraphBio's module
- name: Shiny
  role: R framework in which GraphBio is implemented; demo files are served from the
    Shiny app's www directory
  repo: https://github.com/databio2022/GraphBio
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:2b738e36b50ae9ebc7e7fb72b823ddf1df40fac7c48ea105892b67c6605eeefc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - pROC
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# omics-demo-data-reproduction

## Summary

Reproduces a GraphBio visualization module's default publication-ready figure by loading its shipped demo dataset (e.g., roc_example.csv, surv_example.csv, venn_example.csv from graphbio-english/www/) and re-running the underlying R method (e.g., pROC's roc()) so the output can be validated against the module's reference figure.

## When to use

Use when the goal is to verify or reproduce a GraphBio visualization module's output, or when a user needs a template input file matching a module's expected schema before running their own omics data.

## When NOT to use

- The user's own omics data is already formatted and the goal is analysis, not validating or templating a module's expected input/output
- The analysis requires data preprocessing or computation — GraphBio is explicitly not designed for that, focusing only on plot generation
- The desired plot type is not among the 15 modules GraphBio provides (e.g., other visualization methods not yet integrated)

## Inputs

- Demo CSV file from graphbio-english/www/ (e.g., roc_example.csv, surv_example.csv, venn_example.csv, heatmap_test.csv, volcano_example.csv)
- group_info.csv when the module requires group labels (e.g., heatmap)
- Reference default figure from the GraphBio module for comparison

## Outputs

- Reproduced figure file (roc_curve.pdf or PNG/JPEG/TIFF equivalent)
- Computed statistics such as the AUC value from pROC's roc()
- Schema-verified template data frame showing expected column structure for the module

## How to apply

Locate the demo dataset for the target module in the GraphBio repository's graphbio-english/www/ directory as listed in the README (e.g., roc_example.csv for ROC curve, surv_example.csv for survival curves, venn_example.csv for Venn plot). Load the CSV into R as a data frame with the expected columns — for the ROC module this means a predictor-values column plus group labels. Run the underlying R method directly, e.g., fit the curve with pROC (Robin et al., 2011) by calling roc() on the response labels and predictor column. Render the figure with the package's plot function, annotating the computed AUC in the legend as GraphBio's default module figure does, and export at the module's supported formats (PDF/PNG/JPEG/TIFF with customizable size and resolution). Validate by confirming the exported figure contains the same elements (single curve, AUC legend entry) and matching values as the module's default demo output.

## Related tools

- **GraphBio** (Source of the demo datasets and the reference default publication-ready figures to reproduce) — https://github.com/databio2022/GraphBio
- **pROC** (R package used to fit and plot the ROC curve from roc_example.csv, annotating the AUC value as in GraphBio's module)
- **Shiny** (R framework in which GraphBio is implemented; demo files are served from the Shiny app's www directory) — https://github.com/databio2022/GraphBio

## Examples

```
library(pROC); d <- read.csv('roc_example.csv'); r <- roc(d$group, d$predictor); plot(r, print.auc=TRUE); dev.copy2pdf(file='roc_curve.pdf')
```

## Evaluation signals

- roc_curve.pdf exists and renders a single ROC curve
- The plot includes a legend entry with an AUC value
- The computed AUC matches the AUC shown in the GraphBio ROC module's default figure for the same demo input
- Loaded data frame contains both predictor values and group labels required by roc()
- For other modules, the demo file name matches the README's numbered list entry (e.g., surv_example.csv ↔ survival curves)

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio modules is reported in the article, so reproduction relies on visual comparison with the default figure
- No version number or license is stated in the article text; the demo file location may differ across app versions (Chinese vs English docker images such as v2.2.7-manual and v2.2.5-en-manual)
- GraphBio focuses on plot generation, not data preprocessing, so demo files only cover visualization input formats

## Evidence

- [readme] Demo data for each visualization function (files are placed in the graphbio-english/www/ directory): "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [readme] roc_example.csv for ROC curve: "11. roc_example.csv for ROC curve"
- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
- [abstract] we present GraphBio, a shiny web app to easily perform visualization analysis for omics data: "we present GraphBio, a shiny web app to easily perform visualization analysis for omics data"
- [intro] The default figures are generally ideal for publication: "The default figures are generally ideal for publication"
