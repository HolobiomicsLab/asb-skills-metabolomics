---
name: z-score-sample-subgroup-stratification
description: Use when a user-defined gene panel (e.g., interferon-stimulated genes)
  is available and samples in a transcriptomics matrix must be assigned to positive/negative
  subgroups for downstream per-subgroup DGE and enrichment analysis.
when_to_use_negative:
- When no user-defined gene panel exists and the goal is unsupervised clustering rather
  than panel-based subgrouping (use BiomiX's SNF/NEMO/MOFA integration instead)
- When the input is raw sequencing data rather than a preprocessed expression matrix
  (upstream preprocessing is required first)
- When the aim is multi-omics factor interpretation rather than single-omics panel-driven
  subgroup stratification
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: Provides the gene-panel subpopulation analysis interface that computes panel
    Z scores and assigns subgroup labels
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: Performs differential expression on count data contrasting positive vs negative
    panel-defined subgroups
- name: Limma
  role: Performs differential expression on normalized expression matrices for the
    subgroup contrast
- name: EnrichR
  role: Biological process exploration on significant gene lists from each subgroup
- name: GSEA
  role: Gene set enrichment analysis of subgroup-specific significant gene lists
- name: R
  role: Language in which BiomiX and the subgroup Z-score computation are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:69212f5f03836b3e15456df10d0a8d28f9709e277c4ec306bf83ca8264e60ccd
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
  - EnrichR
  - GSEA
  - Limma
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Z-score sample subgroup stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies transcriptomics samples into user-defined subgroups by computing per-sample mean Z scores over a user-provided gene panel (e.g., 26 IFN-induced genes) within BiomiX, labeling samples positive when at least one panel gene exceeds a Z-score threshold (e.g., > 1), then running per-subgroup differential expression and enrichment.

## When to use

Use when a user-defined gene panel (e.g., interferon-stimulated genes) is available and samples in a transcriptomics matrix must be assigned to positive/negative subgroups for downstream per-subgroup DGE and enrichment analysis.

## When NOT to use

- When no user-defined gene panel exists and the goal is unsupervised clustering rather than panel-based subgrouping (use BiomiX's SNF/NEMO/MOFA integration instead)
- When the input is raw sequencing data rather than a preprocessed expression matrix (upstream preprocessing is required first)
- When the aim is multi-omics factor interpretation rather than single-omics panel-driven subgroup stratification

## Inputs

- Preprocessed transcriptomics expression matrix (restricted to the user-defined gene panel)
- User-defined gene panel (e.g., 26 IFN-induced genes)
- Sample metadata table with condition/label columns

## Outputs

- Subgroup assignment table (per-sample positive/negative labels based on panel Z scores)
- PCA/UMAP plot colored by subgroup (Figure S1-style separation)
- Per-subgroup differential expression results (DESeq2 or Limma)
- Per-subgroup enrichment table written to ifn_subgroup_enrichment.csv

## How to apply

Load the preprocessed transcriptomics expression matrix and the user-defined gene panel into R/BiomiX, restricting the matrix to panel genes. Z-score normalize each panel gene across samples and compute the per-sample mean panel Z score; following the Kirou-style activity-score criteria implemented in BiomiX's gene-panel subpopulation analysis, call a sample positive when at least one panel gene has Z > 1, otherwise negative. Visualize the resulting subgroup separation with PCA/UMAP colored by subgroup in the Preview-QC interface (as in Figure S1 of the PTB analysis). Then run per-subgroup differential expression contrasting positive vs negative subgroups with DESeq2 for count data or Limma for normalized matrices, and submit significant gene lists to EnrichR and GSEA. Accept the output when enrichment terms for IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling are significant at FDR-adjusted p < 0.05 and the PCA/UMAP shows clear separation of the two subgroups.

## Related tools

- **BiomiX** (Provides the gene-panel subpopulation analysis interface that computes panel Z scores and assigns subgroup labels) — https://github.com/IxI-97/BiomiX
- **DESeq2** (Performs differential expression on count data contrasting positive vs negative panel-defined subgroups)
- **Limma** (Performs differential expression on normalized expression matrices for the subgroup contrast)
- **EnrichR** (Biological process exploration on significant gene lists from each subgroup)
- **GSEA** (Gene set enrichment analysis of subgroup-specific significant gene lists)
- **R** (Language in which BiomiX and the subgroup Z-score computation are implemented)

## Examples

```
Rscript -e "panel <- read.csv('ifn_panel.txt'); z <- t(scale(t(expr[panel$gene,]))); subgroups <- ifelse(apply(z, 2, function(x) any(x > 1)), 'positive', 'negative')"
```

## Evaluation signals

- Enrichment table (ifn_subgroup_enrichment.csv) contains significant hits (FDR-adjusted p-value < 0.05) for IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling terms
- Subgroup PCA/UMAP shows clear separation of positive and negative samples (reproducing Figure S1)
- Every sample receives exactly one label (positive if at least one panel gene Z > 1, else negative) and the assignment table covers all samples
- Per-subgroup DESeq2/Limma contrast converges without errors on the restricted panel expression matrix

## Limitations

- Mac installation has known issues; BiomiX recommends Linux or Windows (Mac to be replaced by a Nextflow workflow)
- Users must carefully review datasets before use; BiomiX provides no automated validation
- The subgrouping depends on the user-supplied panel and Z-score threshold (> 1), so results are panel-specific and not a definitive unsupervised classification

## Evidence

- [other] BiomiX subgrouped patients using a panel of 26 IFN-induced genes, labeling samples with at least one IFN gene Z-score > 1 as positive and yielding a clear separation (Figure S1).: [evidence span withheld — anchor status: not_found]
