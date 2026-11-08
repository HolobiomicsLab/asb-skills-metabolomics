---
name: go-term-gene-linkage-interpretation
description: Use when the input is a GO analysis result table linking genes to GO
  terms (csv, e.g. chord_example.csv) and the goal is a publication-ready circos chord
  plot of term–gene associations.
when_to_use_negative:
- The input is not a GO analysis result table (e.g. it is a survival table like surv_example.csv,
  a Venn input like venn_example.csv, or a plain expression matrix for heatmaps) —
  GraphBio routes those to other modules.
- A raw omics data table needing preprocessing or statistical computation is all that
  exists — GraphBio specifically focuses on plot generation, not data preprocessing
  or computing.
- A different plot type is needed for the GO results, such as a GO dot plot (go_bubble_example.csv
  / go_term_bubble.csv inputs), for which the chord module is the wrong choice.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3315
tools:
- name: GraphBio
  role: Shiny web app providing the chord plot module with publication-ready default
    colors and parameters for GO analysis results
  repo: https://github.com/databio2022/GraphBio
- name: GOplot
  role: R package backend wrapped by GraphBio's chord plot module; draws the chord
    plot of Gene Ontology (GO) analysis results on a circos layout
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:ad0df0c06218ffbfe40acd66dda48835902e6565e814324c44170ab33b6e0984
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GOplot
  - GraphBio
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# GO term–gene linkage interpretation

## Summary

Visualizes Gene Ontology (GO) analysis results as a circos chord diagram, where GO terms and their associated genes are arranged around a circle with ribbon links, via GraphBio's chord plot module backed by the GOplot R package (Walter et al., 2015).

## When to use

Use when the input is a GO analysis result table linking genes to GO terms (csv, e.g. chord_example.csv) and the goal is a publication-ready circos chord plot of term–gene associations.

## When NOT to use

- The input is not a GO analysis result table (e.g. it is a survival table like surv_example.csv, a Venn input like venn_example.csv, or a plain expression matrix for heatmaps) — GraphBio routes those to other modules.
- A raw omics data table needing preprocessing or statistical computation is all that exists — GraphBio specifically focuses on plot generation, not data preprocessing or computing.
- A different plot type is needed for the GO results, such as a GO dot plot (go_bubble_example.csv / go_term_bubble.csv inputs), for which the chord module is the wrong choice.

## Inputs

- GO analysis result table as csv linking genes to GO terms (GraphBio convention; reference demo file: chord_example.csv, placed in the graphbio-english/www/ directory)

## Outputs

- chord_plot.pdf — circos chord diagram with GO term–gene ribbon links, default size and resolution, publication-ready styling

## How to apply

Load the GO analysis result table as a csv following GraphBio's supported csv input convention, using the shipped demo file chord_example.csv as the format reference. Build the chord diagram through GraphBio's chord plot module, which wraps the GOplot R package (Walter et al., 2015); GOplot arranges GO terms and their associated genes on a circos layout with ribbon links. Keep the module's default color preset and default parameter values, which GraphBio sets as publication-ready. Export the rendered figure as a PDF named chord_plot.pdf with default size and resolution settings. Users without programming skills can do this entirely through the Shiny web app after preparing data in Excel based on the reference example file.

## Related tools

- **GraphBio** (Shiny web app providing the chord plot module with publication-ready default colors and parameters for GO analysis results) — https://github.com/databio2022/GraphBio
- **GOplot** (R package backend wrapped by GraphBio's chord plot module; draws the chord plot of Gene Ontology (GO) analysis results on a circos layout)

## Evaluation signals

- chord_plot.pdf exists after export.
- chord_plot.pdf renders as a circos chord diagram with ribbon links.
- The GO term–gene ribbons in the figure correspond entry-by-entry to the rows of chord_example.csv.
- Default color preset and default parameters were used unchanged (publication-ready defaults from GraphBio).
- Input csv follows the same column/format structure as the shipped demo file chord_example.csv.

## Limitations

- No validation, benchmarking, or user evaluation is reported for GraphBio in the article.
- No version number stated for GraphBio in the article text (README docker tags reference v2.2.5-en-manual / v2.2.7-manual).
- GraphBio is focused on plot generation only — it does not perform the upstream GO enrichment computation that produces the input table.
- No license information provided in the article discussion (GitHub repo metadata indicates GPL-3.0).

## Evidence

- [intro] GOplot for drawing chord plot of Gene Ontology (GO) analysis results: "pheatmap for drawing heatmap, GOplot for drawing chord plot of Gene Ontology (GO) analysis results"
- [readme] 4. chord_example.csv for chord plot: "4. chord_example.csv for chord plot"
- [methods] GraphBio module list including chord plots: "GraphBio provides 15 visualization analysis modules, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
