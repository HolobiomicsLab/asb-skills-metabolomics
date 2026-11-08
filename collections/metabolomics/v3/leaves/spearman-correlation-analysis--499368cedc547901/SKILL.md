---
name: spearman-correlation-analysis
description: Use when ranking metabolic features (e.g. from 1H NMR metabonomic profiles)
  against a continuous phenotype such as BMI while adjusting for epidemiological confounders
  like Age, Gender, or T2D, and when bootstrap confidence intervals are needed to
  validate the associations.
when_to_use_negative:
- The analysis goal is multivariate modeling rather than univariate hypothesis testing
  — the package positions univariate MWAS with confounder handling as an alternative
  to multivariate software that does not properly account for confounders
- No epidemiological metadata (phenotype/confounder variables) is available to pass
  as disease_id or confounder_ids
- The goal is QC, STOCSY metabolite assignment, or network-based biological interpretation
  rather than association testing
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MWASTools
  role: Provides the confounder-adjusted partial-correlation MWAS model and the exported
    MWAS_bootstrapping function for non-parametric bootstrapping validation of Spearman
    associations
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which the MWAS association and bootstrapping functions
    are executed (MWASTools requires R >= 3.4)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:86828bec9e98636d89748c8edeae9d8c1ede69fccc99c545d763b41e308426ae
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# spearman-correlation-analysis

## Summary

Confounder-adjusted Spearman correlation between metabolic features and a phenotype (e.g. BMI) as implemented in MWASTools' MWAS model, plus non-parametric bootstrapping (MWAS_bootstrapping) to derive percentile confidence intervals that validate each metabolite-phenotype association.

## When to use

Use when ranking metabolic features (e.g. from 1H NMR metabonomic profiles) against a continuous phenotype such as BMI while adjusting for epidemiological confounders like Age, Gender, or T2D, and when bootstrap confidence intervals are needed to validate the associations.

## When NOT to use

- The analysis goal is multivariate modeling rather than univariate hypothesis testing — the package positions univariate MWAS with confounder handling as an alternative to multivariate software that does not properly account for confounders
- No epidemiological metadata (phenotype/confounder variables) is available to pass as disease_id or confounder_ids
- The goal is QC, STOCSY metabolite assignment, or network-based biological interpretation rather than association testing

## Inputs

- metabo_SE: a SummarizedExperiment/MultiAssayExperiment metabolomics data container
- Phenotype metadata with metabolite_id, disease_id (BMI), and confounder_ids (Age, Gender, T2D)
- User-set association method (e.g. spearman) and number of bootstrap iterations

## Outputs

- Per-feature confounder-adjusted Spearman association estimates
- Bootstrap estimate distributions with percentile-based confidence intervals
- Validation table written to bootstrap_validation.csv listing each feature's point estimate, bootstrap mean, and lower/upper CI bounds

## How to apply

Load the metabolomics data as a SummarizedExperiment/MultiAssayExperiment container (metabo_SE) whose phenotype metadata carries metabolite_id, disease_id (BMI), and confounder_ids (Age, Gender, T2D). For each selected metabolic feature, compute the Spearman correlation with BMI adjusted for the confounders, using partial regression on the confounder residuals as in the package's partial-correlation MWAS model; the package explicitly supports univariate hypothesis testing with efficient handling of epidemiological confounders. To validate a given feature's association, call MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id, confounder_ids, assoc_method, iterations), which resamples the study subjects with replacement for the configured number of iterations and recomputes the confounder-adjusted Spearman estimate in every replicate. Collect the per-feature bootstrap estimate distributions and derive percentile-based confidence intervals from them, then assemble a validation table with each feature's point estimate, bootstrap mean, and lower/upper CI bounds. Association method (e.g. spearman) and iteration count are user-set parameters.

## Related tools

- **MWASTools** (Provides the confounder-adjusted partial-correlation MWAS model and the exported MWAS_bootstrapping function for non-parametric bootstrapping validation of Spearman associations) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which the MWAS association and bootstrapping functions are executed (MWASTools requires R >= 3.4))

## Examples

```
MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id, confounder_ids = c("Age", "Gender", "T2D"), assoc_method = "spearman", iterations = 1000)
```

## Evaluation signals

- bootstrap_validation.csv exists and contains a row per metabolic feature
- Each feature row has finite point estimate, bootstrap mean, and lower/upper confidence-interval fields
- The point estimate matches the confounder-adjusted Spearman value obtained from the partial-correlation MWAS model on the full sample
- Bootstrap CI width is consistent with the configured number of iterations (wider CIs expected for few iterations)
- Judgment on whether the bootstrap CIs validate the reported BMI associations is deferred to expert review

## Limitations

- Existing multivariate statistical software for detecting metabolic changes does not properly account for confounding factors such as age and gender, which might distort observed associations between metabolites and the condition under study — confounder adjustment must be handled explicitly
- The bootstrap validation table alone does not settle validity: any judgment on whether the bootstrap CIs validate the reported BMI associations is deferred to expert review
- CIs depend on the user-set number of bootstrap iterations; too few iterations yields unstable percentile intervals

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders.: "Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders."
- [intro] metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender): "metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender)"
- [intro] bootstrapping of association models; visualization of statistical outcomes: "bootstrapping of association models; visualization of statistical outcomes"
