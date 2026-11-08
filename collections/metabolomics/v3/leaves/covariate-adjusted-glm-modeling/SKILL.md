---
name: covariate-adjusted-glm-modeling
description: Use when performing transcriptomics differential expression testing on
  RNA-seq count matrices where sample metadata contains confounders (e.g. CONDITION,
  SEX, AGE columns) that must be included in the design formula before filtering significant
  genes.
when_to_use_negative:
- The input is already a normalized feature/abundance table rather than raw RNA-seq
  counts requiring a DESeq2 negative-binomial model
- The metabolomics module is needed — BiomiX quantifies peak differences via the Wilcoxon
  test with FDR correction, not a covariate-adjusted GLM
- Only QC/exploration is required (scatterplots, PCA, correlation heatmaps, UMAP)
  — the preview-QC Shiny interface handles this without GLM fitting
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: User-friendly GUI pipeline that drives the transcriptomics DESeq2 analysis,
    including advanced-options sample filtering
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: Fits the negative-binomial GLM for differential expression with CONDITION
    plus SEX and AGE covariates; BiomiX uses DESeq2/Limma packages for transcriptomics
- name: Limma
  role: Alternative transcriptomics differential expression package used by BiomiX
    alongside DESeq2
- name: R
  role: R-based tooling in which the GLM modeling and result filtering are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:5aeb7a91a00e330af53000d84a9e449ec8bc6d91078f49c8f69e1d3ab9fd6a22
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# covariate-adjusted-glm-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit a negative-binomial generalized linear model (DESeq2) with the biological condition of interest plus nuisance covariates (e.g. SEX and AGE) in the design formula, to identify differentially expressed genes while correcting for unwanted variation, as demonstrated in BiomiX's transcriptomics pipeline on the PTB whole-blood dataset (ENA PRJNA971365).

## When to use

Use when performing transcriptomics differential expression testing on RNA-seq count matrices where sample metadata contains confounders (e.g. CONDITION, SEX, AGE columns) that must be included in the design formula before filtering significant genes.

## When NOT to use

- The input is already a normalized feature/abundance table rather than raw RNA-seq counts requiring a DESeq2 negative-binomial model
- The metabolomics module is needed — BiomiX quantifies peak differences via the Wilcoxon test with FDR correction, not a covariate-adjusted GLM
- Only QC/exploration is required (scatterplots, PCA, correlation heatmaps, UMAP) — the preview-QC Shiny interface handles this without GLM fitting

## Inputs

- Raw RNA-seq count matrix (e.g. BiomiX example tuberculosis whole-blood dataset, ENA PRJNA971365)
- Sample metadata with CONDITION, SEX, and AGE columns
- BiomiX advanced-options transcriptomics sample filtering (optional outlier removal)

## Outputs

- transcriptomics_de_results.csv with gene, log2FC, p-value, p.adj columns
- volcano_plot.png of log2FC vs -log10(p.adj) with significant genes highlighted
- Pathway-level interpretation of DE genes (e.g. Gene Ontology and Reactome via EnrichR/GSEA)

## How to apply

Load the raw RNA-seq count matrix and sample metadata, ensuring metadata rows and count columns are aligned. Run DESeq2 in R fitting a negative-binomial GLM with a design formula that includes CONDITION plus SEX and AGE covariates for correction, following the BiomiX transcriptomics pipeline. Optionally apply the BiomiX advanced-options transcriptomics sample filtering first (removing low-quality samples/outliers flagged by the v2.5 bug fix). Filter DESeq2 results to significant genes with |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05, and write the full results table (gene, log2FC, p-value, p.adj). For validation, verify the output CSV contains the required DESeq2 fields and that all listed rows satisfy the thresholds, and confirm a volcano plot of log2FC versus -log10(p.adj) matches the table's statistics.

## Related tools

- **BiomiX** (User-friendly GUI pipeline that drives the transcriptomics DESeq2 analysis, including advanced-options sample filtering) — https://github.com/IxI-97/BiomiX
- **DESeq2** (Fits the negative-binomial GLM for differential expression with CONDITION plus SEX and AGE covariates; BiomiX uses DESeq2/Limma packages for transcriptomics)
- **Limma** (Alternative transcriptomics differential expression package used by BiomiX alongside DESeq2)
- **R** (R-based tooling in which the GLM modeling and result filtering are executed)

## Examples

```
dds <- DESeqDataSetFromMatrix(countData, colData, design = ~ SEX + AGE + CONDITION); res <- results(dds); resSig <- subset(res, abs(log2FoldChange) > 1 & padj < 0.05)
```

## Evaluation signals

- transcriptomics_de_results.csv exists with the required DESeq2 fields (gene, log2FC, p-value, p.adj)
- Every row in the results table satisfies |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05
- volcano_plot.png is a readable image and its highlighted significant set matches the table's statistics
- Metadata and count-matrix columns are aligned before model fitting (no dropped/mismatched samples)
- Reproduced pathways (e.g. Arachidonic acid metabolic process, antibody-mediated complement activation) are consistent across Gene Ontology and Reactome databases

## Limitations

- BiomiX v2.5 fixed a bug in transcriptomics sample filtering (advance option) — older versions may mis-handle outlier removal
- BiomiX is not a definitive solution; it is a first attempt to incorporate the gold standard single-omics pipeline, and users are strongly recommended to carefully review the dataset before use
- Results depend on metadata quality: CONDITION, SEX, and AGE columns must be correctly aligned with count columns for the covariate correction to be valid

## Evidence

- [other] Run the BiomiX transcriptomics pipeline using DESeq2 in R, fitting a negative-binomial model with a design formula that includes CONDITION plus SEX and AGE covariates for correction.: [evidence span withheld — anchor status: not_found]
- [other] Filter the DESeq2 results to significant genes with |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05: [evidence span withheld — anchor status: not_found]
