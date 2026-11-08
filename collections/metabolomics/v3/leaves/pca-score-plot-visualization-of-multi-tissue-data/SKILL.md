---
name: pca-score-plot-visualization-of-multi-tissue-data
description: Use when you have untargeted metabolomics peak-area tables with many
  zero-count features across multiple tissues and need to check whether samples cluster
  by study, cohort, genotype, or tissue before downstream stratified analyses.
when_to_use_negative:
- When the goal is single-animal developmental trajectories — note the Longitudinal
  fecal samples lack animal IDs (collected randomly per cage), so PCA clustering by
  cohort cannot be decomposed to individual animals
- When the input is already normalized/transformed compositional data — applying RCLR
  a second time would distort the log-ratio geometry
- When testing genotype-specific metabolite dysregulation — this skill is a global
  structure/batch-check (clustering by study and cohort), not a differential-abundance
  test
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Source of the deposited untargeted metabolomics peak-area data and job outputs
    used as input for the per-tissue PCA workflow
- name: scikit-learn
  role: Runs the principal component analysis on the RCLR-transformed per-tissue peak-area
    matrix, retaining PC1 and PC2 for score plotting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:c956b36aaba80431c334780a5462d2e4cb3b4dc6dcd56caeef19c0092cfc6972
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pca-score-plot-visualization-of-multi-tissue-data

## Summary

Applies robust center log ratio (RCLR) transformation to per-tissue untargeted metabolomics peak-area matrices and runs PCA (PC1 vs PC2) to reveal batch/cohort-level clustering across tissues in the 3xTg vs 5xFAD Alzheimer's disease mouse studies.

## When to use

Use when you have untargeted metabolomics peak-area tables with many zero-count features across multiple tissues and need to check whether samples cluster by study, cohort, genotype, or tissue before downstream stratified analyses.

## When NOT to use

- When the goal is single-animal developmental trajectories — note the Longitudinal fecal samples lack animal IDs (collected randomly per cage), so PCA clustering by cohort cannot be decomposed to individual animals
- When the input is already normalized/transformed compositional data — applying RCLR a second time would distort the log-ratio geometry
- When testing genotype-specific metabolite dysregulation — this skill is a global structure/batch-check (clustering by study and cohort), not a differential-abundance test

## Inputs

- Untargeted metabolomics extracted peak-area feature table (samples × features) per tissue type (brain, serum, liver, cecal content, feces), downloaded from GNPS/MassIVE
- Sample metadata table with study, cohort (Sacrifice vs Longitudinal), genotype, sex, age, and colonization status, from Qiita

## Outputs

- Per-tissue PC1 vs PC2 PCA score plots colored by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal), reproducing Figure S2
- Retained first two principal components per tissue (scores matrix)
- Decision on whether downstream analyses should be stratified by study/cohort

## How to apply

Subset the extracted peak-area table per tissue type (brain, serum, liver, cecal content, feces) and merge it with sample metadata (study, cohort, genotype, sex, age, colonization status). Apply RCLR transformation to each per-tissue peak-area matrix — rather than a plain CLR — because compositional metabolomics data contain zero-count features that RCLR handles by leaving zeros untransformed. Run PCA on the RCLR-transformed matrix per tissue using scikit-learn, retaining the first two principal components for score plotting. Plot PC1 vs PC2 score plots per tissue, coloring samples by study (3xTg vs 5xFAD) and cohort (Sacrifice vs Longitudinal). If clustering by study or cohort is observed, stratify all downstream differential-abundance and modeling analyses accordingly, as the paper did.

## Related tools

- **GNPS** (Source of the deposited untargeted metabolomics peak-area data and job outputs used as input for the per-tissue PCA workflow)
- **scikit-learn** (Runs the principal component analysis on the RCLR-transformed per-tissue peak-area matrix, retaining PC1 and PC2 for score plotting)

## Examples

```
from sklearn.decomposition import PCA; import numpy as np; X = np.loadtxt('brain_rclr.csv', delimiter=','); scores = PCA(n_components=2).fit_transform(X)
```

## Evaluation signals

- PCA score plots visually reproduce the reported observation that 'Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type'
- RCLR output contains no zeros among transformed values (zeros are left as-is / treated as missing) and all transformed values are log-ratio valid
- All five tissue subsets (brain, serum, liver, cecal content, feces) yield a PCA with metadata correctly merged — sample counts match the per-tissue peak-area table
- Colored score plots show separation along PC1/PC2 attributable to study and cohort rather than genotype alone, justifying stratification of downstream analyses

## Limitations

- Differences between Longitudinal and Sacrifice cohorts in metabolomics data are attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so cohort clustering reflects collection-method effects, not biology alone
- Germ-free animals received an antibiotic cocktail during behavioral testing, which may have altered organ metabolic profiles and thus PCA structure
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound that PCA clustering cannot disentangle
- Feature detection in deposited data can be influenced by study-specific extraction methods, workflows, and instrumentation

## Evidence

- [results] Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type: "Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type"
- [discussion] Differences between experimental cohorts ... can be attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
- [discussion] Germ-free animals received antibiotics during behavioral testing: "Germ-free animals have received a cocktail of antibiotics during behavioral testing to maintain their microbiological sterility and this could have potentially altered organ metabolic profiles"
- [discussion] Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation."
