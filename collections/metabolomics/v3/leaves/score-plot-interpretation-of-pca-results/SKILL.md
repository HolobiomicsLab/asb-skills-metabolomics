---
name: score-plot-interpretation-of-pca-results
description: Use when PCA has been run on a pretreated SCMS metabolomics feature table
  (e.g., single-cell peak matrices after TIC normalization and peak alignment) and
  you must assess cluster separation between phenotypes from the score plot.
when_to_use_negative:
- PCA score plots already show clear separation only for phenotypically distinct pairs
  — do not extrapolate discrimination to phenotypically similar pairs like IGR39/IGR37,
  where PCA demonstrably fails
- A quantitative classification with accuracy metrics is required — PCA is unsupervised
  and cannot substitute for a supervised classifier such as MetaPhenotype
- Data are raw single-cell spectra — pretreatment (noise/background subtraction, TIC
  normalization, peak alignment) must be completed first
edam_operation: http://edamontology.org/operation_2939
tools:
- name: MetaboAnalyst 6.0
  role: Web tool used to perform PCA and generate the score plots of the pretreated
    SCMS feature matrices
- name: Geena2
  role: Online bioinformatics tool for peak alignment (10 ppm mass tolerance) upstream
    of PCA, ensuring a consistent feature table
- name: Thermo Xcalibur Qual Browser 3.0
  role: Used to export peak lists from LTQ Orbitrap XL SCMS runs that form the input
    feature matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:2f2a696da9d3899afcc23c893b71634347c14da1e4028e09ff8c5085e270b82f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - MetaboAnalyst 6.0
  - Thermo Xcalibur Qual Browser 3.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# score-plot-interpretation-of-pca-results

## Summary

Interprets PCA score plots of pretreated single-cell mass spectrometry (SCMS) metabolomics feature matrices to judge whether primary and metastatic melanoma cell populations separate or overlap, using MetaboAnalyst 6.0 as in the MetaPhenotype study.

## When to use

Use when PCA has been run on a pretreated SCMS metabolomics feature table (e.g., single-cell peak matrices after TIC normalization and peak alignment) and you must assess cluster separation between phenotypes from the score plot.

## When NOT to use

- PCA score plots already show clear separation only for phenotypically distinct pairs — do not extrapolate discrimination to phenotypically similar pairs like IGR39/IGR37, where PCA demonstrably fails
- A quantitative classification with accuracy metrics is required — PCA is unsupervised and cannot substitute for a supervised classifier such as MetaPhenotype
- Data are raw single-cell spectra — pretreatment (noise/background subtraction, TIC normalization, peak alignment) must be completed first

## Inputs

- Pretreated SCMS metabolomics feature matrix (cells × metabolomic species) from Single-probe SCMS on Thermo LTQ Orbitrap XL, after noise subtraction, background subtraction, TIC normalization, and Geena2 peak alignment (10 ppm)
- Cell phenotype labels (primary vs metastatic) for the melanoma cell pairs, untreated and vemurafenib-treated

## Outputs

- PCA score plots per cell pair and treatment state from MetaboAnalyst 6.0
- Qualitative separation/overlap assessment of primary vs metastatic populations per pair
- Comparative statement of PCA's discrimination ability versus HCA accuracies (e.g., 72%, 7%, 14%)

## How to apply

Perform PCA on the pretreated SCMS metabolomics feature matrices (peaks exported via Xcalibur Qual Browser 3.0, aligned with Geena2 at 10 ppm mass tolerance, noise/background subtracted, TIC-normalized) using MetaboAnalyst 6.0. Generate PCA score plots for each primary/metastatic cell pair — IGR39/IGR37 and WM793/WM1205Lu (and WM115/WM266-4 in the benchmark task), both untreated and vemurafenib-treated. Visually assess whether primary and metastatic cell populations form distinct clusters in the score plot (as reported for the WM pairs) or show heavily overlapping metabolomic profiles (as reported for IGR39/IGR37). Treat overlap in the score plot as evidence that PCA cannot discriminate phenotypically similar cells, and report this as a negative discrimination result rather than forcing a classification accuracy; compare against HCA clustering accuracy values (72% for treated IGR39/IGR37, 7% untreated and 14% treated for WM pairs) to contextualize whether unsupervised methods are adequate.

## Related tools

- **MetaboAnalyst 6.0** (Web tool used to perform PCA and generate the score plots of the pretreated SCMS feature matrices)
- **Geena2** (Online bioinformatics tool for peak alignment (10 ppm mass tolerance) upstream of PCA, ensuring a consistent feature table)
- **Thermo Xcalibur Qual Browser 3.0** (Used to export peak lists from LTQ Orbitrap XL SCMS runs that form the input feature matrix)

## Examples

```
python base.py  # after PCA of the pretreated dataset.csv in MetaboAnalyst 6.0, inspect score plots per cell pair to judge primary/metastatic separation vs overlap
```

## Evaluation signals

- PCA score plot for WM pairs visually separates primary and metastatic cell populations, matching the reported behavior
- PCA score plot for IGR39/IGR37 shows heavily overlapping metabolomic profiles, consistent with the paper's negative finding
- Input feature table is fully pretreated: noise/background subtracted, TIC-normalized, Geena2-aligned at 10 ppm, missing values removed
- PCA-derived discrimination conclusions are consistent with (or explain gaps in) HCA accuracy values such as 72% (treated IGR39/IGR37), 7% and 14% (WM pairs)

## Limitations

- PCA cannot effectively discriminate phenotypically similar cells, e.g., IGR39/IGR37, whose metabolomic profiles overlap heavily in score plots
- Neither PCA nor HCA reliably discriminates primary from metastatic phenotypes across all cell pairs and treatment states — HCA was accurate only for drug-treated IGR39/IGR37 cells (72%) and very poor on WM pairs (7% untreated, 14% treated)
- Assessment of separation/overlap in score plots is qualitative/visual, and reproducibility depends on the same pretreatment protocol and MetaboAnalyst 6.0 settings

## Evidence
