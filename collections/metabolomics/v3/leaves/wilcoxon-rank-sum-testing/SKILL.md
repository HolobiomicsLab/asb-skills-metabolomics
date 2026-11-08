---
name: wilcoxon-rank-sum-testing
description: Use when comparing two unpaired groups on numeric features (metabolomics
  peak intensities, omics abundances) or when relating a MOFA factor to binary clinical
  metadata, without assuming normality.
when_to_use_negative:
- The comparison involves more than two groups or paired/matched samples — the Wilcoxon
  rank-sum test as implemented here covers two unpaired groups only
- The goal is RNA-seq count differential expression, for which BiomiX uses DESeq2/Limma
  instead of the metabolomics Wilcoxon pipeline
- Input is transcriptomics count data or methylation arrays, handled by DESeq2/Limma
  or ChAMP respectively rather than peak-level Wilcoxon testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: R tool whose metabolomics module runs Wilcoxon tests with FDR correction on
    peak matrices and whose MOFA module Wilcoxon-tests binary clinical features against
    factors
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Runtime environment in which the BiomiX Wilcoxon/MOFA factor clinical-correlation
    analysis executes
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Supplies the factor values that are tested against binary clinical data using
    the Wilcoxon rank-sum test
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:7c16ff8b4dec33d2fd4a795931d0517bcd06e6dbe763781e815dca9bf2173474
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
  - MOFA (Multi-Omics Factor Analysis)
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# wilcoxon-rank-sum-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-parametric Wilcoxon (Mann-Whitney) rank-sum testing as implemented in BiomiX to detect differential omics features between two groups (e.g., PTB vs HC metabolomics peaks) and to test binary clinical variables against MOFA factors, with False Discovery Rate (Benjamini-Hochberg) correction of p-values.

## When to use

Use when comparing two unpaired groups on numeric features (metabolomics peak intensities, omics abundances) or when relating a MOFA factor to binary clinical metadata, without assuming normality.

## When NOT to use

- The comparison involves more than two groups or paired/matched samples — the Wilcoxon rank-sum test as implemented here covers two unpaired groups only
- The goal is RNA-seq count differential expression, for which BiomiX uses DESeq2/Limma instead of the metabolomics Wilcoxon pipeline
- Input is transcriptomics count data or methylation arrays, handled by DESeq2/Limma or ChAMP respectively rather than peak-level Wilcoxon testing

## Inputs

- Metabolomics (or other omics) peak/intensity matrix with sample metadata carrying group condition labels (e.g., PTB vs HC)
- MOFA factor values matrix per sample
- Clinical metadata table with numeric and binary clinical features

## Outputs

- Per-feature table with log2FC, Wilcoxon statistic, raw p-value, and FDR-adjusted p-value (e.g., mofa_factor_clinical_correlations.csv for the MOFA module)
- Filtered list of significant features at |log2FC| > 0.5 and adjusted p < 0.1, annotated via CEU Mass Mediator m/z lookup

## How to apply

In the BiomiX metabolomics pipeline, compute per-feature log2 fold change as the log2 of the ratio of median peak intensities between the case and control groups, then test each feature for a group difference with the Wilcoxon rank-sum test and correct p-values with the False Discovery Rate method; the BiomiX case study used |log2FC| > 0.5 and adjusted p < 0.1 to call significant peaks (e.g., reduced acylcarnitine, PC, LysoPE and TG in PTB plasma vs healthy controls). In the MOFA factor interpretation module, apply the same test to binary clinical features by splitting samples into positive/negative groups per factor value, while numeric clinical features are handled separately by Pearson correlation; all raw p-values from both test types are pooled and corrected with Benjamini-Hochberg. Results are assembled into a per-feature or per-factor table with test type, statistic, raw p-value, and FDR-adjusted p-value so discriminant features/factors can be ranked.

## Related tools

- **BiomiX** (R tool whose metabolomics module runs Wilcoxon tests with FDR correction on peak matrices and whose MOFA module Wilcoxon-tests binary clinical features against factors) — https://github.com/IxI-97/BiomiX
- **R** (Runtime environment in which the BiomiX Wilcoxon/MOFA factor clinical-correlation analysis executes) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Supplies the factor values that are tested against binary clinical data using the Wilcoxon rank-sum test)

## Evaluation signals

- Output table contains log2FC and FDR-adjusted p-value columns for every input peak, with all adjusted p-values in [0,1]
- Significant-feature set at |log2FC| > 0.5 and adjusted p < 0.1 recovers the reported enriched pathways (sphingolipid signaling, retrograde endocannabinoid signaling, etc.) after EnrichR/GSEA
- For the MOFA module, the CSV contains an entry for every factor-feature combination tested and includes a BH-adjusted p-value column
- PTB plasma metabolomics reproduces reported reductions in acylcarnitine, PC, LysoPE and TG
- Both Pearson (numeric) and Wilcoxon (binary) p-values are pooled into a single Benjamini-Hochberg correction step

## Limitations

- BiomiX does not support instrument normalization for metabolomics; QC samples must be labelled in the metadata (CONDITION = 'QC') for preview and removed before testing
- Users are strongly recommended to carefully review the dataset before use; BiomiX performs no automated validation of input data
- BiomiX is described as a first attempt rather than a definitive solution, with limited omics analysers (proteomics and genomics not yet supported)

## Evidence

- [other] BiomiX correlates numerical clinical data with each MOFA factor using Pearson correlation, tests binary clinical data with the Wilcoxon test on positive/negative groups: [evidence span withheld — anchor status: not_found]
- [other] Test each feature for a group difference using the Wilcoxon (Mann-Whitney) rank-sum test and correct the resulting p-values with the False Discovery Rate method: [evidence span withheld — anchor status: not_found]
- [other] the output table must contain log2FC and FDR-adjusted p-value columns for every input peak, with adjusted p-values in [0,1]: [evidence span withheld — anchor status: not_found]
