---
name: fdr-multiple-testing-correction
description: Use when testing hundreds to thousands of omics features (metabolomics
  peaks, RNA-seq genes) for group differences in BiomiX-style pipelines and raw p-values
  must be FDR-corrected before filtering at p.adj thresholds.
when_to_use_negative:
- Input is already a table of adjusted p-values or pre-filtered significant hits —
  re-correcting would be invalid
- Few (< ~10) hypothesis tests where family-wise correction (e.g. Bonferroni) is more
  appropriate than FDR
- Raw metabolomics peak or RNA-seq count matrices that have not yet been tested —
  FDR correction is downstream of the Wilcoxon/DESeq2 test step
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: R-based GUI pipeline whose metabolomics module performs Wilcoxon tests with
    FDR correction and whose transcriptomics module (DESeq2/Limma) applies Benjamini-Hochberg
    adjusted p filtering
  repo: https://github.com/IxI-97/BiomiX
- name: DESeq2
  role: Supplies raw p-values from the negative-binomial differential expression model
    that are then BH-adjusted and filtered at p.adj < 0.05
- name: R
  role: Implementation environment for the FDR correction (e.g. p.adjust with method
    'BH') within the BiomiX pipeline
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:37453db15ae692f12a2b55044de0428d16dda11be83ef33d9a9792cb9b91f41a
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
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fdr-multiple-testing-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply False Discovery Rate (Benjamini-Hochberg) correction to raw p-values from high-throughput omics tests — Wilcoxon tests on metabolomics peaks and DESeq2/Limma tests on transcriptomics counts — so that thousands of simultaneous hypotheses can be filtered at controlled adjusted-p thresholds.

## When to use

Use when testing hundreds to thousands of omics features (metabolomics peaks, RNA-seq genes) for group differences in BiomiX-style pipelines and raw p-values must be FDR-corrected before filtering at p.adj thresholds.

## When NOT to use

- Input is already a table of adjusted p-values or pre-filtered significant hits — re-correcting would be invalid
- Few (< ~10) hypothesis tests where family-wise correction (e.g. Bonferroni) is more appropriate than FDR
- Raw metabolomics peak or RNA-seq count matrices that have not yet been tested — FDR correction is downstream of the Wilcoxon/DESeq2 test step

## Inputs

- Vector or table of raw (uncorrected) p-values, one per omics feature
- Metabolomics peak matrix (PTB vs HC plasma intensities) with group-conditioned sample metadata
- DESeq2 results table with gene, log2FC, p-value columns from the PRJNA971365 whole-blood RNA-seq analysis

## Outputs

- Results table with an FDR-adjusted p-value (p.adj / BH) column for every input feature
- Significant-feature subset filtered at |log2FC| > 0.5 and p.adj < 0.1 (metabolomics) or |log2FC| > 1 and p.adj < 0.05 (transcriptomics)
- Adjusted p-values used for volcano plot y-axis (-log10(p.adj)) and for EnrichR/GSEA input ranking

## How to apply

Run the per-feature omnics test first: in the BiomiX metabolomics block, test each metabolic feature for a group difference (PTB vs HC) using the Wilcoxon (Mann-Whitney) rank-sum test on peak intensities; in the transcriptomics block, obtain p-values from DESeq2's negative-binomial model fit with CONDITION plus SEX and AGE covariates. Then correct the resulting p-value vector with the False Discovery Rate method (Benjamini-Hochberg), attaching the p.adj column to every tested feature — no feature should be dropped before correction. Apply the omics-specific significance cutoffs afterwards: |log2FC| > 0.5 and p.adj < 0.1 for metabolomics peaks, |log2FC| > 1 and p.adj < 0.05 for DESeq2 transcriptomics genes. Finally feed the significant feature set forward to EnrichR/GSEA pathway analysis; rationale is that BiomiX's own pipelines evaluate metabolomics peak differences 'via the Wilcoxon test with the False Discovery Rate correction' and filter DESeq2 results on BH-adjusted p-values.

## Related tools

- **BiomiX** (R-based GUI pipeline whose metabolomics module performs Wilcoxon tests with FDR correction and whose transcriptomics module (DESeq2/Limma) applies Benjamini-Hochberg adjusted p filtering) — https://github.com/IxI-97/BiomiX
- **DESeq2** (Supplies raw p-values from the negative-binomial differential expression model that are then BH-adjusted and filtered at p.adj < 0.05)
- **R** (Implementation environment for the FDR correction (e.g. p.adjust with method 'BH') within the BiomiX pipeline)

## Examples

```
res <- DESeq2::results(dds, contrast=c("CONDITION","PTB","HC")); res$p.adj <- p.adjust(res$pvalue, method="BH"); sig <- subset(res, abs(log2FoldChange)>1 & p.adj<0.05)
```

## Evaluation signals

- Adjusted p-values all lie in [0,1] and are monotonically non-decreasing relative to the raw p-values (p.adj >= p for every feature)
- Output table retains a p.adj row for every input feature (correction is applied before filtering, not after)
- Metabolomics significant set satisfies |log2FC| > 0.5 and p.adj < 0.1; transcriptomics set satisfies |log2FC| > 1 and p.adj < 0.05
- EnrichR/GSEA on the FDR-filtered feature set recovers the reported pathway terms (e.g. sphingolipid signaling, retrograde endocannabinoid signaling, arachidonic acid metabolic process)
- Volcano plot y-axis (-log10(p.adj)) matches the p.adj column in the results CSV

## Limitations

- BiomiX strongly recommends users carefully review the dataset before use; there is no automated validation of statistical inputs or outputs
- Transcriptomics sample filtering (an advanced option affecting which tests are run and thus the p-value vector) had a bug fixed only in the v2.5-era releases; Mac installation had known issues
- FDR correction does not handle effect-size direction; both |log2FC| and p.adj thresholds are needed jointly for significance calls

## Evidence
