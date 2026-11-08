---
name: median-log2foldchange-computation
description: Use when comparing untargeted metabolomics peak matrices between two
  condition groups and a per-feature effect size is needed before significance filtering
  (e.g., |log2FC| > 0.5, p.adj < 0.1).
when_to_use_negative:
- Input is transcriptomics count data — BiomiX uses DESeq2/Limma for differential
  transcriptomics rather than median-ratio log2FC with Wilcoxon tests
- The comparison involves more than two groups or repeated measures — the median-ratio
  log2FC formulation is defined for a two-group contrast
- Instrument-level normalization of metabolomics is required — BiomiX does not support
  instrument normalization for metabolomics
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: R-based Shiny toolkit providing the imputation interface and metabolomics
    single-omics pipeline within which the median log2FC computation and Wilcoxon/FDR
    testing are executed
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Implementation language for computing per-feature median-ratio log2FC and
    Wilcoxon rank-sum tests with FDR correction
- name: EnrichR
  role: Receives the annotated significant metabolite list for downstream pathway
    enrichment validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:3f31a015b4fbafddb1550eca58e7f682bfd56a9ebcc4119fc9f6fdd90aeab2c3
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
  - EnrichR
  - GSEA
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# median-log2foldchange-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-feature log2 fold changes from the ratio of median peak intensities between two biological groups (e.g., tuberculosis patients vs healthy controls) as the effect-size step of the BiomiX metabolomics differential analysis pipeline, paired with Wilcoxon rank-sum testing and FDR correction.

## When to use

Use when comparing untargeted metabolomics peak matrices between two condition groups and a per-feature effect size is needed before significance filtering (e.g., |log2FC| > 0.5, p.adj < 0.1).

## When NOT to use

- Input is transcriptomics count data — BiomiX uses DESeq2/Limma for differential transcriptomics rather than median-ratio log2FC with Wilcoxon tests
- The comparison involves more than two groups or repeated measures — the median-ratio log2FC formulation is defined for a two-group contrast
- Instrument-level normalization of metabolomics is required — BiomiX does not support instrument normalization for metabolomics

## Inputs

- Metabolomics peak intensity matrix (features x samples) from LC-MS untargeted analysis (BiomiX example tuberculosis plasma metabolomics matrix)
- Sample metadata table with CONDITION group labels (e.g., PTB vs HC)
- Optionally imputed peak matrix (NIPALS, random forest, lasso, or mean/median imputation)

## Outputs

- Per-feature results table with log2FC and FDR-adjusted p-value columns for every input peak
- Filtered list of significant peaks (|log2FC| > 0.5 and adjusted p < 0.1) annotated by m/z via CEU Mass Mediator

## How to apply

First impute missing peak values in the metabolomics block using one of BiomiX's imputation options (NIPALS, random forest, lasso, or mean/median). For each metabolic feature, compute the log2 of the ratio of median peak intensities between the case group (e.g., PTB) and the control group (HC), using the median to resist outlier intensities in untargeted LC-MS data. Test each feature for a group difference with the Wilcoxon (Mann-Whitney) rank-sum test and correct p-values with the False Discovery Rate method. Filter significant peaks using |log2FC| > 0.5 and adjusted p < 0.1, then annotate by m/z via the CEU Mass Mediator database. The output table must retain log2FC and FDR-adjusted p-value columns for every input peak, with adjusted p-values bounded in [0,1].

## Related tools

- **BiomiX** (R-based Shiny toolkit providing the imputation interface and metabolomics single-omics pipeline within which the median log2FC computation and Wilcoxon/FDR testing are executed) — https://github.com/IxI-97/BiomiX
- **R** (Implementation language for computing per-feature median-ratio log2FC and Wilcoxon rank-sum tests with FDR correction)
- **EnrichR** (Receives the annotated significant metabolite list for downstream pathway enrichment validation)

## Examples

```
log2fc <- log2(tapply(peak_matrix[, f], group, median)["PTB"] / tapply(peak_matrix[, f], group, median)["HC"]); p.adj <- p.adjust(wilcox.test(peak_matrix[, f] ~ group)$p.value, method="BH")
```

## Evaluation signals

- The output table contains log2FC and FDR-adjusted p-value columns for every input peak (no features dropped silently)
- All FDR-adjusted p-values fall within [0,1]
- Significant feature set respects the thresholds |log2FC| > 0.5 and adjusted p < 0.1
- Reported pathway terms (sphingolipid signaling, retrograde endocannabinoid signaling) are recovered when the significant feature set is submitted to EnrichR/GSEA
- log2FC values are consistent with the direction of the PTB-vs-HC median intensity ratio for spot-checked features

## Limitations

- BiomiX does not support instrument normalization for metabolomics; QC samples can only be previewed in the Preview-QC interface before being removed automatically afterwards
- BiomiX is described as a first attempt at the gold-standard single-omics pipeline, not a definitive solution; users must carefully review datasets before use as no automated validation is performed
- Mac installation has known issues; Linux or Windows are recommended

## Evidence
