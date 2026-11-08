---
name: principal-component-analysis-of-scms-feature-matrices
description: Use when you have a pretreated SCMS metabolomics feature table (single
  cells x metabolic species, noise/background-subtracted, TIC-normalized, peak-aligned)
  and want an unsupervised first check of phenotype separability before supervised
  classification.
when_to_use_negative:
- Input is raw SCMS spectra that have not gone through the pretreatment protocol (noise
  subtraction, background subtraction, TIC normalization, 10 ppm Geena2 peak alignment,
  missing-value removal)
- The goal is quantitative phenotype classification accuracy — PCA is unsupervised
  and cannot effectively discriminate phenotypically similar cells such as IGR39/IGR37,
  where the paper shows heavily overlapping profiles
- Cells are too few or features too sparse to form a stable covariance structure in
  the feature matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst 6.0
  role: Web platform used to run PCA and produce score plots from the pretreated SCMS
    feature matrices
- name: Geena2
  role: Online bioinformatics tool for peak alignment (mass tolerance of 10 ppm) during
    upstream data pretreatment feeding the PCA matrix
- name: Thermo Xcalibur Qual Browser 3.0
  role: Used to export peaks from the Thermo LTQ Orbitrap XL SCMS raw data prior to
    matrix construction
- name: MetaPhenotype
  role: Supervised meta-learning alternative adopted because PCA cannot discriminate
    cells with similar metabolomic profiles (e.g., IGR39/IGR37)
  repo: github.com/songyuan93/MetaPhenotype
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:41ef250def2695c1ffefc31a04bf684f89b8e509a7869dd966834469f4527c56
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# principal-component-analysis-of-scms-feature-matrices

## Summary

Unsupervised PCA on pretreated single-cell mass spectrometry (SCMS) metabolomics feature matrices to visually assess whether primary and metastatic melanoma cell populations separate in a 2D score plot. Used in the MetaPhenotype paper as a baseline unsupervised benchmark that motivates machine-learning phenotype classification.

## When to use

Use when you have a pretreated SCMS metabolomics feature table (single cells x metabolic species, noise/background-subtracted, TIC-normalized, peak-aligned) and want an unsupervised first check of phenotype separability before supervised classification.

## When NOT to use

- Input is raw SCMS spectra that have not gone through the pretreatment protocol (noise subtraction, background subtraction, TIC normalization, 10 ppm Geena2 peak alignment, missing-value removal)
- The goal is quantitative phenotype classification accuracy — PCA is unsupervised and cannot effectively discriminate phenotypically similar cells such as IGR39/IGR37, where the paper shows heavily overlapping profiles
- Cells are too few or features too sparse to form a stable covariance structure in the feature matrix

## Inputs

- Pretreated SCMS metabolomics feature matrix (single cells x metabolic species) from Single-probe SCMS on Thermo LTQ Orbitrap XL data
- Cell phenotype labels (primary vs metastatic) per cell, per cell pair and treatment state (untreated / vemurafenib-treated)
- METLIN- and HMDB-tentatively-labeled peak table (dataset.csv format in the MetaPhenotype repo)

## Outputs

- PCA score plots per primary/metastatic cell pair and treatment state (PNG/image via MetaboAnalyst 6.0)
- Qualitative separability assessment: separated clusters (WM pairs) vs overlapping metabolomic profiles (IGR39/IGR37)
- Component-level data used for downstream comparison against supervised model accuracy (AUROC, F1, accuracy)

## How to apply

Start from the pretreated SCMS feature matrices (peaks exported in Thermo Xcalibur Qual Browser 3.0, noise and background subtracted, TIC-normalized, peak-aligned with Geena2 at 10 ppm mass tolerance, missing values removed). Upload the feature table for each primary/metastatic cell pair (e.g., IGR39/IGR37 and WM793/WM1205Lu, untreated and vemurafenib-treated) to MetaboAnalyst 6.0 and run principal component analysis. Generate PCA score plots per pair and inspect whether primary and metastatic cell populations form separated clusters (e.g., the WM pairs) or heavily overlapping clouds (e.g., IGR39/IGR37). Treat overlap as evidence that PCA — and by extension unsupervised clustering — cannot discriminate phenotypically similar cells, which is the rationale for the meta-learning MetaPhenotype model in the paper.

## Related tools

- **MetaboAnalyst 6.0** (Web platform used to run PCA and produce score plots from the pretreated SCMS feature matrices)
- **Geena2** (Online bioinformatics tool for peak alignment (mass tolerance of 10 ppm) during upstream data pretreatment feeding the PCA matrix)
- **Thermo Xcalibur Qual Browser 3.0** (Used to export peaks from the Thermo LTQ Orbitrap XL SCMS raw data prior to matrix construction)
- **MetaPhenotype** (Supervised meta-learning alternative adopted because PCA cannot discriminate cells with similar metabolomic profiles (e.g., IGR39/IGR37)) — github.com/songyuan93/MetaPhenotype

## Evaluation signals

- PCA score plots for the WM pairs show visually separated primary vs metastatic cell populations
- PCA score plots for the IGR39/IGR37 pair show heavily overlapping metabolomic profiles, matching the reported overlap behavior
- Input matrix passed the full pretreatment chain: noise subtraction, background subtraction, TIC normalization, Geena2 10 ppm peak alignment, missing-value removal
- HCA benchmark on the same data reproduces reported accuracies (72% treated IGR39/IGR37; 7% and 14% WM pairs), confirming the feature matrix is comparable to the paper's

## Limitations

- PCA cannot effectively discriminate phenotypical cells with relatively high similarities in metabolomic profiles such as IGR39/IGR37 cells
- Neither PCA nor HCA reliably discriminates phenotypes across all four cell pairs and treatment states, so PCA should be a benchmark, not the final classifier
- Results depend on the established SCMS pretreatment protocol; skipping background subtraction or TIC normalization distorts the score plots

## Evidence
