---
name: subgroup-differential-expression-analysis
description: Use when a user-defined gene panel (e.g., 26 IFN-induced genes) should
  partition samples into activity-score-positive/negative subgroups, and per-subgroup
  differential expression plus pathway exploration (EnrichR/GSEA) is the analysis
  goal.
when_to_use_negative:
- The analysis goal is whole-condition contrast (e.g., PTB vs HC) without a gene-panel-defined
  subpopulation — the standard BiomiX transcriptomics DESeq2 pipeline is the right
  path instead
- The omics data type has no gene-panel equivalent (e.g., metabolomics peaks), since
  subgrouping here depends on gene Z activity scores
- Samples are unannotated/unlabelled data outside the BiomiX single-omics transcriptomics
  pipeline scope
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: Runs the gene-panel subpopulation analysis and Preview-QC subgroup visualization
    within a Shiny GUI
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: Per-subgroup differential expression on count data contrasting positive vs
    negative subgroups
  repo: https://github.com/IxI-97/BiomiX
- name: Limma
  role: Alternative differential expression engine for normalized (non-count) subgroup
    matrices
  repo: https://github.com/IxI-97/BiomiX
- name: EnrichR
  role: Biological process exploration on significant gene lists from each subgroup
  repo: https://github.com/IxI-97/BiomiX
- name: GSEA
  role: Pathway-level enrichment complementing EnrichR for subgroup gene lists
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Language in which the whole subgroup pipeline (Z-scoring, DESeq2/Limma, EnrichR/GSEA
    calls) is implemented
  repo: https://github.com/IxI-97/BiomiX
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:354f4ab602dbedd439eb5081ad664639e4f4021cbf0fabdf6b675efa33d53a54
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# subgroup-differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

BiomiX-style subpopulation analysis that stratifies samples into user-defined subgroups via a gene-panel Z activity score and then runs separate differential expression and enrichment analyses per subgroup. It powers findings such as separating PTB patients with a 26 IFN-induced gene panel.

## When to use

Use when a user-defined gene panel (e.g., 26 IFN-induced genes) should partition samples into activity-score-positive/negative subgroups, and per-subgroup differential expression plus pathway exploration (EnrichR/GSEA) is the analysis goal.

## When NOT to use

- The analysis goal is whole-condition contrast (e.g., PTB vs HC) without a gene-panel-defined subpopulation — the standard BiomiX transcriptomics DESeq2 pipeline is the right path instead
- The omics data type has no gene-panel equivalent (e.g., metabolomics peaks), since subgrouping here depends on gene Z activity scores
- Samples are unannotated/unlabelled data outside the BiomiX single-omics transcriptomics pipeline scope

## Inputs

- Preprocessed transcriptomics expression matrix (raw RNA-seq counts or normalized matrix, e.g., whole-blood PTB dataset from ENA PRJNA971365)
- User-defined gene panel (e.g., list of 26 IFN-induced genes)
- Sample metadata with subgroup-relevant columns (e.g., CONDITION, SEX, AGE)

## Outputs

- Subgroup assignment table (positive vs negative samples per activity-score criteria)
- PCA/UMAP plot showing subgroup separation (e.g., Figure S1 equivalent)
- Per-subgroup differential expression results (DESeq2/Limma: gene, log2FC, p-value, p.adj)
- Subgroup enrichment table (ifn_subgroup_enrichment.csv) with EnrichR/GSEA biological process terms

## How to apply

Load the preprocessed expression matrix restricted to the user-defined panel genes (e.g., the 26 IFN-induced genes) with aligned sample metadata into R/BiomiX. Z-score normalize each panel gene across samples and compute a per-sample activity score, labeling samples positive when at least one panel gene has Z-score > 1 (Kirou-style criteria). Visualize the subgroup separation with PCA/UMAP in the Preview-QC interface (Figure S1 style). Then run differential expression contrasting the positive vs negative subgroups with DESeq2 for count data (or Limma for normalized matrices), submitting significant gene lists per subgroup to EnrichR and GSEA. In the PTB application this recovered IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling enrichments, written to ifn_subgroup_enrichment.csv.

## Related tools

- **BiomiX** (Runs the gene-panel subpopulation analysis and Preview-QC subgroup visualization within a Shiny GUI) — https://github.com/IxI-97/BiomiX
- **DESeq2** (Per-subgroup differential expression on count data contrasting positive vs negative subgroups) — https://github.com/IxI-97/BiomiX
- **Limma** (Alternative differential expression engine for normalized (non-count) subgroup matrices) — https://github.com/IxI-97/BiomiX
- **EnrichR** (Biological process exploration on significant gene lists from each subgroup) — https://github.com/IxI-97/BiomiX
- **GSEA** (Pathway-level enrichment complementing EnrichR for subgroup gene lists) — https://github.com/IxI-97/BiomiX
- **R** (Language in which the whole subgroup pipeline (Z-scoring, DESeq2/Limma, EnrichR/GSEA calls) is implemented) — https://github.com/IxI-97/BiomiX

## Examples

```
Rscript -e 'res <- DESeq2::results(dds, contrast=c("IFN_subgroup","positive","negative")); write.csv(res, "ifn_subgroup_enrichment.csv")'
```

## Evaluation signals

- Subgroup PCA/UMAP shows clear separation of activity-score-positive and negative samples (Figure S1 equivalent)
- ifn_subgroup_enrichment.csv contains significant hits (FDR-adjusted p-value < 0.05) for IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling
- Every positive sample has at least one panel gene with Z-score > 1; negative samples have none, matching the Kirou-style activity-score criteria
- DESeq2/Limma output tables carry the required fields (gene, log2FC, p-value, p.adj) with subgroup contrast correctly specified
- Enrichment results are consistent across Gene Ontology and Reactome databases, as reported for the PTB dataset

## Limitations

- Subgrouping depends on the user-provided gene panel; a poorly chosen panel will not separate the populations
- Mac installation had known issues in BiomiX 2.4/2.5; Linux or Windows is recommended for running the pipeline
- Users must carefully review datasets before use — BiomiX performs no automated dataset validation
- Count-based DESeq2 requires raw counts; Limma must be substituted for already-normalized matrices

## Evidence
