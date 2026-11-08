---
name: vip-feature-selection
description: Use when a supervised PLS-DA genotype/colonization classifier (e.g. 3xTg_SPF
  vs WT_SPF or 5xFAD vs WT per tissue) has been fit on metabolomics feature tables
  and discriminatory molecular features must be extracted via VIP scores.
when_to_use_negative:
- The goal is MS/MS structural annotation rather than supervised feature discrimination
  — use GNPS library spectral matching (Level 2/3 MSI annotations) instead.
- No class labels exist (e.g. exploratory QC or cohort-structure checks) — the paper
  uses unsupervised PCA on RCLR-transformed data for that purpose.
- Input is raw MS/MS spectra rather than a preprocessed peak-area feature table; feature
  extraction must occur before PLS-DA/VIP fitting.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Fits the PLS-DA classification models per tissue and genotype/colonization
    contrast from which VIP > 1 features are extracted and cross-validated CER is
    computed
- name: GNPS/MassIVE
  role: Source of the publicly deposited untargeted metabolomics feature tables and
    metadata stratified for PLS-DA/VIP analysis
- name: Qiita
  role: Repository access point for the study's metabolomics data used to assemble
    the PLS-DA training matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:6f63a71e22c872bc8012b7b17b06b1484009a6e9ddc10bad9a3a86d994505099
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS/MassIVE
  - Qiita
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# vip-feature-selection

## Summary

Extracts genotype-affected molecular features from PLS-DA classification models by ranking variables on Variable Importance in Projection (VIP) scores and retaining those with VIP > 1, as done for the 3xTg and 5xFAD multi-organ mouse metabolomics datasets (Tables S1–S12).

## When to use

Use when a supervised PLS-DA genotype/colonization classifier (e.g. 3xTg_SPF vs WT_SPF or 5xFAD vs WT per tissue) has been fit on metabolomics feature tables and discriminatory molecular features must be extracted via VIP scores.

## When NOT to use

- The goal is MS/MS structural annotation rather than supervised feature discrimination — use GNPS library spectral matching (Level 2/3 MSI annotations) instead.
- No class labels exist (e.g. exploratory QC or cohort-structure checks) — the paper uses unsupervised PCA on RCLR-transformed data for that purpose.
- Input is raw MS/MS spectra rather than a preprocessed peak-area feature table; feature extraction must occur before PLS-DA/VIP fitting.

## Inputs

- Untargeted metabolomics feature table of extracted peak areas per tissue, stratified by study (3xTg vs 5xFAD), cohort (Sacrifice vs Longitudinal), tissue, and genotype/colonization group
- Sample metadata mapping samples to genotype and colonization labels (e.g. 3xTg_SPF vs WT_SPF)
- RCLR-transformed feature matrix produced by the paper's PCA-based QC preprocessing

## Outputs

- VIP-score table ranking molecular features per PLS-DA model (tissue × genotype/colonization contrast)
- List of discriminatory features with VIP > 1 per contrast, corresponding to Tables S1–S12
- Cross-validated classification error rate (CER) per model for the Figure 2A-style results table/bar chart

## How to apply

Start from extracted peak-area feature tables per tissue (brain, serum, liver, cecal content, feces) preprocessed with robust center log ratio (RCLR) transformation as in the paper's PCA-based QC. For each tissue and genotype/colonization contrast, fit a PLS-DA classification model in scikit-learn and compute VIP scores from the latent-variable loadings. Retain features with VIP > 1 as discriminatory, mirroring the paper's Tables S1–S12. Evaluate each model by cross-validated classification error rate (CER) — the expected pattern is lower CER for 3xTg contrasts than 5xFAD contrasts, with cecum and brain giving the best genotype classification in both studies.

## Related tools

- **scikit-learn** (Fits the PLS-DA classification models per tissue and genotype/colonization contrast from which VIP > 1 features are extracted and cross-validated CER is computed)
- **GNPS/MassIVE** (Source of the publicly deposited untargeted metabolomics feature tables and metadata stratified for PLS-DA/VIP analysis)
- **Qiita** (Repository access point for the study's metabolomics data used to assemble the PLS-DA training matrices)

## Examples

```
from sklearn.cross_decomposition import PLSRegression; pls = PLSRegression(n_components=2).fit(X_rclr, y_genotype)
```

## Evaluation signals

- Every retained discriminatory feature has VIP ≥ 1 in the fitted PLS-DA model; the VIP > 1 cutoff matches Tables S1–S12.
- The assembled CER table shows 3xTg models with lower classification error than 5xFAD models across tissues, reproducing the Figure 2A pattern.
- Cecum and brain models yield the lowest CER among the five tissue types in both studies.
- Models are fit on RCLR-transformed peak areas (not raw intensities), consistent with the paper's PCA-based QC preprocessing.
- CER is computed via cross-validation, not resubstitution on the training set.

## Limitations

- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound on genotype-separating features.
- Germ-free animals received an antibiotic cocktail during behavioral testing, which may have altered organ metabolic profiles and thus the discriminatory feature set.
- Differences between Longitudinal and Sacrifice cohorts (fresh fecal vs. colon after sacrifice) mean VIP features should be compared within cohort; longitudinal fecal samples lack animal IDs, preventing single-animal models.
- Hundreds of differentially abundant molecules remain non-annotated, so VIP-selected features may lack Level 1 MSI identifications.

## Evidence

- [results] PLS-DA models classified 3xTg animals better than 5xFAD mice, showing lower classification error rate (CER), with cecal and brain profiles giving the best genotype classification in both studies.: [evidence span withheld — anchor status: not_found]
- [methods] fit a PLS-DA classification model in scikit-learn, using variable importance (VIP > 1) features as in Tables S1-S12: [evidence span withheld — anchor status: not_found]
- [methods] Preprocess extracted peak areas per tissue using robust center log ratio (RCLR) transformation as applied in the paper's PCA-based quality control.: [evidence span withheld — anchor status: not_found]
- [intro] Via machine learning models24,25 and molecular networking26, we identify cross-tissue molecular dysregulation of carnitines, bile acids, B vitamins, indoles, polyamines: "Via machine learning models24,25 and molecular networking26, we identify cross-tissue molecular dysregulation of carnitines, bile acids, B vitamins, indoles, polyamines"
