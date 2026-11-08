---
name: volcano-plot-threshold-classification
description: Use when a differential-expression matrix with geneID, log2 fold change,
  p/padj significance, and optional label columns needs to be visualized as a volcano
  plot with threshold-based up/down classification for publication.
when_to_use_negative:
- The goal is data preprocessing or statistical computation of differential expression
  — GraphBio explicitly focuses on facilitating figure generation, not preprocessing
  and computing
- The input lacks the required geneID / log2 fold change / p or padj / label column
  structure and cannot be reshaped per the reference example files
- The figure needed is a MA plot, four quadrant diagram, or other module output rather
  than a threshold-classified volcano plot
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web app whose 'Volcano plot' module performs the threshold classification
    and figure rendering
  repo: github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework underlying GraphBio, used to load the CSV input and drive
    the module
- name: ggplot2
  role: R plotting library used by the module to render the volcano plot with default
    colors, point size, and label size
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:1b041b4e0b9e3c0e299cbd5acc67739f9117859782b3c0ccc6d9e7f17d7e0ab1
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# volcano-plot-threshold-classification

## Summary

Renders a differential-expression volcano plot with GraphBio's Shiny 'Volcano plot' module: genes in the input matrix are classified as upregulated or downregulated using the module's default significance and fold-change thresholds, labeled genes are highlighted, and a publication-ready PNG figure with up/down gene counts is exported.

## When to use

Use when a differential-expression matrix with geneID, log2 fold change, p/padj significance, and optional label columns needs to be visualized as a volcano plot with threshold-based up/down classification for publication.

## When NOT to use

- The goal is data preprocessing or statistical computation of differential expression — GraphBio explicitly focuses on facilitating figure generation, not preprocessing and computing
- The input lacks the required geneID / log2 fold change / p or padj / label column structure and cannot be reshaped per the reference example files
- The figure needed is a MA plot, four quadrant diagram, or other module output rather than a threshold-classified volcano plot

## Inputs

- volcano_example.csv-style DE matrix with four required columns: geneID, log2 fold change, significance p/padj, and label
- Companion demo input volcano_example1.csv for the volcano plot module
- Excel-prepared data table formatted against the reference example files

## Outputs

- volcano_plot.png — publication-ready PNG figure with default size and resolution
- Summarized counts of upregulated and downregulated genes reported alongside the plot

## How to apply

Prepare the input in Excel following GraphBio's reference demo files (volcano_example.csv / volcano_example1.csv), which carry four required columns: geneID, log2 fold change, significance p/padj, and a label column naming genes to annotate. Load the matrix into GraphBio's 'Volcano plot' module, which renders the figure via ggplot2 within the Shiny web app, applying the module's default significance threshold and fold-change cutoff to partition genes into upregulated (upper corners) and downregulated groups. Keep the default color presets, point size, and label size so that the genes listed in the 'label' column are highlighted and the figure stays publication-ready, then export the result as PNG (volcano_plot.png) at the default size and resolution, and report the summarized upregulated/downregulated gene counts alongside the plot.

## Related tools

- **GraphBio** (Shiny web app whose 'Volcano plot' module performs the threshold classification and figure rendering) — github.com/databio2022/GraphBio
- **Shiny** (R web framework underlying GraphBio, used to load the CSV input and drive the module)
- **ggplot2** (R plotting library used by the module to render the volcano plot with default colors, point size, and label size)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual
```

## Evaluation signals

- volcano_plot.png exists after the run
- volcano_plot.png is a valid PNG image at the default publication-ready size and resolution
- Genes named in the 'label' column appear highlighted in the upper corners as upregulated
- A summarized up/down regulated gene count is reported alongside the plot
- Rendered figure matches the reference rendering shown in Figure 1B of the article

## Limitations

- The article reports no validation, benchmarking, or user evaluation of the module output
- GraphBio focuses on figure generation, not data preprocessing or computing, so inputs must be pre-formatted per the reference example files
- No formal citation, version number, or license is stated in the article's discussion section

## Evidence

- [readme] volcano plot demo files: "volcano_example.csv and volcano_example1.csv for volcano plot."
- [intro] reference example files: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [methods] volcano example workflow step: "Example 2Volcano plots for differential expression analysis."
- [intro] GraphBio focus statement: "GraphBio specifically focuses on facilitating the generation of"
