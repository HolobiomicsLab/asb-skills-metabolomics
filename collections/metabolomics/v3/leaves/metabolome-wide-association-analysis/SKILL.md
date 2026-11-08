---
name: metabolome-wide-association-analysis
description: Use when a SummarizedExperiment of metabolic feature intensities (e.g.
  plasma 1H NMR) plus clinical metadata (phenotype + confounders) must be screened
  feature-by-feature for phenotype associations with multiple-testing control.
when_to_use_negative:
- The goal is multivariate modelling of metabonomic data where confounders are not
  handled feature-wise — the article notes multivariate packages 'do not properly
  account for cofounding factors (e.g. age, gender)', which is precisely why the univariate
  MWAS route is used instead
- The task is metabolite assignment, QC or network/pathway interpretation (STOCSY,
  biological interpretation), which are separate MWASTools functionalities rather
  than the association-testing step
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3277
- http://edamontology.org/topic_0153
tools:
- name: MWASTools
  role: R/Bioconductor package providing the MWAS_stats function used to run partial
    Spearman metabolome-wide association with confounder adjustment and BH multiple-testing
    correction
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (version >= 3.4) in which MWASTools is implemented and
    the MWAS pipeline is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:09978f6fa5560c4d251ec9b2608b26bfc79379b91fa7bd21e85acdc4b5c6894d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3277
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolome-wide-association-analysis

## Summary

Performs metabolome-wide association studies (MWAS) by testing each metabolic feature (e.g. 1H NMR intensity) for association with a phenotype such as BMI, using univariate models — partial Spearman correlations or generalized linear models — with efficient handling of epidemiological confounders like age and gender, plus Benjamini-Hochberg multiple-testing correction.

## When to use

Use when a SummarizedExperiment of metabolic feature intensities (e.g. plasma 1H NMR) plus clinical metadata (phenotype + confounders) must be screened feature-by-feature for phenotype associations with multiple-testing control.

## When NOT to use

- The goal is multivariate modelling of metabonomic data where confounders are not handled feature-wise — the article notes multivariate packages 'do not properly account for cofounding factors (e.g. age, gender)', which is precisely why the univariate MWAS route is used instead
- The task is metabolite assignment, QC or network/pathway interpretation (STOCSY, biological interpretation), which are separate MWASTools functionalities rather than the association-testing step

## Inputs

- CV-filtered SummarizedExperiment object (metabo_SE) of metabolic feature intensities (e.g. plasma 1H NMR features)
- Clinical metadata vector for the target phenotype (e.g. BMI)
- Confounder metadata columns (e.g. Age, Gender, T2D)

## Outputs

- 3-column result matrix per metabolic feature: association (correlation) estimate, raw p-value, BH-adjusted pFDR
- Exported results table (e.g. MWAS_BMI_matrix.csv)

## How to apply

Load the CV-filtered SummarizedExperiment object (metabo_SE) holding metabolic feature intensities and the clinical metadata (e.g. BMI, Age, Gender, T2D). Run MWASTools::MWAS_stats with the target phenotype as disease_id (e.g. "BMI"), the covariates as confounder_ids (e.g. c("Age","Gender","T2D")), and assoc_method="spearman" to compute a rank-based partial association estimate for every metabolic feature while partialling out confounders — the article motivates this because multivariate models 'do not properly account for cofounding factors (e.g. age, gender)'. Apply multiple-testing correction via mt_method="BH" to obtain BH-adjusted pFDR values for all tested features. Assemble the 3-column output matrix (correlation estimate, raw p-value, BH-adjusted pFDR) with one row per metabolic feature, then export it (e.g. MWAS_BMI_matrix.csv). Generalized linear models are an alternative association model when a rank-based partial correlation is not appropriate.

## Related tools

- **MWASTools** (R/Bioconductor package providing the MWAS_stats function used to run partial Spearman metabolome-wide association with confounder adjustment and BH multiple-testing correction) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (version >= 3.4) in which MWASTools is implemented and the MWAS pipeline is executed)

## Examples

```
MWAS_stats(metabo_SE, disease_id="BMI", confounder_ids=c("Age","Gender","T2D"), assoc_method="spearman", mt_method="BH")
```

## Evaluation signals

- The exported CSV exists and contains exactly 3 columns (correlation estimate, raw p-value, BH-adjusted pFDR)
- The result matrix has exactly one row per metabolic feature in the input SummarizedExperiment
- All raw p-values and pFDR values fall within [0,1]
- Every tested feature carries a BH-adjusted pFDR, confirming mt_method="BH" was applied across the full feature set
- Confounders (e.g. Age, Gender, T2D) were supplied via confounder_ids so estimates are partial, not marginal, correlations

## Limitations

- Multivariate models commonly used for detecting metabolic changes do not properly account for confounding factors such as age and gender, which might distort observed associations — hence univariate partial testing is used
- Association model choice matters: only partial correlations and generalized linear models are the supported univariate models mentioned; Spearman is rank-based and captures monotonic, not necessarily linear, associations

## Evidence

- [abstract] metabolome-wide association analysis using various models (partial correlations, generalized linear models): "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
- [intro] a major limitation of these multivariate models from the epidemiological perspective is that they do not properly account for cofounding factors (e.g. age, gender): "However, a major limitation of these multivariate models from the epidemiological perspective is that they do not properly account for cofounding factors (e.g. age, gender), which might distort the"
- [intro] Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders.: "Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders."
- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
