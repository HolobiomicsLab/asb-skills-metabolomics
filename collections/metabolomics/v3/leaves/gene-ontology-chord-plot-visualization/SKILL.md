---
name: gene-ontology-chord-plot-visualization
description: Use when the input is a GO analysis result table (e.g. chord_example.csv)
  linking genes to GO terms and the goal is a circos-style chord diagram rendered
  without programming, via GraphBio's chord plot module.
when_to_use_negative:
- The visualization target is not GO analysis results — GraphBio routes other data
  types to separate modules (e.g. volcano_example.csv for volcano plots, surv_example.csv
  for survival curves, venn_example.csv for Venn plots)
- Data preprocessing or statistical computation is required — GraphBio specifically
  focuses on facilitating the generation of publication-ready plots instead of data
  preprocessing and computing
- The user needs custom layouts beyond the module's default parameters and color presets,
  which the paper does not document
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: GraphBio
  role: Shiny web app providing the chord plot module; user uploads the GO result
    csv and exports the publication-ready chord plot PDF
  repo: https://github.com/databio2022/GraphBio
- name: GOplot
  role: R package backend wrapped by GraphBio's chord plot module to draw the circos
    chord diagram of GO analysis results
- name: Shiny
  role: R web framework underlying GraphBio, enabling no-code use of the chord plot
    function
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:6430aa6e6701940d15ed34afa9777dcc341f2d7db1f02d0bca0a90411ee2dee3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GOplot
  - GraphBio
  - Shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-ontology-chord-plot-visualization

## Summary

Generates a publication-ready chord (circos) plot of Gene Ontology analysis results by uploading a GO-to-gene association table to GraphBio, which wraps the GOplot R backend (Walter et al., 2015) to arrange GO terms and linked genes on a circular layout with ribbon connections.

## When to use

Use when the input is a GO analysis result table (e.g. chord_example.csv) linking genes to GO terms and the goal is a circos-style chord diagram rendered without programming, via GraphBio's chord plot module.

## When NOT to use

- The visualization target is not GO analysis results — GraphBio routes other data types to separate modules (e.g. volcano_example.csv for volcano plots, surv_example.csv for survival curves, venn_example.csv for Venn plots)
- Data preprocessing or statistical computation is required — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing
- The user needs custom layouts beyond the module's default parameters and color presets, which the paper does not document

## Inputs

- GO analysis result table in csv format linking genes to GO terms (e.g. chord_example.csv, the GraphBio demo input)
- Excel-prepared data following the chord_example.csv reference format

## Outputs

- chord_plot.pdf — a circos chord diagram with GO term–gene ribbons, exported with default size and resolution settings

## How to apply

Load the GO analysis result table following GraphBio's supported csv input convention, modeled on the shipped demo file chord_example.csv (placed in graphbio-english/www/ per the README). The chord plot module builds the diagram through the GOplot R package backend (Walter et al., 2015), arranging GO terms and their associated genes on a circos layout with ribbon links. Keep GraphBio's default color preset and default parameter values, which are set as publication-ready, so no manual styling is needed. Users prepare data in Excel based on the reference example files rather than writing R code, since GraphBio is a Shiny web app aimed at experimental biologists without programming skills. Export the rendered figure as chord_plot.pdf with the module's default size and resolution settings.

## Related tools

- **GraphBio** (Shiny web app providing the chord plot module; user uploads the GO result csv and exports the publication-ready chord plot PDF) — https://github.com/databio2022/GraphBio
- **GOplot** (R package backend wrapped by GraphBio's chord plot module to draw the circos chord diagram of GO analysis results)
- **Shiny** (R web framework underlying GraphBio, enabling no-code use of the chord plot function)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual && docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- chord_plot.pdf exists after the run
- The exported figure renders as a circos chord diagram rather than another plot type
- GO term–gene ribbons in the output correspond to the entries in chord_example.csv
- The figure retains GraphBio's default color preset, which the paper states is generally ideal for publication
- Input csv parses under GraphBio's supported csv input convention (columns match the chord_example.csv demo schema)

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio's visualization modules is reported in the paper
- No version number stated for GraphBio in the article (README shows docker tags v2.2.7-manual / v2.2.5-en-manual)
- Future work: additional visualization methods not yet integrated into GraphBio
- The paper reports no DOI or formal citation within the discussion section; citation info comes from the README

## Evidence

- [intro] GOplot backend for chord plots: "ggplot2 (Wickham, 2016), pheatmap for drawing heatmap, GOplot for drawing chord plot of Gene Ontology (GO) analysis results"
- [abstract] chord plots in module list: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [readme] demo data file listing: "4. chord_example.csv for chord plot"
- [readme] docker deployment command: "docker pull databio2022/graphbio:v2.2.5-en-manual"
- [intro] default publication-ready figures: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
