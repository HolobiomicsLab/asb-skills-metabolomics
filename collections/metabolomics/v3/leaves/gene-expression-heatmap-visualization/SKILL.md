---
name: gene-expression-heatmap-visualization
description: Use when a gene expression or omics matrix (e.g. heatmap_test.csv) with
  a matching sample-annotation file (e.g. group_info.csv) must be rendered as a clustered,
  group-annotated heatmap figure.
when_to_use_negative:
- The task is differential-expression visualization rather than expression clustering
  — GraphBio routes that to its volcano plot module with volcano_example.csv inputs
- The goal is data preprocessing or statistical computing rather than visualization
  — GraphBio 'specifically focuses on facilitating the generation of publication-ready
  plots instead of data preprocessing and computing'
- The input matrix has no matching sample-group annotation file, so sample group annotation
  cannot be applied
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web app whose heatmap module wraps the workflow; demo files heatmap_test.csv
    and group_info.csv are placed in graphbio-english/www/
  repo: https://github.com/databio2022/GraphBio
- name: pheatmap
  role: Underlying R package GraphBio's heatmap module is built on; performs hierarchical
    clustering of rows/columns and draws the annotated heatmap
- name: Shiny
  role: R framework hosting the GraphBio web app through which users upload Excel-prepared
    inputs
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:a748d197ec5657ed03486b0ab61f0c2512864351181e6caccd3347d1151c9da0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - pheatmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gene-expression-heatmap-visualization

## Summary

Generates a publication-ready, hierarchically clustered heatmap from an omics/gene expression matrix, annotating samples by group using GraphBio's heatmap module, which is built on the R pheatmap package.

## When to use

Use when a gene expression or omics matrix (e.g. heatmap_test.csv) with a matching sample-annotation file (e.g. group_info.csv) must be rendered as a clustered, group-annotated heatmap figure.

## When NOT to use

- The task is differential-expression visualization rather than expression clustering — GraphBio routes that to its volcano plot module with volcano_example.csv inputs
- The goal is data preprocessing or statistical computing rather than visualization — GraphBio 'specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing'
- The input matrix has no matching sample-group annotation file, so sample group annotation cannot be applied

## Inputs

- Gene expression matrix CSV (e.g. heatmap_test.csv) with genes as rows and samples as columns
- Sample annotation CSV (e.g. group_info.csv) with group labels aligned to expression matrix columns

## Outputs

- Clustered, group-annotated heatmap figure exported as heatmap.png at publication-ready size and resolution

## How to apply

Load the gene expression matrix from the GraphBio demo file heatmap_test.csv (placed in the graphbio-english/www/ directory of github.com/databio2022/GraphBio) and the sample annotation file group_info.csv, aligning group labels to the expression matrix columns. Render the heatmap with pheatmap — the package GraphBio's heatmap module is built on — applying hierarchical clustering of rows and columns, the default colorblind-friendly color preset, and group annotation of samples. GraphBio's design goal is that users prepare the two input tables in Excel based on the reference example files, and the default figures are generally ideal for publication, so no fine-tuning of parameters is required. Export the figure as heatmap.png at publication-ready size and resolution. Correct application reproduces the clustered, group-annotated heatmap style of GraphBio's demo (Figure 1A), with visible cluster structure and group annotation.

## Related tools

- **GraphBio** (Shiny web app whose heatmap module wraps the workflow; demo files heatmap_test.csv and group_info.csv are placed in graphbio-english/www/) — https://github.com/databio2022/GraphBio
- **pheatmap** (Underlying R package GraphBio's heatmap module is built on; performs hierarchical clustering of rows/columns and draws the annotated heatmap)
- **Shiny** (R framework hosting the GraphBio web app through which users upload Excel-prepared inputs)

## Examples

```
library(pheatmap); pheatmap(expr, annotation_col = group_info, cluster_rows = TRUE, cluster_cols = TRUE, filename = "heatmap.png")
```

## Evaluation signals

- heatmap.png exists and is a valid PNG image at publication-ready size and resolution
- Rendered figure reproduces the clustered, group-annotated heatmap style of GraphBio's heatmap module demo (Figure 1A)
- Cluster structure (hierarchical clustering of rows and columns) is visible in the output figure
- Sample group annotation from group_info.csv is visible and correctly aligned to expression matrix columns
- Default colorblind-friendly color preset is used without manual parameter tuning

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio was reported in the article
- No license information is provided in the article; the GitHub repo lists GPL-3.0
- Input must match the reference example file format prepared in Excel; users without programming skills rely entirely on the predefined module defaults

## Evidence

- [intro] pheatmap for drawing heatmap: "ggplot2 (Wickham, 2016), pheatmap for drawing heatmap"
- [abstract] Publication-ready goal: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
- [methods] Heatmap module is one of 15 modules: "GraphBio provides 15 visualization analysis modules, including heatmap, volcano plots, MA plots"
- [readme] Demo files for the heatmap module: "1. heatmap_test.csv and group_info.csv for heatmap."
- [intro] Users prepare data in Excel from reference example files; defaults are publication-ready: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
