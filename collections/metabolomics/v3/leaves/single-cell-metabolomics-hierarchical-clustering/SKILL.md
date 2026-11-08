---
name: single-cell-metabolomics-hierarchical-clustering
description: Use when you have pretreated SCMS single-cell metabolomics feature matrices
  (e.g., primary/metastatic melanoma cell pairs, untreated or vemurafenib-treated)
  and want a label-free baseline check of phenotype separability before training a
  classifier.
when_to_use_negative:
- The goal is accurate phenotype prediction rather than exploratory separability assessment
  — the article shows PCA cannot effectively discriminate cells with similar metabolomic
  profiles such as IGR39/IGR37, and HCA accuracy was as low as 7% on WM pairs, so
  a supervised model like MetaPhenotype is needed
- The input is raw single-probe SCMS spectra — data must first undergo the established
  pretreatment protocol (noise/background subtraction, TIC normalization, peak alignment,
  missing-value removal) before MetaboAnalyst 6.0 clustering
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalyst 6.0
  role: Online bioinformatics platform used to perform HCA and PCA on the pretreated
    single-cell metabolomics feature matrices and generate score plots
- name: Geena2
  role: Online tool for peak alignment (10 ppm mass tolerance) during SCMS data pretreatment,
    producing the feature matrices fed into HCA/PCA
- name: Thermo Xcalibur Qual Browser 3.0
  role: Used for peak export from Thermo LTQ Orbitrap XL raw data prior to pretreatment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:12ac8daefd28079aa3a4a30c9d2a42fbef174c0432f90d5830755c38e2754c6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - MetaboAnalyst 6.0
  - Thermo Xcalibur Qual Browser 3.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9210526315789473
n_claims: 38
---

# single-cell-metabolomics-hierarchical-clustering

## Summary

Unsupervised benchmarking of single-cell mass spectrometry (SCMS) metabolomics data via hierarchical clustering analysis (HCA) and principal component analysis (PCA) in MetaboAnalyst 6.0, to test whether untreated feature matrices can discriminate primary from metastatic melanoma phenotypes before resorting to supervised learning.

## When to use

Use when you have pretreated SCMS single-cell metabolomics feature matrices (e.g., primary/metastatic melanoma cell pairs, untreated or vemurafenib-treated) and want a label-free baseline check of phenotype separability before training a classifier.

## When NOT to use

- The goal is accurate phenotype prediction rather than exploratory separability assessment — the article shows PCA cannot effectively discriminate cells with similar metabolomic profiles such as IGR39/IGR37, and HCA accuracy was as low as 7% on WM pairs, so a supervised model like MetaPhenotype is needed
- The input is raw single-probe SCMS spectra — data must first undergo the established pretreatment protocol (noise/background subtraction, TIC normalization, peak alignment, missing-value removal) before MetaboAnalyst 6.0 clustering

## Inputs

- Pretreated SCMS single-cell metabolomics feature matrix (dataset.csv-style feature table, cells x m/z species) for each primary/metastatic melanoma cell pair (IGR39/IGR37, WM793/WM1205Lu; untreated and vemurafenib-treated)
- Known phenotype labels (primary vs metastatic) per cell for accuracy scoring

## Outputs

- HCA dendrograms and per-pair cluster assignments of single cells to primary/metastatic clusters with classification accuracies (e.g., 72% treated IGR39/IGR37; 7% untreated and 14% treated WM pairs)
- PCA score plots per cell pair showing separation (WM pairs) or overlapping metabolomic profiles (IGR39/IGR37)

## How to apply

Load the pretreated SCMS feature matrices (after noise/background subtraction, TIC normalization, peak alignment at 10 ppm mass tolerance via Geena2, and missing-value removal) for each primary/metastatic cell pair. Run hierarchical clustering analysis in MetaboAnalyst 6.0, assigning cells to primary vs metastatic clusters per pair and treatment state, then compute classification accuracy against known cell labels. Run PCA on the same pretreated data in MetaboAnalyst 6.0 and inspect score plots for primary/metastatic separation per pair. Use the results as a decision point: in this study HCA reached only 72% for treated IGR39/IGR37 cells and 7–14% for WM pairs, and PCA score plots overlapped heavily for IGR39/IGR37, motivating a supervised meta-learning model (MetaPhenotype) instead.

## Related tools

- **MetaboAnalyst 6.0** (Online bioinformatics platform used to perform HCA and PCA on the pretreated single-cell metabolomics feature matrices and generate score plots)
- **Geena2** (Online tool for peak alignment (10 ppm mass tolerance) during SCMS data pretreatment, producing the feature matrices fed into HCA/PCA)
- **Thermo Xcalibur Qual Browser 3.0** (Used for peak export from Thermo LTQ Orbitrap XL raw data prior to pretreatment)

## Examples

```
python base.py
```

## Evaluation signals

- Reproduced HCA accuracies match reported values within reporting tolerance: 72% (treated IGR39/IGR37), 7% (untreated WM pair), 14% (treated WM pair)
- PCA score plots for WM cell pairs show visual separation of primary and metastatic populations
- PCA score plots for the IGR39/IGR37 pair show heavily overlapping primary/metastatic metabolomic profiles
- Feature matrix was fully pretreated before clustering (noise/background subtraction, TIC normalization, 10 ppm peak alignment, missing-value removal)

## Limitations

- Neither HCA nor PCA reliably discriminates phenotypes across all cell pairs and treatment states — HCA failed on WM115/WM266-4 cells (7% untreated, 14% treated) and PCA overlapped for IGR39/IGR37
- PCA cannot effectively discriminate phenotypically similar cells with high similarity in metabolomic profiles
- Unsupervised clustering of large SCMS data is ill-suited relative to machine learning approaches when prediction, not just exploration, is the goal

## Evidence

- [readme] README run instructions: [evidence span withheld — anchor status: external_doc]
