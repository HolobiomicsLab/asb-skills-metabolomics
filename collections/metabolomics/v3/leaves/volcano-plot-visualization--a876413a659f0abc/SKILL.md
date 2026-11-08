---
name: volcano-plot-visualization
description: Use when a peak-aligned, TIC-normalized single-cell (SCMS) metabolomics
  feature table must be compared between two biological groups (e.g., primary vs metastatic
  melanoma cells, ± vemurafenib) to flag significantly altered lipid species as biomarker
  candidates.
when_to_use_negative:
- When the goal is cell phenotype classification rather than differential species
  identification — use a supervised model (e.g., MetaPhenotype) instead, since PCA
  also cannot discriminate highly similar metabolomic profiles such as IGR39/IGR37
  cells
- When the input is raw single-cell spectra rather than a pretreated feature table
  — noise/background subtraction, TIC normalization, and Geena2 10 ppm alignment must
  be completed first
- When annotations must be definitive rather than tentative — METLIN/HMDB assignments
  in this workflow are explicitly tentative labels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: MetaboAnalyst 6.0
  role: Renders the volcano plots (log2 fold change versus -log10 FDR-adjusted p-value)
    from the filtered differential table; the article also uses it for PCA on the
    same SCMS data
- name: Geena2
  role: Upstream peak alignment of single-cell feature lists with a 10 ppm mass tolerance,
    producing the table the volcano analysis consumes
- name: METLIN
  role: Online database used to tentatively label the detected differential species
- name: HMDB
  role: Online database used to tentatively label the detected differential species
- name: MetaPhenotype
  role: Complementary meta-learning model that selects essential species critical
    for phenotype classification; its top species can be cross-referenced with volcano-plot
    hits
  repo: github.com/songyuan93/MetaPhenotype
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:16d4d8bd9306f7678e598d7e8d99b93732cdb9feadfb0cd6381e086779032704
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - HMDB
  - METLIN
  - MetaPhenotype
  - MetaboAnalyst 6.0
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# volcano-plot-visualization

## Summary

Volcano-plot differential abundance analysis of SCMS lipid feature tables to nominate metabolite biomarker candidates: species are filtered on fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05, plotted as log2 fold change versus -log10 FDR-adjusted p-value, and tentatively annotated by lipid class against METLIN/HMDB.

## When to use

Use when a peak-aligned, TIC-normalized single-cell (SCMS) metabolomics feature table must be compared between two biological groups (e.g., primary vs metastatic melanoma cells, ± vemurafenib) to flag significantly altered lipid species as biomarker candidates.

## When NOT to use

- When the goal is cell phenotype classification rather than differential species identification — use a supervised model (e.g., MetaPhenotype) instead, since PCA also cannot discriminate highly similar metabolomic profiles such as IGR39/IGR37 cells
- When the input is raw single-cell spectra rather than a pretreated feature table — noise/background subtraction, TIC normalization, and Geena2 10 ppm alignment must be completed first
- When annotations must be definitive rather than tentative — METLIN/HMDB assignments in this workflow are explicitly tentative labels

## Inputs

- Peak-aligned, TIC-normalized SCMS metabolomics/lipidomics feature table (primary/metastatic melanoma cell pairs, control and vemurafenib-treated)
- Group labels per single-cell sample (primary vs metastatic, per condition)
- Statistical test results per feature (log2 fold change and FDR-adjusted p-value)

## Outputs

- One volcano plot per cell pair/treatment condition (log2 FC vs -log10 FDR-adjusted p)
- Filtered list of significant lipid species (fold change ≥ 2.0, FDR ≤ 0.05)
- Tentative lipid-class annotations (TG, Cer, SM, PG, PE) for significant species, cross-checked against METLIN/HMDB
- Biomarker-candidate tables (reproducing Tables S4/S5)

## How to apply

Start from the feature table produced by the established SCMS pretreatment protocol (Geena2 peak alignment with 10 ppm mass tolerance, noise/background subtraction, TIC normalization, missing-value removal). For each cell pair and treatment condition, compute the log2 fold change of each lipid species between the primary and metastatic groups and derive FDR-adjusted p-values from the statistical tests. Filter species to fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05, then render one volcano plot per pair/condition (log2 FC versus -log10 FDR p) using MetaboAnalyst 6.0 or an equivalent Python plotting tool, annotating significant species by lipid class. Tentatively assign lipid classes (e.g., TG, Cer, SM, PG, PE) by querying METLIN and HMDB. Validation: the significant species and their lipid-class composition should reproduce the reported differential species tabulated in Tables S4/S5.

## Related tools

- **MetaboAnalyst 6.0** (Renders the volcano plots (log2 fold change versus -log10 FDR-adjusted p-value) from the filtered differential table; the article also uses it for PCA on the same SCMS data)
- **Geena2** (Upstream peak alignment of single-cell feature lists with a 10 ppm mass tolerance, producing the table the volcano analysis consumes)
- **METLIN** (Online database used to tentatively label the detected differential species)
- **HMDB** (Online database used to tentatively label the detected differential species)
- **MetaPhenotype** (Complementary meta-learning model that selects essential species critical for phenotype classification; its top species can be cross-referenced with volcano-plot hits) — github.com/songyuan93/MetaPhenotype

## Examples

```
import numpy as np, matplotlib.pyplot as plt; plt.scatter(log2FC, -np.log10(fdr_p)); plt.axhline(-np.log10(0.05)); plt.axvline(1); plt.axvline(-1)
```

## Evaluation signals

- Plot axes are exactly log2 fold change (x) versus -log10 FDR-adjusted p-value (y), with one plot generated per cell pair/condition
- The filtered significant set applies fold change ≥ 2.0 AND FDR-adjusted p ≤ 0.05 — species failing either cutoff are excluded
- Significant species' lipid classes match the reported differential classes (TG, Cer, SM, PG, PE) in Tables S4/S5
- Every annotated species has a tentative METLIN or HMDB assignment recorded
- The feature table derives from the established pretreatment chain (Geena2 10 ppm alignment, TIC normalization) — re-running the workflow on the same table should reproduce the same significant-species list

## Limitations

- Species annotations are only tentative labels from METLIN/HMDB, not confirmed identities
- Results depend on the upstream pretreatment (Geena2 10 ppm alignment, TIC normalization); misalignment propagates into fold-change estimates
- Ambient SCMS methods yield relatively small numbers of cells, which limits statistical power for the per-condition differential tests

## Evidence
