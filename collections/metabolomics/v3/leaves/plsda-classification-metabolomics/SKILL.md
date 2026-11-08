---
name: plsda-classification-metabolomics
description: Use when a labeled metabolomics feature table (peak areas per sample)
  must be classified into genotype/colonization groups (e.g., 3xTg_SPF vs WT_SPF)
  and discriminatory features (VIP > 1) plus cross-validated classification error
  rates are the desired outputs.
when_to_use_negative:
- The goal is molecular annotation/identification of MS/MS spectra (use GNPS spectral
  library matching or tissueMASST instead of a supervised classifier)
- Fecal Longitudinal-cohort samples lacking animal IDs are the unit of analysis —
  single-animal developmental classification is not possible per the paper's own caveat
- The input is already a small, curated VIP-selected feature list rather than a full
  peak-area feature table, so refitting PLS-DA and recomputing VIP would be circular
edam_operation: http://edamontology.org/operation_2990
tools:
- name: scikit-learn
  role: Fit the PLS-DA classification models and compute cross-validated classification
    error rate (CER) per tissue and genotype/colonization contrast
- name: GNPS/MassIVE
  role: Public repository from which the untargeted metabolomics feature tables and
    metadata are downloaded and stratified
- name: Qiita
  role: Data access point for the deposited study artifacts used to build the stratified
    feature tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:5d31fe65e174362809b38cd56d657a31e5af903ec273346c7848b60c64194fe6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS/MassIVE
  - Qiita
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# plsda-classification-metabolomics

## Summary

Supervised PLS-DA classification of untargeted metabolomics feature tables to extract genotype-affected molecular features and compare classification error rates (CER) across contrasts, tissues, and studies (e.g., 3xTg vs 5xFAD AD mouse models under SPF/germ-free colonization).

## When to use

Use when a labeled metabolomics feature table (peak areas per sample) must be classified into genotype/colonization groups (e.g., 3xTg_SPF vs WT_SPF) and discriminatory features (VIP > 1) plus cross-validated classification error rates are the desired outputs.

## When NOT to use

- The goal is molecular annotation/identification of MS/MS spectra (use GNPS spectral library matching or tissueMASST instead of a supervised classifier)
- Fecal Longitudinal-cohort samples lacking animal IDs are the unit of analysis — single-animal developmental classification is not possible per the paper's own caveat
- The input is already a small, curated VIP-selected feature list rather than a full peak-area feature table, so refitting PLS-DA and recomputing VIP would be circular

## Inputs

- Untargeted metabolomics feature table (peak areas per molecular feature per sample) from GNPS/MassIVE for the 3xTg and 5xFAD studies
- Sample metadata mapping each sample to study (3xTg/5xFAD), cohort (Sacrifice/Longitudinal), tissue (brain, serum, liver, cecal content, feces), genotype, and colonization status (SPF/germ-free)

## Outputs

- Per-model PLS-DA classifier for each tissue x genotype/colonization contrast
- List of VIP > 1 discriminatory features per contrast (Tables S1-S12 style)
- Cross-validated classification error rate (CER) table per study, tissue, and stratification
- Figure 2A-style bar chart comparing CER across studies and tissues

## How to apply

Download the deposited untargeted metabolomics feature tables and metadata from GNPS/MassIVE (or Qiita) and stratify samples by study (3xTg vs 5xFAD), cohort (Sacrifice vs Longitudinal), tissue (brain, serum, liver, cecal content, feces), and genotype/colonization group. Preprocess extracted peak areas per tissue with robust center log ratio (RCLR) transformation, matching the paper's PCA-based quality control. For each tissue and genotype/colonization contrast, fit a PLS-DA classifier in scikit-learn and extract variable importance (VIP > 1) features as reported in Tables S1-S12. Evaluate each model by cross-validated classification error rate (CER) and assemble a results table per study, tissue, and stratification, reproducing the Figure 2A-style bar chart. Rationale: PLS-DA is a supervised method suited to multi-class, high-dimensional, low-n metabolomics where the goal is both discrimination and feature extraction, unlike the unsupervised PCA used for QC.

## Related tools

- **scikit-learn** (Fit the PLS-DA classification models and compute cross-validated classification error rate (CER) per tissue and genotype/colonization contrast)
- **GNPS/MassIVE** (Public repository from which the untargeted metabolomics feature tables and metadata are downloaded and stratified)
- **Qiita** (Data access point for the deposited study artifacts used to build the stratified feature tables)

## Examples

```
from sklearn.cross_decomposition import PLSRegression; model = PLSRegression(n_components=2).fit(X_rclr, y_genotype)
```

## Evaluation signals

- 3xTg PLS-DA models must achieve lower classification error rates (CER) than 5xFAD models across tissues
- Cecum and brain profiles must yield the best (lowest-CER) genotype classification in both the 3xTg and 5xFAD studies
- Every reported feature list must be restricted to VIP > 1 features, matching Tables S1-S12
- Preprocessing must be RCLR-transformed peak areas, consistent with the paper's PCA-based QC pipeline
- Final results table must cover all five tissue types for both genotype contrasts within each study and reproduce the Figure 2A pattern

## Limitations

- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound that can inflate apparent genotype separability
- Differences between Longitudinal and Sacrifice cohorts in microbiome and metabolomics data (fresh fecal vs. colon after sacrifice) mean CERs should not be pooled across cohorts
- Germ-free animals received an antibiotic cocktail during behavioral testing, which could alter organ metabolic profiles and shift discriminatory features

## Evidence

- [other] PLS-DA finding (task_002): [evidence span withheld — anchor status: not_found]
- [other] Feature extraction via VIP (task_002): [evidence span withheld — anchor status: not_found]
- [intro] Machine learning identifies cross-tissue dysregulation: "Via machine learning models24,25 and molecular networking26, we identify cross-tissue molecular dysregulation of carnitines, bile acids, B vitamins, N-acyl lipids, indoles, polyamines, and many other"
- [intro] Discriminatory serum carnitines: "we find microbially-modulated carnitines as major discriminatory features in the serum of 3xTg animals, including benzoyl-, phenylacetyl- and phenylpropionyl-carnitine"
