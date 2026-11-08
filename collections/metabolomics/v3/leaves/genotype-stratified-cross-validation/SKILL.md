---
name: genotype-stratified-cross-validation
description: Use when you have a multi-tissue untargeted metabolomics feature table
  with metadata for two distinct genetic disease models (e.g. 3xTg and 5xFAD) and
  need to compare how well each model's genotype classes separate using supervised
  PLS-DA classification.
when_to_use_negative:
- When the dataset contains only one genotype contrast or a single genetic model —
  the CER comparison between 3xTg and 5xFAD is the core deliverable
- When the feature table has not been RCLR-transformed per tissue — applying PLS-DA
  on raw peak areas introduces compositional bias
- When the goal is unsupervised exploration of genotype-dysregulated molecular classes
  — use PCA-based quality control and cross-tissue molecular networking instead of
  PLS-DA classification
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: GNPS/MassIVE
  role: Public repository from which the untargeted metabolomics feature tables and
    metadata for the 3xTg and 5xFAD studies are downloaded before PLS-DA modelling
- name: Qiita
  role: Platform used to access the deposited metabolomics and paired metagenomics
    data for stratifying samples by study, cohort, tissue and genotype/colonization
    group
- name: scikit-learn
  role: Implements the PLS-DA classification models fitted per tissue and per genotype
    contrast, including VIP-based feature selection and cross-validated CER evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:57f9ed85cef220fe5b370f47354f29719ef02eced462de88ace9b206693a990e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS/MassIVE
  - Qiita
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genotype-stratified-cross-validation

## Summary

Fit and cross-validate PLS-DA models separately for each genotype contrast in a multi-tissue untargeted metabolomics dataset, comparing classification error rates (CER) between 3xTg and 5xFAD mouse models to identify which model and tissue yield the most discriminative genotype signatures.

## When to use

Use when you have a multi-tissue untargeted metabolomics feature table with metadata for two distinct genetic disease models (e.g. 3xTg and 5xFAD) and need to compare how well each model's genotype classes separate using supervised PLS-DA classification.

## When NOT to use

- When the dataset contains only one genotype contrast or a single genetic model — the CER comparison between 3xTg and 5xFAD is the core deliverable
- When the feature table has not been RCLR-transformed per tissue — applying PLS-DA on raw peak areas introduces compositional bias
- When the goal is unsupervised exploration of genotype-dysregulated molecular classes — use PCA-based quality control and cross-tissue molecular networking instead of PLS-DA classification

## Inputs

- Publicly deposited untargeted metabolomics feature tables and sample metadata from GNPS/MassIVE for 3xTg and 5xFAD mouse studies
- Sample metadata mapping each sample to study (3xTg or 5xFAD), cohort (Sacrifice or Longitudinal), tissue (brain, serum, liver, cecal content, feces), and genotype/colonization group (e.g. 3xTg_SPF, WT_SPF)
- Extracted LC-MS/MS peak areas per sample per molecular feature

## Outputs

- Cross-validated PLS-DA classification error rate (CER) table stratified by study, tissue, and genotype/colonization contrast
- Per-tissue PLS-DA models fitted on VIP > 1 features for each genotype contrast
- Reproduced Figure 2A-style bar chart comparing CER between 3xTg and 5xFAD models across tissues

## How to apply

Download the publicly deposited untargeted metabolomics feature tables and metadata from GNPS/MassIVE for both studies and stratify samples by study (3xTg vs 5xFAD), cohort (Sacrifice vs Longitudinal), tissue, and genotype/colonization group. Preprocess extracted peak areas per tissue using robust center log ratio (RCLR) transformation as applied in the paper's PCA-based quality control. For each tissue and each genotype/colonization contrast (e.g. 3xTg_SPF vs WT_SPF, 5xFAD_SPF vs WT_SPF), fit a PLS-DA classification model in scikit-learn, restricting features to those with variable importance in projection (VIP) > 1 as in Tables S1–S12. Evaluate each model via cross-validated classification error rate (CER) for both 3xTg and 5xFAD contrasts across all five tissue types (brain, serum, liver, cecal content, feces). Assemble a CER results table per study, tissue, and stratification, and compare error rates to confirm that PLS-DA models achieve lower CER in 3xTg animals than in 5xFAD mice, with cecal and brain profiles giving the best genotype classification in both studies.

## Related tools

- **GNPS/MassIVE** (Public repository from which the untargeted metabolomics feature tables and metadata for the 3xTg and 5xFAD studies are downloaded before PLS-DA modelling)
- **Qiita** (Platform used to access the deposited metabolomics and paired metagenomics data for stratifying samples by study, cohort, tissue and genotype/colonization group)
- **scikit-learn** (Implements the PLS-DA classification models fitted per tissue and per genotype contrast, including VIP-based feature selection and cross-validated CER evaluation)

## Examples

```
from sklearn.cross_decomposition import PLSRegression; from sklearn.model_selection import cross_val_score; pls = PLSRegression(n_components=2); cer = 1 - cross_val_score(pls, X_rclr[vip_features], y_genotype, cv=5).mean()
```

## Evaluation signals

- Generated error-rate table and bar chart must show that 3xTg PLS-DA models achieve lower classification error rates than 5xFAD models across tissues, matching the reported Figure 2A pattern
- Cecal and brain tissue PLS-DA models must yield the lowest CER values for genotype classification in both the 3xTg and 5xFAD studies
- Feature space used for each PLS-DA model must be restricted to VIP > 1 features, consistent with Tables S1–S12
- CER values must be computed by cross-validation, not on the training set, so that reported error rates reflect out-of-sample classification performance
- Number of stratified models fitted should equal the number of unique combinations of study × tissue × genotype/colonization contrast present in the metadata

## Limitations

- Germ-free animals received an antibiotic cocktail during behavioral testing, which may have altered organ metabolic profiles and thus confound genotype classification
- Differences between Longitudinal and Sacrifice cohorts exist for both microbiome and metabolomics data, attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so stratification by cohort is required
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound for genotype discrimination
- Hundreds of differentially abundant molecules remain non-annotated, so the discriminatory features underlying PLS-DA separation may not all be chemically interpretable
- Whether conjugation of microbial phenylalanine-derived precursors into carnitines is microbe- or host-mediated remains unclear, complicating biological interpretation of top PLS-DA features

## Evidence

- [results] PLS-DA models classified 3xTg animals better than 5xFAD mice, showing lower classification error rate (CER): [evidence span withheld — anchor status: not_found]
- [intro] Cross-tissue molecular dysregulation via machine learning models: "Via machine learning models24,25 and molecular networking26, we identify cross-tissue molecular dysregulation of carnitines, bile acids, B vitamins, N-acyl lipids, indoles, polyamines, and many other"
- [discussion] Cohort differences attributed to sample collection differences: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
- [discussion] Breeding/housing confound between 3xTg and WT mice: "Finally, 3xTg and WT mice were bred and housed separately before the start of the experiment"
