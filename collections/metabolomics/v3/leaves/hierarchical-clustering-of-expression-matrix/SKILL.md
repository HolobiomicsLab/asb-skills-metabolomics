---
name: hierarchical-clustering-of-expression-matrix
description: Use when input is a gene expression matrix CSV (e.g. heatmap_test.csv)
  plus a sample annotation CSV (e.g. group_info.csv) and the goal is a clustered,
  group-annotated heatmap figure.
when_to_use_negative:
- The goal is not a clustered heatmap but another GraphBio-supported plot (e.g. volcano
  plot, MA plot, chord plot) — use the corresponding module/example files instead.
- Data preprocessing, differential analysis, or computation is needed — GraphBio specifically
  focuses on visualization generation, not data preprocessing or computing.
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web app whose heatmap module wraps pheatmap; users upload Excel-prepared
    CSVs to get publication-ready clustered heatmaps
  repo: https://github.com/databio2022/GraphBio
- name: pheatmap
  role: R package used to draw the heatmap with hierarchical clustering of rows/columns
    and group annotation
- name: Shiny
  role: R framework in which GraphBio's heatmap module is implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:6ba5ca7ffe8a9452f80172df4964702742315794002e4c632b85db8a211fcfbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - pheatmap
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hierarchical-clustering-of-expression-matrix

## Summary

Render a publication-ready, group-annotated clustered heatmap from a gene expression matrix using pheatmap (the engine behind GraphBio's heatmap module), with hierarchical clustering of rows and columns and sample group annotation.

## When to use

Use when input is a gene expression matrix CSV (e.g. heatmap_test.csv) plus a sample annotation CSV (e.g. group_info.csv) and the goal is a clustered, group-annotated heatmap figure.

## When NOT to use

- The goal is not a clustered heatmap but another GraphBio-supported plot (e.g. volcano plot, MA plot, chord plot) — use the corresponding module/example files instead.
- Data preprocessing, differential analysis, or computation is needed — GraphBio specifically focuses on visualization generation, not data preprocessing or computing.

## Inputs

- Gene expression matrix CSV (heatmap_test.csv from graphbio-english/www/)
- Sample group annotation CSV (group_info.csv) with group labels aligned to matrix columns

## Outputs

- Clustered, group-annotated heatmap figure (heatmap.png) at publication-ready size and resolution

## How to apply

Load the gene expression matrix heatmap_test.csv (from graphbio-english/www/ in the GraphBio repository) into R and the sample annotation file group_info.csv, aligning group labels to the expression matrix columns. Render with pheatmap, the package GraphBio's heatmap module is built on, applying hierarchical clustering of both rows and columns, the default colorblind-friendly color preset, and group annotation of samples. Data is expected to be prepared in Excel following the reference example files shipped with GraphBio. Export the figure as heatmap.png at publication-ready size and resolution; GraphBio's design goal is that default parameters already yield figures ideal for publication.

## Related tools

- **GraphBio** (Shiny web app whose heatmap module wraps pheatmap; users upload Excel-prepared CSVs to get publication-ready clustered heatmaps) — https://github.com/databio2022/GraphBio
- **pheatmap** (R package used to draw the heatmap with hierarchical clustering of rows/columns and group annotation)
- **Shiny** (R framework in which GraphBio's heatmap module is implemented)

## Examples

```
pheatmap(expr_matrix, annotation_col = group_info, cluster_rows = TRUE, cluster_cols = TRUE, filename = "heatmap.png")
```

## Evaluation signals

- Rendered figure reproduces the clustered, group-annotated heatmap style of GraphBio's heatmap module demo (Figure 1A).
- Hierarchical cluster structure is visible on both rows and columns of the heatmap.
- Group annotation bar from group_info.csv is visible and correctly aligned to expression matrix columns.
- Figure is exported as heatmap.png at publication-ready size and resolution.
- Default preset is used with no unnecessary parameters, consistent with GraphBio's design goal.

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio's modules is reported.
- No version number stated for the heatmap module (Docker tags v2.2.7-manual / v2.2.5-en-manual exist for the app).
- GraphBio focuses on figure generation, not data preprocessing or computing — the expression matrix must already be analysis-ready.

## Evidence

- [intro] pheatmap is the heatmap engine: "they were usually published as R packages, such as ggplot2 (Wickham, 2016), pheatmap for drawing heatmap"
- [intro] design goal of default publication-ready figures: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [methods] heatmap example for gene expression: "Example 1Heatmap for gene expression profiles."
- [readme] demo files for heatmap module: "1. heatmap_test.csv and group_info.csv for heatmap."
- [intro] GraphBio focuses on visualization not preprocessing: [evidence span withheld — anchor status: not_found]
