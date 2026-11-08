---
name: shiny-web-app-plot-rendering
description: Use when an experimental biologist holds an omics data table (e.g., csv
  gene expression values) and needs a publication-ready plot such as an MA plot without
  writing R code.
when_to_use_negative:
- The task requires data preprocessing or computation rather than plot generation
  — GraphBio specifically focuses on facilitating the generation of publication-ready
  plots instead of data preprocessing and computing
- The desired plot type is not among GraphBio's 15 visualization modules (the paper
  notes additional methods were only planned as future work)
- Highly customized figure styling is required beyond the module's parameters — the
  skill relies on GraphBio's default styling being sufficient for publication
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: The Shiny web application whose module renders the plot from the user's csv
    table with default parameters
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework underlying the GraphBio dashboard through which the user loads
    data and exports the plot
- name: ggplot2
  role: Graphics library on which the rendered figures within the Shiny dashboard
    are built
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:bb26a31249fed96a613299962b2f7ad3a53947cf595c9759fe54b160e2bca387
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# shiny-web-app-plot-rendering

## Summary

Generate publication-ready omics visualization plots (e.g., MA plots, heatmaps, volcano plots) by loading a csv-formatted data table into GraphBio's Shiny web app and rendering through the matching ggplot2-based module with default parameters.

## When to use

Use when an experimental biologist holds an omics data table (e.g., csv gene expression values) and needs a publication-ready plot such as an MA plot without writing R code.

## When NOT to use

- The task requires data preprocessing or computation rather than plot generation — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing
- The desired plot type is not among GraphBio's 15 visualization modules (the paper notes additional methods were only planned as future work)
- Highly customized figure styling is required beyond the module's parameters — the skill relies on GraphBio's default styling being sufficient for publication

## Inputs

- csv-format omics data table prepared in Excel following the module's reference example file (e.g., ma_example.csv for the MA plot)
- Demo reference files from the graphbio-english/www/ directory (e.g., ma_example.csv, volcano_example.csv, venn_example.csv)

## Outputs

- Publication-ready figure exported as a PNG image at default size and resolution (e.g., ma_plot.png plotting log2 fold change versus average expression per gene)

## How to apply

First select the GraphBio module matching the desired plot type from its 15 visualization analysis methods (heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots, four quadrant diagrams, Venn diagrams, cumulative distribution curves, PCA, survival analysis, ROC analysis, correlation analysis, text cluster analysis). Prepare the input data in Excel based on the reference demo file for that module placed in graphbio-english/www/ (e.g., ma_example.csv for the MA plot), since csv is one of GraphBio's four supported input formats. Feed the table into the module and leave all module parameters at their defaults — GraphBio's design goal is that default figures are generally ideal for publication, and for MA plots the module plots log2 fold change versus average expression for each gene. Export the rendered figure, built on ggplot2 within the Shiny dashboard, as a PNG image at the default size and resolution. If the public site (http://www.graphbio1.com/en/) is unavailable, self-host the English version via the docker image databio2022/graphbio:v2.2.5-en-manual, mapping port 80 to the container's 3838.

## Related tools

- **GraphBio** (The Shiny web application whose module renders the plot from the user's csv table with default parameters) — https://github.com/databio2022/GraphBio
- **Shiny** (R web framework underlying the GraphBio dashboard through which the user loads data and exports the plot)
- **ggplot2** (Graphics library on which the rendered figures within the Shiny dashboard are built)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual; docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- The exported PNG (e.g., ma_plot.png) renders successfully with GraphBio's default styling
- For the MA plot module, the figure axes show log2 fold change versus average expression for each gene
- The input csv matches the structure of the module's reference demo file from graphbio-english/www/ (e.g., ma_example.csv)
- All module parameters were left at their defaults, satisfying the criterion that default figures are publication-ready
- The chosen module is one of GraphBio's 15 supported visualization analysis methods

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio was reported in the paper
- GraphBio handles plot generation only — data preprocessing and computing are out of scope
- Additional visualization methods were not yet integrated at publication time (listed as future work)
- No version number or formal citation is stated in the discussion section; the README separately identifies docker versions v2.2.7-manual (Chinese) and v2.2.5-en-manual (English)

## Evidence

- [abstract] GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [intro] we developed an online web application called GraphBio using a shiny framework in R software: "we developed an online web application called GraphBio using a shiny framework in R software"
- [intro] The default figures are generally ideal for publication: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [discussion] Experimental biologists can easily perform online analysis and get publication-ready plots via accessing the website: "Experimental biologists can easily perform online analysis and get publication-ready plots via accessing the website"
- [readme] Demo data for each visualization function (files are placed in the graphbio-english/www/ directory): "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [readme] ma_example.csv for MA plot: "9. ma_example.csv for MA plot"
