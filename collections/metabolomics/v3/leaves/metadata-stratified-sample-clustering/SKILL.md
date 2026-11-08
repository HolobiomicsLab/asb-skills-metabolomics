---
name: metadata-stratified-sample-clustering
description: Use when a multi-batch untargeted metabolomics peak-area table (e.g.,
  brain, serum, liver, cecal content, feces from 3xTg and 5xFAD mouse cohorts) must
  be checked for batch/cohort clustering before differential analysis is pooled across
  studies.
when_to_use_negative:
- Data are already a normalized, compositional feature table that has been stratified
  and does not require a batch/clustering check.
- The analysis goal is a targeted quantification of a single metabolite rather than
  an exploratory clustering assessment of an untargeted peak-area matrix.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: GNPS
  role: Public repository hosting the deposited untargeted metabolomics peak-area
    data and associated MS/MS spectra used as the input for per-tissue RCLR-PCA clustering.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:d4c5a89307ec75bfeb354bf19def1af15b72834037bfd33972551fff040a9c83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metadata-stratified-sample-clustering

## Summary

Applies robust center log ratio (RCLR) transformation to per-tissue untargeted metabolomics peak-area matrices followed by PCA to reveal whether samples cluster by study (3xTg vs 5xFAD), cohort (Sacrifice vs Longitudinal), genotype, or colonization status, guiding downstream stratification.

## When to use

Use when a multi-batch untargeted metabolomics peak-area table (e.g., brain, serum, liver, cecal content, feces from 3xTg and 5xFAD mouse cohorts) must be checked for batch/cohort clustering before differential analysis is pooled across studies.

## When NOT to use

- Data are already a normalized, compositional feature table that has been stratified and does not require a batch/clustering check.
- The analysis goal is a targeted quantification of a single metabolite rather than an exploratory clustering assessment of an untargeted peak-area matrix.

## Inputs

- untargeted metabolomics peak-area table (per tissue: brain, serum, liver, cecal content, feces)
- sample metadata (study, cohort, genotype, sex, age, colonization status)

## Outputs

- PC1 vs PC2 PCA score plots colored by study and cohort per tissue
- stratification decision (whether analyses are pooled or split by study/cohort)

## How to apply

Subset the deposited untargeted metabolomics peak-area table (GNPS/MassIVE or Qiita) by tissue (brain, serum, liver, cecal content, feces) and merge sample metadata columns for study, cohort, genotype, sex, age, and colonization status. Apply robust center log ratio (RCLR) transformation to each per-tissue peak-area matrix to handle zero-count features, then run PCA (scikit-learn) retaining PC1 and PC2 for score plotting. Color samples by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal) to reproduce the clustering pattern reported in Figure S2; use observed clustering to decide whether downstream analyses must be stratified rather than pooled.

## Related tools

- **GNPS** (Public repository hosting the deposited untargeted metabolomics peak-area data and associated MS/MS spectra used as the input for per-tissue RCLR-PCA clustering.)

## Examples

```
python pca_rclr_plot.py --input metabolomics_peak_areas.csv --metadata sample_metadata.tsv --tissue serum --color-by study,cohort
```

## Evaluation signals

- RCLR-transformed PCA score plots per tissue visually reproduce the paper's Figure S2 pattern where samples cluster by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal).
- Per-tissue matrices are zero-inflated appropriately: RCLR handles zero-count features without requiring imputation before PCA.
- The stratification decision derived from the PCA matches the paper's reported rationale that downstream analyses were stratified because clustering was observed across studies and cohorts.
- Sample metadata merge yields correct group labels for each sample (study, cohort, genotype, sex, age, colonization status) before plotting.

## Limitations

- Differences between Longitudinal and Sacrifice cohorts in both microbiome and metabolomics data can be attributed to differing sample collection (fresh fecal vs. colon after sacrifice), which complicates interpretation of clustering.
- Feature detection can be influenced by study-specific extraction methods, workflows, and instrumentation; lack of detection does not necessarily mean absence in tissue.
- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound that could drive observed clustering.
- Germ-free animals received an antibiotic cocktail during behavioral testing that could have altered organ metabolic profiles and thus affected clustering patterns.

## Evidence

- [other] PCA of RCLR-transformed peak areas shows metabolomics clustering by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal) for each tissue, so downstream analyses were stratified accordingly.: [evidence span withheld — anchor status: not_found]
- [results] Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type: "Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type"
- [discussion] Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
