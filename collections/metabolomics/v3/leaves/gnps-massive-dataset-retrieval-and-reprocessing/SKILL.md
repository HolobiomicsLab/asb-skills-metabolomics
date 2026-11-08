---
name: gnps-massive-dataset-retrieval-and-reprocessing
description: Use when reproducing or extending analyses from a publicly deposited
  untargeted metabolomics dataset (e.g. peak-area feature tables plus sample metadata
  hosted on GNPS/MassIVE or Qiita) instead of acquiring new instrument data.
when_to_use_negative:
- You have raw instrument files (e.g. mzML) and need feature detection/peak picking
  first — this skill starts from a deposited peak-area table
- The dataset lacks usable sample metadata, so samples cannot be keyed to study, cohort,
  or tissue factors for stratified reprocessing
- The goal is MS/MS molecular networking or spectral annotation (e.g. against GNPS
  reference libraries), not ordination of quantified peak areas
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Public metabolomics repository from which the deposited untargeted peak-area
    data and MS/MS spectra are retrieved for downstream reprocessing (RCLR + PCA per
    tissue)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:80055883d1d76681f08dd414dd13df8032c16b161cccc6c2f5e18423a51e42c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - molDiscovery
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- gnps-massive-dataset-retrieval
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gnps-massive-dataset-retrieval-and-reprocessing

## Summary

Retrieve deposited untargeted metabolomics peak-area tables and sample metadata from public repositories (GNPS/MassIVE, Qiita) and reprocess them — here, per-tissue RCLR transformation followed by PCA — to reproduce or extend published analyses from secondary data.

## When to use

Use when reproducing or extending analyses from a publicly deposited untargeted metabolomics dataset (e.g. peak-area feature tables plus sample metadata hosted on GNPS/MassIVE or Qiita) instead of acquiring new instrument data.

## When NOT to use

- You have raw instrument files (e.g. mzML) and need feature detection/peak picking first — this skill starts from a deposited peak-area table
- The dataset lacks usable sample metadata, so samples cannot be keyed to study, cohort, or tissue factors for stratified reprocessing
- The goal is MS/MS molecular networking or spectral annotation (e.g. against GNPS reference libraries), not ordination of quantified peak areas

## Inputs

- Deposited untargeted metabolomics peak-area feature table (e.g. from GNPS/MassIVE; 2,271 samples across five tissue types)
- Sample metadata table keyed to the feature table (study, cohort, genotype, sex, age, colonization status)
- Tissue-type labels for stratifying the peak-area matrix (brain, serum, liver, cecal content, feces)

## Outputs

- Per-tissue RCLR-transformed peak-area matrices
- PC1 vs PC2 score plots per tissue, colored by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal)
- Reproduction verdict on reported clustering (here: clustering across studies and cohorts per tissue, as in Figure S2)

## How to apply

Download the deposited peak-area feature table and sample metadata for the study of interest from GNPS/MassIVE (and Qiita for paired microbiome data). Subset the feature table per tissue type (here: brain, serum, liver, cecal content, feces) and merge the corresponding metadata columns (study, cohort, genotype, sex, age, colonization status). Apply robust center log ratio (RCLR) transformation to each per-tissue peak-area matrix, because RCLR explicitly handles zero-count features that break standard CLR. Run PCA on the RCLR-transformed matrix per tissue using scikit-learn, retaining PC1 and PC2 for score plotting, and color samples by the metadata factors of interest (study: 3xTg vs 5xFAD; cohort: Sacrifice vs Longitudinal) to check whether reported clustering patterns are reproduced.

## Related tools

- **GNPS** (Public metabolomics repository from which the deposited untargeted peak-area data and MS/MS spectra are retrieved for downstream reprocessing (RCLR + PCA per tissue))

## Examples

```
from sklearn.decomposition import PCA; pcs = PCA(n_components=2).fit_transform(rclr_transform(peak_area_table))
```

## Evaluation signals

- RCLR-transformed per-tissue matrices contain no undefined values caused by zero-count features (RCLR is chosen specifically to handle zeros)
- Merged sample count matches the deposited dataset scale (2,271 samples / 2,266 unique biological samples from 307 animals)
- PCA is run separately per tissue, and PC1 vs PC2 score plots visually reproduce the reported separation by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal)
- Downstream stratification decision (analyze per study and per cohort) is justified by the observed clustering, matching the paper's logic

## Limitations

- Feature detection in reprocessed reference datasets is influenced by study-specific extraction methods, workflows, and instrumentation, so lack of detection does not mean absence in a tissue
- In this dataset, Longitudinal fecal samples were collected randomly per cage without animal IDs, preventing single-animal developmental analysis on the retrieved data
- Observed cohort differences (Longitudinal vs Sacrifice) can partly reflect differing sample collection (fresh fecal vs. colon after sacrifice), a confound the reprocessing cannot remove

## Evidence

- [other] Download deposited data from GNPS/MassIVE and Qiita: [evidence span withheld — anchor status: not_found]
- [methods] RCLR transformation to handle zero-count features: [evidence span withheld — anchor status: not_found]
- [abstract] Scale and composition of the deposited dataset: "a multi-organ untargeted metabolomics dataset (2,271 samples) generated from five tissue types in two genetic AD mouse models under colonized or germ-free conditions"
- [results] Reported clustering observation to validate against: "Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type"
