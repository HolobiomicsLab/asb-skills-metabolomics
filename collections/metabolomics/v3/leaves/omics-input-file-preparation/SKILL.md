---
name: omics-input-file-preparation
description: Use when a tabular omics result (e.g. a gene expression table, differential-expression
  table, or group annotation file) must be reshaped in Excel to match one of GraphBio's
  reference example files before feeding a specific visualization module.
when_to_use_negative:
- The data require preprocessing, quality control, or statistical computation first
  — GraphBio 'specifically focuses on facilitating the generation of' plots rather
  than data preprocessing and computing
- The input table already matches the reference example file's schema and can be loaded
  directly into the module
- The target visualization is not among GraphBio's 15 modules (e.g. a custom plot
  type the tool does not provide)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: GraphBio
  role: Target web application whose 15 visualization modules consume the prepared
    CSV files; its example files define the required input schemas
  repo: https://github.com/databio2022/GraphBio
- name: Excel
  role: Used to reshape and format the user's data to match the reference example
    file layouts before saving as CSV
- name: Shiny
  role: R framework underlying the GraphBio dashboard that ingests the prepared CSV
    input
- name: ggplot2
  role: Rendering library behind the modules that validates the prepared input by
    producing the default publication-ready figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:00149bdb7d1096b263f98af2e45cd8dbfa0e2f21aea8c30a96d5dbb057422167
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - Excel
  - GraphBio
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# omics-input-file-preparation

## Summary

Prepare omics data tables as CSV files whose column layout mirrors GraphBio's deposited reference example files (e.g. ma_example.csv, group_info.csv in graphbio-english/www/), so that they load cleanly into the corresponding Shiny visualization module and produce publication-ready default figures.

## When to use

Use when a tabular omics result (e.g. a gene expression table, differential-expression table, or group annotation file) must be reshaped in Excel to match one of GraphBio's reference example files before feeding a specific visualization module.

## When NOT to use

- The data require preprocessing, quality control, or statistical computation first — GraphBio 'specifically focuses on facilitating the generation of' plots rather than data preprocessing and computing
- The input table already matches the reference example file's schema and can be loaded directly into the module
- The target visualization is not among GraphBio's 15 modules (e.g. a custom plot type the tool does not provide)

## Inputs

- User-supplied omics data table (CSV or Excel-editable tabular data)
- Reference example file for the target module (e.g. ma_example.csv, volcano_example.csv, heatmap_test.csv with group_info.csv) from graphbio-english/www/

## Outputs

- CSV file formatted to the target module's expected column schema
- Publication-ready plot (PNG export) rendered by the GraphBio Shiny module with default parameters

## How to apply

Locate the reference example file for the target module in the GraphBio repository's graphbio-english/www/ directory — e.g. ma_example.csv for the MA plot module, heatmap_test.csv plus group_info.csv for the heatmap, volcano_example.csv for volcano plots, roc_example.csv for ROC curves, surv_example.csv for survival curves, or venn_example.csv for Venn plots. Open your own data in Excel and reshape columns to match the reference file's structure exactly, since GraphBio deliberately lets users 'prepare data to be visualized by Excel software based on given reference example files'. Save in CSV format, which is one of GraphBio's supported input formats. Load the prepared CSV into the corresponding module in the Shiny dashboard (hosted at http://www.graphbio1.com/en/ or self-hosted via the databio2022/graphbio Docker image) and keep module parameters at their defaults, because 'the default figures are generally ideal for publication'. Export the rendered ggplot2 figure (e.g. as a PNG) once the plot renders without input errors.

## Related tools

- **GraphBio** (Target web application whose 15 visualization modules consume the prepared CSV files; its example files define the required input schemas) — https://github.com/databio2022/GraphBio
- **Excel** (Used to reshape and format the user's data to match the reference example file layouts before saving as CSV)
- **Shiny** (R framework underlying the GraphBio dashboard that ingests the prepared CSV input)
- **ggplot2** (Rendering library behind the modules that validates the prepared input by producing the default publication-ready figure)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual
```

## Evaluation signals

- The prepared CSV loads into the target GraphBio module without input-format errors, mirroring the behavior of the deposited demo file
- Column headers and ordering in the user's CSV are indistinguishable in structure from the corresponding reference file (e.g. ma_example.csv for the MA plot module)
- With all module parameters left at defaults, the rendered figure is judged publication-ready, per the paper's criterion that default figures are generally ideal for publication
- The exported PNG renders successfully with GraphBio's default ggplot2 styling
- The correct example file was matched to the correct module (e.g. group_info.csv accompanies heatmap_test.csv for heatmaps, not volcano_example.csv)

## Limitations

- GraphBio does not perform data preprocessing or computation, so input tables must already contain the analysis results to be visualized
- The paper reports no validation, benchmarking, or user evaluation of the tool's workflow
- Correct formatting depends on the user manually matching each example file's layout in Excel; no automated schema validation is described
- No version number or license is stated in the article text (the repo indicates GPL-3.0; Docker tags distinguish v2.2.7-manual and v2.2.5-en-manual)

## Evidence

- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
- [readme] ma_example.csv for MA plot: "9. ma_example.csv for MA plot"
- [readme] Demo data for each visualization function (files are placed in the graphbio-english/www/ directory): "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [intro] Design goal: GraphBio focuses on plotting rather than preprocessing: "GraphBio specifically focuses on facilitating the generation of"
- [readme] Example group-annotation pairing for heatmap module: "1. heatmap_test.csv and group_info.csv for heatmap."
