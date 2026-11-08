---
name: metaboset-object-preprocessing
description: Use when starting from a preprocessed LC-MS metabolomics peak table (MS-DIAL
  Excel export) and you need a quality-controlled, drift-corrected, imputed feature-by-sample
  abundance matrix in R.
when_to_use_negative:
- Input is raw LC-MS instrument data — notame expects a peak table already produced
  by peak-picking software such as MS-DIAL and does not do peak detection itself
- You want to impute before quality assessment — QCs must be dropped and low-quality
  features flagged first; imputing the raw table with QCs included contradicts the
  workflow
- You are not working in R with MetaboSet objects — the functions (mark_nas, correct_drift,
  impute_rf) operate on the exprs part of a MetaboSet
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: R package providing the MetaboSet object and the preprocessing functions mark_nas,
    flag_detection, correct_drift, flag_quality, merge_metabosets, drop_qcs, impute_rf
    and impute_simple
  repo: https://github.com/antonvsdata/notame
- name: missForest
  role: random forest imputation engine behind notame's impute_rf function
- name: R
  role: implementation language; drift correction models the systematic intensity
    drift via the R function smooth.spline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:6f061e5ea4609cd028ea09b437e5e7353c5ff32cab66364f1ecb057df22605fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - R
  - missForest
  - notame
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metaboset-object-preprocessing

## Summary

Build and preprocess a notame MetaboSet object from an MS-DIAL-derived Excel peak table (e.g. sample_data_whole.xlsx): mark zeros as NA, flag low-quality features, apply cubic-spline drift correction, and finish with missForest random-forest imputation after QC removal.

## When to use

Use when starting from a preprocessed LC-MS metabolomics peak table (MS-DIAL Excel export) and you need a quality-controlled, drift-corrected, imputed feature-by-sample abundance matrix in R.

## When NOT to use

- Input is raw LC-MS instrument data — notame expects a peak table already produced by peak-picking software such as MS-DIAL and does not do peak detection itself
- You want to impute before quality assessment — QCs must be dropped and low-quality features flagged first; imputing the raw table with QCs included contradicts the workflow
- You are not working in R with MetaboSet objects — the functions (mark_nas, correct_drift, impute_rf) operate on the exprs part of a MetaboSet

## Inputs

- MS-DIAL-derived Excel peak table (e.g. sample_data_whole.xlsx) containing feature abundances, feature metadata and sample metadata
- R objects: one MetaboSet per LC-MS mode (e.g. positive and negative ionization modes)

## Outputs

- A merged, QC-dropped MetaboSet whose exprs matrix has no remaining NA values among non-flagged features (saved e.g. as full_data.RData)
- Flagged feature table recording detection and quality flags per molecular feature

## How to apply

Read the MS-DIAL peak table with read_from_excel and construct one MetaboSet object per LC-MS mode using construct_metabosets, so every feature, sample and abundance value lives in a single object. First represent missing values correctly by running mark_nas to replace zero abundances with NA, then flag features with poor detection via flag_detection and correct signal intensity drift with correct_drift, which models the drift using the R function smooth.spline (cubic spline correction in the style of Kirwan & Broadhurst). Run assess_quality and flag_quality so features failing the quality metrics of Broadhurst et al. are flagged, then merge the per-mode MetaboSets with merge_metabosets. Drop QC samples with drop_qcs before imputation, because QCs are only needed for quality assessment and imputation must be based on patterns in the biological data; finally set a reproducible seed and run impute_rf (missForest-based random forest imputation, the recommended strategy per Kokla et al.), keeping all_features = FALSE so only good-quality unflagged features are imputed.

## Related tools

- **notame** (R package providing the MetaboSet object and the preprocessing functions mark_nas, flag_detection, correct_drift, flag_quality, merge_metabosets, drop_qcs, impute_rf and impute_simple) — https://github.com/antonvsdata/notame
- **missForest** (random forest imputation engine behind notame's impute_rf function)
- **R** (implementation language; drift correction models the systematic intensity drift via the R function smooth.spline)

## Examples

```
set.seed(42); imputed <- impute_rf(qc_dropped, all_features = FALSE); saveRDS(imputed, "full_data.RData")
```

## Evaluation signals

- Output save file (e.g. full_data.RData) exists after the pipeline runs
- The imputed MetaboSet's exprs matrix contains no remaining NA values among non-flagged (good-quality) features
- Sample and feature dimensions of the imputed object match the pre-imputation, QC-dropped object
- Flagged/low-quality features (flag_detection, flag_quality) are excluded from imputation when all_features = FALSE
- QCs are absent from the object that enters imputation (drop_qcs succeeded)

## Limitations

- The notame package API is 'still quite experimental, and breaking changes are possible' — the antonvsdata repo is migrated and no longer maintained; active development is at hanhineva-lab/notame
- Random forest imputation is the recommended strategy, but simpler alternatives (impute_simple) exist if RF imputation is undesirable
- Untargeted LC-MS datasets inherently contain a significant number of low-quality features, so imputation quality depends on prior flagging of poor features

## Evidence

- [methods] mark_nas replaces given values in exprs part of the MetaboSet object with NA: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
- [methods] drift correction motivation: "Untargeted LC-MS metabolomics data usually suffers from signal intensity drift"
- [methods] quality flagging of features: "```flag_quality``` is used to flag features based on the other quality metrics"
- [methods] missing-value representation is the first preprocessing step: "As a first step in data preprocessing, missing values should be correctly represented."
- [readme] random forest imputation is the recommended strategy: [evidence span withheld — anchor status: external_doc]
- [readme] input must be preprocessed LC-MS data: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics."
- [readme] cubic spline drift correction: [evidence span withheld — anchor status: external_doc]
