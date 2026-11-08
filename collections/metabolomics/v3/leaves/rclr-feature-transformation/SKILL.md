---
name: rclr-feature-transformation
description: Use when a sparse, zero-inflated metabolomics feature table (e.g. peak
  areas from GNPS/MassIVE deposits) must be made compositional-ready for PCA-based
  QC or PLS-DA classification across tissues, cohorts, and genotype/colonization groups.
when_to_use_negative:
- Input is raw MS/MS spectra or mzML files rather than an extracted peak-area feature
  table — feature quantification must happen first
- The goal is absolute-quantification or concentration comparison, since RCLR produces
  scale-invariant compositional values unsuitable for absolute abundance claims
- Analysis targets single-animal longitudinal trajectories from the fecal Longitudinal
  cohort, which lacks animal IDs
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: GNPS/MassIVE
  role: Source of the publicly deposited untargeted metabolomics feature tables and
    metadata that serve as RCLR input
- name: Qiita
  role: Repository providing study metadata used to stratify samples by study, cohort,
    tissue, and genotype/colonization group
- name: scikit-learn
  role: Downstream PLS-DA classification on the RCLR-transformed features, with VIP-based
    feature selection and cross-validated CER evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:40e5f2ec56fb746c7b7bf868c6619cf27393a49f10739fd08b38a5f9b127167d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS/MassIVE
  - Qiita
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rclr-feature-transformation

## Summary

Applies robust center log ratio (RCLR) transformation to untargeted metabolomics peak-area feature tables (brain, serum, liver, cecal content, feces) before ordination and machine-learning analyses such as the PLS-DA genotype classification used in the 3xTg and 5xFAD Alzheimer's disease mouse studies.

## When to use

Use when a sparse, zero-inflated metabolomics feature table (e.g. peak areas from GNPS/MassIVE deposits) must be made compositional-ready for PCA-based QC or PLS-DA classification across tissues, cohorts, and genotype/colonization groups.

## When NOT to use

- Input is raw MS/MS spectra or mzML files rather than an extracted peak-area feature table — feature quantification must happen first
- The goal is absolute-quantification or concentration comparison, since RCLR produces scale-invariant compositional values unsuitable for absolute abundance claims
- Analysis targets single-animal longitudinal trajectories from the fecal Longitudinal cohort, which lacks animal IDs

## Inputs

- Untargeted metabolomics feature table (sample x molecular feature peak areas) downloaded from GNPS/MassIVE
- Sample metadata sheet with study (3xTg vs 5xFAD), cohort (Sacrifice vs Longitudinal), tissue, genotype, and colonization (SPF vs germ-free) labels
- Tissue-stratified peak-area matrices (brain, serum, liver, cecal content, feces)

## Outputs

- RCLR-transformed feature matrix per tissue
- PCA quality-control plots built on the RCLR-transformed data
- PLS-DA model outputs with VIP > 1 feature lists and cross-validated CER values per study/tissue/contrast
- Figure 2A-style CER bar chart comparing 3xTg vs 5xFAD classification error rates

## How to apply

Download the publicly deposited untargeted metabolomics feature tables and metadata from GNPS/MassIVE for the 3xTg and 5xFAD studies, stratifying by study, cohort (Sacrifice vs Longitudinal), tissue, and genotype/colonization group (e.g. 3xTg_SPF vs WT_SPF). Extract peak areas per tissue and apply the robust center log ratio (RCLR) transformation, as the paper used it in its PCA-based quality control, because RCLR tolerates the compositional zeros typical of untargeted LC-MS/MS feature tables. Feed the RCLR-transformed matrix into downstream models, e.g. a PLS-DA classifier in scikit-learn, restricting to variable-importance (VIP > 1) features as reported in Tables S1-S12. Evaluate via cross-validated classification error rate (CER) per tissue and contrast, then assemble the CER table and Figure 2A-style bar chart comparing 3xTg vs 5xFAD error rates.

## Related tools

- **GNPS/MassIVE** (Source of the publicly deposited untargeted metabolomics feature tables and metadata that serve as RCLR input)
- **Qiita** (Repository providing study metadata used to stratify samples by study, cohort, tissue, and genotype/colonization group)
- **scikit-learn** (Downstream PLS-DA classification on the RCLR-transformed features, with VIP-based feature selection and cross-validated CER evaluation)

## Examples

```
from sklearn.cross_decomposition import PLSRegression; model = PLSRegression().fit(X_rclr, genotype_labels)
```

## Evaluation signals

- Cross-validated CER table shows 3xTg PLS-DA models achieve lower classification error than 5xFAD models across tissues, matching the reported Figure 2A pattern
- Cecum and brain yield the best genotype classification in both studies after RCLR transformation
- Transformed matrix contains no undefined values: RCLR handles compositional zeros without the undefined-log problem of plain CLR
- PCA quality-control ordination on RCLR values separates samples by expected covariates (study, cohort, tissue, colonization)
- Selected features meet the VIP > 1 threshold consistent with Tables S1-S12

## Limitations

- Differences between Longitudinal and Sacrifice cohorts in both microbiome and metabolomics data are attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so cohort must be stratified before transformation
- Germ-free animals received an antibiotic cocktail during behavioral testing, which could have potentially altered organ metabolic profiles and confound transformed profiles
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound in any genotype classification built on transformed features

## Evidence

- [other] Preprocess extracted peak areas per tissue using robust center log ratio (RCLR) transformation as applied in the paper's PCA-based quality control.: [evidence span withheld — anchor status: not_found]
- [other] fit a PLS-DA classification model in scikit-learn, using variable importance (VIP > 1) features: [evidence span withheld — anchor status: not_found]
- [intro] a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples: "we generate and share a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples – including brain, serum, liver, cecal content, and feces – from 307 animals"
- [other] Evaluate each model via cross-validated classification error rate (CER): [evidence span withheld — anchor status: not_found]
