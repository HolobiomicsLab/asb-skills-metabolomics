---
name: tabular-gene-matrix-input-preparation
description: Use when input is a gene-level result table (e.g. differential-expression
  results with geneID, log2 fold change, and p/padj columns) that must be reshaped
  to match GraphBio's reference example files before loading into a module.
when_to_use_negative:
- The task requires data preprocessing, normalization, or differential-expression
  computation itself — GraphBio explicitly focuses on facilitating generation of publication-ready
  plots instead of data preprocessing and computing.
- The data are raw sequencing files (e.g. FASTQ, mzML) rather than an already-computed
  gene-level results table.
- The target visualization is not one of GraphBio's 15 supported modules (heatmap,
  volcano, MA, network, dot, chord, pie, four quadrant, Venn, cumulative distribution,
  PCA, survival, ROC, correlation, text cluster).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3360
tools:
- name: GraphBio
  role: target Shiny web app whose module-specific reference example files define
    the required CSV schema; user data must be prepared in Excel to match these files
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R framework underlying GraphBio's data-reading step that ingests the prepared
    CSV matrix
- name: Excel
  role: software used to arrange and export the user's gene-level results into the
    demo-file CSV format
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:9dc9c6a79387606a42a8be34e369f649ac5e9510059ace73b68b1c867f170ec1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - Excel
  - GraphBio
  - Shiny
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tabular-gene-matrix-input-preparation

## Summary

Prepares gene-level tabular input files (CSV prepared in Excel) formatted to GraphBio's reference example files so that a chosen visualization module, e.g. the volcano plot module, can ingest the matrix directly without preprocessing inside the app.

## When to use

Use when input is a gene-level result table (e.g. differential-expression results with geneID, log2 fold change, and p/padj columns) that must be reshaped to match GraphBio's reference example files before loading into a module.

## When NOT to use

- The task requires data preprocessing, normalization, or differential-expression computation itself — GraphBio explicitly focuses on facilitating generation of publication-ready plots instead of data preprocessing and computing.
- The data are raw sequencing files (e.g. FASTQ, mzML) rather than an already-computed gene-level results table.
- The target visualization is not one of GraphBio's 15 supported modules (heatmap, volcano, MA, network, dot, chord, pie, four quadrant, Venn, cumulative distribution, PCA, survival, ROC, correlation, text cluster).

## Inputs

- gene-level differential-expression results table (geneID, log2 fold change, p or padj, label columns)
- GraphBio reference example CSV for the target module (e.g. volcano_example.csv, volcano_example1.csv)
- user spreadsheet in Excel format

## Outputs

- module-ready CSV file matching the demo schema (e.g. volcano_example.csv column layout) loadable by the Shiny app

## How to apply

Identify the GraphBio module to target (e.g. volcano plot) and download its reference demo file from the repository's graphbio-english/www/ directory (volcano_example.csv or volcano_example1.csv for the volcano plot). Arrange the user's gene-level results in Excel to match the demo schema exactly: for the volcano plot module this means the four required columns geneID, log2 fold change, significance value (p or padj), and a label column naming genes to be highlighted. Export the sheet as a comma-separated .csv file rather than .xlsx, since the Shiny app reads CSV via its data-reading step. GraphBio deliberately focuses on plotting rather than data preprocessing, so any gene ID mapping, filtering, or multiple-testing correction must be completed before this file is produced. Validate the prepared file by loading it alongside the shipped demo file and confirming column names, column order, and value types match before feeding it into the module.

## Related tools

- **GraphBio** (target Shiny web app whose module-specific reference example files define the required CSV schema; user data must be prepared in Excel to match these files) — https://github.com/databio2022/GraphBio
- **Shiny** (R framework underlying GraphBio's data-reading step that ingests the prepared CSV matrix)
- **Excel** (software used to arrange and export the user's gene-level results into the demo-file CSV format)

## Evaluation signals

- The prepared CSV has exactly the four required columns of volcano_example.csv (geneID, log2 fold change, significance p/padj, label) in matching order and naming.
- The file loads into the GraphBio module without a data-reading error, as the shipped demo files do.
- The label column contains a non-empty designation only for genes the user wants highlighted on the plot.
- File is plain-text comma-separated (.csv), not an Excel binary format.
- Downstream output (e.g. volcano_plot.png) renders upregulated genes in the upper corners with a summarized up/down gene count, matching the reference rendering in Figure 1B of the article.

## Limitations

- GraphBio provides no data preprocessing or computation — the user must arrive with computed fold changes, p/padj values, and gene labels already in place.
- No version number for the app and no license information are stated in the article; the source repository is GPL-3.0 per external metadata.
- The article reports no validation, benchmarking, or user evaluation of the input-preparation workflow; only demo-file-driven examples are provided.
- Each module has its own example-file schema, so a file prepared for one module (e.g. volcano_example.csv) is not interchangeable with other modules' inputs (e.g. ma_example.csv, surv_example.csv).

## Evidence

- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio.: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
- [readme] volcano_example.csv and volcano_example1.csv for volcano plot: "2. volcano_example.csv and volcano_example1.csv for volcano plot."
- [readme] Demo data for each visualization function (files are placed in the graphbio-english/www/ directory): "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [intro] GraphBio specifically focuses on facilitating the generation of publication-ready plots: "GraphBio specifically focuses on facilitating the generation of"
