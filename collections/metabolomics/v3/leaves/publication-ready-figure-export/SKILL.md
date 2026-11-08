---
name: publication-ready-figure-export
description: Use when a figure produced by a GraphBio module (heatmap, volcano, MA,
  ROC, chord, network, etc.) needs to be saved in a downloadable image format with
  publication-appropriate dimensions and resolution rather than only viewed on screen.
when_to_use_negative:
- The task requires data preprocessing, cleaning, or computation — GraphBio specifically
  focuses on facilitating the generation of publication-ready plots instead of data
  preprocessing and computing.
- The user needs custom, non-default styling beyond fine-tuning (e.g., novel color
  schemes); the paper notes existing tools' 'outdated color' as a limitation GraphBio
  addresses with its presets, but deep customization is not its scope.
- The source data is raw instrument output or an unprocessed omics file that must
  first be converted to GraphBio's accepted tabular formats (csv/txt/xls/xlsx) with
  columns matching the reference example files.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GraphBio
  role: Shiny web app whose 15 modules render the plots and provide the figure download
    controls (format, size, resolution)
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R framework underlying the GraphBio dashboard through which figures are rendered
    and exported
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: R plotting backend wrapped by modules such as volcano plot, MA plot, and network
    plot to produce the figures being exported
- name: pheatmap
  role: Backend for the heatmap module whose clustered, group-annotated output is
    exported as heatmap.png
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:c07f1039d02ebc8426d460fc289d40abb0b77201ddff2becebdc0c0d1c5c86d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GOplot
  - GraphBio
  - Shiny
  - ggplot2
  - pROC
  - pheatmap
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# publication-ready-figure-export

## Summary

Export rendered plots from GraphBio's 15 visualization modules as publication-ready figures in PNG, PDF, JPEG, or TIFF with customizable size and resolution, so experimental biologists without programming skills can obtain journal-usable figures directly from the Shiny dashboard.

## When to use

Use when a figure produced by a GraphBio module (heatmap, volcano, MA, ROC, chord, network, etc.) needs to be saved in a downloadable image format with publication-appropriate dimensions and resolution rather than only viewed on screen.

## When NOT to use

- The task requires data preprocessing, cleaning, or computation — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing.
- The user needs custom, non-default styling beyond fine-tuning (e.g., novel color schemes); the paper notes existing tools' 'outdated color' as a limitation GraphBio addresses with its presets, but deep customization is not its scope.
- The source data is raw instrument output or an unprocessed omics file that must first be converted to GraphBio's accepted tabular formats (csv/txt/xls/xlsx) with columns matching the reference example files.

## Inputs

- Tabular demo/user data files prepared in Excel: csv (non-UTF-8), txt (tab-separated), xls, or xlsx (e.g., heatmap_test.csv with group_info.csv, volcano_example.csv, ma_example.csv, roc_example.csv, chord_example.csv, corr_net.csv)
- A rendered plot from one of GraphBio's 15 visualization modules in the bs4Dash-based Shiny dashboard

## Outputs

- Downloadable figure files in PDF, PNG, JPEG, or TIFF with customizable size and resolution (e.g., heatmap.png, volcano_plot.png, ma_plot.png, roc_curve.pdf, chord_plot.pdf, new_module_example_figure.png)

## How to apply

Render the plot with the module's default settings first, since the paper's design goal is that 'The default figures are generally ideal for publication' with only fine-tuning needed in some cases. Choose the export format per target use — GraphBio's modules support downloadable plots in PDF, PNG, JPEG, or TIFF with customizable size and resolution (e.g., export the ROC curve as roc_curve.pdf, the heatmap as heatmap.png, the chord plot as chord_plot.pdf). Keep parameter changes minimal and data in GraphBio's supported formats (csv not UTF-8 encoded, txt tab-separated, xls, xlsx) prepared in Excel against the module's reference example file (e.g., volcano_example.csv, roc_example.csv). Validate the exported file exists, opens as the claimed image type, and reproduces the module's default styling — e.g., labeled upregulated genes in the upper corners of a volcano plot with a summarized up/down gene count, or clustered, group-annotated rows/columns in a heatmap built on pheatmap.

## Related tools

- **GraphBio** (Shiny web app whose 15 modules render the plots and provide the figure download controls (format, size, resolution)) — https://github.com/databio2022/GraphBio
- **Shiny** (R framework underlying the GraphBio dashboard through which figures are rendered and exported) — https://github.com/databio2022/GraphBio
- **ggplot2** (R plotting backend wrapped by modules such as volcano plot, MA plot, and network plot to produce the figures being exported)
- **pheatmap** (Backend for the heatmap module whose clustered, group-annotated output is exported as heatmap.png)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual; docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init  # then upload volcano_example.csv in the 'Volcano plot' module and download volcano_plot.png
```

## Evaluation signals

- The exported file exists and is a valid image of the claimed type (e.g., volcano_plot.png is a valid PNG; roc_curve.pdf and chord_plot.pdf open as PDFs).
- The figure reproduces the module's default publication-ready styling: e.g., the volcano plot shows labeled upregulated genes in the upper corners with a summarized up/down gene count, matching Figure 1B of the article.
- For the heatmap, the exported heatmap.png shows hierarchical clustering of rows/columns and group annotation aligned to group_info.csv, matching the Figure 1A demo style.
- The ROC export contains a single ROC curve with the computed AUC value annotated in the legend and reproduces the AUC shown by the GraphBio ROC module for roc_example.csv.
- Default figures require only fine-tuning, consistent with the paper's criterion that 'The default figures are generally ideal for publication'.

## Limitations

- No validation, benchmarking, or user evaluation of the exported figure quality is reported in the paper.
- No version number or license for the GraphBio application is stated in the article (the GitHub repo is GPL-3.0 and Docker tags exist, e.g., databio2022/graphbio:v2.2.7-manual and v2.2.5-en-manual for the English version).
- csv inputs must not be UTF-8 encoded and txt inputs must be tab-separated; deviating encodings/delimiters can break module input handling before export.

## Evidence

- [intro] Design goal with default styling: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [full_text] Downloadable formats and size/resolution: [evidence span withheld — anchor status: not_found]
- [readme] Demo data files in www directory: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [intro] Scope excludes preprocessing: [evidence span withheld — anchor status: not_found]
- [abstract] Skill purpose: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
