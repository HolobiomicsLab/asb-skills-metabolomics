---
name: partial-correlation-confounder-adjustment
description: Use when testing metabolite–phenotype associations in a metabolomics
  dataset (e.g. plasma 1H NMR features vs BMI) while adjusting for epidemiological
  confounders such as age, gender or disease status, and a rank-based univariate model
  with pFDR control is needed.
when_to_use_negative:
- When the analysis goal is multivariate modelling without explicit confounder adjustment
  — the article states existing multivariate packages fail to properly account for
  confounding factors such as age and gender, which is exactly what this univariate
  partial-correlation approach addresses.
- When the data container is not a SummarizedExperiment/MultiAssayExperiment object
  with matched phenotype metadata (BMI, confounder columns); MWAS_stats expects this
  structure.
- When per-feature stability of estimates is not needed and bootstrapping would add
  unnecessary compute — bootstrap CIs only validate, they do not replace the BH-corrected
  association test.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: MWASTools
  role: R/Bioconductor package providing MWAS_stats for confounder-adjusted partial
    Spearman MWAS with BH correction, and MWAS_bootstrapping for non-parametric validation
    of the association model
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (version >= 3.4) in which MWASTools is implemented and
    the MWAS pipeline is executed
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:ecb5302576d2fca331fb3a3c14efa57fc2144fbc8af391ecf82eeb821ba3b9c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# partial-correlation-confounder-adjustment

## Summary

Perform metabolome-wide association studies (MWAS) of a phenotype such as BMI using partial Spearman correlations that partial out epidemiological confounders (e.g. Age, Gender, T2D) with BH multiple-testing correction, as implemented in MWASTools::MWAS_stats on a SummarizedExperiment metabolomics container.

## When to use

Use when testing metabolite–phenotype associations in a metabolomics dataset (e.g. plasma 1H NMR features vs BMI) while adjusting for epidemiological confounders such as age, gender or disease status, and a rank-based univariate model with pFDR control is needed.

## When NOT to use

- When the analysis goal is multivariate modelling without explicit confounder adjustment — the article states existing multivariate packages fail to properly account for confounding factors such as age and gender, which is exactly what this univariate partial-correlation approach addresses.
- When the data container is not a SummarizedExperiment/MultiAssayExperiment object with matched phenotype metadata (BMI, confounder columns); MWAS_stats expects this structure.
- When per-feature stability of estimates is not needed and bootstrapping would add unnecessary compute — bootstrap CIs only validate, they do not replace the BH-corrected association test.

## Inputs

- SummarizedExperiment/MultiAssayExperiment object (metabo_SE) with CV-filtered metabolic feature intensities
- Clinical/phenotype metadata columns: disease variable (e.g. BMI), confounders (e.g. Age, Gender, T2D)
- Feature identifier (metabolite_id) and association method choice (e.g. "spearman")
- For validation: number of bootstrap iterations and confounder_ids passed to MWAS_bootstrapping

## Outputs

- 3-column result matrix per metabolic feature (correlation estimate, raw p-value, BH-adjusted pFDR), e.g. MWAS_BMI_matrix.csv
- Optional bootstrap validation table per feature: point estimate, bootstrap mean, lower/upper percentile confidence-interval bounds, e.g. bootstrap_validation.csv

## How to apply

Load the CV-filtered SummarizedExperiment object (metabo_SE) holding metabolic feature intensities plus clinical metadata (BMI, Age, Gender, T2D), then call MWASTools::MWAS_stats with disease_id="BMI", confounder_ids=c("Age","Gender","T2D") and assoc_method="spearman" to compute a rank-based association estimate for each metabolic feature while partialling out the confounders via partial regression on confounder residuals. Apply multiple-testing correction with mt_method="BH" to obtain BH-adjusted pFDR values across all tested features. Assemble the 3-column output matrix (correlation estimate, raw p-value, BH-adjusted pFDR) with one row per metabolic feature and export it, e.g. as MWAS_BMI_matrix.csv. The rationale, per the article, is that existing multivariate packages 'do not properly account for cofounding factors (e.g. age, gender)', so univariate hypothesis testing with explicit confounder handling avoids distorted metabolite–condition associations. Optionally validate significant associations by non-parametric bootstrapping with MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id, confounder_ids, assoc_method, iterations), which resamples subjects with replacement and derives percentile-based confidence intervals for each feature's estimate.

## Related tools

- **MWASTools** (R/Bioconductor package providing MWAS_stats for confounder-adjusted partial Spearman MWAS with BH correction, and MWAS_bootstrapping for non-parametric validation of the association model) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (version >= 3.4) in which MWASTools is implemented and the MWAS pipeline is executed)

## Examples

```
MWAS_stats(metabo_SE, disease_id="BMI", confounder_ids=c("Age","Gender","T2D"), assoc_method="spearman", mt_method="BH")
```

## Evaluation signals

- The exported CSV exists and contains exactly 3 columns and one row per metabolic feature.
- Raw p-values and BH-adjusted pFDR values in the output fall within [0,1].
- If bootstrapping is applied: bootstrap_validation.csv exists, contains per-feature rows, and estimate and CI fields are finite.
- The association model is a Spearman correlation computed with disease_id="BMI" and confounder_ids=c("Age","Gender","T2D"), matching the article's method statement.
- BH correction (mt_method="BH") has been applied so every tested feature carries a pFDR value, not only raw p-values.

## Limitations

- The package implements univariate hypothesis testing; the article notes multivariate models are the alternative that previously 'do not properly account for cofounding factors (e.g. age, gender)' — this skill trades multivariate structure for explicit confounder handling.
- Bootstrapping (MWAS_bootstrapping) resamples subjects with replacement per feature, so compute scales with (features × iterations); judgment on whether bootstrap CIs validate the reported BMI associations is deferred to expert review.
- Requires R >= 3.4 per the article.

## Evidence

- [intro] A major limitation of existing multivariate statistical software packages for detecting metabolic changes is that they do not properly account for confounding factors such as age and gender: "However, a major limitation of these multivariate models from the epidemiological perspective is that they do not properly account for cofounding factors (e.g. age, gender), which might distort the"
- [intro] Key functionalities: "metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender)"
- [intro] bootstrapping of association models: "bootstrapping of association models; visualization of statistical outcomes; and metabolite assignment using Statistical Total Correlation Spectroscopy (STOCSY)"
- [abstract] MWAS abstract key functionalities: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
