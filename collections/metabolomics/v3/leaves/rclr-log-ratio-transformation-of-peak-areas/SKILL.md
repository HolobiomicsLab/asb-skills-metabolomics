---
name: rclr-log-ratio-transformation-of-peak-areas
description: Use when per-tissue untargeted metabolomics peak-area tables contain
  many zero-count features and the goal is compositional-aware ordination (e.g., PCA
  score plots) of samples across conditions.
when_to_use_negative:
- Input data are not compositional peak-area intensities (e.g. count data or already-normalized
  ratios) where log-ratio transformation is inappropriate
- The analysis goal is per-sample longitudinal trajectories rather than group-level
  ordination — the Longitudinal fecal samples lack animal IDs, so single-animal developmental
  analysis is not supported
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Source of the deposited untargeted metabolomics peak-area data used as input
    for the RCLR transformation and PCA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:19d73385efc4cc6ca3aee267c2bcd473a7bd0fe269b43f6eb9d349e49ad5e47f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# rclr-log-ratio-transformation-of-peak-areas

## Summary

Apply robust center log-ratio (RCLR) transformation to per-tissue untargeted metabolomics peak-area matrices so that sparse, compositional feature data become suitable for PCA ordination and downstream stratified analyses.

## When to use

Use when per-tissue untargeted metabolomics peak-area tables contain many zero-count features and the goal is compositional-aware ordination (e.g., PCA score plots) of samples across conditions.

## When NOT to use

- Input data are not compositional peak-area intensities (e.g. count data or already-normalized ratios) where log-ratio transformation is inappropriate
- The analysis goal is per-sample longitudinal trajectories rather than group-level ordination — the Longitudinal fecal samples lack animal IDs, so single-animal developmental analysis is not supported

## Inputs

- Extracted untargeted metabolomics peak-area tables per tissue type (brain, serum, liver, cecal content, feces) from the 3xTg and 5xFAD studies (Sacrifice and Longitudinal cohorts)
- Sample metadata (study, cohort, genotype, sex, age, colonization status) matched to the peak-area rows

## Outputs

- RCLR-transformed peak-area matrix per tissue
- PCA PC1 vs PC2 score plots per tissue, colored by study and cohort (reproducing Figure S2)

## How to apply

For each tissue type (brain, serum, liver, cecal content, feces), subset the extracted peak-area table and merge the corresponding sample metadata (study, cohort, genotype, sex, age, colonization status). Apply RCLR to the per-tissue peak-area matrix — RCLR replaces zero-count features with a small pseudocount before log-ratio computation, handling zero-count features that break a plain CLR. Run PCA (e.g. scikit-learn) on the RCLR-transformed matrix, retaining PC1 and PC2 for score plotting. Color samples by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal) to check clustering, then stratify downstream analyses accordingly.

## Related tools

- **GNPS** (Source of the deposited untargeted metabolomics peak-area data used as input for the RCLR transformation and PCA)

## Examples

```
from sklearn.decomposition import PCA; import skbio.stats.composition as cmp; X_rclr = cmp.rclr(X_peaks); pc = PCA(n_components=2).fit_transform(X_rclr)
```

## Evaluation signals

- PCA score plots per tissue visually reproduce the reported observation that 'Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type'
- All zero-count features are handled by the RCLR pseudocount without producing NaN/inf values in the transformed matrix
- Transformed matrices retain the same sample-by-feature shape as the raw per-tissue peak-area tables
- Sample metadata (study, cohort, genotype, sex, age, colonization status) aligns 1:1 with the transformed rows

## Limitations

- Differences between the Longitudinal and Sacrifice cohorts in both microbiome and metabolomics data are attributed to differing sample collection (fresh fecal vs. colon after sacrifice), which the RCLR/PCA step cannot disentangle — downstream analyses must be stratified by cohort
- Germ-free animals received an antibiotic cocktail during behavioral testing, which could have altered organ metabolic profiles and thus the compositional structure of the peak-area matrices
- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound in the clustering pattern observed after RCLR/PCA

## Evidence

- [methods] Apply robust center log ratio (RCLR) transformation to each per-tissue peak-area matrix to handle zero-count features.: [evidence span withheld — anchor status: not_found]
- [results] Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type.: "Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type"
- [intro] we generate and share a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples – including brain, serum, liver, cecal content, and feces: "we generate and share a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples – including brain, serum, liver, cecal content, and feces – from 307 animals"
- [discussion] Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
