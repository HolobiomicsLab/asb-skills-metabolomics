---
name: bh-multiple-testing-correction
description: Use when univariate association tests (e.g. partial Spearman correlations)
  have been run for every metabolic feature against a clinical phenotype and a multiple-testing-corrected
  pFDR column is needed before calling associations.
when_to_use_negative:
- Multivariate models (e.g. PCA/PLS-based screening) are being used where confounders
  like age and gender cannot be partialled out per-feature — MWASTools' univariate
  + confounder-adjusted pipeline is the intended route
- The association model is not univariate hypothesis testing on metabolic features
  (BH correction in this pipeline assumes per-feature p-values from MWAS_stats)
- A different multiple-testing family is required (the skill is specific to BH-adjusted
  pFDR via mt_method='BH')
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MWASTools
  role: Runs the confounder-adjusted univariate MWAS and applies BH multiple-testing
    correction internally via the mt_method argument, returning raw p-values and BH-adjusted
    pFDR per metabolic feature
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which MWASTools (Bioconductor package, R >= 3.4) is
    executed for the MWAS statistics and correction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:683dea31997a76678d1d8051723b4e51ade17a644b274b93781536955c5d07a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bh-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg (BH) false-discovery-rate correction to raw p-values from a metabolome-wide association study (MWAS), so that per-metabolite associations with a phenotype (e.g. BMI) survive testing across thousands of metabolic features.

## When to use

Use when univariate association tests (e.g. partial Spearman correlations) have been run for every metabolic feature against a clinical phenotype and a multiple-testing-corrected pFDR column is needed before calling associations.

## When NOT to use

- Multivariate models (e.g. PCA/PLS-based screening) are being used where confounders like age and gender cannot be partialled out per-feature — MWASTools' univariate + confounder-adjusted pipeline is the intended route
- The association model is not univariate hypothesis testing on metabolic features (BH correction in this pipeline assumes per-feature p-values from MWAS_stats)
- A different multiple-testing family is required (the skill is specific to BH-adjusted pFDR via mt_method='BH')

## Inputs

- CV-filtered SummarizedExperiment object (metabo_SE) with metabolic feature intensities (e.g. plasma 1H NMR buckets) per sample
- Clinical metadata dataframe/vector for the phenotype (disease_id, e.g. BMI) and confounders (e.g. Age, Gender, T2D)
- Raw per-feature p-value vector/matrix from univariate association testing

## Outputs

- 3-column result matrix per metabolic feature: correlation estimate, raw p-value, BH-adjusted pFDR
- Exported table (e.g. MWAS_BMI_matrix.csv) with one row per metabolic feature

## How to apply

Within MWASTools::MWAS_stats, set assoc_method='spearman' (rank-based partial correlation) with disease_id set to the phenotype of interest and confounder_ids to epidemiological covariates such as Age, Gender and T2D; the univariate-hypothesis-testing design of MWASTools exists precisely because multivariate models 'do not properly account for cofounding factors (e.g. age, gender)'. Pass mt_method='BH' so the function computes BH-adjusted pFDR values for all tested features in one call rather than correcting post hoc. The function returns a 3-column matrix per metabolic feature (correlation estimate, raw p-value, BH-adjusted pFDR); verify pFDR spans [0,1] and monotonicity vs. raw p-values. Finally export the corrected matrix (e.g. as MWAS_BMI_matrix.csv) with one row per metabolic feature.

## Related tools

- **MWASTools** (Runs the confounder-adjusted univariate MWAS and applies BH multiple-testing correction internally via the mt_method argument, returning raw p-values and BH-adjusted pFDR per metabolic feature) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which MWASTools (Bioconductor package, R >= 3.4) is executed for the MWAS statistics and correction)

## Examples

```
mwas <- MWASTools::MWAS_stats(metabo_SE, disease_id="BMI", confounder_ids=c("Age","Gender","T2D"), assoc_method="spearman", mt_method="BH")
```

## Evaluation signals

- Output matrix has exactly 3 columns (correlation estimate, raw p-value, BH-adjusted pFDR)
- Output has exactly one row per metabolic feature in metabo_SE
- All raw p-values and pFDR values lie within [0,1]
- BH-adjusted pFDR values are monotonically non-decreasing relative to raw p-values and always >= raw p-values
- Exported CSV (MWAS_BMI_matrix.csv) exists and matches the in-memory matrix row/column count

## Limitations

- BH correction adjusts for the number of features tested; if features are pre-filtered inconsistently the correction denominator changes, so the CV-filtered SummarizedExperiment must define the final feature set before correction
- pFDR adjustment addresses only multiplicity — residual confounding must still be handled explicitly via confounder_ids (e.g. Age, Gender, T2D)
- Rank-based (Spearman) associations capture monotonic, not necessarily linear, relationships; BH-corrected significance should be interpreted alongside the correlation estimate

## Evidence

- [intro] univariate MWAS with confounder handling motivating BH correction: "Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders."
- [abstract] association models adjusted for confounders: "metabolome-wide association analysis using various models (partial correlations, generalized linear models)"
- [intro] confounder limitation motivating per-feature univariate testing and correction: "However, a major limitation of these multivariate models from the epidemiological perspective is that they do not properly account for cofounding factors (e.g. age, gender), which might distort the"
