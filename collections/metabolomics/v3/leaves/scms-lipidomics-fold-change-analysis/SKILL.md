---
name: scms-lipidomics-fold-change-analysis
description: Use when a peak-aligned, TIC-normalized SCMS lipid/metabolomics feature
  table must be screened for species differing between two cell phenotypes (e.g.,
  primary vs metastatic) under control and drug-treated conditions to nominate biomarker
  candidates.
when_to_use_negative:
- The input is raw, unprocessed single-cell spectra rather than a pretreated feature
  table — the established pretreatment protocol (Geena2 alignment at 10 ppm, TIC normalization)
  must run first.
- The goal is phenotype classification rather than differential abundance screening
  — use the MetaPhenotype meta-learning classifier, since PCA cannot effectively discriminate
  closely similar phenotypes such as IGR39/IGR37.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: MetaboAnalyst 6.0
  role: Renders volcano plots (log2 fold change vs -log10 FDR-adjusted p-value) and
    performs the statistical comparison of lipid species between groups
- name: Geena2
  role: Produces the peak-aligned feature table (10 ppm mass tolerance) that feeds
    the differential abundance analysis
- name: METLIN
  role: Tentative annotation of significantly different lipid species by database
    lookup
- name: HMDB
  role: Tentative annotation of significantly different lipid species by database
    lookup
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:fb86d01fd92d20cb81e1148d646f4ef80bf0d13f25cea9d7258eb8f5f0183942
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - HMDB
  - METLIN
  - MetaPhenotype
  - MetaboAnalyst 6.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 38
---

# scms-lipidomics-fold-change-analysis

## Summary

Volcano-plot differential abundance analysis of Single-probe SCMS metabolomics feature tables to prioritize lipid biomarker candidates between primary and metastatic melanoma cells, applying fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05 thresholds.

## When to use

Use when a peak-aligned, TIC-normalized SCMS lipid/metabolomics feature table must be screened for species differing between two cell phenotypes (e.g., primary vs metastatic) under control and drug-treated conditions to nominate biomarker candidates.

## When NOT to use

- The input is raw, unprocessed single-cell spectra rather than a pretreated feature table — the established pretreatment protocol (Geena2 alignment at 10 ppm, TIC normalization) must run first.
- The goal is phenotype classification rather than differential abundance screening — use the MetaPhenotype meta-learning classifier, since PCA cannot effectively discriminate closely similar phenotypes such as IGR39/IGR37.

## Inputs

- Peak-aligned, TIC-normalized SCMS metabolomics feature table (primary/metastatic melanoma cell pairs, control and vemurafenib-treated)
- Group labels for cells (primary vs metastatic, treatment condition)
- Reference databases METLIN and HMDB for tentative species annotation

## Outputs

- Volcano plots (log2 fold change vs -log10 FDR-adjusted p-value) per cell pair and treatment condition
- List of significant lipid species meeting fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05 with tentative lipid-class annotations (Tables S4/S5 equivalents)

## How to apply

Start from the pretreated SCMS feature table produced by the established protocol (noise/background subtraction, TIC normalization, Geena2 peak alignment with 10 ppm mass tolerance, missing-value removal). For each cell pair and condition, compute log2 fold change of each lipid species abundance between primary and metastatic groups and derive FDR-adjusted p-values from the group comparison. Filter species to fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05, then render volcano plots (log2 fold change vs -log10 FDR-adjusted p-value), one per pair/condition, using MetaboAnalyst 6.0 or an equivalent Python plotting tool with lipid classes annotated. Tentatively annotate the significant species (e.g., TG, Cer, SM, PG, PE classes) against METLIN and HMDB to reproduce the differential species tabulated in Tables S4/S5 as metabolite biomarker candidates.

## Related tools

- **MetaboAnalyst 6.0** (Renders volcano plots (log2 fold change vs -log10 FDR-adjusted p-value) and performs the statistical comparison of lipid species between groups)
- **Geena2** (Produces the peak-aligned feature table (10 ppm mass tolerance) that feeds the differential abundance analysis)
- **METLIN** (Tentative annotation of significantly different lipid species by database lookup)
- **HMDB** (Tentative annotation of significantly different lipid species by database lookup)

## Evaluation signals

- All reported differential species belong to the TG, Cer, SM, PG, and PE lipid classes, matching Tables S4/S5.
- Every species in the candidate list passes both thresholds: fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05.
- Volcano plot axes correspond to log2 fold change (x) and -log10 FDR-adjusted p-value (y), with one plot per cell pair/condition.
- Significant species are tentatively annotated against METLIN/HMDB rather than left unlabelled m/z features.

## Limitations

- Volcano-plot analysis is an univariate pairwise test; it does not capture cell-to-cell heterogeneity, which conventional bulk-style summaries cannot evaluate.
- Species assignments are only tentative labels against METLIN/HMDB, not confirmed identifications.
- Differential-abundance statistics on large SCMS feature tables can be ill-suited to simple conventional methods when the goal shifts to classification; machine learning (MetaPhenotype) was needed for phenotype prediction.

## Evidence
