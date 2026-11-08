---
name: transcriptomics-gene-panel-activity-scoring
description: Use when you have a transcriptomics expression matrix (counts or normalized)
  plus a curated gene panel (e.g. 26 IFN-induced genes) and need to split samples
  into biologically defined subgroups for downstream DGE and pathway exploration.
when_to_use_negative:
- Input has no curated gene panel — the activity-score subgrouping requires a user-defined
  panel; for unsupervised clustering of multi-omics use BiomiX's MOFA/SNF/NEMO integration
  instead
- Data are metabolomics, methylomics, or proteomics — this panel-Z-score subgroup
  workflow applies to transcriptomics expression matrices
- The goal is outlier/QC filtering rather than subpopulation analysis — use the Preview-QC
  interface's outlier removal based on PCA centroid distances instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: BiomiX
  role: R/Shiny toolkit hosting the gene-panel subgroup analysis, Preview-QC PCA/UMAP
    visualization, and the DGE-enrichment pipeline
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: Performs differential expression between panel-positive and panel-negative
    subgroups on count data
- name: Limma
  role: Performs differential expression contrasting subgroups when the input is a
    normalized expression matrix
- name: EnrichR
  role: Biological process enrichment on the significant gene lists per subgroup
- name: GSEA
  role: Complementary pathway/phenotype enrichment on subgroup gene lists
- name: R
  role: Runtime environment for Z-score normalization, activity scoring, and subgroup
    assignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:68d49f2deafb00e8350e20e89b9e772a3e82c217b9acf3d8db4a288ae2fd45b4
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# transcriptomics-gene-panel-activity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratify transcriptomics samples using a user-defined gene panel activity score: Z-score normalize each panel gene across samples, call samples panel-positive/negative by the Z>1 criterion (Kirou-style), then run per-subgroup DGE and enrichment. This is the BiomiX subgroup analysis demonstrated on a 26 IFN-induced gene panel in preterm birth (PTB) transcriptomes.

## When to use

Use when you have a transcriptomics expression matrix (counts or normalized) plus a curated gene panel (e.g. 26 IFN-induced genes) and need to split samples into biologically defined subgroups for downstream DGE and pathway exploration.

## When NOT to use

- Input has no curated gene panel — the activity-score subgrouping requires a user-defined panel; for unsupervised clustering of multi-omics use BiomiX's MOFA/SNF/NEMO integration instead
- Data are metabolomics, methylomics, or proteomics — this panel-Z-score subgroup workflow applies to transcriptomics expression matrices
- The goal is outlier/QC filtering rather than subpopulation analysis — use the Preview-QC interface's outlier removal based on PCA centroid distances instead

## Inputs

- Preprocessed transcriptomics expression matrix (raw counts for DESeq2 or normalized values for Limma)
- User-defined gene panel list (e.g. 26 IFN-induced genes)
- Sample metadata/phenotype table

## Outputs

- Subgroup assignment table (per-sample positive/negative panel labels)
- PCA/UMAP plot colored by panel subgroup (Figure S1 analogue)
- Per-subgroup differential expression results (DESeq2/Limma)
- ifn_subgroup_enrichment.csv — EnrichR/GSEA enrichment table per subgroup

## How to apply

Load the preprocessed expression matrix and the user-defined panel gene list into R/BiomiX, restricting the matrix to panel genes. Z-score normalize each panel gene across samples and compute the per-sample mean panel Z score, labeling samples with at least one IFN gene Z-score > 1 as positive per the Kirou-style activity-score criteria to obtain the subgroup assignment table. Visualize positive vs negative separation with PCA/UMAP in the BiomiX Preview-QC interface, then run per-subgroup differential expression with DESeq2 for count data (or Limma for normalized matrices) contrasting positive vs negative subgroups. Submit significant gene lists to EnrichR and GSEA for biological process exploration, and accept the result when enrichments (FDR-adjusted p < 0.05) recover IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling terms.

## Related tools

- **BiomiX** (R/Shiny toolkit hosting the gene-panel subgroup analysis, Preview-QC PCA/UMAP visualization, and the DGE-enrichment pipeline) — https://github.com/IxI-97/BiomiX
- **DESeq2** (Performs differential expression between panel-positive and panel-negative subgroups on count data)
- **Limma** (Performs differential expression contrasting subgroups when the input is a normalized expression matrix)
- **EnrichR** (Biological process enrichment on the significant gene lists per subgroup)
- **GSEA** (Complementary pathway/phenotype enrichment on subgroup gene lists)
- **R** (Runtime environment for Z-score normalization, activity scoring, and subgroup assignment)

## Examples

```
panel_z <- t(scale(t(expr[panel_genes, , drop=FALSE]))); subgroup <- ifelse(colMeans(panel_z) > 1, "positive", "negative")
```

## Evaluation signals

- Subgroup assignment table contains a binary positive/negative label per sample, defined by at least one panel gene with Z-score > 1
- Enrichment table (ifn_subgroup_enrichment.csv) contains significant hits (FDR-adjusted p-value below 0.05) for IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling terms
- PCA/UMAP colored by subgroup shows clear separation of panel-positive and panel-negative samples (reproducing Figure S1)
- Per-subgroup DGE tables are produced with the correct engine: DESeq2 for count inputs, Limma for normalized matrices
- Only panel genes enter the activity-score computation (expression matrix restricted to the 26-gene panel before Z-scoring)

## Limitations

- BiomiX is not a definitive solution; it is a first attempt to incorporate the gold standard single-omics pipeline, and users are strongly recommended to carefully review the dataset before using it
- The Mac installation has known issues; Linux or Windows is suggested for running BiomiX
- BiomiX does not support instrument normalization for metabolomics, and transcriptomics sample filtering had a bug fixed in the April 2025 release — verify the BiomiX version before applying the advance filtering options

## Evidence

- [results] Applied to the PTB dataset, BiomiX subgrouped patients using a panel of 26 IFN-induced genes, labeling samples with at least one IFN gene Z-score > 1 as positive: [evidence span withheld — anchor status: not_found]
