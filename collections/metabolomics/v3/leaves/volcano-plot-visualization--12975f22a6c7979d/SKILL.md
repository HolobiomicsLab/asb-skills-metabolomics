---
name: volcano-plot-visualization
description: Use when a DESeq2/Limma results table (gene, log2FC, p-value, p.adj)
  is available and a visual summary of magnitude-versus-significance of differential
  expression is needed.
when_to_use_negative:
- Input is raw counts or a sample metadata table rather than a fitted differential
  expression results table with log2FC and adjusted p-values
- The aim is pathway-level exploration (EnrichR/GSEA biological process analysis)
  rather than per-gene differential expression visualization
- The omics module is metabolomics or methylomics, where BiomiX applies different
  statistical tests (e.g. Wilcoxon with FDR correction) instead of the DESeq2 log2FC/p.adj
  output this plot requires
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: Shiny-based GUI that runs the transcriptomics DESeq2 pipeline whose results
    table feeds the volcano plot; includes Preview-QC outlier filtering and advanced-options
    transcriptomics sample filtering
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: R package that produces the negative-binomial differential expression statistics
    (log2FC, p-value, Benjamini-Hochberg p.adj) plotted on the volcano axes
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Environment in which the plot of log2FC versus -log10(p.adj) is generated
    and saved as volcano_plot.png
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:6219a93fd0ac5fa589bffe0d3ff1c5caae278ddcb6d85598e4da46e23c000fac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - DESeq2
  - Limma
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Volcano plot visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a volcano plot of log2 fold change versus -log10(adjusted p-value) for all tested genes from a DESeq2 differential expression analysis, highlighting the significant set. In the BiomiX transcriptomics workflow this visually summarizes PTB-vs-HC whole-blood RNA-seq results after Benjamini-Hochberg correction.

## When to use

Use when a DESeq2/Limma results table (gene, log2FC, p-value, p.adj) is available and a visual summary of magnitude-versus-significance of differential expression is needed.

## When NOT to use

- Input is raw counts or a sample metadata table rather than a fitted differential expression results table with log2FC and adjusted p-values
- The aim is pathway-level exploration (EnrichR/GSEA biological process analysis) rather than per-gene differential expression visualization
- The omics module is metabolomics or methylomics, where BiomiX applies different statistical tests (e.g. Wilcoxon with FDR correction) instead of the DESeq2 log2FC/p.adj output this plot requires

## Inputs

- Raw RNA-seq count matrix (e.g. ENA PRJNA971365 whole-blood counts) with sample metadata containing CONDITION, SEX, AGE columns
- DESeq2 differential expression results table with columns: gene, log2FC, p-value, p.adj (transcriptomics_de_results.csv)

## Outputs

- volcano_plot.png — image of log2FC versus -log10(p.adj) for all tested genes with the significant set (|log2FC| > 1, p.adj < 0.05) highlighted

## How to apply

Run the BiomiX transcriptomics pipeline (DESeq2 negative-binomial model with CONDITION plus SEX and AGE covariates on the PRJNA971365 PTB whole-blood count matrix), then filter results to significant genes with |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05. Plot log2FC on the x-axis against -log10(p.adj) on the y-axis for all tested genes, coloring/highlighting points that pass both thresholds, and save the figure as volcano_plot.png. Optionally use the BiomiX Preview-QC interface to remove outliers based on distances from PCA centroids before fitting, so the plotted statistics are not distorted by low-quality samples. Validate that the highlighted points in the plot match the rows of transcriptomics_de_results.csv that satisfy the cutoffs.

## Related tools

- **BiomiX** (Shiny-based GUI that runs the transcriptomics DESeq2 pipeline whose results table feeds the volcano plot; includes Preview-QC outlier filtering and advanced-options transcriptomics sample filtering) — https://github.com/IxI-97/BiomiX
- **DESeq2** (R package that produces the negative-binomial differential expression statistics (log2FC, p-value, Benjamini-Hochberg p.adj) plotted on the volcano axes) — https://github.com/IxI-97/BiomiX
- **R** (Environment in which the plot of log2FC versus -log10(p.adj) is generated and saved as volcano_plot.png)

## Examples

```
Rscript -e 'res <- read.csv("transcriptomics_de_results.csv"); plot(res$log2FC, -log10(res$p.adj), pch=16, xlab="log2FC", ylab="-log10(p.adj)"); points(subset(res, abs(log2FC)>1 & p.adj<0.05), col="red", pch=16)'
```

## Evaluation signals

- transcriptomics_de_results.csv exists and contains the required DESeq2 fields (gene, log2FC, p-value, p.adj)
- Every row listed as significant satisfies both |log2FC| > 1 and Benjamini-Hochberg p.adj < 0.05
- volcano_plot.png is a readable image whose highlighted points match the significant rows in the results table
- Plot x-axis range covers the log2FC span of all tested genes and the y-axis corresponds to -log10(p.adj)
- Metadata and count matrix columns were aligned (and outlier samples removed) before the DESeq2 fit, so plotted statistics reflect the filtered cohort

## Limitations

- Mac version of BiomiX has installation issues; Linux or Windows is recommended (Nextflow workflow planned as replacement)
- A bug affecting transcriptomics sample filtering (advanced option) existed in earlier versions; users should review the dataset carefully before use since BiomiX performs no automated validation
- BiomiX does not support instrument normalization for metabolomics, so cross-omics comparisons beyond the transcriptomics results shown here are out of scope

## Evidence

- [other] Filter the DESeq2 results to significant genes with |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05: [evidence span withheld — anchor status: not_found]
