---
name: omics-network-plot-generation
description: Use when the input is a correlation matrix deposited as a tabular csv/txt/xls/xlsx
  file (e.g. corr_net.csv) and the goal is a publication-ready network visualization
  of pairwise relationships between omics features.
when_to_use_negative:
- Input is an expression table, DE result table, or GO term table rather than a correlation-style
  pairwise matrix — those belong to heatmap, volcano, MA, or dotplot modules
- The task requires statistical computation or data preprocessing (e.g. computing
  the correlations themselves) — GraphBio explicitly focuses on generating plots,
  not preprocessing or computing
- A different visualization is needed from GraphBio's other 14 modules (e.g. Venn
  diagram needs venn_example.csv, chord plot needs chord_example.csv)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web application providing the 'network plots' module among its 15 visualization
    analysis methods; accepts the correlation matrix and produces the publication-ready
    figure
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R framework underlying the GraphBio web app through which the network plot
    module is accessed
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: R plotting package that serves as GraphBio's rendering backend for producing
    the network plot figure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:34dc00dc2d77b7a22237e212a9a4c52a9a979971dd966aa2961ab67378dbf272
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

# omics-network-plot-generation

## Summary

Generates a publication-ready network plot in GraphBio's R Shiny dashboard from a correlation-matrix CSV (corr_net.csv), rendered via the ggplot2-based plotting backend and exportable as PDF/PNG/JPEG/TIFF.

## When to use

Use when the input is a correlation matrix deposited as a tabular csv/txt/xls/xlsx file (e.g. corr_net.csv) and the goal is a publication-ready network visualization of pairwise relationships between omics features.

## When NOT to use

- Input is an expression table, DE result table, or GO term table rather than a correlation-style pairwise matrix — those belong to heatmap, volcano, MA, or dotplot modules
- The task requires statistical computation or data preprocessing (e.g. computing the correlations themselves) — GraphBio explicitly focuses on generating plots, not preprocessing or computing
- A different visualization is needed from GraphBio's other 14 modules (e.g. Venn diagram needs venn_example.csv, chord plot needs chord_example.csv)

## Inputs

- corr_net.csv — correlation matrix (tabular csv/txt/xls/xlsx) for the network plot module
- User-prepared correlation matrix formatted in Excel against GraphBio's reference example files

## Outputs

- Publication-ready network plot rendered by the ggplot2-based backend
- Downloadable figure file in PDF, PNG, JPEG, or TIFF with customizable size and resolution

## How to apply

Load the demo correlation matrix corr_net.csv (placed in graphbio-english/www/ per the README) into the 'network plots' module of GraphBio, a Shiny web app built with bs4Dash. The module expects a tabular csv/txt/xls/xlsx input; corr_net.csv demonstrates that the network plot consumes a correlation-matrix structure rather than an expression table. Render the plot with GraphBio's ggplot2-based backend using the module's default parameters, since 'the default figures are generally ideal for publication'. Optionally adjust default display parameters such as colors and node/label presentation while keeping publication-ready styling. Export the rendered figure in PDF, PNG, JPEG, or TIFF with customizable size and resolution. GraphBio deliberately focuses on visualization generation, not data preprocessing or computing, so the matrix must be prepared beforehand in Excel following the reference example files.

## Related tools

- **GraphBio** (Shiny web application providing the 'network plots' module among its 15 visualization analysis methods; accepts the correlation matrix and produces the publication-ready figure) — https://github.com/databio2022/GraphBio
- **Shiny** (R framework underlying the GraphBio web app through which the network plot module is accessed) — https://github.com/databio2022/GraphBio
- **ggplot2** (R plotting package that serves as GraphBio's rendering backend for producing the network plot figure)

## Examples

```
docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- Output must be a rendered network plot image generated specifically from corr_net.csv
- Figure must match the module's default publication-ready styling (GraphBio states default figures are generally ideal for publication)
- Figure must be downloadable successfully in one of the supported export formats (PDF, PNG, JPEG, TIFF) with customizable size and resolution
- Input file must be in one of GraphBio's supported formats (csv/txt/xls/xlsx) and load without error in the network plots module
- Deployed app must match the GraphBio version tag used (e.g. databio2022/graphbio:v2.2.5-en-manual for the English version)

## Limitations

- GraphBio focuses on plot generation only — it does not handle data preprocessing or computing, so the correlation matrix must be produced upstream
- No validation, benchmarking, or user evaluation of GraphBio is reported in the article
- No version number is stated in the article itself (README gives docker tags v2.2.7-manual / v2.2.5-en-manual) and no license information is provided in the paper text
- Default figure styling is preferred for publication; heavy customization may depart from the validated publication-ready defaults

## Evidence

- [readme] README demo-data mapping: "5. corr_net.csv for network plot"
- [methods] network plots among the 15 modules: "GraphBio provides 15 visualization analysis modules, including heatmap, volcano plots, MA plots, network plots"
- [intro] data preparation via Excel using reference example files: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
- [abstract] GraphBio is a Shiny web app for omics visualization: "we present GraphBio, a shiny web app to easily perform visualization analysis for omics data"
