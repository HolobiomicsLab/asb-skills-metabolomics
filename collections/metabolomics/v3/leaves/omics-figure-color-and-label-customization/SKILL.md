---
name: omics-figure-color-and-label-customization
description: Use when a publication-ready omics figure (e.g., a differential-expression
  volcano plot) is being generated in GraphBio and the default color presets, point
  size, or label size need adjustment or the 'label' column genes must be highlighted.
when_to_use_negative:
- The task requires data preprocessing or statistical computation — GraphBio specifically
  focuses on facilitating the generation of publication-ready plots, not upstream
  analysis.
- The figure requires many bespoke parameters or a non-default color scheme beyond
  GraphBio's presets — the tool was built to avoid 'many unnecessary parameters' and
  'outdated color' in prior web tools, and deep restyling is out of scope.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web app providing the visualization modules (volcano plots, heatmaps,
    etc.) whose default color presets, point size, and label size are customized here
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R framework underlying GraphBio's interactive modules in which color/label
    options are adjusted before export
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: R plotting library via which GraphBio modules (e.g., the volcano plot module)
    render the customized figure
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:2587bc8c857502bff71f07c970bf5abb6402758d80ad442ddce7e93f43c2dab1
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
yes_rate: 0.75
n_claims: 8
---

# omics-figure-color-and-label-customization

## Summary

Customizes the appearance of omics figures produced by GraphBio — color presets, point size, and gene-label highlighting (e.g., labeled upregulated genes in volcano plot corners) — so the exported figure is publication-ready as rendered in the article's examples.

## When to use

Use when a publication-ready omics figure (e.g., a differential-expression volcano plot) is being generated in GraphBio and the default color presets, point size, or label size need adjustment or the 'label' column genes must be highlighted.

## When NOT to use

- The task requires data preprocessing or statistical computation — GraphBio specifically focuses on facilitating the generation of publication-ready plots, not upstream analysis.
- The figure requires many bespoke parameters or a non-default color scheme beyond GraphBio's presets — the tool was built to avoid 'many unnecessary parameters' and 'outdated color' in prior web tools, and deep restyling is out of scope.

## Inputs

- volcano_example.csv / volcano_example1.csv — demo differential-expression table with columns geneID, log2 fold change, significance p/padj, label
- Other module-specific reference example CSVs from the graphbio-english/www/ directory (heatmap_test.csv, ma_example.csv, pie_example.csv, etc.)

## Outputs

- Publication-ready PNG figure (e.g., volcano_plot.png) with colored up/downregulated points and labeled genes highlighted
- Summarized counts of upregulated and downregulated genes displayed alongside the plot

## How to apply

Load the module-specific reference CSV from GraphBio's www directory (e.g., volcano_example.csv with the four required columns: geneID, log2 fold change, significance p/padj, and label) and feed it into the corresponding module — e.g., the 'Volcano plot' module, which renders via ggplot2. The module applies the default significance threshold and fold-change cutoff to classify upregulated versus downregulated genes. For appearance tuning, adjust the default color presets, point size, and label size within the module so that genes listed in the 'label' column are highlighted on the figure. The rationale is GraphBio's design goal: users prepare data in Excel based on the given reference example files, and the default figures are generally ideal for publication, so customization should be minimal and focused on color/label polish rather than re-parameterizing the analysis. Finally, export the figure in PNG format at default publication-ready size and resolution (e.g., as volcano_plot.png) and compare it against the article's reference rendering (Figure 1B) for the volcano module.

## Related tools

- **GraphBio** (Shiny web app providing the visualization modules (volcano plots, heatmaps, etc.) whose default color presets, point size, and label size are customized here) — https://github.com/databio2022/GraphBio
- **Shiny** (R framework underlying GraphBio's interactive modules in which color/label options are adjusted before export) — https://github.com/databio2022/GraphBio
- **ggplot2** (R plotting library via which GraphBio modules (e.g., the volcano plot module) render the customized figure)

## Evaluation signals

- Exported file (e.g., volcano_plot.png) exists and is a valid PNG image at the default publication-ready size and resolution.
- Labeled upregulated genes (from the 'label' column) appear highlighted in the upper corners of the volcano plot.
- The figure displays a summarized up/down gene count consistent with the default significance threshold and fold-change cutoff.
- Rendering matches the reference figure shown for the module in the article (e.g., Figure 1B for the volcano plot example).
- The plot was produced from the module's designated reference example CSV without altering the input data format.

## Limitations

- GraphBio provides no validation, benchmarking, or user evaluation of its rendered figures, so customization quality must be judged visually against the reference examples.
- GraphBio focuses on plot generation rather than data preprocessing or computing, so color/label customization cannot compensate for malformed input columns.
- No version number or formal customization-parameter documentation is given in the article; the versioned Docker tags (e.g., v2.2.7-manual, v2.2.5-en-manual) differ between the Chinese and English deployments.

## Evidence

- [intro] default figures ideal for publication: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
- [intro] color motivation: "However, these tools come with limitations, such as many unnecessary parameters, single function, input formats, and outdated color."
- [readme] volcano demo files: "2. volcano_example.csv and volcano_example1.csv for volcano plot."
- [readme] demo data location: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
