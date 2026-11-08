---
name: shiny-web-application-operation
description: Use when the goal is to reproduce a GraphBio visualization module (e.g.,
  network plots from a correlation matrix file like corr_net.csv) through its Shiny
  web interface rather than hand-coded R scripts.
when_to_use_negative:
- The input is not a correlation matrix (e.g., raw expression counts) — network plots
  expects corr_net.csv-style matrix input; other modules like heatmap use heatmap_test.csv
  instead
- The task requires data preprocessing or computation — GraphBio 'specifically focuses
  on facilitating the generation of' publication-ready plots, not preprocessing
- A scriptable, version-controlled pipeline is required — GraphBio is an interactive
  Shiny web app aimed at biologists without programming skills
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: The R Shiny dashboard (bs4Dash) whose network plots module ingests the correlation
    matrix and renders the figure
  repo: github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework on which the GraphBio application and its module UI are built
- name: ggplot2
  role: Plotting backend underlying GraphBio's rendered figures, including the network
    plot output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:c679636103ba0062f17736ed209b72535a0a198d981ec14e2313dd4b0296dfb9
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# shiny-web-application-operation

## Summary

Operates the GraphBio R Shiny web application (a bs4Dash dashboard) to render publication-ready visualization modules — here the network plots module — from user-prepared tabular demo data such as corr_net.csv.

## When to use

Use when the goal is to reproduce a GraphBio visualization module (e.g., network plots from a correlation matrix file like corr_net.csv) through its Shiny web interface rather than hand-coded R scripts.

## When NOT to use

- The input is not a correlation matrix (e.g., raw expression counts) — network plots expects corr_net.csv-style matrix input; other modules like heatmap use heatmap_test.csv instead
- The task requires data preprocessing or computation — GraphBio 'specifically focuses on facilitating the generation of' publication-ready plots, not preprocessing
- A scriptable, version-controlled pipeline is required — GraphBio is an interactive Shiny web app aimed at biologists without programming skills

## Inputs

- corr_net.csv — GraphBio's demo correlation matrix file for the network plots module (tabular csv/txt/xls/xlsx input)
- User-prepared correlation matrix in csv format modeled on the reference example files

## Outputs

- Rendered network plot figure in GraphBio's default publication-ready styling
- Downloadable figure exported as PDF, PNG, JPEG, or TIFF with customizable size and resolution

## How to apply

Launch GraphBio either via the public site (http://www.graphbio1.com/en/) or self-host the English version with the documented docker image `databio2022/graphbio:v2.2.5-en-manual`. Prepare the input in Excel/csv format following GraphBio's reference example files; for the network plots module the deposited demo file is corr_net.csv, a correlation matrix, consistent with the module's matrix input. Load corr_net.csv into the 'network plots' module of the Shiny dashboard and render with the module's default parameters, since 'default figures are generally ideal for publication'. GraphBio's plotting backend is ggplot2-based, so the rendered figure is a ggplot2-style network plot. Export the figure via the module's download function in PDF, PNG, JPEG, or TIFF with customizable size and resolution.

## Related tools

- **GraphBio** (The R Shiny dashboard (bs4Dash) whose network plots module ingests the correlation matrix and renders the figure) — github.com/databio2022/GraphBio
- **Shiny** (R web framework on which the GraphBio application and its module UI are built)
- **ggplot2** (Plotting backend underlying GraphBio's rendered figures, including the network plot output)

## Examples

```
docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- The rendered output is a network plot image generated from corr_net.csv, not another module's output (e.g., heatmap or volcano)
- The figure matches GraphBio's default publication-ready styling for the network plots module
- The figure can be downloaded successfully in one of the four supported formats (PDF, PNG, JPEG, TIFF)
- Default module parameters were used (or adjustments limited to colors/node-label presentation) rather than altered analysis settings
- The input file was accepted by the module, confirming csv/txt/xls/xlsx format compliance

## Limitations

- GraphBio focuses on visualization only; data preprocessing and computing are out of scope
- No validation, benchmarking, or user evaluation of GraphBio is reported in the article
- Version handling is fragmented: the Chinese-version docker tag is v2.2.7-manual while the English version is v2.2.5-en-manual, and no version number or license is stated in the article itself

## Evidence

- [abstract] we present GraphBio, a shiny web app to easily perform visualization analysis for omics data: "we present GraphBio, a shiny web app to easily perform visualization analysis for omics data"
- [methods] GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots"
- [readme] Demo data for each visualization function (files are placed in the graphbio-english/www/ directory): "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio"
- [abstract] It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
