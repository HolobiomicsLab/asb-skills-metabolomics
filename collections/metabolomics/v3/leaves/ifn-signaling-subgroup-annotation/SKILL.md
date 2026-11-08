---
name: ifn-signaling-subgroup-annotation
description: Use when a transcriptomics dataset needs patient/sample stratification
  by a specific gene-signature activity (e.g., IFN signaling in preterm birth PTB
  transcriptomes) and downstream per-subgroup pathway enrichment.
when_to_use_negative:
- When the goal is generic unsupervised clustering rather than annotation against
  a biologically defined gene panel
- When no user-defined gene panel exists and MOFA factor discovery from the full multi-omics
  matrix is the intended approach
- When data are metabolomics or methylomics rather than transcriptomics, since DESeq2/Limma
  and gene-panel subgrouping apply to transcriptomics
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: BiomiX
  role: GUI pipeline performing subgroup analysis based on user-defined gene panels,
    Z-score labeling, PCA/UMAP visualization, and orchestration of downstream steps
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: Differential expression testing on count data contrasting positive vs negative
    panel-defined subgroups
  repo: https://github.com/IxI-97/BiomiX
- name: Limma
  role: Differential expression testing on normalized expression matrices for subgroup
    contrasts
  repo: https://github.com/IxI-97/BiomiX
- name: EnrichR
  role: Biological process exploration of significant gene lists per subgroup
  repo: https://github.com/IxI-97/BiomiX
- name: GSEA
  role: Gene set enrichment analysis of per-subgroup significant genes
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Underlying language of BiomiX and its Shiny Preview-QC interface
  repo: https://github.com/IxI-97/BiomiX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:14975605a41205b43311f6ef37e7d1b105b6ddd66ffda03d5828bb56be0b90ad
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# ifn-signaling-subgroup-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotation of transcriptomics samples into biologically meaningful subgroups using a user-defined gene panel (e.g., 26 IFN-induced genes) via Z activity-score criteria in BiomiX, followed by per-subgroup differential expression (DESeq2/Limma) and EnrichR/GSEA pathway exploration.

## When to use

Use when a transcriptomics dataset needs patient/sample stratification by a specific gene-signature activity (e.g., IFN signaling in preterm birth PTB transcriptomes) and downstream per-subgroup pathway enrichment.

## When NOT to use

- When the goal is generic unsupervised clustering rather than annotation against a biologically defined gene panel
- When no user-defined gene panel exists and MOFA factor discovery from the full multi-omics matrix is the intended approach
- When data are metabolomics or methylomics rather than transcriptomics, since DESeq2/Limma and gene-panel subgrouping apply to transcriptomics

## Inputs

- Preprocessed transcriptomics expression matrix (counts or normalized values) restricted to the gene panel genes
- User-defined gene panel (e.g., 26 IFN-induced genes) as a gene list
- Sample metadata with condition/subgroup labels

## Outputs

- Subgroup assignment table (positive vs negative per-sample labels)
- PCA/UMAP figures showing subgroup separation (e.g., Figure S1)
- Per-subgroup differential expression results from DESeq2 or Limma
- Enrichment table (ifn_subgroup_enrichment.csv) with EnrichR/GSEA terms and FDR-adjusted p-values

## How to apply

Load the preprocessed expression matrix restricted to the user-defined gene panel (e.g., 26 IFN-induced genes) into R/BiomiX, Z-score normalize each panel gene across samples, and label samples positive when at least one panel gene has Z-score > 1 (Kirou-style activity-score criteria). Visualize positive vs negative subgroups via PCA/UMAP in the Preview-QC interface, which 'provides an overview of the data through scatterplots (for two selected variables), PCA, correlation heatmaps, and UMAP'. Run per-subgroup differential expression with DESeq2 for count data or Limma for normalized matrices, contrasting positive vs negative subgroups. Submit significant gene lists to EnrichR and GSEA for biological process exploration, accepting the result when terms such as IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling reach FDR-adjusted p < 0.05 and the PCA/UMAP shows clear subgroup separation.

## Related tools

- **BiomiX** (GUI pipeline performing subgroup analysis based on user-defined gene panels, Z-score labeling, PCA/UMAP visualization, and orchestration of downstream steps) — https://github.com/IxI-97/BiomiX
- **DESeq2** (Differential expression testing on count data contrasting positive vs negative panel-defined subgroups) — https://github.com/IxI-97/BiomiX
- **Limma** (Differential expression testing on normalized expression matrices for subgroup contrasts) — https://github.com/IxI-97/BiomiX
- **EnrichR** (Biological process exploration of significant gene lists per subgroup) — https://github.com/IxI-97/BiomiX
- **GSEA** (Gene set enrichment analysis of per-subgroup significant genes) — https://github.com/IxI-97/BiomiX
- **R** (Underlying language of BiomiX and its Shiny Preview-QC interface) — https://github.com/IxI-97/BiomiX

## Examples

```
bash Biomix_linux_exe.sh
```

## Evaluation signals

- Subgroup PCA/UMAP (Preview-QC interface) shows clear separation of positive and negative samples, reproducing Figure S1
- ifn_subgroup_enrichment.csv contains significant hits (FDR-adjusted p-value below 0.05) for IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling terms
- Every sample receives exactly one subgroup label based on the Z-score > 1 activity-score criterion
- Per-subgroup DESeq2/Limma contrasts run without errors and produce gene lists of sufficient size for EnrichR/GSEA submission

## Limitations

- Users are required to carefully review datasets before use; BiomiX performs no automated validation ('We strongly recommend users to follow the guidelines and to carefully review the dataset before using BiomiX.')
- Threshold choice (Z-score > 1 labeling) is user-defined and must match the Kirou-style activity-score criteria; results depend on this cutoff
- Transcriptomics sample filtering had a known bug in an earlier release, fixed in the 4 April 2025 update

## Evidence
