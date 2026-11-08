---
name: bulk-rna-seq-count-matrix-loading
description: Use when the input is a raw whole-blood or tissue RNA-seq count matrix
  (CSV/Excel) plus a metadata file with a CONDITION column and covariates, and the
  goal is differential expression via BiomiX/DESeq2.
when_to_use_negative:
- Input is already normalized, transformed, or a precomputed DE results table — BiomiX
  transcriptomics expects raw counts for DESeq2's negative-binomial model
- Data is metabolomics or methylomics — use BiomiX's TidyMass/CEU Mass Mediator or
  ChAMP modules instead of the transcriptomics module
- Single-cell or proteomics/genomics data — the article notes BiomiX does not yet
  support proteomics and genomics analysers
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: GUI-based transcriptomics pipeline that loads the count matrix and metadata,
    applies optional sample filtering, and runs DESeq2
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: R package used downstream of count-matrix loading to fit the negative-binomial
    differential expression model on CONDITION plus SEX/AGE covariates
- name: Limma
  role: Alternative DESeq2/Limma transcriptomics engine offered by BiomiX for differential
    expression on the loaded counts
- name: R
  role: Language in which BiomiX and its Shiny QC/loading interfaces are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:1ee7505991c34db414fd684dc0dc8a5c944a74299012ca75aa2336ae0879fa9a
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
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Bulk RNA-seq count matrix loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loading a raw RNA-seq count matrix together with its sample metadata (e.g., CONDITION, SEX, AGE columns) into BiomiX's transcriptomics module for DESeq2-based differential expression analysis, ensuring metadata and count columns are aligned before running.

## When to use

Use when the input is a raw whole-blood or tissue RNA-seq count matrix (CSV/Excel) plus a metadata file with a CONDITION column and covariates, and the goal is differential expression via BiomiX/DESeq2.

## When NOT to use

- Input is already normalized, transformed, or a precomputed DE results table — BiomiX transcriptomics expects raw counts for DESeq2's negative-binomial model
- Data is metabolomics or methylomics — use BiomiX's TidyMass/CEU Mass Mediator or ChAMP modules instead of the transcriptomics module
- Single-cell or proteomics/genomics data — the article notes BiomiX does not yet support proteomics and genomics analysers

## Inputs

- Raw RNA-seq count matrix (CSV or Excel; features x samples), e.g., the ENA PRJNA971365 whole-blood counts
- Sample metadata table with CONDITION, SEX, and AGE columns aligned to count-matrix columns
- Optional user-selected list of outlier samples to remove manually

## Outputs

- Aligned count matrix and metadata passed to the DESeq2 transcriptomics pipeline
- transcriptomics_de_results.csv with gene, log2FC, p-value, p.adj for significant genes (|log2FC| > 1, p.adj < 0.05)
- QC previews from the Preview-QC interface (scatterplots, PCA, correlation heatmaps, UMAP)

## How to apply

Load the raw RNA-seq count matrix (genes/features as rows, samples as columns) and the sample metadata containing a CONDITION column plus covariates such as SEX and AGE, as in the PRJNA971365 PTB vs healthy-control example. BiomiX accepts Excel files directly (supported since v2.3), and the count-matrix columns must be aligned with the metadata rows before analysis. Optionally enable the transcriptomics sample filtering advanced option (fixed in v2.4/v2.5) to remove outliers based on their distances from PCA centroids, and use the Preview-QC interface to inspect PCA, correlation heatmaps, and UMAP before proceeding. Once loaded, BiomiX fits the DESeq2 negative-binomial model on the counts using the design including CONDITION and covariates; results are filtered to |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05.

## Related tools

- **BiomiX** (GUI-based transcriptomics pipeline that loads the count matrix and metadata, applies optional sample filtering, and runs DESeq2) — https://github.com/IxI-97/BiomiX
- **DESeq2** (R package used downstream of count-matrix loading to fit the negative-binomial differential expression model on CONDITION plus SEX/AGE covariates)
- **Limma** (Alternative DESeq2/Limma transcriptomics engine offered by BiomiX for differential expression on the loaded counts)
- **R** (Language in which BiomiX and its Shiny QC/loading interfaces are implemented)

## Examples

```
res <- DESeq2::results(dds); write.csv(subset(as.data.frame(res), abs(log2FoldChange) > 1 & padj < 0.05), "transcriptomics_de_results.csv")
```

## Evaluation signals

- transcriptomics_de_results.csv exists with the required DESeq2 fields (gene, log2FC, p-value, p.adj)
- All rows in the results table satisfy |log2FC| > 1 and Benjamini-Hochberg adjusted p < 0.05
- Number of columns in the count matrix equals the number of rows in the metadata (sample alignment invariant)
- Preview-QC PCA/UMAP plots render without error and flagged outliers are excluded before DESeq2 fitting
- Pathways recovered (e.g., Arachidonic acid metabolic process, antibody-mediated complement activation) reproduce the original PTB manuscript's findings across Gene Ontology and Reactome

## Limitations

- BiomiX does not support instrument normalization for metabolomics and users must carefully review the dataset before use — no automated validation
- A bug in transcriptomics sample filtering (advanced option) existed before the v2.4/v2.5 fix, so older versions may mis-handle outlier removal
- Mac installation has known issues; the team recommends Linux or Windows, so loading/running on macOS may fail

## Evidence

- [abstract] using DESeq2/Limma packages for transcriptomics: "using DESeq2/Limma packages for transcriptomics"
- [readme] Bug fix in transcriptomics sample filtering (advance option): "Bug fix in transcriptomics sample filtering (advance option)"
- [readme] This interface also includes a tab for removing highly variable features: "This interface also includes a tab for removing highly variable features that are likely driven by noise"
- [readme] Excel File Format Support: "Excel File Format Support: Now handle Excel files seamlessly within the application."
