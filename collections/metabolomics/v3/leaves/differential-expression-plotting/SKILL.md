---
name: differential-expression-plotting
description: Use when an experimental biologist has a gene expression table (e.g.
  ma_example.csv) from differential expression analysis and needs an MA plot rendered
  via a web interface instead of R code.
when_to_use_negative:
- The goal is differential expression testing or data preprocessing itself — GraphBio
  specifically focuses on facilitating the generation of publication-ready plots instead
  of data preprocessing and computing.
- A different plot type is needed (e.g. volcano plot, heatmap, Venn diagram) — GraphBio's
  15 modules each have their own dedicated example files and module.
- The input data does not follow the layout of the reference example files that users
  are expected to match via Excel.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: GraphBio
  role: Web application providing the MA plot module among its 15 visualization analysis
    methods for omics data
  repo: github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework underlying GraphBio's dashboard interface through which the
    MA plot module is operated
- name: ggplot2
  role: R plotting library on which GraphBio's rendered figures (including the exported
    MA plot PNG) are built
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:6835959500c646adc4a2f909b6c36c902bf83823574b8228998f0381aea78003
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# differential-expression-plotting

## Summary

Generate publication-ready MA plots (log2 fold change versus average expression per gene) from a gene expression table using GraphBio's MA plot module in its R Shiny dashboard, without programming skills.

## When to use

Use when an experimental biologist has a gene expression table (e.g. ma_example.csv) from differential expression analysis and needs an MA plot rendered via a web interface instead of R code.

## When NOT to use

- The goal is differential expression testing or data preprocessing itself — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing.
- A different plot type is needed (e.g. volcano plot, heatmap, Venn diagram) — GraphBio's 15 modules each have their own dedicated example files and module.
- The input data does not follow the layout of the reference example files that users are expected to match via Excel.

## Inputs

- Gene expression table in CSV format (reference file: ma_example.csv from graphbio-english/www/), one of GraphBio's four supported input formats

## Outputs

- Publication-ready MA plot (log2 fold change vs. average expression per gene) rendered on ggplot2, exportable as a PNG image at default size and resolution

## How to apply

Load the demo/reference file ma_example.csv, which is placed in the graphbio-english/www/ directory of the GraphBio repository and is one of GraphBio's four supported input formats (CSV among them). Feed the expression table into GraphBio's MA plot module through the Shiny web app (http://www.graphbio1.com/en/), leaving module parameters at their defaults, so each gene is plotted as log2 fold change versus average expression. The plot is built on ggplot2 inside the dashboard, and the article states 'the default figures are generally ideal for publication', so export the rendered figure as a PNG at default size and resolution. Users are expected to prepare their own data by Excel based on the given reference example files. To self-host rather than use the hosted site, deploy the Docker image databio2022/graphbio:v2.2.5-en-manual and expose port 3838 via Shiny.

## Related tools

- **GraphBio** (Web application providing the MA plot module among its 15 visualization analysis methods for omics data) — github.com/databio2022/GraphBio
- **Shiny** (R web framework underlying GraphBio's dashboard interface through which the MA plot module is operated)
- **ggplot2** (R plotting library on which GraphBio's rendered figures (including the exported MA plot PNG) are built)

## Examples

```
docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- The exported ma_plot.png renders successfully with GraphBio's default styling.
- The plot axes show log2 fold change versus average expression for each gene, matching MA plot semantics.
- The input file matches the format of the reference example ma_example.csv (CSV, one of four supported input formats).
- The figure is produced with all module parameters left at their defaults and passes the paper's criterion that default figures are publication-ready.
- The workflow reproduces the demo output obtained from ma_example.csv as deposited in graphbio-english/www/.

## Limitations

- No version number or license information is stated in the article itself (the README/Docker tags indicate v2.2.x images).
- No validation, benchmarking, or user evaluation of GraphBio is reported in the article.
- GraphBio focuses on plot generation, not on data preprocessing or differential expression computation; users must prepare data themselves in Excel based on the example files.

## Evidence

- [abstract] GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [intro] we developed an online web application called GraphBio using a shiny framework in R software. In comparison to other web tools, GraphBio specifically focuses on facilitating the generation of: "we developed an online web application called GraphBio using a shiny framework in R software. In comparison to other web tools, GraphBio specifically focuses on facilitating the generation of"
- [readme] 9. ma_example.csv for MA plot: [evidence span withheld — anchor status: not_found]
- [readme] docker pull databio2022/graphbio:v2.2.5-en-manual: [evidence span withheld — anchor status: not_found]
