---
name: ggplot2-plot-wrapper-implementation
description: Use when adding a new plotting method to a Shiny dashboard where users
  supply tabular omics data (csv, txt, xls, xlsx) and the goal is publication-ready
  default figures with minimal tuning.
when_to_use_negative:
- The task requires data preprocessing or computing rather than visualization — GraphBio
  specifically focuses on facilitating the generation of publication-ready plots instead
  of preprocessing
- The user has no tabular input prepared from a reference example file and expects
  the module to fetch or transform raw omics data itself
- Non-figure outputs (e.g. statistical tables, model objects) are the primary deliverable
  rather than a plot
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GraphBio
  role: Reference Shiny dashboard whose 15-module pattern and input contract the new
    module must follow
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R framework used to build the bs4Dash dashboard hosting the new module
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: R package wrapped by the new module's plotting function to produce the publication-ready
    figure
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:471ce5e437b3c721eb126375030e9fdb2eb0127c829cd798ec495c377ac2d9db
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ggplot2-plot-wrapper-implementation

## Summary

Implement a new visualization module by wrapping ggplot2 (R 4.0.3) with publication-ready defaults and integrating it as a module in a bs4Dash-based Shiny dashboard, following GraphBio's input/output contract.

## When to use

Use when adding a new plotting method to a Shiny dashboard where users supply tabular omics data (csv, txt, xls, xlsx) and the goal is publication-ready default figures with minimal tuning.

## When NOT to use

- The task requires data preprocessing or computing rather than visualization — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of preprocessing
- The user has no tabular input prepared from a reference example file and expects the module to fetch or transform raw omics data itself
- Non-figure outputs (e.g. statistical tables, model objects) are the primary deliverable rather than a plot

## Inputs

- User-prepared tabular data file (csv, txt, xls, or xlsx), formatted against a reference example file
- Small self-made example input file for the module's demo/example button

## Outputs

- A publication-ready default figure rendered in the Shiny dashboard
- Exported figure file in PNG format with configurable size and resolution (e.g. new_module_example_figure.png)

## How to apply

Design the module's input contract to accept the four supported file formats (csv, txt, xls, xlsx), where csv files are not UTF-8 encoded and txt files are tab-separated, mirroring how GraphBio users prepare data in Excel from reference example files. Implement the plotting function in R (R 4.0.3) by wrapping ggplot2 with sensible defaults so the output figure is publication-ready without manual styling, in the same way GraphBio wraps packages such as pheatmap, survminer, pROC, FactoMineR, factoextra, and GOplot. Integrate the module into the bs4Dash-based Shiny dashboard following the pattern of the 15 existing modules, including a 'run example' button and a 'view example file' button. Prepare a small self-made example input file, run the module through the dashboard to render the default figure, and expose configurable figure size and resolution for PNG export.

## Related tools

- **GraphBio** (Reference Shiny dashboard whose 15-module pattern and input contract the new module must follow) — https://github.com/databio2022/GraphBio
- **Shiny** (R framework used to build the bs4Dash dashboard hosting the new module) — https://github.com/databio2022/GraphBio
- **ggplot2** (R package wrapped by the new module's plotting function to produce the publication-ready figure)

## Examples

```
Rscript -e "library(shiny); library(ggplot2); df <- read.csv('new_module_example.csv'); p <- ggplot(df, aes(x, y)) + geom_point(); ggsave('new_module_example_figure.png', p, width=6, height=5, dpi=300)"
```

## Evaluation signals

- The module script runs without error on the small self-made example input file
- The exported figure file exists and is in PNG format (e.g. new_module_example_figure.png)
- The default figure styling is judged publication-ready per the paper's criterion that 'The default figures are generally ideal for publication'
- The module accepts all four supported formats (csv, txt, xls, xlsx) with correct parsing (non-UTF-8 csv, tab-separated txt)
- The module includes functional 'run example' and 'view example file' buttons and supports configurable export size and resolution

## Limitations

- csv inputs are not UTF-8 encoded and txt inputs are tab-separated, so encoding/delimiter handling must match GraphBio's conventions or parsing will fail
- GraphBio does no data preprocessing or computing; the module will fail or produce misleading plots if given unprepared data
- No validation, benchmarking, or user evaluation is reported for GraphBio, so default-figure quality must be judged manually against the publication-readiness criterion
- Only PNG export is specified for validation; other formats are not covered by the workflow

## Evidence

- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [intro] GraphBio specifically focuses on facilitating the generation of publication-ready plots: "we developed an online web application called GraphBio using a shiny framework in R software. In comparison to other web tools, GraphBio specifically focuses on facilitating the generation of"
- [intro] ggplot2 (Wickham, 2016), pheatmap for drawing heatmap, GOplot for drawing chord plot: "they were usually published as R packages, such as ggplot2 (Wickham, 2016), pheatmap for drawing heatmap, GOplot for drawing chord plot"
- [readme] 1. heatmap_test.csv and group_info.csv for heatmap.: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [abstract] GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots: "GraphBio provides 15 popular visualization analysis methods, including heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots"
