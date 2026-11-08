---
name: shiny-dashboard-module-development
description: Use when extending a Shiny omics-visualization dashboard with an additional
  plotting method and you need to match the existing input/output contract (csv/txt/xls/xlsx
  upload, example files, PNG export).
when_to_use_negative:
- The task requires data preprocessing or heavy computation — GraphBio's design explicitly
  focuses on facilitating publication-ready plot generation instead of preprocessing
  and computing.
- The user lacks a correctly formatted example-based input file and cannot prepare
  one in Excel; the module contract relies on users matching given reference example
  files.
- Interactive exploratory parameters or non-PNG publication outputs are required beyond
  GraphBio's configurable size/resolution PNG export.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: GraphBio
  role: The bs4Dash-based Shiny dashboard whose 15-module pattern the new module must
    follow
  repo: github.com/databio2022/GraphBio
- name: Shiny
  role: R framework used to build the dashboard and the new module UI/server logic
  repo: github.com/databio2022/GraphBio
- name: ggplot2
  role: R plotting package wrapped (in an R 4.0.3 environment) to produce the module's
    default publication-ready figure
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:a3114e26b237000bcb1ccfcb9f0e560d8ff2c89d52a66fc8549e4c0168a1c161
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# shiny-dashboard-module-development

## Summary

Develop a new visualization module for the GraphBio bs4Dash-based R Shiny dashboard by wrapping an R plotting package (e.g. ggplot2) so users upload Excel-prepared tabular data and get publication-ready figures with only fine-tuning needed.

## When to use

Use when extending a Shiny omics-visualization dashboard with an additional plotting method and you need to match the existing input/output contract (csv/txt/xls/xlsx upload, example files, PNG export).

## When NOT to use

- The task requires data preprocessing or heavy computation — GraphBio's design explicitly focuses on facilitating publication-ready plot generation instead of preprocessing and computing.
- The user lacks a correctly formatted example-based input file and cannot prepare one in Excel; the module contract relies on users matching given reference example files.
- Interactive exploratory parameters or non-PNG publication outputs are required beyond GraphBio's configurable size/resolution PNG export.

## Inputs

- Tabular user data prepared in Excel: csv (non-UTF-8), txt (tab-separated), xls, or xlsx files
- Reference example input files placed in graphbio-english/www/ (e.g. venn_example.csv, roc_example.csv, sixiangxian_example.csv)

## Outputs

- A new Shiny module integrated in the bs4Dash dashboard with 'run example' and 'view example file' buttons
- Publication-ready rendered figure exported with configurable size and resolution as PNG (e.g. new_module_example_figure.png)

## How to apply

Design the module's input contract to accept tabular files in the four accepted formats (csv, txt, xls, xlsx), where csv files are not UTF-8 encoded and txt files are tab-separated, with users preparing data in Excel based on reference example files shipped in the www/ directory (e.g. heatmap_test.csv, surv_example.csv). Implement the plotting in R (4.0.3 environment) by wrapping ggplot2 or a domain package (pheatmap, survminer, pROC, FactoMineR, factoextra, GOplot) with sensible defaults so the figure is publication-ready without tuning, mirroring the 15 existing GraphBio modules (heatmap, volcano, MA, network, dot, chord, pie, four-quadrant, Venn, cumulative distribution curves, PCA, survival, ROC, correlation, text cluster). Integrate into the bs4Dash-based Shiny dashboard including a 'run example' button and a 'view example file' button, since GraphBio deliberately focuses on figure generation rather than preprocessing or computing. Validate by running the module on a small self-made example input through the dashboard, then export the rendered figure with configurable size and resolution as a PNG (e.g. new_module_example_figure.png).

## Related tools

- **GraphBio** (The bs4Dash-based Shiny dashboard whose 15-module pattern the new module must follow) — github.com/databio2022/GraphBio
- **Shiny** (R framework used to build the dashboard and the new module UI/server logic) — github.com/databio2022/GraphBio
- **ggplot2** (R plotting package wrapped (in an R 4.0.3 environment) to produce the module's default publication-ready figure)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual && docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- The module script runs without error on the small self-made example input file.
- The exported figure exists on disk as a PNG file (e.g. new_module_example_figure.png).
- Default figure styling is judged publication-ready against the paper's criterion 'The default figures are generally ideal for publication'.
- The module supports all four accepted upload formats with the documented encodings (csv non-UTF-8, tab-separated txt, xls/xlsx).
- The module exposes 'run example' and 'view example file' buttons like the 15 existing modules.

## Limitations

- No version number or license for GraphBio is stated in the article; the GitHub repo indicates GPL-3.0 and Docker tags v2.2.7-manual / v2.2.5-en-manual.
- No validation, benchmarking, or user evaluation is reported for the dashboard.
- csv inputs must not be UTF-8 encoded and txt inputs must be tab-separated; deviations break the upload contract.
- Additional visualization methods were still pending integration, so coverage beyond the 15 listed modules is unverified.

## Evidence

- [methods] input contract: [evidence span withheld — anchor status: not_found]
- [intro] design goal: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [intro] framework: "we developed an online web application called GraphBio using a shiny framework in R software. In comparison to other web tools, GraphBio specifically focuses on facilitating the generation of"
- [readme] example data: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [discussion] future work: "In the future, we will continue integrating more popular visualization analysis methods into GraphBio"
