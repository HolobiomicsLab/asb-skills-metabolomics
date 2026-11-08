---
name: lipid-class-annotation-against-metlin-hmdb
description: Use when a peak-aligned, TIC-normalized SCMS feature table from two cell
  phenotypes (e.g., primary vs metastatic melanoma, ± drug treatment) needs differential
  lipid species filtered at fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05 and annotated
  to lipid classes.
when_to_use_negative:
- Input is raw single-cell spectra not yet pretreated — peak export, noise/background
  subtraction, TIC normalization, and Geena2 alignment (10 ppm) must be done first
- The goal is cell phenotype classification (primary vs metastatic) rather than differential
  abundance/biomarker discovery — use the MetaPhenotype model instead
- Feature table is from bulk (pooled) metabolomics rather than Single-probe SCMS single-cell
  data, since cell-heterogeneity-resolving SCMS measurements are the intended input
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: Geena2
  role: Online bioinformatics tool used for peak alignment of SCMS peaks with a mass
    tolerance of 10 ppm in the upstream pretreatment that produces the feature table
- name: MetaboAnalyst 6.0
  role: Used to render volcano plots of log2 fold change versus -log10 FDR-adjusted
    p-value for differential species
- name: METLIN
  role: Online database used to tentatively label/annotate the detected lipid species
- name: HMDB
  role: Online database used to tentatively label/annotate the detected lipid species
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:d208d2715e4bcdab0f96ebd7ee19c7c632624a56b6764be5ac66e0e38b699514
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - HMDB
  - METLIN
  - MetaPhenotype
  - MetaboAnalyst 6.0
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lipid-class-annotation-against-metlin-hmdb

## Summary

Identifies differentially abundant lipid species from single-cell mass spectrometry (SCMS) metabolomics feature tables via volcano-plot analysis (log2 fold change vs -log10 FDR-adjusted p-value) and tentatively annotates their lipid classes against METLIN and HMDB to nominate metabolite biomarker candidates.

## When to use

Use when a peak-aligned, TIC-normalized SCMS feature table from two cell phenotypes (e.g., primary vs metastatic melanoma, ± drug treatment) needs differential lipid species filtered at fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05 and annotated to lipid classes.

## When NOT to use

- Input is raw single-cell spectra not yet pretreated — peak export, noise/background subtraction, TIC normalization, and Geena2 alignment (10 ppm) must be done first
- The goal is cell phenotype classification (primary vs metastatic) rather than differential abundance/biomarker discovery — use the MetaPhenotype model instead
- Feature table is from bulk (pooled) metabolomics rather than Single-probe SCMS single-cell data, since cell-heterogeneity-resolving SCMS measurements are the intended input

## Inputs

- Peak-aligned, TIC-normalized SCMS metabolomics feature table for primary/metastatic melanoma cell pairs (control and vemurafenib-treated), produced by the established pretreatment protocol (Geena2 alignment, 10 ppm mass tolerance)
- Group labels distinguishing primary vs metastatic cells per pair and treatment condition
- METLIN and HMDB online databases for tentative species annotation

## Outputs

- Volcano plots (log2 fold change vs -log10 FDR-adjusted p-value), one per cell pair/treatment condition, with lipid classes annotated
- Filtered list of significant lipid species meeting fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05
- Tentative lipid-class annotations (e.g., TG, Cer, SM, PG, PE) for significant species, tabulated as metabolite biomarker candidates (Tables S4/S5 equivalents)

## How to apply

Start from the pretreated SCMS feature table produced by the established protocol (Geena2 peak alignment with 10 ppm mass tolerance, noise/background subtraction, TIC normalization). Compute log2 fold change of each lipid species abundance between primary and metastatic groups for each cell pair and treatment condition (control and vemurafenib-treated), and derive FDR-adjusted p-values from the statistical tests. Filter species to fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05, then render volcano plots (log2 fold change versus -log10 FDR-adjusted p-value) using MetaboAnalyst 6.0 or an equivalent Python plotting tool, one plot per pair/condition. Tentatively annotate the significant species' lipid classes (e.g., TG, Cer, SM, PG, PE) against METLIN and HMDB, tabulating them as biomarker candidates analogous to Tables S4/S5.

## Related tools

- **Geena2** (Online bioinformatics tool used for peak alignment of SCMS peaks with a mass tolerance of 10 ppm in the upstream pretreatment that produces the feature table)
- **MetaboAnalyst 6.0** (Used to render volcano plots of log2 fold change versus -log10 FDR-adjusted p-value for differential species)
- **METLIN** (Online database used to tentatively label/annotate the detected lipid species)
- **HMDB** (Online database used to tentatively label/annotate the detected lipid species)

## Evaluation signals

- Volcano plots visually match the reported differential species sets in Tables S4/S5 (TG, Cer, SM, PG, PE members)
- Applying fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05 thresholds reproduces the reported biomarker candidates for each cell pair/condition
- Tentative annotation against METLIN/HMDB assigns the expected lipid classes (TG, Cer, SM, PG, PE) to the significant species
- One volcano plot is generated per cell pair and treatment condition (control and vemurafenib-treated), with no conditions missing
- Input feature table reflects the required pretreatment: Geena2 peak alignment with 10 ppm mass tolerance and TIC normalization

## Limitations

- Species annotations are only tentative, as METLIN and HMDB were used to 'tentatively label the detected species'
- The analysis depends on the previously established SCMS pretreatment protocol; misalignment or normalization errors propagate into the differential results
- PCA alone cannot effectively discriminate phenotypically similar cells (e.g., IGR39/IGR37), so discrimination relies on the modeled/filtered species rather than unsupervised plots

## Evidence
