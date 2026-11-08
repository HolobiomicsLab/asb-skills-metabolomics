---
name: sample-level-data-normalization
description: Use when raw metabolomics concentration matrices (e.g., introgression-line
  tomato pericarp profiles) are loaded and must be normalized per sample and per metabolite
  before missing-value imputation and pairwise Pearson correlation networks.
when_to_use_negative:
- Input is already a correlation-based nodes-and-edges table (e.g., metabolite_corr_network.csv)
  — normalization acts on raw metabolite concentration profiles, upstream of Metabolite
  CN construction.
- The analysis goal is downstream NetAurHPD work (auralization, M5 training, Youden
  threshold optimization) rather than the metabolite-profile preprocessing stage.
edam_operation: http://edamontology.org/operation_2935
tools:
- name: Rstudio
  role: Environment in which the normalization and the subsequent pcaMethods probabilistic-PCA
    imputation of the metabolite profiles are executed.
- name: pcaMethods
  role: R package applied immediately after normalization to impute missing values
    in the normalized metabolite profiles matrix via probabilistic PCA.
- name: NetAurHPD
  role: Overall framework whose metabolomics pipeline consumes the normalized (and
    imputed) profiles for Metabolite CN construction and pathway prediction.
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_content_hash: sha256:26633a4f30c7acff724e6dd4e7172ff4dca22e9a64cdf419ab0e435424e93b8a
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - NetAurHPD
  - Rstudio
  - pcaMethods
  edam_operation: http://edamontology.org/operation_2935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Sample-level data normalization

## Summary

Preprocessing skill from the NetAurHPD metabolomics pipeline: normalize tomato pericarp metabolic profiles (harvesting seasons 2001, 2003, 2004) at both the sample and metabolite levels, as the preliminary step before probabilistic-PCA missing-value imputation and Metabolite correlation network construction.

## When to use

Use when raw metabolomics concentration matrices (e.g., introgression-line tomato pericarp profiles) are loaded and must be normalized per sample and per metabolite before missing-value imputation and pairwise Pearson correlation networks.

## When NOT to use

- Input is already a correlation-based nodes-and-edges table (e.g., metabolite_corr_network.csv) — normalization acts on raw metabolite concentration profiles, upstream of Metabolite CN construction.
- The analysis goal is downstream NetAurHPD work (auralization, M5 training, Youden threshold optimization) rather than the metabolite-profile preprocessing stage.

## Inputs

- Raw tomato pericarp metabolic profiles matrix for harvesting seasons 2001, 2003, and 2004 (samples = introgression lines × metabolite concentrations)
- Normalization parameters at the sample and metabolite levels

## Outputs

- Normalized metabolite profiles matrix (prerequisite for pcaMethods probabilistic-PCA imputation)
- metabolite_profiles_imputed.csv — completed profiles matrix with no missing values, feeding downstream Metabolite CN construction

## How to apply

Load the tomato pericarp metabolic profiles dataset covering the three harvesting seasons (2001, 2003, 2004), where each sample records metabolite concentrations. Normalize at both levels — sample level (across metabolites within each biological replicate) and metabolite level (across replicates) — as the preliminary preprocessing step before the pcaMethods probabilistic PCA imputation in RStudio. Work in the RStudio environment with the pcaMethods package alongside the NetAurHPD toolchain. After normalization, apply probabilistic PCA to fill missing values and save the completed matrix (metabolite_profiles_imputed.csv); the imputed matrix must contain no missing values and feeds Metabolite CN construction, where Pearson correlations are thresholded at |r| >= 0.3 with p <= 0.01. Correct normalization is judged indirectly: correlation networks built from the processed data should reproduce the reported reference sizes (2001: 75 nodes/473 edges; 2003: 75 nodes/869 edges; 2004: 78 nodes/338 edges).

## Related tools

- **Rstudio** (Environment in which the normalization and the subsequent pcaMethods probabilistic-PCA imputation of the metabolite profiles are executed.)
- **pcaMethods** (R package applied immediately after normalization to impute missing values in the normalized metabolite profiles matrix via probabilistic PCA.)
- **NetAurHPD** (Overall framework whose metabolomics pipeline consumes the normalized (and imputed) profiles for Metabolite CN construction and pathway prediction.) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
Rscript -e 'library(pcaMethods); norm <- normalize(profiles); imp <- pca(norm, method="ppca", nPcs=2); complete <- completeObs(imp); write.csv(complete, "metabolite_profiles_imputed.csv", row.names=FALSE)'
```

## Evaluation signals

- The completed profiles matrix saved as metabolite_profiles_imputed.csv must contain no missing values after normalization + probabilistic PCA imputation.
- Downstream Metabolite CN sizes must match the reported reference values: 2001 — 75 metabolites/473 links; 2003 — 75 nodes/869 edges; 2004 — 78 nodes/338 edges.
- Retained correlation links in the Metabolite CN must satisfy the threshold |r| >= 0.3 and p <= 0.01.
- Imputation must use the pcaMethods probabilistic PCA method (available in RStudio) rather than a generic fill, per the article's stated preliminary step.

## Limitations

- The article does not specify the exact normalization formula/statistic used at each level, so reproducibility of the sample/metabolite-level normalization step itself relies on the original authors' implementation.
- Normalization quality is only observable through downstream effects (network sizes, AUC variability), which the article reports can vary: AUC range averaged 0.082 for NetAurHPD alone, decreasing to 0.07 with one augmentation.

## Evidence

- [methods] we calculate the Pearson Correlation Coefficient between every pair of metabolites: [evidence span withheld — anchor status: not_found]
- [methods] the selected correlation threshold was set to be |r|≥ 0.3 and p≤0.01: [evidence span withheld — anchor status: not_found]
