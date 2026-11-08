---
name: metabonic-feature-clinical-variable-testing
description: Use when a SummarizedExperiment of metabolomic feature intensities plus
  clinical metadata (phenotype + confounders) must be screened feature-by-feature
  for phenotype associations with rank-based, confounder-adjusted statistics and FDR
  control.
when_to_use_negative:
- The analysis goal is multivariate modeling (e.g. PLS-based discrimination) rather
  than univariate feature-by-feature phenotype association testing with confounder
  adjustment
- Downstream steps such as quality control analysis, STOCSY metabolite assignment,
  or network-based biological interpretation — this skill covers only the association-testing
  stage of the MWASTools pipeline
- The phenotype/confounder metadata columns are missing or not aligned with the feature
  intensity matrix, since partial Spearman correlations cannot be computed without
  covariate rows matched to samples
tools:
- name: MWASTools
  role: Provides MWAS_stats, the function that computes confounder-adjusted partial
    Spearman correlations per metabolic feature and applies BH multiple-testing correction
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment; MWASTools is implemented in R (version >=3.4) and the
    association matrix is assembled and exported in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:b0a69ea05b2782cc3a5930197b93c42f2f61f666e06b0dc02f221e61cdfba135
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabonic-feature-clinical-variable-testing

## Summary

Performs metabolome-wide association testing of metabolic features (e.g. 1H NMR plasma intensities) against a clinical variable such as BMI using partial Spearman correlations adjusted for epidemiological confounders (age, gender, T2D), with BH multiple-testing correction, via MWASTools.

## When to use

Use when a SummarizedExperiment of metabolomic feature intensities plus clinical metadata (phenotype + confounders) must be screened feature-by-feature for phenotype associations with rank-based, confounder-adjusted statistics and FDR control.

## When NOT to use

- The analysis goal is multivariate modeling (e.g. PLS-based discrimination) rather than univariate feature-by-feature phenotype association testing with confounder adjustment
- Downstream steps such as quality control analysis, STOCSY metabolite assignment, or network-based biological interpretation — this skill covers only the association-testing stage of the MWASTools pipeline
- The phenotype/confounder metadata columns are missing or not aligned with the feature intensity matrix, since partial Spearman correlations cannot be computed without covariate rows matched to samples

## Inputs

- CV-filtered SummarizedExperiment object (metabo_SE) of metabolic feature intensities, e.g. from plasma 1H NMR spectra
- Clinical metadata table with the phenotype column (e.g. BMI) and confounder columns (Age, Gender, T2D)

## Outputs

- 3-column association matrix per metabolic feature: correlation estimate, raw p-value, BH-adjusted pFDR
- Exported results file such as MWAS_BMI_matrix.csv

## How to apply

Load the CV-filtered SummarizedExperiment (metabo_SE) holding metabolic feature intensities and the clinical metadata columns (e.g. BMI, Age, Gender, T2D). Call MWASTools::MWAS_stats with disease_id set to the phenotype of interest and confounder_ids listing the epidemiological confounders, choosing assoc_method='spearman' for a rank-based partial correlation that handles non-normal metabolite distributions. Request mt_method='BH' so all features receive Benjamini-Hochberg-adjusted pFDR values, addressing the package authors' rationale that univariate testing properly accounts for confounders that distort associations in multivariate models. Assemble the returned 3-column matrix (correlation estimate, raw p-value, BH-adjusted pFDR) with one row per metabolic feature and export it, e.g. as MWAS_BMI_matrix.csv. Validate that the CSV has exactly 3 columns, one row per feature, and that all raw p-values and pFDR values fall within [0,1].

## Related tools

- **MWASTools** (Provides MWAS_stats, the function that computes confounder-adjusted partial Spearman correlations per metabolic feature and applies BH multiple-testing correction) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment; MWASTools is implemented in R (version >=3.4) and the association matrix is assembled and exported in R)

## Examples

```
MWAS_stats(metabo_SE, disease_id="BMI", confounder_ids=c("Age","Gender","T2D"), assoc_method="spearman", mt_method="BH")
```

## Evaluation signals

- The exported matrix (e.g. MWAS_BMI_matrix.csv) exists and contains exactly 3 columns
- The result matrix has exactly one row per metabolic feature in the SummarizedExperiment object
- All raw p-values and BH-adjusted pFDR values are within [0,1]
- The association statistic is a correlation-type estimate consistent with assoc_method='spearman' rather than a regression coefficient
- pFDR values reflect Benjamini-Hochberg correction across all tested features (monotone relative to raw p-values after rank ordering)

## Limitations

- Univariate rank-based testing (Spearman) captures monotonic, per-feature associations only; it does not model multivariate metabolic patterns
- Results depend on correct specification of confounders; the article notes that failing to account for cofounding factors such as age and gender can distort observed metabolite-condition associations
- The skill covers association testing only; MWASTools' other pipeline stages (QC, visualization, STOCSY assignment, biological interpretation) are separate

## Evidence

- [abstract] Integrated MWAS pipeline: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [abstract] Available association models: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
- [intro] Univariate testing with confounder handling: "Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders."
- [intro] Rationale for confounder adjustment: "However, a major limitation of these multivariate models from the epidemiological perspective is that they do not properly account for cofounding factors (e.g. age, gender), which might distort the"
