---
name: ma-plot-visualization
description: Use when an experimental biologist needs to visualize gene expression
  changes across thousands of genes as an MA plot from a CSV expression table, without
  writing R code.
when_to_use_negative:
- The desired figure is a different visualization such as a heatmap, volcano plot,
  or chord plot — GraphBio provides 15 separate modules, and the MA plot module handles
  only MA-style log2FC-versus-expression plots.
- The task requires data preprocessing or statistical computing rather than visualization
  — GraphBio specifically focuses on facilitating generation of publication-ready
  plots instead of data preprocessing and computing.
- The user needs custom styling or non-default parameters — the documented workflow
  validates only the default-parameter render; other web-tool limitations (e.g. many
  unnecessary parameters) are what GraphBio was designed to avoid.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0092
tools:
- name: GraphBio
  role: Shiny web app whose MA plot module renders the MA plot from the csv expression
    table; accessed via web interface or self-hosted docker image
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R Shiny framework hosting the GraphBio dashboard in which the MA plot module
    runs
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: R graphics package underlying the rendered MA plot, enabling PNG export with
    default publication-ready styling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:a1e4deda253335ae3fff3bf070afa3e3132c782c29b36232f83279c1b7ac37fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ma-plot-visualization

## Summary

Generate publication-ready MA plots (log2 fold change versus average expression for each gene) from a gene expression table using GraphBio's MA plot module, a Shiny web app built on ggplot2, without programming skills.

## When to use

Use when an experimental biologist needs to visualize gene expression changes across thousands of genes as an MA plot from a CSV expression table, without writing R code.

## When NOT to use

- The desired figure is a different visualization such as a heatmap, volcano plot, or chord plot — GraphBio provides 15 separate modules, and the MA plot module handles only MA-style log2FC-versus-expression plots.
- The task requires data preprocessing or statistical computing rather than visualization — GraphBio specifically focuses on facilitating generation of publication-ready plots instead of data preprocessing and computing.
- The user needs custom styling or non-default parameters — the documented workflow validates only the default-parameter render; other web-tool limitations (e.g. many unnecessary parameters) are what GraphBio was designed to avoid.

## Inputs

- ma_example.csv demo expression table (graphbio-english/www/ directory) or a user-prepared gene expression table in csv format (one of GraphBio's four supported input formats)

## Outputs

- ma_plot.png — MA plot rendered with GraphBio's default ggplot2 styling, exported at default size and resolution
- Publication-ready figure plotting log2 fold change versus average expression for each gene

## How to apply

Load the deposited demo file ma_example.csv from GraphBio's graphbio-english/www/ directory, or prepare your own expression table in Excel following the reference example files; GraphBio accepts csv as one of its four supported input formats. Feed the expression table into GraphBio's MA plot module through its web interface (http://www.graphbio1.com/en/) or a self-hosted Shiny instance. Render the MA plot with all module parameters left at their defaults, which plots log2 fold change versus average expression for each gene. Export the rendered plot — built on ggplot2 within the Shiny dashboard — as a PNG image at the default size and resolution. The rationale is that default figures in GraphBio are generally ideal for publication, so no parameter tuning is required to meet publication-readiness criteria.

## Related tools

- **GraphBio** (Shiny web app whose MA plot module renders the MA plot from the csv expression table; accessed via web interface or self-hosted docker image) — https://github.com/databio2022/GraphBio
- **Shiny** (R Shiny framework hosting the GraphBio dashboard in which the MA plot module runs) — https://github.com/databio2022/GraphBio
- **ggplot2** (R graphics package underlying the rendered MA plot, enabling PNG export with default publication-ready styling)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual && docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- The exported ma_plot.png renders successfully with GraphBio's default styling, satisfying the criterion that default figures are publication-ready.
- The plotted axes show log2 fold change (x or y) versus average expression for each gene, matching the MA plot definition in the workflow.
- The input file resolves to the deposited demo file ma_example.csv located in the graphbio-english/www/ directory of the repository.
- All MA plot module parameters are left at their defaults during the render — no parameter customization should appear in the run.
- The output is a PNG image at the default size and resolution exported from the ggplot2-based Shiny dashboard.

## Limitations

- No version number was stated in the article for GraphBio (the README separately references docker tags v2.2.7-manual and v2.2.5-en-manual for Chinese and English versions).
- No validation, benchmarking, or user evaluation of GraphBio is reported in the article.
- GraphBio focuses only on visualization, not on data preprocessing or computing; users must prepare data themselves via Excel based on reference example files.

## Evidence

- [abstract] GraphBio provides MA plots as one of its visualization modules: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [intro] Users can prepare data from example files; default figures are publication-ready: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [readme] Demo file ma_example.csv is provided for the MA plot module: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [readme] ma_example.csv is the demo input for MA plot: "9. ma_example.csv for MA plot"
- [abstract] GraphBio enables biologists without programming skills to get publication-ready figures: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
